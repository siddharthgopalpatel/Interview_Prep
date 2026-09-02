# Monitoring Observability SRE — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 5, 27

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 5: Monitoring & Alerting

---

### Q: How will your monitoring system inform you if a user is not able to hit an endpoint?

**Project Reference:** P3 (Observability stack), P1 (Canary monitoring)

**Answer:**

> "We monitor from outside-in:
>
> 1. **Synthetic monitoring / Blackbox exporter** — Prometheus Blackbox exporter hits the endpoint every 30 seconds from outside the cluster. If it gets non-200 or timeout → alert fires immediately. This catches what internal monitoring misses (ingress down, DNS broken, certificate expired).
>
> 2. **Ingress-level metrics** — ALB access logs + CloudWatch metrics show 5xx spikes, increased latency, or connection refused.
>
> 3. **AlertManager routing** — Endpoint down for >1 min = P1 alert → PagerDuty → engineer paged. Not just a Slack message that gets ignored.
>
> The key: monitor from the USER's perspective (external probe), not just from inside the cluster. Your pod might be 'healthy' internally but unreachable due to ingress/DNS/network issue."

---

### Q: How are Prometheus, Grafana, CloudWatch not useful in such scenario?

**Project Reference:** P3 (Observability stack)

**Answer:**

> "They ARE useful — but only if configured correctly. The mistake is relying solely on **internal metrics** (pod CPU, memory, container health). A pod can report 'healthy' while the ingress is misconfigured and no traffic reaches it.
>
> What's needed:
> - **Prometheus** — useful IF you add external probes (Blackbox exporter) that test the actual endpoint from outside
> - **CloudWatch** — useful for ALB metrics (5xx count, healthy host count, target response time)
> - **Grafana** — visualization layer, only as good as the data sources feeding it
>
> The gap people miss: they monitor the **component** (pod is running) but not the **user journey** (can a user actually access the service end-to-end). Blackbox monitoring + synthetic tests close that gap."

---

### Q: Where is the security monitoring? How do you know your containers are not exposed?

**Project Reference:** P1 (DevSecOps — runtime security), P6 (Istio — NetworkPolicies)

**Answer:**

> "Multiple layers:
>
> 1. **Network Policies (Calico)** — Default-deny. Pods can only talk to explicitly allowed destinations. If a container is compromised, it can't reach other services laterally.
>
> 2. **Istio AuthorizationPolicies** — Zero-trust. Even if network allows it, Istio requires valid mTLS identity. Service A can only call Service B if explicitly authorized.
>
> 3. **Runtime security (Falco)** — Detects anomalous behavior INSIDE containers: unexpected shell spawned, binary executed, file accessed outside normal pattern, network connection to unusual IP.
>
> 4. **AWS GuardDuty for EKS** — Detects compromised containers, crypto-mining, privilege escalation attempts at the cluster level.
>
> So: NetworkPolicies prevent lateral movement, Istio enforces identity-based access, Falco detects runtime anomalies."

---

### Q: If someone hacks into your container and starts running commands, which tool will tell you?

**Project Reference:** P1 (DevSecOps — runtime security), P3 (Kubernetes security)

**Answer:**

> "**Falco**. It's a runtime security tool that monitors system calls (syscalls) inside containers using eBPF/kernel modules.
>
> Examples of what Falco detects:
> - Shell spawned inside container (`bash`, `sh`) — containers shouldn't have interactive shells in production
> - Unexpected binary executed (attacker downloads and runs a tool)
> - Sensitive file read (`/etc/shadow`, `/etc/passwd`)
> - Outbound connection to unknown IP (data exfiltration, C2 communication)
>
> Falco fires an alert → goes to our AlertManager → P1 incident: 'Container shell detected in production pod X.'
>
> Additionally, our containers run with **read-only root filesystem** — even if an attacker gets in, they can't write or install anything. And no capabilities like `CAP_NET_RAW` — so they can't even run packet sniffers."

---

---
---

# SECTION 27: SRE (Site Reliability Engineering)

---

### Q: What do you understand by Site Reliability Engineering?

**Project Reference:** P3 (Observability), P8 (HA/DR), General

**Answer:**

> "SRE is **applying software engineering principles to operations problems.** Instead of manually firefighting, you automate reliability.
>
> **Core SRE principles:**
> 1. **Error budgets** — Accept that 100% uptime is impossible and uneconomical. Define acceptable failure rate (e.g., 99.95% = 22 min downtime/month allowed). Use remaining 'budget' for releases and changes.
> 2. **Eliminate toil** — If a human does the same manual task repeatedly, automate it. Our patching automation (P9) is pure SRE thinking.
> 3. **Measure everything** — SLIs/SLOs define what 'reliable' means, not gut feel.
> 4. **Blameless postmortems** — When things break, fix the system, not the person.
> 5. **Progressive rollouts** — Canary deployments (P1) — don't risk 100% of traffic on unvalidated code.
>
> **How I practice SRE daily:**
> - Define SLOs for every service
> - Automated canary rollback = software solving reliability
> - OS patching automation = eliminating toil
> - DR drills quarterly = testing reliability before we need it"

---

### Q: Have you heard of SLI, SLO, and SLA? How do you measure each?

**Project Reference:** P3 (Observability), P8 (HA/DR — 99.95% SLA)

**Answer:**

> | Term | What | Example | Who Defines |
> |---|---|---|---|
> | **SLI** (Service Level Indicator) | The METRIC you measure | Request latency P99, error rate, availability percentage | Engineering team |
> | **SLO** (Service Level Objective) | The TARGET for that metric | P99 latency < 300ms, availability > 99.95% | Engineering + Product |
> | **SLA** (Service Level Agreement) | The CONTRACT with customer (with consequences) | 99.9% uptime or customer gets credits | Business + Legal |
>
> **How we measure:**
>
> - **SLI (what we monitor):**
>   - Availability: `successful_requests / total_requests` (from Prometheus)
>   - Latency: P99 response time (from Istio/Envoy metrics)
>   - Error rate: `5xx_responses / total_responses`
>
> - **SLO (our internal target):**
>   - Availability SLO: 99.95% (allows ~22 min downtime/month)
>   - Latency SLO: P99 < 300ms
>   - Error budget: 0.05% of requests can fail before we freeze deployments
>
> - **SLA (customer contract):**
>   - 99.9% availability guaranteed (more lenient than our SLO — internal bar is higher)
>   - Breach = service credits or contract penalties
>
> **Key insight:** SLO should be STRICTER than SLA. If your SLA is 99.9%, target 99.95% internally. That gives you buffer before you breach the contract."

---

### Q: How do you define and calculate the SLO for an application?

**Project Reference:** P3 (Observability), P8 (HA/DR)

**Answer:**

> "**Step-by-step:**
>
> 1. **Identify what 'working' means for users** — Can they complete their critical journey? For our contact center: can calls connect and maintain quality?
>
> 2. **Choose SLIs** — Pick 2-3 indicators that reflect user experience:
>    - Availability (success rate)
>    - Latency (response time)
>    - Throughput (if applicable)
>
> 3. **Set the target based on business need:**
>    - Critical service (payment, voice) → 99.95% (4.4 hrs downtime/year)
>    - Non-critical (admin dashboard) → 99.5% (44 hrs/year)
>    - Don't target 99.99% unless you're willing to pay for multi-region active-active
>
> 4. **Calculate error budget:**
>    - 99.95% SLO over 30 days = 0.05% error budget = 21.6 minutes allowed failure/month
>    - If you burn 15 minutes in week 1 → slow down deployments for the rest of the month
>
> 5. **Measure in Prometheus:**
>    ```promql
>    # Availability SLI
>    sum(rate(http_requests_total{status!~\"5..\"}[30d])) /
>    sum(rate(http_requests_total[30d])) * 100
>    ```
>
> **Key principle:** SLO is a conversation between engineering and product. Engineering says what's achievable. Product says what users need. The SLO lives in between."

---

### Q: What is the difference between RTO and RPO?

**Project Reference:** P8 (Multi-Region HA/DR — 3-min RTO, <1s RPO)

**Answer:**

> | | RTO | RPO |
> |---|---|---|
> | **Full name** | Recovery Time Objective | Recovery Point Objective |
> | **Asks** | How LONG can you be down? | How much DATA can you lose? |
> | **Measures** | Time from failure to recovery | Time between last backup and failure |
> | **Example** | 3 minutes (P8) | <1 second (P8) |
>
> **Simplified:**
> - **RTO = downtime tolerance.** 'We can afford to be down for 3 minutes max.'
> - **RPO = data loss tolerance.** 'We can lose at most 1 second of transactions.'
>
> **How we achieve ours (P8):**
> - **RTO (3 min):** Route53 health check detects failure (~90s) + DNS failover + Aurora promotes secondary (~60s) = ~3 minutes total
> - **RPO (<1s):** Aurora Global Database replicates asynchronously with ~100ms lag. At most 1 second of uncommitted transactions lost.
>
> **Cost relationship:** Lower RTO/RPO = more expensive. Cold DR (RTO: hours) is cheap. Active-active (RTO: seconds) is expensive. We chose warm standby — balances cost vs recovery speed."

---
---

