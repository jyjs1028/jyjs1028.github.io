---
title: "Blockchain — 구조·채굴·합의·PoS·스마트 컨트랙트 완전 정리"
date: 2026-06-13 01:00:00 +0900
categories: [Security, Software Security]
tags: [blockchain, bitcoin, pow, pos, consensus, merkle-tree, smart-contract, 51-attack, double-spending]
---

>
> 큰 흐름: **중앙집중형의 한계 → 비트코인의 등장 동기 → 블록체인 구조 → 채굴(Mining)과 PoW → 합의(Consensus)와 공격 → PoW의 한계 → PoS와 변형 → 스마트 컨트랙트**

---

## 1. 중앙집중형 구조와 그 한계

### Server-Client Model (Centralized Architecture)

- 모든 데이터를 **중앙 권한 또는 서버(central authority or server)** 가 관리·저장하는 구조
- 사용자는 그 중앙 주체가 데이터/서비스를 제대로 처리할 것이라고 **신뢰(trust)** 해야만 함

**장점(Pros)**
- 관리가 단순함 (Simple to manage)
- 빠른 거래 처리 (Fast transaction processing)

**단점(Cons)**
- 하나의 오작동이 전체 시스템을 마비시킴 → **단일 장애점(Single Point of Failure, SPOF)**
- **해킹(hacking)** 이나 내부자 **조작(manipulation)** 에 취약
- **검열(censorship)** 과 사용자 통제가 가능함

### 분산 구조의 필요성

봇넷이 중앙집중형의 단점(SPOF)을 극복하기 위해 **peer-to-peer(P2P) model** 을 활용한 것처럼, 금융 시스템도 **탈중앙(Decentralized) + 분산(Distributed)** 아키텍처가 필요하다는 인식이 생겨났다.

- **P2P**: 각 호스트가 **클라이언트이면서 동시에 서버(client and a server)** 역할을 함
- **Decentralized**: 중앙 권한이 없음
- **Distributed**: 데이터가 여러 곳에 분산 저장됨

---

## 2. 경제적 배경 — 비트코인 등장 동기

비트코인은 **2009년에 출시(launched in 2009)** 됐다. 그렇다면 2008년에 무슨 일이 있었는가?

### 2008 글로벌 금융위기

**모기지(Mortgage)** 란 주택을 **담보(collateral)** 로 잡고 받는 대출이다. 상환하지 못하면 대출자(lender)가 그 부동산을 가져간다.

**LTV (Loan-to-Value)**: 담보의 감정가 대비 대출금의 비율로, **대출 위험(lending risk)** 을 평가하는 데 사용된다.

```
집값 10억, LTV 70% → 최대 7억까지 대출 가능
집값이 0.6억으로 하락 → 부채(Debt) 0.7B > 담보(Collateral) 0.6B
→ 담보 가치보다 빚이 커지는 상황
```

2008년 9월, Lehman Brothers 같은 대형 금융기관이 붕괴하고 **글로벌 신용 경색(global credit crunch)** 이 발생했다. 정부와 중앙은행이 **대규모 구제금융(massive bailouts)** 을 시행하면서 **대중의 분노(public outrage)** 와 **은행 시스템에 대한 불신(distrust in the banking system)** 이 커졌다.

> 2008년 위기 → 중앙 기관에 대한 신뢰 붕괴 → "신뢰가 필요 없는(trustless)" 시스템에 대한 갈망 → 비트코인

---

## 3. 비트코인과 이중지불 문제

2008년 10월, **"Satoshi Nakamoto(닉네임)"** 가 *"Bitcoin: A Peer-to-Peer Electronic Cash System"* 논문을 발표했다.

### Double-spending Problem (이중지불 문제)

- **이중지불**: 디지털 화폐가 **데이터를 복제(조작)하여** **두 번 이상 사용(spent more than once)** 되는 위험
- 이유: 디지털 파일은 **쉽게 복제 가능(easily copyable)**
- 실물 현금은 한 번에 한 장소에만 존재하지만, 디지털 토큰은 복제 방지를 위한 추가 메커니즘이 필요

**탈중앙에서의 문제 시나리오** (User 잔액: 1 coin)

```
1. User가 1 디지털 코인을 Alice에게 보냄
2. 동시에 같은 코인을 Bob에게도 보냄
3. 시스템이 어떤 거래가 유효한지 판단할 수 없으면, 둘 다 성공한 것처럼 보일 수 있음
```

**공유 공개 원장(shared public ledger)** 이 없으면 거래 순서나 정당성에 대해 **합의(consensus)** 에 도달할 방법이 없다. 블록체인은 공유 공개 원장으로 이를 해결한다.

---

## 4. 블록체인의 핵심 개념과 특성

### Blockchain의 핵심 아이디어

- 모든 거래가 **공개적으로 기록됨(publicly recorded)**
- 조작이 사실상 불가능함 (Tampering is virtually impossible)
- 코드로 통제되는 **탈중앙 디지털 화폐(decentralized digital currency)**
- **신뢰가 필요 없는 인프라(Trustless Infrastructure)**
- 정의: **Blockchain은 공개 분산 원장(public distributed ledger)**

### Key Characteristics (★시험 핵심)

| 특성 | 설명 |
|------|------|
| **Decentralization(탈중앙성)** | 중앙 권한 없이 P2P로 동작 |
| **Immutability(불변성)** | 한 번 기록된 데이터는 변경 불가 |
| **Transparency(투명성)** | 모든 거래가 참여자에게 공개됨 |
| **Consensus(합의)** | PoW, PoS 등 프로그래밍된 메커니즘을 통한 동의 |

### 탈중앙 구조의 신뢰성

- **중앙집중형**: 공격자가 은행 DB 서버 하나를 해킹하면 잔액 조작 가능
- **탈중앙형**: 공격자가 원장을 바꾸려면 **다수(또는 모든) 참여 노드를 해킹·조작** 해야 함 → 훨씬 어려운 작업

> 탈중앙 구조의 신뢰성은 "공격 비용을 천문학적으로 만든다"에서 나온다.

---

## 5. 블록체인 구조 (Block 내부 구조)

블록은 **Header**와 **Body**로 나뉜다.

```
Block Header
├── Hash of Previous Block
├── Time Stamp
├── Nonce
└── Merkle Root

Block Body
└── Transaction Data (수백~수천 개의 거래)
```

### Previous Block Hash

- 각 블록은 **이전 블록의 해시값(double SHA-256)** 을 가짐 → 블록들이 사슬처럼 연결됨
- **SHA-256(SHA-256(header))** 이중 해시 사용 → 해시 충돌을 낮추고 보안을 강화
- 이전 블록 header가 조금이라도 위조되면 → **Avalanche effect**: 입력이 조금만 바뀌어도 해시값이 완전히 달라짐 → 위조가 즉시 들통남

### Merkle Root (★중요)

- **Merkle Root**: 각 블록 body에 담긴 **거래의 무결성(integrity of transactions)** 을 보장
- **Merkle tree는 이진 트리(binary tree)** 의 일종

```
        Merkle Root
           /    \
       H(AB)   H(CD)
       /  \    /  \
    H(A) H(B) H(C) H(D)
     |    |    |    |
    TX1  TX2  TX3  TX4
```

- **Leaf(잎)**: `SHA-256(SHA-256(TX N))`
- **Parents(부모)**: `SHA-256(SHA-256(자식들의 연결(concatenation, 512 bits)))`
- **Merkle Root**: Merkle tree의 최상위 루트 값

거래 하나라도 위조되면 → **Avalanche effect** 로 인해 상위 해시들이 줄줄이 변함 → Merkle Root 값이 크게 변화한다.

구조 요약:
- **Previous Block Hash**: 블록 간 연결(체인 전체)을 보호
- **Merkle Root**: 블록 내부 거래들을 보호

### Nonce

- **Nonce (number only used once)**: **32비트 숫자**
- 채굴자(miner)가 블록의 **해시값이 특정 조건을 만족하도록 조정하는 값**

---

## 6. 채굴 (Mining)

### 거래 발생 시 전체 흐름

Alice가 Bob에게 1 BTC를 보내는 과정:

```
1. Alice가 새 거래 생성 (수수료(fee) 포함하여 Bob에게 1 BTC 전송)
2. Alice가 개인키(private key)로 서명(signs)
3. Alice가 거래를 네트워크에 브로드캐스트(broadcasts)
4. 여러 노드가 거래를 검증(validate) — 전자서명과 잔액 확인
5. 노드가 거래를 메모리 풀(mempool)에 저장
6. 채굴자가 mempool에서 수수료가 높은 거래 수백~수천 개를 선택
7. 채굴자가 블록을 생성(builds a block)
8. [PoW 수행]
9. 채굴자가 새 블록을 네트워크에 브로드캐스트
10. 다른 노드가 블록을 검증 (header hash, 유효한 TX 등)
11. 유효하면 블록체인에 연결(attached)
```

### Fork 문제

블록 생성이 너무 쉬우면:

```
Miner A: TX(120, 135, 152) 선택 → Block(A) 생성
Miner B: TX(120, 152, 163) 선택 → Block(B) 생성
→ 두 블록이 동시에 브로드캐스트 → 노드마다 다른 블록체인 보유 → "Fork"
```

**블록 생성이 너무 쉬울 때 생기는 문제:**
- 정직한 채굴자라도 계속 fork를 유발
- 악의적 채굴자가 무작위 블록을 몇 초 만에 수백만 개 생성 가능

해결 방향: **"Don't trust, verify — and make it costly to cheat"**

---

## 7. Proof of Work (PoW)

### PoW의 조건

**조건: `SHA256(SHA256(Block Header)) < Target`**

- SHA-256 출력은 256비트 (범위: 0 ~ 2²⁵⁶ − 1)
- nonce는 32비트 → 범위 0 ~ 2³² − 1 (약 42억)
- 채굴자는 **nonce를 바꿔가며 조건을 만족하는 값을 탐색**

### Target의 의미

- **Target = 채굴의 난이도(difficulty level)**

| Target | 조건 만족 난이도 | 채굴 난이도 |
|--------|----------------|-----------|
| 작은 Target | 어려움 | 어려움 |
| 큰 Target | 쉬움 | 쉬움 |

- Target은 일관된 블록 생성(비트코인: **10분/블록**)을 위해 **네트워크가 동적으로 조정(dynamically adjusted)**

### Target이 너무 낮을 때

42억 범위의 nonce를 다 써도 조건을 못 맞출 수 있다. 그러면 **블록 헤더 자체** 를 바꾼다:
- 타임스탬프(timestamp)를 약간 변경
- Merkle tree를 재배열(rearrange) → Merkle root 값이 바뀜

### PoW의 효과

- 정직한 채굴자: fork가 줄어들고 consensus가 쉬워짐
- 악성 채굴자: 무작위 블록 대량 생성이 불가능해짐

**결론: PoW는 악성 채굴자가 무작위로 블록을 생성하는 것을 방지한다.**

fork 발생 확률은 낮지만(두 명 이상의 채굴자가 동시에 조건을 만족하는 경우), 여전히 가능하다 → **Consensus** 가 필요하다.

---

## 8. Consensus (합의)

### Longest Chain Rule

fork가 둘 이상 존재하면, 블록체인은 **가장 긴 사슬(the longest chain)** 을 선택한다.

- 여기서 "가장 길다"는 것은 **최대 누적 작업량(maximum total cumulative work)** 을 의미
- 더 긴 사슬은 **살아남고(survive)**, 더 짧은 사슬은 **폐기(discarded)** 됨

### Consensus 진행 과정

```
1. Fork 발생 → 일부 노드는 chain A, 다른 노드는 chain B를 가짐
   → 새 블록은 아직 확정되지 않음(NOT confirmed)
2. 각 노드와 채굴자는 계속 작업
3. Miner C가 nonce를 찾아 새 블록 C를 생성
4. Block C는 chain A에 연결 (chain A가 chain B보다 길어짐)
5. Chain B는 폐기, block B의 TX는 mempool로 되돌아감(pushed back)
6. 6개 블록이 더 생성되면(6 confirmations) 거래가 확정(confirmed) (약 1시간 소요)
```

> **6 confirmations(약 1시간)** 는 비트코인에서 거래가 사실상 번복 불가능해지는 기준이다.

### 채굴자(Miner)의 보상

- 채굴자는 **병렬 해시 연산(parallel hash computations)** 이 많을수록 유리
- 따라서 **GPU(그리고 ASIC) 아키텍처** 가 적합 (GPU 가격 급등의 이유)
- 이 노력에 대해 **채굴 보상(mining reward)** 을 지급

---

## 9. 블록체인에 대한 위협 — 51% Attack

블록체인이 완벽한 보안을 보장하는가? → **"거의 그렇지만, 아니다(Almost yes, but NO)"**

### 공격 시나리오

Alice(공격자)가 Bob에게 100 BTC를 지불하는 TX 137이 있다고 가정한다.

```
공개 체인(Blockchain A): Block(A)에 TX 137 포함 → 정상 진행
Alice의 비공개 체인: TX 137이 없는 별도 체인을 비공개로 채굴
→ Alice는 매우 강한 해시 파워로 자신의 비공개 체인을 공개 체인보다 길게 유지
```

**진행:**

```
1. Alice의 체인은 비공개 → 공개 노드들은 chain A를 유효하다고 봄
2. 6 confirmations 후 TX 137 확정 → Bob이 돈을 인출(cashed out)
3. Alice가 자신의 체인을 공개
4. Alice의 체인이 chain A보다 길기 때문에 chain A가 교체됨
5. TX 137이 mempool로 되돌아감 → 이중지불(double-spending) 성공
```

### 51% Attack 요구 조건

- Alice가 자신의 비공개 체인을 공개 체인보다 **항상 더 길게 유지** 할 수 있어야 함
- 즉, 전체 네트워크 해시 파워의 **과반(>51%)** 을 차지해야 함 → **51% attack**

---

## 10. PoW의 한계와 PoS

### Bitcoin/PoW의 문제점

- **에너지 문제(Energy problem)**: 100~150 TWh 소비 (네덜란드 전체 에너지 사용량에 상당)
- **경제적 비효율(Economic efficiency)**: 모든 채굴자가 에너지를 소비하지만 하나만 유효 → 나머지는 에너지 낭비
- **진입 장벽(Barriers to entry)**: 막대한 연산 능력이 필요 → 소수의 기업이 해시 파워 독점 → **탈중앙성을 약화**
- **확장성·성능(Scalability)**: 블록 생성이 느리고, **TPS(transactions per second)가 제한적**

### Proof of Stake (PoS)

**PoS**: **새 블록을 제안·검증할 권리** 가 참여자가 **"스테이킹(staked)"한** (담보로 잠근) 암호화폐 양에 따라 결정되는 합의 메커니즘이다.

더 많은 코인을 잠글수록, 다음 블록을 만들 확률이 높아진다 — 에너지 집약적 채굴이 필요 없다.

원래 이더리움은 PoW 합의로 설계(2015)됐으나, 합의 알고리즘이 **PoS로 교체됨(2021)**.

### PoS 블록 생성 과정

```
1~5. PoW와 동일 (거래 생성·검증·mempool 저장)
6. 네트워크가 무작위로 다음 블록을 제안할 검증자(validator)를 선택
   (스테이크에 비례, proportional to the validator's stake)
7. 다른 검증자들이 무작위로 선택되어 블록 유효성에 투표(vote = attest)
8. 초다수(supermajority, 예: 2/3)가 동의하면 블록이 체인에 추가
9. epoch 동안 체크포인트 블록(checkpoint block)에 충분한 투표가 모이면
   → 블록이 확정(finalized) → 더 이상 번복 불가(can no longer be reverted)
10. 정직한 검증자는 스테이킹 보상(staking rewards)을 받음
```

**Finality 계산 예시**: 새 블록 53, 마지막 확정 블록 36, Epoch 36이라면 → 다음 finality check는 블록 72(36+36) 이후 → 그 후 블록 53은 번복 불가능해짐

### Validators (검증자)

- 암호화폐를 잠그고(locking up cryptocurrency) 블록 생성·검증에 참여하여, **보상(rewards)을 받는 대신 페널티(penalties)의 위험**을 짊어짐
- 이더리움은 **32 ETH** 스테이킹 필요

### 악성 검증자에 대한 대응: Slashing

악성 검증자가 할 수 있는 행위:
- 여러 블록을 동시에 생성하여 네트워크 교란
- 다수의 충돌하는 체인(forks)에 동시에 투표
- 블록 제안·검증을 제때 하지 않음

**대응책:**
- **Slashing**: 스테이크의 일부를 소각(burning parts of stakes)
- **Validator ejection**: 검증자 강제 퇴출

### PoS Variants (변형)

| 유형 | 대표 체인 | 특징 |
|------|----------|------|
| **Classic PoS** | - | 스테이크 양에 따라 확률적 선택. 자산이 적은 사용자는 거의 참여 불가 |
| **Delegated PoS (DPoS)** | Solana, EOS, Tron | 사용자가 자산을 검증자에게 위임하고 보상의 일부를 받음. 언제든 스테이크 회수 가능 |
| **Leased PoS (LPoS)** | - | 특정 기간(specific period) 동안 코인을 검증자에게 임대하여 스테이킹 파워를 높임 |
| **Liquid Staking PoS** | Lido, Rocket Pool | 스테이킹하면서도 유동성 유지 — 파생 토큰(예: stETH)을 발행해 다른 곳에서 거래·활용 가능 |

---

## 11. Smart Contract (스마트 컨트랙트)

### 개념

**Smart contract**: 미리 정해진 조건이 충족되면 자동으로 실행되는, **블록체인에 저장된 자가 실행 프로그램(self-executing program stored on a blockchain)**.

제3자의 강제 없이도 동작하는 **디지털 합의(digital agreement)** — **코드 그 자체가 계약(the code itself is the contract)**.

### 중고나라 딜레마 → 에스크로 → 스마트 컨트랙트

**Escrow(에스크로)**: 구매자와 판매자가 각자의 의무를 이행할 때까지 결제를 보관하는 신뢰할 수 있는 제3자 서비스.

```
Escrow 흐름:
Alice → Escrow에 200만 원(+수수료) 입금 → Bob → Alice에게 맥북 전달
→ Escrow → Bob에게 200만 원 전달
```

스마트 컨트랙트가 Escrow를 대체하는 방식:

```
Alice → Smart Contract에 20 coin B 전송
Bob   → Smart Contract에 10 coin A 전송
조건 충족 시 → Smart Contract가 자동으로
              Alice에게 10 coin A 전달
              Bob에게 20 coin B 전달
→ 신뢰 제3자(Escrow) 없이 코드가 처리
```

### Smart Contracts의 특성

- **Automation(자동화)**: 중개자(intermediaries) 불필요 → 비용·시간 절감
- **Transparency(투명성)**: 누구나 on-chain에서 코드와 로직 검증 가능
- **Immutability(불변성)**: 한 번 배포되면 코드를 변조할 수 없음
- **Composable(조합 가능성)**: 다른 컨트랙트와 상호작용 가능 (DeFi에 적합)

### Use Cases

- **DeFi**: Lending, Swapping, Staking (예: Aave, Uniswap)
- **Gaming**: NFT 기반 자산, P2E(Play-to-Earn) 보상 (예: OpenSea)
- **DAOs (Decentralized Autonomous Organizations)**: 투표 및 거버넌스 자동화 (예: Aave Governance)
- **Staking**: 검증자를 위한 이더리움 예치 컨트랙트(deposit contracts)

---

## 핵심 한눈 정리 (시험 대비)

| 개념 | 핵심 키워드 | 한 줄 요약 |
|------|-------------|------------|
| Centralized 단점 | SPOF, censorship | 한 곳 뚫리면 전체 마비, 신뢰 필요 |
| Double-spending | easily copyable | 디지털 데이터 복제로 두 번 사용 |
| Blockchain 4특성 | Decentralization, Immutability, Transparency, Consensus | 시험 단골 |
| Previous Block Hash | double SHA-256, Avalanche effect | 블록 간 연결 보호 |
| Merkle Root | binary tree, integrity | 블록 내부 거래 보호 |
| Nonce | 32-bit, number only used once | 해시 조건 맞추는 조정 값 |
| PoW 조건 | SHA256(SHA256(Header)) < Target | Target 작을수록 어려움 |
| Target | difficulty, 10 mins/block | 네트워크가 동적 조정 |
| Consensus | Longest Chain Rule, 6 confirmations | 누적 작업량 최대 체인 선택 |
| 51% Attack | hash power >51%, double-spending | 비공개 체인을 더 길게 유지 |
| PoW 단점 | energy, barriers, low TPS | 에너지 낭비·중앙화·느림 |
| PoS | staked, validator, finalized | 스테이크 비례 선택, 에너지 불필요 |
| PoS 위협 대응 | Slashing, validator ejection | 스테이크 소각·퇴출 |
| PoS Variants | DPoS, LPoS, Liquid Staking | 위임/임대/유동성 스테이킹 |
| Smart Contract | self-executing, code is the contract | escrow를 코드가 대체 |
