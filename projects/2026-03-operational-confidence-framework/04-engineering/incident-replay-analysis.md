# Incident Replay Analysis

**Purpose:** Validate the Operational Confidence Framework using realistic Kubernetes failure scenarios.  
**Principle:** Use only the existing framework signals (P1–P3, S1–S2, I1–I3, M1–M2). No new signals are introduced.

---

## Overview

This document reconstructs five production-style incidents and maps each step to the framework stages (Pressure → Strain → Instability → Impact). For each incident we show:

- **Failure timeline** — when each phase occurred.
- **Stage mapping** — which framework stage corresponds to each step.
- **First signal that would have appeared** — enabling earlier detection.
- **How the framework could have detected the incident earlier** — using Pressure/Strain as leading indicators.

---

## Incident 1: Node Memory Exhaustion

**Scenario:** Workload memory usage grows (e.g., leak or traffic spike). Node available memory drops; kubelet starts evicting pods and OOM kills occur. New pods cannot be scheduled; endpoints drop and SLO is violated.

### Failure timeline and stage mapping

| Timeline (relative) | Stage        | Observed signal(s)        | Recommended action |
|---------------------|-------------|----------------------------|---------------------|
| T-45min             | **Pressure**  | **P2** Node Memory High    | Capacity planning: add nodes, rebalance workloads, or adjust memory limits. |
| T-20min             | **Strain**    | **S1** Pending Pods Rising; **S2** Scheduling Delay (if available) | Preemptive action: free memory, scale down non-critical workloads, or add nodes. |
| T-5min              | **Instability** | **I1** Excessive Restarts (OOM/eviction); **I3** Pending Pods High | Investigate: identify top memory consumers, check logs, consider rollback or limit increase. |
| T0                  | **Impact**     | **M1** Empty Endpoints; **M2** Service Degradation | Immediate recovery: scale up, rollback, or traffic failover. |

**First signal that would have appeared:** **P2** (Node Memory High) at T-45min.

**How the framework could have detected the incident earlier:** Monitoring P2 with an 80% (warning) threshold gives tens of minutes of lead time before Instability. Acting on P2 or on S1/S2 when they first appear would allow node scaling or workload rebalancing before OOM kills and endpoint loss.

---

## Incident 2: Disk Pressure / Image GC Failure

**Scenario:** Node disk fills (logs, images, or ephemeral storage). Image pull and container start slow down; kubelet evicts pods or fails to start new ones. Image garbage collection cannot free enough space. Pods restart or stay Pending; brief 5xx or deployment delays occur.

### Failure timeline and stage mapping

| Timeline (relative) | Stage        | Observed signal(s)        | Recommended action |
|---------------------|-------------|----------------------------|---------------------|
| T-2h to T-30min     | **Pressure**  | **P3** Node Disk Low      | Log/image cleanup, disk expansion, or log rotation. |
| T-25min             | **Strain**    | **S1** Pending Pods Rising (slow image pull / container start) | Reduce pod churn, clean up images on affected nodes. |
| T-10min             | **Instability** | **I1** Excessive Restarts (eviction); **I2** NotReady Nodes (if kubelet is impacted) | Investigate node disk usage; drain and clean or replace node. |
| T0                  | **Impact**     | **M2** Service Degradation (brief 5xx or latency) | Immediate recovery: restore endpoints, then fix disk. |

**First signal that would have appeared:** **P3** (Node Disk Low) at T-2h to T-30min.

**How the framework could have detected the incident earlier:** P3 (e.g., &lt;20% free) appears long before evictions and 5xx. Acting on P3 allows log rotation, image pruning, or disk expansion before workload instability and user impact.

---

## Incident 3: Scheduler Overload

**Scenario:** API server or scheduler is overloaded (e.g., many concurrent deployments or list/watch load). Scheduling latency increases; new pods stay Pending. Deployments appear stuck; eventually timeouts or partial availability affect users.

### Failure timeline and stage mapping

| Timeline (relative) | Stage        | Observed signal(s)        | Recommended action |
|---------------------|-------------|----------------------------|---------------------|
| T-30min             | **Strain**    | **S1** Pending Pods Rising; **S2** Scheduling Delay (if control plane metrics available) | Reduce deployment churn; scale control plane or add nodes to reduce API load. |
| T-15min             | **Instability** | **I3** Pending Pods High  | Investigate: check scheduler/API metrics, events; prioritize critical workloads. |
| T0                  | **Impact**     | **M2** Service Degradation (timeouts, partial unavailability) | Immediate recovery: scale up existing pods if possible, reduce rollout pace. |

**Note:** In this scenario there may be no strong **Pressure** signal on worker nodes (P1–P3 normal). Strain appears first at the control plane / scheduling layer. If control plane runs on dedicated nodes, **P1** (Node CPU High) on those nodes could appear earlier and be mapped as Pressure.

**First signal that would have appeared:** **S1** (Pending Pods Rising) or **S2** (Scheduling Delay) at T-30min.

**How the framework could have detected the incident earlier:** Watching S1 (trend of Pending count) and S2 (when available) gives lead time before I3 and M2. Responding to Strain (e.g., pausing non-critical rollouts, checking API/scheduler health) can prevent Pending buildup and user-facing impact.

---

## Incident 4: Pod Restart Storm

**Scenario:** A bad deployment or dependency change causes many pods to crash in a loop (e.g., CrashLoopBackOff or OOM on a new image). Restart count spikes; if enough replicas fail, endpoints drop and SLO is violated.

### Failure timeline and stage mapping

| Timeline (relative) | Stage        | Observed signal(s)        | Recommended action |
|---------------------|-------------|----------------------------|---------------------|
| T-20min             | **Instability** | **I1** Excessive Restarts | Investigate: identify top restarted workloads, check logs (e.g. `kubectl logs --previous`), consider rollback. |
| T-5min              | **Instability** | **I3** Pending Pods High (if new replicas cannot start) | Scale or rollback; fix image/config. |
| T0                  | **Impact**     | **M1** Empty Endpoints; **M2** Service Degradation | Immediate recovery: rollback deployment, scale up, or traffic failover. |

**Note:** A restart storm can be **application- or deployment-induced** (e.g., bug, wrong config) without prior node Pressure or Strain. In that case the **first observable framework signal is I1**. The framework still correctly places the incident: Instability (I1, I3) → Impact (M1, M2). If the root cause is resource-related (e.g., new image uses more memory), **P2** could appear before I1 and would be the first signal.

**First signal that would have appeared:** **I1** (Excessive Restarts) at T-20min (or **P2** if the cause is memory pressure).

**How the framework could have detected the incident earlier:** If the cause is resource pressure, P2 (or P1) would give earlier warning. For pure app/deploy faults, the framework does not invent signals; it still correctly identifies Instability (I1) as the stage to act on before Impact (M1/M2). Acting on I1 as soon as it fires (investigation and rollback) minimizes time to recovery.

---

## Incident 5: Ingress Gateway Overload

**Scenario:** Traffic spike or a few slow backends cause the ingress gateway (e.g., NGINX, Envoy) to queue requests or exhaust connections. Latency and 5xx increase; users see errors or timeouts.

### Failure timeline and stage mapping

| Timeline (relative) | Stage        | Observed signal(s)        | Recommended action |
|---------------------|-------------|----------------------------|---------------------|
| T-15min             | **Pressure**  | **P1** Node CPU High (on nodes running ingress pods) | Scale ingress replicas or add nodes; optimize gateway config. |
| T-10min             | **Strain**    | **S1** Pending Pods Rising (if scaling gateway and new pods are slow to schedule) | Optional: reduce load or scale gateway; check scheduler. |
| T-5min              | **Instability** | **I1** Excessive Restarts (if gateway pods OOM or crash under load) | Investigate gateway pods; scale or tune resources. |
| T0                  | **Impact**     | **M2** Service Degradation (latency spike, 5xx increase) | Immediate recovery: scale gateway, tune timeouts, or shed load. |

**First signal that would have appeared:** **P1** (Node CPU High) on gateway nodes at T-15min, or **M2** (Service Degradation) if traffic spike is sudden and gateway is the first place latency/5xx are observed.

**How the framework could have detected the incident earlier:** If gateway pods run on dedicated or identifiable nodes, P1 on those nodes is a leading indicator before M2. The framework maps P1 → Pressure and M2 → Impact; acting on P1 (or on S1/I1 if scaling causes Pending or restarts) allows scaling or tuning before user-visible SLO violation.

---

## Summary: First signals and lead time

| Incident                    | First signal(s) that would appear     | Typical lead time before Impact |
|----------------------------|---------------------------------------|----------------------------------|
| Node memory exhaustion     | **P2** Node Memory High               | Tens of minutes                  |
| Disk pressure / image GC   | **P3** Node Disk Low                  | Tens of minutes to hours         |
| Scheduler overload         | **S1** Pending Rising, **S2** Scheduling Delay | Minutes to tens of minutes |
| Pod restart storm          | **I1** Excessive Restarts (or **P2** if resource-related) | Minutes before Impact |
| Ingress gateway overload  | **P1** Node CPU High (gateway nodes) or **M2** | Minutes to tens of minutes |

---

## Conclusion

- The framework **correctly explains** these failures by mapping them to Pressure → Strain → Instability → Impact.
- **No new signals** were introduced; only P1–P3, S1–S2, I1–I3, M1–M2 were used.
- **Earlier detection** is achieved when operators watch Pressure (P1–P3) and Strain (S1–S2) and act before Instability/Impact. For app-induced restart storms, Instability (I1) is the first actionable signal and the framework still guides the right response (investigate, rollback) before Impact.
- This replay analysis supports using the framework as a **validation and training** asset: real incidents can be reconstructed into the same stage/signal/action tables to reinforce the operator mental model and tune alerts.
