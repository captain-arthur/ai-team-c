# Architecture: Operational Confidence Framework

**Role:** architect
**Date:** 2026-03-17

---

## 요구사항 요약

### 기능 요구사항

| # | 요구사항 |
|---|----------|
| FR1 | "지금 안전한가?"에 명확히 답할 수 있어야 함 |
| FR2 | "내일도 안전한가?"에 명확히 답할 수 있어야 함 |
| FR3 | "무엇을 해야 하는가?"에 구체적 행동을 제시할 수 있어야 함 |
| FR4 | 신호 간의 인과관계가 명확해야 함 |
| FR5 | 5분 안에 설명할 수 있어야 함 |

### 비기능 요구사항

| # | 요구사항 |
|---|----------|
| NFR1 | 최소성 — 필수 신호가 10개 이하 |
| NFR2 | 기술 중립 — 특정 도구에 종속되지 않는 개념 모델 |
| NFR3 | 확장 가능 — 새로운 신호를 체계적으로 추가 가능 |

### 제약 조건

| # | 제약 |
|---|------|
| C1 | ai-team-lab의 올바른 아이디어를 존중 |
| C2 | Managed Kubernetes에서 control plane metric이 없을 수 있음 |

---

## 설계 옵션

### Option A: 기존 Block 모델 유지

**설명:**
ai-team-lab의 Block1/Block2/Block3 구조를 유지하고, 설명만 개선.

**장점:**
- 기존 작업과의 연속성
- 이미 검증된 구조

**단점:**
- 본질적인 문제(복잡성, 인과관계 불명확)가 해결되지 않음
- Block 간의 관계가 여전히 직관적이지 않음

### Option B: 체인 기반 계층 모델 (Chain-Based Hierarchy)

**설명:**
**고장 전파 체인**을 중심으로 설계. 신호를 체인에서의 위치로 분류하고, 각 단계가 하나의 운영 질문에 답함.

```
[Pressure] → [Strain] → [Instability] → [Impact]
```

**장점:**
- **직관적**: 인과관계가 명확히 보임
- **단순함**: 하나의 체인으로 모든 것을 설명
- **행동 연결**: 각 단계가 자연스럽게 행동으로 연결됨

**단점:**
- 기존 Block 모델과의 매핑이 필요
- 새로운 개념 학습 필요

---

## 트레이드오프 분석

| 기준 | Option A (Block 유지) | Option B (체인 기반) |
|------|----------------------|---------------------|
| 단순성 | 중 | **상** |
| 인과관계 명확성 | 하 | **상** |
| 설명 용이성 | 중 | **상** |
| 기존 작업과의 연속성 | **상** | 중 |
| 행동 연결성 | 중 | **상** |

---

## 결정

**선택:** Option B — 체인 기반 계층 모델

**이유:**

1. **단순성**: "하나의 체인"이라는 핵심 아이디어로 5분 안에 설명 가능
2. **인과관계**: 신호 간의 관계가 체인에서의 위치로 자연스럽게 설명됨
3. **행동 연결**: 체인의 위치가 곧 행동의 종류를 결정함

**수용하는 트레이드오프:**

- 기존 Block 모델과의 **개념적 불연속** — 그러나 Block 모델의 핵심 아이디어는 체인 모델에 흡수됨

---

## 상세 설계

### 1. 핵심 개념: 고장 전파 체인 (Failure Propagation Chain)

#### 1.1 핵심 아이디어

> **시스템 문제는 한 번에 발생하지 않는다. 단계적으로 전파된다.**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    [Pressure]  ────→  [Strain]  ────→  [Instability]  ────→  [Impact]      │
│                                                                             │
│     시스템에          첫 번째           실제 실패가         사용자/서비스    │
│     스트레스가        증상이            발생하기            영향이           │
│     가해짐            나타남            시작함              발생함           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### 1.2 각 단계의 정의

| 단계 | 영문 | 정의 | 특성 |
|------|------|------|------|
| **1. Pressure** | Resource Pressure | 시스템 리소스에 스트레스가 가해지는 상태 | 가장 선행. Lead time이 가장 김 |
| **2. Strain** | Operational Strain | 스트레스로 인해 시스템 동작에 어려움이 생기는 상태 | 조기 경보 단계 |
| **3. Instability** | Workload Instability | 실제 실패가 발생하기 시작하는 상태 | 현재 안전성 판단 기준 |
| **4. Impact** | Service Impact | 사용자나 서비스에 영향이 발생하는 상태 | 가장 후행. 진단 및 복구 |

#### 1.2.1 Strain 단계의 의미 (시스템 운영 마찰)

**Strain**은 단순히 "Pending이 늘어난다"가 아니라, **분산 시스템의 운영 마찰(operational friction)**을 나타낸다.

- **정의**: Pressure로 인해 리소스 여유가 줄어들었을 때, 시스템이 **아직 실패하지는 않았지만 정상 동작에 어려움을 겪기 시작하는** 구간이다. "실패가 발생하기 직전의 마찰"이다.
- **분산 시스템에서의 의미**: 노드/컨트롤 플레인은 동작하지만, 스케줄링·조정·동기화 등 **일상 연산이 지연되거나 백로그가 쌓이는** 상태. 사용자 요청에는 아직 직접 영향이 없을 수 있으나, **시간이 지나면 Instability로 이어질 수 있는** 단계이다.
- **이 단계에 속하는 신호의 특성**:
  - **추세 또는 지연**을 다룸 (예: Pending **증가**, 스케줄링 **지연**) — "이미 실패한 개수"가 아니라 "나쁜 방향으로 움직이고 있음"을 나타냄.
  - **인프라/컨트롤 플레인 동작**에 관한 것. 애플리케이션 에러나 사용자 체감 품질은 아직 Impact 단계의 영역.
- **Pressure와 구분되는 이유**: Pressure는 "리소스 사용률이 높다"는 **상태**만 본다. Strain은 "그 결과로 **시스템 동작(스케줄링, 조정 등)이 힘들어지고 있다**"는 **동작 수준의 마찰**을 본다. 같은 CPU 80%라도 스케줄링이 원활하면 Strain은 없고, Pending이 늘어나기 시작하면 Strain이 있다.
- **Instability와 구분되는 이유**: Instability는 **이미 발생한 실패** (재시작, NotReady, Pending이 임계값 초과 등)를 다룬다. Strain은 **아직 실패로 기록되지 않은, 실패로 가는 경로 위의 마찰**이다. 따라서 Strain을 별도 단계로 두면 "내일 안전한가?"에 답할 때 **실패가 나타나기 전에** 사전 대응할 수 있다.

#### 1.2.2 Impact 단계의 의미 (사용자 가시 영향)

**Impact**은 **사용자가 체감하는 서비스 수준의 영향**을 나타내는 체인의 최종 단계이다.

- **정의**: 인프라·워크로드 이상이 **요청 가용성, 지연, 에러율** 등 **사용자 관점의 서비스 품질**에 반영된 상태. "클러스터 내부 문제"가 "서비스 수준 결과"로 전환된 구간이다.
- **SLO와의 관계**: Impact 단계의 신호는 **SLO(Service Level Objective) 위반 또는 그 직전**과 맞닿아 있다. M1(Empty Endpoints)은 가용성 SLO 위반, M2(Service Degradation)는 지연/에러율 SLO 위반에 해당한다. 따라서 Impact는 "인프라 모니터링"이 아니라 **서비스 수준의 결과**를 보는 단계이다.
- **체인의 최종 단계인 이유**: Pressure → Strain → Instability는 모두 **인프라·워크로드 관점**의 신호이다. Impact는 **그 결과가 최종 사용자에게 어떻게 도달했는가**를 다룬다. 여기서 이상이 없으면 "지금 안전하다"고 판단할 수 있고, 여기서 이상이 있으면 즉시 복구가 필요하다.
- **인프라 신호에서 서비스 신호로의 전환**: Instability까지는 노드/파드/스케줄러 등 **클러스터 리소스**의 상태다. Impact는 **엔드포인트 가용성, 요청 성공률, 지연**처럼 **트래픽이 서비스를 통과한 결과**를 본다. 따라서 M1은 kube-state-metrics 기반이지만 "서비스에 백엔드가 붙어 있는가?"라는 **서비스 관점**이고, M2는 Ingress/메시 등 **트래픽 관측점**의 지표다.

#### 1.3 단계 간의 관계

```
Pressure       "리소스에 압박이 있다"
    ↓          시간이 지나면...
Strain         "시스템이 힘들어한다"
    ↓          방치하면...
Instability    "실패가 발생하기 시작한다"
    ↓          계속되면...
Impact         "서비스가 영향받는다"
```

**핵심 통찰:**
- **체인의 앞쪽**을 보면 **미래를 예측**할 수 있다
- **체인의 뒤쪽**을 보면 **현재를 진단**할 수 있다
- **어느 단계에 이상이 있는지**에 따라 **행동이 결정**된다

---

### 2. Kubernetes 클러스터에 적용

#### 2.1 단계별 현상

| 단계 | Kubernetes에서의 현상 |
|------|----------------------|
| **Pressure** | 노드 CPU/메모리/디스크 사용률 상승, 요청량 증가 |
| **Strain** | 스케줄러가 파드를 배치하기 어려움, API server 지연 |
| **Instability** | 파드 재시작 증가, NotReady 노드, OOM kill |
| **Impact** | Endpoints 감소, 서비스 응답 지연, 가용성 저하 |

#### 2.2 최소 신호 집합 (Minimal Signal Set)

각 단계에 **최소 1개, 최대 3개**의 대표 신호를 선정:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Stage 1: Pressure (압박)                                                    │
│  ─────────────────────────────────────────────────────────────────────────  │
│  P1. Node CPU High        노드 CPU 사용률 > 80%                              │
│  P2. Node Memory High     노드 메모리 사용률 > 80%                            │
│  P3. Node Disk Low        노드 디스크 여유 < 10%                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  Stage 2: Strain (부담)                                                      │
│  ─────────────────────────────────────────────────────────────────────────  │
│  S1. Pending Pods Rising  Pending 파드 수가 증가 중                          │
│  S2. Scheduling Delay     스케줄러 지연 (control plane 있을 때)               │
├─────────────────────────────────────────────────────────────────────────────┤
│  Stage 3: Instability (불안정)                                               │
│  ─────────────────────────────────────────────────────────────────────────  │
│  I1. Excessive Restarts   파드 재시작 급증                                    │
│  I2. NotReady Nodes       NotReady 상태 노드 존재                            │
│  I3. Pending Pods High    Pending 파드 수가 임계값 초과                       │
├─────────────────────────────────────────────────────────────────────────────┤
│  Stage 4: Impact (영향)                                                      │
│  ─────────────────────────────────────────────────────────────────────────  │
│  M1. Empty Endpoints      핵심 서비스 Endpoints 비어 있음                     │
│  M2. Service Degradation  서비스 응답 지연 또는 에러율 상승                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

**총 10개 신호** — 최소성 요구사항 충족.

#### 2.3 최소 신호 선정 원칙 (Minimal Signal Selection Principle)

이 10개 신호는 아래 **선정 기준**을 만족하도록 선택되었다. 새 신호를 추가할 때도 동일 기준을 적용한다.

| 기준 | 설명 | 적용 예 |
|------|------|---------|
| **Stage 대표성 (representative of the stage)** | 해당 단계의 **핵심 현상**을 나타내야 함. Pressure는 리소스 압박, Strain은 운영 마찰, Instability는 실패 발생, Impact는 사용자 영향. | P1–P3는 노드 CPU/메모리/디스크라는 리소스 압박의 대표; S1–S2는 스케줄링 마찰의 대표. |
| **직접 행동 가능 (directly actionable)** | 이상 시 **무엇을 할지**가 Runbook 수준으로 구체적이어야 함. "조사 필요"만으로 끝나면 부족. | 각 신호별로 "리소스 확인 → 조정/확장/롤백" 등 구체 행동이 정의됨. |
| **최소 중복 (minimal overlap)** | 다른 신호와 **같은 현상을 다른 각도로만** 보는 중복을 피함. 하나의 판단에 여러 신호가 동시에 필요하면 최소한으로 유지. | S1(추세)과 I3(절대값)은 둘 다 Pending이지만 "증가 중인가?" vs "이미 많은가?"로 역할이 구분됨. |
| **대부분의 Kubernetes 환경에서 관측 가능 (observable in most K8s environments)** | node_exporter, kube-state-metrics만 있어도 **핵심 판단**이 가능해야 함. control plane/Ingress 메트릭은 선택 보강. | S2, M2는 Managed K8s 또는 Ingress 미사용 환경에서는 생략 가능하도록 설계됨. |

**왜 이 10개인가:**  
Pressure 단계는 노드 리소스(CPU, 메모리, 디스크) 세 가지가 **서로 대체 불가**라 각각 필요하다. Strain은 "스케줄링 마찰" 하나의 현상을 **추세(S1)**와 **지연(S2)** 두 관점으로 보며, S2는 환경에 따라 생략 가능하다. Instability는 **워크로드 실패**(재시작), **노드 실패**(NotReady), **스케줄 실패 지속**(Pending High) 세 가지 형태를 구분한다. Impact는 **가용성(M1)**과 **품질(M2)** 두 가지 사용자 관점을 다룬다. 이 조합으로 "지금/내일 안전한가?", "무엇을 해야 하나?"에 **최소한의 신호만으로** 답할 수 있도록 했다.

---

### 3. 운영 질문과의 매핑

#### 3.1 세 가지 질문에 대한 답

| 질문 | 답하는 방법 | 참조 단계 |
|------|-------------|-----------|
| **"지금 안전한가?"** | Stage 3 (Instability)가 **모두 정상**인가? | I1, I2, I3 |
| **"내일도 안전한가?"** | Stage 1, 2 (Pressure, Strain)에 **경고가 없는가?** | P1-P3, S1-S2 |
| **"무엇을 해야 하는가?"** | **어느 Stage에 이상이 있는가?**로 결정 | 전체 |

#### 3.2 판단 규칙

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          Operational Confidence Rules                         │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Q1: "지금 클러스터가 안전한가?"                                              │
│  ──────────────────────────────                                              │
│  IF Stage 3 (Instability) 모든 신호 정상                                     │
│     AND Stage 4 (Impact) 모든 신호 정상                                      │
│  THEN → "예, 지금 안전합니다"                                                 │
│  ELSE → "아니오, 조사가 필요합니다"                                           │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Q2: "내일도 안전할 것인가?"                                                  │
│  ──────────────────────────                                                  │
│  IF Stage 1 (Pressure) 모든 신호 정상                                        │
│     AND Stage 2 (Strain) 모든 신호 정상                                      │
│  THEN → "예, 당분간 안전할 것입니다"                                          │
│  ELSE → "아니오, 조기 리스크가 있습니다"                                       │
│                                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Q3: "무엇을 해야 하는가?"                                                    │
│  ────────────────────────                                                    │
│  이상이 있는 Stage에 따라 행동이 결정됨 (아래 행동 매트릭스 참조)               │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### 4. 행동 매트릭스 (Action Matrix)

#### 4.1 Stage별 행동

| Stage | 이상 시 의미 | 권장 행동 | 긴급도 |
|-------|-------------|-----------|--------|
| **Pressure** | 곧 문제가 발생할 수 있음 | **용량 계획**: 노드 추가, 워크로드 분산, 정리 | 낮음 (시간 여유) |
| **Strain** | 문제가 임박함 | **사전 대응**: 리소스 확장, 우선순위 조정, 스케일 조정 | 중간 |
| **Instability** | 문제가 진행 중 | **원인 조사**: 로그 확인, 이벤트 분석, 롤백 검토 | 높음 |
| **Impact** | 이미 영향 발생 | **즉시 복구**: 긴급 대응, 롤백, 트래픽 우회 | 매우 높음 |

#### 4.2 신호별 구체적 행동

```
┌────────────────────┬────────────────────────────────────────────────────────┐
│  신호              │  구체적 행동                                            │
├────────────────────┼────────────────────────────────────────────────────────┤
│  P1. Node CPU High │  • 워크로드 분산 검토                                    │
│                    │  • 리소스 request/limit 조정                            │
│                    │  • 노드 추가 계획                                       │
├────────────────────┼────────────────────────────────────────────────────────┤
│  P2. Node Mem High │  • 메모리 누수 워크로드 식별                             │
│                    │  • 메모리 limit 조정                                    │
│                    │  • 노드 추가 계획                                       │
├────────────────────┼────────────────────────────────────────────────────────┤
│  P3. Node Disk Low │  • 로그/이미지 정리                                     │
│                    │  • 디스크 확장                                          │
│                    │  • 로그 로테이션 확인                                    │
├────────────────────┼────────────────────────────────────────────────────────┤
│  S1. Pending Rising│  • 리소스 여유 확인                                     │
│                    │  • 낮은 우선순위 파드 정리                               │
│                    │  • 스케일 조정                                          │
├────────────────────┼────────────────────────────────────────────────────────┤
│  I1. Restarts High │  • 재시작 TOP 워크로드 확인                              │
│                    │  • 로그에서 원인 분석 (OOM, CrashLoop 등)                │
│                    │  • 필요시 롤백                                          │
├────────────────────┼────────────────────────────────────────────────────────┤
│  I2. NotReady Node │  • 노드 상태 확인 (kubectl describe)                    │
│                    │  • kubelet 상태 확인                                    │
│                    │  • 노드 drain/replace 검토                              │
├────────────────────┼────────────────────────────────────────────────────────┤
│  I3. Pending High  │  • Pending 원인 분석 (리소스 부족? 노드 선택?)           │
│                    │  • 즉시 리소스 확장                                     │
│                    │  • 우선순위 높은 파드 먼저 처리                          │
├────────────────────┼────────────────────────────────────────────────────────┤
│  M1. Empty Endpoint│  • 해당 서비스의 파드 상태 즉시 확인                      │
│                    │  • 긴급 스케일업 또는 롤백                               │
│                    │  • 트래픽 우회 검토                                     │
└────────────────────┴────────────────────────────────────────────────────────┘
```

---

### 5. 대시보드 매핑

#### 5.1 기존 Block 모델과의 관계

| 기존 모델 | 체인 기반 모델 | 관계 |
|-----------|---------------|------|
| Block 1 (Operational Confidence) | Stage 3 + Stage 4 | "지금 안전한가?" |
| Block 2 (Early Risk) | Stage 1 + Stage 2 | "내일 안전한가?" |
| Block 3 (Investigation) | 각 Stage의 상세 드릴다운 | "어디를 조사할까?" |

#### 5.2 대시보드 레이아웃 제안

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Cluster Operational Confidence Dashboard                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  🟢 SAFE NOW                          🟡 RISK AHEAD                   │  │
│  │  ─────────────────────                ────────────────                 │  │
│  │  Stage 3-4: All Clear                 Stage 1-2: 1 Warning            │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  Failure Propagation Chain                                              ││
│  │                                                                         ││
│  │  [Pressure]     [Strain]      [Instability]    [Impact]                ││
│  │   P1: 🟢         S1: 🟡        I1: 🟢           M1: 🟢                  ││
│  │   P2: 🟢         S2: 🟢        I2: 🟢           M2: 🟢                  ││
│  │   P3: 🟢                       I3: 🟢                                   ││
│  │                                                                         ││
│  │  ────────────────────────────────────────────────────────────────────  ││
│  │  Stage 2 Warning: S1 (Pending Pods Rising)                             ││
│  │  Action: 리소스 여유 확인, 스케일 조정 검토                              ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
│  [▼ Investigation: Click to expand Stage details]                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 6. 프레임워크 요약: 한 장 정리

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│              OPERATIONAL CONFIDENCE FRAMEWORK                               │
│              for Kubernetes Clusters                                        │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  핵심 아이디어:                                                              │
│  ────────────                                                               │
│  "시스템 문제는 단계적으로 전파된다.                                          │
│   체인의 앞쪽을 보면 예측하고, 뒤쪽을 보면 진단한다."                          │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  고장 전파 체인:                                                             │
│  ──────────────                                                             │
│                                                                             │
│    [Pressure]  ───→  [Strain]  ───→  [Instability]  ───→  [Impact]         │
│       압박            부담              불안정             영향              │
│                                                                             │
│    예측 ◄───────────────────────────────────────────────────► 진단          │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  세 가지 질문:                                                               │
│  ─────────────                                                              │
│                                                                             │
│    Q1: "지금 안전한가?"      →  Stage 3-4 정상 여부로 판단                    │
│    Q2: "내일도 안전한가?"    →  Stage 1-2 경고 여부로 판단                    │
│    Q3: "무엇을 해야 하나?"   →  이상 Stage에 따라 행동 결정                    │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  최소 신호 집합 (10개):                                                      │
│  ─────────────────────                                                      │
│                                                                             │
│    Pressure:    P1 CPU, P2 Memory, P3 Disk                                 │
│    Strain:      S1 Pending Rising, S2 Scheduling Delay                     │
│    Instability: I1 Restarts, I2 NotReady, I3 Pending High                  │
│    Impact:      M1 Empty Endpoints, M2 Service Degradation                 │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  행동 원칙:                                                                  │
│  ──────────                                                                 │
│                                                                             │
│    Pressure 경고    →  용량 계획 (시간 여유 있음)                             │
│    Strain 경고      →  사전 대응 (곧 문제 발생)                               │
│    Instability 경고 →  원인 조사 (문제 진행 중)                               │
│    Impact 경고      →  즉시 복구 (이미 영향 발생)                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Operational Confidence Framework v2: 운영 계층 확장

v1의 체인(Pressure → Strain → Instability → Impact)은 **개념적으로 올바르지만**, 실제 Kubernetes 플랫폼에는 노드·파드 외에도 **트래픽, 게이트웨이, 멀티클러스터** 등 여러 운영 계층이 있다. v2는 이 체인을 **운영 계층(Operational Signal Topology)**에 매핑하고, **계층 간 전파 경로**와 **레이어별 최소 대표 신호**를 정의하여, 그대로 회의·블로그·논문에서 설명 가능하면서도 **실제 운영에 즉시 적용**할 수 있게 한다. v1의 최소성·행동 연결·5분 설명 원칙은 유지한다.

### 7.1 Operational Signal Topology (운영 신호 위상)

실제 Kubernetes 플랫폼에서 신호가 나오는 **시스템 계층**을 아래와 같이 정의한다. 각 계층은 한 가지 관점의 "어디를 보는가"를 나타내며, 고장 전파는 **아래 계층 → 위 계층**으로 전달되는 경우가 많다.

| 계층 (Layer) | 관찰 대상 | v1 체인 단계와의 대응 |
|--------------|-----------|------------------------|
| **Infrastructure Layer** | 노드 리소스(CPU, 메모리, 디스크), 네트워크 용량 | Pressure |
| **Cluster Control Layer** | Scheduler, controller, API server 부하·지연 | Strain (제어 동작 마찰) |
| **Workload Layer** | Pod, Deployment, 재시작 패턴, 리소스 단편화, 스케일링 | Strain(추세) + Instability |
| **Traffic Layer** | 서비스 트래픽, Ingress/Gateway 동작, 지연·백로그 | Instability 말기 ~ Impact 직전 |
| **Service Reliability Layer** | SLO 위반, Endpoint 가용성, 에러율 | Impact |
| **Platform Topology Layer** | 멀티클러스터/환경 간 설정·버전 drift, 중요 리소스 불일치 | 모든 단계에 **주입 가능** (교차 계층) |

**계층 간 상호작용:**

- **전파 방향 (일반적)**: Infrastructure 압박 → Control에서 스케줄링·API 지연(Strain) → Workload에서 실패·Pending(Instability) → Traffic에서 지연·에러 증가 → Service Reliability에서 SLO 위반(Impact). 즉 **아래에서 위로** 전파된다.
- **Platform Topology**는 **수직이 아닌 수평** 관점이다. 여러 클러스터 또는 여러 환경 간 **설정·버전·리소스 정의**가 어긋나면, 특정 계층(예: Workload의 limit, Traffic의 라우팅)에 갑자기 Strain/Instability를 만들 수 있다. 따라서 "어느 한 계층의 문제"라기보다 **플랫폼 전반의 일관성**을 보는 계층이다.
- **트래픽 계층의 위치**: Traffic Layer는 "요청이 게이트웨이·메시를 통과하는 구간"이다. 워크로드는 살아 있지만 **라우팅·대기열·지연**에서 이상이 나면 여기서 먼저 보인다. Impact(Service Reliability)는 "그 결과가 사용자·SLO에 도달했는가"이므로, Traffic은 **Impact 직전**의 관측점이다.

### 7.2 계층 간 고장 전파 (Chain Across Layers)

개념 체인을 **계층별로 구체화**하면 다음과 같다.

```
Resource pressure (Infrastructure)
    → Scheduling / API strain (Cluster Control)
    → Workload instability (Workload)
    → Traffic degradation (Traffic)
    → Service impact (Service Reliability)
```

**전파 경로 예시:**

| 경로 | Infrastructure | Control | Workload | Traffic | Service Reliability |
|------|----------------|---------|----------|---------|----------------------|
| **경로 A** | Node CPU/Memory high | Pending rising, Scheduler delay | Restarts, Pending high | Gateway latency up, 5xx 증가 | Endpoint 비어 있음, SLO 위반 |
| **경로 B** | (정상) | API server 지연 (과다 요청) | 새 파드 스케줄 지연 | 요청 타임아웃 증가 | 에러율 상승 |
| **경로 C** | Node disk low | 이미지 풀/로그 쓰기 지연 | Eviction, Restart | 일시적 5xx | 가용성 저하 |

**Platform Topology 주입 예:** 클러스터 A와 B에서 동일 서비스의 `resource.limits`가 다르게 적용되어, A에서는 Pressure 없이 B에서만 OOM·Restart가 발생 → Workload Instability가 **설정 drift**로 인해 특정 환경에서만 나타남. 이때 계층별로 보면 Infrastructure는 정상이지만 Workload에서 불안정이 나오므로, **설정 일관성(Platform Topology)** 점검이 필요하다.

### 7.3 확장 신호 모델 (레이어별 대표 신호)

v1의 10개 신호는 **그대로 핵심**이다. v2에서는 이들을 **어느 계층에서 보는가**로 매핑하고, **레이어당 최소한의 대표 신호**만 추가하여 신호 수를 폭증시키지 않는다.

**v1 신호의 계층 매핑:**

| Layer | v1 신호 | 비고 |
|-------|---------|------|
| Infrastructure | P1, P2, P3 | Node CPU/Memory/Disk |
| Cluster Control | S1, S2 | Pending rising, Scheduling delay |
| Workload | I1, I2, I3 | Restarts, NotReady, Pending high |
| Traffic | — | v1에는 명시적 신호 없음; M2가 일부 포괄 |
| Service Reliability | M1, M2 | Empty Endpoints, Service degradation |
| Platform Topology | — | v1에는 없음 |

**v2에서 추가하는 대표 신호 (레이어당 최대 1개, 선택 적용):**

| Layer | 추가 신호 ID | 이름 | 의미 | 적용 조건 |
|-------|--------------|------|------|------------|
| **Traffic** | T1 | Gateway/Ingress health | Ingress 또는 Gateway의 에러율·지연 급증 | Ingress/Gateway 사용 시. 없으면 M2만으로 판단. |
| **Platform Topology** | PL1 | Config/version drift | 멀티클러스터 또는 Git vs 클러스터 간 중요 리소스·버전 불일치 | 멀티클러스터 또는 다환경 운영 시. |

**Workload Layer 보강 (선택):** 파드 수준 리소스·스케일링을 이미 I1(재시작)으로 간접 보는 경우가 많다. **W1 Pod resource pressure** (예: 파드 메모리/CPU 사용률이 request 대비 지속적으로 높은 비율)를 두면 "노드는 여유 있는데 특정 워크로드만 압박"인 경우를 구분할 수 있으나, v2에서는 **최소성**을 위해 **선택 신호**로만 두고, 기본 판단은 v1 10개로 유지한다.

**원칙 유지:**  
- 여전히 **최소 신호 집합**을 지킨다. 필수는 v1 10개; T1, PL1, W1은 **환경에 따라 추가**하는 선택이다.  
- 각 추가 신호는 **직접 행동 가능**해야 하며(예: T1 이상 → Gateway/업스트림 점검, PL1 이상 → drift 정리·동기화), **Stage 대표성**과 **중복 최소**를 만족해야 한다.

### 7.4 실제 운영 사용 (Real Operational Usage)

**일상 클러스터 헬스 체크 (Daily cluster health check)**  
1. **지금 안전한가?** → Instability + Impact (v1 기준): I1, I2, I3, M1, M2 확인.  
2. **내일도 안전한가?** → Pressure + Strain: P1–P3, S1–S2 확인.  
3. **레이어 뷰 (v2)**: 같은 신호를 Infrastructure → Control → Workload → Traffic → Service Reliability 순으로 훑어 "어느 계층에서 먼저 경고가 나오는가"를 확인.  
4. 멀티클러스터/다환경이면 **Platform Topology**: PL1(drift) 주기적 확인.  
→ 5분 이내에 "안전 / 조기 리스크 / 조사 필요"를 결론 내고, 필요 시 해당 계층 Runbook으로 드릴다운.

**인시던트 조기 감지 (Incident early detection)**  
- **Strain 계층**을 먼저 본다: S1(Pending rising), S2(Scheduling delay). 여기서 경고가 나오면 아직 Instability/Impact가 없어도 "곧 문제 가능"으로 보고 사전 대응(리소스 확장, 우선순위 조정).  
- **Traffic 계층** 신호(T1 또는 M2의 지연/에러 추이)가 나쁜 방향으로 움직이면, Instability(재시작 등)가 없어도 "트래픽 경로 문제"를 의심하고 Gateway/메시/업스트림을 점검.  
→ 체인의 **앞쪽·중간 계층**을 보면 Impact 전에 조기 대응할 수 있다.

**용량 계획 (Capacity planning)**  
- **Pressure + Strain** 추이를 계층별로 본다: Infrastructure(P1–P3)가 꾸준히 상승하거나, Control(S1–S2)에서 Pending·지연이 늘어나면 "곧 리소스 또는 스케줄링 한계"로 해석.  
- 레이어별로 "어느 계층이 먼저 한계에 다다르는가"를 비교하면, 노드 증설만으로 해결되는지, 스케줄러/API 튜닝이 필요한지, 워크로드 스케일/리소스 조정이 필요한지 구분할 수 있다.

**멀티클러스터/설정 drift 감지 (Multi-cluster drift detection)**  
- **Platform Topology** 계층: PL1을 주기적으로 실행하여, 클러스터 간 또는 Git(선언) vs 클러스터(실제) 간 **중요 리소스(예: LimitRange, HPA 정책, Ingress 설정)** 불일치를 확인.  
- drift가 있으면 특정 클러스터에서만 Strain/Instability가 발생할 수 있으므로, "동일 서비스인데 한 클러스터만 이상"일 때 **설정 일관성**을 먼저 점검한다.

이렇게 v2는 **v1 체인을 그대로 두고**, 운영 계층(위상), 계층 간 전파, 레이어별 최소 대표 신호, 그리고 일상 점검·조기 감지·용량 계획·drift 감지라는 **실제 사용 패턴**을 명시하여, 이론과 실제 Kubernetes 운영을 연결한다.

---

## 8. Operational Confidence Framework v3: 개념 기반 강화 및 업계 프레임워크화

v3는 v1·v2를 **컨퍼런스·블로그·연구 수준**의 업계 프레임워크로 끌어올린다. 기존 모니터링·Observability와의 **위치 관계**, **실제 인시던트 전파 사례**, **참조 대시보드 모델**, **운영자 멘탈 모델**을 명시하여, "왜 이 프레임워크가 가치 있는가"를 설명 가능하게 한다. 5분 설명 원칙과 최소 신호 원칙은 유지한다.

### 8.1 기존 모니터링 모델 대비 프레임워크 위치 (Framework Positioning)

**Monitoring / Observability / Operational Confidence**는 목표가 다르다. 이 프레임워크는 세 번째에 위치한다.

| 관점 | 핵심 목표 | 질문 예시 | 한계(이 프레임워크가 채우는 것) |
|------|-----------|-----------|--------------------------------|
| **Traditional Monitoring** | 메트릭·로그·이벤트 **수집** | "어떤 지표가 있나?" | 수집만으로는 **판단 규칙**이 없음. "그래서 괜찮은가?"에 답하지 못함. |
| **Observability** | 시스템 동작 **설명** (explain behavior) | "왜 이런 일이 일어났나?" | 설명은 **사후**에 유용. **사전 예측·의사결정**을 위한 구조는 아님. |
| **Operational Confidence** | **운영 의사결정** 가능하게 함 | "지금 안전한가?", "내일도 안전한가?", "무엇을 해야 하나?" | 최소 신호 + 체인 + 행동 매핑으로 **즉시 결론과 행동**을 낼 수 있음. |

**개념적 비교 (한 문장씩):**

```
Monitoring        →  collecting metrics
Observability     →  explaining system behavior
Operational Confidence  →  enabling operational decisions
```

**이 프레임워크의 고유 가치:**  
- **판단 규칙 내장**: 메트릭 나열이 아니라 "이 조건이면 안전, 저 조건이면 조사/대응"이라는 **규칙**이 체인·Stage에 붙어 있음.  
- **시간 축 명시**: 앞쪽 Stage(Pressure, Strain)로 **예측**, 뒤쪽 Stage(Instability, Impact)로 **진단**. Observability는 "왜?"에 초점이 있지만, "언제 대응할지"는 이 프레임워크가 명시함.  
- **행동 연결**: 각 Stage·신호마다 **무엇을 할지**가 Runbook 수준으로 연결됨. "설명"에서 멈추지 않고 **행동**까지 이어짐.

따라서 Monitoring은 **입력**, Observability는 **심층 분석**, Operational Confidence는 **운영 결론과 행동**을 담당한다. 세 가지는 상호 배타적이 아니라 **역할이 다른 레이어**이며, 이 프레임워크는 그중 "운영 확신" 레이어를 채운다.

### 8.2 인시던트 전파 사례 (Incident Propagation Examples)

체인 모델이 **실제 Kubernetes 장애 시나리오**에서 어떻게 동작하는지, 조기 감지에 어떻게 기여하는지 보여 주는 사례다.

**사례 1: 노드 메모리 압박 → 스케줄링 지연 → 파드 재시작 루프 → 게이트웨이 지연 → SLO 위반**

| 단계 | Stage | 관찰 신호 | 운영자가 볼 수 있는 시점 | 조기 대응 가능 시점 |
|------|-------|-----------|---------------------------|----------------------|
| 1 | Pressure | P2 Node Memory High (노드 메모리 >80%) | 트래픽 영향 전 수십 분 | **여기서 대응 시** 노드 추가·워크로드 분산으로 Impact 예방 |
| 2 | Strain | S1 Pending Pods Rising, S2 Scheduling Delay | 새 파드 배치 지연, Pending 증가 | Pressure만 보고 넘겨도, **여기서** 리소스 확장·우선순위 조정 가능 |
| 3 | Instability | I1 Excessive Restarts, I3 Pending High | OOM kill·재시작 루프, Pending 다수 | 이미 실패 발생; 원인 조사·롤백 검토 |
| 4 | Impact | M1 Empty Endpoints 또는 M2 Service Degradation | 게이트웨이 지연 급증, SLO 위반 | 즉시 복구 필요 |

**교훈:** Pressure(P2) 또는 Strain(S1,S2)에서 경고를 잡으면, Instability·Impact **전에** 용량·스케일 조정으로 인시던트를 줄일 수 있다. 체인은 "어느 단계에서 먼저 반응할 것인가"를 명확히 한다.

**사례 2: 디스크 압박 → 이미지 풀/로그 지연 → Eviction·재시작 → 일시적 5xx**

| 단계 | Stage | 관찰 신호 | 설명 |
|------|-------|-----------|------|
| 1 | Pressure | P3 Node Disk Low | 노드 디스크 여유 &lt;20%. 이미지 풀, 로그 쓰기 지연 시작. |
| 2 | Strain | (Control/Workload) 파드 생성·시작 지연 | 스케줄러는 동작하지만 노드에서 컨테이너 시작이 느려짐. |
| 3 | Instability | I1 Restarts (Eviction), 일부 I2 NotReady | kubelet이 디스크 부족으로 Eviction, 파드 재시작. |
| 4 | Impact | M2 Service Degradation (일시적 5xx) | 백엔드 일시 부족으로 게이트웨이 5xx, SLO 위반. |

**교훈:** P3(디스크)만 주기적으로 봐도 "곧 Eviction·5xx"를 예측할 수 있다. 체인은 **인프라 원인 → 서비스 영향**까지의 경로를 공통 언어로 제공한다.

**사례 3: API server 부하 → 스케줄링 지연 → Pending 증가 → 배포 지연 (Impact는 경미)**

| 단계 | Stage | 관찰 신호 | 설명 |
|------|-------|-----------|------|
| 1 | (Pressure 약함) | 노드 리소스는 여유 있음 | 인프라 자체는 여유. |
| 2 | Strain | S2 Scheduling Delay, S1 Pending Rising | API server 또는 Scheduler 부하로 새 파드 스케줄 지연. |
| 3 | Instability | I3 Pending High | 배포·스케일업 시 Pending이 오래 유지. |
| 4 | Impact | (경미) 배포 SLA 지연 또는 일부 타임아웃 | 사용자-facing SLO는 약간만 나쁘거나 정상. |

**교훈:** Strain만 올라와도 "지금은 Impact가 없지만, 계속되면 문제"라고 판단할 수 있다. **조기 Stage에 반응**하면 Impact를 아예 만들지 않거나 줄일 수 있다.

이런 **인시던트 전파 사례**를 팀·과거 사고와 매핑해 두면, 체인 모델이 "실제로 조기 감지에 도움이 된다"는 것을 검증·설명하는 데 쓸 수 있다.

### 8.3 참조 대시보드 모델 (Reference Dashboard Model)

**Operational Confidence Dashboard**는 "메트릭 나열"이 아니라 **Stage·레이어·드릴다운 경로**가 한눈에 보이는 **개념 레이아웃**을 따른다. 구현은 Grafana 등 자유이나, 아래 구조를 만족하면 운영 확신에 맞는 대시보드가 된다.

**1) Stage 건강 상태 (Stage Health)**

- **한 줄 요약**: Pressure / Strain / Instability / Impact 네 Stage별로 **정상 / 경고 / 위험** 상태를 하나의 뱃지 또는 색으로 표시.
- **판단 규칙**: v1과 동일. Instability·Impact가 모두 정상이면 "지금 안전"; 하나라도 비정상이면 "조사 필요". Pressure·Strain에 경고가 있으면 "조기 리스크".
- **위치**: 대시보드 **최상단**. 5초 안에 "지금 안전한가?", "내일 안전한가?"에 답할 수 있어야 함.

**2) Operational Signal Topology (운영 신호 위상) 레이어**

- **레이어별 블록**: Infrastructure → Cluster Control → Workload → Traffic → Service Reliability (및 선택적으로 Platform Topology) 순으로 **한 행 또는 한 섹션**씩 배치.
- **각 레이어 내**: 해당 레이어의 대표 신호(v1 + v2 선택 신호)만 표시. 예: Infrastructure에는 P1, P2, P3; Control에는 S1, S2; … .
- **시각적 연속성**: 체인 방향(왼쪽 → 오른쪽 또는 위 → 아래)이 **전파 방향**과 일치하도록 배치. "어디에서 먼저 막혔는가"를 직관적으로 볼 수 있게 함.

**3) 드릴다운 경로 (Drill-down Path)**

- **Stage 또는 레이어 클릭** → 해당 Stage/레이어의 **상세 패널**로 이동 (예: Strain 클릭 → S1, S2 추이, Pending 목록, Scheduler 메트릭).
- **신호 클릭** → 해당 신호의 **TOP N** (예: CPU 높은 노드 TOP10, 재시작 많은 파드 TOP10) 및 **권장 행동** 한 줄 표시.
- **Runbook 링크**: 각 경고 상태 패널에서 "무엇을 해야 하나?"에 해당하는 Runbook 또는 행동 요약으로 바로 연결.

**4) 5분 원칙**

- **첫 화면만으로** Stage 건강 + "무엇을 해야 하나?"(가장 심한 Stage 기준)가 결정 가능해야 함.  
- 상세 메트릭·트레이스·로그는 **드릴다운 후**에만 나오도록 하여, "메트릭 폭발"을 피함.

이 레이아웃을 **참조 모델**로 두고, Grafana 등에서 패널만 채우면 된다. JSON은 요구하지 않으며, 구현체는 팀·환경에 맞게 조정 가능하다.

### 8.4 운영자 멘탈 모델 (Operator Mental Model)

운영자가 이 프레임워크를 **생각하는 방식**을 명시하면, 실제 채택과 일관된 판단이 쉬워진다.

**1) 앞쪽 Stage로 예측 (Predict using early stages)**

- **Pressure, Strain**을 "미래 리스크"의 선행 지표로 본다.  
- "지금 Impact는 없지만 Strain이 나쁘다 → 곧 Instability·Impact가 될 수 있다"고 **예측**하고, **사전에** 용량·스케일·우선순위 조정을 한다.  
- 멘탈: **"앞쪽이 나쁘면, 뒤쪽을 막기 위해 움직인다."**

**2) 뒤쪽 Stage로 진단 (Diagnose using later stages)**

- **Instability, Impact**를 "현재 상태"의 결론으로 본다.  
- "지금 안전한가?"는 Instability·Impact가 정상인지로 **진단**한다.  
- Impact가 나쁘면 "이미 사용자/SLO에 영향이 있다" → 즉시 복구; Instability만 나쁘면 "원인 조사·롤백 검토"로 **원인**을 좁힌다.  
- 멘탈: **"뒤쪽이 나쁘면, 지금 무슨 일이 일어나고 있는지 결론 내리고 행동한다."**

**3) Stage에 따라 행동 우선순위 (Prioritize actions based on stage)**

- **Impact** → 최우선: 즉시 복구(롤백, 트래픽 우회, 스케일업).  
- **Instability** → 그다음: 원인 조사(로그, 이벤트, TOP 워크로드), 필요 시 롤백.  
- **Strain** → 그다음: 사전 대응(리소스 확장, 우선순위 조정).  
- **Pressure** → 시간 여유 있음: 용량 계획(노드 추가, 워크로드 분산, 정리).  
- 멘탈: **"가장 뒤쪽에서 나쁜 Stage가 오늘의 우선순위를 정한다."**

**4) 레이어로 원인 위치 좁히기 (v2 확장)**

- "어느 Stage가 나쁜가?"에 더해 **"어느 레이어(Infrastructure / Control / Workload / Traffic)에서 먼저 나쁜가?"**를 보면, 원인이 노드인지, 스케줄러인지, 워크로드인지, 게이트웨이인지 빠르게 좁힐 수 있다.  
- 멘탈: **"Stage는 '언제/얼마나 심한가', 레이어는 '어디서'를 답한다."**

이 **멘탈 모델**을 팀 표준으로 두면, "무엇을 먼저 볼 것인가", "무엇을 먼저 할 것인가"가 프레임워크와 일치하여, 5분 설명으로도 실제 운영에서 일관되게 쓰일 수 있다.

---

## 9. Operational Confidence Framework v4: 검증 및 구현 가이드

v4는 v1–v3를 **검증 가능하고 구현 가능한** 프레임워크로 완성한다. **인시던트 분석을 통한 검증 방법**, **참조 신호 구현**(Prometheus 메트릭 예시), **Operational Confidence Score**(선택 개념), **도입 단계(Deployment Model)**를 추가하여, 컨퍼런스·블로그·연구 문서에서 “실제 시스템 동작에 기반하며 즉시 적용 가능하다”는 것을 보여 준다. 5분 설명·최소 신호 원칙은 유지한다.

### 9.1 인시던트 분석을 통한 프레임워크 검증 (Framework Validation Using Incident Analysis)

프레임워크가 **실제 시스템 동작**에 기반함을 보이려면, 과거 또는 가상의 인시던트를 **체인 단계별로 재구성**하고, 각 단계가 **어떤 메트릭에 어떻게 나타나는지** 매핑하면 된다. 이렇게 하면 “체인이 단순 이론이 아니라 메트릭으로 관측 가능한 현상”임을 입증할 수 있다.

**인시던트 재구성 예: Node memory pressure → scheduler delay → pod restart loops → gateway latency spike → service SLO violation**

| 체인 단계 | 현상 | 메트릭에서의 관측 (대표 예) | 검증 포인트 |
|-----------|------|-----------------------------|-------------|
| **Pressure** | 노드 메모리 압박 | `node_memory_MemAvailable_bytes` / `node_memory_MemTotal_bytes` 비율 저하; 또는 (1 - MemAvailable/MemTotal)*100 > 80. 노드별 시계열에서 **트래픽 영향 전** 수십 분~수 시간 전에 상승. | “Impact 발생 **이전**에 Pressure 구간이 메트릭에 존재했는가?” |
| **Strain** | 스케줄링 지연, Pending 증가 | `kube_pod_status_phase{phase="Pending"}` 개수 **추세 증가**; control plane 있으면 `scheduler_pending_pods` 또는 스케줄링 레이턴시. Pressure 구간 **이후**, Instability **이전** 시점에 지연·Pending 증가가 보여야 함. | “Pressure와 Instability **사이**에 Strain 구간이 메트릭에 존재했는가?” |
| **Instability** | 파드 재시작 루프, Pending 다수 | `kube_pod_container_status_restarts_total` 의 `increase(...[10m])` 급증; `kube_pod_status_phase{phase="Pending"}` 절대값이 임계값 초과. OOM/Eviction 시 메모리 Pressure 노드와 시간적 연관. | “실패(재시작·Pending high)가 **Impact 직전 또는 동시**에 메트릭에 나타났는가?” |
| **Impact** | 게이트웨이 지연 급증, SLO 위반 | Ingress/Gateway 요청 지연·5xx 비율 상승; 또는 `kube_endpoint_address_available` 이 0이 되는 서비스. 사용자-facing SLO 위반 시점과 일치. | “서비스/SLO 영향 시점에 Impact에 해당하는 메트릭이 악화되었는가?” |

**검증 절차 요약:**  
1) 과거 인시던트에서 **Impact 발생 시점**을 기준으로 삼고, 2) 그 시점 **이전**으로 거슬러 올라가며 위 표의 메트릭 시계열을 확인한다. 3) Pressure → Strain → Instability → Impact 순서로 **각 단계가 메트릭에 존재하는지** 기록한다. 4) 여러 인시던트에서 반복되면 “체인이 실제 동작을 잘 묘사한다”는 **경험적 근거**가 된다.  
이렇게 **인시던트 재구성 + 메트릭 매핑**을 문서화해 두면, 외부 발표 시 “프레임워크가 실제 시스템 행동에 기반한다”는 주장을 뒷받침할 수 있다.

### 9.2 참조 신호 구현 (Reference Signal Implementation)

최소 신호 집합이 **실제로 어떤 Prometheus 메트릭으로 구현 가능한지** 대표 예만 제시한다. 전체 대시보드나 AlertRule 전부를 정의하지는 않으며, “즉시 구현 가능함”을 보이는 수준으로 한정한다.

| 신호 (v1) | 의미 | Prometheus 메트릭 예시 (대표) | 구현 참고 |
|-----------|------|-------------------------------|-----------|
| **P1 Node CPU High** | 노드 CPU 압박 | `node_cpu_seconds_total` (mode="idle" 제외 후 100-idle 비율). 예: `100 - (avg by (node) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)` > 80 | node_exporter |
| **P2 Node Memory High** | 노드 메모리 압박 | `node_memory_MemAvailable_bytes`, `node_memory_MemTotal_bytes`. 사용률 = `100 * (1 - (MemAvailable/MemTotal))` > 80 | node_exporter |
| **P3 Node Disk Low** | 노드 디스크 여유 부족 | `node_filesystem_avail_bytes`, `node_filesystem_size_bytes` (mountpoint="/" 등). 여유 비율 < 20% | node_exporter |
| **S1 Pending Pods Rising** | Pending 파드 수 추세 증가 | `kube_pod_status_phase{phase="Pending"} == 1` 의 count; 15분 구간 `increase` 또는 `delta` 로 “증가 중” 판단 | kube-state-metrics |
| **S2 Scheduling Delay** | 스케줄링 지연 | `scheduler_pending_pods` 또는 `scheduler_e2e_scheduling_duration_seconds_*` (control plane 메트릭) | kube-scheduler (Managed K8s에서는 미제공 가능) |
| **I1 Excessive Restarts** | 파드 재시작 급증 | `kube_pod_container_status_restarts_total` 의 `sum(increase(...[10m]))` 또는 노드/파드별 increase. 팀 정의 임계값 초과 | kube-state-metrics |
| **I2 NotReady Nodes** | NotReady 노드 존재 | `kube_node_status_condition{condition="Ready",status!="true"}` count > 0 | kube-state-metrics |
| **I3 Pending Pods High** | Pending 파드 수 절대값 높음 | `kube_pod_status_phase{phase="Pending"} == 1` count > 0 (Warning), > 5 (Critical) 등 | kube-state-metrics |
| **M1 Empty Endpoints** | 핵심 서비스 Endpoint 없음 | `kube_endpoint_address_available` (또는 `kube_endpoints_address_available`) == 0 인 서비스. namespace/label로 critical 서비스 필터 | kube-state-metrics |
| **M2 Service Degradation** | 서비스 지연·에러율 악화 | Ingress/Gateway 메트릭: 요청 지연 histogram, 5xx rate 등. SLO 임계값 초과 | Ingress controller / Gateway exporter |

**구현 시 유의사항:**  
- 메트릭 이름·레이블은 Prometheus·exporter 버전에 따라 다를 수 있음. 위는 **대표 예**이며, 실제 환경에 맞게 조정해야 함.  
- Managed Kubernetes에서는 S2용 control plane 메트릭이 없을 수 있으므로, S1만으로 Strain을 판단.  
- M2는 트래픽 관측점(Ingress/메시)이 있을 때만 구현; 없으면 M1만으로 Impact 판단.  
이 표만으로도 “프레임워크의 각 신호가 **즉시 구현 가능한 메트릭**과 1:1로 연결된다”는 것을 보여 줄 수 있다.

### 9.3 Operational Confidence Score (선택 개념)

Stage 건강 상태를 **단일 점수**로 요약하는 것은 선택 사항이다. 복잡한 점수 공식은 5분 설명 원칙에 어긋나므로, **개념적으로 단순한** 수준만 정의한다.

**개념 정의:**  
시스템이 “지금 운영 관점에서 얼마나 확신을 가질 수 있는가”를 **High / Medium / Low / Critical** 네 단계로 표현한다.  
- **High**: 모든 Stage(Pressure, Strain, Instability, Impact) 정상. → “지금 안전하고, 당분간 안전할 가능성이 높다.”  
- **Medium**: Pressure 또는 Strain에 경고. Instability·Impact는 정상. → “지금은 괜찮지만, 조기 리스크가 있다. 사전 대응 권장.”  
- **Low**: Instability에 이상. Impact는 정상일 수 있음. → “문제가 진행 중이다. 원인 조사 필요.”  
- **Critical**: Impact에 이상. → “이미 서비스/SLO 영향. 즉시 복구 필요.”

**구현 시 권장:**  
점수 공식(가중합, 세부 점수)을 과하게 만들지 말고, **가장 나쁜 Stage 하나**로 위 네 구간을 결정하는 방식이 단순하고 설명하기 쉽다. 예: “가장 뒤쪽에서 비정상인 Stage가 Confidence 수준을 정한다.”  
이 Score는 **대시보드 상단 한 줄 요약**이나 **알림 정책**(예: Critical일 때만 즉시 호출)에 활용할 수 있으며, 프레임워크의 필수 구성요소는 아니다.

### 9.4 도입 단계 (Deployment Model)

팀이 이 프레임워크를 **단계적으로 도입**하는 방법을 간단히 정리한다. 재설계가 아니라 **기존 메트릭·대시보드·Runbook을 프레임워크에 맞추는** 순서다.

| 단계 | 내용 | 산출물 예시 |
|------|------|-------------|
| **Step 1: 기존 메트릭을 프레임워크 신호에 매핑** | 이미 수집 중인 Prometheus 메트릭 중에서, v1 10개 신호(및 필요 시 v2 선택 신호)에 대응하는 메트릭·쿼리를 나열. 없으면 node_exporter, kube-state-metrics 등으로 보강. | “신호–메트릭 매핑표”(9.2절 표 참고). |
| **Step 2: Stage 대시보드 구축** | v3 참조 대시보드 모델에 따라, Stage Health(최상단) + Operational Signal Topology 레이어 + 드릴다운 경로를 한 화면에 구성. Grafana 등으로 구현. | Operational Confidence Dashboard 초안. |
| **Step 3: Runbook 정의** | 각 Stage·신호별로 “이상 시 무엇을 할 것인가”를 Runbook으로 작성. 기존 Runbook이 있으면 Stage/레이어 태그를 붙여 연결. | Stage/신호별 Runbook 또는 기존 Runbook에 Stage 링크. |
| **Step 4: 운영자 멘탈 모델 채택** | v3 멘탈 모델(앞쪽으로 예측, 뒤쪽으로 진단, Stage별 우선순위, 레이어로 원인 좁히기)을 팀 표준으로 정하고, 일상 점검·인시던트 시 “먼저 Stage/레이어를 보라”는 절차를 공유. | 팀 가이드 1페이지 또는 온보딩 자료. |

**원칙:**  
한 번에 전부 바꾸지 말고, Step 1부터 순서대로 적용한다. Step 1만 완료해도 “우리 메트릭이 프레임워크의 어디에 해당하는가”가 명확해지고, Step 2가 되면 “지금/내일 안전한가?”를 5초 안에 답할 수 있는 뷰가 생긴다.  
이 **Deployment Model**을 문서에 포함하면, “이론만이 아니라 우리 팀이 이렇게 도입할 수 있다”는 실용적 가이드가 되어, 외부 발표·블로그·연구 문서의 신뢰도를 높일 수 있다.

---

## 기존 모델과의 비교

### ai-team-lab 모델 vs 체인 기반 모델

| 측면 | ai-team-lab (Block 모델) | 체인 기반 모델 |
|------|--------------------------|---------------|
| **핵심 구조** | 3개의 독립적 Block | 1개의 연속적 체인 |
| **인과관계** | 암묵적, 문서에만 서술 | 명시적, 구조에 내장 |
| **질문 매핑** | Block별로 분리 | 체인 위치로 자연스럽게 도출 |
| **설명 용이성** | "3개 Block이 있고..." | "체인이 있고, 앞쪽은 예측, 뒤쪽은 진단" |
| **행동 연결** | 별도 문서에 정의 | 체인 위치가 곧 행동 종류 |

### 무엇이 유지되었는가

- **운영 확신(Operational Confidence)** 개념
- **"지금 안전한가?" / "곧 위험한가?"** 두 질문
- **선행/후행 지표** 구분
- **최소 신호** 원칙
- **드릴다운** 구조

### 무엇이 개선되었는가

- **인과관계 명시화**: 체인 구조로 신호 간 관계가 명확해짐
- **단순화**: "하나의 체인"으로 모든 것을 설명
- **행동 연결 강화**: 체인 위치 → 행동 종류 직접 매핑
- **확장 용이성**: 새 신호를 추가할 때 "어느 Stage인가?"만 결정하면 됨

---

## 체인 모델의 한계와 다른 모니터링과의 공존

### 한계: 체인을 따르지 않는 실패

고장 전파 체인(Pressure → Strain → Instability → Impact)은 **리소스 압박이 단계적으로 전파되는 유형**의 문제를 모델링한다. 다음 유형의 실패는 **체인으로 잘 설명되지 않거나**, 체인 앞단 없이 **갑자기** 나타난다.

| 유형 | 예시 | 왜 체인과 맞지 않는가 | 권장 대응 |
|------|------|------------------------|-----------|
| **급성 하드웨어/네트워크 장애** | 노드 갑작스런 다운, 네트워크 파티션, 디스크 고장 | Pressure/Strain 없이 Instability 또는 Impact로 바로 나타남 | 노드/네트워크 상태 알람, Liveness 체크를 **체인과 별도**로 유지. 체인은 "예측 가능한 전파"용, 급성 장애는 **즉시 복구 플로우**로 처리. |
| **배포/설정 오류** | 잘못된 이미지, 잘못된 리소스 limit, 잘못된 설정맵 | 인프라 압박이 아니라 **변경**으로 인한 실패. 체인 단계 순서와 무관. | **변경 관리·배포 모니터링** (롤아웃 상태, 배포 전후 비교)을 별도로 두고, Instability/Impact 신호와 연계해 "최근 변경"을 함께 확인. |
| **외부 의존성 장애** | 외부 API 타임아웃, 외부 DB 장애 | 클러스터 내부 Pressure/Strain과 무관하게 서비스 품질이 나빠짐. | **의존성·연동 모니터링** (외부 엔드포인트 가용성, 에러율)을 Impact 또는 별도 레이어로 두고, "클러스터 원인 vs 외부 원인" 구분에 활용. |

### 다른 모니터링과의 공존

이 프레임워크는 **전체 모니터링을 대체하는 것이 아니라**, "운영 확신"을 위한 **최소 판단 축**을 제공한다.

- **체인 기반 판단**: "지금/내일 안전한가?", "무엇을 해야 하나?"는 **이 체인과 최소 신호 집합**으로 답한다.
- **급성 장애·설정 오류·외부 의존성**은 위 표와 같이 **별도 관점**으로 모니터링하고, 필요 시 Runbook에서 "Impact만 나왔고 Instability는 없음 → 최근 배포/외부 의존성 확인"처럼 **체인 결과와 조합**해 원인 후보를 좁힌다.
- **세부 진단·성능 튜닝**은 체인 밖의 메트릭(상세 프로파일링, 분산 트레이스, 로그)으로 수행한다. 체인은 **결론과 행동**을 내리는 데 초점을 두고, 깊은 조사는 기존 도구와 절차를 그대로 둔다.

이렇게 **체인 = 리소스 전파 기반의 예측·판단**, **그 외 = 급성·설정·외부·심층 조사**로 역할을 나누면, 프레임워크가 단순성을 유지하면서도 실제 환경의 다양한 실패 유형과 공존할 수 있다.

---

## 리스크

| 리스크 | 가능성 | 영향 | 대응 |
|--------|--------|------|------|
| 체인 모델이 모든 실패 유형을 포괄하지 못함 | 중 | 중 | 위 "체인 모델의 한계" 절에 명시; 별도 모니터링과 공존 방식 정의 |
| 10개 신호가 환경에 따라 부족할 수 있음 | 중 | 낮 | 확장 가능한 구조; 최소 신호 선정 원칙에 따라 추가 |
| 기존 Block 모델 사용자의 혼란 | 낮 | 낮 | Block 모델과의 매핑 문서 제공 |

---

## 다음 단계

- [x] 아키텍처 설계 완료
- [ ] 실용 가이드 작성 (engineering)
- [ ] 검토 (review)
- [ ] 최종 문서화 (documentation)
