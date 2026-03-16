# Signal Specification

**Role:** engineer
**Date:** 2026-03-17

이 문서는 Operational Confidence Framework의 **10개 신호**에 대한 상세 정의를 제공합니다.

---

## 신호 개요

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Stage 1: Pressure     │  Stage 2: Strain       │  Stage 3: Instability    │
├─────────────────────────────────────────────────────────────────────────────┤
│  P1. Node CPU High     │  S1. Pending Rising    │  I1. Excessive Restarts  │
│  P2. Node Memory High  │  S2. Scheduling Delay  │  I2. NotReady Nodes      │
│  P3. Node Disk Low     │                        │  I3. Pending High        │
├─────────────────────────────────────────────────────────────────────────────┤
│                        Stage 4: Impact                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                        M1. Empty Endpoints                                  │
│                        M2. Service Degradation                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Stage 1: Pressure (압박)

### P1. Node CPU High

| 항목 | 내용 |
|------|------|
| **ID** | P1 |
| **이름** | Node CPU High |
| **설명** | 노드 CPU 사용률이 임계값을 초과 |
| **의미** | 리소스 압박 시작. 방치하면 스케줄링 어려움, 성능 저하로 이어짐 |
| **Threshold** | > 80% (Warning), > 90% (Critical) |
| **Lead Time** | 높음 (분~시간) |

**PromQL:**
```promql
# 노드별 CPU 사용률
100 - (avg by (node) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# 임계값 초과 노드 수
count(
  (100 - (avg by (node) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)) > 80
)
```

**필요 metric:** `node_cpu_seconds_total` (node_exporter)

---

### P2. Node Memory High

| 항목 | 내용 |
|------|------|
| **ID** | P2 |
| **이름** | Node Memory High |
| **설명** | 노드 메모리 사용률이 임계값을 초과 |
| **의미** | 메모리 압박. 방치하면 OOM kill, eviction으로 이어짐 |
| **Threshold** | > 80% (Warning), > 90% (Critical) |
| **Lead Time** | 높음 (분~시간) |

**PromQL:**
```promql
# 노드별 메모리 사용률
100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))

# 임계값 초과 노드 수
count(
  (100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))) > 80
)
```

**필요 metric:** `node_memory_MemAvailable_bytes`, `node_memory_MemTotal_bytes` (node_exporter)

---

### P3. Node Disk Low

| 항목 | 내용 |
|------|------|
| **ID** | P3 |
| **이름** | Node Disk Low |
| **설명** | 노드 디스크 여유 공간이 임계값 미만 |
| **의미** | 디스크 압박. 방치하면 이미지 풀 실패, 로그 쓰기 실패로 이어짐 |
| **Threshold** | < 20% (Warning), < 10% (Critical) |
| **Lead Time** | 높음 (시간~일) |

**PromQL:**
```promql
# 노드별 디스크 여유율 (root 파티션)
100 * (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})

# 임계값 미만 노드 수
count(
  (100 * (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) < 20
)
```

**필요 metric:** `node_filesystem_avail_bytes`, `node_filesystem_size_bytes` (node_exporter)

**환경별 조정:** `mountpoint`가 "/"가 아닌 환경에서는 "/var" 등으로 변경 필요

---

## Stage 2: Strain (부담)

### S1. Pending Pods Rising

| 항목 | 내용 |
|------|------|
| **ID** | S1 |
| **이름** | Pending Pods Rising |
| **설명** | Pending 상태 파드 수가 증가하는 추세 |
| **의미** | 스케줄링에 어려움이 생기기 시작. 리소스 부족 또는 제약 조건 문제 |
| **Threshold** | 15분간 증가량 > 5 (Warning), > 10 (Critical) |
| **Lead Time** | 중간 (분) |

**PromQL:**
```promql
# 현재 Pending 파드 수
count(kube_pod_status_phase{phase="Pending"} == 1)

# 15분간 증가량
increase(
  count(kube_pod_status_phase{phase="Pending"} == 1)[15m:]
)

# 또는 delta 사용 (부호 있는 변화량)
delta(
  count(kube_pod_status_phase{phase="Pending"} == 1)[15m]
)
```

**필요 metric:** `kube_pod_status_phase` (kube-state-metrics)

---

### S2. Scheduling Delay

| 항목 | 내용 |
|------|------|
| **ID** | S2 |
| **이름** | Scheduling Delay |
| **설명** | 스케줄러의 파드 처리 지연 |
| **의미** | Control plane에 부담. 스케줄링 백로그 증가 |
| **Threshold** | scheduler_pending_pods > 0 (Warning), > 10 (Critical) |
| **Lead Time** | 중간 (분) |

**PromQL:**
```promql
# 스케줄러 pending 파드
scheduler_pending_pods

# 또는 스케줄링 레이턴시 P99
histogram_quantile(0.99, sum(rate(scheduler_e2e_scheduling_duration_seconds_bucket[5m])) by (le))
```

**필요 metric:** `scheduler_pending_pods`, `scheduler_e2e_scheduling_duration_seconds_bucket` (kube-scheduler)

**참고:** Managed Kubernetes(EKS, GKE, AKS)에서는 스케줄러 metric을 스크래핑할 수 없을 수 있음. 이 경우 S1만 사용.

---

## Stage 3: Instability (불안정)

### I1. Excessive Restarts

| 항목 | 내용 |
|------|------|
| **ID** | I1 |
| **이름** | Excessive Restarts |
| **설명** | 파드 재시작 횟수가 급증 |
| **의미** | 워크로드 불안정. OOM kill, CrashLoopBackOff, 애플리케이션 오류 등 |
| **Threshold** | 10분간 재시작 > N (팀에서 정의, 예: 10~20) |
| **Lead Time** | 낮음 (이미 발생) |

**PromQL:**
```promql
# 10분간 전체 재시작 수
sum(increase(kube_pod_container_status_restarts_total[10m]))

# 재시작 TOP10 파드
topk(10, increase(kube_pod_container_status_restarts_total[10m]))
```

**필요 metric:** `kube_pod_container_status_restarts_total` (kube-state-metrics)

**팀 정의 필요:** threshold N은 클러스터 규모와 워크로드 특성에 따라 팀에서 정의해야 함.

---

### I2. NotReady Nodes

| 항목 | 내용 |
|------|------|
| **ID** | I2 |
| **이름** | NotReady Nodes |
| **설명** | NotReady 상태인 노드 존재 |
| **의미** | 노드 실패 또는 심각한 문제. 스케줄 가능 용량 감소 |
| **Threshold** | > 0 (Critical) |
| **Lead Time** | 매우 낮음 (이미 발생) |

**PromQL:**
```promql
# NotReady 노드 수
count(kube_node_status_condition{condition="Ready", status="false"} == 1)

# 또는 True가 아닌 노드
count(kube_node_status_condition{condition="Ready", status!="true"} == 1)
```

**필요 metric:** `kube_node_status_condition` (kube-state-metrics)

---

### I3. Pending Pods High

| 항목 | 내용 |
|------|------|
| **ID** | I3 |
| **이름** | Pending Pods High |
| **설명** | Pending 상태 파드 수가 임계값 초과 |
| **의미** | 스케줄 실패 지속. 워크로드가 정상 동작하지 못함 |
| **Threshold** | > 0 (Warning), > 5 (Critical) |
| **Lead Time** | 낮음 (이미 발생) |

**PromQL:**
```promql
# Pending 파드 수
count(kube_pod_status_phase{phase="Pending"} == 1)

# namespace별 Pending 파드
count by (namespace) (kube_pod_status_phase{phase="Pending"} == 1)
```

**필요 metric:** `kube_pod_status_phase` (kube-state-metrics)

**S1과의 차이:** S1은 **추세**(증가 중인가?), I3은 **현재 상태**(지금 몇 개인가?)

---

## Stage 4: Impact (영향)

### M1. Empty Endpoints

| 항목 | 내용 |
|------|------|
| **ID** | M1 |
| **이름** | Empty Endpoints |
| **설명** | 핵심 서비스의 Endpoints가 비어 있음 |
| **의미** | 트래픽이 백엔드에 도달 불가. 서비스 다운 |
| **Threshold** | = 0 for critical services (Critical) |
| **Lead Time** | 없음 (이미 영향 발생) |

**PromQL:**
```promql
# 모든 서비스의 available endpoints
kube_endpoint_address_available

# endpoints가 0인 서비스 (복수형 metric 이름 환경)
kube_endpoints_address_available == 0

# 특정 namespace의 critical 서비스 필터링 (예시)
kube_endpoints_address_available{namespace="production"} == 0
```

**필요 metric:** `kube_endpoint_address_available` 또는 `kube_endpoints_address_available` (kube-state-metrics 버전에 따라 다름)

**팀 정의 필요:** "핵심 서비스"의 namespace/label 필터는 팀에서 정의해야 함.

---

### M2. Service Degradation

| 항목 | 내용 |
|------|------|
| **ID** | M2 |
| **이름** | Service Degradation |
| **설명** | 서비스 응답 지연 또는 에러율 상승 |
| **의미** | 사용자 영향 발생 중 |
| **Threshold** | 에러율 > 1%, 지연 > SLO (팀 정의) |
| **Lead Time** | 없음 (이미 영향 발생) |

**PromQL (Ingress 기반 예시):**
```promql
# Ingress 에러율
sum(rate(nginx_ingress_controller_requests{status=~"5.."}[5m]))
/ sum(rate(nginx_ingress_controller_requests[5m]))

# Ingress 지연 P99
histogram_quantile(0.99, sum(rate(nginx_ingress_controller_request_duration_seconds_bucket[5m])) by (le, ingress))
```

**필요 metric:** Ingress controller metric (환경에 따라 다름)

**참고:** 이 신호는 Ingress controller 또는 서비스 메시가 설치된 환경에서만 사용 가능. 없는 경우 M1만 사용.

---

## 신호 요약표

| ID | 이름 | Stage | Threshold | Lead Time | 필수 metric |
|----|------|-------|-----------|-----------|------------|
| P1 | Node CPU High | Pressure | >80% | 높음 | node_exporter |
| P2 | Node Memory High | Pressure | >80% | 높음 | node_exporter |
| P3 | Node Disk Low | Pressure | <20% | 높음 | node_exporter |
| S1 | Pending Rising | Strain | 추세 증가 | 중간 | kube-state-metrics |
| S2 | Scheduling Delay | Strain | >0 | 중간 | kube-scheduler (optional) |
| I1 | Excessive Restarts | Instability | >N (팀 정의) | 낮음 | kube-state-metrics |
| I2 | NotReady Nodes | Instability | >0 | 매우 낮음 | kube-state-metrics |
| I3 | Pending High | Instability | >0 | 낮음 | kube-state-metrics |
| M1 | Empty Endpoints | Impact | =0 | 없음 | kube-state-metrics |
| M2 | Service Degradation | Impact | SLO 위반 | 없음 | Ingress (optional) |

---

## Managed Kubernetes 환경 (EKS, GKE, AKS)

Control plane metric을 스크래핑할 수 없는 환경에서는:

- **S2 (Scheduling Delay)** — 사용 불가. S1만 사용.
- **나머지 신호** — 모두 사용 가능.

Control plane 건강은 **클라우드 제공자의 콘솔/알림**으로 확인.

---

## 필요 구성요소 요약

| 구성요소 | 역할 | 필수 여부 |
|---------|------|----------|
| **node_exporter** | 노드 리소스 metric (P1, P2, P3) | 필수 |
| **kube-state-metrics** | Kubernetes 객체 상태 metric (S1, I1, I2, I3, M1) | 필수 |
| **kube-scheduler metric** | 스케줄러 상태 (S2) | 선택 (Managed K8s에서는 없을 수 있음) |
| **Ingress controller metric** | 서비스 에러율/지연 (M2) | 선택 |
