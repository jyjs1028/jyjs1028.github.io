---
title: "Linux Work Queue 분석: ftrace로 Soft IRQ와 latency 비교하기"
date: 2026-06-01 00:00:00 +0900
categories: [Linux, Kernel]
tags: [linux, kernel, workqueue, softirq, ftrace, bottom-half, raspberry-pi, threaded-irq]
---

## 개요

지난 포스트에서 Bottom Half 중 하나인 Soft IRQ를 ftrace로 관찰했다.  
이번에는 또 다른 Bottom Half인 **Work Queue**를 분석하고, Soft IRQ와 latency를 직접 비교해본다.

실습 환경은 동일하게 Raspberry Pi 3A+이며, ftrace를 사용해 실제 수치를 측정했다.

---

## Bottom Half 전체 구조

인터럽트 처리는 Top Half와 Bottom Half로 나뉜다.

```
하드웨어 인터럽트 발생
       ↓
  [Top Half]  → 최소한의 작업만, 빠르게 처리
       ↓
  [Bottom Half] → 나머지 무거운 작업 지연 처리
```

Bottom Half에서 네 가지 메커니즘을 우리가 확인했는데 다음과 같다.

| 메커니즘 | 컨텍스트 | 동시 실행 | Sleep 가능 |
|---|---|---|---|
| Soft IRQ | Interrupt | ✅ (멀티코어) | ❌ |
| Tasklet | Interrupt | ❌ | ❌ |
| **Work Queue** | **Process** | ✅ | ✅ |
| Threaded IRQ | Process | ✅ | ✅ |

---


## Work Queue란?

Work Queue는 Bottom Half 메커니즘 중 **Process 컨텍스트**에서 실행되는 범용 지연 처리 방식이다.

Soft IRQ와 Tasklet은 인터럽트 컨텍스트라 Sleep이 불가능하지만, Work Queue는 일반 프로세스처럼 스케줄링되기 때문에 Sleep, mutex, 메모리 할당 등 블로킹 작업이 가능하다.

---

## Work Queue 동작 원리

Work Queue는 내부적으로 **kworker** 커널 스레드들이 처리한다.

비유하면:
```
Work Queue = 작업 접수 창구
kworker    = 실제 일하는 직원
```

커널 코드가 창구에 작업을 맡기면, kworker가 가져다가 처리하는 구조다.

**실제 흐름:**

```
1. 커널 코드가 queue_work() 호출
         ↓
2. work가 workqueue 대기열에 등록됨  (pending 상태)
         ↓
3. workqueue_activate_work           (active 상태로 전환)
         ↓
4. kworker 스레드가 work 가져다가 실행
```

**pending vs active:**

Work Queue는 동시에 실행할 수 있는 work 수에 제한이 있다. 시스템이 한가하면 등록되자마자 바로 activate되지만, work가 많이 쌓이면 pending 상태로 대기하다가 순서가 되면 activate된다.

아까 ftrace에서 본 게 정확히 이 흐름이다:

```
sshd-session  → workqueue_queue_work    ← 2번: work 등록
kworker/u17:0 → workqueue_execute_start ← 4번: kworker가 실행 시작
kworker/u17:0 → workqueue_execute_end   ← 4번: 실행 완료
```

**kworker가 별도 스레드인 이유:**

인터럽트 핸들러 안에서 직접 처리하면 Sleep이 필요한 작업을 처리할 수 없다. kworker는 일반 커널 스레드라서 Sleep이 필요한 시점에 스케줄러가 다른 프로세스를 실행하고, 작업이 완료되면 kworker를 다시 깨울 수 있다.

---

## kworker 구조

```bash
$ ps aux | grep kworker | head -10
root     4  [kworker/R-kvfree_rcu_reclaim]
root     5  [kworker/R-rcu_gp]
root    12  [kworker/u16:0-ipv6_addrconf]
root    38  [kworker/u17:0-kvfree_rcu_reclaim]
root    57  [kworker/3:1-events]
root    58  [kworker/3:1H-mmc_complete]
```

**kworker 이름 패턴 읽는 법:**

| 패턴 | 의미 |
|---|---|
| `kworker/3:1` | CPU3에 바인딩된 worker |
| `kworker/u17:0` | unbound worker (CPU 제한 없음) |
| `kworker/R-writeback` | rescuer thread (긴급 처리용) |

ksoftirqd와 비교하면:
```
ksoftirqd/0~3  → CPU별 딱 4개 고정
kworker        → 워크큐 종류별로 여러 개, 동적 생성/삭제
```

---

## Bound vs Unbound Worker

**bound worker (예: kworker/3:1)**

특정 CPU에 고정(CPU affinity)되어 그 CPU에서만 실행된다.

```
장점: 같은 CPU에서 반복 실행 → CPU 캐시 재사용 → 빠름
단점: 해당 CPU가 바빠도 다른 CPU로 못 감
용도: 특정 하드웨어와 연관된 작업 (해당 CPU에 연결된 장치 인터럽트 등)
```

**unbound worker (예: kworker/u17:0)**

CPU 제한이 없어 스케줄러가 여유 있는 CPU에서 실행한다.

```
장점: CPU 부하 분산 가능, 유연한 실행
단점: 매번 다른 CPU → CPU 캐시 재사용 어려움
용도: 특정 CPU에 종속될 필요 없는 범용 작업
```

**실습에서 확인한 것:**

```
kworker/0:2-64    [000]  → CPU0에서 work 등록
kworker/u17:0-61  [003]  → CPU3에서 실행
```

work를 등록한 건 CPU0인데, CPU3의 unbound worker가 여유가 있어서 CPU3에서 실행됐다. bound worker였으면 무조건 CPU0에서만 실행됐을 것이다.

---

## 실습 환경

- **보드**: Raspberry Pi 3 Model A+
- **CPU**: ARM Cortex-A53 Quad-core 1.4GHz
- **OS**: Raspberry Pi OS Lite 64-bit
- **커널**: 6.12.75+rpt-rpi-v8

---

## 실습 1: ftrace 이벤트 확인

Work Queue 관련 ftrace 이벤트 목록 확인:

```bash
$ ls /sys/kernel/debug/tracing/events/workqueue/
enable  filter  workqueue_activate_work  workqueue_execute_end
workqueue_execute_start  workqueue_queue_work
```

| 이벤트 | 의미 |
|---|---|
| `workqueue_queue_work` | work가 workqueue 대기열에 등록됨 (pending) |
| `workqueue_activate_work` | pending → active 상태로 전환 |
| `workqueue_execute_start` | kworker가 work 실행 시작 |
| `workqueue_execute_end` | kworker가 work 실행 완료 |

`queue_work` → `execute_start` 간격이 Work Queue의 **latency**다.

---

## 실습 2: Soft IRQ latency 측정

비교 기준을 위해 먼저 Soft IRQ latency를 측정한다.

```bash
cd /sys/kernel/debug/tracing
sudo sh -c 'echo 0 > tracing_on'
sudo sh -c 'echo > trace'
sudo sh -c 'echo "irq:softirq_raise irq:softirq_entry irq:softirq_exit" > set_event'
sudo sh -c 'echo 1 > tracing_on'
sleep 5
sudo sh -c 'echo 0 > tracing_on'
sudo cat trace | grep NET_RX | head -20
```

**실제 측정 결과:**

```
kworker/u17:0-61  [002]  134.464961: softirq_raise: vec=3 [action=NET_RX]
kworker/u17:0-61  [002]  134.464963: softirq_entry: vec=3 [action=NET_RX]
kworker/u17:0-61  [002]  134.465017: softirq_exit:  vec=3 [action=NET_RX]

kworker/u17:0-61  [002]  134.470649: softirq_raise: vec=3 [action=NET_RX]
kworker/u17:0-61  [002]  134.470651: softirq_entry: vec=3 [action=NET_RX]
kworker/u17:0-61  [002]  134.470681: softirq_exit:  vec=3 [action=NET_RX]
```

**latency 계산:**

```
첫 번째:
  raise → entry (대기시간): 134.464963 - 134.464961 = 2µs
  entry → exit  (처리시간): 134.465017 - 134.464963 = 54µs

두 번째:
  raise → entry (대기시간): 134.470651 - 134.470649 = 2µs
  entry → exit  (처리시간): 134.470681 - 134.470651 = 30µs
```

Soft IRQ는 등록 즉시 인터럽트 컨텍스트에서 실행되므로 대기시간이 **2µs**로 매우 짧다.

---

## 실습 3: Work Queue latency 측정

```bash
sudo sh -c 'echo > trace'
sudo sh -c 'echo "workqueue:workqueue_queue_work workqueue:workqueue_execute_start workqueue:workqueue_execute_end" > set_event'
sudo sh -c 'echo 1 > tracing_on'
sleep 5
sudo sh -c 'echo 0 > tracing_on'
sudo cat trace | grep brcmf | head -20
```

**실제 측정 결과:**

```
kworker/0:2-64    [000]  172.963040: workqueue_queue_work: function=brcmf_sdio_dataworker workqueue=brcmf_wq/mmc1:0001:1
kworker/u17:0-61  [003]  172.963067: workqueue_execute_start: function brcmf_sdio_dataworker
kworker/u17:0-61  [003]  172.963298: workqueue_execute_end:   function brcmf_sdio_dataworker

sshd-session-1019 [000]  172.963548: workqueue_queue_work: function=brcmf_sdio_dataworker workqueue=brcmf_wq/mmc1:0001:1
kworker/u17:0-61  [003]  172.963573: workqueue_execute_start: function brcmf_sdio_dataworker
kworker/u17:0-61  [003]  172.963652: workqueue_execute_end:   function brcmf_sdio_dataworker
```

> `brcmf_sdio_dataworker`는 라즈베리파이의 Wi-Fi 드라이버(brcmfmac)가 사용하는 Work Queue다. SSH 통신이 발생할 때마다 Wi-Fi 데이터 처리를 위해 호출된다.

**latency 계산:**

```
첫 번째:
  queue → execute_start (대기시간): 172.963067 - 172.963040 = 27µs
  execute_start → end   (처리시간): 172.963298 - 172.963067 = 231µs

두 번째:
  queue → execute_start (대기시간): 172.963573 - 172.963548 = 25µs
  execute_start → end   (처리시간): 172.963652 - 172.963573 = 79µs
```

Work Queue는 스케줄러를 거쳐 kworker 스레드가 실행되므로 대기시간이 **25~27µs**로 Soft IRQ보다 길다.

---

## 최종 비교: Soft IRQ vs Work Queue

| 항목 | Soft IRQ (NET_RX) | Work Queue (brcmf_sdio) |
|---|---|---|
| 대기시간 (등록→실행) | **2µs** | **25~27µs** |
| 처리시간 | 30~54µs | 79~231µs |
| 컨텍스트 | Interrupt | Process |
| Sleep 가능 | ❌ | ✅ |
| 처리 주체 | ksoftirqd / 인터럽트 핸들러 | kworker 스레드 |

**핵심 결론:**

Soft IRQ는 스케줄링 없이 인터럽트 컨텍스트에서 바로 실행되므로 latency가 극히 짧다. 반면 Work Queue는 kworker 스레드가 스케줄링을 거쳐 실행되므로 latency가 상대적으로 길지만, Process 컨텍스트에서 동작하므로 Sleep과 블로킹 작업이 가능하다.

이것이 네트워크 수신처럼 극한의 성능이 필요한 경우 Soft IRQ를 사용하고, Wi-Fi 드라이버처럼 복잡한 I/O 처리가 필요한 경우 Work Queue를 사용하는 이유다.

---

## 참고

- [Linux Kernel Source - kernel/workqueue.c](https://elixir.bootlin.com/linux/latest/source/kernel/workqueue.c)
- [Linux Kernel Source - kernel/irq/manage.c](https://elixir.bootlin.com/linux/latest/source/kernel/irq/manage.c)
- Linux Kernel Development, Robert Love (3rd Edition) — Chapter 8
- [kernel.org ftrace documentation](https://www.kernel.org/doc/html/latest/trace/ftrace.html)