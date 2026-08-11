# Interview Prep — JD Coverage & Gaps Master Document

**Role:** Custom Software Engineering Lead / Platform Engineer — AWS  
**Author:** Siddharth Patel  
**Experience:** 10 YOE — DevOps & Cloud Engineer  
**Purpose:** Map every JD requirement to your 9 projects. For gaps, explain concept from scratch with diagrams and talking points.

---

## How to Read This Document

- ✅ **Covered** — You built this. Project reference included.
- ⚠️ **Partial** — You have adjacent experience. Reframing talking point provided.
- ❌ **Gap** — New concept. Full explanation + how it maps to your projects.

---

## Table of Contents

| # | Section | Status |
|---|---------|--------|
| **PART 1** | **Fully Covered — Own These** | |
| 1.1 | AWS Account Vending & Landing Zone | ✅ Project 4 |
| 1.2 | VPC Topology & Networking | ✅ Project 3 |
| 1.3 | IAM, IRSA, KMS, Secrets Manager | ✅ Projects 3, 4 |
| 1.4 | CI/CD Pipeline — Jenkins, ArgoCD, GitOps | ✅ Project 1 |
| 1.5 | Containers, Docker, ECR | ✅ Project 1 |
| 1.6 | Helm Charts & Kubernetes Deployments | ✅ Projects 1, 3 |
| 1.7 | EKS — Cluster, IRSA, HPA, Karpenter | ✅ Projects 3, K8s doc |
| 1.8 | Istio Service Mesh — Sidecar, mTLS, Zero-Trust | ✅ Project 6 |
| 1.9 | Serverless — Lambda, EventBridge, SQS, Step Functions | ✅ Project 5 |
| 1.10 | Multi-Region HA & DR — Route53, Aurora Global, S3 CRR | ✅ Project 8 |
| 1.11 | FinOps & Cost Optimization | ✅ Project 7 |
| 1.12 | Terraform — Modules, Remote State, Environment Gates | ✅ All Projects |
| 1.13 | Security — Zero-Trust, SCPs, Compliance | ✅ Projects 4, 6 |
| 1.14 | Monitoring — Prometheus, Grafana, Jaeger, EFK | ✅ Projects 1, 6 |
| **PART 2** | **Gaps — Learn These This Week** | |
| 2.1 | Amazon Bedrock | ❌ Gap → Maps to Project 5 |
| 2.2 | OpenTelemetry (OTel) & ADOT Collector | ⚠️ Gap → Maps to Project 6 |
| 2.3 | ElastiCache Redis — Cluster Mode & Sharding | ⚠️ Gap → Maps to Project 3 |
| 2.4 | Amazon Neptune Serverless | ❌ Gap → Maps to Projects 3, 4 |
| 2.5 | OpenSearch Serverless | ⚠️ Gap → Maps to Elasticsearch knowledge |
| 2.6 | ECS Fargate + Helm Deployment | ⚠️ Gap → Maps to Project 1 |
| 2.7 | Transit Gateway — On-Prem Peering & Direct Connect | ⚠️ Gap → Maps to Project 4 |
| 2.8 | Route 53 — Private Hosted Zones | ⚠️ Gap → Maps to Project 8 |
| **PART 3** | **Quick Reference — Interview Talking Points** | |
| 3.1 | Project → JD Responsibility Mapping Table | |
| 3.2 | Gap Talking Points (one sentence each) | |
| 3.3 | Numbers to Remember | |

---


---

# PART 1 — Fully Covered Topics

---

## 1.1 AWS Account Vending & Landing Zone
**✅ Project 4 — Multi-Account Landing Zone**

### What is it?
A Landing Zone is a pre-configured, secure, multi-account AWS environment. Account vending means new AWS accounts are created automatically with all security baselines already applied — networking, logging, guardrails — without anyone doing it manually.

### Why it matters
Without it: every team creates their own account, does whatever they want → security holes, cost chaos, impossible to audit.  
With it: every account born with security baseline, SCPs, centralized logging, networking — automatically.

### Architecture
```
AWS Organizations (Root)
├── Management Account (billing only — no workloads)
├── Security OU
│   ├── Audit Account      (CloudTrail, Config aggregator)
│   └── Log Archive Account (immutable S3 logs)
├── Infrastructure OU
│   └── Shared Services Account (VPC, DNS, tooling)
├── Workloads OU
│   ├── Dev Account
│   ├── Staging Account
│   └── Prod Account
└── Sandbox OU
    └── Developer Accounts (SCPs prevent costly resources)
```

### Key Components
- **AWS Organizations** — tree structure of all accounts
- **Control Tower** — AWS-managed landing zone setup wizard
- **SCPs (Service Control Policies)** — guardrails attached to OUs. Even account root cannot override them.
- **IAM Identity Center (SSO)** — one login, role-based access to all accounts
- **OIDC** — CI/CD pipelines assume IAM roles without static credentials

### Your Project Numbers
- 15 accounts, 5 OUs
- Account provisioning: 2 weeks → 30 minutes
- SOC2 passed first attempt

### Talking Point
> "In Project 4, I architected a 15-account Landing Zone with Control Tower. SCPs are the hard guardrails — even an account admin cannot override them. For example, we had an SCP that blocked any action unless MFA was present, and another that prevented disabling GuardDuty. OIDC federation meant our Jenkins and GitHub Actions pipelines assumed IAM roles via web identity tokens — zero static credentials anywhere."

---

## 1.2 VPC Topology & Networking
**✅ Project 3 — 3-Tier AWS Architecture**

### What is it?
A VPC (Virtual Private Cloud) is your private network inside AWS. You define IP ranges, subnets, routing, and internet access. Nothing gets in or out unless you explicitly allow it.

### Architecture — Production 3-Tier VPC
```
VPC: 10.0.0.0/16
│
├── AZ-a (us-east-1a)
│   ├── Public Subnet   10.0.1.0/24  → Internet Gateway route → ALB, NAT GW
│   ├── Private Subnet  10.0.2.0/24  → NAT GW route           → App Tier (EC2/ECS)
│   └── Data Subnet     10.0.3.0/24  → No internet at all     → RDS, ElastiCache
│
├── AZ-b (us-east-1b)
│   ├── Public Subnet   10.0.4.0/24
│   ├── Private Subnet  10.0.5.0/24
│   └── Data Subnet     10.0.6.0/24
│
└── AZ-c (us-east-1c)
    ├── Public Subnet   10.0.7.0/24
    ├── Private Subnet  10.0.8.0/24
    └── Data Subnet     10.0.9.0/24

Key components:
  Internet Gateway  → allows public subnets to reach internet
  NAT Gateway       → allows private subnets outbound-only internet (patches, pip install)
  VPC Endpoints     → S3, DynamoDB, ECR accessed without NAT (saves cost, improves security)
  Security Groups   → stateful firewall per resource
  NACLs             → stateless firewall per subnet (second layer)
```

### Traffic Rules (Simple)
- Public subnet → can receive inbound from internet (ALB lives here)
- Private subnet → cannot receive inbound from internet, can go outbound via NAT
- Data subnet → completely isolated, only accepts connections from Private subnet SG

### Your Project Numbers
- 9 subnets across 3 AZs
- VPC Endpoints eliminated NAT data transfer cost (part of 35% FinOps saving)
- 5,000 req/s handled by the ALB tier

---

## 1.3 IAM, IRSA, KMS, Secrets Manager
**✅ Projects 3, 4**

### IAM — Identity and Access Management
Every AWS action requires an identity (who are you?) and a policy (what are you allowed to do?). IAM defines both.

```
Principal (who)     → Action (what)      → Resource (on what)
Lambda function     → s3:GetObject       → arn:aws:s3:::my-bucket/*
EC2 instance role   → secretsmanager:... → arn:aws:secretsmanager:...
```

**Least privilege rule:** Give only what is needed, nothing more. If Lambda only reads from one S3 bucket, its role only has `s3:GetObject` on that specific bucket ARN.

### IRSA — IAM Roles for Service Accounts
**Problem:** Pods in EKS need AWS credentials to call services (S3, DynamoDB, Secrets Manager). Old way: bake credentials into the pod as env vars → security disaster.

**IRSA solution:** Pod uses a Kubernetes Service Account. That Service Account is annotated with an IAM Role ARN. AWS OIDC provider trusts the EKS cluster. When the pod calls AWS, it gets a temporary token — no static credentials anywhere.

```
Kubernetes Pod
  └── ServiceAccount: my-app-sa
        └── Annotation: eks.amazonaws.com/role-arn: arn:aws:iam::123:role/my-app-role
              └── IAM Role Trust Policy:
                    {
                      "Principal": {
                        "Federated": "arn:aws:iam::123:oidc-provider/..."
                      },
                      "Condition": {
                        "StringEquals": {
                          "sub": "system:serviceaccount:default:my-app-sa"
                        }
                      }
                    }
```

Pod calls AWS → AWS checks OIDC token → matches trust policy → issues temporary credentials → pod calls S3/DynamoDB/etc.

### KMS — Key Management Service
AWS-managed encryption key service. Every secret, database, S3 bucket, EBS volume should be encrypted with a KMS key.

```
KMS Key (CMK)
  ├── Used by: RDS Aurora (encrypts data at rest)
  ├── Used by: Secrets Manager (encrypts secret values)
  ├── Used by: S3 (encrypts objects)
  └── Used by: EBS volumes (encrypts VM disks)

Key rotation: automatic every 365 days
Key policy: controls who can use/manage the key
```

### Secrets Manager
Stores secrets (DB passwords, API keys, TLS certs) encrypted with KMS. Applications fetch secrets at runtime — never stored in code or environment variables.

```
Application startup:
  1. Pod calls Secrets Manager API using IRSA credentials
  2. Secrets Manager decrypts value using KMS key
  3. Returns plaintext to application in memory
  4. Secret never touches disk, never in Dockerfile, never in Git
```

**Your talking point:** "No static credentials anywhere in the platform" — IRSA for pods, OIDC for CI/CD, Secrets Manager for application secrets, Vault for Ansible.

---

## 1.4 CI/CD Pipeline — Jenkins, ArgoCD, GitOps
**✅ Project 1 — DevSecOps Pipeline**

### What is it?
CI/CD automates the journey from code commit to production deployment. CI (Continuous Integration) = build and test automatically. CD (Continuous Deployment) = deploy automatically after tests pass.

### Your 18-Stage Pipeline Flow
```
Developer pushes code
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│  JENKINS — Build & Security Phase                           │
│                                                             │
│  1. Checkout       → git pull latest code                  │
│  2. Secret Scan    → Gitleaks (no API keys in code)        │
│  3. Unit Tests     → pytest, coverage report               │
│  4. SCA            → Snyk (vulnerable dependencies?)       │
│  5. SAST           → SonarQube (insecure code patterns?)   │
│  6. Quality Gate   → SonarQube threshold check             │
│  7. Docker Build   → multi-stage, 900MB → 150MB           │
│  8. Trivy Scan     → container vulnerabilities             │
│  9. ECR Push       → tagged with Git SHA (never :latest)  │
│  10. Cosign Sign   → cryptographic image signature         │
│  11. S3 Reports    → audit trail for compliance            │
│  12. ArgoCD Sync   → triggers GitOps deployment            │
└───────────────────┬─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────┐
│  ARGOCD — GitOps Deployment Phase                           │
│                                                             │
│  13. Deploy Dev    → automatic                             │
│  14. Smoke Tests   → health check on dev                   │
│  15. Deploy Staging→ automatic after dev passes            │
│  16. DAST          → OWASP ZAP (runtime security scan)     │
│  17. Manual Gate   → human approval for Production         │
│  18. Deploy Prod   → Argo Rollouts canary (5%→100%)       │
└─────────────────────────────────────────────────────────────┘
```

### GitOps — Why ArgoCD?
GitOps means Git is the single source of truth. Kubernetes manifests live in a Git repo. ArgoCD watches that repo and ensures the cluster matches what Git says.

```
Without GitOps:
  kubectl apply -f deployment.yaml  ← manual, no audit trail, drift possible

With GitOps:
  Git repo (desired state) ← ArgoCD watches this
  EKS cluster (actual state) ← ArgoCD makes this match
  
  If someone manually changes a deployment → ArgoCD detects drift → auto-reverts
  Rollback = git revert → ArgoCD applies previous state automatically
```

### Canary Deployment — Argo Rollouts
```
Traffic split during canary:
  
  ┌──────────┐        ┌─────────────────────────────────────┐
  │  Users   │──────▶ │  Argo Rollouts                     │
  └──────────┘        │                                     │
                      │  Step 1: 5% → new version           │
                      │  Step 2: Check Prometheus metrics   │
                      │  Step 3: If OK → 50%                │
                      │  Step 4: If OK → 100%               │
                      │  If error rate spikes → auto-rollback│
                      └─────────────────────────────────────┘
```

**Your metric:** Blast radius reduced from 100% → 5% (only 5% of users hit a bad version before auto-rollback).

---

## 1.5 Containers & Docker
**✅ Project 1**

### What is it?
A container packages your application + all its dependencies into one portable unit. It runs the same everywhere — dev laptop, CI server, production Kubernetes.

### Multi-Stage Dockerfile — Why It Matters
```dockerfile
# Stage 1: Builder (has all build tools — never goes to production)
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: Production (only runtime, no build tools)
FROM python:3.11-slim AS production
WORKDIR /app
# Copy only installed packages from builder — not pip, not gcc
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY . .
# Non-root user — security best practice
RUN useradd -m appuser && chown -R appuser /app
USER appuser
CMD ["gunicorn", "--workers=3", "--bind=0.0.0.0:8000", "app:application"]
```

**Result:** 900MB (with build tools) → 150MB (slim runtime only). Smaller image = faster pull = smaller attack surface.

### Image Security Chain (Project 1)
```
Build → Trivy scan (no HIGH/CRITICAL CVEs) → ECR push → Cosign sign
                                                              │
                                           Kyverno in cluster:
                                           "Reject any pod whose image
                                            is not signed by our key"
```

---

## 1.6 Helm Charts
**✅ Projects 1, 3, K8s doc**

### What is it?
Helm is the package manager for Kubernetes. Instead of writing separate YAML files for every environment, you write one template with variables. Helm fills in the variables per environment.

### Structure
```
my-app/
├── Chart.yaml          ← name, version, description
├── values.yaml         ← default values (dev)
├── values-staging.yaml ← staging overrides
├── values-prod.yaml    ← production overrides
└── templates/
    ├── deployment.yaml ← uses {{ .Values.image.tag }}
    ├── service.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    └── _helpers.tpl    ← reusable template snippets
```

### Example — Environment-Specific Deploys
```bash
# Deploy to Dev (uses values.yaml defaults)
helm upgrade --install my-app ./my-app \
  --namespace dev \
  --set image.tag=a3f7b2c

# Deploy to Production (override with prod values)
helm upgrade --install my-app ./my-app \
  --namespace prod \
  -f values-prod.yaml \
  --set image.tag=a3f7b2c \
  --atomic         # rollback automatically if deploy fails
  --timeout 5m
```

### HPA — Horizontal Pod Autoscaler
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70   # scale up if CPU > 70%
```

**How it works:** Every 15 seconds, HPA checks CPU. If above 70% → add pods. If below 70% for 5 minutes → remove pods.

---

## 1.7 EKS — Elastic Kubernetes Service
**✅ Projects 3, K8s doc**

### What is it?
EKS is AWS-managed Kubernetes. AWS runs the control plane (API server, etcd, scheduler). You run the worker nodes (EC2 or Fargate).

### Architecture
```
┌─────────────────────────────────────────────────────────────┐
│  EKS Control Plane (AWS-managed — you don't touch this)     │
│  API Server │ etcd │ Scheduler │ Controller Manager         │
└──────────────────────┬──────────────────────────────────────┘
                       │ (kubelet registers nodes)
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  Node (AZ-a) │ │  Node (AZ-b) │ │  Node (AZ-c) │
│  Karpenter   │ │  Karpenter   │ │  Karpenter   │
│  managed     │ │  managed     │ │  managed     │
└──────────────┘ └──────────────┘ └──────────────┘
```

### Karpenter — Node Autoscaler
```
Pod scheduled but no node has capacity
        │
        ▼
Karpenter sees pending pod
        │
        ▼
Karpenter picks CHEAPEST node type that fits the pod's requests
(Spot if available, Graviton if compatible, right-sized — not oversized)
        │
        ▼
New node joins cluster in ~60 seconds
        │
        ▼
Pod schedules on new node
```

**Your number:** 60-70% node cost reduction via Spot + bin-packing consolidation.

### EKS vs Self-Managed vs OpenShift (Quick Reference)
| | Self-Managed | EKS | OpenShift |
|---|---|---|---|
| Control plane | You manage | AWS manages | Red Hat manages |
| Cost | EC2 only | $73/mo + EC2 | Subscription |
| Upgrades | Manual | Semi-automated | OTA operator |
| Best for | Learning, on-prem | AWS-native teams | Enterprise/regulated |

---

## 1.8 Istio Service Mesh — mTLS, Zero-Trust
**✅ Project 6**

### What is it?
Istio is a service mesh — an infrastructure layer that handles all communication between microservices without changing application code. A tiny proxy (Envoy) is injected as a sidecar into every pod.

### Sidecar Pattern
```
Without Istio:
  [App Pod] ──── HTTP (plain text) ────▶ [Other App Pod]
  
With Istio:
  [App Container + Envoy sidecar] ══mTLS══▶ [Envoy sidecar + App Container]
  
  App still calls: http://other-service:80 (plain HTTP, to localhost)
  Envoy intercepts: encrypts, verifies identity, adds metrics, adds trace ID
  Zero code changes in the application
```

### mTLS — Mutual TLS
Normal TLS: client verifies server identity (your browser verifies bank's certificate).  
mTLS: BOTH sides verify each other. Every service proves its identity.

```
Service A's Envoy:     "My certificate says I am auth-service"
Service B's Envoy:     "Let me verify... yes, Istio CA signed this. You are auth-service."
                       "My certificate says I am payment-service"
Service A's Envoy:     "Verified. Here is the encrypted request."

Result: Even if attacker gets inside the cluster, they can't impersonate a service.
```

### AuthorizationPolicy — Zero Trust
```yaml
# Only allow order-service to call payment-service
# Everything else is denied by default
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: payment-service-policy
spec:
  selector:
    matchLabels:
      app: payment-service
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/order-service"]
    to:
    - operation:
        methods: ["POST"]
        paths: ["/api/v1/charge"]
```

### Your Numbers
- MTTR reduced from 2+ hours → <5 minutes (Jaeger tracing shows exact failing service instantly)
- 15 microservices under mTLS STRICT mode

---

## 1.9 Serverless — Lambda, EventBridge, SQS, Step Functions
**✅ Project 5**

### What is it?
Serverless means you write code, AWS runs it. No servers to manage, no capacity planning. You pay only when your code executes.

```
Traditional (EC2):    Server runs 24/7 → pay 24/7 even at 3 AM with zero traffic
Serverless (Lambda):  Function runs only when triggered → pay only for execution time
                      Zero traffic = $0
```

### Your Project 5 Architecture — Security Remediation Engine
```
AWS Config detects violation (e.g., S3 bucket made public)
        │
        ▼
EventBridge Rule matches event pattern
        │
        ▼
SQS Queue buffers the event (if Lambda is busy, event waits safely)
        │
        ▼
Lambda Function executes:
  - Reads event from SQS
  - Calls AWS API to fix violation (make S3 bucket private)
  - Writes audit record to DynamoDB
  - Sends notification to SNS
        │
        ▼
SNS fans out:
  ├── Slack webhook (#security channel)
  ├── Email (security team)
  └── PagerDuty (if critical)

Cost: $0.07/month (runs only when violations occur)
Speed: violation detected → fixed in 90 seconds
```

### Step Functions — Orchestrating Multi-Step Workflows
```
Step Functions = workflow engine for Lambda functions

Example: Multi-step remediation workflow:
  
  State 1: ValidateViolation
      └── Lambda checks if violation is real (not false positive)
  State 2: CheckApprovalRequired
      └── Is this a critical resource? Route to human approval if yes
  State 3: ExecuteRemediation
      └── Lambda fixes the violation
  State 4: ValidateRemediation
      └── Lambda confirms fix worked
  State 5: NotifyAndAudit
      └── Lambda writes to DynamoDB, sends Slack message
  
  If any state fails → Step Functions handles retry and error routing
  Full visual workflow visible in AWS console
```

---

## 1.10 Multi-Region HA & DR
**✅ Project 8**

### RTO vs RPO — The Two Numbers That Matter
```
Incident happens (region goes down)
        │
        │←── RPO ──→│←──────── RTO ────────→│
        │            │                        │
        │      Last backup/                  System is
        │      replication                   back online
        │      point
        
RPO (Recovery Point Objective): How much data can you afford to lose?
  Your answer: <1 second (Aurora Global replication lag)

RTO (Recovery Time Objective): How fast must you be back online?
  Your answer: <3 minutes (Route53 failover + warm compute)
```

### Architecture
```
                    Internet
                        │
               ┌────────▼────────┐
               │    Route 53     │
               │  Health Check   │
               │  every 10s      │
               └──┬──────────┬───┘
                  │          │
         Primary  │          │  DR (Standby)
       us-east-1  │          │  us-west-2
                  ▼          ▼
            ┌────────┐   ┌────────┐
            │  ALB   │   │  ALB   │
            └───┬────┘   └───┬────┘
                │            │
            ┌───▼────┐   ┌───▼────┐
            │ EC2 ASG│   │ EC2 ASG│ ← Warm pool (pre-warmed, not serving)
            └───┬────┘   └───┬────┘
                │            │
            ┌───▼────────────▼────┐
            │  Aurora Global DB   │
            │  Primary ←──────────┼── <1s replication lag
            │  (us-east-1)        │   to us-west-2 replica
            └─────────────────────┘
            
S3 Cross-Region Replication (CRR):
  s3://primary-bucket (us-east-1) ──auto-replicate──▶ s3://dr-bucket (us-west-2)
```

### Failover Sequence (What happens when us-east-1 dies)
```
1. Route53 health check fails (3 checks × 10s = 30s detection)
2. Route53 removes primary A record, activates secondary
3. DNS TTL expires (60s) — clients get new IP
4. Aurora Global promotes us-west-2 replica to primary (<1 min)
5. Warm pool EC2s activate in us-west-2 ASG
6. Total RTO: ~3 minutes
```

---

## 1.11 FinOps & Cost Optimization
**✅ Project 7**

### What is it?
FinOps = Financial Operations. Making engineering teams responsible for cloud costs with visibility, automation, and governance.

### Your Savings Breakdown ($180K/year saved)
```
Before: $45,000/month and growing 20% monthly

Savings achieved:
  ├── Karpenter Spot + consolidation     → 60-70% node cost reduction
  ├── VPA rightsizing                    → oversized EC2s downsized
  ├── Non-prod auto-stop (Lambda)        → dev/staging off nights + weekends
  ├── GP2 → GP3 EBS migration           → 20% cheaper, same performance
  ├── VPC Endpoints                      → eliminated NAT data transfer costs
  ├── S3 lifecycle policies              → old logs moved to Glacier
  └── Savings Plans                      → committed use discount
  
After: ~$29,000/month (35% reduction = $16,000/month = $192,000/year)
```

### Kubecost — Kubernetes Cost Visibility
```
Without Kubecost: "Our EKS cluster costs $8,000/month. Which team is responsible?"
                  → Nobody knows.

With Kubecost:    namespace: team-payments    → $2,400/month
                  namespace: team-auth        → $1,800/month
                  namespace: team-frontend    → $900/month
                  
Each team gets monthly chargeback report.
Alerts when namespace spend exceeds budget.
```

---

## 1.12 Terraform — Modules, Remote State, Environment Gates
**✅ All Projects**

### Module Structure
```
modules/
├── vpc/              ← reusable VPC module
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── eks/              ← reusable EKS module
├── rds/              ← reusable RDS Aurora module
└── alb/              ← reusable ALB module

environments/
├── dev/
│   ├── main.tf       ← calls modules with dev values
│   └── backend.tf    ← remote state: s3://state/dev/terraform.tfstate
├── staging/
└── prod/
```

### Remote State — Why It Matters
```
Problem:  Two engineers run terraform apply at same time → state corruption
Solution: Remote state in S3 with DynamoDB locking

backend.tf:
  terraform {
    backend "s3" {
      bucket         = "my-terraform-state"
      key            = "prod/terraform.tfstate"
      region         = "us-east-1"
      dynamodb_table = "terraform-locks"  ← prevents concurrent applies
      encrypt        = true
    }
  }

DynamoDB lock entry created on terraform plan/apply.
Deleted when complete. Second engineer gets "state is locked" error.
```

### Environment Promotion Gates
```
CI/CD pipeline for Terraform:

  PR opened
      │
      ▼
  terraform fmt + validate (syntax check)
      │
      ▼
  tfsec + Checkov + OPA (security scan — no misconfigs)
      │
      ▼
  Infracost (cost impact shown on PR — "this PR adds $45/month")
      │
      ▼
  terraform plan (show what will change — no apply yet)
      │
      ▼
  [HUMAN REVIEW] — engineer approves PR
      │
      ▼
  terraform apply to DEV (automatic after merge)
      │
      ▼
  [MANUAL GATE] — approve for staging
      │
      ▼
  terraform apply to STAGING
      │
      ▼
  [MANUAL GATE] — approve for prod
      │
      ▼
  terraform apply to PROD
```

---

## 1.13 Security — Zero-Trust, SCPs, Compliance
**✅ Projects 4, 6**

### Zero-Trust Networking
Zero-trust means: **never trust, always verify**. Nothing inside your network is automatically trusted — every connection must be authenticated and authorized, every time.

```
Old model (castle and moat):
  Outside: dangerous → blocked
  Inside:  trusted   → anything goes
  Problem: one compromised machine inside = attacker moves freely

Zero-trust model:
  Every service must prove its identity (mTLS certificates)
  Every request must be authorized (AuthorizationPolicy)
  Every action must be logged (CloudTrail, Istio access logs)
  Least privilege everywhere (IRSA, SCPs)
```

### Your Zero-Trust Implementation Across Projects
```
Network layer:     Default-deny NetworkPolicies (Project 3)
                   Pods cannot talk to each other unless explicitly allowed

Service layer:     Istio mTLS STRICT (Project 6)
                   Every service-to-service call encrypted + identity verified

Identity layer:    IRSA (Project 3) — pods get temporary AWS credentials
                   OIDC (Project 4) — CI/CD gets temporary AWS credentials
                   No static credentials anywhere

Guardrail layer:   SCPs (Project 4) — hard stops on what accounts can do
                   Even account admin cannot override SCPs
```

### SCPs — Service Control Policies
```
SCP = JSON policy attached to an OU or account in AWS Organizations
Effect: limits what IAM users/roles in that account can do

Example SCP — prevent disabling GuardDuty:
  {
    "Version": "2012-10-17",
    "Statement": [{
      "Sid": "DenyDisableGuardDuty",
      "Effect": "Deny",
      "Action": [
        "guardduty:DeleteDetector",
        "guardduty:DisassociateFromMasterAccount"
      ],
      "Resource": "*"
    }]
  }

Even if an engineer has AdministratorAccess IAM role,
this SCP prevents them from disabling GuardDuty.
SCP overrides IAM.
```

---

## 1.14 Monitoring — Prometheus, Grafana, Jaeger, EFK
**✅ Projects 1, 6**

### Observability Stack
```
┌─────────────────────────────────────────────────────────────┐
│  THREE PILLARS OF OBSERVABILITY                             │
│                                                             │
│  METRICS              LOGS                  TRACES          │
│  (numbers over time)  (events with context) (request path)  │
│                                                             │
│  Prometheus           Elasticsearch         Jaeger           │
│  "CPU is at 85%"      "User login failed:   "Request took   │
│                        invalid password"     300ms — 280ms  │
│                                              was in DB call" │
│                                                             │
│  Grafana dashboards   Kibana UI              Jaeger UI       │
└─────────────────────────────────────────────────────────────┘
```

### Alert Routing — P1 vs P2 vs P3
```
Prometheus fires alert
        │
        ▼
AlertManager (routes based on severity)
        │
        ├── P1 (production down)     → PagerDuty (wakes engineer at 3 AM)
        ├── P2 (degraded performance) → Slack #alerts (seen during work hours)
        └── P3 (warning)              → Jira ticket (address next sprint)
```

### Jaeger Distributed Tracing — Why MTTR Dropped to 5 Minutes
```
Without tracing: User reports "checkout is slow"
  Engineer checks: frontend logs? fine. backend logs? fine. DB logs? fine.
  1 hour later: finds slow Redis query buried in microservice #8

With Jaeger tracing:
  User reports "checkout is slow"
  Engineer opens Jaeger: finds the trace in 30 seconds
  Trace shows: 280ms of 300ms total time was in redis-service → DB call
  Fix identified in 5 minutes
```

---

---

# PART 2 — Gap Topics (Learn These This Week)

---

## 2.1 Amazon Bedrock
**❌ Gap → Maps to Project 5 (Serverless)**

### What is it?
Amazon Bedrock is a fully managed AWS service that gives you access to powerful AI foundation models (FMs) from AWS and third parties — via a single API. You don't train models, you don't manage GPU servers. You just call an API and get AI-generated responses.

Think of it like this:
```
Without Bedrock:  You need GPUs, model training, model hosting, scaling infrastructure
With Bedrock:     You call an API → AWS handles everything → you get the AI response
                  Pay per API call. Zero infrastructure.
```

### Foundation Models Available on Bedrock
```
Model Provider    Model Name           Best For
─────────────────────────────────────────────────────
Anthropic         Claude Haiku         Fast, cheap — simple tasks, classification
Anthropic         Claude Sonnet        Balanced — most tasks, code, reasoning
Anthropic         Claude Opus          Most powerful — complex reasoning
Amazon            Titan Text v2        AWS-native, good for RAG and embeddings
Amazon            Titan Embeddings     Convert text to vectors (for search)
Meta              Llama 3              Open-source alternative
Stability AI      SDXL                 Image generation
```

### How Bedrock Works — Simple Flow
```
Your Application
        │
        │  API call (boto3 or HTTP)
        ▼
Amazon Bedrock API
        │
        │  Routes to model
        ▼
Foundation Model (Claude, Titan, etc.)
        │
        │  AI-generated response
        ▼
Your Application receives response

No GPU management. No model deployment. No scaling.
```

### InvokeModel — The Core API
```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

# Call Claude Sonnet
response = bedrock.invoke_model(
    modelId='anthropic.claude-3-sonnet-20240229-v1:0',
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1024,
        "messages": [{
            "role": "user",
            "content": "Analyze this AWS Config violation and suggest remediation: S3 bucket public access enabled"
        }]
    })
)

result = json.loads(response['body'].read())
print(result['content'][0]['text'])
# Output: "The S3 bucket has public access enabled which poses a security risk.
#          Recommended action: Set BlockPublicAcls, BlockPublicPolicy,
#          IgnorePublicAcls, RestrictPublicBuckets to true..."
```

### IAM — Who Can Call Bedrock?
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "bedrock:InvokeModel",
      "bedrock:InvokeModelWithResponseStream"
    ],
    "Resource": [
      "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet*",
      "arn:aws:bedrock:us-east-1::foundation-model/amazon.titan*"
    ]
  }]
}
```

This IAM policy is attached to your Lambda function's execution role (via IRSA in EKS, or Lambda role directly).

### Cross-Region Inference Profiles
```
Problem: Claude Sonnet in us-east-1 is overloaded → requests fail or are slow
Solution: Cross-region inference profile

Instead of calling:
  modelId = "anthropic.claude-3-sonnet-20240229-v1:0"  (single region)

You call:
  modelId = "us.anthropic.claude-3-sonnet-20240229-v1:0"  (cross-region profile)

AWS automatically routes to: us-east-1, us-west-2, us-east-2
  → whichever has capacity right now
  → higher availability, lower latency under load

For BBY (Best Buy) specifically: SCPs may restrict which regions Bedrock
can be called from. Cross-region inference must be within approved regions.
```

### Bedrock + SCP Considerations
```
BBY organizational SCPs may:
  1. Restrict Bedrock to specific regions only
     → SCP: Deny bedrock:* if region is not us-east-1 or us-west-2

  2. Restrict which models can be accessed
     → SCP: Deny bedrock:InvokeModel on * except specific model ARNs

  3. Require VPC endpoint for Bedrock
     → Bedrock calls must go through VPC endpoint, not public internet
     → No data leaves your VPC

Model access must be explicitly enabled in Bedrock console per region.
Default: all models are disabled until you click "Request access".
```

### How It Maps to Your Project 5
```
Your current Project 5:
  AWS Config violation → EventBridge → SQS → Lambda → rule-based fix
  
  Lambda code today:
    if violation == "sg-open-ssh":
        remove_sg_rule(sg_id, port=22)
    elif violation == "s3-public":
        block_public_access(bucket_name)
    # 50 more elif statements...

Natural evolution with Bedrock:
  AWS Config violation → EventBridge → SQS → Lambda → Bedrock → intelligent fix
  
  Lambda code with Bedrock:
    violation_description = event['detail']['configRuleName']
    context = get_resource_context(event)
    
    response = bedrock.invoke_model(
        modelId='us.anthropic.claude-3-haiku...',
        body=json.dumps({
            "messages": [{
                "role": "user",
                "content": f"AWS Config violation: {violation_description}. 
                            Resource context: {context}.
                            What boto3 API calls should I make to remediate?"
            }]
        })
    )
    # Bedrock returns the remediation steps
    # Lambda executes them
```

**Your talking point:** "Project 5 built the serverless architecture for automated remediation. Bedrock is the intelligence layer I'd add — replacing hardcoded if/else logic with AI-driven analysis. The infrastructure (EventBridge → SQS → Lambda) doesn't change. The IAM role gets `bedrock:InvokeModel` added. The Lambda calls Bedrock instead of a static dictionary."

---

## 2.2 OpenTelemetry (OTel) & ADOT Collector
**⚠️ Partial Gap → Maps to Project 6 (Istio + Jaeger)**

### What is OpenTelemetry?
OpenTelemetry (OTel) is the open-source standard for collecting telemetry data — traces, metrics, and logs — from your applications. It's vendor-neutral: you instrument once, export to any backend (Jaeger, Datadog, Prometheus, CloudWatch, etc.).

```
Before OTel:
  Team A uses Datadog agent   → data only in Datadog
  Team B uses Jaeger SDK      → data only in Jaeger
  Team C uses New Relic agent → data only in New Relic
  Problem: switching vendor means re-instrumenting every app

With OTel:
  All teams use OTel SDK → data goes to OTel Collector → Collector exports to ANY backend
  Switch from Datadog to Prometheus? Change one Collector config line. Zero app changes.
```

### Three Components You Must Know
```
1. OTel SDK (in your application code)
   → Instruments the app: creates spans, records metrics, attaches context
   → Available for Python, Java, Go, Node.js, .NET, etc.

2. OTel Collector (sidecar or daemonset)
   → Receives telemetry from SDK
   → Processes it (batch, filter, enrich)
   → Exports to backend (Jaeger, Prometheus, CloudWatch)

3. OTLP (OpenTelemetry Protocol)
   → The wire protocol SDK uses to send data to Collector
   → gRPC on port 4317
   → HTTP on port 4318
```

### OTel Collector Architecture
```
┌──────────────────────────────────────────────────────────────┐
│  OTel Collector                                               │
│                                                               │
│  RECEIVERS          PROCESSORS          EXPORTERS            │
│  (accept data)      (transform data)    (send to backend)    │
│                                                               │
│  • otlp (gRPC)     • batch             • jaeger              │
│  • prometheus      • filter            • prometheus           │
│  • jaeger          • attributes        • awsxray             │
│  • zipkin          • memory_limiter    • otlp (to another    │
│                    • tail_sampling      collector)            │
└──────────────────────────────────────────────────────────────┘
```

### ADOT — AWS Distro for OpenTelemetry
ADOT is Amazon's production-ready, tested distribution of the OTel Collector. It includes:
- AWS-specific exporters (CloudWatch, X-Ray, Managed Prometheus)
- AWS authentication handled automatically (uses IRSA)
- Validated by AWS for security and performance

```
OTel Collector (upstream, community)  →  ADOT Collector (AWS-validated distribution)

Difference:
  OTel Collector: you configure AWS credentials manually
  ADOT:           uses pod's IRSA automatically → no credential management
```

### Sidecar Deployment Pattern
```yaml
# Each pod gets an ADOT sidecar (same pattern as Istio's Envoy)
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      # Main application container
      - name: my-service
        image: my-service:latest
        env:
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://localhost:4317"  # sends to sidecar
        - name: OTEL_SERVICE_NAME
          value: "my-service"
      
      # ADOT sidecar container
      - name: adot-collector
        image: public.ecr.aws/aws-observability/aws-otel-collector:latest
        args: ["--config=/conf/otel-collector-config.yaml"]
        volumeMounts:
        - name: otel-config
          mountPath: /conf
```

### ADOT Collector Config
```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"  # receives from app on localhost

processors:
  batch:
    timeout: 1s
    send_batch_size: 50

exporters:
  awsxray:                          # traces → AWS X-Ray
    region: us-east-1
  prometheusremotewrite:            # metrics → Amazon Managed Prometheus
    endpoint: "https://aps-workspaces.us-east-1.amazonaws.com/..."
  awscloudwatchlogs:                # logs → CloudWatch
    region: us-east-1
    log_group_name: "/my-app/traces"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [awsxray]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheusremotewrite]
```

### Distributed Trace Propagation — The JD's Key Requirement
The JD says: "single trace ID propagates from orchestrator through all agents to Bedrock."

```
How trace propagation works:

  AI Orchestrator receives request → generates Trace ID: abc-123
        │
        │ HTTP header: traceparent: 00-abc123-span001-01
        ▼
  Agent Service A receives request → reads trace ID abc-123
        │ creates child span: span002 (parent: span001)
        │
        │ HTTP header: traceparent: 00-abc123-span002-01
        ▼
  Agent Service B receives request → reads trace ID abc-123
        │ creates child span: span003 (parent: span002)
        │
        │ HTTP header: traceparent: 00-abc123-span003-01
        ▼
  Bedrock API call → OTel SDK wraps this as span: span004

Result in Jaeger/X-Ray:
  abc-123 (total: 1.2s)
    ├── span001: orchestrator (50ms)
    ├── span002: agent-service-A (200ms)
    ├── span003: agent-service-B (800ms)  ← THIS is the slow one
    └── span004: bedrock-call (150ms)
```

**Key:** `traceparent` header (W3C standard) is what carries the trace ID across service boundaries. OTel SDK auto-injects and reads this header for HTTP/gRPC calls.

### How It Maps to Project 6
**Your talking point:** "In Project 6, I implemented the sidecar pattern with Istio's Envoy proxy — every pod gets a sidecar injected automatically. Envoy collected traces and shipped to Jaeger. ADOT is the same pattern: sidecar injected per pod, collects OTLP telemetry, exports to AWS backends. Jaeger uses the same W3C TraceContext propagation standard that ADOT uses. The concept is identical — I'd be swapping Envoy/Jaeger for ADOT/X-Ray and adding OTel SDK instrumentation to application code."

---

## 2.3 ElastiCache Redis — Cluster Mode & Sharding
**⚠️ Partial Gap → Maps to Project 3**

### What is Redis?
Redis is an in-memory key-value store. Extremely fast (sub-millisecond reads/writes) because data lives in RAM, not on disk.

```
Use cases:
  Session storage    → user login state (expires after 30 min)
  Caching            → expensive DB query results cached for 5 min
  Rate limiting      → "this IP made 100 requests in 1 minute, block it"
  Event bus          → pub/sub messaging between services (this JD's use case)
  Leaderboards       → sorted sets for real-time rankings
```

### Redis Cluster Mode — What and Why
```
Standard Redis (no cluster):
  One primary node handles ALL reads and writes
  One replica for failover
  Problem: limited to one node's memory (~30GB max)
           all traffic hits one node → bottleneck

Cluster Mode:
  Data is split (sharded) across multiple primary nodes
  Each primary has its own replica
  
  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
  │  Shard 1    │  │  Shard 2    │  │  Shard 3    │
  │  Primary    │  │  Primary    │  │  Primary    │
  │  Slots 0-   │  │  Slots 5461-│  │  Slots 10923│
  │  5460       │  │  10922      │  │  -16383     │
  │  + Replica  │  │  + Replica  │  │  + Replica  │
  └─────────────┘  └─────────────┘  └─────────────┘
  
  Total: 16,384 hash slots divided across shards
  Reads/writes distributed → 3x throughput
  Memory distributed → 3x total capacity
```

### Hash Slots — How Data is Distributed
```
When you write key "user:123":
  1. Redis calculates: CRC16("user:123") % 16384 = 7823
  2. Slot 7823 belongs to Shard 2
  3. Write goes to Shard 2's primary

When you write key "user:456":
  1. CRC16("user:456") % 16384 = 2341
  2. Slot 2341 belongs to Shard 1
  3. Write goes to Shard 1's primary

Result: data automatically distributed without application knowing
```

### Shard-by-Run-ID (This JD's Specific Pattern)
The JD says: "Redis event bus — HA cluster mode, shard-by-run-id for MI and CR orchestrators."

```
What this means:
  Each AI orchestration "run" has a unique run-id (e.g., "run-abc123")
  All events for that run should go to the SAME Redis shard
  Why: avoids cross-shard transactions, keeps all run state in one place

How to implement this with Redis hash tags:
  Instead of key: "event:run-abc123:step-1"
  Use key:        "{run-abc123}:event:step-1"
                   ^^^^^^^^^^^^^^^^
                   Curly braces = hash tag
                   Redis only hashes the part in curly braces
                   All keys with {run-abc123} → same hash slot → same shard

Application code:
  redis.set(f"{{run-{run_id}}}:event:step-1", payload)
  redis.set(f"{{run-{run_id}}}:event:step-2", payload)
  redis.set(f"{{run-{run_id}}}:status", "running")
  # All three keys go to the same shard → no cross-shard coordination
```

### HA Failover in Cluster Mode
```
Normal state:
  Shard 1: Primary (slots 0-5460)   + Replica
  Shard 2: Primary (slots 5461-10922) + Replica
  Shard 3: Primary (slots 10923-16383) + Replica

Shard 2 primary dies:
  1. Cluster detects failure (within 15 seconds)
  2. Remaining primaries vote: "Is Shard 2 primary really down?"
  3. Consensus reached → Shard 2 replica promoted to primary
  4. Cluster updates slot ownership map
  5. New writes to slots 5461-10922 go to new primary
  
Total failover time: 15-60 seconds
RPO: seconds of data loss possible (replica may lag slightly)
```

### ElastiCache Cluster Mode Config (Terraform)
```hcl
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id = "ai-platform-redis"
  description          = "Redis event bus for AI orchestrators"
  
  node_type            = "cache.r7g.large"  # memory-optimized
  num_cache_clusters   = 2                   # primary + 1 replica per shard
  
  # Enable cluster mode
  parameter_group_name = "default.redis7.cluster.on"
  
  # Number of shards
  num_node_groups         = 3   # 3 shards
  replicas_per_node_group = 1   # 1 replica per shard = 6 nodes total
  
  automatic_failover_enabled = true
  multi_az_enabled          = true
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  
  subnet_group_name = aws_elasticache_subnet_group.redis.name
  security_group_ids = [aws_security_group.redis.id]
}
```

### Your Talking Point
> "In Project 3, I deployed ElastiCache Redis as the caching layer for the application tier. For this platform's use case — Redis as an event bus with shard-by-run-id — cluster mode is the right choice. Hash tags ensure all events for a given orchestration run land on the same shard, eliminating cross-shard coordination. The HA pattern follows the same principle as Aurora Multi-AZ in Project 3: primary failure triggers automatic replica promotion, handled by ElastiCache within 60 seconds."

---

## 2.4 Amazon Neptune Serverless
**❌ Gap → Maps to Projects 3, 4 (managed service provisioning pattern)**

### What is it?
Amazon Neptune is AWS's fully managed graph database. A graph database is designed for data where relationships between things matter as much as the things themselves.

```
Relational DB (RDS):        Graph DB (Neptune):
  Users table               Users as NODES
  Products table            Products as NODES
  Orders table              "purchased" as EDGES connecting User → Product
  
  Query: "Users who bought X also bought Y?"
    → 3 table JOINs → slow at scale
  
  Query in graph: "Traverse 2 hops from Product X"
    → follows edges → fast regardless of scale
```

### When to Use Neptune (vs RDS)
```
Use Neptune when:
  ✅ Knowledge graphs         → AI agents' memory of entities and relationships
  ✅ Recommendation engines   → "users similar to you also liked..."
  ✅ Fraud detection           → "this account is connected to 5 flagged accounts"
  ✅ Network topology          → "which servers can reach this database?"
  ✅ Social graphs             → followers, connections, relationships

Use RDS when:
  ✅ Tabular data              → orders, users, products, transactions
  ✅ Complex aggregations      → SUM, COUNT, GROUP BY
  ✅ ACID transactions         → financial records
```

### Why Neptune for an AI Platform
In an AI agent platform (like this JD describes), Neptune stores the **knowledge graph**:
- Entities the AI has learned about (people, companies, products)
- Relationships between entities (works-at, purchased, is-related-to)
- Historical context that agents use for reasoning

```
Example:
  Node: User "John Smith"
  Node: Company "Acme Corp"
  Edge: John "works-at" Acme Corp since 2020
  
  Node: Product "Widget Pro"
  Edge: John "purchased" Widget Pro on 2024-01-15
  Edge: Widget Pro "is-compatible-with" Widget Basic
  
  AI Agent query: "What products should we recommend to John's colleagues at Acme Corp?"
  → Graph traversal: John → works-at → Acme Corp → other-employees → their-purchases
```

### Neptune Serverless — What Changes vs Standard Neptune
```
Standard Neptune:
  You choose instance size: db.r6g.large, db.r6g.xlarge, etc.
  Runs 24/7 at fixed capacity
  You pay even when zero queries run
  You manually scale up when traffic grows

Neptune Serverless:
  No instance size — you set min/max Neptune Capacity Units (NCUs)
  Automatically scales 1 NCU → 128 NCUs based on traffic
  Scales to zero when idle (after 5 minutes of no activity)
  Pay per NCU-hour of actual usage
  
  Perfect for: AI platforms where agent workloads are bursty
    Dev/test: scales to zero at night = $0
    Production peak: scales up automatically
```

### Architecture in This Platform
```
AI Agent needs entity data
        │
        ▼
Neptune Serverless (graph DB)
  VPC endpoint (no internet exposure)
  Accessed via IAM authentication (no passwords)
  
  ┌─────────────────────────────────────────────────┐
  │  Neptune Serverless Cluster                      │
  │                                                 │
  │  Writer endpoint: cluster.cluster-xxx.neptune.. │
  │  Reader endpoint: cluster.cluster-ro-xxx...     │
  │                                                 │
  │  NCUs: min=1, max=32                            │
  │  Auto-scales based on query load                │
  └─────────────────────────────────────────────────┘
```

### Terraform Provisioning
```hcl
resource "aws_neptune_cluster" "ai_knowledge_graph" {
  cluster_identifier                  = "ai-platform-knowledge-graph"
  engine                              = "neptune"
  serverless_v2_scaling_configuration {
    min_capacity = 1.0   # minimum NCUs (scales to near-zero)
    max_capacity = 32.0  # maximum NCUs (handles peak load)
  }
  
  iam_database_authentication_enabled = true   # no passwords, use IAM
  storage_encrypted                   = true
  kms_key_arn                         = aws_kms_key.neptune.arn
  
  vpc_security_group_ids = [aws_security_group.neptune.id]
  neptune_subnet_group_name = aws_neptune_subnet_group.main.name
  
  skip_final_snapshot = false
  final_snapshot_identifier = "ai-platform-neptune-final"
}
```

### Query Language — Gremlin (know the basics)
```groovy
// Add a person node
g.addV('person').property('name', 'John Smith').property('age', 35)

// Add a company node
g.addV('company').property('name', 'Acme Corp')

// Add edge: John works at Acme
g.V().has('name', 'John Smith').addE('works-at').to(g.V().has('name', 'Acme Corp'))

// Query: Who works at Acme?
g.V().has('name', 'Acme Corp').in('works-at').values('name')
// Returns: ["John Smith", "Jane Doe", ...]

// Query: What did John's colleagues buy?
g.V().has('name', 'John Smith')
  .out('works-at')       // → Acme Corp
  .in('works-at')        // → all Acme employees
  .out('purchased')      // → their products
  .values('name')        // → product names
  .dedup()               // remove duplicates
```

### Your Talking Point
> "Neptune Serverless is new to me specifically, but graph databases and managed service provisioning are patterns I know well. In Projects 3 and 4, I provisioned Aurora, ElastiCache, and DynamoDB via Terraform with the same patterns: VPC placement, IAM authentication, KMS encryption, security group rules. Neptune Serverless adds the serverless scaling dimension — same as Lambda in Project 5. The IAM auth model is something I've used with Aurora IAM authentication. I'd ramp up on Gremlin query language, but the infrastructure layer is familiar ground."

---

## 2.5 Amazon OpenSearch Serverless
**⚠️ Partial Gap → Maps to your Elasticsearch knowledge**

### What is it?
OpenSearch is Amazon's open-source fork of Elasticsearch. When Elastic changed licensing in 2021, AWS forked Elasticsearch and called it OpenSearch. The API is largely compatible.

```
Elasticsearch 7.x  ──fork──▶  OpenSearch 1.x, 2.x, 3.x
  • Same query DSL           • Same query DSL
  • Same index/mapping API   • Same index/mapping API
  • Kibana UI                • OpenSearch Dashboards (same but renamed)
  • X-Pack features          • OpenSearch Security (same functionality)
```

**Your Elasticsearch knowledge (10 chapters) maps directly to OpenSearch.**

### Standard OpenSearch vs OpenSearch Serverless
```
Standard OpenSearch Service:
  You choose: instance type (m6g.large.search), instance count
  You manage: cluster sizing, shard count, replica count
  Cost: runs 24/7 at fixed capacity
  Complexity: high — shard planning, hot/warm/cold tiers

OpenSearch Serverless:
  No cluster sizing — AWS manages capacity automatically
  You create "collections" instead of clusters
  Scales automatically based on load
  Two types:
    • Search collection    → for full-text search and queries
    • Time-series collection → for log/metric data (like what you'd ship from EFK)
```

### Architecture — OpenSearch Serverless
```
┌─────────────────────────────────────────────────────────────┐
│  OpenSearch Serverless                                       │
│                                                             │
│  Collection (type: search)                                  │
│  ├── Index: "agent-conversations"                          │
│  ├── Index: "ai-responses"                                 │
│  └── Index: "platform-logs"                                │
│                                                             │
│  No shard management — AWS handles distribution             │
│  VPC endpoint for private access                           │
│  Data access policies (IAM-based)                          │
└─────────────────────────────────────────────────────────────┘

Data Access Policy (controls who queries/writes):
  ├── Role A (Lambda)      → can write to all indexes
  ├── Role B (Grafana)     → can read from all indexes
  └── Role C (Admin)       → can manage collection settings
```

### Key Differences from Standard OpenSearch
| | Standard OpenSearch | OpenSearch Serverless |
|---|---|---|
| Capacity management | You choose instance type/count | Automatic |
| Sharding | You configure | Automatic |
| Cost model | Per instance-hour | Per OCU (OpenSearch Capacity Unit) hour |
| Idle cost | Runs 24/7 | Scales down (min 2 OCUs) |
| VPC access | VPC + security groups | VPC endpoint (different model) |
| Use case | Predictable load, large clusters | Variable load, dev/test, bursty |

### Terraform Provisioning
```hcl
resource "aws_opensearchserverless_collection" "ai_platform" {
  name = "ai-platform-search"
  type = "SEARCH"   # or "TIMESERIES" for log data
  
  description = "AI agent conversation history and knowledge search"
}

# VPC endpoint for private access
resource "aws_opensearchserverless_vpc_endpoint" "main" {
  name       = "ai-platform-opensearch-vpc"
  subnet_ids = var.private_subnet_ids
  vpc_id     = var.vpc_id
  security_group_ids = [aws_security_group.opensearch.id]
}

# Access policy
resource "aws_opensearchserverless_access_policy" "data" {
  name        = "ai-platform-data-access"
  type        = "data"
  policy = jsonencode([{
    Rules = [
      {
        ResourceType = "index"
        Resource     = ["index/ai-platform-search/*"]
        Permission   = ["aoss:ReadDocument", "aoss:WriteDocument", "aoss:CreateIndex"]
      }
    ]
    Principal = [aws_iam_role.lambda_role.arn]
  }])
}
```

### Your Talking Point
> "OpenSearch is the AWS fork of Elasticsearch — the API is compatible, and I have deep Elasticsearch knowledge covering indexing, querying, aggregations, security, backup, and performance tuning. OpenSearch Serverless removes the cluster management layer — no shard planning, no instance sizing. I'd use it for storing AI agent conversation history and enabling semantic search over platform logs. The provisioning is a Terraform collection resource with a VPC endpoint and data access policy."

---

## 2.6 ECS Fargate + Helm Deployment
**⚠️ Partial Gap → Maps to Project 1**

### What is ECS Fargate?
ECS (Elastic Container Service) is AWS's container orchestration service. Fargate is the serverless compute option for ECS — you don't manage EC2 nodes. AWS runs the containers for you.

```
ECS with EC2:     You provision EC2 instances → ECS schedules containers on them
                  You manage patching, sizing, scaling of EC2 fleet
                  
ECS with Fargate: No EC2 instances → AWS runs containers directly
                  You define: how much CPU/memory each container needs
                  AWS finds the capacity and runs it
                  Pay per task (per container running), not per server
```

### ECS vs EKS — Key Differences
```
                ECS Fargate             EKS
─────────────────────────────────────────────────────
Complexity:     Low (AWS manages all)   High (you manage nodes, networking)
Portability:    AWS-only                Standard Kubernetes (runs anywhere)
Helm support:   No native Helm          Full Helm support
Flexibility:    Limited                 Full Kubernetes ecosystem
Cost (small):   Cheaper (no cluster fee) $73/mo cluster + nodes
Cost (large):   Can be expensive         More cost-efficient
Migration path: To EKS when limits hit  Already there
```

### Why This JD Uses Fargate First
```
Platform is starting from PoC → production
  Phase 1: Fargate (simpler, faster to get running, no cluster management)
  Phase 2: EKS (when Fargate limits are hit: 16 vCPU per task, complex networking)
  
  JD says: "Define EKS migration trigger criteria and execute migration when Fargate limits exceeded"
  Your Project 3 and K8s comparison doc covers EKS migration criteria perfectly.
```

### ECS Task Definition — The Equivalent of a Kubernetes Deployment
```json
{
  "family": "ai-orchestrator",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",      // 1 vCPU
  "memory": "2048",   // 2 GB
  "executionRoleArn": "arn:aws:iam::123:role/ecs-execution-role",
  "taskRoleArn": "arn:aws:iam::123:role/orchestrator-task-role",
  "containerDefinitions": [
    {
      "name": "orchestrator",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/orchestrator:a3f7b2c",
      "portMappings": [{ "containerPort": 8080 }],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123:secret:db-password"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/orchestrator",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    },
    {
      "name": "adot-collector",
      "image": "public.ecr.aws/aws-observability/aws-otel-collector:latest",
      "essential": false   // sidecar — doesn't kill task if it dies
    }
  ]
}
```

### CI/CD Pipeline for ECS Fargate (Maps Directly to Project 1)
```
Your Project 1 pipeline stages map to ECS:

  Stage 1-11: IDENTICAL
    Checkout → Secret Scan → Tests → SCA → SAST → Quality Gate 
    → Docker Build → Trivy → ECR Push → Cosign → S3 Reports
  
  Stage 12-18: DIFFERENT (ECS instead of Helm/EKS)
    
    Instead of:
      helm upgrade --install ... --namespace prod
    
    You do:
      # Register new task definition version
      aws ecs register-task-definition \
        --family ai-orchestrator \
        --container-definitions "[{...new image tag...}]"
      
      # Update service to use new task definition
      aws ecs update-service \
        --cluster ai-platform \
        --service orchestrator \
        --task-definition ai-orchestrator:47 \
        --force-new-deployment
      
      # ECS does rolling deployment: new tasks start → old tasks drain
```

### Helm on ECS — The Truth
```
Helm does NOT natively manage ECS resources (task definitions, services).
Helm manages Kubernetes resources.

However, some teams use Helm as a TEMPLATING tool for ECS:
  values.yaml → Helm template → CloudFormation template → deployed via cfn

Or more commonly:
  Helm = used only for EKS
  ECS  = managed via Terraform (task definitions, services, ALB target groups)

In practice: JD likely means "use Helm to deploy to EKS" for the services
that are already containerized, and ECS task definitions for Fargate-native ones.
```

### Your Talking Point
> "In Project 1, I built the full CI/CD pipeline — image build, ECR push, Helm deploy to EKS with approval gates between environments. The pipeline structure for ECS Fargate is identical through the ECR push stage. The deployment stage replaces `helm upgrade` with `aws ecs update-service` pointing to the new task definition revision. I've delivered the EKS/Helm side completely. Fargate extends the same pipeline pattern — just a different deployment command at the end."

---

## 2.7 Transit Gateway — On-Prem Peering & Direct Connect
**⚠️ Partial Gap → Maps to Project 4**

### What is Transit Gateway?
Transit Gateway (TGW) is AWS's central hub for connecting multiple VPCs and on-premises networks. Without TGW, connecting 15 VPCs requires 15×14/2 = 105 VPC peering connections. With TGW, everything connects to one hub.

```
Without Transit Gateway (VPC Peering mesh — Project 4 without TGW):

  VPC-Dev ──────────── VPC-Staging
    │  ╲               ╱  │
    │   ╲─────────────╱   │
    │                     │
    VPC-Prod ─── VPC-Security
    
  Problem: N accounts = N*(N-1)/2 peering connections
           15 accounts = 105 connections to manage

With Transit Gateway:

  VPC-Dev ──────────┐
  VPC-Staging ───── TGW (hub) ──── On-Premises (via Direct Connect)
  VPC-Prod ─────────┘    │
  VPC-Security ──────────┘
  
  15 accounts = 15 TGW attachments (simple, manageable)
```

### TGW Route Tables — Traffic Control
```
TGW has route tables that control which attachments can talk to which.

Example setup (Project 4 pattern):
  
  Route Table: "prod-only"
    → Associated with: Prod VPC attachment
    → Routes: 
        10.0.0.0/8 (all internal) → blackhole  (can't reach dev/staging)
        172.16.0.0/12 (on-prem)  → Direct Connect attachment (can reach on-prem)
  
  Route Table: "dev-staging"
    → Associated with: Dev and Staging VPC attachments  
    → Routes:
        10.0.0.0/8 → propagated from all VPCs (can reach each other)
        172.16.0.0/12 → blackhole  (CANNOT reach on-prem — security)
```

### Direct Connect — Enterprise On-Premises Connection
```
What it is:
  A dedicated physical network connection from your data center to AWS.
  Not over the public internet. Private fiber circuit.

Why enterprises use it:
  ✅ Consistent bandwidth (not shared with internet traffic)
  ✅ Lower latency (dedicated path vs internet routing)
  ✅ Higher security (traffic never touches public internet)
  ✅ Cost predictable (fixed monthly cost vs variable data transfer)

Connection types:
  Dedicated: 1Gbps or 10Gbps — your own physical port at AWS Direct Connect location
  Hosted: 50Mbps → 10Gbps — shared infrastructure, ordered from AWS partner

For BBY (Best Buy): They have existing Direct Connect circuits to their data centers.
Your job: attach their Direct Connect Gateway to the Transit Gateway.
```

### Architecture — TGW + Direct Connect
```
BBY Data Center (on-premises)
        │
        │ Dedicated fiber circuit
        ▼
Direct Connect Location (colocation facility)
        │
        │ Virtual Interface (VIF)
        ▼
Direct Connect Gateway
        │
        │ Transit VIF
        ▼
Transit Gateway (TGW) ←──── all 15 AWS account VPCs attached here
        │
        ├── VPC-Prod (us-east-1)
        ├── VPC-Staging (us-east-1)
        ├── VPC-Dev (us-east-1)
        ├── Shared Services VPC
        └── Security VPC

Traffic flow (app in Prod VPC → on-prem DB):
  App → Prod VPC route table → TGW → Direct Connect Gateway → DX circuit → on-prem
```

### Terraform — TGW Attachment
```hcl
# Create Transit Gateway (done once in Shared Services account)
resource "aws_ec2_transit_gateway" "main" {
  description                     = "BBY Platform Transit Gateway"
  amazon_side_asn                 = 64512
  auto_accept_shared_attachments  = "enable"
  default_route_table_association = "disable"  # we manage route tables manually
  default_route_table_propagation = "disable"
  
  tags = { Name = "bby-platform-tgw" }
}

# Attach Prod VPC to TGW
resource "aws_ec2_transit_gateway_vpc_attachment" "prod" {
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.prod.id
  subnet_ids         = aws_subnet.prod_private[*].id
  
  tags = { Name = "prod-vpc-tgw-attachment" }
}

# Route in Prod VPC: send on-prem traffic to TGW
resource "aws_route" "prod_to_onprem" {
  route_table_id         = aws_route_table.prod_private.id
  destination_cidr_block = "172.16.0.0/12"   # on-prem CIDR
  transit_gateway_id     = aws_ec2_transit_gateway.main.id
}
```

### Route 53 Private Hosted Zones (Bonus — related to on-prem)
```
What it is:
  A private DNS zone that resolves only within your VPC.
  Not visible on the public internet.

Example:
  Public DNS:  api.mybbyapp.com → 52.x.x.x (public ALB)
  Private DNS: api.mybbyapp.internal → 10.0.2.45 (internal service, no internet exposure)

Why it matters for on-prem integration:
  On-prem systems need to resolve AWS service names privately
  
  Solution: Route 53 Resolver
    ├── Inbound endpoint  → on-prem DNS queries → Route 53 private zone
    └── Outbound endpoint → AWS Lambda/ECS queries → on-prem DNS

Terraform:
  resource "aws_route53_zone" "private" {
    name = "platform.internal"
    vpc {
      vpc_id = aws_vpc.main.id
    }
  }
  
  resource "aws_route53_record" "redis" {
    zone_id = aws_route53_zone.private.zone_id
    name    = "redis.platform.internal"
    type    = "CNAME"
    ttl     = 60
    records = [aws_elasticache_replication_group.redis.primary_endpoint_address]
  }
```

### Your Talking Point
> "In Project 4, Transit Gateway was the backbone for cross-account VPC routing across 15 accounts — all spoke VPCs attach to a central TGW, and route tables control which accounts can reach which. Extending this to on-prem is a Direct Connect Gateway attachment to the same TGW. The routing principle is identical: add the Direct Connect Gateway as another TGW attachment, add the on-prem CIDR to TGW route tables, update spoke VPC route tables. BBY likely has the DX circuit already provisioned — my job is the TGW attachment and route configuration."

---

---

# PART 3 — Quick Reference & Interview Talking Points

---

## 3.1 JD Responsibility → Your Project Mapping

| JD Requirement | Your Project | Key Proof Point |
|---|---|---|
| Account vending, VPC topology, NAT, ALB | Project 4 + Project 3 | 15 accounts, 9 subnets, 3 AZs |
| Transit Gateway peering to on-prem | Project 4 (extend TGW) | Cross-account routing backbone |
| Neptune Serverless | Projects 3+4 pattern | Managed DB provisioning via Terraform |
| OpenSearch Serverless | Elasticsearch knowledge | API-compatible fork |
| ElastiCache Redis cluster mode | Project 3 (extend) | Hash slots, shard-by-run-id pattern |
| RDS Postgres | Project 3 | Aurora Multi-AZ, <30s failover |
| S3, ECR | Projects 1, 7, 8 | CRR, Cosign, Trivy scanning |
| IAM, IRSA, KMS, Secrets Manager | Projects 3, 4 | No static credentials anywhere |
| Bedrock (Claude, Titan) | Project 5 (natural evolution) | EventBridge→SQS→Lambda→Bedrock |
| Cross-region inference profiles | Project 8 + Bedrock concept | Same multi-region pattern |
| Terraform/CDK modules | All projects | Full module library: VPC/EKS/RDS/ALB |
| Remote state, environment gates | Projects 1, 3, 4 | S3 state + DynamoDB locking |
| CI/CD: image build, ECR, Helm deploy | Project 1 | 18-stage Jenkins + ArgoCD |
| ECS Fargate deployment | Project 1 (same pipeline, different deploy cmd) | Pipeline stages 1-11 identical |
| Helm charts for 11 services | Projects 1, 3 | Env-specific value overrides |
| HPA on stateless services | Projects 3, K8s doc | CPU-based autoscaling |
| ALB ingress with OIDC/SSO | Projects 3, 4 | IAM Identity Center + OIDC |
| OTel SDK + ADOT sidecars | Project 6 (sidecar pattern) | Istio Envoy → ADOT same pattern |
| Single trace ID propagation | Project 6 (Jaeger tracing) | W3C TraceContext across 15 services |
| Redis event bus cluster mode | Project 3 (ElastiCache) | Cluster mode + shard-by-hash-tag |
| EKS migration trigger criteria | K8s environments comparison doc | Fargate limits → EKS decision matrix |
| Zero-trust networking | Projects 4, 6 | mTLS STRICT + default-deny NetworkPolicies |
| SCP interpretation | Project 4 | 15-account governance |
| Python scripting | Projects 5, 7 | Lambda functions, automation scripts |

---

## 3.2 Gap Talking Points — Say These in the Interview

**Amazon Bedrock:**
> "I haven't used Bedrock in production yet, but I built the exact architecture it would plug into — Project 5's serverless remediation engine (EventBridge → SQS → Lambda). Bedrock is the intelligence layer replacing hardcoded if/else logic. The Lambda IAM role gets `bedrock:InvokeModel` added, the function calls Claude via boto3, and the rest of the infrastructure doesn't change. I understand cross-region inference profiles and the SCP implications around model access."

**OTel / ADOT:**
> "In Project 6, I implemented Istio's Envoy sidecar — injected automatically into every pod, collecting traces to Jaeger. ADOT is the same sidecar pattern, AWS-native. The OTel SDK handles W3C TraceContext propagation, which is how I implemented distributed tracing across 15 microservices in Project 6. ADOT exports to X-Ray instead of Jaeger, but the propagation standard and sidecar injection pattern are identical."

**Redis Cluster Mode:**
> "In Project 3, I deployed ElastiCache Redis as the caching layer. For this platform's event bus use case, cluster mode is the right choice. The shard-by-run-id pattern uses Redis hash tags — curly braces around the run-id ensure all events for one orchestration run hash to the same slot and the same shard. Cluster mode gives horizontal write scaling and each shard has its own replica for HA, the same principle as Aurora Multi-AZ in Project 3."

**Neptune Serverless:**
> "Neptune is new to me specifically, but the provisioning pattern is identical to Aurora and ElastiCache in Projects 3 and 4 — Terraform resource, VPC subnet placement, security group, IAM auth, KMS encryption. Neptune Serverless adds auto-scaling NCUs, similar to Lambda in Project 5. I'd ramp up on Gremlin query language; the infrastructure layer is familiar ground."

**OpenSearch Serverless:**
> "OpenSearch is AWS's fork of Elasticsearch — same query DSL, same mapping API. I have deep Elasticsearch expertise covering indexing, search, aggregations, security, and backup. OpenSearch Serverless removes cluster management — no shard planning, no instance sizing, automatic capacity. I'd provision it with a Terraform collection resource, VPC endpoint, and data access policy."

**ECS Fargate:**
> "My Project 1 pipeline handles everything through ECR push identically for Fargate. The difference is the deployment step: Helm upgrade becomes aws ecs update-service with a new task definition revision. The CI/CD approval gate pattern, the image build, the ECR push, the Trivy scan — all the same. ECS task definitions are equivalent to Kubernetes Deployments, just ECS-native syntax."

**Transit Gateway + Direct Connect:**
> "In Project 4, TGW is the routing hub across 15 accounts. Extending to on-prem means attaching a Direct Connect Gateway to that same TGW. BBY likely has the DX circuit — my job is the TGW attachment, route table propagation for the on-prem CIDR, and Route 53 Resolver endpoints so on-prem systems can resolve AWS private DNS names."

---

## 3.3 Numbers to Remember

| Metric | Number | Source |
|---|---|---|
| Cost saved | 35% ($180K/year) | Project 7 |
| Accounts managed | 15 | Project 4 |
| Pipeline stages | 18 | Project 1 |
| Security layers in pipeline | 6 | Project 1 |
| Blast radius reduction | 100% → 5% | Project 1 |
| Microservices under mTLS | 15 | Project 6 |
| MTTR improvement | 2 hours → <5 minutes | Project 6 |
| DR RTO | <3 minutes | Project 8 |
| DR RPO | <1 second | Project 8 |
| Servers patched monthly | 500+ | Project 9 |
| Image size reduction | 900MB → 150MB | Project 1 |
| Aurora failover time | <30 seconds | Project 3 |
| Account provisioning | 2 weeks → 30 minutes | Project 4 |
| SOC2 | Passed first attempt | Project 4 |
| Node cost reduction | 60-70% | Project 7 |
| Serverless remediation cost | $0.07/month | Project 5 |
| Serverless fix time | 90 seconds | Project 5 |
| Throughput handled | 5,000 req/s | Project 3 |

---

## 3.4 Decision Frameworks (Interviewers Love These)

### When to use ECS Fargate vs EKS
```
Use ECS Fargate when:
  ✅ Starting a new platform (faster to production)
  ✅ Small team, low operational overhead needed
  ✅ Simple microservices without complex scheduling needs
  ✅ Budget conscious (no $73/mo cluster fee)
  ✅ Task execution < 15 min bursts

Migrate to EKS when:
  ✅ Need advanced scheduling (node affinity, taints, tolerations)
  ✅ Fargate task limits hit (4 vCPU, 30GB RAM max per task)
  ✅ Need DaemonSets (ADOT, security agents on every node)
  ✅ Need Helm ecosystem fully
  ✅ Need cross-namespace service mesh
  ✅ Team has Kubernetes expertise
```

### When to use Serverless Lambda vs Containers vs VMs
```
Lambda: < 15min execution, event-driven, variable/unpredictable traffic, $0 at idle
ECS/EKS: long-running services, steady traffic, need full control
EC2: legacy apps, need full OS, GPU workloads, consistent high compute
```

### When to use Neptune vs RDS vs DynamoDB
```
Neptune:    relationship-heavy data, graph traversal, AI knowledge graphs
RDS:        structured tabular data, complex queries, ACID transactions
DynamoDB:   key-value/document, single-digit millisecond at any scale, no joins needed
```

### When to use OpenSearch vs RDS vs DynamoDB
```
OpenSearch:   full-text search, log analytics, complex filtering, relevance scoring
RDS:          structured queries, aggregations, joins
DynamoDB:     simple key-value lookups at scale
```

---

## 3.5 The "Tell Me About Yourself" Opening (Tie to This Role)

> "I'm a Senior DevOps and Cloud Engineer with 10 years at Ericsson. I build platforms — not just deploy apps. My recent work includes a 15-account AWS Landing Zone with full governance, an 18-stage DevSecOps pipeline, a multi-region DR setup with 3-minute RTO, and a FinOps platform that saved 35% of our cloud spend.
>
> For this role specifically — building the AWS environment for an AI platform from scratch — I've done every component of that: account provisioning, VPC design, IAM and IRSA for zero-credential-everywhere, Terraform module libraries, containerizing multiple services with Helm, EKS at scale, and sidecar-based observability. The gaps for me are the AI-specific services — Bedrock and Neptune — but the infrastructure patterns are the same ones I've been running in production."

---

## 3.6 Study Priority for Your One Week

```
Day 1-2:   Read PART 2 sections 2.1 (Bedrock) and 2.2 (OTel/ADOT)
           Practice the talking points out loud
           
Day 3:     Read PART 2 sections 2.3 (Redis cluster) and 2.6 (ECS Fargate)
           Review your Project 1 and Project 3 documentation
           
Day 4:     Read PART 2 sections 2.4 (Neptune) and 2.5 (OpenSearch)
           Review your Elasticsearch learning notes
           
Day 5:     Read PART 2 sections 2.7 (Transit Gateway + Direct Connect)
           Review your Project 4 documentation
           
Day 6:     Full mock interview — answer every question in Section 3.1
           Practice numbers from Section 3.3
           
Day 7:     Rest. You're ready.
```

---

*Document created: August 2026*  
*Role: Custom Software Engineering Lead / AWS Platform Engineer*  
*Status: Interview-ready — 8.2/10 overall coverage*
