# Operational Confidence Framework for Kubernetes Clusters

**Role:** writer
**Date:** 2026-03-17
**Version:** v4 (v1–v3 + 검증·구현 가이드·Confidence Score·도입 단계)

---

## Executive Summary (경영진 요약)

### 문제

Prometheus와 Grafana로 수많은 metric을 수집하고 대시보드를 운영해도, 운영자는 여전히 **"그래서 클러스터가 괜찮은 건가요?"**라는 질문에 자신 있게 답하기 어렵습니다. metric은 넘쳐나지만 **운영 확신(Operational Confidence)**이 없습니다.

### 해결책

**Operational Confidence Framework**는 클러스터 안전을 **하나의 체인**으로 이해합니다:

```
[Pressure] → [Strain] → [Instability] → [Impact]
   압박         부담        불안정          영향
```

- **체인의 앞쪽**을 보면 **미래를 예측**합니다
- **체인의 뒤쪽**을 보면 **현재를 진단**합니다
- **어느 단계에 이상이 있는지**에 따라 **행동이 결정**됩니다

### 세 가지 질문에 대한 답

| 질문 | 답하는 방법 |
|------|-------------|
| "지금 안전한가?" | Instability + Impact 단계가 모두 정상이면 안전 |
| "내일도 안전한가?" | Pressure + Strain 단계에 경고가 없으면 안전 |
| "무엇을 해야 하나?" | 이상이 있는 단계에 따라 행동이 결정됨 |

### 기대 효과

- **명확한 판단**: "안전한가?"에 5초 내에 답할 수 있음
- **조기 대응**: 서비스 영향 전에 리스크를 감지하고 대응
- **일관된 기준**: 팀 전체가 동일한 기준으로 클러스터 상태를 판단
- **효율적 운영**: 일상 점검을 5분 내에 완료

---

## 1. 개요

### 1.1 이 프레임워크가 필요한 이유

기존 모니터링의 문제:

| 문제 | 설명 |
|------|------|
| **가시성은 있지만 확신이 없다** | metric은 많지만 "괜찮은 건가?"에 답하기 어려움 |
| **판단 기준이 없다** | 숫자는 있지만 "이게 정상인가?"를 모름 |
| **행동으로 이어지지 않는다** | 경고를 봐도 "뭘 해야 하나?"가 불명확 |

이 프레임워크가 제공하는 것:

| 제공 | 설명 |
|------|------|
| **명확한 판단 규칙** | "이 조건이면 안전, 저 조건이면 조사 필요" |
| **인과관계 이해** | 왜 이 신호가 중요한지, 다음에 무슨 일이 일어나는지 |
| **구체적 행동 연결** | 신호를 보고 바로 할 수 있는 행동 |

### 1.2 핵심 아이디어

> **시스템 문제는 한 번에 발생하지 않는다. 단계적으로 전파된다.**

이 통찰이 프레임워크의 핵심입니다. 문제가 단계적으로 전파된다면:

- **앞 단계**를 보면 **뒷 단계**를 예측할 수 있습니다
- **뒷 단계**에 이상이 없으면 **지금은 안전**합니다
- **어느 단계**에 이상이 있는지에 따라 **대응이 달라집니다**

---

## 2. 고장 전파 체인 (Failure Propagation Chain)

### 2.1 4단계 모델

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   Stage 1          Stage 2          Stage 3          Stage 4               │
│   PRESSURE    →    STRAIN     →    INSTABILITY  →    IMPACT               │
│                                                                             │
│   시스템에          시스템 동작에      실제 실패가       사용자/서비스       │
│   스트레스가        어려움이 생김      발생하기 시작      영향이 발생         │
│   가해짐                                                                    │
│                                                                             │
│   ──────────────────────────────────────────────────────────────────────   │
│   예측 가능         조기 경보          현재 상태          진단               │
│   (시간 여유 큼)    (시간 여유 중간)   (시간 여유 없음)   (이미 발생)         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Kubernetes에서의 각 단계

| Stage | 현상 | 예시 |
|-------|------|------|
| **Pressure** | 리소스에 스트레스가 가해짐 | 노드 CPU 90%, 메모리 85%, 디스크 여유 5% |
| **Strain** | 시스템 동작에 어려움이 생김 | Pending 파드 증가, 스케줄링 지연 |
| **Instability** | 실패가 발생하기 시작 | 파드 재시작 급증, NotReady 노드, OOM kill |
| **Impact** | 서비스에 영향 | Endpoints 비어 있음, 응답 지연, 에러율 상승 |

### 2.2.1 Strain 단계: 시스템 운영 마찰

Strain은 **실패가 발생하기 전의 시스템 운영 마찰(operational friction)**을 나타낸다. 분산 시스템에서 리소스 압박(Pressure)이 쌓이면, 먼저 "동작은 하지만 힘들어지는" 구간이 생긴다 — 스케줄링 지연, Pending 추세 증가 등. 아직 파드 실패나 노드 NotReady는 없지만, **그 방향으로 가고 있다**는 신호가 Strain이다.  
이 단계의 신호는 **추세·지연**에 초점을 둔다 (예: Pending **Rising**, Scheduling **Delay**). Pressure(리소스 상태)와 Instability(이미 발생한 실패) 사이를 구분해 두기 때문에, "내일도 안전한가?"에 **실패가 나타나기 전에** 사전 대응할 수 있다.

### 2.2.2 Impact 단계: 사용자 가시 영향

Impact는 **사용자가 체감하는 서비스 수준의 영향**을 다루는 체인의 최종 단계다. 인프라·워크로드 이상이 **요청 가용성, 지연, 에러율** 같은 **사용자 관점 결과**로 이어진 구간이다.  
**SLO와의 관계**: M1(Empty Endpoints)은 가용성 SLO 위반, M2(Service Degradation)는 지연/에러율 SLO 위반에 대응한다. 즉 Impact 신호는 인프라 메트릭이 아니라 **서비스 수준 결과**를 본다.  
**인프라 → 서비스 전환**: Instability까지는 노드/파드/스케줄러 등 **클러스터 리소스** 상태다. Impact는 **엔드포인트 가용성, 요청 성공률·지연**처럼 트래픽이 서비스를 통과한 **결과**를 본다. 따라서 Impact가 정상이면 "지금 안전하다"고 판단하고, 이상이 있으면 즉시 복구가 필요하다.

### 2.3 체인의 의미

```
Pressure       "리소스에 압박이 있다"
    ↓          시간이 지나면...
Strain         "시스템이 힘들어한다"
    ↓          방치하면...
Instability    "실패가 발생하기 시작한다"
    ↓          계속되면...
Impact         "서비스가 영향받는다"
```

**핵심 통찰**: 체인의 앞쪽에서 이상을 감지하면, 뒤쪽의 문제를 **예방**할 수 있습니다.

---

## 3. 최소 신호 집합 (Minimal Signal Set)

### 3.1 10개의 핵심 신호

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PRESSURE (3개)              │  STRAIN (2개)                                │
│  ───────────────────────────────────────────────────────────────────────── │
│  P1. Node CPU High (>80%)   │  S1. Pending Pods Rising (추세 증가)         │
│  P2. Node Memory High (>80%)│  S2. Scheduling Delay (스케줄러 지연)        │
│  P3. Node Disk Low (<20%)   │                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  INSTABILITY (3개)           │  IMPACT (2개)                                │
│  ───────────────────────────────────────────────────────────────────────── │
│  I1. Excessive Restarts     │  M1. Empty Endpoints (=0)                    │
│  I2. NotReady Nodes (>0)    │  M2. Service Degradation (SLO 위반)          │
│  I3. Pending Pods High (>0) │                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 최소 신호 선정 원칙 (Minimal Signal Selection Principle)

이 10개 신호는 아래 **선정 기준**을 만족하도록 선택되었다. 새 신호를 추가할 때도 동일 기준을 적용한다.

| 기준 | 설명 |
|------|------|
| **Stage 대표성 (representative of the stage)** | 해당 단계의 핵심 현상을 나타냄. Pressure=리소스 압박, Strain=운영 마찰, Instability=실패 발생, Impact=사용자 영향. |
| **직접 행동 가능 (directly actionable)** | 이상 시 Runbook 수준의 구체적 행동이 정의됨. "조사 필요"만으로 끝나지 않음. |
| **최소 중복 (minimal overlap)** | 같은 현상을 중복해서 보지 않음. S1(추세)과 I3(절대값)은 Pending의 서로 다른 관점으로 역할이 구분됨. |
| **대부분의 K8s 환경에서 관측 가능 (observable in most K8s environments)** | node_exporter, kube-state-metrics만으로 핵심 판단 가능. S2, M2는 환경에 따라 선택 보강. |

**왜 이 10개인가:** Pressure는 노드 CPU/메모리/디스크 세 가지가 서로 대체 불가해 각각 필요하다. Strain은 스케줄링 마찰을 추세(S1)와 지연(S2) 두 관점으로 보며, S2는 생략 가능하다. Instability는 워크로드 실패(재시작), 노드 실패(NotReady), 스케줄 실패 지속(Pending High)을 구분한다. Impact는 가용성(M1)과 품질(M2) 두 사용자 관점을 다룬다. 이 조합으로 세 가지 운영 질문에 **최소한의 신호만으로** 답할 수 있다.

---

## 4. 세 가지 운영 질문

### 4.1 "지금 클러스터가 안전한가?"

**답하는 방법:**

```
IF Stage 3 (Instability) 모든 신호 정상
   AND Stage 4 (Impact) 모든 신호 정상
THEN → "예, 지금 안전합니다"
ELSE → "아니오, 조사가 필요합니다"
```

**실무에서:**
- 대시보드에서 **I1, I2, I3, M1, M2**를 확인
- 모두 정상(녹색)이면 → **"지금 안전"**
- 하나라도 비정상(빨강)이면 → **"조사 필요"**

### 4.2 "내일도 안전할 것인가?"

**답하는 방법:**

```
IF Stage 1 (Pressure) 모든 신호 정상
   AND Stage 2 (Strain) 모든 신호 정상
THEN → "예, 당분간 안전할 것입니다"
ELSE → "아니오, 조기 리스크가 있습니다"
```

**실무에서:**
- 대시보드에서 **P1, P2, P3, S1, S2**를 확인
- 모두 정상이면 → **"당분간 안전"**
- 하나라도 경고(노랑)이면 → **"조기 리스크, 사전 대응 필요"**

### 4.3 "무엇을 해야 하는가?"

**답하는 방법:**

이상이 있는 **Stage**에 따라 행동이 결정됩니다:

| 이상 Stage | 긴급도 | 권장 행동 |
|-----------|--------|----------|
| **Pressure** | 낮음 | 용량 계획: 노드 추가, 워크로드 분산, 정리 |
| **Strain** | 중간 | 사전 대응: 리소스 확장, 우선순위 조정 |
| **Instability** | 높음 | 원인 조사: 로그 확인, 이벤트 분석, 롤백 검토 |
| **Impact** | 매우 높음 | 즉시 복구: 긴급 대응, 롤백, 트래픽 우회 |

---

## 5. 일상 운영에서의 사용

### 5.1 5분 점검 워크플로

```
Step 1 (30초): Stage 3-4 확인
              모두 정상? → Step 2로
              이상 있음? → 긴급 대응

Step 2 (30초): Stage 1-2 확인
              모두 정상? → 점검 완료 ✓
              경고 있음? → 사전 대응 계획

Step 3 (1분): 결과 기록
```

### 5.2 의사결정 플로차트

```
                    ┌─────────────────┐
                    │  대시보드 확인   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Impact 이상?   │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │ Yes                         │ No
              ▼                             ▼
     🔴🔴 즉시 복구                   ┌──────────────┐
                                     │ Instability  │
                                     │ 이상?        │
                                     └──────┬───────┘
                                            │
                          ┌─────────────────┴──────────────────┐
                          │ Yes                                │ No
                          ▼                                    ▼
                  🔴 원인 조사                          ┌──────────────┐
                                                       │ Pressure/    │
                                                       │ Strain 이상? │
                                                       └──────┬───────┘
                                                              │
                                        ┌─────────────────────┴───────────────┐
                                        │ Yes                                 │ No
                                        ▼                                     ▼
                                 🟡 사전 대응                          🟢 모두 정상
                                                                      점검 완료
```

---

## 6. 기존 설계 대비 개선점

### 6.1 ai-team-lab 모델과의 비교

| 측면 | ai-team-lab | 새 프레임워크 |
|------|-------------|---------------|
| **구조** | 3 Block + 5 조건 + 4 경로 | 1 체인 + 4 단계 |
| **인과관계** | 문서에만 서술 | 구조에 내장 |
| **설명 시간** | 10분 이상 | 5분 이내 |
| **행동 연결** | 추상적 ("조사 필요") | 구체적 (신호별 행동) |

### 6.2 유지된 좋은 아이디어

- ✅ 운영 확신(Operational Confidence) 개념
- ✅ "지금 안전?" / "곧 위험?" 시간 관점
- ✅ 선행/후행 지표 구분
- ✅ 최소 신호 원칙

---

## 7. 체인 검증 및 한계

### 7.1 "이 체인이 실제로 맞는가?"

체인 모델이 실제로 동작하는지 검증하는 방법:

**과거 인시던트 분석:**
1. 과거 서비스 영향 사례를 수집
2. 영향 발생 **이전**에 Pressure, Strain 신호가 있었는지 확인
3. 체인이 예측력을 가지는지 검증

**예상 결과:**
- 대부분의 인시던트에서 **Impact 전에 Instability**가 있었음
- 많은 경우 **Instability 전에 Strain**이 있었음
- 일부 경우 **Strain 전에 Pressure**가 있었음

### 7.2 체인 모델의 한계: 체인을 따르지 않는 실패

고장 전파 체인은 **리소스 압박이 단계적으로 전파되는 유형**을 모델링한다. 아래 유형은 체인 앞단 없이 **갑자기** 나타나거나, 체인으로 설명하기 어렵다.

| 유형 | 예시 | 왜 체인과 맞지 않는가 | 권장 대응 |
|------|------|------------------------|-----------|
| **급성 하드웨어/네트워크 장애** | 노드 갑작스런 다운, 네트워크 파티션 | Pressure/Strain 없이 Instability 또는 Impact로 바로 나타남 | 노드/네트워크 알람을 체인과 **별도**로 유지. 체인은 예측·판단용, 급성 장애는 즉시 복구 플로우로 처리. |
| **배포/설정 오류** | 잘못된 이미지, 리소스 limit, 설정맵 | 인프라 압박이 아니라 **변경**으로 인한 실패. 체인 단계 순서와 무관. | **변경 관리·배포 모니터링**을 별도로 두고, Instability/Impact와 연계해 "최근 변경" 확인. |
| **외부 의존성 장애** | 외부 API 타임아웃, 외부 DB 장애 | 클러스터 내부 Pressure/Strain과 무관하게 서비스 품질 악화. | **의존성·연동 모니터링**을 Impact 또는 별도 레이어로 두고, "클러스터 원인 vs 외부 원인" 구분에 활용. |

### 7.3 다른 모니터링과의 공존

이 프레임워크는 전체 모니터링을 **대체하지 않고**, "운영 확신"을 위한 **최소 판단 축**을 제공한다.

- **체인 기반 판단**: "지금/내일 안전한가?", "무엇을 해야 하나?"는 이 체인과 최소 신호 집합으로 답한다.
- **급성 장애·설정 오류·외부 의존성**은 위 표와 같이 **별도 관점**으로 모니터링하고, Runbook에서 "Impact만 있고 Instability는 없음 → 최근 배포/외부 의존성 확인"처럼 **체인 결과와 조합**해 원인 후보를 좁힌다.
- **세부 진단·성능 튜닝**은 체인 밖의 메트릭(프로파일링, 트레이스, 로그)으로 수행한다. 체인은 결론과 행동에 초점을 두고, 깊은 조사는 기존 도구와 절차를 유지한다.

**체인 = 리소스 전파 기반의 예측·판단**, **그 외 = 급성·설정·외부·심층 조사**로 역할을 나누면, 프레임워크가 단순성을 유지하면서 다양한 실패 유형과 공존할 수 있다.

---

## 8. 향후 고려사항

### 8.1 확장 가능한 영역

| 영역 | 설명 |
|------|------|
| **Alert 정책** | 각 신호에 대한 알림 규칙 정의 |
| **Runbook 상세화** | 신호별 더 구체적인 대응 절차 |
| **대시보드 템플릿** | Grafana JSON 파일 제공 |
| **자동화** | 일부 대응의 자동화 |

### 8.2 새 신호 추가 시 기준

새 신호를 추가할 때는 **최소 신호 선정 원칙**(3.2절)의 네 가지 기준을 충족하는지 확인한 뒤:

1. **어느 Stage에 속하는가?** — 체인에서의 위치 결정
2. **기존 신호와 중복되지 않는가?** — 최소 중복 유지
3. **구체적 행동으로 연결되는가?** — 행동 없는 신호는 추가하지 않음

---

## 9. 관련 자료

| 자료 | 위치 |
|------|------|
| **신호 상세 명세** | `04-engineering/signal-specification.md` |
| **운영 Runbook** | `04-engineering/operational-runbook.md` |
| **Incident Replay Analysis** | `04-engineering/incident-replay-analysis.md` |
| **아키텍처 상세** | `03-architecture.md` |
| **조사 배경** | `02-research.md` |
| **ai-team-lab 컨텍스트** | `knowledge/imported-context/ai-team-lab-context.md` |

---

## 10. Framework v2: 운영 계층 확장

v1의 체인(Pressure → Strain → Instability → Impact)은 개념적으로 올바르다. v2는 이 체인을 **실제 Kubernetes 플랫폼의 운영 계층**에 매핑하고, **계층 간 전파**와 **레이어별 최소 대표 신호**, **실제 운영 사용 패턴**을 정의하여, 외부 발표(컨퍼런스·블로그·논문)와 실무에 동시에 쓸 수 있게 한다. v1의 최소성·행동 연결·5분 설명 원칙은 그대로 유지한다.

### 10.1 Operational Signal Topology (운영 신호 위상)

실제 플랫폼에서 신호가 나오는 **시스템 계층**은 다음과 같다. 고장은 보통 **아래 계층 → 위 계층**으로 전파된다.

| 계층 | 관찰 대상 | v1 체인과의 대응 |
|------|-----------|------------------|
| **Infrastructure Layer** | 노드 리소스(CPU, 메모리, 디스크), 네트워크 용량 | Pressure |
| **Cluster Control Layer** | Scheduler, controller, API server 부하·지연 | Strain |
| **Workload Layer** | Pod, Deployment, 재시작·스케일링·리소스 단편화 | Strain(추세) + Instability |
| **Traffic Layer** | 서비스 트래픽, Ingress/Gateway, 지연·백로그 | Instability 말기 ~ Impact 직전 |
| **Service Reliability Layer** | SLO 위반, Endpoint 가용성, 에러율 | Impact |
| **Platform Topology Layer** | 멀티클러스터/환경 간 설정·버전 drift | 모든 단계에 주입 가능(교차 계층) |

**계층 간 관계:** Infrastructure 압박이 쌓이면 Control에서 스케줄링·API 지연(Strain)이 생기고, 이어서 Workload에서 실패·Pending(Instability), Traffic에서 지연·에러, 마지막으로 Service Reliability에서 SLO 위반(Impact)으로 이어진다. Platform Topology는 수평 관점으로, 여러 클러스터/환경 간 **설정·버전 불일치**가 특정 계층에 Strain/Instability를 만들 수 있다.

### 10.2 계층 간 고장 전파

개념 체인을 계층별로 쓰면:

```
Resource pressure (Infrastructure)
    → Scheduling/API strain (Cluster Control)
    → Workload instability (Workload)
    → Traffic degradation (Traffic)
    → Service impact (Service Reliability)
```

**전파 경로 예:**  
- **경로 A**: Node CPU/Memory high → Pending rising, Scheduler delay → Restarts, Pending high → Gateway latency/5xx 증가 → Empty Endpoints, SLO 위반.  
- **경로 B**: API server 지연(과다 요청) → 새 파드 스케줄 지연 → 요청 타임아웃 증가 → 에러율 상승.  
- **Platform Topology 주입**: 클러스터 간 `resource.limits` 불일치로 한쪽에서만 OOM·Restart → Workload Instability가 설정 drift로 발생. 이때 Infrastructure는 정상이므로 **설정 일관성**을 점검한다.

### 10.3 확장 신호 모델 (레이어별 대표 신호)

v1의 10개 신호는 **핵심으로 유지**한다. v2에서는 이들을 계층에 매핑하고, **레이어당 최소한의 대표 신호**만 선택적으로 추가한다.

**v1 신호의 계층 매핑:**  
Infrastructure: P1, P2, P3 | Control: S1, S2 | Workload: I1, I2, I3 | Service Reliability: M1, M2. Traffic·Platform Topology에는 v1에 명시적 신호가 없으므로, 필요 시 아래를 **선택**으로 추가한다.

| 계층 | 추가 신호 (선택) | 의미 | 적용 조건 |
|------|------------------|------|-----------|
| **Traffic** | T1 Gateway/Ingress health | Ingress/Gateway 에러율·지연 급증 | Ingress/Gateway 사용 시. 없으면 M2만 사용. |
| **Platform Topology** | PL1 Config/version drift | 멀티클러스터 또는 Git vs 클러스터 간 중요 리소스·버전 불일치 | 멀티클러스터·다환경 운영 시. |

Workload에서 파드 수준 압박을 더 보려면 **W1 Pod resource pressure**(선택)를 둘 수 있으나, 기본 판단은 v1 10개로 한다. **원칙:** 필수는 v1 10개; T1, PL1, W1은 환경에 따라 추가하며, 추가 시에도 직접 행동 가능·Stage 대표성·최소 중복을 만족해야 한다.

### 10.4 실제 운영 사용

**일상 클러스터 헬스 체크**  
1) "지금 안전한가?" → I1, I2, I3, M1, M2 확인. 2) "내일도 안전한가?" → P1–P3, S1–S2 확인. 3) v2 레이어 뷰: 같은 신호를 Infrastructure → Control → Workload → Traffic → Service Reliability 순으로 훑어 어느 계층에서 먼저 경고가 나오는지 확인. 4) 멀티클러스터/다환경이면 PL1(drift) 주기 확인. → 5분 이내에 결론을 내고, 필요 시 해당 계층 Runbook으로 드릴다운.

**인시던트 조기 감지**  
Strain 계층(S1 Pending rising, S2 Scheduling delay)을 먼저 본다. 여기서 경고가 나오면 Instability/Impact 전에 사전 대응(리소스 확장, 우선순위 조정). Traffic 계층(T1 또는 M2 추이)이 나쁘면 Gateway/메시/업스트림을 점검하여 Impact 전에 대응한다.

**용량 계획**  
Pressure + Strain을 계층별로 추이 본다. Infrastructure(P1–P3) 또는 Control(S1–S2)이 꾸준히 악화되면 "곧 한계"로 해석. 어느 계층이 먼저 한계에 다다르는지에 따라 노드 증설, 스케줄러/API 조정, 워크로드 스케일/리소스 조정 중 무엇을 할지 결정한다.

**멀티클러스터/설정 drift 감지**  
Platform Topology(PL1)를 주기 실행하여 클러스터 간·Git vs 클러스터 간 중요 리소스 불일치를 확인. "동일 서비스인데 한 클러스터만 이상"일 때 설정 일관성을 먼저 점검한다.

---

## 11. Framework v3: 개념 기반 강화 및 업계 프레임워크화

v3는 이 프레임워크를 **컨퍼런스·블로그·연구** 수준의 업계 프레임워크로 정리한다. 기존 모니터링·Observability와의 **위치**, **실제 인시던트 전파 사례**, **참조 대시보드 모델**, **운영자 멘탈 모델**을 명시하여, 왜 이 프레임워크가 가치 있는지 설명할 수 있게 한다. 5분 설명·최소 신호 원칙은 유지한다.

### 11.1 기존 모니터링 모델 대비 위치 (Framework Positioning)

**Monitoring / Observability / Operational Confidence**는 목표가 다르다.

| 관점 | 핵심 목표 | 한계(이 프레임워크가 채우는 것) |
|------|-----------|--------------------------------|
| **Traditional Monitoring** | 메트릭·로그 **수집** | 수집만으로는 "그래서 괜찮은가?"에 답하지 못함. |
| **Observability** | 시스템 동작 **설명** (explain behavior) | 설명은 사후에 유용; **사전 예측·의사결정** 구조는 아님. |
| **Operational Confidence** | **운영 의사결정** 가능하게 함 | 최소 신호 + 체인 + 행동 매핑으로 즉시 결론과 행동. |

**개념적 비교:**

```
Monitoring                  →  collecting metrics
Observability               →  explaining system behavior
Operational Confidence      →  enabling operational decisions
```

**이 프레임워크의 고유 가치:** (1) **판단 규칙 내장** — Stage·체인에 "이 조건이면 안전/조사/대응"이 붙어 있음. (2) **시간 축 명시** — 앞쪽 Stage로 예측, 뒤쪽으로 진단. (3) **행동 연결** — 각 Stage·신호가 Runbook 수준 행동으로 이어짐. Monitoring은 입력, Observability는 심층 분석, Operational Confidence는 **운영 결론과 행동**을 담당하는 별도 레이어다.

### 11.2 인시던트 전파 사례 (Incident Propagation Examples)

체인이 **실제 Kubernetes 장애**에서 어떻게 조기 감지에 기여하는지 보여 주는 사례다.

**사례 1: 노드 메모리 압박 → 스케줄링 지연 → 파드 재시작 루프 → 게이트웨이 지연 → SLO 위반**

| 단계 | Stage | 신호 | 조기 대응 가능 시점 |
|------|-------|------|----------------------|
| 1 | Pressure | P2 Node Memory High | **여기서** 노드 추가·워크로드 분산 시 Impact 예방 |
| 2 | Strain | S1 Pending Rising, S2 Scheduling Delay | 리소스 확장·우선순위 조정으로 Instability 전 대응 |
| 3 | Instability | I1 Restarts, I3 Pending High | 이미 실패 발생; 원인 조사·롤백 |
| 4 | Impact | M1/M2 | 즉시 복구 |

**교훈:** Pressure 또는 Strain에서 경고를 잡으면 Instability·Impact **전에** 대응할 수 있다.

**사례 2: 디스크 압박 → Eviction·재시작 → 일시적 5xx**  
P3 Node Disk Low → (Strain) 파드 생성·시작 지연 → I1 Restarts(Eviction) → M2 일시적 5xx. P3만 주기적으로 봐도 "곧 Eviction·5xx"를 **예측**할 수 있다.

**사례 3: API server 부하 → 스케줄링 지연 → Pending 증가**  
노드 여유 있는데 S2 Scheduling Delay, S1 Pending Rising → I3 Pending High. Strain만 올라와도 "지금은 Impact 없지만 곧 문제"로 판단하고, **조기 Stage에 반응**하면 Impact를 줄이거나 막을 수 있다.

이런 **인시던트 전파 사례**를 팀·과거 사고와 매핑하면, 체인 모델이 조기 감지에 도움이 됨을 검증·설명할 수 있다.

### 11.3 참조 대시보드 모델 (Reference Dashboard Model)

**Operational Confidence Dashboard**는 Stage·레이어·드릴다운이 한눈에 보이는 **개념 레이아웃**을 따른다. Grafana JSON이 아니라 **구조**를 정의한다.

1. **Stage 건강 상태 (Stage Health)**  
   Pressure / Strain / Instability / Impact 네 Stage별 **정상·경고·위험**을 최상단에 표시. 5초 안에 "지금 안전한가?", "내일 안전한가?"에 답할 수 있어야 함.

2. **Operational Signal Topology 레이어**  
   Infrastructure → Control → Workload → Traffic → Service Reliability (및 선택적 Platform Topology) 순으로 한 행/섹션씩 배치. 각 레이어에는 해당 대표 신호만 표시. 체인 방향(전파 방향)과 시각적 방향을 맞춤.

3. **드릴다운 경로 (Drill-down Path)**  
   Stage 또는 레이어 클릭 → 해당 상세 패널. 신호 클릭 → TOP N(노드/파드 등) + 권장 행동 한 줄. 각 경고에서 Runbook 또는 행동 요약으로 연결.

4. **5분 원칙**  
   첫 화면만으로 Stage 건강 + "무엇을 해야 하나?"가 결정 가능. 상세 메트릭·트레이스·로그는 드릴다운 후에만 노출.

구현은 Grafana 등으로 팀·환경에 맞게 하면 된다.

### 11.4 운영자 멘탈 모델 (Operator Mental Model)

운영자가 이 프레임워크로 **생각하는 방식**을 표준화하면 채택과 일관된 판단이 쉬워진다.

- **앞쪽 Stage로 예측 (Predict using early stages)**  
  Pressure, Strain을 "미래 리스크" 선행 지표로 본다. "앞쪽이 나쁘면, 뒤쪽을 막기 위해 움직인다."

- **뒤쪽 Stage로 진단 (Diagnose using later stages)**  
  Instability, Impact로 "지금 안전한가?"를 **진단**한다. "뒤쪽이 나쁘면, 지금 무슨 일이 일어나고 있는지 결론 내리고 행동한다."

- **Stage에 따라 행동 우선순위 (Prioritize actions based on stage)**  
  Impact → 즉시 복구. Instability → 원인 조사·롤백 검토. Strain → 사전 대응. Pressure → 용량 계획. "가장 뒤쪽에서 나쁜 Stage가 오늘의 우선순위를 정한다."

- **레이어로 원인 위치 좁히기 (v2 확장)**  
  "어느 Stage가 나쁜가?"에 더해 "어느 레이어(Infrastructure/Control/Workload/Traffic)에서 먼저 나쁜가?"를 보면 원인 위치를 빠르게 좁힌다. "Stage는 언제/얼마나 심한가, 레이어는 어디서를 답한다."

이 **멘탈 모델**을 팀 표준으로 두면, 5분 설명으로도 실제 운영에서 일관되게 사용할 수 있다.

---

## 12. Framework v4: 검증 및 구현 가이드

v4는 프레임워크를 **검증 가능하고 구현 가능**하게 만든다. **인시던트 분석을 통한 검증**, **참조 신호 구현**(Prometheus 메트릭 예), **Operational Confidence Score**(선택), **도입 단계(Deployment Model)**를 추가한다. 5분 설명·최소 신호 원칙은 유지한다.

### 12.1 인시던트 분석을 통한 프레임워크 검증

프레임워크가 **실제 시스템 동작**에 기반함을 보이려면, 인시던트를 **체인 단계별로 재구성**하고, 각 단계가 **어떤 메트릭에 나타나는지** 매핑한다.

**예: Node memory pressure → scheduler delay → pod restart loops → gateway latency spike → SLO violation**

| 체인 단계 | 메트릭에서의 관측 (대표 예) |
|-----------|-----------------------------|
| **Pressure** | `node_memory_MemAvailable_bytes`/`MemTotal_bytes` 비율 저하; Impact **이전** 수십 분~수 시간 전에 상승. |
| **Strain** | `kube_pod_status_phase{phase="Pending"}` 추세 증가; `scheduler_pending_pods` 또는 스케줄 레이턴시. Pressure **이후**, Instability **이전**에 나타남. |
| **Instability** | `kube_pod_container_status_restarts_total` increase 급증; Pending 절대값 임계값 초과. |
| **Impact** | Ingress/Gateway 지연·5xx; 또는 `kube_endpoint_address_available == 0`. SLO 위반 시점과 일치. |

**검증 절차:** Impact 시점을 기준으로 **이전** 메트릭 시계열을 확인하여, Pressure → Strain → Instability → Impact 순서로 각 단계가 메트릭에 존재하는지 기록. 여러 인시던트에서 반복되면 체인이 실제 동작을 잘 묘사한다는 **경험적 근거**가 된다.

### 12.2 참조 신호 구현 (Reference Signal Implementation)

최소 신호를 **어떤 Prometheus 메트릭으로 구현할 수 있는지** 대표 예만 제시한다. 전체 대시보드/AlertRule은 정의하지 않는다.

| 신호 | Prometheus 메트릭 예시 | 소스 |
|------|------------------------|------|
| **P1 Node CPU High** | `node_cpu_seconds_total` (idle 제외 후 사용률). 100 - avg(rate(…{mode="idle"}[5m]))*100 > 80 | node_exporter |
| **P2 Node Memory High** | `node_memory_MemAvailable_bytes`, `node_memory_MemTotal_bytes`. 100*(1 - MemAvailable/MemTotal) > 80 | node_exporter |
| **P3 Node Disk Low** | `node_filesystem_avail_bytes`/`node_filesystem_size_bytes` (mountpoint="/"). 여유 비율 < 20% | node_exporter |
| **S1 Pending Rising** | `kube_pod_status_phase{phase="Pending"}==1` count; 15분 increase/delta로 추세 | kube-state-metrics |
| **S2 Scheduling Delay** | `scheduler_pending_pods` 또는 `scheduler_e2e_scheduling_duration_seconds_*` | kube-scheduler (Managed K8s에서는 없을 수 있음) |
| **I1 Excessive Restarts** | `kube_pod_container_status_restarts_total` 의 sum(increase(...[10m])) > N | kube-state-metrics |
| **I2 NotReady Nodes** | `kube_node_status_condition{condition="Ready",status!="true"}` count > 0 | kube-state-metrics |
| **I3 Pending High** | `kube_pod_status_phase{phase="Pending"}==1` count > 0 (또는 > 5) | kube-state-metrics |
| **M1 Empty Endpoints** | `kube_endpoint_address_available` (또는 `kube_endpoints_address_available`) == 0. critical 서비스 필터 | kube-state-metrics |
| **M2 Service Degradation** | Ingress/Gateway 지연·5xx rate. SLO 임계값 초과 | Ingress/Gateway |

메트릭 이름·레이블은 환경에 따라 다를 수 있음. Managed K8s에서는 S2 생략, M2는 트래픽 관측점이 있을 때만 구현 가능.

### 12.3 Operational Confidence Score (선택 개념)

Stage 건강을 **단일 수준**으로 요약하는 **선택** 개념이다. 단순하게 유지한다.

- **High**: 모든 Stage 정상. 지금·당분간 안전.
- **Medium**: Pressure 또는 Strain 경고. Instability·Impact 정상. 조기 리스크, 사전 대응 권장.
- **Low**: Instability 이상. 원인 조사 필요.
- **Critical**: Impact 이상. 즉시 복구 필요.

**권장:** 가장 나쁜 Stage 하나로 위 네 구간을 결정. 복잡한 점수 공식은 피하고, 대시보드 상단 요약·알림 정책에 활용. 필수 구성요소는 아님.

### 12.4 도입 단계 (Deployment Model)

팀이 프레임워크를 **단계적으로 도입**하는 순서다.

| 단계 | 내용 |
|------|------|
| **Step 1: 메트릭 매핑** | 기존 Prometheus 메트릭을 v1 10개 신호(및 v2 선택 신호)에 매핑. 없으면 node_exporter, kube-state-metrics 보강. → 신호–메트릭 매핑표. |
| **Step 2: Stage 대시보드** | v3 참조 대시보드 모델대로 Stage Health + 레이어 + 드릴다운 구축. → Operational Confidence Dashboard 초안. |
| **Step 3: Runbook** | Stage/신호별 “이상 시 무엇을 할 것인가” Runbook. 기존 Runbook에 Stage 링크. |
| **Step 4: 멘탈 모델 채택** | v3 멘탈 모델을 팀 표준으로 하고, 일상 점검·인시던트 시 Stage/레이어 우선 확인 절차 공유. → 팀 가이드 1페이지. |

한 번에 전부 하지 말고 Step 1부터 순서대로 적용. Step 1만으로도 “우리 메트릭이 프레임워크의 어디에 해당하는가”가 명확해지고, Step 2부터 “지금/내일 안전한가?”를 5초 안에 답할 수 있다.

---

## 한 장 요약

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│              OPERATIONAL CONFIDENCE FRAMEWORK                               │
│              for Kubernetes Clusters                                        │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  핵심: "시스템 문제는 단계적으로 전파된다"                                     │
│                                                                             │
│    [Pressure]  ───→  [Strain]  ───→  [Instability]  ───→  [Impact]         │
│       압박            부담              불안정             영향              │
│                                                                             │
│    예측 ◄────────────────────────────────────────────────────► 진단         │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  세 가지 질문:                                                               │
│                                                                             │
│    "지금 안전?"      →  Stage 3-4 모두 정상인가?                             │
│    "내일 안전?"      →  Stage 1-2 경고 없는가?                               │
│    "무엇을 해야?"    →  이상 Stage에 따라 행동                               │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  행동 원칙:                                                                  │
│                                                                             │
│    Pressure 경고    →  용량 계획 (시간 여유)                                 │
│    Strain 경고      →  사전 대응 (곧 문제)                                   │
│    Instability 경고 →  원인 조사 (진행 중)                                   │
│    Impact 경고      →  즉시 복구 (이미 발생)                                 │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  v2: 운영 계층  v3: 위치·인시던트·대시보드·멘탈 모델                         │
│  v4: 인시던트 검증, 참조 신호(Prometheus), Confidence Score, 도입 단계       │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

*Operational Confidence Framework v4. 2026-03-17.*
