---
title: "6.3 IRQ 스레드는 어떻게 생성할까?"
date: 2026-05-14 15:00:00 +0900
categories: [Linux, Kernel]
tags: [linux, kernel, os]
---
IRQ스레드를 실행하려면 우선 IRQ스레드를 생성해야함. 일반적으로는 부팅 과정에서 IRQ스레드를 한 번 생성하고 이후에 인터럽트 핸들러에서 IRQ스레드를 깨우는 방식으로 IRQ스레드가 동작함.

**이번 게시물에서 분석할 함수 목록**
- request_threaded_irq()
- setup_irq_thread()
## 6.3.1 IRQ 스레드는 언제 생성할까?


