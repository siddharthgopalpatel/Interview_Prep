# AWS — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 11, 30, 39, 51 (Architecture & Infrastructure) + 22, 28, 31, 34, 44 (Networking)

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 11: AWS Architectural Design (E-Commerce System Design)

---

### Q: Design a highly scalable, robust, and available international e-commerce application using AWS services.

**Project Reference:** P2 (3-Tier AWS Architecture), P8 (Multi-Region HA/DR)

**Answer:**

> "Here's the architecture I'd design — same edge/load-balancing spine as my 3-tier project (P2), evolved to EKS microservices and multi-region data for international scale.
>
> **Architecture Diagram:**
>
> ```
>                     ┌─────────────┐
>                     │   Route 53  │  Latency-based routing → nearest region
>                     └──────┬──────┘
>                            │
>                     ┌──────▼──────┐
>                     │ CloudFront  │  Global CDN (static assets at edge)
>                     └──────┬──────┘
>                            │
>                     ┌──────▼──────┐
>                     │   AWS WAF   │  SQLi / XSS / rate limiting
>                     └──────┬──────┘
>                            │
>               ┌────────────▼────────────┐
>               │           ALB           │  L7 load balancing (3 AZs)
>               └────────────┬────────────┘
>                            │
>               ┌────────────▼────────────┐
>               │   EKS (Microservices)   │  catalog · cart · orders · payments
>               │   Auto-scales per svc   │  (stateless → scale horizontally)
>               └───┬───────────┬─────────┘
>                   │           │
>           ┌───────▼──┐   ┌────▼─────────┐   ┌──────────────┐
>           │ElastiCache│   │ Aurora Global│   │  SQS (async  │
>           │  (Redis)  │   │   Database   │   │ order queue) │
>           │  cache    │   │ primary +    │   └──────────────┘
>           └───────────┘   │ cross-region │
>                           │ read replicas│
>           ┌───────────────▼──────────────┐
>           │  DynamoDB Global Tables       │  cart (active-active, multi-region)
>           └───────────────────────────────┘
>           ┌───────────────────────────────┐
>           │  S3 — product images / uploads │
>           └───────────────────────────────┘
> ```
>
> **Edge (global reach):** Route53 uses latency-based routing to send each user to the nearest region. CloudFront caches static content at edge locations worldwide, so international users get fast load times. WAF blocks SQL injection, XSS, and abusive traffic before it reaches the app.
>
> **Compute (scalable):** ALB distributes traffic across EKS microservices — catalog, cart, orders, payments — each scaling independently. During flash sales, only the services under load scale up. Stateless compute means easy horizontal scaling.
>
> **Data (robust + international):**
> - **Aurora Global Database** — primary region for writes, cross-region read replicas for low-latency reads and DR (sub-second replication). Strong consistency for orders/payments.
> - **ElastiCache (Redis)** — caches sessions and the product catalog, cutting DB load significantly.
> - **DynamoDB Global Tables** — the shopping cart, replicated active-active across regions so a user never loses their cart even if a region fails.
> - **S3** — product images and user uploads.
>
> **Async (resilient):** SQS decouples order placement from fulfillment — the checkout page returns instantly while order processing happens in the background.
>
> **Key design decisions:**
> - Stateless compute → horizontal scaling during peak sales
> - Aurora Global DB → strong consistency for money, low-latency reads globally
> - DynamoDB Global Tables for cart → survives a full region outage
> - SQS async → fast checkout, background fulfillment"

**Optional add-ons (mention only if asked):**
- **DDoS at scale** → AWS Shield Advanced (on top of WAF)
- **External partner APIs** → API Gateway (rate limiting, API keys)
- **Event fan-out** (order → inventory + notification + analytics) → EventBridge + Lambda
- **Service-to-service security** → Istio mTLS

---

### Q: What is the difference between a monolith and microservices?

**Project Reference:** P1 (microservices), P3 (Kubernetes platform)

**Answer:**

> "**Monolith** — the entire application is built and deployed as one single unit. All features (UI, business logic, data access) live in one codebase and run as one process.
>
> **Microservices** — the application is split into small, independent services, each owning one business capability (catalog, cart, orders, payments). Each runs, scales, and deploys separately, communicating over APIs.
>
> | Aspect | Monolith | Microservices |
> |---|---|---|
> | Deployment | One unit — redeploy everything for any change | Deploy each service independently |
> | Scaling | Scale the whole app | Scale only the service under load |
> | Tech stack | One shared stack | Each service can use its own language/DB |
> | Failure blast radius | A bug can crash the whole app | Failure isolated to one service |
> | Data | Usually one shared database | Database per service |
> | Complexity | Simple to start, harder at scale | Operationally complex (needs CI/CD, monitoring, service mesh) |
> | Team fit | Small teams | Large teams (each owns a service) |
>
> **Analogy:** A monolith is one big store where everything happens under one roof — if it closes, everything stops. Microservices are like a mall of specialized shops — one can close for renovation while the rest stay open.
>
> **Rule of thumb:** Start with a monolith, then split into microservices when you feel the pain — deployment bottlenecks, team conflicts, or uneven scaling needs."

---

### Q: Under what circumstances would you choose between a monolith and microservices/container-based systems?

**Project Reference:** P1 (microservices), P3 (Kubernetes platform)

**Note:** Different from existing "difference between monolith and microservices" — this asks WHEN to choose which.

**Answer:**

> "**Choose Monolith when:**
> - Small team (3-5 developers) — microservices add overhead without enough people to own each service
> - Early-stage product — requirements changing rapidly, boundaries unclear. Don't prematurely split.
> - Simple domain — not enough complexity to justify distributed system overhead
> - Time-to-market is critical — monolith ships faster initially
>
> **Choose Microservices when:**
> - Large team (10+) — each team owns a service independently. No stepping on each other.
> - Independent scaling needed — one service handles 10x more traffic than others
> - Independent deployment needed — can't afford to redeploy everything for a one-line change
> - Different tech stacks per service — Python for ML, Go for API, Node for real-time
> - Failure isolation — a bug in search shouldn't crash checkout
>
> **My rule of thumb:** Start monolith, split when you feel the pain (deployment bottleneck, team conflicts, scaling limits). Don't start with microservices unless you have the team size and infra maturity (CI/CD, monitoring, service mesh) to support it."

---

### Q: With a large team (10-12 developers), which architecture would you choose and why?

**Project Reference:** P1 (15+ microservices), P3 (Kubernetes platform)

**Answer:**

> "**Microservices** — for a 10-12 person team, without question.
>
> **Why:**
> 1. **Team ownership** — Split into 3-4 squads, each owning 2-3 services. Clear boundaries, no merge conflicts across teams.
> 2. **Independent deployments** — Team A deploys their service without waiting for Team B to finish their feature. Velocity stays high.
> 3. **Blast radius** — If Team A introduces a bug, only their service is affected. Not the entire application.
> 4. **Independent scaling** — The catalog service gets 100x traffic during Black Friday, but the admin panel stays the same. Scale only what needs scaling.
>
> **But you MUST have:**
> - CI/CD per service (ArgoCD + Helm — P1)
> - Observability (Prometheus + Jaeger — can't debug distributed systems without tracing)
> - Service mesh or API gateway (Istio — P6) for service-to-service communication
> - Clear API contracts between teams
>
> Without this supporting infrastructure, microservices with 12 developers becomes chaos. With it — it's fast, scalable, and resilient."

---

### Q: How would you monitor this ecosystem — native AWS services or external tools?

**Project Reference:** P3 (Observability stack — Prometheus/Grafana/Jaeger)

**Answer:**

> "**Hybrid approach** — AWS native for infrastructure, external for application:
>
> | Layer | Tool | Why |
> |---|---|---|
> | Infrastructure (EC2, RDS, ALB) | CloudWatch | Native, no agent needed, built-in alarms |
> | Kubernetes (pods, nodes) | Prometheus + Grafana | Richer metrics, PromQL, custom dashboards, ServiceMonitor auto-discovery |
> | Distributed tracing | Jaeger (or X-Ray) | Request-level visibility across microservices |
> | Logs | EFK (Elasticsearch + Fluentd + Kibana) | Structured search across all services, better than CloudWatch Logs for K8s |
> | Alerting | AlertManager | Tiered routing: P1→PagerDuty, P2→Slack, P3→Jira |
> | Cost | Kubecost + AWS Cost Explorer | Per-namespace cost visibility + account-level trends |
>
> **Why not pure CloudWatch?** It works for EC2/RDS metrics but is limited for Kubernetes observability. No native service mesh metrics, no distributed tracing correlation, PromQL is far more powerful than CloudWatch Metrics Insights for custom queries."

---

### Q: If CloudWatch and X-Ray have the required features, why would you use DataDog?

**Project Reference:** P3 (Observability stack)

**Answer:**

> "Honestly — if CloudWatch + X-Ray genuinely meet your needs, you DON'T need DataDog. Save the money.
>
> **But people choose DataDog when:**
>
> 1. **Single pane of glass** — DataDog correlates metrics, traces, and logs in one view. With AWS, you're jumping between CloudWatch, X-Ray, and CloudWatch Logs — different UIs, different query languages.
>
> 2. **Multi-cloud or hybrid** — If you have workloads on AWS + Azure + on-prem, DataDog gives unified visibility. CloudWatch only sees AWS.
>
> 3. **Better Kubernetes support** — DataDog's K8s integration is richer than CloudWatch Container Insights. Live container view, real-time pod mapping, APM correlation.
>
> 4. **Team experience** — If your team already knows DataDog, the learning curve for CloudWatch Metrics Insights + X-Ray is time wasted.
>
> **My position:** We use Prometheus + Grafana (open-source, no vendor lock-in) for Kubernetes, and CloudWatch for AWS-native services. DataDog is great but expensive at scale — $15-30/host/month adds up fast with 500+ servers."

---

### Q: How would you ensure the application isn't impacted if a region or AZ goes down?

**Project Reference:** P8 (Multi-Region HA/DR), P2 (3-Tier Multi-AZ)

**Answer:**

> "Two levels — AZ failure and Region failure:
>
> **AZ Failure (common, design for this always):**
> - Deploy across 3 AZs minimum. ALB auto-routes to healthy AZs.
> - ASG/EKS nodes spread across AZs. One AZ dies → others absorb traffic.
> - Aurora Multi-AZ — automatic failover in <30 seconds.
> - Result: Zero user impact. Automatic recovery.
>
> **Region Failure (rare but catastrophic, active-passive DR):**
> - **Route53 health checks** — detect primary region failure (endpoint unhealthy for 3 consecutive checks = ~90 seconds)
> - **DNS failover** — Route53 switches traffic to DR region
> - **Aurora Global Database** — secondary region promotes to primary (~60 seconds)
> - **Warm pool in DR** — pre-baked AMIs, scaled-down but ready to scale up
> - **S3 Cross-Region Replication** — static assets available in both regions
>
> **Our results (P8):** 3-minute RTO, <1-second RPO. Validated quarterly through automated DR drills using AWS FIS (Fault Injection Simulator)."

---

### Q: What are five key cost optimization measures for this environment?

**Project Reference:** P7 (Cost Optimization / FinOps)

**Answer:**

> 1. **Right-sizing** — 70% of instances run at <15% CPU. Downsize m5.xlarge → m5.large. Use VPA recommendations. We saved 30% compute cost from rightsizing alone.
>
> 2. **Non-prod auto-stop** — Dev/staging environments stop at 7 PM, start at 8 AM. Weekends off. Lambda + EventBridge schedule. **65% savings on non-prod compute**.
>
> 3. **Spot/Karpenter for K8s** — Karpenter provisions right-sized Spot instances for non-critical workloads. 60-70% cheaper than on-demand. Critical services stay on On-Demand.
>
> 4. **Savings Plans + Reserved** — Compute Savings Plans for baseline (always-on prod). 40% discount vs on-demand with flexibility across instance types.
>
> 5. **Data transfer optimization** — VPC endpoints for S3/DynamoDB (eliminate NAT gateway data processing charges — $0.045/GB adds up fast). Keep traffic in-AZ where possible.
>
> **Bonus:** ECR lifecycle policies (delete images older than 30 days), S3 Intelligent-Tiering for logs, mandatory tagging via SCPs (untagged = auto-terminated)."

---

### Q: How would you establish connectivity between on-premises systems and the AWS e-commerce application?

**Project Reference:** P4 (Landing Zone — network architecture)

**Answer:**

> "Depends on requirements — bandwidth, latency, cost, redundancy:
>
> | Option | Bandwidth | Latency | Cost | Use Case |
> |---|---|---|---|---|
> | **Site-to-Site VPN** | Up to 1.25 Gbps | Variable (internet) | Low (~$0.05/hr) | Quick setup, acceptable for non-critical |
> | **AWS Direct Connect** | 1-100 Gbps | Consistent, low | High (port + cross-connect) | Production workloads, compliance requirements |
> | **Direct Connect + VPN backup** | Hybrid | Best of both | Medium | HA — DX primary, VPN failover |
>
> **What I'd recommend for e-commerce:**
> - **Direct Connect (primary)** — consistent latency for real-time inventory sync between warehouse (on-prem) and AWS
> - **Site-to-Site VPN (backup)** — if DX link fails, VPN kicks in automatically
> - **Transit Gateway** — central hub connecting VPN, DX, and multiple VPCs. Single attachment point.
>
> **Security:** Traffic encrypted in transit (VPN = IPsec, DX = add MACsec or overlay VPN). Private VIF for VPC access, public VIF only if accessing AWS public services directly."

---

### Q: Strategy for database caching — ElastiCache vs DynamoDB Global Tables?

**Project Reference:** P2 (3-Tier), P8 (Multi-Region)

**Answer:**

> "They solve **different problems** — not interchangeable:
>
> | Aspect | ElastiCache (Redis) | DynamoDB Global Tables |
> |---|---|---|
> | **Purpose** | Caching layer — reduce DB reads | Primary data store — multi-region active-active |
> | **Data type** | Ephemeral — cache product catalog, sessions | Persistent — cart items, user preferences |
> | **Multi-region** | Redis Global Datastore (read-only replicas) | Active-active writes in any region |
> | **Consistency** | Eventually consistent cache | Eventually consistent across regions (ms) |
> | **Failure mode** | Cache miss → hit DB (slight latency increase) | Region fails → other region handles writes |
>
> **For e-commerce, I'd use both:**
> - **ElastiCache Redis** — Cache product details, search results, session tokens. Cache hit = 1ms response. Cache miss = 50ms (hit Aurora). Reduces Aurora read load by 80%.
> - **DynamoDB Global Tables** — Shopping cart. Active-active so a user in Europe writes to eu-west-1, user in US writes to us-east-1. No failover needed for cart data.
>
> **Key insight:** Cache (ElastiCache) is for SPEED. Global Tables is for AVAILABILITY. Use cache for read-heavy, Global Tables for write-heavy multi-region data."

---

---
---

# SECTION 30: AWS Environment & Account Management

---

### Q: How do you manage different AWS environments (Dev, Staging, QA, Prod)?

**Project Reference:** P4 (Multi-Account Landing Zone — 15 accounts, 5 OUs)

**Answer:**

> "**Separate AWS accounts per environment** — not just separate VPCs in one account.
>
> ```
> AWS Organization
> ├── Management OU (billing, SCPs, SSO)
> ├── Security OU (GuardDuty, CloudTrail, Security Hub)
> ├── Shared Services OU (CI/CD, DNS, Transit Gateway)
> ├── Workloads OU
> │   ├── Dev Account
> │   ├── Staging Account
> │   └── Prod Account
> └── Sandbox OU (experiments)
> ```
>
> **Why separate accounts (not just VPCs):**
> 1. **Blast radius** — Dev mistake can't touch Prod. Account = hard boundary.
> 2. **Cost separation** — Per-account billing. Clear chargeback.
> 3. **IAM isolation** — Dev admin can't accidentally modify Prod resources.
> 4. **Service limits** — One environment hitting API limits doesn't affect others.
> 5. **Compliance** — Audit Prod independently. SOC2 scope is Prod account only.
>
> **How environments connect:**
> - Transit Gateway for cross-account networking (if needed)
> - CI/CD pipeline assumes roles into target account via OIDC (P4)
> - Same Terraform code, different tfvars per account"

---

### Q: How do you differentiate between Dev, QA, and Production accounts in your pipeline?

**Project Reference:** P1 (DevSecOps Pipeline), P4 (Landing Zone — OIDC)

**Answer:**

> "**Branch-based targeting + cross-account IAM roles:**
>
> ```groovy
> // Jenkinsfile logic
> if (env.BRANCH_NAME == 'develop') {
>     targetAccount = '111111111111'  // Dev account
>     targetCluster = 'eks-dev'
>     role = 'arn:aws:iam::111111111111:role/cicd-deploy'
> } else if (env.BRANCH_NAME =~ /release\\/.*/) {
>     targetAccount = '222222222222'  // Staging account
>     targetCluster = 'eks-staging'
>     role = 'arn:aws:iam::222222222222:role/cicd-deploy'
> } else if (env.BRANCH_NAME == 'main') {
>     targetAccount = '333333333333'  // Prod account
>     targetCluster = 'eks-prod'
>     role = 'arn:aws:iam::333333333333:role/cicd-deploy'
> }
> ```
>
> **The mechanism:**
> - Pipeline assumes different IAM role per environment (OIDC federation — no static keys)
> - Each role only has permissions for its own account
> - Prod role additionally requires manual approval gate before `apply`
> - Same Terraform/Helm code → different role → different account → different cluster
>
> **Guardrails:**
> - SCPs block certain actions in Prod (no `ec2:TerminateInstances` without tag, no public S3 buckets)
> - Prod pipeline has extra stages: canary, manual approval, extended smoke tests"

---

### Q: Do you have your Dev and Production clusters in the same AWS account?

**Project Reference:** P4 (Multi-Account Landing Zone)

**Answer:**

> "**No — absolutely not.** Separate accounts for Dev and Prod.
>
> **Why not same account:**
> - Developer with Dev EKS permissions could accidentally target Prod cluster (wrong kubeconfig context)
> - A runaway Dev workload consuming resources could hit account-level service limits affecting Prod
> - IAM policy mistakes in Dev could expose Prod resources
> - Compliance auditors don't like Dev and Prod in same account (SOC2 scope becomes the entire account)
>
> **Our setup:**
> - Dev EKS → Dev AWS account
> - Staging EKS → Staging AWS account  
> - Prod EKS → Prod AWS account
> - Each account has its own VPC, its own IAM roles, its own state file
>
> **The only shared resources:** ECR (container images — pulled cross-account via resource policy), Transit Gateway (for cross-env networking if needed), and the CI/CD pipeline (lives in shared-services account, assumes roles into target accounts).
>
> Same account for Dev+Prod is a startup shortcut that becomes a liability at enterprise scale."

---

### Q: What steps would you take to control costs if a customer's EKS/Fargate/ECS costs are rising?

**Project Reference:** P7 (Cost Optimization / FinOps)

**Answer:**

> "Systematic approach — identify waste, then optimize:
>
> 1. **Kubecost / Cost Explorer** — First, understand WHERE money is going. Per-namespace, per-service cost breakdown. Usually 2-3 services are 80% of the bill.
>
> 2. **Rightsizing pods** — VPA (Vertical Pod Autoscaler) recommendations. Most pods over-request CPU/memory. Requesting 1 CPU but using 0.1 CPU = 90% waste. Adjust requests/limits.
>
> 3. **Karpenter instead of managed node groups** — Karpenter picks optimal instance types per workload (mix of sizes). No over-provisioned nodes. Consolidates pods onto fewer, fuller nodes.
>
> 4. **Spot instances for non-critical** — Dev/staging: 100% Spot (70% savings). Prod: Spot for stateless workers, On-Demand for critical. Karpenter handles Spot diversification.
>
> 5. **Scale to zero (non-prod)** — Dev cluster scales to 0 nodes at 7 PM, back up at 8 AM. Karpenter handles this via empty namespaces = no pods = no nodes.
>
> 6. **Fargate-specific:** Check if tasks are oversized. Fargate charges per vCPU/memory-hour. A 4 vCPU task using 0.5 vCPU = paying 8x too much. Rightsize task definitions.
>
> 7. **Savings Plans** — Compute Savings Plans cover EKS/Fargate/ECS. Commit to baseline for 40% discount.
>
> **Our result (P7):** 35% cost reduction ($180K/year) through Karpenter + rightsizing + non-prod auto-stop + Savings Plans."

---
---

# SECTION 39: AWS Infrastructure (Volumes, Scaling, Encryption)

---

### Q: What are the different types of EC2 volumes and their primary use cases?

**Project Reference:** P2 (3-Tier AWS), P7 (Cost Optimization — GP2→GP3 migration)

**Answer:**

> | Volume Type | IOPS | Throughput | Use Case |
> |---|---|---|---|
> | **gp3** (General Purpose SSD) | 3,000 baseline (up to 16,000) | 125 MiB/s (up to 1,000) | Default choice — boot volumes, app workloads. Independent IOPS/throughput scaling. **What we use.** |
> | **gp2** (older General Purpose) | 3 IOPS/GB (burst to 3,000) | 250 MiB/s max | Legacy — migrate to gp3 (20% cheaper, better performance). |
> | **io2/io2 Block Express** | Up to 256,000 | Up to 4,000 MiB/s | High-performance databases (production Aurora on EC2, SAP). Expensive. |
> | **st1** (Throughput HDD) | N/A | Up to 500 MiB/s | Big data, log processing, sequential reads. Can't be boot volume. |
> | **sc1** (Cold HDD) | N/A | Up to 250 MiB/s | Archival, infrequent access. Cheapest. Can't be boot volume. |
>
> **In our environment:**
> - **gp3** for everything (app servers, K8s worker nodes, databases)
> - We migrated from gp2 → gp3 in P7 cost optimization: 20% cost reduction + independent IOPS tuning. No downtime — modify volume type online.
> - **io2** only for the most critical database workloads requiring guaranteed high IOPS
>
> **Key tip:** gp3 is almost always the right choice. It's cheaper than gp2, has better baseline performance, and you can independently scale IOPS and throughput without changing volume size."

---

### Q: How would you handle data encryption in transit and at rest? What is the difference between server-side and client-side encryption?

**Project Reference:** P2 (3-Tier), P6 (Istio mTLS), P1 (DevSecOps)

**Answer:**

> "**Encryption in transit:** Data protected while moving between systems.
> - HTTPS/TLS between client and ALB (ACM certificate)
> - mTLS between microservices (Istio — P6)
> - SSL for database connections (RDS `require_ssl` parameter)
> - VPN/Direct Connect encryption for hybrid connectivity
>
> **Encryption at rest:** Data protected while stored.
> - EBS volumes: encrypted with AWS KMS key
> - S3: server-side encryption (SSE-S3, SSE-KMS, or SSE-C)
> - RDS: encrypted storage + automated backups encrypted
> - EKS Secrets: encrypted in etcd with KMS envelope encryption
>
> **Server-side vs Client-side encryption:**
>
> | Aspect | Server-side (SSE) | Client-side (CSE) |
> |---|---|---|
> | **Who encrypts** | AWS service (S3, RDS, EBS) | Your application before sending to AWS |
> | **Key management** | AWS manages (KMS) or you provide key | You fully manage keys |
> | **Data in transit to AWS** | Arrives unencrypted (TLS protects transport) | Arrives already encrypted |
> | **Trust model** | Trust AWS with your data momentarily | AWS never sees plaintext |
> | **Use case** | Most workloads (simpler) | Highly regulated (PCI, healthcare — zero-trust on cloud provider) |
>
> **What we use:** Server-side encryption for everything (SSE-KMS). Simpler to manage, KMS provides audit trail (CloudTrail logs every key usage). Client-side only if compliance mandates 'provider must never see plaintext' — rare."

---

### Q: Why is auto-scaling necessary, and what are the different types available?

**Project Reference:** P2 (3-Tier — ASG), P3 (Kubernetes — HPA + Karpenter)

**Answer:**

> "**Why necessary:**
> - Traffic is unpredictable. Without auto-scaling: over-provision (waste money) or under-provision (users get errors).
> - Auto-scaling = right capacity at all times. Scale up for demand, scale down to save costs.
> - Also handles fault tolerance: instance crashes → ASG replaces it automatically.
>
> **Types in AWS:**
>
> | Type | What It Scales | Trigger |
> |---|---|---|
> | **EC2 Auto Scaling (ASG)** | EC2 instances | Target tracking (CPU 70%), step scaling, scheduled |
> | **EKS — HPA** | Pod replicas | CPU, memory, or custom metrics (requests/sec) |
> | **EKS — Karpenter** | Worker nodes | Pending pods (no capacity → add nodes) |
> | **EKS — VPA** | Pod resource requests | Historical usage (adjusts CPU/memory requests) |
> | **Aurora Auto Scaling** | Read replicas | CPU utilization or connections |
> | **DynamoDB Auto Scaling** | Read/write capacity | Consumed capacity vs provisioned |
> | **Application Auto Scaling** | ECS tasks, Lambda concurrency, etc. | CloudWatch metrics |
>
> **Our setup:**
> - EC2: ASG with target tracking (maintain 70% average CPU). Min 3, Max 20.
> - K8s pods: HPA based on CPU (scale pods)
> - K8s nodes: Karpenter (scale infrastructure under pods)
> - Karpenter is smarter than Cluster Autoscaler — picks optimal instance type, handles Spot, consolidates underutilized nodes."

---

### Q: What is connection draining, and what does it do when an application instance is terminated?

**Project Reference:** P2 (3-Tier — ALB), P9 (OS Patching — traffic drain)

**Answer:**

> "**Connection draining** (called 'deregistration delay' in ALB) gives in-flight requests time to complete before an instance is removed from the load balancer.
>
> **What happens without it:**
> - Instance terminated immediately → active connections severed → users get 502 errors mid-request
>
> **What happens with connection draining:**
> 1. Instance marked for termination (scaling-in, health check failure, or patching)
> 2. ALB stops sending NEW requests to that instance
> 3. Existing in-flight requests continue until they complete (or timeout — default 300 seconds)
> 4. Once all connections drain (or timeout hits) → instance terminated safely
>
> **In our OS patching (P9):**
> - Before patching a server, we deregister it from the ALB target group
> - Wait for connection draining (we set 60 seconds — our requests are short-lived)
> - Verify zero active connections: `ss -tn | grep ESTABLISHED | wc -l`
> - THEN start patching. No user impact.
> - After patching + validation → re-register to target group → ALB starts sending traffic again
>
> **Key settings:**
> - `deregistration_delay.timeout_seconds = 60` (default 300 — too long for most apps)
> - For WebSocket/long-polling apps: set higher (match your longest expected connection)"

---
---

# SECTION 51: Deployment Operations & AWS Details

---

### Q: How big is your Ansible inventory?

**Project Reference:** P9 (OS Patching — 500+ servers)

**Answer:**

> "**500+ RHEL servers** managed through Ansible Automation Platform.
>
> **Inventory structure:**
> ```ini
> [webservers]        # ~150 servers
> web-us-east-[01:50]
> web-us-west-[01:50]
> web-eu-west-[01:50]
>
> [appservers]        # ~200 servers
> app-us-east-[01:80]
> app-us-west-[01:60]
> app-eu-west-[01:60]
>
> [dbservers]         # ~50 servers
> db-us-east-[01:20]
> ...
>
> [all:vars]
> ansible_user=deploy
> ansible_ssh_private_key_file=/path/to/key
> ```
>
> **How we manage at scale:**
> - **Dynamic inventory** — We actually use AWS dynamic inventory plugin (not static files). Pulls EC2 instances by tags: `Role=webserver`, `Environment=production`. Servers added/removed automatically.
> - **Grouped by role, region, and environment** — Allows targeting like `ansible-playbook patch.yml --limit 'webservers:&us-east:&production'`
> - **AAP (Ansible Automation Platform)** — Manages inventory sync, credentials, scheduling. Not running from a laptop.
>
> At this scale, static inventory files are impossible. Tags + dynamic inventory = always current."

---

### Q: How frequently is your backup?

**Project Reference:** P8 (Multi-Region DR), P2 (Aurora)

**Answer:**

> "Depends on the tier:
>
> | Component | Backup Frequency | Retention | RPO |
> |---|---|---|---|
> | **Aurora DB** | Continuous (automated) | 7 days point-in-time, daily snapshots for 30 days | 5 minutes (PITR) |
> | **Aurora Global** | Continuous replication | <1 second lag to DR region | <1 second |
> | **EBS volumes** | Daily snapshots (via AWS Backup) | 30 days | 24 hours |
> | **S3 data** | Versioning + CRR | Infinite (versioned), cross-region replica | Near-zero |
> | **Kubernetes (Velero)** | Every 6 hours | 7 days | 6 hours |
> | **Terraform state** | S3 versioning (every change) | 90 days | Per-change |
> | **Configuration (Git)** | Every commit | Indefinite | Per-commit |
>
> **Key principles:**
> - Critical data (DB) = continuous. Can restore to any second in last 7 days.
> - Infrastructure config = Git is the backup. Terraform + Helm charts in Git = reproducible infra.
> - We test restores quarterly as part of DR drills (P8). Backup that's never tested = no backup."

---

### Q: What are a few limitations of AWS Lambda?

**Project Reference:** P5 (Serverless — Lambda remediation engine)

**Answer:**

> "Key limits that affect architecture decisions:
>
> | Limit | Value | Impact |
> |---|---|---|
> | **Execution timeout** | 15 minutes max | Can't run long-running tasks |
> | **Memory** | 128 MB – 10 GB | Limits data processing size |
> | **Package size** | 50 MB (zip), 250 MB (unzipped) | Large ML models don't fit |
> | **Concurrent executions** | 1000 default (soft limit, raiseable) | Can throttle under high load |
> | **Cold start** | 100ms–2s (language dependent) | Latency-sensitive APIs suffer |
> | **/tmp storage** | 512 MB (10 GB with ephemeral storage) | Limited scratch space |
> | **No persistent state** | Stateless between invocations | Need external DB/cache |
>
> **Real limitations I've hit:**
> - Cold starts with Python + boto3 = ~800ms first invocation. Fine for async remediation (P5), bad for user-facing APIs.
> - 15-min timeout: our patching validation script takes 20 min → can't run in Lambda → runs on EC2 via Ansible instead.
> - Concurrency limits: during an AWS Config rule evaluation across 500 resources, Lambda throttled. Had to implement SQS buffering.
>
> **When Lambda is wrong:** Long-running, stateful, high-throughput, or latency-critical workloads. Use ECS/EKS instead."

---

### Q: If something has to run for 16 minutes, what would you suggest?

**Project Reference:** P5 (Serverless), P3 (Kubernetes)

**Answer:**

> "Lambda max is 15 minutes. For 16+ minute tasks, alternatives:
>
> 1. **AWS Step Functions + Lambda** — Break the task into smaller steps (<15 min each). Step Functions orchestrates the chain. Each step is a Lambda. State passed between steps.
>
> 2. **ECS Fargate task** — Run a container for as long as needed (no timeout). Fire-and-forget. Pay per second of compute. Good for batch jobs.
>
> 3. **EKS Job** — Kubernetes Job runs to completion. Can run hours. Already have the cluster.
>
> 4. **AWS Batch** — For compute-intensive batch workloads. Manages queue + compute environment.
>
> **My recommendation depends on the workload:**
> - One-off data processing → **Fargate task** (simple, no infra management)
> - Complex multi-step workflow → **Step Functions** (orchestration, error handling, retries)
> - Already on K8s → **K8s Job** (no new service to manage)
> - Frequently recurring → **AWS Batch** (managed queue + auto-scaling compute)
>
> **Key point:** Don't try to hack around Lambda's 15-min limit (breaking tasks artificially, chaining Lambdas via SNS). Use the right compute model for the workload duration."

---

### Q: How would the autoscaling group know which AMI to use?

**Project Reference:** P2 (3-Tier — ASG + Launch Template)

**Answer:**

> "**Launch Template** tells the ASG everything about how to create instances — including the AMI.
>
> ```hcl
> resource \"aws_launch_template\" \"app\" {
>   image_id      = \"ami-0abc123def456\"   # AMI ID here
>   instance_type = \"m5.large\"
>   key_name      = \"deploy-key\"
>   
>   user_data = base64encode(file(\"bootstrap.sh\"))
>   
>   iam_instance_profile {
>     name = aws_iam_instance_profile.app.name
>   }
> }
>
> resource \"aws_autoscaling_group\" \"app\" {
>   launch_template {
>     id      = aws_launch_template.app.id
>     version = \"$Latest\"
>   }
>   min_size = 3
>   max_size = 20
> }
> ```
>
> **AMI lifecycle in our setup:**
> 1. Packer builds a new AMI monthly (base OS + patches + app runtime + CloudWatch agent)
> 2. Terraform updates the `image_id` in Launch Template → new version created
> 3. ASG Instance Refresh → gradually replaces old instances with new AMI
> 4. Old instances terminated after traffic drains
>
> **Key point:** ASG doesn't 'know' the AMI — Launch Template defines it. To update the AMI, update the Launch Template and trigger an instance refresh."

---

### Q: How long does it take for a new instance to come up and be active? How can you minimize that?

**Project Reference:** P2 (3-Tier — ASG), P8 (DR — warm pool)

**Answer:**

> "**Typical time: 3-5 minutes** (instance launch + boot + app start + health check pass).
>
> **Breakdown:**
> - Instance launch: ~30-60 seconds (AWS provisioning)
> - Boot + cloud-init: ~60-90 seconds (OS boot, run userdata script)
> - Application startup: ~30-120 seconds (JVM warmup, DB connection pool, cache priming)
> - Health check: 30 seconds (ALB checks every 10s, needs 3 consecutive successes)
>
> **How to minimize:**
>
> 1. **Pre-baked AMI (golden AMI)** — Install everything at AMI build time (Packer). Userdata only does config (10 seconds vs 3 minutes of `yum install` at boot).
>
> 2. **Warm pool** — ASG keeps pre-initialized instances in 'Stopped' state. When needed: start (not launch from scratch). Saves 2-3 minutes. We use this for DR (P8).
>
> 3. **Predictive scaling** — Scale BEFORE the traffic hits (ML-based prediction from historical patterns). Instances ready before demand arrives.
>
> 4. **Reduce health check thresholds** — If app boots in 30 seconds, set health check grace period to 60 seconds (not 300). But don't make it too aggressive — false positives kill instances prematurely.
>
> 5. **Smaller instances** — Smaller = faster boot. Graviton instances boot marginally faster.
>
> **For Kubernetes:** Karpenter provisions nodes in ~30 seconds (faster than ASG). Pod scheduling on existing nodes: <5 seconds."

---

### Q: At what percentage do you scale your application?

**Project Reference:** P2 (3-Tier — ASG target tracking), P3 (Kubernetes — HPA)

**Answer:**

> "**70% CPU utilization** is our target tracking threshold for both ASG and HPA.
>
> **Why 70% and not higher:**
> - Scaling takes time (3-5 min for EC2, 30s for pods). Need headroom for traffic spikes while new capacity comes online.
> - At 90% → by the time new instances are ready, existing ones might be at 100% → degraded user experience.
> - 70% gives ~30% buffer for burst absorption.
>
> **Our configuration:**
> ```hcl
> # ASG - EC2
> target_tracking_configuration {
>   predefined_metric_specification {
>     predefined_metric_type = \"ASGAverageCPUUtilization\"
>   }
>   target_value = 70.0
> }
> ```
>
> ```yaml
> # HPA - Kubernetes
> metrics:
> - type: Resource
>   resource:
>     name: cpu
>     target:
>       type: Utilization
>       averageUtilization: 70
> ```
>
> **Exceptions:**
> - Non-critical/batch workloads: scale at 85% (cost efficient, latency less important)
> - Latency-sensitive APIs: scale at 50% (more aggressive — always have spare capacity)
> - Custom metrics: sometimes scale on request count or queue depth instead of CPU (more accurate for I/O-bound apps)"

---

---
---

# SECTION 22: AWS Networking

---

### Q: What is an Internet Gateway?

**Project Reference:** P2 (3-Tier AWS Architecture — VPC)

**Answer:**

> "An **Internet Gateway (IGW)** is a VPC component that allows communication between your VPC and the internet. It's horizontally scaled, redundant, and highly available — AWS manages it.
>
> **What it does:**
> 1. Provides a target in route tables for internet-bound traffic
> 2. Performs NAT (Network Address Translation) for instances with public IPs — translates private IP ↔ public IP
>
> **How it works in our architecture (P2):**
> - Public subnets have a route: `0.0.0.0/0 → igw-xxxxx` — traffic to internet goes through IGW
> - Private subnets do NOT route to IGW directly — they route to NAT Gateway (which then uses IGW)
> - ALB sits in public subnet → reachable from internet via IGW
> - App servers in private subnet → NOT directly reachable from internet (security)
>
> **Key facts:**
> - One IGW per VPC (not per subnet)
> - No bandwidth limit (AWS manages scaling)
> - No cost for IGW itself (you pay for data transfer)
> - Without IGW, nothing in your VPC can reach the internet (or be reached from it)
>
> **IGW vs NAT Gateway:**
> - IGW = allows INBOUND + OUTBOUND internet access (for public subnets)
> - NAT GW = allows OUTBOUND only (for private subnets to reach internet without being reachable from outside)"

---
---

# ~~SECTION 23: Terraform Fundamentals~~ → Moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)

---
---

# SECTION 28: AWS Networking (Advanced)

---

### Q: What is the difference between a region, an edge, and a zone?

**Project Reference:** P2 (3-Tier — Multi-AZ), P8 (Multi-Region DR)

**Answer:**

> | Concept | What It Is | Example | Use |
> |---|---|---|---|
> | **Region** | Geographically separate cluster of data centers | us-east-1 (Virginia), eu-west-1 (Ireland) | Disaster recovery, data residency, latency reduction |
> | **Availability Zone (AZ)** | One or more data centers within a region, isolated power/network | us-east-1a, us-east-1b, us-east-1c | High availability within a region |
> | **Edge Location** | CDN point-of-presence for CloudFront/Route53 | 400+ worldwide | Caching static content close to users, low-latency DNS |
>
> **Key facts:**
> - AZs within a region: connected by high-bandwidth, low-latency links (~1ms between AZs)
> - Regions: isolated from each other. Cross-region = over the internet or dedicated connection. Higher latency (~50-100ms)
> - Edge: only for CDN/DNS. Not for compute or databases.
>
> **Our architecture:**
> - HA within region: Multi-AZ (P2) — ASG, ALB, Aurora across 3 AZs
> - DR across regions: Multi-region (P8) — primary us-east-1, DR us-west-2
> - Global performance: CloudFront edge locations serve static content worldwide"

---

### Q: Would you make an application HA within regions or zones? Is cross-region more expensive?

**Project Reference:** P2 (Multi-AZ), P8 (Multi-Region)

**Answer:**

> "**Always start with Multi-AZ (within region).** Only go multi-region if you NEED it.
>
> **Multi-AZ (within region):**
> - Protects against: single data center failure, hardware failure, rack failure
> - Cost: Minimal extra (ALB is multi-AZ by default, ASG spreads free, Aurora Multi-AZ is ~2x single instance)
> - Covers 99% of failure scenarios
>
> **Multi-Region:**
> - Protects against: entire region failure (extremely rare — happened once in 10 years for most regions)
> - Cost: SIGNIFICANTLY more — duplicate compute, database replica, cross-region data transfer ($0.02/GB), management complexity
> - Only needed for: compliance (data residency), latency (global users), regulatory (some industries mandate it)
>
> **Yes, cross-region is much more expensive:**
> - DB: Aurora Global = ~$200-400/month extra minimum
> - Compute: warm pool instances running in DR region
> - Data transfer: S3 CRR, cross-region traffic
> - Operational: managing two regions, DR drills, keeping configs in sync
>
> **My recommendation:** Multi-AZ for everyone (it's table stakes). Multi-region only if business justifies the 2-3x cost increase."

---

### Q: What is a Network Access Control List (NACL)? What's the difference between NACLs and Security Groups?

**Project Reference:** P2 (3-Tier AWS — VPC security)

**Answer:**

> | Aspect | NACL | Security Group |
> |---|---|---|
> | **Level** | Subnet level | Instance/ENI level |
> | **State** | Stateless (must allow both inbound AND outbound) | Stateful (allow inbound → outbound auto-allowed) |
> | **Rules** | Allow AND Deny rules | Allow rules only (implicit deny) |
> | **Evaluation** | Rules evaluated in order (lowest number first) | All rules evaluated together |
> | **Default** | Allows all traffic | Denies all inbound, allows all outbound |
> | **Scope** | Applies to ALL instances in subnet | Only applies to associated instances |
>
> **Practical difference:**
> - NACL: 'Block ALL traffic from IP 1.2.3.4 to this entire subnet' — broad, blunt, subnet-wide
> - Security Group: 'Allow port 443 from ALB to these specific instances' — granular, per-resource
>
> **How we use both (P2):**
> - **NACLs:** Broad deny rules — block known bad CIDR ranges at subnet level. Also restrict inter-subnet traffic (public subnet can't reach DB subnet directly).
> - **Security Groups:** Fine-grained — ALB SG allows 443 from internet. App SG allows 8080 only from ALB SG. DB SG allows 5432 only from App SG.
>
> **Key gotcha:** NACLs are stateless — if you allow inbound port 443, you MUST also allow outbound ephemeral ports (1024-65535) for the response. Security Groups handle this automatically."

---
---

# SECTION 31: Networking & Access

---

### Q: What are public and private subnets, and how do they work?

**Project Reference:** P2 (3-Tier AWS Architecture — VPC)

**Answer:**

> "The difference is simple: **route table determines public vs private.**
>
> | Aspect | Public Subnet | Private Subnet |
> |---|---|---|
> | **Route to internet** | `0.0.0.0/0 → IGW` (Internet Gateway) | `0.0.0.0/0 → NAT Gateway` (or no internet) |
> | **Inbound from internet** | Yes (if SG allows) | No (not directly reachable) |
> | **Outbound to internet** | Yes (directly via IGW) | Yes (via NAT GW — for patches, API calls) |
> | **Public IP** | Instances can have public IP | Instances have private IP only |
> | **Use case** | ALB, bastion host, NAT GW | App servers, databases, K8s workers |
>
> **In our architecture (P2):**
> ```
> VPC (10.0.0.0/16)
> ├── Public subnets (10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24)
> │   └── ALB, NAT Gateway
> ├── Private app subnets (10.0.11.0/24, 10.0.12.0/24, 10.0.13.0/24)
> │   └── EC2/EKS worker nodes
> └── Private data subnets (10.0.21.0/24, 10.0.22.0/24, 10.0.23.0/24)
>     └── Aurora, ElastiCache
> ```
>
> **Security principle:** Minimize public surface. Only the load balancer is internet-facing. App and data layers are private — not reachable from outside."

---

### Q: How will users hit a website if everything is in a private subnet?

**Project Reference:** P2 (3-Tier AWS Architecture)

**Answer:**

> "Users don't hit the private subnet directly. The flow:
>
> ```
> User → Internet → ALB (in public subnet) → App (in private subnet)
> ```
>
> 1. **ALB lives in PUBLIC subnet** — it has a public IP, reachable from internet
> 2. **ALB forwards to app in PRIVATE subnet** — ALB and app are in the same VPC, so they can communicate via private IPs
> 3. **App never exposed to internet** — no public IP, no route from internet to it
>
> **The ALB is the bridge** between public internet and private infrastructure. It's the ONLY entry point. This means:
> - Only port 443 is open to the world (on ALB's Security Group)
> - WAF inspects traffic at ALB level
> - App Security Group allows traffic ONLY from ALB (not from `0.0.0.0/0`)
>
> **For Kubernetes (EKS):** Same concept — AWS Load Balancer Controller creates an ALB in public subnets, routes to pod IPs in private subnets via IP-mode target groups."

---

### Q: Are there other options to connect to private subnet instances besides a Bastion host?

**Project Reference:** P4 (Landing Zone — secure access)

**Answer:**

> "Yes, and we prefer NOT using bastions:
>
> | Option | How It Works | Pros | Cons |
> |---|---|---|---|
> | **SSM Session Manager** | AWS Systems Manager — browser/CLI access without SSH | No open ports, no keys to manage, fully audited in CloudTrail | Requires SSM agent + IAM role |
> | **VPN (Client VPN)** | Tunnel from laptop to VPC | Full network access, secure | Cost, client software needed |
> | **EC2 Instance Connect** | Temporary SSH key pushed for 60 seconds | No permanent keys, short-lived | Only for EC2, needs security group rule |
> | **Bastion/Jump host** | SSH to bastion → SSH to target | Simple, well-understood | Bastion is an attack target, key management |
>
> **What we use: SSM Session Manager.**
>
> Why:
> - No port 22 open anywhere (Security Group has NO SSH inbound rule)
> - No SSH keys to manage or rotate
> - Full audit trail in CloudTrail (who accessed what, when, all commands logged)
> - Works through NAT/private subnets without any inbound connectivity
> - IAM-controlled access (only certain roles can start sessions)
>
> Bastions are legacy. SSM is the modern, secure alternative."

---
---

# SECTION 34: Cloud Security & Networking (Fundamentals)

---

### Q: Do you have any idea how Single Sign-On (SSO) functionality works?

**Project Reference:** P4 (Landing Zone — IAM Identity Center/SSO)

**Answer:**

> "Yes, we implemented it via **AWS IAM Identity Center** (formerly AWS SSO) in our Landing Zone.
>
> **How SSO works (high level):**
> 1. User goes to SSO portal (single URL)
> 2. Authenticates ONCE (username + password + MFA) against an Identity Provider (IdP) — could be Active Directory, Okta, or AWS-managed directory
> 3. SSO portal shows all AWS accounts/roles the user is authorized for
> 4. User clicks 'Prod Account - ReadOnly' → gets temporary credentials (STS AssumeRole) — valid for 1-12 hours
> 5. No permanent access keys. Session expires automatically.
>
> **The protocol behind it:** SAML 2.0 or OIDC. IdP sends a signed assertion to AWS saying 'this user is authenticated and authorized for these roles.'
>
> **In our setup (P4):**
> - AWS IAM Identity Center connected to corporate AD
> - Permission sets define what roles exist (Admin, ReadOnly, Developer, PowerUser)
> - Assignment: User Group X → Permission Set Y → Account Z
> - Result: One login, access to 15 accounts with appropriate roles. No static credentials anywhere."

---

### Q: What is the difference between HTTP and HTTPS?

**Project Reference:** P2 (3-Tier — ALB + TLS)

**Answer:**

> "**HTTP** = plain text communication. Anyone on the network can read the data (passwords, tokens, personal info) in transit.
>
> **HTTPS** = HTTP + TLS encryption. Data is encrypted between client and server. Even if intercepted, it's unreadable.
>
> | Aspect | HTTP | HTTPS |
> |---|---|---|
> | Port | 80 | 443 |
> | Encryption | None | TLS (symmetric + asymmetric crypto) |
> | Certificate | Not needed | Requires SSL/TLS certificate |
> | Speed | Slightly faster (no handshake) | TLS handshake adds ~1 RTT (negligible today) |
> | SEO/Trust | Browsers show 'Not Secure' | Green padlock, required for modern web |
>
> **In production:** We NEVER use HTTP for anything user-facing. Our ALB listens on 443 only. Port 80 exists only to redirect to 443. All internal service-to-service communication uses mTLS (Istio) — encrypted even inside the cluster.
>
> **The TLS handshake (simplified):** Client hello → Server sends certificate → Client verifies cert against CA → Both agree on encryption key → All subsequent traffic is encrypted."

---

### Q: What are routing tables?

**Project Reference:** P2 (3-Tier AWS — VPC networking)

**Answer:**

> "A routing table is a set of rules (routes) that determine where network traffic is directed. Every subnet in a VPC must be associated with a route table.
>
> **Example from our architecture (P2):**
>
> **Public subnet route table:**
> | Destination | Target | Purpose |
> |---|---|---|
> | 10.0.0.0/16 | local | Traffic within VPC stays local |
> | 0.0.0.0/0 | igw-xxxx | Internet-bound traffic → Internet Gateway |
>
> **Private subnet route table:**
> | Destination | Target | Purpose |
> |---|---|---|
> | 10.0.0.0/16 | local | Traffic within VPC stays local |
> | 0.0.0.0/0 | nat-xxxx | Internet-bound traffic → NAT Gateway (outbound only) |
>
> **Key facts:**
> - What makes a subnet 'public' or 'private' is ONLY the route table (whether it routes to IGW or NAT)
> - Each subnet has exactly one route table (but one route table can serve multiple subnets)
> - Most specific route wins (10.0.1.0/24 is more specific than 0.0.0.0/0)
> - `local` route (VPC CIDR) is always there and cannot be removed — ensures intra-VPC communication"

---

### Q: Scenario — You install a web app on EC2, hit the external endpoint, and get an error. What are different types of errors and probable causes?

**Project Reference:** P2 (3-Tier — troubleshooting)

**Answer:**

> "Depends on the error type:
>
> **Connection timeout (no response):**
> - Security Group doesn't allow inbound on port 80/443
> - NACL blocking traffic
> - Instance is in private subnet with no public IP / no route to internet
> - Route table missing route to IGW
> - Application not listening on the expected port (`ss -tlnp` shows nothing on :80)
>
> **Connection refused (immediate rejection):**
> - Application not running (`systemctl status app`)
> - App listening on wrong port or only on 127.0.0.1 (not 0.0.0.0)
> - Firewall on instance (iptables/firewalld) blocking
>
> **HTTP 502 Bad Gateway:**
> - ALB can't reach backend (app crashed, health check failing)
> - App started but not ready yet (still initializing)
>
> **HTTP 503 Service Unavailable:**
> - All targets unhealthy in target group
> - Auto-scaling group has 0 healthy instances
>
> **HTTP 504 Gateway Timeout:**
> - App is too slow to respond (DB connection timeout, deadlock)
> - ALB timeout (default 60s) exceeded
>
> **My troubleshooting order:** Security Group → NACL → Route Table → Instance status → App running? → Port listening? → App logs"

---
---

# SECTION 44: AWS Networking (Deep Dive)

---

### Q: What is the difference between a Route Table and a Security Group?

**Project Reference:** P2 (3-Tier AWS — VPC)

**Answer:**

> "Completely different layers — one handles WHERE traffic goes, the other handles WHETHER traffic is ALLOWED.
>
> | Aspect | Route Table | Security Group |
> |---|---|---|
> | **Function** | Routing — determines where packets go | Firewall — determines if packets are allowed |
> | **Level** | Subnet level | Instance/ENI level |
> | **Answers** | 'Where should traffic for 10.0.2.0/24 be sent?' | 'Is this traffic from port 443 allowed into this instance?' |
> | **Controls** | Next hop (IGW, NAT, VPC peering, Transit GW) | Allow/block by port, protocol, source IP |
> | **Example** | `0.0.0.0/0 → nat-gw` (internet via NAT) | `Allow TCP 443 from 10.0.0.0/16` |
>
> **Order of operations:**
> 1. Packet arrives at subnet → **Route table** decides where to send it (can it even reach the destination?)
> 2. Packet arrives at instance → **Security Group** decides if it's allowed in
>
> **Analogy:**
> - Route table = road signs (telling you which highway to take)
> - Security Group = bouncer at the door (checking if you're on the guest list)
>
> You can have a correct route BUT blocked by SG — traffic won't reach the instance. You can have SG allowing traffic BUT wrong route — traffic never gets there."

---

### Q: How does DNS work, and what steps do you take to expose an application (like xyz.com) to the internet?

**Project Reference:** P2 (3-Tier — Route53 + ALB + CloudFront)

**Answer:**

> "**How DNS works (simplified):**
> 1. User types `xyz.com` → browser asks local DNS resolver
> 2. Resolver checks cache. Miss → asks Root nameserver → `.com` TLD nameserver → authoritative nameserver for `xyz.com`
> 3. Authoritative NS (Route53 for us) returns: `xyz.com → A record → 54.23.xx.xx` (ALB IP)
> 4. Browser connects to that IP
>
> **Steps to expose our app:**
>
> 1. **Register domain** — Route53 or external registrar. Point NS records to Route53 hosted zone.
>
> 2. **Deploy infrastructure** — ALB in public subnet (gets a public DNS name from AWS: `xxx.us-east-1.elb.amazonaws.com`)
>
> 3. **Create DNS record** — Route53 Alias record: `xyz.com → ALB DNS name` (Alias is free, no query charge, supports apex domain)
>
> 4. **SSL certificate** — Request ACM certificate for `xyz.com` + `*.xyz.com`. Validate via DNS (add CNAME record). Attach cert to ALB listener.
>
> 5. **Configure ALB** — Listener on 443 (HTTPS) → forward to target group (EC2/EKS pods). Listener on 80 → redirect to 443.
>
> 6. **Optional: CloudFront** — For global performance. Route53 → CloudFront (edge caching) → ALB (origin).
>
> **Result:** User types `xyz.com` → DNS resolves to ALB → HTTPS connection → ALB routes to healthy backend → response served."

---

### Q: What is transitive peering?

**Project Reference:** P4 (Landing Zone — Transit Gateway)

**Answer:**

> "**Transitive peering is NOT supported in VPC Peering.** That's the key point.
>
> **What it means:**
> - VPC A peers with VPC B. VPC B peers with VPC C.
> - VPC A CANNOT reach VPC C through VPC B. Traffic doesn't 'transit' through B.
> - Each VPC pair needs its own direct peering connection.
>
> **Problem at scale:** With 15 VPCs (our Landing Zone), you'd need N×(N-1)/2 = 105 peering connections. Unmanageable.
>
> **Solution: Transit Gateway (what we use in P4):**
> - Central hub. All VPCs connect to Transit Gateway.
> - Any VPC can reach any other VPC through TGW (transitive routing IS supported)
> - 15 VPCs = 15 attachments (not 105 peering connections)
> - Plus: on-prem VPN/Direct Connect also attaches to TGW — single entry point
>
> ```
> VPC A ──┐
> VPC B ──┼── Transit Gateway ──── On-Prem (VPN)
> VPC C ──┘
> ```
>
> **Bottom line:** VPC Peering = point-to-point, non-transitive. Transit Gateway = hub-and-spoke, transitive. Use TGW for multi-VPC architectures."

---

### Q: How do you establish private communication between an EC2 instance and an RDS database?

**Project Reference:** P2 (3-Tier — App to DB connectivity)

**Answer:**

> "Simple — keep both in the same VPC, in private subnets, and control access via Security Groups:
>
> 1. **Same VPC** — EC2 in private app subnet (10.0.11.0/24), RDS in private data subnet (10.0.21.0/24). Both private — no internet exposure.
>
> 2. **Security Group on RDS** — Allow inbound on port 5432 (PostgreSQL) or 3306 (MySQL) ONLY from the EC2's Security Group ID:
>    ```
>    Inbound: TCP 5432 from sg-app-servers
>    ```
>    NOT from a CIDR range — from the SG itself. This means only instances in that SG can reach the DB.
>
> 3. **RDS endpoint** — App connects to `mydb.cluster-xxx.us-east-1.rds.amazonaws.com:5432`. This resolves to a private IP within the VPC. Never public.
>
> 4. **No public accessibility** — RDS created with `publicly_accessible = false`. No public IP assigned.
>
> 5. **SSL enforced** — RDS parameter: `rds.force_ssl = 1`. Even private traffic is encrypted in transit.
>
> **Result:** EC2 → private IP → RDS. Traffic never leaves the VPC. Not internet-routable. Only the app security group can connect. Encrypted via SSL."

---

### Q: What is the difference between symmetric and asymmetric encryption?

**Project Reference:** P6 (Istio — mTLS), P2 (ALB — TLS)

**Answer:**

> | Aspect | Symmetric | Asymmetric |
> |---|---|---|
> | **Keys** | ONE key (same for encrypt + decrypt) | TWO keys (public encrypts, private decrypts) |
> | **Speed** | Fast (AES-256) | Slow (RSA, ECDSA) |
> | **Use case** | Bulk data encryption (EBS, S3, database) | Key exchange, digital signatures, TLS handshake |
> | **Challenge** | How to share the key securely? | Computationally expensive for large data |
> | **AWS example** | KMS encrypting S3 objects (AES-256) | TLS certificate (RSA/ECDSA for handshake) |
>
> **How TLS uses BOTH (real-world):**
> 1. **Asymmetric** — Initial TLS handshake: server sends public key (in certificate), client encrypts a session key with it, only server can decrypt (has private key)
> 2. **Symmetric** — After handshake: both sides have the shared session key. All subsequent data encrypted with fast symmetric encryption (AES)
>
> **Summary:** Asymmetric solves the key exchange problem (securely agree on a key). Symmetric does the actual fast encryption. Together = TLS.
>
> In our mTLS setup (Istio): every service has its own certificate (asymmetric key pair). Envoy sidecars perform mutual authentication (both sides verify certificates), then switch to symmetric encryption for actual data transfer."

---

### Q: What load balancer services do you know in AWS, and what are the key differences?

**Project Reference:** P2 (3-Tier — ALB), P3 (EKS — NLB)

**Answer:**

> | Type | Layer | Protocol | Use Case |
> |---|---|---|---|
> | **ALB** (Application LB) | Layer 7 | HTTP/HTTPS | Web apps, microservices, path/host routing, WebSocket |
> | **NLB** (Network LB) | Layer 4 | TCP/UDP/TLS | Ultra-low latency, static IP, non-HTTP (databases, gaming, IoT) |
> | **CLB** (Classic LB) | Layer 4+7 | TCP/HTTP | Legacy — don't use for new projects |
> | **GWLB** (Gateway LB) | Layer 3 | IP | Third-party appliances (firewall, IDS — traffic inspection) |
>
> **Key differences:**
>
> | Feature | ALB | NLB |
> |---|---|---|
> | Routing | Path, host, header, query string based | Port-based only |
> | Latency | ~400ms added | ~100μs added (near wire-speed) |
> | Static IP | No (DNS name only) | Yes (Elastic IP per AZ) |
> | WAF integration | Yes | No |
> | SSL termination | Yes (HTTP to backend) | Yes (TLS to TCP to backend) |
> | Target types | Instance, IP, Lambda | Instance, IP |
> | Cost | Per LCU (request-based) | Per NLCU (connection-based) |
>
> **What we use:**
> - **ALB** — 90% of cases. Web apps, APIs, K8s ingress (AWS Load Balancer Controller). Path-based routing, WAF, ACM certs, target group health checks.
> - **NLB** — When we need static IPs (whitelist by IP), extreme performance, or non-HTTP protocols. Used for Istio Ingress Gateway (TCP passthrough to let Istio handle mTLS)."

---

---
---

