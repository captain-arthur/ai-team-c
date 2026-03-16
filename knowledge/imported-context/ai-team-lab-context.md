# ai-team-lab Context Import

**Source:** `/Users/hooni/Documents/github/ai-team-lab`
**Import Date:** 2026-03-17
**Purpose:** ai-team-c가 이전 작업의 맥락을 공유하고, 동일한 철학과 방향 위에서 발전할 수 있도록 핵심 컨텍스트를 정리한다.

---

## 1. 해결하려던 핵심 문제

### 1.1 근본 질문

ai-team-lab에서 다룬 두 가지 핵심 문제는 모두 **"어떻게 확신을 가질 수 있는가?"**라는 질문에서 출발한다.

| 영역 | 질문 |
|------|------|
| **CAT** | "이 Kubernetes 클러스터를 **수용해도 되는가**?" — 클러스터가 기능적으로, 성능적으로 기준을 만족하는지 **확신 있게 판단**할 수 있는가? |
| **SRE Monitoring** | "이 클러스터가 **지금 안전한가**?" — 운영자가 metric을 보고 **확신 있게 "안전하다"고 말할 수 있는가**? |

두 문제 모두 **"많은 데이터가 있지만 확신이 없다"**는 상황에서 출발한다. metric이 넘쳐도, 벤치마크 결과가 있어도, **"그래서 괜찮은 건가?"**에 명확히 답하기 어려운 상태다. ai-team-lab의 작업은 이 **확신의 부재**를 해결하려는 시도였다.

### 1.2 문제 해결 접근

두 영역 모두에서 동일한 접근을 취했다:

1. **"무엇이 중요한가?"를 먼저 정의** — metric 목록이 아니라 **운영적으로 의미 있는 조건**을 먼저 정의한다.
2. **최소한의 신호로 결정** — 모든 것을 보는 대신, **결정에 필요한 최소 신호**만 선별한다.
3. **명확한 판단 규칙** — "이 조건이 만족되면 X, 아니면 Y"라는 **단순하고 명시적인 판단 규칙**을 둔다.
4. **이론적 정당화** — 왜 이 신호들인지, 왜 이 임계값인지를 **문서화된 근거**로 설명한다.

---

## 2. 두 가지 주요 작업 스트림

### 2.1 CAT (Cluster Acceptance Test)

#### 목표

**실용적인 CAT 시스템을 완성하는 것**이 목표다. 단순한 벤치마크나 연구가 아니라, **"이 클러스터를 수용한다 / 거부한다"는 명확한 판단을 내릴 수 있는 실행 가능한 시스템**을 만든다.

#### 범위

CAT는 두 가지 관점을 모두 다뤄야 한다:

| 관점 | 내용 | 현재 상태 |
|------|------|-----------|
| **Conformance / 기능성** | Kubernetes API·핵심 동작이 표준을 따르는가? | 향후 단계 |
| **Performance / 부하** | 부하 하에서 기대 성능·안정성을 만족하는가? | **현재 초점** |

#### 핵심 산출물: CAT v1 명세

**Stable SLI (프로덕션 수용 기준):**

| SLI | SLO | 의미 |
|-----|-----|------|
| ClusterOOMsTracker failures | = 0 | OOM 1건이라도 발생하면 FAIL |
| SystemPodMetrics max restartCount | = 0 | 시스템 파드 1회라도 재시작하면 FAIL |

**Provisional SLI (임시, 정제 대상):**

| SLI | SLO (임시) | 비고 |
|-----|------------|------|
| PodStartupLatency P99 | ≤ 130 s | 실험 베이스라인 기반, 향후 조정 |
| StatelessPodStartupLatency P99 | ≤ 75 s | 실험 베이스라인 기반, 향후 조정 |

**판정 로직:**

```
Stable SLI 중 하나라도 FAIL  →  CAT_RESULT = FAIL (수용 불가)
Stable 전부 PASS, Provisional 중 FAIL  →  CAT_RESULT = PASS_WITH_WARNINGS
전부 PASS  →  CAT_RESULT = PASS
```

#### 검증에서 배운 것

- **레이턴시 변동성이 매우 크다**: kind/로컬 환경에서 PodStartupLatency P99가 64s ~ 122s로 약 2배 차이.
- **변동 원인**: 이미지 풀/cold start, 노드 리소스 압박, 스케줄링 지연이 likely. Control plane 병목, 스토리지/네트워크는 possible.
- **환경별 SLO 구분 필요**: 로컬(kind)과 실제 클러스터의 SLO를 같은 숫자로 묶지 않는다.
- **Stable SLI는 안정적**: OOM failures=0, restartCount=0은 4회 실험 모두에서 안정. 신뢰할 수 있는 기준점.

#### devcat 현실

- **devcat**은 실제 구현 저장소 (ai-team-lab과 별개).
- ClusterLoader2 + perfdash + config.yaml + overrides 구조.
- 현재는 assertion/PASS·FAIL 표현이 명확하지 않음.
- 작업 모델: **research → experiment → interpretation → devcat improvement**

---

### 2.2 SRE Monitoring (Dashboard Design)

#### 목표

**운영 확신(Operational Confidence)**을 제공하는 것이 목표다. metric 양이 아니라 **"지금 안전한가?"에 확신 있게 답할 수 있는지**로 성공을 측정한다.

#### 핵심 개념: 운영 확신 이론 (Operational Confidence Theory)

**5가지 운영 안전 조건 (Operational Safety Conditions):**

| 조건 | 설명 |
|------|------|
| **C1** | Scheduling path is healthy — 스케줄 경로 정상 |
| **C2** | Critical traffic path is healthy — 트래픽 경로 유효 |
| **C3** | Core workload stability is intact — 워크로드 안정 |
| **C4** | No evidence of active resource collapse — 리소스 붕괴 없음 |
| **C5** | No evidence of cascading instability — 연쇄 불안정 없음 |

**고장 전파 경로 (Failure Propagation Paths):**

```
경로 1: 노드 압박 → 스케줄 지연 → Pending 증가 → Endpoint 부족 → 서비스 저하
경로 2: 메모리 압박 → OOM/재시작 폭증 → 백엔드 불안정 → 서비스 저하
경로 3: Ingress 스트레스 → 지연·에러 증가 → 서비스 영향
경로 4: 제어면 장애 → 스케줄·조정 정지 → 전역 영향
```

**선행(leading) vs 후행(lagging) 지표:**

| 유형 | 의미 | 예시 |
|------|------|------|
| **Leading** | 서비스 영향 **이전**에 나타남 | Node CPU/메모리/디스크, Pending 추세 |
| **Lagging / Already-damaged** | 손상 **이후**에 나타남 | NotReady, Pending count, 재시작 폭증 |

#### 핵심 산출물: 3계층 대시보드 모델

| Block | 이름 | 답하는 질문 | 구성 |
|-------|------|-------------|------|
| **Block 1** | Operational Confidence | 지금 클러스터가 안전한가? | 4~6개 패널 (안전 조건 위반 여부) |
| **Block 2** | Early Risk | 곧 불안전해질 징후가 있는가? | 4~6개 패널 (선행 지표) |
| **Block 3** | Investigation / Top Offenders | 어디를 먼저 조사할 것인가? | 4~6개 패널 (드릴다운 전용) |

**운영 확신 규칙:**

```
Block 1 전부 정상 = C1~C5 만족 = "현재 안전"
Block 2 경고 없음 = 조기 리스크 없음 = "가까운 미래도 안전할 가능성 높음"
Block 1 비정상 = "조사 필요" → Block 3 해당 TOP10으로 원인 좁히기
```

#### Cluster Health Monitoring Model

**세 가지 운영 질문:**

1. 지금 클러스터가 건강한가? → **Cluster Health Summary**
2. 클러스터가 곧 불건강해질 징후가 있는가? → **Trend / Risk Indicators**
3. 문제가 있다면 어디를 먼저 조사할 것인가? → **Top Offenders / Drill-down**

**5가지 건강 카테고리:**

- Node / Infrastructure
- Control Plane
- Workload / Pod
- Capacity / Resource pressure
- Network / Ingress

---

## 3. 확립된 철학과 원칙

### 3.1 SLI/SLO 철학

| 원칙 | 설명 |
|------|------|
| **도구 기본값을 그대로 쓰지 않는다** | 벤치마크 도구가 주는 넓은 임계값을 무비판적으로 SLO로 쓰지 않는다. |
| **정당화가 필요하다** | User expectations, environment assumptions, repeatable measurement, practical acceptance meaning으로 정당화해야 한다. |
| **"You promise, we promise"** | SLO는 **약속**이다. 팀이 "이 조건에서 이 수준을 보장한다"고 명시하고, 시스템이 그것을 만족하는지 검증한다. |
| **소규모 ≠ 대규모** | 소규모 클러스터의 수용 기준은 대규모 벤치마크 임계값과 다를 수 있다. 규모·용도에 맞게 별도 정의한다. |
| **Stable vs Provisional** | 검증된 지표(Stable)와 정제가 필요한 지표(Provisional)를 구분한다. |

### 3.2 CAT 설계 원칙

| 원칙 | 설명 |
|------|------|
| **완벽한 단일 도구는 없다** | 여러 도구를 실용적으로 조합한다. |
| **도구는 as-is로 사용** | 내부 소스 수정 없이 설정·플러그인·래퍼로 연동한다. |
| **테스트 = Job** | Scenario injector + SLI/SLO measurement + Assertion으로 구성. |
| **결과는 정의된 디렉터리에** | 일관된 형식으로 저장하면 자동 시각화가 가능해진다. |
| **단순한 오케스트레이션** | Prow 같은 복잡한 시스템은 정말 필요할 때만. |
| **ClusterLoader2가 출발점** | 가장 현실적인 시작은 CL2 부하 테스트 기반 수용 케이스. |

### 3.3 모니터링 철학

| 원칙 | 설명 |
|------|------|
| **가시성 ≠ 확신** | metric이 많다고 운영 확신이 생기지 않는다. |
| **최소 신호 모델** | 모든 것을 보지 않고, "안전한가?"에 직접 답하는 소수의 상위 신호만 둔다. |
| **운영 조건 먼저** | "어떤 metric을 볼까?"가 아니라 "어떤 조건이 만족되면 안전한가?"를 먼저 정의한다. |
| **선행 vs 후행 구분** | 사전 대응이 가능한 선행 지표와, 이미 손상된 상태를 보는 후행 지표를 구분한다. |
| **드릴다운 분리** | 판단은 Summary에서, 조사는 Top Offenders에서. 역할을 명확히 나눈다. |

---

## 4. 미해결 문제

### 4.1 CAT 영역

| 문제 | 상태 | 다음 단계 |
|------|------|-----------|
| **레이턴시 SLO 정제** | Provisional로 남아 있음 | 이미지 preload, warm-up run, 시나리오 규모 축소 실험 후 조정 |
| **환경별 SLO** | 개념만 정의됨 | 로컬(kind) vs 실제 클러스터별 SLO 구체화 |
| **Conformance 통합** | 미착수 | Sonobuoy 등 기능 테스트 도구 통합 |
| **Assertion 표현** | 형식 미정 | JUnit XML, JSON, 공통 스키마 결정 |
| **시각화 개선** | perfdash만으로는 부족 | PASS/FAIL 표현이 포함된 대시보드 검토 |

### 4.2 Monitoring 영역

| 문제 | 상태 | 다음 단계 |
|------|------|-----------|
| **Alert 정책** | 미착수 | Block 1/2 신호 중 알림 대상, 임계치, 심각도 정의 |
| **Runbook** | 미착수 | Block 1 비정상 시 조사 순서, 체크리스트, 명령어 정리 |
| **Eviction 신호** | v1에서 제외 | kubelet metric 또는 이벤트로 수집 방법 검토 |
| **Ingress 신호** | 선택적 | Ingress controller metric 안정화 후 추가 |
| **Threshold 팀 정의** | 예시만 있음 | Excessive restarts N, Critical namespace 목록 등 팀에서 확정 |

---

## 5. 현재 공유 컨텍스트로 삼아야 할 것

### 5.1 CAT 관련

- **CAT v1 명세**는 devcat에서 **즉시 사용 가능한** 첫 수용 테스트 명세다.
- **Stable SLI**(OOM=0, restartCount=0)는 신뢰할 수 있는 **기준점**이다.
- **Provisional SLI**(레이턴시)는 **정제가 필요**하며, 환경·실험에 따라 조정된다.
- 작업은 **research → experiment → interpretation → devcat improvement** 흐름을 따른다.
- **cat-vision, cat-design-principles, sli-slo-philosophy, devcat-program-brief**가 방향을 정의한다.

### 5.2 Monitoring 관련

- **Cluster Health Monitoring Model**이 모니터링 모델의 기초다.
- **Operational Confidence Theory**가 대시보드 설계의 이론적 근거다.
- **3계층 대시보드 모델**(Operational Confidence / Early Risk / Investigation)이 구조다.
- **Central Kubernetes Operational Dashboard Design v1**이 구현 가능한 산출물이다.
- Alert 정책과 Runbook은 **후속 프로젝트**로 남아 있다.

### 5.3 공통

- **"확신을 위한 최소 신호"**가 두 영역의 공통 원칙이다.
- **정당화된 임계값**만 사용한다. 도구 기본값이나 막연한 숫자를 쓰지 않는다.
- **환경·맥락**에 맞게 기준을 조정한다. 범용 숫자를 무비판적으로 적용하지 않는다.

---

## 6. ai-team-c에서 이 컨텍스트를 사용하는 방법

### 6.1 새 프로젝트 시작 시

1. **관련 컨텍스트 확인**: CAT 또는 Monitoring 관련 작업이면 이 문서와 관련 원칙을 먼저 읽는다.
2. **기존 철학 존중**: SLI/SLO 철학, CAT 설계 원칙, 모니터링 철학을 따른다.
3. **미해결 문제 참조**: 새 작업이 미해결 문제와 관련되면 기존 맥락 위에서 진행한다.

### 6.2 CAT 관련 작업 시

- **CAT v1 명세**를 기준점으로 삼는다.
- **Provisional SLI 정제**는 실험 → 해석 → 조정 흐름을 따른다.
- **devcat** 저장소의 현실(CL2, config.yaml, overrides, results/)을 존중하고 점진적으로 개선한다.
- 새로운 SLI/SLO를 정의할 때는 반드시 **정당화**(user expectations, environment assumptions, repeatable measurement, practical acceptance meaning)를 문서화한다.

### 6.3 Monitoring 관련 작업 시

- **Cluster Health Monitoring Model**과 **Operational Confidence Theory**를 기반으로 한다.
- 새 신호를 추가할 때는 **어떤 안전 조건·전파 단계·역할(leading/lagging)**에 해당하는지 먼저 정리한다.
- **3블록 구조**를 유지한다. 새 카테고리가 필요하면 기존 구조 위에 확장한다.
- Alert 정책·Runbook은 대시보드 설계와 별개 프로젝트로 진행한다.

### 6.4 지식 축적

- CAT/Monitoring 작업에서 배운 것은 `knowledge/`에 저장한다.
- 새로운 원칙이나 철학은 기존 원칙과 일관성을 유지하며 추가한다.
- 미해결 문제가 해결되면 이 문서를 업데이트한다.

### 6.5 Claude Code 행동 지침

ai-team-c에서 Claude가 CAT 또는 Monitoring 관련 작업을 수행할 때:

1. **이 컨텍스트 문서를 공유 지식으로 취급**한다. ai-team-lab의 결론과 철학을 알고 있다고 가정한다.
2. **기존 원칙과 충돌하는 제안을 피한다**. 예: 도구 기본값을 무비판적으로 SLO로 쓰는 것.
3. **미해결 문제를 인지**하고, 새 작업이 그 문제와 어떻게 관련되는지 명시한다.
4. **점진적 개선**을 지향한다. 한 번에 완전히 새로 짜는 것보다 "다음 한 단계"가 명확한 제안을 한다.
5. **정당화를 요구**한다. 새로운 임계값, 기준, 설계 결정에는 항상 "왜?"에 대한 답을 문서화한다.

---

## 7. 참조 문서 (ai-team-lab)

### CAT 관련

| 문서 | 위치 | 용도 |
|------|------|------|
| CAT v1 명세 | `projects/sample-cl2-sli-slo-analysis/08-cat-v1-spec/cat-v1-specification.md` | CAT 수용 테스트 명세 |
| SLO 검증 분석 | `projects/sample-cl2-sli-slo-analysis/07-validation/slo-validation-analysis.md` | SLO 가설 검증 결과 |
| 레이턴시 변동성 분석 | `projects/sample-cl2-sli-slo-analysis/07-validation/latency-variance-analysis.md` | 변동 원인 및 정제 방향 |
| CAT 비전 | `knowledge/principles/cat-vision.md` | 목표와 방향 |
| CAT 설계 원칙 | `knowledge/principles/cat-design-principles.md` | 설계 원칙 |
| SLI/SLO 철학 | `knowledge/principles/sli-slo-philosophy.md` | SLO 정당화 철학 |
| devcat 프로그램 브리프 | `knowledge/principles/devcat-program-brief.md` | 현재 상태와 방향 |

### Monitoring 관련

| 문서 | 위치 | 용도 |
|------|------|------|
| Cluster Health Model | `projects/sre-monitoring-cluster-health-model/06-documentation/final-report.md` | 모니터링 모델 |
| Dashboard Design | `projects/sre-monitoring-dashboard-design/07-documentation/central-kubernetes-operational-dashboard-design.md` | 대시보드 최종 설계 |
| Operational Confidence Theory | `projects/sre-monitoring-dashboard-design/03-architecture/operational-confidence-theory.md` | 운영 이론 |
| Executive Summary | `projects/sre-monitoring-dashboard-design/07-documentation/executive-summary.md` | 경영진 요약 |
| Project Closeout | `projects/sre-monitoring-dashboard-design/07-documentation/project-closeout.md` | 프로젝트 마감 |

---

*ai-team-lab 컨텍스트 임포트. ai-team-c의 공유 지식으로 활용.*
