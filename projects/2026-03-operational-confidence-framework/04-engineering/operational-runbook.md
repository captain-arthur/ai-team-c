# Operational Runbook

**Role:** engineer
**Date:** 2026-03-17

이 문서는 Operational Confidence Framework를 사용한 **일상 운영 가이드**입니다.

---

## 일상 점검 워크플로 (Daily Check)

### 5분 점검 절차

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Step 1: 현재 안전 확인 (30초)                                               │
│  ─────────────────────────────────                                          │
│  대시보드에서 Stage 3 (Instability) + Stage 4 (Impact) 확인                  │
│                                                                             │
│  모두 🟢 → "지금 안전합니다" → Step 2로                                      │
│  하나라도 🔴 → "조사 필요" → 긴급 대응 절차로                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  Step 2: 미래 리스크 확인 (30초)                                             │
│  ─────────────────────────────────                                          │
│  대시보드에서 Stage 1 (Pressure) + Stage 2 (Strain) 확인                     │
│                                                                             │
│  모두 🟢 → "내일도 안전할 것입니다" → 점검 완료                               │
│  하나라도 🟡 → "조기 리스크" → 사전 대응 절차로                                │
├─────────────────────────────────────────────────────────────────────────────┤
│  Step 3: 기록 (1분)                                                         │
│  ────────────────                                                           │
│  점검 결과와 발견 사항 기록                                                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 상황별 대응 절차

### 상황 A: 모든 신호 정상 🟢

```
결론: 클러스터가 현재 안전하고, 가까운 미래에도 안전할 것입니다.

행동:
1. 점검 완료 기록
2. 다음 정기 점검까지 대기
```

---

### 상황 B: Stage 1-2 경고, Stage 3-4 정상 🟡

```
결론: 지금은 안전하지만, 조기 리스크가 있습니다.

긴급도: 중간 (시간 여유 있음)

대응 절차:
```

#### B1. Pressure 경고 (P1/P2/P3)

| 신호 | 대응 |
|------|------|
| **P1. Node CPU High** | 1. CPU TOP10 노드 확인 → 2. 해당 노드의 워크로드 확인 → 3. 워크로드 분산 또는 리소스 조정 → 4. 필요시 노드 추가 계획 |
| **P2. Node Memory High** | 1. Memory TOP10 노드 확인 → 2. 메모리 소비 워크로드 식별 → 3. 메모리 limit 조정 또는 누수 확인 → 4. 필요시 노드 추가 계획 |
| **P3. Node Disk Low** | 1. 디스크 사용 노드 확인 → 2. 로그/이미지 정리 (`docker system prune`, 로그 로테이션) → 3. 필요시 디스크 확장 |

#### B2. Strain 경고 (S1/S2)

| 신호 | 대응 |
|------|------|
| **S1. Pending Rising** | 1. Pending 파드 namespace 확인 → 2. Pending 원인 분석 (리소스? 노드 선택? PVC?) → 3. 낮은 우선순위 파드 정리 또는 스케일 조정 |
| **S2. Scheduling Delay** | 1. 스케줄러 상태 확인 → 2. Control plane 부하 확인 → 3. 대량 파드 생성 작업 연기 검토 |

---

### 상황 C: Stage 3 경고, Stage 4 정상 🔴

```
결론: 문제가 진행 중이지만 아직 서비스 영향은 없습니다.

긴급도: 높음 (즉시 조사)

대응 절차:
```

#### C1. Instability 경고 (I1/I2/I3)

| 신호 | 대응 |
|------|------|
| **I1. Excessive Restarts** | 1. Restart TOP10 파드 확인 → 2. `kubectl logs <pod> --previous` 로 원인 확인 → 3. OOM이면 limit 조정, CrashLoop이면 애플리케이션 수정/롤백 |
| **I2. NotReady Nodes** | 1. `kubectl describe node <node>` → 2. kubelet 상태 확인 → 3. 노드 drain 후 조사 또는 replace |
| **I3. Pending High** | 1. `kubectl describe pod <pending-pod>` → 2. Events에서 원인 확인 (리소스 부족? affinity? taint?) → 3. 원인에 따라 리소스 확장, 제약 완화, 노드 추가 |

---

### 상황 D: Stage 4 경고 🔴🔴

```
결론: 서비스가 이미 영향받고 있습니다.

긴급도: 매우 높음 (즉시 복구)

대응 절차:
```

#### D1. Impact 경고 (M1/M2)

| 신호 | 대응 |
|------|------|
| **M1. Empty Endpoints** | 1. 해당 서비스의 파드 상태 즉시 확인 → 2. 파드가 없으면 긴급 스케일업 → 3. 파드가 실패 중이면 롤백 검토 → 4. 필요시 트래픽 우회 |
| **M2. Service Degradation** | 1. 에러 원인 파악 (백엔드? 네트워크? 설정?) → 2. 백엔드 문제면 Stage 3 신호 확인 후 해당 대응 → 3. 필요시 롤백 또는 트래픽 분산 |

---

## 의사결정 플로차트

```
                            ┌─────────────────────┐
                            │   대시보드 확인      │
                            └──────────┬──────────┘
                                       │
                            ┌──────────▼──────────┐
                            │  Stage 4 (Impact)   │
                            │  이상 있음?          │
                            └──────────┬──────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    │ Yes              │                  │ No
                    ▼                  │                  ▼
          ┌─────────────────┐          │        ┌─────────────────┐
          │ 🔴🔴 즉시 복구   │          │        │  Stage 3 이상?  │
          │ (상황 D)        │          │        └────────┬────────┘
          └─────────────────┘          │                 │
                                       │    ┌────────────┼────────────┐
                                       │    │ Yes        │            │ No
                                       │    ▼            │            ▼
                                       │  ┌─────────────┐│  ┌─────────────────┐
                                       │  │ 🔴 원인 조사 ││  │  Stage 1-2 이상? │
                                       │  │ (상황 C)    ││  └────────┬────────┘
                                       │  └─────────────┘│           │
                                       │                 │  ┌────────┼────────┐
                                       │                 │  │ Yes    │        │ No
                                       │                 │  ▼        │        ▼
                                       │                 │ ┌─────────┐│ ┌─────────────┐
                                       │                 │ │🟡 사전   ││ │🟢 모두 정상  │
                                       │                 │ │대응     ││ │점검 완료     │
                                       │                 │ │(상황 B) ││ │(상황 A)     │
                                       │                 │ └─────────┘│ └─────────────┘
                                       └─────────────────┴────────────┴───────────────
```

---

## 유용한 명령어

### 진단 명령어

```bash
# NotReady 노드 확인
kubectl get nodes | grep -v Ready

# NotReady 노드 상세
kubectl describe node <node-name>

# Pending 파드 확인
kubectl get pods --all-namespaces --field-selector=status.phase=Pending

# Pending 원인 확인
kubectl describe pod <pod-name> -n <namespace>

# 재시작 많은 파드 찾기
kubectl get pods --all-namespaces --sort-by='.status.containerStatuses[0].restartCount' | tail -10

# 파드 이전 로그 (재시작 원인)
kubectl logs <pod-name> -n <namespace> --previous

# 노드 리소스 사용량
kubectl top nodes

# 파드 리소스 사용량
kubectl top pods --all-namespaces --sort-by=memory
```

### 대응 명령어

```bash
# 노드 drain (워크로드 이동)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# 디플로이먼트 롤백
kubectl rollout undo deployment/<deployment-name> -n <namespace>

# 긴급 스케일업
kubectl scale deployment/<deployment-name> -n <namespace> --replicas=<N>

# 파드 강제 삭제 (stuck 상태)
kubectl delete pod <pod-name> -n <namespace> --force --grace-period=0
```

---

## 에스컬레이션 기준

| 상황 | 에스컬레이션 대상 |
|------|------------------|
| Stage 4 경고 지속 (5분 이상) | 팀 리드 + 온콜 담당자 |
| Stage 3 경고 복수 동시 발생 | 팀 리드 |
| Stage 1-2 경고 지속 (1시간 이상) | 용량 계획 담당자 |
| 원인 파악 불가 | 시니어 엔지니어 |

---

## 점검 기록 템플릿

```markdown
## 일상 점검 기록

**날짜:** YYYY-MM-DD HH:MM
**점검자:**

### 결과 요약

- [ ] Stage 1 (Pressure): 🟢 정상 / 🟡 경고
- [ ] Stage 2 (Strain): 🟢 정상 / 🟡 경고
- [ ] Stage 3 (Instability): 🟢 정상 / 🔴 경고
- [ ] Stage 4 (Impact): 🟢 정상 / 🔴 경고

### 발견 사항

(있는 경우 기록)

### 조치 사항

(있는 경우 기록)
```
