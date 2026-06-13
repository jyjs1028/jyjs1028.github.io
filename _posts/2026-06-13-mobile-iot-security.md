---
title: "Mobile & IoT Security — iOS/Android 비교·SIM Cloning·Permission·Adware 정리"
date: 2026-06-13 02:00:00 +0900
categories: [Security, Software Security]
tags: [mobile-security, ios, android, sim-cloning, permission, adware, malware, webview, accessibility-service, sonicspy]
---

> 보안 강의 Chapter 7 Mobile & IoT Security 내용을 정리한 학습 노트입니다.
>
> 큰 흐름: **① iOS vs. Android 보안 철학 비교 → ② Mobile Network Security (SKT 사건 중심) → ③ Permission-based Access Control → ④ SonicSpy (스파이웨어 사례) → ⑤ Accessibility Service 악용 → ⑥ Adware (Judy 사례 + WebView)**

---

## 1. iOS vs. Android — 4가지 축으로 비교

핵심 프레임: **"iOS = 통제, Android = 유연성"**

### (1) App Sandboxing — Very strict vs. Flexible

**App Sandboxing**(앱 샌드박싱)은 OS가 앱들을 **서로 격리(isolate apps from each other)** 하고 시스템으로부터도 격리하는 보안 기법이다. 각 앱은 자신만의 **"sandbox"** (제한된 환경)에서 실행 → 자기 파일·데이터만 접근 가능, 특별 권한 없이는 다른 앱 데이터나 시스템 파일에 접근 불가.

**Android (상대적으로 Flexible):**
- 모든 앱에 **unique user ID (UID)** 부여
- 기본적으로 자기 파일/데이터만 접근, 다른 앱 데이터·시스템 자원은 적절한 권한 없으면 차단
- 앱 간 데이터 공유는 **Intents, ContentProviders** 등 **IPC (inter-process communication)** 로 가능
- 사용자 권한이 있으면 **shared storage, camera, location** 등 접근 가능

### (2) Rooting / Jailbreak (Kernel Access) — Strictly prohibited vs. Not prohibited

iOS는 **엄격히 금지**, Android는 **금지하지 않음** (다만 권장하지 않음).

**Android의 Rooting:**
- 구조적으로 rooting을 허용. **AOSP (Android Open Source Project)** 기반이라 누구나 OS 수정, custom kernel 빌드, custom ROM 설치 가능
- 대부분의 제조사(**OEMs**)가 **unlock the bootloader**(부트로더 잠금 해제) 공식 방법 제공 → custom firmware 설치 및 root 권한 획득 가능
- Android는 보안 위험을 **경고(warns about security risks)** 하지만 완전히 막지는 않음

### (3) App Signing — Strict regulation vs. Flexible

**App Signing**(앱 서명)은 앱의 **authenticity, integrity, trustworthiness**(진위성·무결성·신뢰성)를 보장하는 보안 메커니즘이다. OS가 서명을 검증해 확인하는 것: ① 신뢰할 수 있는 개발자가 만들었는지, ② 서명 후 변조되지 않았는지.

iOS와 Android 모두 설치를 위해 app signing을 엄격히 요구한다. 차이점은 다음과 같다:

| 구분 | iOS | Android |
|------|-----|---------|
| **Certificate Issuer** | **Apple-issued certificate** 필수 | 개발자가 **자체 키 생성 가능** (self-signed certificate) |
| **Verification** | **install + runtime 모두** 검증 | **install time only** (런타임에서는 반드시 검증하지 않음) |

### (4) App Management

**App Source (Marketplace):**
- iOS: **Apple App Store만**
- Android: **Play Store + 외부 소스** + **sideloading(.apk)** 까지 허용

**Sideloading**: 공식 앱스토어 밖에서 앱을 설치하는 것.
- Android: 웹사이트·이메일·USB로 **APK file** 받아 수동 설치 가능. 제조사 자체 마켓(예: Galaxy Store)도 존재
- iOS: 훨씬 제한적, 보통 **developer tools, TestFlight, 또는 jailbreaking** 필요

**App Vetting Process:**
- Apple App Store: **Manual vetting** (사람이 직접 검토)
- Google Play Store: **Automated vetting** (자동 검수)

### 결론

| | iOS | Android |
|---|---|---|
| 보안 철학 | **Security Through Control** (통제를 통한 보안) | **Security Through Flexibility** (유연성을 통한 보안) |
| 우선 가치 | 일관된 보안을 위해 control 우선 | openness와 진화하는 보안 기능 사이의 균형 |

---

## 2. Mobile Network Security — SKT SIM Cloning 사건 중심

### 인증의 필요성

모바일 네트워크에서 가입자 본인임을 증명하는 식별 수단:
- **IMEI (International Mobile Equipment Identity)** — 기기 식별
- **IMSI (International Mobile Subscriber Identity)** — 가입자 식별

### IMEI

- 셀룰러 모뎀을 가진 모든 기기에 **제조사가 부여하는 고유 번호** — 폰의 "serial number for the mobile network"
- **박스에 적혀 있음 → NOT a secret (비밀이 아님)**
- **사용자가 아니라 물리적 기기(physical device)** 를 식별
- 정상 조건에서는 **변경·수정 불가 (cannot be changed or modified)**
- **결론: IMEI만으로는 인증 목적에 충분하지 않음 (NOT sufficient)**

### SIM과 IMSI

**SIM card (Subscriber Identity Module):** 셀룰러 네트워크에서 **사용자를 식별·인증**하는 작은 스마트카드.
- 실리콘 칩이 내장된 플라스틱 카드 (CPU + memory, 최대 256 KB 저장)
- 각 SIM 카드는 **고유한 IMSI와 KI 값**을 가짐

가입 시 통신사가 새 SIM 발급 (예: IMSI: 9876, KI: abcd):
- 통신사는 **IMSI·KI·customer 매핑 정보를 SIM에 미리 심고(embed), 데이터베이스에 저장**
- 이 Customer Database가 바로 **HSS (Home Subscriber Server)**

### HSS (Home Subscriber Server)

셀룰러 네트워크의 **중앙 데이터베이스(central database)**. 저장 항목:

```
IMSI, KI, customer, Phone number, Service Profile, Base Station Location
```

### 인증 과정과 KI

기지국에 연결할 때:
1. SIM 카드가 **IMSI를 전송**
2. 통신사가 **HSS와 대조해 검증**하고 네트워크 서비스 제공

**KI (Authentication Key):**
- 사용자 SIM과 통신사 서버 양쪽에 안전하게 저장되는 **secret 128-bit 대칭키(Symmetric Encryption Key)**
- 기기 연결 시 **SIM과 네트워크 양쪽이 KI로 암호 연산** 수행 → **KI 자체를 노출하지 않고** 가입자 신원 확인
- 연결 요청 형태: `IMSI + KI로 암호화된 데이터`

### SKT HSS 해킹 → SIM Cloning

**유출 가능 정보**: IMEI, IMSI, KI, customer information (전화번호, 주민번호, 주소 등)

공격자가 Alice의 **IMSI, KI, IMEI**를 알면:
1. 이를 **새 SIM 카드에 주입(inject)**
2. 이 SIM으로 통신사 서비스에 접속
3. SIM이 정상적으로 **인증을 통과(proceed with the authentication)**
4. 서비스는 **공격자 폰을 Alice 폰으로 착각**

**SIM Cloning**: SIM 카드의 고유 credential을 복사해 **복제(duplicating)** 함으로써, 공격자가 원래 사용자를 **사칭(impersonate)** 하는 것.

결과:
- **Account takeover** (계정 탈취)
- **SMS interception** (문자 가로채기 — **2FA codes** 포함)
- **Identity theft** (신원 도용·부정 사용)

### 대응책 3종

**① IMSI Reattachment Denial**
- **Fraud Detection System (FDS)** 강화: 이미 연결된 상태에서 **중복(Duplicated) 연결 요청이 오면 거부(Refuse)**
- 단점: 고객은 스마트폰을 **재부팅하면 안 됨 (must NOT reboot)** — 재부팅 시 재연결 과정에서 틈이 생김

**② USIM Protection Service**
- IMSI·KI에 더해, **IMEI가 이력(history)과 일치하는지** 추가 확인
- **한계**: IMEI 정보까지 유출되었다면 → 무력화. 공격자가 IMEI를 위조하면 통과. 또한 고객이 단순히 SIM만 옮겨서는 **폰 교체 불가**

**③ Reissuance (재발급)**

| 방법 | 내용 | 비고 |
|------|------|------|
| **IMSI reissuance** (USIM format by software) | 통신사가 IMSI 재발급, 고객은 **OTA (Over The Air)** 로 다운로드 | KI는 immutable이라 변경 불가 |
| **IMSI and KI reissuance** (USIM replacement) | 고객이 **SIM 카드를 교체** → IMSI와 KI 둘 다 변경 | **더 나은 해결책 (Better solution)** |

> **KI는 immutable**: SIM 카드에 한 번 심으면 **변경·재프로그래밍 불가**. 이 불변성이 인증 과정의 무결성 유지에 필수.

---

## 3. Permission-based Access Control

### Permission이란

**Permission**(권한)은 모바일 기기에서 **민감한 자원(sensitive resources)** — 예: camera, microphone, location — 에 대한 접근을 통제하는 보안 메커니즘이다. 앱은 그런 자원을 쓰기 전에 반드시 **사용자 승인을 요청·획득** 해야 한다.

- 사용자 프라이버시를 보호하고, 악성 앱이 민감 데이터·기기 기능에 접근하는 것을 **방지**
- 사용자가 **각 앱이 무엇을 할 수 있는지 통제**하도록 도움

### 실제로 도움이 되는가?

Permission은 기능별로 **매우 세분화(highly granular)** 되어 있어 종류가 매우 많다:
- **Android: 100+ permissions** (Normal, Dangerous, Signature, Appop로 그룹화)
- **iOS: 20–30개의 major permissions**

권한 종류가 너무 많고 앱마다 요구 집합이 달라서, 사용자는 결과를 인지하지 못한 채 권한 요청을 **무시하거나 자동으로 허용(tend to ignore or automatically allow)** 하는 경향이 있다.

### 권한 남용 탐지가 어려운 이유

권한 사용 자체가 **앱이 악성이라는 뜻은 아님** — 정상 앱(benign apps)도 수많은 권한을 사용한다. 권한 남용을 식별하려면, 관련 API가 **어떤 맥락(context)** 에서 쓰이는지를 이해하는 것이 중요하다.

예시: `ACCESS_FINE_LOCATION`

| 앱 유형 | 사용 방식 |
|---------|----------|
| **Benign apps** | 지도 앱이 사용자 위치를 pinpoint·display |
| **Malicious apps** | **백그라운드(in the background)** 에서 위치를 추적해 **C&C server** 로 전송 |

---

## 4. SonicSpy (2017) — 스파이웨어 사례

### 개요

- **SonicSpy**: Android 스마트폰을 노린 mobile malware family
- 2017년 2~8월 사이 **4,000개 이상의 샘플**이 Google Play 등에 숨겨져 배포됨
- 정상 앱(특히 messaging apps)으로 위장, 공식 마켓을 통해 배포 → **supply chain attack**

### 감염 과정

```
1. Disguise: 정상 메시징 앱으로 위장 (예: Telegram 커스텀 버전 "Soniac")
2. Icon Removal After Installation: 설치 직후 런처 아이콘 제거 → 사용자 탐지 회피
3. C&C Server Connection: C&C 서버에 연결해 원격 명령 수신 (73+ commands)
4. Malicious Activity Execution: audio recording, photo capture, data exfiltration 수행
```

### 남용 권한과 행위

**권한 남용:**

| 권한 | 목적 |
|------|------|
| `RECORD_AUDIO` | 마이크로 ambient sound·phone calls 녹음 |
| `CAMERA` | 사용자 모르게 사진 무음 촬영(silently) |
| `SEND_SMS` | 공격자 통제 번호로 SMS 전송 |
| `RECEIVE_SMS, READ_SMS` | 수신 문자 가로채기 (**2FA codes** 포함) |
| `READ_CONTACTS` | 연락처 탈취 → 추가 phishing/spam |
| `READ_CALL_LOG` | 통화 기록 수집·발신 통화 감시 |
| `ACCESS_FINE_LOCATION` | 정밀 GPS 위치 추적 |
| `READ_PHONE_STATE` | 기기 정보 수집 (IMEI, carrier 등) |
| `WAKE_LOCK, RECEIVE_BOOT_COMPLETED` | **재부팅 후에도 백그라운드 지속 동작** 유지 |

**행위:**
- **Surveillance**: 대화 녹음, 사진 캡처, 실시간 위치 추적, SMS·통화 기록 모니터링
- **Data Exfiltration**: 탈취 데이터를 원격 서버로 업로드
- **Persistence**: 앱 아이콘 숨김, 부팅 시 자동 실행, 코드 난독화로 탐지 회피
- **Command Execution**: 원격 카메라·오디오 녹화 트리거, 추가 payload 다운로드

---

## 5. Accessibility Service 악용

### Accessibility Service란

장애 사용자(시각·운동 장애 등)를 돕기 위해 설계된 **특수 Android framework**. 앱이 **다른 앱의 UI를 관찰·상호작용** 할 수 있게 한다 — 화면 텍스트 읽기, 탭 수행, 자동 스크롤 등 → **매우 강력한 기능(powerful capabilities)**.

악성코드가 남용하면 **전체 시스템에 걸쳐(entire system) UI 요소를 관찰·제어** 가능 → Android 생태계에서 **가장 강력한 공격 표면(most powerful attack surfaces)** 중 하나가 된다.

사례: **Android.BankBot, Android.Joker (a.k.a. Bread), EventBot**

### 악용 능력 8가지

| # | 능력 | 설명 |
|---|------|------|
| 1 | **Keylogging** | 텍스트 필드 입력(비밀번호, 카드번호, 채팅) 읽기 |
| 2 | **UI Content Snooping** | 다른 앱 화면 내용(알림, 메시지, 로그인 화면) 관찰 |
| 3 | **Click Automation** | 탭·스와이프·스크롤·텍스트 입력 시뮬레이션 → 전체 사용자 흐름 자동화 |
| 4 | **Silent App Installation** | 설치 대화상자를 자동 클릭해 **사용자 동의 없이 스스로 권한 부여** |
| 5 | **Blocking Uninstallation** | 가짜 UI 오버레이나 삭제 동작 가로채기로 **악성코드 제거 방지** |
| 6 | **Fraudulent Purchases** | "Subscribe" 버튼 자동 클릭으로 사용자 모르게 프리미엄 SMS 결제 |
| 7 | **Overlay Phishing** | 정상 앱(예: 뱅킹 앱) 위에 **가짜 로그인 화면**을 띄워 credential 탈취 |
| 8 | **Full Device Automation** | 위 모든 것을 결합 → 앱 실행·로그인·**송금**까지 완전 자동화 |

---

## 6. Adware

### Adware란

사용자 기기에 **원치 않는 광고(unwanted advertisements)** 를 종종 동의 없이 표시하는 소프트웨어. 흔히 **PUA (Potentially Unwanted Application)** 로 불린다. **monetization(수익화)과 exploitation(악용) 사이의 grey zone(회색지대)** 에서 작동한다.

### Ad Networks 과금 모델

**Ad Networks**: Advertisers(광고를 띄우려는 측)와 publishers / app developers(광고 공간을 파는 측)를 연결하는 플랫폼.

| 모델 | 의미 |
|------|------|
| **CPM** (Cost Per Mille) | **1,000회 노출(impressions/views)** 당 과금 |
| **CPC** (Cost Per Click) | 사용자가 **클릭**할 때 과금 |
| **CPA** (Cost Per Action) | 특정 **행동 완료**(install, sign-up) 시에만 과금 |
| **CPI** (Cost Per Install) | CPA의 특화형. 광고 통해 **앱 설치** 시 과금 |

핵심: **개발자가 광고를 더 많이 보게 할수록 더 많은 수익** → adware의 악용 동기

악성 adware의 수익 극대화 수법:
- **Fake impressions/clicks** (click fraud)
- **Auto-click** (JavaScript 또는 **Accessibility Services** 이용)
- **background WebViews** 에 숨겨진 광고 로드

### Judy (2017) — 대표 Adware 사례

- **Check Point** 가 2017년 5월 발견한 대규모 Android adware campaign
- Google Play에 게시된 **40개 이상의 앱**에 내장, 주로 한국 회사 **ENISTUDIO** 가 개발
- **백그라운드에서 자동으로 광고를 클릭**해 부정 광고 수익 생성
- 규모: **41개 이상**의 앱(fashion·cooking·pet care 테마), 다운로드 **850만~3,650만** 추정

**감염 과정:**

```
1. App Store Vector: Google Play에서 무해한 게임/유틸리티("Fashion Judy")로 믿고 다운로드
2. Delayed Activation: 악성 행위를 설치 후에 활성화(실행 즉시가 아님) → 의심 감소
3. Payload Download: C&C 서버에 접속해 실제 악성 코드(JavaScript + ad-clicking logic) 다운로드
4. Background Execution: payload를 hidden WebView에서 실행 → 광고 상호작용이 보이지 않음
```

Detection Difficulty: 앱들이 초기에는 **benign·functional**(정상·작동) → Google 앱 심사 우회

### WebView

Android·iOS 같은 모바일 OS가 제공하는 **built-in component**로, 앱 내부에서 **웹 콘텐츠(HTML, JavaScript, CSS) 표시**를 가능하게 한다.

일반적 용도: 앱 내 외부 웹페이지 표시, in-app 문서/가이드 렌더링, 하이브리드 앱(Cordova, React Native), 결제 게이트웨이·embedded third-party services

### WebView 악용 3가지

**① JavaScript Interface Abuse**

Android에서 개발자가 **native Java method를 JavaScript에 노출**(`addJavaScriptInterface`) 가능. 안전하게 처리하지 않으면 **악성 JavaScript가 민감한 동작을 트리거**한다.

```java
// 예시: 악성 웹페이지가 호출 가능한 함수를 노출
window.android.getContacts()
window.android.executeCommand()
window.android.readFile()
window.android.sendSMS()
```

**② Phishing within WebView**

정상 사이트처럼 보이는 **가짜 로그인 페이지**를 렌더링해 credential 캡처. 예: 가짜 뱅킹 로그인 페이지로 아이디·비밀번호 수집.

**③ WebView Injection**

로드되는 콘텐츠가 제대로 **sanitize(정제)** 되지 않으면, 공격자가 **URL 파라미터나 외부 HTML 파일을 통해 악성 스크립트 주입**. 앱이 **untrusted input** (URL 파라미터, HTML 조각, 사용자 생성 콘텐츠)을 정제 없이 WebView에 로드할 때 발생.

```java
String unsafeHtml = "<html><body>" + getIntent().getStringExtra("html") + "</body></html>";
webView.loadData(unsafeHtml, "text/html", "UTF-8");
// 예: html=<script>alert('hacked')</script> → 스크립트 주입 가능
```

### Judy가 WebView를 악용하는 방식

```
1. Invisible WebView: 0×0 px (또는 display:none) WebView를 생성 → 광고 요소가 보이지 않음
2. Off-Screen Placement: WebView를 화면 밖(음수 X/Y 좌표)에 배치
3. Background Loading: 사용자가 앱을 최소화하거나 벗어난 후에만 광고 URL을 로드 → 백그라운드에서 자동 클릭
4. No UI Feedback: toast 메시지·알림·애니메이션을 억제 → 어떤 흔적도 남기지 않음
```

---

## 전체 흐름 정리

```
iOS vs. Android
└── 4개 축(Sandboxing, Rooting, Signing, App Management) → 통제 vs. 유연성

Mobile Network Security
└── IMEI(기기) ≠ 인증
    → SIM의 IMSI(가입자) + KI(대칭키) → HSS 중앙 DB
    → 유출 시 SIM Cloning (account takeover, SMS interception, 2FA bypass)
    → 대응책 3종: IMSI Reattachment Denial / USIM Protection / Reissuance

Permission
└── 보호 수단이지만 너무 granular해서 사용자가 무심코 허용
    → 남용 탐지는 context(맥락)가 핵심

SonicSpy
└── 위장 + 아이콘 제거 + C&C + 다수 권한 남용 → 감시·유출형 스파이웨어

Accessibility Service
└── 접근성 기능 악용 → 시스템 전반 UI 제어 → 8대 악용 능력

Adware (Judy)
└── 광고 과금 모델(CPM/CPC/CPA/CPI) 악용
    → hidden WebView로 백그라운드 클릭 사기
    → WebView 악용 3종: JS Interface / Phishing / Injection
```

---

**Key Terms**

`App Sandboxing` / `AOSP` / `Sideloading` / `App Signing` / `IMEI` / `IMSI` / `KI` / `HSS` / `SIM Cloning` / `OTA` / `FDS` / `USIM Protection` / `Permission` / `ACCESS_FINE_LOCATION` / `Accessibility Service` / `Overlay Phishing` / `PUA` / `CPM` / `CPC` / `CPA` / `CPI` / `WebView` / `addJavaScriptInterface` / `WebView Injection` / `C&C server` / `SonicSpy` / `Judy`
