# Troubleshooting Scenarios — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 14, 15, 32, 47

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 14: Scenarios & Real-World Challenges

---

### Q: You're the only DevOps engineer available on Saturday. Management is pressuring you to deploy a new container version to fix a production issue without following the standard process. What do you do?

**Project Reference:** P1 (DevSecOps Pipeline — deployment process)

**Answer:**

> "I don't skip the process — but I accelerate it. Here's my approach:
>
> 1. **Assess severity** — Is this a P1 (users impacted now) or can it wait until Monday? If P1, I act fast but smart.
>
> 2. **Use the pipeline, not manual deploy** — Even on emergency, the image goes through CI. I might skip DAST (takes 10 min) but NOT container scanning or image signing. A rushed manual `kubectl set image` with no validation is how outages get worse.
>
> 3. **Reduce blast radius** — Deploy canary (5% traffic) first. If fix works on canary → promote. If it breaks something else → instant rollback. This takes 10 extra minutes but saves hours of firefighting.
>
> 4. **Document as I go** — Open a Slack thread, note what I'm doing, get verbal approval from a peer (even remote). Screenshot the approval.
>
> 5. **Raise a retrospective ticket** — Monday morning: 'Why did this need an emergency deploy? What process gap allowed the bug to reach production?'
>
> **What I tell management:** 'I'll deploy it fast, but through the pipeline. A manual deploy without scanning could introduce a worse problem. Give me 20 minutes instead of 5, and I guarantee a safe fix.'
>
> **What I NEVER do:** `kubectl set image` directly on production with no scanning, no canary, no rollback plan. That's how a fix becomes a bigger outage."

---

### Q: Comparison of EKS vs ECS — why choose one over the other?

**Project Reference:** P3 (Kubernetes Platform — EKS)

**Answer:**

> | Aspect | EKS (Kubernetes) | ECS (AWS Native) |
> |---|---|---|
> | **Complexity** | Higher — more knobs to turn | Lower — AWS manages more |
> | **Portability** | Multi-cloud, on-prem (K8s everywhere) | AWS-only (vendor lock-in) |
> | **Ecosystem** | Helm, Istio, ArgoCD, Prometheus, etc. | Fewer third-party integrations |
> | **Scaling** | Karpenter (powerful, flexible) | Auto Scaling built-in (simpler) |
> | **Networking** | VPC CNI, Calico, NetworkPolicies, service mesh | awsvpc mode, simpler networking |
> | **Cost** | $0.10/hr control plane + worker nodes | No control plane cost (Fargate pricing) |
> | **Team skills** | Need K8s expertise | Lower learning curve |
>
> **Choose EKS when:**
> - Team already knows Kubernetes
> - Multi-cloud or hybrid strategy (avoid lock-in)
> - Need advanced networking (service mesh, NetworkPolicies)
> - Need rich ecosystem (Helm, GitOps, policy engines)
> - Running 15+ microservices with complex deployment strategies (canary, blue-green)
>
> **Choose ECS when:**
> - Small team, fewer services (3-5)
> - Pure AWS shop, no multi-cloud plans
> - Want simplicity over flexibility
> - ECS Fargate: no node management at all
>
> **Our choice:** EKS — because we have 15+ microservices, need Istio service mesh, ArgoCD GitOps, Kyverno policies, and our team has deep K8s expertise. ECS would be limiting."

---

### Q: Disaster recovery strategy — Cold vs Warm Standby for a cost-sensitive e-commerce app?

**Project Reference:** P8 (Multi-Region HA/DR)

**Answer:**

> | Strategy | RTO | Cost | What's Running in DR |
> |---|---|---|---|
> | **Cold** | Hours (4-24hr) | Lowest | Nothing. Infra defined in Terraform, deploy on demand |
> | **Warm Standby** | Minutes (3-15min) | Medium | Scaled-down infra running (1 instance, DB replica) |
> | **Hot/Active-Active** | Seconds | Highest | Full capacity in both regions |
>
> **For cost-sensitive e-commerce, I'd recommend Warm Standby:**
>
> - DB: Aurora Global Database (read replica in DR region — ~$200/month for a small instance). Promotes in ~60 seconds.
> - Compute: Warm pool with 1 small instance (pre-baked AMI). ASG scales up on failover.
> - DNS: Route53 health checks → automatic failover in ~90 seconds.
> - Static assets: S3 Cross-Region Replication (pennies for storage).
>
> **Total DR cost:** ~$300-500/month for a small-medium app. RTO: ~3 minutes.
>
> **Why not Cold:** For e-commerce, hours of downtime = massive revenue loss. The $300/month for warm standby is insurance worth having.
>
> **Why not Active-Active:** Double the cost for compute + complex data consistency (DynamoDB Global Tables needed). Only justified if you need multi-region for latency (international users) or zero-second RTO."

---

### Q: How do you trace requests flowing from pods in EKS?

**Project Reference:** P6 (Istio — Jaeger distributed tracing), P3 (Observability)

**Answer:**

> "**Distributed tracing with Jaeger (via Istio).**
>
> How it works:
> 1. Istio sidecar (Envoy) automatically generates trace spans for every request entering/leaving a pod — no code changes needed for basic tracing.
> 2. Each request gets a unique trace ID (propagated via headers: `x-request-id`, `x-b3-traceid`).
> 3. Spans are sent to Jaeger collector → stored → queryable in Jaeger UI.
>
> **What I see in Jaeger:**
> - Full request path: Ingress → Service A (50ms) → Service B (200ms) → Database (150ms)
> - Exactly WHERE latency is introduced
> - Error locations (which service returned 500)
>
> **For deeper application-level tracing:** Developers add OpenTelemetry SDK to their code — adds custom spans for business logic (e.g., 'payment validation took 500ms').
>
> **Alternative on pure AWS:** AWS X-Ray with X-Ray daemon as DaemonSet on EKS. Similar concept, tighter AWS integration but less ecosystem flexibility."

---

### Q: Experience with Ingress controllers — ALB Ingress Controller?

**Project Reference:** P3 (Kubernetes Platform — EKS)

**Answer:**

> "Yes, we use **AWS Load Balancer Controller** (successor to ALB Ingress Controller) on EKS.
>
> **How it works:**
> - You create a K8s `Ingress` resource with annotations
> - The controller provisions an actual ALB in AWS automatically
> - Routing rules (host, path) map to K8s Services → target groups
>
> **Key features we use:**
> - **IP-mode targets** — ALB routes directly to pod IPs (VPC CNI). Skips NodePort. Lower latency, better distribution.
> - **SSL termination** — ACM certificate ARN in annotation. ALB handles TLS.
> - **WAF integration** — WAF WebACL attached via annotation.
> - **Multiple ingress grouping** — `alb.ingress.kubernetes.io/group.name` — multiple services share one ALB (cost saving).
>
> **vs Nginx Ingress:** ALB controller is better on EKS because it's AWS-native (auto-provisions ALB, integrates with WAF, ACM, Shield). Nginx Ingress is better for self-managed or multi-cloud clusters."

---

### Q: Requirement for service discovery tools like Istio?

**Project Reference:** P6 (Istio Service Mesh)

**Answer:**

> "Istio is more than service discovery — but here's when you NEED it:
>
> **You need Istio when:**
> 1. **mTLS everywhere** — zero-trust networking between all services. Without Istio, you'd implement TLS in each app individually (nightmare at 15+ services).
> 2. **Traffic management** — Canary deployments (5% to new version), traffic mirroring, fault injection for testing.
> 3. **Observability for free** — Envoy sidecar gives you request metrics, tracing, and access logs without code changes.
> 4. **Authorization policies** — 'Service A can call Service B, but Service C cannot' — enforced at network level, not application code.
>
> **You DON'T need Istio when:**
> - <5 services (overhead not justified)
> - Simple networking needs (K8s Services + DNS enough)
> - Team doesn't have bandwidth to learn/operate mesh
>
> **Kubernetes has basic service discovery built-in** (CoreDNS resolves `service-name.namespace.svc.cluster.local`). Istio adds security, observability, and traffic control ON TOP of that."

---

### Q: A challenge or project you're proud of?

**Project Reference:** P9 (OS Patching Automation)

**Answer:**

> "The OS Patching Automation — going from a manual, error-prone, 3-day exercise to a zero-touch, zero-downtime system.
>
> **The challenge:** 500+ production RHEL servers running a carrier-grade voice platform. Patching previously required 4 engineers, 3 days, and averaged 2 incidents per month. The client was losing confidence.
>
> **What I built:**
> - 18-step automation on Ansible AAP with 12 independent roles
> - 7-dimension post-patch validation (services, ports, connectivity, disk, certs, integrity, logs)
> - ALB traffic drain → serial 20% rolling update → traffic return only after ALL validations pass
> - ServiceNow ITIL integration (auto CR open → close)
> - One-click rollback if any dimension fails
>
> **Result:** Zero downtime, zero incidents for 18 months. Patching reduced from 3 days to 4 hours. Client renewed contract citing this as the differentiator.
>
> **Why I'm proud:** It wasn't just automation — it was building TRUST. The client went from 'we're afraid to patch' to 'we patch monthly without even thinking about it.' That's the impact I want to have."

---

### Q: Rollback strategy and contingency planning for cluster upgrades?

**Project Reference:** P3 (Kubernetes Platform — upgrades)

**Answer:**

> "You can't downgrade a K8s control plane — so rollback planning is critical BEFORE you upgrade.
>
> **My contingency plan:**
>
> 1. **Pre-upgrade:**
>    - etcd snapshot (self-managed clusters) or Terraform state backup (EKS)
>    - Full Velero backup of all workloads, PVCs, configs
>    - Document current versions of ALL add-ons (CNI, CoreDNS, ingress, Istio)
>    - Test in Dev cluster first — run full test suite
>
> 2. **During upgrade (if something breaks):**
>    - **Node-level rollback** — Don't upgrade all node groups at once. If new nodes have issues, drain them, keep old nodes running.
>    - **Workload rollback** — ArgoCD can revert any application to previous Git commit.
>    - **Add-on rollback** — If new CoreDNS/CNI version is broken, Helm rollback to previous version.
>
> 3. **If control plane upgrade breaks things (EKS):**
>    - Can't downgrade EKS control plane. Instead: spin up a NEW cluster at old version from Terraform, restore Velero backup, switch DNS. Takes ~30 min.
>    - This is why we keep Terraform code for cluster creation always current — recreating is the rollback.
>
> 4. **PodDisruptionBudgets** — Ensure at least N-1 pods running during node drain. No service interruption during the upgrade process itself.
>
> **Key learning:** The 'rollback' for K8s upgrades is 'stand up a new cluster and migrate workloads' — not 'downgrade in place.' Plan accordingly."

---

*Note: Terraform-specific IaC questions (emergency fix, long-term IaC, collaboration & remote state) have been moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)*

---

### Q: Impact of AI and LLMs on the DevOps role and your future plan?

**Project Reference:** General — forward-looking

**Answer:**

> "AI is already changing DevOps — and will accelerate. My view:
>
> **What AI will automate (already happening):**
> - Writing boilerplate Terraform/YAML/Helm charts
> - Log analysis and anomaly detection (AIOps — pattern recognition at scale humans can't do)
> - Incident triage (auto-correlate alerts → suggest root cause)
> - Generating runbooks from incident history
>
> **What AI WON'T replace:**
> - Architecture decisions (trade-offs, business context, cost vs reliability judgment)
> - Production incident response (real-time pressure, creative problem solving)
> - Security thinking (threat modeling requires adversarial mindset)
> - Team collaboration, stakeholder communication
>
> **My future plan:**
> - I'm already using AI tools (GitHub Copilot, Kiro) for faster IaC/pipeline development
> - Learning to integrate LLMs into observability (AI-driven anomaly detection for Prometheus metrics)
> - Building toward **Platform Engineering** — self-service golden paths for developers, where AI assists in policy decisions
>
> **My take:** DevOps engineers who use AI will replace those who don't. But AI won't replace engineers who understand systems at a deep level — it'll just make them 3x more productive."

---

---
---

# SECTION 15: Infrastructure & Troubleshooting Scenarios

---

*Note: "How will you create a highly available infrastructure using Terraform best practices from scratch?" has been moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)*

---

### Q: What is a Managed Instance Group (MIG), what is it, what does it do?

**Project Reference:** No direct project (GCP concept) — equivalent to AWS ASG

**Answer:**

> "MIG is **Google Cloud's Auto Scaling Group** — it's GCP's equivalent of AWS ASG.
>
> **What it does:**
> - Maintains a group of identical VM instances based on an instance template
> - **Auto-healing** — if a VM fails health check, MIG recreates it automatically
> - **Auto-scaling** — scales up/down based on CPU, load balancer utilization, or custom metrics
> - **Rolling updates** — update instance template → MIG gradually replaces VMs with new version
> - **Multi-zone distribution** — regional MIG spreads instances across multiple zones for HA
>
> **Components:**
> - **Instance template** — defines VM config (machine type, image, startup script) — like an AWS Launch Template
> - **Health check** — determines if VM is healthy (HTTP, TCP, or command-based)
> - **Autoscaler** — policy for scale up/down
>
> **AWS equivalents:**
> | GCP | AWS |
> |---|---|
> | MIG | Auto Scaling Group (ASG) |
> | Instance Template | Launch Template |
> | Regional MIG | Multi-AZ ASG |
> | MIG auto-healing | ASG health check replacement |
>
> I haven't used GCP directly, but the concepts map 1:1 to our AWS ASG setup in Project 2 — multi-AZ, health-check-driven replacement, rolling updates."

---

### Q: How is MIG helping you in high availability?

**Project Reference:** No direct project (GCP) — equivalent concept in P2 (AWS ASG)

**Answer:**

> "MIG provides HA through three mechanisms (same as AWS ASG):
>
> 1. **Multi-zone distribution** — Regional MIG spreads VMs across 3+ zones. One zone fails → others absorb traffic. No single point of failure.
>
> 2. **Auto-healing** — Health check fails → MIG terminates unhealthy VM and launches a new one automatically. No human intervention. In our AWS setup, ASG does the same — unhealthy instance is replaced within minutes.
>
> 3. **Auto-scaling** — Traffic spike → MIG adds instances. Traffic drops → removes them. Always has capacity to serve. Maintains minimum healthy instances at all times.
>
> **The HA guarantee:** At any point, `min_size` healthy instances are running across multiple zones. Even if instances crash, even if a zone goes down — MIG ensures the desired count is maintained. Paired with a load balancer that only routes to healthy instances — users never see downtime."

---

### Q: Dockerized app in production keeps crashing. CPU, memory, disk are fine. Logs don't state much. MongoDB is rejecting new requests. What are some reasons a database will reject new requests?

**Project Reference:** P3 (Kubernetes troubleshooting), No direct MongoDB project

**Answer:**

> "If MongoDB is rejecting connections while system resources look fine — here's what I'd investigate:
>
> 1. **Connection pool exhausted** — MongoDB has a max connections limit (default: 65,536, but often set lower). If the app isn't closing connections properly (connection leak), new requests get rejected. Check: `db.serverStatus().connections` — if `current` is near `available`, that's your problem.
>
> 2. **Disk IOPS saturated (not disk space)** — Disk has space but no IOPS left. MongoDB write locks when I/O is saturated. CPU/memory look fine but every write is queued. Check: `iostat -x` → look at `%util` and `await`.
>
> 3. **WiredTiger cache pressure** — MongoDB's storage engine cache is full. It starts evicting, blocking new operations. Default cache = 50% of RAM. If working set exceeds cache → thrashing.
>
> 4. **Lock contention** — Long-running queries holding write locks. New writes queue up and timeout. Check: `db.currentOp()` → look for long-running operations.
>
> 5. **Replica set election / primary stepdown** — If it's a replica set, primary may have stepped down. App is trying to write to a secondary (read-only). Check: `rs.status()`.
>
> 6. **Max document size / write concern timeout** — Write concern set to `majority` but secondaries are lagging. Writes timeout waiting for acknowledgment.
>
> 7. **Authentication/authorization failure** — Credentials rotated or expired. App is authenticated but new connections fail auth.
>
> **My approach:** Check `db.serverStatus()`, `db.currentOp()`, replica set status, and connection count FIRST. 80% of 'DB rejecting requests' is connection pool or IOPS."

---
---

# SECTION 32: Troubleshooting & Logging

---

### Q: If an auto-scaling event occurs, machines are created and destroyed, and some crash — how do you investigate crashed machines?

**Project Reference:** P2 (3-Tier — ASG), P3 (Kubernetes — nodes)

**Answer:**

> "Ephemeral instances that crash and terminate are tricky — the evidence is gone. Here's how:
>
> 1. **CloudWatch Logs (pre-configured in AMI)** — Our AMI has CloudWatch Agent baked in. Application logs and system logs (`/var/log/messages`, cloud-init output) stream to CloudWatch BEFORE the instance terminates. Log group persists even if instance is gone.
>
> 2. **ASG Activity History** — AWS Console → ASG → Activity tab shows why instances were terminated (health check failure, scaling-in, spot interruption). `aws autoscaling describe-scaling-activities`.
>
> 3. **ALB Access Logs + Target Health** — Shows when the instance was marked unhealthy and why (health check response code, timeout).
>
> 4. **EC2 Instance Status Checks** — `aws ec2 describe-instance-status` shows if it failed system or instance status checks.
>
> 5. **Serial Console / Screenshot (if still running)** — For boot failures, EC2 serial console output shows kernel panics or boot errors.
>
> 6. **For Kubernetes nodes:** kubelet logs in CloudWatch (EKS sends these automatically). Pod eviction events in `kubectl get events`. Karpenter logs show why node was terminated.
>
> **Prevention:** Health check grace period (give instance time to boot before ASG checks health). Proper health check endpoint (not just TCP, but `/health` that verifies app is ready)."

---

### Q: How do you get application logs into AWS CloudWatch?

**Project Reference:** P2 (3-Tier — EC2), P3 (EKS)

**Answer:**

> "**For EC2 instances:**
> - **CloudWatch Agent** installed in AMI (baked via Packer). Config file specifies which log files to ship:
> ```json
> {
>   \"logs\": {
>     \"logs_collect_list\": [
>       {
>         \"file_path\": \"/var/log/app/application.log\",
>         \"log_group_name\": \"/app/production\",
>         \"log_stream_name\": \"{instance_id}\"
>       }
>     ]
>   }
> }
> ```
> - Agent tails the log file and ships to CloudWatch Logs in near real-time.
>
> **For EKS/Kubernetes:**
> - **Fluentd/Fluent Bit DaemonSet** — Runs on every node, collects container stdout/stderr, ships to CloudWatch Logs (or Elasticsearch).
> - OR: **AWS for Fluent Bit** — AWS-maintained image, lightweight, direct CloudWatch integration.
> - Container logs are automatically captured from `/var/log/containers/` on the node.
>
> **For Lambda:**
> - Automatic. Lambda writes to CloudWatch Logs by default (log group: `/aws/lambda/function-name`). No configuration needed.
>
> **Our primary stack:** EFK (Elasticsearch + Fluentd + Kibana) for K8s logs (better search), CloudWatch for AWS service logs (VPC Flow, CloudTrail, Lambda)."

---

### Q: What information do you get from SonarQube analysis?

**Project Reference:** P1 (DevSecOps Pipeline — Stage 7: SAST)

**Answer:**

> "SonarQube gives us code quality AND security analysis:
>
> **Quality metrics:**
> - **Bugs** — Code that will behave unexpectedly (null pointer, resource leaks)
> - **Code smells** — Maintainability issues (long methods, duplicated code, complexity)
> - **Technical debt** — Time estimate to fix all issues (e.g., '3 days of tech debt')
> - **Coverage** — Unit test coverage percentage (we gate at >80%)
> - **Duplications** — Percentage of duplicated code blocks
>
> **Security (SAST):**
> - **Vulnerabilities** — SQL injection patterns, XSS, hardcoded credentials, insecure deserialization
> - **Security hotspots** — Code that MIGHT be vulnerable, needs human review (crypto usage, regex patterns)
>
> **Our quality gate (pipeline fails if):**
> - Any new Critical/Blocker bug
> - Any new security vulnerability
> - Coverage drops below 80%
> - Duplications above 5%
>
> SonarQube runs in Stage 7 of our pipeline. Results posted back to the PR as a comment. Developer sees 'Quality Gate: FAILED — 2 vulnerabilities found' before merge."

---

### Q: What options do you have for scaling databases (RDS) in AWS?

**Project Reference:** P2 (3-Tier — Aurora), P8 (Multi-Region)

**Answer:**

> "**Vertical scaling (scale UP):**
> - Change instance class (db.r5.large → db.r5.xlarge). Requires brief downtime (~30s with Multi-AZ failover).
> - Good for: write-heavy workloads where you need more CPU/memory on the writer.
>
> **Horizontal scaling (scale OUT — reads):**
> - **Read replicas** — Up to 15 Aurora replicas. Application reads from replicas, writes go to primary. Near-instant replication.
> - Good for: read-heavy workloads (reporting, search, analytics).
>
> **Aurora Auto Scaling:**
> - Automatically adds/removes read replicas based on CPU or connections.
> - Set min 1, max 5 replicas. Aurora scales readers automatically during peak.
>
> **Aurora Serverless v2:**
> - Auto-scales compute (ACUs) up and down. No instance class management. Pay for actual usage.
> - Good for: variable/unpredictable workloads (dev environments, spiky traffic).
>
> **Cross-region (DR + performance):**
> - Aurora Global Database — read replica in another region. <1s replication. Serves regional reads with low latency.
>
> **Caching (offload DB entirely):**
> - ElastiCache (Redis) in front of RDS. Cache frequent reads. Reduces DB load by 80%.
>
> | Scaling Type | Use Case | Downtime |
> |---|---|---|
> | Vertical (bigger instance) | Write bottleneck | ~30s (Multi-AZ failover) |
> | Read replicas | Read bottleneck | None (add online) |
> | Aurora Serverless | Variable workload | None (auto) |
> | Caching (Redis) | Reduce DB hits | None |"

---
---

# SECTION 47: Production Incidents & Operational Scenarios

---

### Q: You arrive at the office and one of your critical applications has crashed/is inaccessible. What are the key steps you would take?

**Project Reference:** P3 (Observability), P8 (HA/DR), General — incident response

**Answer:**

> "Structured triage — don't panic, follow the process:
>
> **First 2 minutes (Assess):**
> 1. Check monitoring — Grafana/PagerDuty alert. What exactly is down? Entire app or partial functionality?
> 2. Check scope — One user reporting? Or full outage (Blackbox exporter confirms endpoint unreachable)?
> 3. Declare severity — P1 (all users affected) → open war room. P2 (degraded) → investigate, no war room yet.
>
> **Next 5 minutes (Recent changes):**
> 4. Check: 'What changed recently?' — ArgoCD: any recent deployments? Terraform: any infra changes? CloudTrail: any IAM/security changes?
> 5. 90% of outages follow a recent change. If a deployment happened in last hour → **rollback immediately** (ArgoCD revert). Investigate later.
>
> **If no recent change (infra issue):**
> 6. Check EKS: `kubectl get pods` — CrashLoopBackOff? Pending? OOMKilled?
> 7. Check nodes: Any NotReady? Disk pressure? Instance terminated?
> 8. Check external dependencies: DB connection? DNS? Certificate expiry?
> 9. Check ALB: healthy host count = 0? Target group issues?
>
> **Communicate:**
> 10. Update stakeholders every 15 min. 'Investigating. Impact: [X]. ETA: [working on it].'
>
> **After resolution:** Blameless postmortem within 48 hours. Document root cause, timeline, and action items to prevent recurrence."

---

### Q: What if you rolled out an application in production, and it is not stable after the release — how would you handle that?

**Project Reference:** P1 (DevSecOps — Canary + auto-rollback)

**Answer:**

> "**Immediate action: Rollback first, investigate later.**
>
> **With our canary setup (ideal):**
> - This shouldn't happen because Argo Rollouts auto-rolls back if metrics are bad during canary phase
> - If it somehow got past canary (issue appears only at 100% traffic / under specific conditions):
>
> **Steps:**
> 1. **Rollback** — `kubectl argo rollouts undo <rollout>` or ArgoCD: revert Git commit to previous Helm values. Takes 2 minutes. Users restored to stable version.
>
> 2. **Contain** — If rollback isn't instant (database schema changed), consider: feature flag OFF, traffic shift back to old version, or maintenance page.
>
> 3. **Investigate (after users are stable):**
>    - What's different? Diff the Helm values / code between versions
>    - Check logs: new errors? New stack traces?
>    - Check metrics: latency spike? Error rate? Memory growing?
>    - Check traces (Jaeger): which service/call is failing?
>
> 4. **Root cause + fix:**
>    - Developer fixes the bug on a branch
>    - Full pipeline re-run (tests, security scan)
>    - Re-deploy via canary (5% first, watch carefully this time)
>
> **Key principle:** Production stability > understanding the problem. Rollback FIRST. Rootcause SECOND. Never let users suffer while you debug."

---

### Q: When you identify performance issues in production, how would you go about investigating that?

**Project Reference:** P3 (Observability — Prometheus, Jaeger), P6 (Istio)

**Answer:**

> "**Systematic top-down approach (USE + RED method):**
>
> **1. Confirm the symptom (RED — Rate, Errors, Duration):**
> - Grafana dashboard: Is latency up? Error rate up? Throughput dropping?
> - Which service? Which endpoint? When did it start?
>
> **2. Resource check (USE — Utilization, Saturation, Errors):**
> - `kubectl top pods` — any pod hitting CPU/memory limits (throttled)?
> - Node-level: CPU steal time? Disk I/O wait?
> - DB: Aurora Performance Insights → slow queries? Connection pool full?
>
> **3. Distributed tracing (pinpoint bottleneck):**
> - Jaeger: trace a slow request end-to-end
> - 'Request takes 3s total: Auth service 50ms, Payment service 2.8s, DB query inside payment: 2.5s'
> - Found it: specific DB query in payment service is slow
>
> **4. Deep dive into the bottleneck:**
> - Slow query? → Check query plan (missing index? table scan on 10M rows?)
> - CPU throttled? → Increase pod limits or fix CPU-intensive code
> - External API slow? → Add circuit breaker, increase timeout, add cache
> - Network latency? → Cross-AZ calls? DNS resolution slow?
>
> **5. Fix + validate:**
> - Apply fix (add index, increase limits, add cache)
> - Watch Grafana: latency dropping back to normal?
> - Document: 'Performance issue at [time], caused by [X], fixed by [Y]'"

---

### Q: How do you handle a scenario where deployment requires database schema changes?

**Project Reference:** P1 (DevSecOps — deployment strategy), P2 (Aurora)

**Answer:**

> "Database schema changes are the hardest part of deployments — you can't just rollback a schema change like you rollback code. My approach:
>
> **Rule: Schema changes must be backward-compatible.**
>
> **Strategy: Expand-Migrate-Contract (3 phases):**
>
> 1. **Expand** (Deploy schema change first):
>    - Add new column (nullable) / new table. Don't remove or rename anything.
>    - Old code still works (ignores new column). New code can use it.
>    - Deploy migration: `ALTER TABLE orders ADD COLUMN status_v2 VARCHAR(50);`
>
> 2. **Migrate** (Deploy new application code):
>    - New code writes to BOTH old and new columns (dual-write)
>    - Backfill old data: populate `status_v2` from `status` for existing rows
>    - Validate: both columns in sync
>
> 3. **Contract** (Cleanup — separate deployment later):
>    - Once all code uses new column and old column is no longer read
>    - Drop old column in a future release (after verification period)
>
> **Why NOT just change the schema and deploy together:**
> - During canary: 5% of pods run new code (expects new schema), 95% run old code (expects old schema). If you change schema first → old code breaks. If you deploy code first → new code breaks.
> - Backward-compatible schema = both old and new code work simultaneously.
>
> **Tools:** Flyway or Liquibase for versioned schema migrations. Run as a pre-deploy step in the pipeline (not inside the application startup)."

---

### Q: How did you move gp2 to gp3 on so many servers?

**Project Reference:** P7 (Cost Optimization — EBS migration)

**Answer:**

> "**Automated with a combination of AWS CLI scripting + Terraform:**
>
> **For Terraform-managed volumes:**
> - Changed `volume_type = \"gp2\"` → `volume_type = \"gp3\"` in Terraform
> - `terraform plan` showed: 'modify volume type (no replacement needed)'
> - `terraform apply` — AWS modifies EBS volume type ONLINE. No downtime, no detach needed.
> - Rolled out environment by environment: Dev → Staging → Prod (over 2 weeks)
>
> **For non-Terraform volumes (legacy):**
> - Python script using boto3:
>   ```python
>   volumes = ec2.describe_volumes(Filters=[{'Name': 'volume-type', 'Values': ['gp2']}])
>   for vol in volumes:
>       ec2.modify_volume(VolumeId=vol['VolumeId'], VolumeType='gp3')
>   ```
> - Batched: 50 volumes per batch, with 5-minute wait between batches
> - Monitored: CloudWatch VolumeReadOps/WriteOps to ensure no performance degradation during modification
>
> **Key facts:**
> - gp2 → gp3 modification is **online** — zero downtime, no reboot, no detach
> - Volume enters 'optimizing' state for a few hours but remains fully usable
> - Can't modify again until optimization completes (~6 hours per volume)
> - Result: 20% cost reduction on EBS + independent IOPS/throughput tuning capability"

---

### Q: How did you research Graviton being cheaper, and why doesn't everyone on AWS migrate to Graviton?

**Project Reference:** P7 (Cost Optimization — ARM/Graviton evaluation)

**Answer:**

> "**How I researched:**
> 1. AWS pricing page showed m6g (Graviton) is 20% cheaper than m5 (x86) for equivalent performance
> 2. AWS published benchmarks: Graviton gives 40% better price-performance for many workloads
> 3. We tested in Dev: ran same workload on m5.large vs m6g.large, compared response times and throughput — matched or better on Graviton
> 4. Kubecost showed per-node cost difference instantly after switching one node group
>
> **Why NOT everyone migrates:**
>
> 1. **Binary compatibility** — Graviton is ARM (aarch64). x86 binaries don't run on ARM. You need to recompile or use multi-arch container images. If your app has native x86 dependencies (compiled C libraries, proprietary software) — it won't work without effort.
>
> 2. **Container images** — Need `linux/arm64` images. Most popular images support multi-arch now, but some internal/third-party images are x86-only.
>
> 3. **Testing effort** — Need to validate entire stack on ARM. Some subtle bugs only appear on ARM (endianness edge cases, assembly optimizations).
>
> 4. **Third-party software** — Some monitoring agents, security tools, or vendor software don't have ARM builds yet.
>
> 5. **Inertia** — Works fine on x86, migration has risk, team is busy with features.
>
> **Our approach:** Graviton for stateless K8s workers (containers are multi-arch — just rebuild with `docker buildx`). Keep x86 for anything with native binary dependencies. Mixed node groups — Karpenter schedules ARM-compatible pods on Graviton nodes automatically."

---

---
---

