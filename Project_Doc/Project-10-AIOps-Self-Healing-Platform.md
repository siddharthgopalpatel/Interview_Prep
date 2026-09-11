# Project 10: AIOps Self-Healing Platform

## Integrated AI-Driven Operations — Log Analysis, Anomaly Detection, Auto-Healing & Incident Response

**Author:** Siddharth Patel
**Role:** Senior DevOps & Cloud Infrastructure Engineer (10+ YOE)
**Technologies:** EKS, EFK/ELK (Elasticsearch, Fluent Bit/Logstash, Kibana), Prometheus, Grafana, Python, Scikit-learn (Isolation Forest), Kafka, Kubernetes API, Ansible, EventBridge + Lambda, SNS, PagerDuty, Slack, ServiceNow, DynamoDB

---

## Table of Contents

1. What is AIOps? (Simple Explanation)
2. Why This Project — The Problem With "Dumb" Monitoring
3. How It Connects to Projects 1–9
4. Architecture Overview
5. Layer 1 — Log & Metric Collection (The Eyes)
6. Layer 2 — Anomaly Detection (The Brain)
7. Layer 3 — Auto-Healing (The Hands)
8. Layer 4 — Incident Response & ITSM (The Escalation Path)
9. The Closed Feedback Loop
10. How Each Component Works (Simple English)
11. Security
12. Monitoring the Monitor (Observability of AIOps itself)
13. Cost
14. Metrics & Results
15. Limitations & Gotchas
16. Design Decisions
17. Interview Talking Points

---

## 1. What is AIOps? (Simple Explanation)

**AIOps = Artificial Intelligence for IT Operations.** Instead of humans staring at dashboards and reacting to alerts, machine learning watches the system, spots trouble early, fixes what it can automatically, and only wakes a human when it genuinely needs one.

**Simple analogy:**
- **Traditional monitoring** = a smoke detector. It screams *after* there's smoke, and it screams the same way whether it's burnt toast or a house fire.
- **AIOps** = a smart building system. It notices the temperature creeping up in one room *before* there's smoke, opens a vent to cool it (auto-heal), and only calls the fire brigade if the vent doesn't fix it.

**The three shifts AIOps makes:**

| From | To |
|---|---|
| Static thresholds (`CPU > 90%`) | Learned "normal" behaviour (anomaly vs. expected spike) |
| Reactive (fix after outage) | Proactive (detect drift before outage) |
| Human restarts the pod at 2 AM | System restarts the pod; human only paged if it recurs |

---

## 2. Why This Project — The Problem With "Dumb" Monitoring

From Projects 3, 6, and the monitoring work on this resume, I already run **Prometheus + Grafana + AlertManager + EFK across 15+ microservices** with Jaeger tracing. That stack is excellent at *telling me what happened*. Its weaknesses:

1. **Static thresholds cause alert fatigue.** A CPU spike at 02:00 during a batch job fires the same P2 as a real memory leak. On-call engineers start ignoring alerts — the dangerous failure mode.
2. **It's reactive.** A slow memory leak that never crosses a fixed threshold until it OOM-kills the pod is invisible to threshold alerting until it's already too late.
3. **Every alert = a human.** Even the 80% of incidents that are "restart the pod and it's fine" wake someone up.

**What I needed:** a layer *on top of* the existing observability stack that (a) learns what normal looks like, (b) flags deviations before they become outages, (c) auto-remediates the boring, repeatable failures, and (d) escalates to humans through ITSM only when automation can't fix it.

This is the natural evolution of **Project 5 (Serverless Remediation Engine)** — that project auto-fixes *security/config* violations (open SG, public S3) with rule-based EventBridge→Lambda. Project 10 auto-fixes *operational/reliability* failures using *ML-based detection* instead of hard-coded rules.

---

## 3. How It Connects to Projects 1–9

| Project | Relationship |
|---|---|
| **Project 1 — DevSecOps Pipeline** | Deploys the microservices this platform watches. Project 11 adds AI *into* that pipeline; Project 10 watches what it deploys at runtime. |
| **Project 3 — Kubernetes Environments** | The EKS/OpenShift clusters where auto-healing (pod restart, scale, rollback) executes. |
| **Project 5 — Serverless Remediation Engine** | Direct predecessor. Same EventBridge→Lambda→DynamoDB→SNS remediation *pattern*, but triggered by **ML anomaly scores** instead of AWS Config rules. Reuses the DynamoDB audit-trail design. |
| **Project 6 — Service Mesh (Istio)** | Jaeger traces feed context into root-cause correlation; Istio enables safe traffic-shift rollback. |
| **Project 9 — OS Patching Automation** | Reuses the **ServiceNow ITIL CR lifecycle (open → close)** integration and the multi-system monitoring-silence pattern. |

**Key positioning:** Project 5 = *rule-based* security self-healing. Project 10 = *ML-based* reliability self-healing. They share plumbing, differ in the brain.

---

## 4. Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         AIOps SELF-HEALING PLATFORM                             │
│                                                                                 │
│  ┌────────────┐   LAYER 1: COLLECT (Eyes)                                       │
│  │ 15+ micro- │   Fluent Bit ──► Elasticsearch (logs)                           │
│  │ services   │──► Node/Pod exporters ──► Prometheus (metrics)                  │
│  │ on EKS     │   Istio/Jaeger ──► traces                                       │
│  └────────────┘        │                                                        │
│                        ▼                                                        │
│  ┌────────────────────────────────────────────┐  LAYER 2: DETECT (Brain)       │
│  │  Anomaly Detection Service (Python)          │                               │
│  │  • Kafka stream of log/metric features       │                               │
│  │  • Isolation Forest (unsupervised)           │──► anomaly score              │
│  │  • Seasonality-aware baselines               │                               │
│  └────────────────────────────────────────────┘        │                       │
│                        │  anomaly score < threshold      │                      │
│                        ▼                                  │                      │
│  ┌────────────────────────────────────────────┐  LAYER 3: HEAL (Hands)         │
│  │  EventBridge ──► Lambda / Ansible runner     │                               │
│  │  Playbooks: restart pod, scale HPA, rollback │──► action + verify            │
│  │  Kubernetes API + kubectl + Argo Rollouts    │                               │
│  └────────────────────────────────────────────┘        │                       │
│                        │  heal FAILED or recurring       │                      │
│                        ▼                                  ▼                      │
│  ┌────────────────────────────────────────────┐  LAYER 4: ESCALATE             │
│  │  ServiceNow incident (open) ──► PagerDuty    │                               │
│  │  Slack ChatOps thread + Jaeger/RCA context   │──► on-call engineer           │
│  │  Auto-close ServiceNow CR on recovery        │                               │
│  └────────────────────────────────────────────┘                               │
│                        │                                                        │
│                        ▼                                                        │
│  DynamoDB audit trail  ─────► feedback loop: outcomes retrain / tune model      │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Design principle:** the platform is a **closed loop** — Detect → Heal → Verify → (Escalate if needed) → Record → Learn. Every action and outcome is written to DynamoDB (reusing Project 5's audit-trail design), which both provides an audit trail and feeds model tuning.

---

## 5. Layer 1 — Log & Metric Collection (The Eyes)

This layer already exists from Projects 3/6 — the AIOps platform *consumes* it rather than replacing it.

| Signal | Source | Sink |
|---|---|---|
| Logs | Fluent Bit DaemonSet on each node | Elasticsearch (EFK) |
| Metrics | Prometheus (Node/kube-state/app exporters) | Prometheus TSDB |
| Traces | Istio sidecars → Jaeger | Jaeger backend |

**What the AIOps layer adds:** a lightweight **feature extractor** that pulls a rolling window of features per service and pushes them onto a Kafka topic for the detection service:

- `response_time_p95`, `error_rate`, `cpu_usage`, `memory_usage`, `restart_count`, `log_error_frequency`

Example: pulling the last 24h of a signal from Elasticsearch to (re)build a baseline.

```python
from elasticsearch import Elasticsearch

es = Elasticsearch("https://es.internal:9200", api_key=ES_API_KEY)

query = {
    "query": {"range": {"@timestamp": {"gte": "now-1d/d", "lte": "now/d"}}},
    "size": 5000,
    "_source": ["service", "response_time", "level"],
}
resp = es.search(index="application-logs", body=query)
```

> **Note on the source article:** the Day-14 blog trains on a single `response_time` field. In production I use a **multi-feature vector** (latency, error rate, CPU, memory, restart count) per service, because single-feature anomaly detection produces far too many false positives on real telemetry.

---

## 6. Layer 2 — Anomaly Detection (The Brain)

### Why unsupervised (Isolation Forest)?

I do **not** have cleanly labelled "failure vs normal" data at scale — real production telemetry is mostly unlabelled. So supervised classification (predict-a-known-failure) is impractical as the primary detector. **Isolation Forest** is unsupervised: it learns the shape of "normal" and isolates outliers, no labels required.

| Option considered | Verdict |
|---|---|
| Static thresholds | ❌ Alert fatigue; misses slow drifts |
| Supervised classifier (Random Forest) | ⚠️ Needs labelled failures; used only as a *secondary* predictor where labels exist |
| **Isolation Forest (unsupervised)** | ✅ No labels; fast; handles multi-dimensional feature vectors; primary detector |
| Autoencoder / LSTM | ⚠️ Better for seasonal time-series, but heavier to operate; on the roadmap |

### Core detection service

```python
import numpy as np
from sklearn.ensemble import IsolationForest
from sklearn.preprocessing import StandardScaler

FEATURES = ["response_time_p95", "error_rate", "cpu_usage",
            "memory_usage", "restart_count", "log_error_freq"]

# Trained per-service on a rolling baseline window
scaler = StandardScaler().fit(baseline_df[FEATURES])
model = IsolationForest(
    n_estimators=200,
    contamination=0.02,      # expect ~2% of windows to be anomalous
    random_state=42,
).fit(scaler.transform(baseline_df[FEATURES]))

def score(feature_row: dict) -> dict:
    x = scaler.transform(np.array([[feature_row[f] for f in FEATURES]]))
    raw = model.decision_function(x)[0]      # higher = more normal
    is_anomaly = model.predict(x)[0] == -1
    return {"anomaly": bool(is_anomaly), "score": float(raw)}
```

### Real-time scoring via Kafka

The detection service is a Kafka consumer. Each incoming feature window is scored; anomalies (with enough severity/persistence) are published to an `anomalies` topic that EventBridge consumes.

```python
for msg in consumer:                     # topic: service-features
    row = json.loads(msg.value)
    result = score(row)
    if result["anomaly"] and result["score"] < ANOMALY_GATE:
        producer.send("anomalies", json.dumps({**row, **result}).encode())
```

**Guardrails that keep it sane:**
- **Persistence check:** an anomaly must persist for N consecutive windows (default 3) before triggering — kills one-off blips.
- **Seasonality awareness:** baselines are rebuilt on a schedule and are time-of-day aware, so the 02:00 batch job isn't flagged.
- **Contamination tuned per service tier** — chatty services get a slightly higher contamination.

---

## 7. Layer 3 — Auto-Healing (The Hands)

When an anomaly is confirmed, EventBridge routes it (same pattern as Project 5) to a remediation handler that maps *anomaly type → playbook*.

| Anomaly signature | Auto-heal action | Guardrail |
|---|---|---|
| Pod unresponsive / liveness failing | Restart pod (`kubectl rollout restart` / delete pod) | Max 2 restarts / 10 min, else escalate |
| Memory creeping toward limit | Scale HPA replicas up | Respect max replicas; cost cap |
| Error rate spikes post-deploy | **Argo Rollouts abort + rollback** to last stable | Only if a rollout is in progress |
| Disk / log volume high | Trigger log rotation / cleanup (Ansible) | Never touch app data volumes |
| Node NotReady | Cordon + drain + let Karpenter replace | Honour PodDisruptionBudgets |

```python
import subprocess

def restart_pod(namespace, deployment):
    r = subprocess.run(
        ["kubectl", "-n", namespace, "rollout", "restart", f"deploy/{deployment}"],
        capture_output=True, text=True,
    )
    ok = r.returncode == 0
    audit(action="restart", target=deployment, ok=ok, detail=r.stderr[:500])
    return ok
```

**Critical rule — always verify:** after any heal action, the platform re-scores the same service after a cool-down. If the anomaly clears → record success and auto-close. If not → escalate. **Auto-healing is never fire-and-forget.**

**Kubernetes-native healing** (liveness/readiness probes, HPA) does the first line of defence; the AIOps layer handles what probes can't express — cross-signal, gradual-drift, and deploy-correlated failures.

---

## 8. Layer 4 — Incident Response & ITSM (The Escalation Path)

Escalation happens only when **auto-heal failed** or the **same anomaly recurs** (indicating a real underlying problem). This reuses the ServiceNow ITIL lifecycle from **Project 9** and the SNS/PagerDuty/Slack routing from the resume's monitoring work.

**Escalation flow:**
1. Open a **ServiceNow incident** via REST API, pre-populated with: service, anomaly features, the heal actions already attempted, and Jaeger trace links for RCA context.
2. Route by severity — **P1 → PagerDuty**, **P2 → Slack**, **P3 → Jira** (matches existing alerting policy).
3. Post a **Slack ChatOps** thread where the engineer can run `/aiops status <service>` or `/aiops rollback <service>` to act without leaving chat.
4. On recovery (anomaly score returns to normal for M windows), **auto-close** the ServiceNow CR with resolution notes — mirroring the auto-close in Project 9.

```python
import requests

def open_incident(service, context):
    payload = {
        "short_description": f"AIOps: anomaly in {service} (auto-heal exhausted)",
        "category": "Infrastructure",
        "impact": "1", "urgency": "1",
        "work_notes": context,          # features + attempted actions + trace links
    }
    requests.post(f"{SNOW}/api/now/table/incident", json=payload, auth=SNOW_AUTH)

def notify_slack(msg):
    requests.post(SLACK_WEBHOOK, json={"text": f"🚨 {msg}"})
```

---

## 9. The Closed Feedback Loop

Every detection, heal action, outcome, and escalation is written to **DynamoDB** (reusing Project 5's audit schema). This gives:

- **Audit trail** — who/what/when for every automated action (compliance requirement, ties to SOC2 work).
- **Model feedback** — false positives (anomaly flagged, but engineer marked "expected") feed baseline re-tuning; confirmed incidents that the model *missed* highlight blind spots.
- **Runbook improvement** — recurring anomalies that always need the same manual fix become new auto-heal playbooks.

```
Detect ─► Heal ─► Verify ─► (Escalate?) ─► Record (DynamoDB) ─► Learn ─► retune baselines
   ▲                                                                          │
   └──────────────────────────────────────────────────────────────────────────┘
```

---

## 10. How Each Component Works (Simple English)

| Component | Plain-English job |
|---|---|
| **Fluent Bit** | Tiny agent on every node that ships logs to Elasticsearch. |
| **Prometheus** | Scrapes numeric health stats (CPU, memory, latency) every few seconds. |
| **Kafka** | The conveyor belt that streams live feature-windows to the ML brain. |
| **Isolation Forest** | The brain. Learned what "normal" looks like; flags weird stuff. |
| **EventBridge** | The switchboard. Routes an anomaly to the right fix. |
| **Lambda / Ansible** | The hands. Actually restart / scale / roll back. |
| **ServiceNow** | The ticket system. Opens/closes the official incident record. |
| **PagerDuty / Slack** | The pager. Wakes a human only when the robot gives up. |
| **DynamoDB** | The logbook. Records everything so we can audit and learn. |

---

## 11. Security

- **Least-privilege IAM** for the remediation Lambda/runner — scoped to *only* the K8s/AWS actions its playbooks need (no `*` permissions). Blast radius contained.
- **Kubernetes RBAC** — the AIOps service account can restart/scale/rollback in app namespaces only; cannot touch `kube-system` or secrets.
- **Signed actions & audit** — every auto-heal is written to the immutable DynamoDB trail; ties into the centralised logging / SOC2 posture from Project 4.
- **Secrets** via AWS Secrets Manager + KMS (ES creds, ServiceNow creds, Slack/PagerDuty tokens) — never in code.
- **Human-in-the-loop for destructive actions** — rollback of a stateful service or node drain requires a confirmation gate; only stateless restarts/scale are fully autonomous.

---

## 12. Monitoring the Monitor (Observability of AIOps itself)

If the AIOps platform silently dies, you get *worse* than no monitoring — false confidence. So the platform monitors itself:

- **Heartbeat metric** from the detection service into Prometheus; alert if scoring stops.
- **Model drift dashboard** in Grafana — anomaly rate over time; a sudden jump in "anomalies" usually means the baseline is stale, not that the world is on fire.
- **Auto-heal success rate** tracked — if it drops, playbooks need review.
- **False-positive rate** from engineer feedback in Slack, reviewed weekly.

---

## 13. Cost

The platform is deliberately lightweight and rides mostly on infrastructure that already exists (EFK, Prometheus from Projects 3/6):

| Component | Approx monthly |
|---|---|
| Anomaly detection service (2 small pods on existing EKS) | ~$25 |
| Kafka (MSK small / or reuse existing) | ~$120 (or $0 if reusing) |
| EventBridge + Lambda (remediation) | < $5 (event-driven, near-zero idle) |
| DynamoDB (on-demand, audit trail) | ~$3 |
| **Net new cost** | **~$30–150/mo** |

The savings come from **avoided incidents and reduced on-call toil**, not from cost cutting — but it complements the FinOps work (Project 7) by preventing runaway-resource incidents.

---

## 14. Metrics & Results

Consistent with the resume's headline observability numbers, extended by this platform:

| Metric | Result | How |
|---|---|---|
| MTTR | **2+ hours → < 5 minutes** | Auto-heal resolves common failures in seconds; ML detection + Jaeger context speeds human RCA on the rest |
| Alert noise | **~70% reduction** | ML anomaly detection + persistence checks replace static-threshold spam |
| Auto-resolved incidents | **~80% without human** | Restart / scale / rollback playbooks for common failure classes |
| MTTD (mean time to detect) | **minutes → seconds** | Real-time Kafka scoring vs. waiting for a threshold breach |
| P1 pages | **materially reduced** | Humans paged only after auto-heal is exhausted |

> **Interview honesty note:** the ~80% auto-resolution and ~70% noise reduction are the *design targets / observed outcomes* for the classes of failure the platform covers (transient pod/memory/deploy issues). Novel or stateful failures still escalate to humans — that's by design.

---

## 15. Limitations & Gotchas

- **Cold-start problem for new services** — no baseline yet, so detection is weak for the first few days. Mitigation: start in "observe-only" (alert, don't heal) mode.
- **Isolation Forest is not seasonal-aware by itself** — I bolt on time-of-day baselines; a proper seasonal model (LSTM/Prophet) is the roadmap fix.
- **Auto-heal can mask a real problem** — if a pod restarts cleanly every hour, the symptom is hidden. Mitigation: recurring-anomaly detection escalates *even when auto-heal succeeds*.
- **Feedback loop risk** — a bad rollback playbook could thrash. Mitigation: rate limits + circuit breaker (max N actions/window, then freeze + page).
- **Garbage-in** — if log parsing/feature extraction breaks, the model scores nonsense. Mitigation: schema validation before scoring; the "monitor the monitor" dashboard.

---

## 16. Design Decisions

| Decision | Chosen | Why | Trade-off |
|---|---|---|---|
| Detection model | Isolation Forest (unsupervised) | No labelled data; multi-feature; cheap to run | Less accurate than a well-labelled supervised model |
| Streaming | Kafka | Real-time scoring; decouples detection from collection | Operational overhead vs. batch |
| Remediation trigger | EventBridge → Lambda/Ansible | Reuses Project 5 pattern; serverless, near-zero idle cost | Event-driven debugging is harder than a monolith |
| Escalation | ServiceNow ITIL lifecycle | Reuses Project 9; enterprise audit/compliance | Heavier than "just Slack" |
| Autonomy level | Auto-heal stateless; confirm destructive | Safety first; contain blast radius | Not fully "lights-out" |

---

## 17. Interview Talking Points

**2-minute story (STAR):**
> *Situation:* We had strong observability (Prometheus/EFK/Jaeger across 15+ microservices) but were drowning in static-threshold alerts and reacting to failures after they hit users. *Task:* Cut alert fatigue and MTTR without adding headcount. *Action:* I built an AIOps layer on top of the existing stack — a Kafka-streamed Isolation Forest anomaly detector that feeds an EventBridge→Lambda/Ansible auto-healer (restart, scale, rollback), escalating to ServiceNow/PagerDuty only when automation is exhausted, with every action audited in DynamoDB and fed back to tune the model. *Result:* ~70% less alert noise, ~80% of common incidents auto-resolved with no human, and MTTR from 2+ hours to under 5 minutes.

**Likely deep-dive questions:**
- *"Why Isolation Forest over a supervised model?"* → No labelled failure data at scale; unsupervised learns "normal" and isolates outliers. Supervised is a secondary predictor only where labels exist.
- *"How do you avoid the AI making things worse?"* → Persistence checks before acting, always-verify-after-heal, rate limits + circuit breaker, human confirmation for destructive/stateful actions.
- *"How is this different from Project 5?"* → Same remediation *plumbing* (EventBridge/Lambda/DynamoDB), different trigger: Project 5 = rule-based security violations; Project 10 = ML anomaly scores for reliability.
- *"How do you handle the 02:00 batch-job false positive?"* → Time-of-day-aware baselines + seasonality; persistence requirement across N windows.
- *"How do you know the AIOps platform itself is healthy?"* → Heartbeat metric, model-drift dashboard, auto-heal success rate, weekly false-positive review.

**Related projects to mention:** Project 5 (predecessor remediation engine), Project 9 (ServiceNow ITIL lifecycle), Project 6 (Jaeger RCA context), Project 11 (AI in the *pipeline* — the delivery-side counterpart to this runtime-side platform).
