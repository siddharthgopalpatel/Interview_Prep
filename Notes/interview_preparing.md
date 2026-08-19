# Interview Preparation — JD Breakdown (Simple English)

**Role:** Custom Software Engineering Lead  
**Focus:** AWS Platform Engineering (Containers — ECS/EKS)  
**Author:** Siddharth Patel

---

## About the Role

> "Platform runs on AWS inside the AWS tenant. Before any agent can think or any connector can ingest, the environment must exist: accounts provisioned, networking locked down, IAM correct, managed services live, and CI/CD flowing.
> This engineer builds the AWS environment from scratch, containerizes all eleven services, writes the Helm charts, wires the OTel sidecars, and keeps everything running as the platform scales from proof-of-concept to production."

---

### What This Means — In Simple English

**You build the foundation. Everything else depends on you.**

This platform is an AI system (agents that "think" + connectors that pull data). But none of that can run until someone builds the AWS environment underneath it.

```
┌─────────────────────────────────────────────────────┐
│  WHAT DEPENDS ON YOU (AI team builds these):        │
│  • AI Agents (analyze, reason, respond)             │
│  • Connectors (pull data from external systems)     │
│  ↑↑↑ Can't work until everything below exists ↑↑↑  │
├─────────────────────────────────────────────────────┤
│  WHAT YOU BUILD:                                    │
│                                                     │
│  5. CI/CD flowing                                   │
│     → Code → Build → Test → Deploy (automated)     │
│                                                     │
│  4. Managed services live                           │
│     → Redis, Neptune, OpenSearch, RDS, S3 running   │
│                                                     │
│  3. IAM correct                                     │
│     → Right permissions, zero passwords anywhere    │
│                                                     │
│  2. Networking locked down                          │
│     → VPC, subnets, nothing open unless required    │
│                                                     │
│  1. Accounts provisioned                            │
│     → Dev/Staging/Prod AWS accounts created & wired │
└─────────────────────────────────────────────────────┘
```

**Analogy:** You're building the stage, lights, sound system, and security before the performers (AI agents) walk on. No stage = no show.

**Your proof points for each:**

| JD Phrase | Your Experience |
|---|---|
| Accounts provisioned | 15-account Landing Zone, 5 OUs (Project 4) |
| Networking locked down | 9 subnets, 3 AZs, VPC endpoints, default-deny (Project 3) |
| IAM correct | IRSA + OIDC + Secrets Manager — no static credentials (Projects 3, 4) |
| Managed services live | Aurora, ElastiCache, DynamoDB via Terraform (Projects 3, 5) |
| CI/CD flowing | 18-stage pipeline, ECR push, Helm deploy, approval gates (Project 1) |

---

### What is "AWS Tenant"?

"Tenant" = the company's private space inside AWS.

Think of AWS as a large apartment building:
- AWS owns the building (hardware, data centers, global infrastructure)
- Each company **rents a floor** = their "tenant"
- BBY (Best Buy) has their own floor — their own AWS Organization with multiple accounts
- Nobody else can see inside their floor. They control who enters.

```
AWS (the building)
│
├── Tenant: Best Buy (BBY)          ← THIS is what "AWS tenant" means
│   ├── Prod Account
│   ├── Staging Account
│   ├── Dev Account
│   └── Security Account
│
├── Tenant: Some Other Company
│   └── Their accounts...
│
└── Tenant: Another Company
    └── Their accounts...
```

**Technically:** "AWS tenant" = their **AWS Organization** — the root-level container that holds all their accounts, SCPs, billing, and governance.

**Why the JD says this:** It's telling you the platform does NOT run on BBY's on-premises data centers. Everything is inside AWS, under BBY's organizational control. You build within their tenant boundaries and their SCPs (guardrails).

---

### Your Five Core Responsibilities (Second Sentence Breakdown)

> "This engineer builds the AWS environment from scratch, containerizes all eleven services, writes the Helm charts, wires the OTel sidecars, and keeps everything running as the platform scales from proof-of-concept to production."

**1. "Builds the AWS environment from scratch"**
— You start with nothing. No VPC, no accounts, no databases. You create the entire infrastructure using Terraform/CDK. From zero to a fully working platform.

**2. "Containerizes all eleven services"**
— The platform has 11 microservices (likely: orchestrator, multiple AI agents, connectors, API gateway, etc.). You write the Dockerfile for each one — taking the application code and packaging it into a container image.

**3. "Writes the Helm charts"**
— For each of those 11 services, you write the Helm chart that defines how it runs: how many replicas, what CPU/memory, what environment variables, what secrets, how it scales (HPA), how traffic reaches it (ingress). One chart template, different values per environment (dev/staging/prod).

**4. "Wires the OTel sidecars"**
— Every container gets a sidecar (ADOT collector) attached to it. You configure this so that when Service A calls Service B calls Bedrock, a single trace ID follows the entire request. If something is slow, you can see exactly where. "Wires" = configures the sidecar container, sets the OTLP endpoint, ensures trace propagation headers pass between services.

**5. "Keeps everything running as the platform scales from proof-of-concept to production"**
— This is the lifecycle:

```
PoC (proof of concept)          →→→          Production
─────────────────────────────────────────────────────────
• 1-2 users testing             • Thousands of requests
• Fargate (simple, fast start)  • EKS (when Fargate limits hit)
• 1 environment                 • 3 environments (dev/staging/prod)
• Basic monitoring              • Full observability + alerting
• "Does it work?"               • "Is it reliable, secure, cost-efficient?"
```

You don't just build it and leave — you **own it** through that entire journey. When Fargate hits its limits (4 vCPU/30GB max per task), you're the one who migrates to EKS.

**In one line:** You're the single engineer responsible for the platform's entire infrastructure lifecycle — from empty AWS account to production-grade, observable, scalable system.

---

## Responsibilities & Required Skills — SDLC Order

> All JD responsibilities and required skills explained in simple English, arranged by SDLC phase.

```
PLAN → PROVISION → SECURE → BUILD → DEPLOY → OPERATE → MONITOR
```

---

### Phase 1: PLAN & PROVISION (Foundation — before anything runs)

**1. Account Vending**
Automated process to create AWS accounts with security baselines (VPC, IAM, logging, SCPs) pre-applied. Like a vending machine — team requests, account comes out ready. Lives under AWS Organizations.

**2. VPC Topology**
Virtual Private Cloud — your private network in AWS. You design the CIDR, subnets (public/private/data), across multiple AZs. Controls all traffic flow in and out. Foundation for everything else.

**3. NAT Gateway**
Sits in public subnet, gives private subnet resources (containers, databases) outbound-only internet access. Used for pulling updates, calling external APIs. No inbound allowed.

**4. Transit Gateway**
Central networking hub connecting multiple VPCs + on-premise data centers (via Direct Connect). Hub-and-spoke model — instead of 100+ VPC peering connections, all VPCs attach to one TGW. In this JD: connects AI platform to BBY's on-prem systems.

**5. ALB (Application Load Balancer)**
Layer 7 (HTTP/HTTPS) load balancer that distributes traffic to ECS tasks or EKS pods. Supports path-based routing, host-based routing, and **OIDC authentication** (users must login via SSO before traffic reaches your containers).

---

### Phase 2: SECURE (Identity & Encryption — before any service starts)

**6. IAM Roles**
Defines WHO can do WHAT on which AWS resource. Used by services (not humans) to access other services securely. Attached policies set permissions. Principle: least privilege — only what's needed, nothing more.

**7. IRSA (IAM Roles for Service Accounts)**
EKS-specific. Pods use Kubernetes Service Accounts annotated with IAM Role ARN to access AWS services. Temporary credentials via OIDC — no keys stored anywhere. On ECS Fargate, the equivalent is **Task Role**.

**8. KMS (Key Management Service)**
Create and manage encryption keys. Every service (RDS, S3, Neptune, Secrets Manager, OpenSearch) encrypts data at rest using KMS keys. You control who can use each key via key policy. Supports automatic rotation.

**9. Secrets Manager**
Stores secrets (DB passwords, API keys) encrypted with KMS. Applications fetch secrets at runtime — never stored in code, environment variables, or Docker images. Supports automatic secret rotation.

---

### Phase 3: BUILD (Managed Services — the data and AI layer)

**10. RDS Postgres**
Relational database (SQL, schema, tables). Stores structured data — user configs, application state, metadata. AWS manages patching, backups, failover. Multi-AZ for HA.

**11. Neptune Serverless**
Graph database — stores data as nodes (things) and edges (relationships). Used for AI agents' knowledge graph (entities and how they relate). "Serverless" = auto-scales, no instance sizing, pay for what you use.

**12. OpenSearch Serverless**
Search and analytics engine (AWS fork of Elasticsearch). Used for full-text search, log analytics, searching AI conversation history. NOT just for logs — it's a search engine. "Serverless" = no cluster management, auto-scales.

**13. ElastiCache Redis (Cluster Mode)**
In-memory data store. In this JD: used as an **event bus** (messaging system between AI orchestrators), NOT just a cache. Cluster mode = data sharded across multiple nodes. "Shard-by-run-id" = all events for one AI run go to the same shard using hash tags.

**14. S3**
Object storage — any file type (documents, ML artifacts, pipeline outputs, logs). Supports versioning, encryption, lifecycle policies. In this platform: stores documents AI agents read, model artifacts, build outputs.

**15. Bedrock**
Managed AI service. Call an API → get AI response (from Claude Haiku, Sonnet, Titan). No GPUs, no model training, no hosting. Pay per API call. "Cross-region inference profiles" = AWS routes to whichever region has capacity. SCPs control which models/regions are allowed.

**16. EventBridge**
Serverless event router. AWS services emit events → EventBridge matches patterns against your rules → routes to targets (Lambda, SQS, etc.). "If-this-then-that" for AWS. Triggers automation when things happen (security violations, scaling events).

---

### Phase 4: CONTAINERIZE & DEPLOY (CI/CD — shipping code to production)

**17. ECR (Elastic Container Registry)**
Private Docker image storage inside your AWS account. You build images → push to ECR → ECS/EKS pulls from here. Pipeline scans images (Trivy) and signs them (Cosign) before pushing.

**18. ECS (Elastic Container Service)**
AWS's own container orchestration. Schedules and manages containers. Uses Task Definitions (which image, CPU, memory, secrets, IAM role). Simpler than Kubernetes but AWS-only. Starting point for this platform.

**19. Fargate**
Serverless compute engine for containers. Works with both ECS and EKS. You define CPU/memory — AWS runs it. No servers to manage or patch. Limits: max 4 vCPU, 30GB RAM per task. This platform starts here.

**20. EKS (Elastic Kubernetes Service)**
AWS-managed Kubernetes. Full K8s ecosystem: Helm, HPA, DaemonSets, service mesh, NetworkPolicies. Migration target when Fargate limits are hit (need >4 vCPU, DaemonSets, advanced scheduling).

**ECS/ECR/EKS/Fargate — How They Fit Together:**

```
ECR (stores images) → ECS or EKS (orchestrates) → Fargate or EC2 (runs)

This JD's path:
  Phase 1: ECR → ECS + Fargate  (simple, fast, PoC → early prod)
  Phase 2: ECR → EKS + EC2      (when Fargate limits hit)

Combinations that exist:
  ECS + EC2       → ECS schedules on EC2 instances YOU manage
  ECS + Fargate   → ECS schedules, AWS manages servers       ← JD starts here
  EKS + EC2       → Kubernetes on EC2 nodes (with Karpenter) ← JD migrates here
  EKS + Fargate   → Kubernetes pods on Fargate (no nodes)

Migration triggers (Fargate → EKS):
  • Need more than 4 vCPU or 30GB RAM per task
  • Need DaemonSets (e.g., ADOT collector on every node)
  • Need advanced scheduling (node affinity, taints)
  • Need full Helm/service mesh ecosystem
```

---

### Phase 5: OPERATE & MONITOR (Keeping it running at scale)

All the above items operate together:
- HPA scales stateless services based on CPU/memory
- ADOT sidecars collect traces from every container
- Redis event bus handles inter-service communication
- ALB routes and authenticates incoming traffic
- Bedrock cross-region profiles handle AI call routing

---

### SDLC Flow Diagram

```
┌─────────┐    ┌──────────┐    ┌─────────┐    ┌──────────────┐    ┌─────────┐
│  PLAN   │───▶│  SECURE  │───▶│  BUILD  │───▶│   DEPLOY     │───▶│ OPERATE │
│         │    │          │    │         │    │              │    │         │
│ Account │    │ IAM      │    │ RDS     │    │ ECR          │    │ HPA     │
│ VPC     │    │ IRSA     │    │ Neptune │    │ ECS/Fargate  │    │ Redis   │
│ NAT     │    │ KMS      │    │ OpenSrch│    │ EKS          │    │ ADOT    │
│ TGW     │    │ Secrets  │    │ Redis   │    │ Helm         │    │ ALB     │
│ ALB     │    │ Manager  │    │ S3      │    │ CI/CD        │    │ Bedrock │
│         │    │          │    │ Bedrock │    │              │    │         │
│         │    │          │    │ EventBr │    │              │    │         │
└─────────┘    └──────────┘    └─────────┘    └──────────────┘    └─────────┘
```

---

## Terraform Workflow for This Platform

> **JD:** "Write Terraform or CDK modules for all AWS resources, maintain remote state and environment promotion gates (dev staging prod)"
> **Required Skill:** "Terraform or AWS CDK: module design, remote state, environment promotion, secrets management"

---

### What This Means

You write **reusable Terraform modules** for every AWS resource (VPC, ECS, RDS, Redis, Neptune, etc.), store state remotely (S3 + DynamoDB), and promote the same infrastructure code through **dev → staging → prod** with approval gates between each.

---

### Repository Structure

```
terraform/
├── modules/              ← Reusable building blocks
│   ├── vpc/
│   ├── ecs-service/
│   ├── rds/
│   ├── redis/
│   ├── neptune/
│   ├── opensearch/
│   └── iam-role/
│
├── environments/         ← Environment-specific configs
│   ├── dev/
│   │   ├── main.tf      (calls modules with dev values)
│   │   ├── variables.tf
│   │   └── terraform.tfvars (dev-specific: small size)
│   ├── staging/
│   │   ├── main.tf      (same modules, staging values)
│   │   └── terraform.tfvars (staging: medium size)
│   └── prod/
│       ├── main.tf      (same modules, prod values)
│       └── terraform.tfvars (prod: full size, multi-AZ)
│
└── backend.tf            ← Remote state config
```

---

### Key Concepts

**1. Modules = Reusable building blocks**

Write one VPC module. All 3 environments use it with different values:

```hcl
# environments/dev/main.tf
module "vpc" {
  source       = "../../modules/vpc"
  cidr         = "10.0.0.0/16"
  azs          = 2              # dev: 2 AZs (cheaper)
  nat_gateways = 1              # dev: 1 NAT (cheaper)
}

# environments/prod/main.tf
module "vpc" {
  source       = "../../modules/vpc"
  cidr         = "10.1.0.0/16"
  azs          = 3              # prod: 3 AZs (HA)
  nat_gateways = 3              # prod: 1 NAT per AZ (HA)
}
```

Same module, different inputs. Change module once → all environments get the fix.

**2. Remote State = S3 + DynamoDB**

```hcl
terraform {
  backend "s3" {
    bucket         = "bby-platform-terraform-state"
    key            = "prod/terraform.tfstate"    # separate key per environment
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"           # prevents two people applying at once
  }
}
```

```
Separate state per environment:
  dev/terraform.tfstate     → only tracks dev resources
  staging/terraform.tfstate → only tracks staging resources
  prod/terraform.tfstate    → only tracks prod resources

If dev state corrupts → prod is completely unaffected.
```

**3. Environment Promotion Gates = CI/CD Pipeline**

```
PR merged to main
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│  CI/CD Pipeline (GitHub Actions)                            │
│                                                             │
│  ┌─────────────────────────┐                               │
│  │  DEV                    │                               │
│  │  terraform plan         │  ← shows what will change     │
│  │  terraform apply (auto) │  ← applies automatically      │
│  └───────────┬─────────────┘                               │
│              │                                              │
│              ▼                                              │
│  ┌─────────────────────────┐                               │
│  │  STAGING                │                               │
│  │  terraform plan         │  ← shows what will change     │
│  │  terraform apply (auto) │  ← applies after dev passes   │
│  └───────────┬─────────────┘                               │
│              │                                              │
│         ⛔ APPROVAL GATE ⛔  ← human clicks "Approve"       │
│              │                                              │
│              ▼                                              │
│  ┌─────────────────────────┐                               │
│  │  PROD                   │                               │
│  │  terraform plan         │  ← shows what will change     │
│  │  (review plan output)   │  ← team reviews the plan      │
│  │  terraform apply        │  ← applies ONLY after approval│
│  └─────────────────────────┘                               │
└─────────────────────────────────────────────────────────────┘
```

**4. Secrets Management in Terraform**

Terraform creates the secret container but **never stores the secret value in code or state**:

```hcl
# Create the secret container (Terraform manages this)
resource "aws_secretsmanager_secret" "db_password" {
  name       = "prod/rds/password"
  kms_key_id = aws_kms_key.secrets.arn
}

# The actual password value is set OUTSIDE Terraform:
#   - Rotated automatically by Secrets Manager
#   - Or set manually once, never in Git
#   - Terraform only creates the "box", not what's inside
```

---

### Real Example: Deploying RDS Postgres to Prod

```
Step 1: Write the module (once)
  modules/rds/main.tf → creates RDS instance, subnet group, security group, KMS key

Step 2: Prod calls the module with prod values
  environments/prod/main.tf:
    module "rds" {
      source            = "../../modules/rds"
      engine            = "postgres"
      instance_class    = "db.r6g.xlarge"    # prod = large
      multi_az          = true                # prod = HA
      storage_encrypted = true
      kms_key_id        = module.kms.key_arn
    }

Step 3: CI/CD runs
  terraform plan  → "Will create: RDS instance, subnet group, SG..."
  ⛔ Approval gate → lead reviews plan
  terraform apply → RDS created in prod

Step 4: State updated
  prod/terraform.tfstate now tracks this RDS instance
  Next time you run plan → Terraform knows it already exists
```

---

### One-Line Summary

You write Terraform modules once → each environment calls them with different values → CI/CD promotes through dev/staging/prod with approval gates → state stored in S3 per environment → secrets never in code.

---

## CI/CD Pipeline + Containers + Helm

> **JD:** "Build and own the CI/CD pipeline: container image builds, ECR push, Helm deploy to ECS Fargate, with approval gates between environments"
> **JD:** "Write Dockerfiles and Helm charts for all eleven services, configure HPA on stateless services and ALB ingress with OIDC/SSO"
> **Required Skills:** "Container platforms: Docker, Helm, Kubernetes (EKS) scaling stateless microservices with HPA"
> **Required Skills:** "CI/CD pipelines: Jenkins — image build, test, push, Helm deploy with approval gates"

---

### What This Means

You build the entire pipeline: developer pushes code → Jenkins builds Docker image → scans it → pushes to ECR → deploys to ECS Fargate using Helm charts → promotes through dev/staging/prod with approval gates. You also write the Dockerfiles (one per service × 11 services), the Helm charts (how each service runs), configure HPA (auto-scaling), and ALB ingress with OIDC (authentication before traffic reaches the app).

---

### The Full CI/CD Flow (Jenkins)

```
Developer pushes code to Git
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│  JENKINS PIPELINE                                           │
│                                                             │
│  Stage 1: Checkout                                          │
│     └── git pull latest code                                │
│                                                             │
│  Stage 2: Unit Tests                                        │
│     └── run tests, fail fast if broken                      │
│                                                             │
│  Stage 3: Docker Build                                      │
│     └── docker build -t my-service:${GIT_SHA} .             │
│         (multi-stage: build deps → slim runtime image)      │
│                                                             │
│  Stage 4: Security Scan                                     │
│     └── trivy image my-service:${GIT_SHA}                   │
│         (block if HIGH/CRITICAL vulnerabilities found)      │
│                                                             │
│  Stage 5: ECR Push                                          │
│     └── docker push 123456.dkr.ecr.us-east-1/my-service:${GIT_SHA}│
│                                                             │
│  Stage 6: Deploy to DEV (automatic)                         │
│     └── helm upgrade --install my-service ./chart \         │
│           --set image.tag=${GIT_SHA} \                      │
│           -f values-dev.yaml                                │
│                                                             │
│  Stage 7: Smoke Tests on DEV                                │
│     └── curl health endpoint, verify 200 OK                 │
│                                                             │
│  Stage 8: Deploy to STAGING (automatic)                     │
│     └── helm upgrade --install my-service ./chart \         │
│           --set image.tag=${GIT_SHA} \                      │
│           -f values-staging.yaml                            │
│                                                             │
│  Stage 9: ⛔ APPROVAL GATE                                  │
│     └── human reviews → clicks "Approve" in Jenkins         │
│                                                             │
│  Stage 10: Deploy to PROD                                   │
│     └── helm upgrade --install my-service ./chart \         │
│           --set image.tag=${GIT_SHA} \                      │
│           -f values-prod.yaml \                             │
│           --atomic --timeout 5m                             │
│         (--atomic = auto rollback if deploy fails)          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Dockerfile — What You Write for Each Service

```dockerfile
# Stage 1: Build (heavy — has all build tools)
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: Production (slim — only runtime)
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY . .
RUN useradd -m appuser && chown -R appuser /app
USER appuser
EXPOSE 8080
CMD ["gunicorn", "--workers=3", "--bind=0.0.0.0:8080", "app:application"]
```

**Key points:**
- Multi-stage build → small image (900MB → 150MB)
- Non-root user → security
- No secrets in Dockerfile → pulled from Secrets Manager at runtime
- Tag with Git SHA (never `:latest`) → traceability

---

### Helm Chart — What You Write for Each Service

```
charts/my-service/
├── Chart.yaml              ← name, version
├── values.yaml             ← defaults (dev)
├── values-staging.yaml     ← staging overrides
├── values-prod.yaml        ← prod overrides
└── templates/
    ├── deployment.yaml     ← how many pods, what image, CPU/memory
    ├── service.yaml        ← internal networking (ClusterIP)
    ├── ingress.yaml        ← ALB routing rules + OIDC auth
    ├── hpa.yaml            ← auto-scaling rules
    └── serviceaccount.yaml ← IRSA annotation for AWS access
```

**values-dev.yaml vs values-prod.yaml:**

```yaml
# values-dev.yaml                    # values-prod.yaml
replicaCount: 1                      replicaCount: 3
resources:                           resources:
  cpu: 256m                            cpu: 1024m
  memory: 512Mi                        memory: 2048Mi
hpa:                                 hpa:
  enabled: false                       enabled: true
  minReplicas: 1                       minReplicas: 3
  maxReplicas: 5                       maxReplicas: 20
ingress:                             ingress:
  oidc: false                          oidc: true
```

---

### HPA — Horizontal Pod Autoscaler (Stateless Services Only)

**What:** Automatically adds/removes pods based on CPU or memory usage.

**Why "stateless services only":** Stateless = no data stored inside the pod. Any pod can handle any request. Safe to add/remove pods freely. Stateful services (like Redis) can't just be added/removed — they hold data.

```yaml
# templates/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    kind: Deployment
    name: my-service
  minReplicas: {{ .Values.hpa.minReplicas }}
  maxReplicas: {{ .Values.hpa.maxReplicas }}
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # scale up when CPU > 70%
```

```
How it works:
  CPU at 40% → 3 pods (minimum)      ← idle, no scaling
  CPU at 75% → 5 pods                ← HPA adds pods
  CPU at 90% → 10 pods               ← HPA adds more
  CPU drops to 30% → 3 pods (after 5 min cooldown)  ← HPA removes pods
```

---

### ALB Ingress with OIDC/SSO

**Problem:** You don't want unauthenticated users hitting your services.

**Solution:** ALB handles authentication BEFORE traffic reaches your containers. User must login via SSO (OIDC) first. If not logged in → ALB redirects to login page. If logged in → ALB forwards request to container.

```
User hits https://platform.bby.com
        │
        ▼
┌───────────────────────────────────┐
│  ALB                              │
│                                   │
│  Step 1: Is user authenticated?   │
│     NO → redirect to SSO login    │
│     YES → forward to container    │
│                                   │
│  Step 2: Route by path            │
│     /api/orchestrator → Service A │
│     /api/agents → Service B       │
│     /api/connectors → Service C   │
└───────────────────────────────────┘
```

```yaml
# templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/auth-type: oidc
    alb.ingress.kubernetes.io/auth-idp-oidc:
      issuer: "https://login.bby.com"
      authorizationEndpoint: "https://login.bby.com/authorize"
      tokenEndpoint: "https://login.bby.com/oauth/token"
      userInfoEndpoint: "https://login.bby.com/userinfo"
      secretName: oidc-secret
spec:
  rules:
  - host: platform.bby.com
    http:
      paths:
      - path: /api/orchestrator
        service:
          name: orchestrator-service
          port: 8080
```

**Simple English:** Users hit ALB → ALB checks "are you logged in?" → if yes, routes to correct service. No authentication code needed inside your containers.

---

### How All 11 Services Get Deployed

```
11 services × same pattern:

Each service gets:
  1. Dockerfile          → how to build it
  2. Helm chart          → how to run it (replicas, CPU, secrets, scaling)
  3. Pipeline stage      → build → scan → push → deploy
  4. HPA (if stateless)  → auto-scale based on load
  5. ALB ingress rule    → path-based routing + OIDC

All 11 share:
  • Same ECR registry (different image names)
  • Same ALB (different paths)
  • Same Helm chart structure (different values)
  • Same Jenkins pipeline template (parameterized per service)
```

---

### One-Line Summary

Jenkins builds Docker images for 11 services → pushes to ECR → Helm deploys to ECS Fargate (dev auto, staging auto, prod with approval gate) → HPA auto-scales stateless services → ALB routes traffic with OIDC authentication enforced before anything reaches your containers.

---

## Networking

> **Required Skill:** "Networking: VPC design, Transit Gateway, Direct Connect, security groups, Route 53 private hosted zones"

---

### What This Means

You design and build the entire network layer: how traffic flows between services, how the platform connects to BBY's on-prem data centers, how DNS resolves privately, and how every resource is firewalled.

---

### 1. VPC Design

Your private network in AWS. You decide the layout:

```
VPC: 10.0.0.0/16 (65,536 IPs)
│
├── Public Subnets (3 AZs)     → ALB lives here, NAT Gateway lives here
│   └── Can receive traffic FROM internet
│
├── Private Subnets (3 AZs)    → ECS Fargate tasks / EKS pods live here
│   └── NO inbound from internet, outbound via NAT only
│
└── Data Subnets (3 AZs)       → RDS, Redis, Neptune, OpenSearch live here
    └── ZERO internet access, only accepts from private subnets
```

**Key decision:** 3 tiers × 3 AZs = 9 subnets. Each tier has different security rules.

---

### 2. Transit Gateway

Central hub connecting everything:

```
Without TGW (messy):              With TGW (clean):

VPC-A ←→ VPC-B                    VPC-A ──┐
VPC-A ←→ VPC-C                    VPC-B ──┼── TGW ──── On-Prem (Direct Connect)
VPC-B ←→ VPC-C                    VPC-C ──┘
VPC-A ←→ On-Prem
VPC-B ←→ On-Prem                  3 connections instead of 6+
```

TGW route tables control who can talk to whom:
- Prod VPC → can reach on-prem (for enterprise data)
- Dev VPC → CANNOT reach on-prem (security isolation)

---

### 3. Direct Connect

Private physical cable from BBY's data center to AWS. NOT over public internet.

```
BBY Data Center ════ dedicated fiber ════ AWS Direct Connect Location ════ TGW
                     (private, fast,                                        │
                      consistent)                                    All your VPCs
```

**Why not just VPN?**
- VPN = over internet = variable latency, shared bandwidth
- Direct Connect = dedicated fiber = consistent speed, lower latency, more secure

**In this JD:** BBY already has Direct Connect. Your job = attach it to Transit Gateway and configure routing.

---

### 4. Security Groups

Firewall rules attached to each resource. **Stateful** = if you allow traffic in, response automatically goes out.

```
Security Group: "ecs-tasks-sg"
  Inbound:  Allow port 8080 from ALB security group ONLY
  Outbound: Allow all (tasks can call Redis, RDS, Neptune, etc.)

Security Group: "rds-sg"
  Inbound:  Allow port 5432 from ecs-tasks-sg ONLY
  Outbound: None needed (stateful — responses go back automatically)

Security Group: "redis-sg"
  Inbound:  Allow port 6379 from ecs-tasks-sg ONLY

Result:
  Internet → ALB ✅
  Internet → ECS task ❌ (blocked)
  Internet → RDS ❌ (blocked)
  ECS task → RDS ✅ (allowed by SG reference)
  ECS task → Redis ✅
  RDS → Internet ❌ (no route, no SG rule)
```

**Key pattern:** Reference security groups by ID, not IP ranges. "Allow from sg-abc123" = only resources in that SG can connect.

---

### 5. Route 53 Private Hosted Zones

Private DNS that only resolves INSIDE your VPC. Not visible on the public internet.

```
Public DNS (everyone can see):
  api.bby.com → 54.x.x.x (public ALB IP)

Private DNS (only inside your VPC):
  redis.platform.internal     → elasticache-cluster-endpoint.aws...
  neptune.platform.internal   → neptune-cluster-endpoint.aws...
  rds.platform.internal       → rds-instance-endpoint.aws...
  opensearch.platform.internal → opensearch-vpc-endpoint.aws...
```

**Why?**
- Containers call `redis.platform.internal` instead of ugly AWS endpoint names
- If you replace Redis with a different cluster → update DNS record → zero code changes
- On-prem systems can resolve these too via **Route 53 Resolver** (inbound/outbound endpoints)

```
Route 53 Resolver:
  Inbound endpoint  → on-prem DNS queries → resolve AWS private names
  Outbound endpoint → AWS services query   → resolve on-prem DNS names

Example:
  On-prem app calls: redis.platform.internal
  → Route 53 Resolver inbound endpoint receives query
  → Resolves to ElastiCache endpoint
  → Returns IP to on-prem app
```

---

### Complete Network Diagram

```
                         INTERNET
                             │
                        ┌────▼────┐
                        │   ALB   │ (public subnet, OIDC auth)
                        └────┬────┘
                             │
              ┌──────────────▼──────────────┐
              │      Private Subnets         │
              │  ECS Fargate tasks (11 svc)  │
              │  Security Group: ecs-tasks   │
              └──────┬──────────┬──────┬────┘
                     │          │      │
         ┌───────────▼─┐  ┌────▼───┐  ▼────────────┐
         │ Data Subnets │  │ NAT GW │  │ VPC Endpoint│
         │ RDS, Redis,  │  │(outbound│  │ (S3, ECR)  │
         │ Neptune,     │  │ only)   │  │ no NAT cost│
         │ OpenSearch   │  └────────┘  └────────────┘
         └──────────────┘
                │
                │ (via Transit Gateway)
                ▼
         ┌──────────────┐
         │  On-Premise  │ (via Direct Connect)
         │  BBY Data    │
         │  Centers     │
         └──────────────┘

DNS Resolution:
  redis.platform.internal → ElastiCache endpoint (private)
  neptune.platform.internal → Neptune endpoint (private)
  api.bby.com → ALB (public)
```

---

### One-Line Summary

VPC with 3 tiers (public/private/data) across 3 AZs → Transit Gateway connects all VPCs + on-prem via Direct Connect → Security Groups firewall every resource (reference by SG, not IP) → Route 53 private hosted zones give friendly DNS names that only resolve inside the VPC.

---

## OpenTelemetry & ADOT

> **JD:** "Instrument all services with OpenTelemetry SDK and ADOT collector sidecars, ensure a single trace ID propagates from orchestrator through all agents to Bedrock"
> **Required Skill:** "OpenTelemetry / ADOT: SDK instrumentation, collector sidecar configuration, distributed trace propagation"

---

### What This Means

Every one of the 11 services gets instrumented (code-level) with OpenTelemetry SDK + an ADOT sidecar container attached. When a request flows through the system (orchestrator → agent A → agent B → Bedrock), a single trace ID follows it end-to-end. If something is slow or broken, you can see exactly WHERE in one trace view.

---

### The Problem Without This

```
User reports: "The platform is slow"

Without tracing:
  Which of the 11 services is slow? 🤷
  Is it Redis? Neptune? Bedrock? Network? 🤷
  Spend hours checking logs of each service manually

With tracing (single trace ID):
  Open trace ID abc-123 in X-Ray/Jaeger:
    orchestrator     → 50ms  ✅
    agent-service-A  → 200ms ✅
    agent-service-B  → 3.5s  ❌ ← THIS is the problem
      └── bedrock-call → 3.2s    ← specifically, this Bedrock call
  
  Found in 30 seconds.
```

---

### Three Components

```
1. OTel SDK (inside your application code)
   → Creates spans (start/end time for each operation)
   → Attaches trace ID to outgoing HTTP/gRPC calls
   → Available for Python, Go, Java, Node.js

2. ADOT Collector (sidecar container next to your app)
   → Receives telemetry from SDK on localhost:4317
   → Batches, processes, exports to backend
   → AWS-validated version of OTel Collector
   → Uses IRSA/Task Role for AWS auth (no credentials)

3. Backend (where traces are stored and visualized)
   → AWS X-Ray (or Jaeger, or Datadog)
   → You view traces here
```

---

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│  Pod / ECS Task                                                  │
│                                                                  │
│  ┌──────────────────┐       ┌──────────────────────┐           │
│  │  App Container    │       │  ADOT Sidecar         │           │
│  │                   │       │                       │           │
│  │  OTel SDK inside  │──────▶│  Receives on :4317    │──────▶ X-Ray
│  │  sends to         │ gRPC  │  Batches traces       │           │
│  │  localhost:4317   │       │  Exports to backend   │           │
│  └──────────────────┘       └──────────────────────┘           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

App doesn't know about X-Ray. App only talks to localhost:4317 (the sidecar).
Sidecar handles everything else.
```

---

### Trace Propagation — The Key Requirement

**"Single trace ID propagates from orchestrator through all agents to Bedrock"**

```
Request arrives at Orchestrator
│  OTel SDK generates: Trace ID = abc-123, Span = span-001
│
│  Orchestrator calls Agent-A via HTTP
│  OTel SDK auto-injects header:
│    traceparent: 00-abc123-span001-01
│
▼
Agent-A receives request
│  OTel SDK reads traceparent header → knows Trace ID = abc-123
│  Creates child span: span-002 (parent: span-001)
│
│  Agent-A calls Agent-B via HTTP
│  Header: traceparent: 00-abc123-span002-01
│
▼
Agent-B receives request
│  OTel SDK reads header → Trace ID still abc-123
│  Creates child span: span-003 (parent: span-002)
│
│  Agent-B calls Bedrock API
│  OTel SDK wraps this as span: span-004
│
▼
Result in X-Ray:

Trace: abc-123 (total: 1.2s)
├── span-001: orchestrator (50ms)
├── span-002: agent-service-A (200ms)
├── span-003: agent-service-B (800ms)
│   └── span-004: bedrock-invoke-model (750ms)
└── Done
```

**Key:** `traceparent` header (W3C standard) carries the trace ID across service boundaries. OTel SDK automatically injects it on outgoing calls and reads it on incoming calls. Zero manual header passing needed.

---

### SDK Instrumentation — What You Add to Code

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.instrumentation.flask import FlaskInstrumentor

# Setup: send traces to ADOT sidecar on localhost:4317
provider = TracerProvider()
provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="http://localhost:4317"))
)
trace.set_tracer_provider(provider)

# Auto-instrument: all HTTP calls automatically get traceparent header
RequestsInstrumentor().instrument()   # outgoing HTTP calls
FlaskInstrumentor().instrument(app)   # incoming HTTP requests
```

---

### ADOT Sidecar Configuration

```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"

processors:
  batch:
    timeout: 1s
    send_batch_size: 50

exporters:
  awsxray:
    region: us-east-1

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [awsxray]
```

---

### ECS Task Definition with ADOT Sidecar

```json
{
  "family": "orchestrator",
  "containerDefinitions": [
    {
      "name": "orchestrator",
      "image": "123456.dkr.ecr.us-east-1.amazonaws.com/orchestrator:abc123",
      "portMappings": [{ "containerPort": 8080 }],
      "environment": [
        { "name": "OTEL_EXPORTER_OTLP_ENDPOINT", "value": "http://localhost:4317" },
        { "name": "OTEL_SERVICE_NAME", "value": "orchestrator" }
      ]
    },
    {
      "name": "adot-collector",
      "image": "public.ecr.aws/aws-observability/aws-otel-collector:latest",
      "essential": false
    }
  ]
}
```

Every ECS task gets 2 containers: your app + ADOT sidecar. Same pattern for all 11 services.

---

### Maps to Your Experience (Project 6)

```
Your Project 6 (Istio + Jaeger):          This JD (ADOT + X-Ray):
─────────────────────────────────────────────────────────────────
Envoy sidecar (auto-injected)        →    ADOT sidecar (configured per task)
Jaeger (trace backend)               →    AWS X-Ray (trace backend)
Istio propagates trace headers       →    OTel SDK propagates traceparent
15 microservices traced              →    11 services traced
MTTR: 2 hours → 5 minutes           →    Same outcome expected

Same concept. Different tools.
```

---

### One-Line Summary

OTel SDK in each service creates spans + propagates `traceparent` header across all calls → ADOT sidecar (one per task) collects traces on localhost:4317 → exports to X-Ray → you get a single trace view showing the full request journey from orchestrator through agents to Bedrock.

---

## Redis Event Bus

> **JD:** "Manage the Redis event bus (HA cluster mode, shard-by-run id) for the MI and CR orchestrators"
> **Required Skill:** "Redis: cluster-mode configuration, sharding strategies, HA failover"

---

### What This Means

Redis is used here as a **messaging system (event bus)** between AI orchestrators — NOT as a traditional cache. Two orchestrators (MI and CR) communicate by publishing/reading events from Redis. Cluster mode splits data across multiple nodes for scale. "Shard-by-run-id" means all events for one AI run stay on the same shard.

---

### What is an Event Bus?

```
Without event bus (direct calls):
  Orchestrator-MI → calls Agent-A directly → waits → calls Agent-B → waits
  Problem: tightly coupled, if Agent-B is slow everything waits

With event bus (Redis):
  Orchestrator-MI → publishes event to Redis: "run-123: step-1 complete"
  Agent-A → reads event → does work → publishes: "run-123: step-2 complete"
  Agent-B → reads event → does work → publishes: "run-123: step-3 complete"
  
  Benefits:
    • Decoupled — services don't call each other directly
    • Async — nobody waits for anyone
    • Auditable — all events stored in Redis with timestamps
    • Resilient — if Agent-B is down, events wait in Redis until it's back
```

---

### MI and CR Orchestrators

```
MI = Market Intelligence orchestrator
  → Coordinates AI agents that gather and analyze market data

CR = Customer Research orchestrator
  → Coordinates AI agents that analyze customer behavior

Each orchestrator manages "runs":
  Run = one complete execution of a workflow
  Example: "Analyze Q3 market trends for electronics" = one run (run-id: run-abc123)
  
  A run has multiple steps:
    Step 1: Gather data (connector pulls from sources)
    Step 2: Agent-A analyzes trends
    Step 3: Agent-B generates insights
    Step 4: Store results in Neptune
    
  All steps for run-abc123 communicate via Redis events
```

---

### Redis Cluster Mode — Why?

```
Standard Redis (single node):
  One node handles ALL reads/writes
  Max ~30GB memory
  One failure = everything down
  ❌ Not enough for production event bus

Cluster Mode (multiple shards):
  Data SPLIT across 3+ primary nodes (shards)
  Each shard has a replica for failover
  Total memory = sum of all shards
  One shard fails = only that shard's replica takes over, rest unaffected

  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
  │  Shard 1    │  │  Shard 2    │  │  Shard 3    │
  │  Primary    │  │  Primary    │  │  Primary    │
  │  Slots 0-  │  │  Slots 5461-│  │  Slots 10923│
  │  5460       │  │  10922      │  │  -16383     │
  │  + Replica  │  │  + Replica  │  │  + Replica  │
  └─────────────┘  └─────────────┘  └─────────────┘
  
  Total: 16,384 hash slots divided across shards
  3× throughput, 3× memory, HA per shard
```

---

### Shard-by-Run-ID — The Key Concept

**Problem:** A single run has many events. If these land on different shards, you need cross-shard operations (slow, complex).

**Solution:** Force all keys for one run to the SAME shard using **hash tags** (curly braces).

```
Normal (no hash tags):
  CRC16("event:run-abc123:step-1") % 16384 = slot 7823 → Shard 2
  CRC16("event:run-abc123:step-2") % 16384 = slot 2341 → Shard 1  ← DIFFERENT!

With hash tags:
  Key: "{run-abc123}:event:step-1" → Redis only hashes {run-abc123}
  Key: "{run-abc123}:event:step-2" → Same hash → same slot → SAME shard
  Key: "{run-abc123}:status"       → Same hash → same slot → SAME shard

  All keys with {run-abc123} → always land on same shard. Guaranteed.
```

**Application code:**
```python
# All keys for one run use hash tag → same shard
redis.set(f"{{run-{run_id}}}:event:step-1", payload)
redis.set(f"{{run-{run_id}}}:event:step-2", payload)
redis.set(f"{{run-{run_id}}}:status", "running")
redis.lpush(f"{{run-{run_id}}}:events", event_json)

# Reading all events for a run — single shard, fast
events = redis.lrange(f"{{run-{run_id}}}:events", 0, -1)
```

---

### HA Failover

```
Shard 2 primary dies:
  1. ElastiCache detects failure (~15 seconds)
  2. Shard 2 replica promoted to primary (~30 seconds)
  3. Cluster updates slot ownership
  4. Applications reconnect automatically
  5. Total downtime: 15-60 seconds for that shard only
  
  Shards 1 and 3: completely unaffected (zero downtime)
```

---

### How Events Flow

```
MI Orchestrator starts run-abc123
        │
        │ LPUSH {run-abc123}:events {"type":"start","step":1}
        ▼
┌─────────────────────────────────────────────────────┐
│  Redis Cluster (Shard 2 — all run-abc123 keys)      │
│                                                     │
│  {run-abc123}:status = "running"                    │
│  {run-abc123}:events = [                            │
│    {"type":"start", "step":1},                      │
│    {"type":"data_ready", "step":2},                 │
│    {"type":"analysis_done", "step":3}               │
│  ]                                                  │
└────────────────────┬────────────────────────────────┘
                     │
                     │ BRPOP (blocking read — waits for new events)
                     ▼
Agent-A picks up event → processes → LPUSH next event
Agent-B picks up event → processes → LPUSH result
CR Orchestrator reads results from MI's run if needed
```

---

### Terraform

```hcl
resource "aws_elasticache_replication_group" "redis_event_bus" {
  replication_group_id = "ai-platform-event-bus"
  description          = "Redis event bus for MI and CR orchestrators"

  node_type                  = "cache.r7g.large"
  parameter_group_name       = "default.redis7.cluster.on"

  num_node_groups         = 3    # 3 shards
  replicas_per_node_group = 1    # 1 replica per shard = 6 nodes total

  automatic_failover_enabled = true
  multi_az_enabled           = true

  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  subnet_group_name          = aws_elasticache_subnet_group.redis.name
  security_group_ids         = [aws_security_group.redis.id]
}
```

---

### One-Line Summary

Redis cluster mode splits data across 3 shards for scale → hash tags `{run-id}` force all events for one AI run to the same shard → each shard has a replica for automatic failover → MI and CR orchestrators publish/read events asynchronously through this bus without calling each other directly.

---

## EKS Migration Trigger

> **JD:** "Define EKS migration trigger criteria and execute the migration when Fargate limits are exceeded"

---

### What This Means

You start on ECS Fargate (simple, fast). At some point, Fargate can't handle what you need. You define **specific criteria** (not gut feeling) that trigger the migration to EKS. When those criteria are hit, you execute the migration.

---

### Why Start on Fargate at All?

```
Fargate (Phase 1):                    EKS (Phase 2):
─────────────────────                 ─────────────────────
• No cluster to manage                • You manage nodes/cluster
• Deploy in days, not weeks           • Takes weeks to set up properly
• Perfect for PoC → early prod        • Perfect for scale + complex needs
• Low operational overhead             • High operational overhead
• Pay per task (simple)               • Pay for cluster + nodes

Start simple → migrate when forced. Don't over-engineer from day 1.
```

---

### Fargate Hard Limits

```
• Max 4 vCPU per task
• Max 30 GB RAM per task
• Max 200 GB ephemeral storage per task
• No DaemonSets (can't run agent on every node)
• No GPU support
• No privileged containers
• Limited networking control (no host networking)
• Cold start latency (~1-2 min for new tasks)
```

---

### Migration Trigger Criteria

```
Category 1: Resource Limits
  □ Any service needs > 4 vCPU per instance
  □ Any service needs > 30 GB RAM per instance
  □ AI model inference needs GPU

Category 2: Operational Requirements
  □ Need DaemonSets (ADOT collector, security agent, log forwarder on every node)
  □ Need node-level access (custom kernel tuning, sysctl)
  □ Need privileged containers

Category 3: Scheduling & Scaling
  □ Need node affinity / anti-affinity
  □ Need taints/tolerations (dedicated nodes for specific workloads)
  □ Cold start latency unacceptable (Fargate: 1-2 min vs EKS: seconds)
  □ Need Spot instances for cost savings (Karpenter)

Category 4: Ecosystem Requirements
  □ Need service mesh (Istio)
  □ Need NetworkPolicies (pod-to-pod firewall rules)
  □ Need full Helm ecosystem with CRDs
  □ Need multi-tenant namespace isolation

Category 5: Cost
  □ Fargate cost > equivalent EKS + EC2 at scale
  □ Need Spot instances (60-70% savings — not available on Fargate)
```

---

### Most Likely Triggers for THIS Platform

```
1. ADOT as DaemonSet (most likely first trigger)
   → Fargate: ADOT sidecar per task (11 sidecars = 11× cost)
   → EKS: ADOT DaemonSet (1 per node, shared by all pods = cheaper)

2. AI services exceeding 4 vCPU / 30 GB RAM
   → Heavy AI processing may need more resources

3. Cold start latency
   → Fargate: 1-2 min to spin up new tasks
   → EKS: pods schedule on existing nodes in seconds

4. Cost at scale
   → 11 services × 3 replicas × Fargate pricing > EKS + Spot
   → Karpenter with Spot = 60-70% node cost reduction
```

---

### What Changes vs What Stays

```
CHANGES:
  ECS Task Definitions        → Kubernetes Deployments (via Helm)
  ECS Services                → Kubernetes Services
  ALB Target Groups (IP mode) → ALB Ingress Controller
  Task IAM Roles              → IRSA (Service Account → IAM Role)
  ECS Service Auto Scaling    → HPA + Karpenter
  ADOT sidecar per task       → ADOT DaemonSet (shared per node)
  aws ecs update-service      → helm upgrade --install

STAYS THE SAME:
  • ECR (same images, same registry)
  • ALB (same load balancer, different target type)
  • VPC (same network, same subnets)
  • RDS, Redis, Neptune, OpenSearch (unchanged)
  • Secrets Manager, KMS (unchanged)
  • Jenkins pipeline (stages 1-5 identical, only deploy stage changes)
  • Dockerfiles (unchanged — same images)
  • Helm charts (already written for both)
```

---

### Migration Execution Steps

```
1. Provision EKS cluster (Terraform module)
   → Control plane, node groups (or Karpenter), OIDC provider

2. Install cluster components
   → AWS Load Balancer Controller (ALB Ingress)
   → ADOT DaemonSet
   → Karpenter (node autoscaling)
   → External Secrets Operator (pulls from Secrets Manager)

3. Deploy services one at a time (not big bang)
   → Start with lowest-risk service
   → Validate in EKS
   → Shift traffic from Fargate → EKS via ALB target group weights
   → Monitor for errors
   → Repeat for next service

4. Decommission Fargate tasks
   → Once all services healthy on EKS
   → Delete ECS services and task definitions
   → Update CI/CD deploy stage: ecs update-service → helm upgrade
```

---

### One-Line Summary

Define measurable criteria (>4 vCPU, need DaemonSets, cold start too slow, cost too high) → when any are hit, migrate service-by-service from ECS Fargate to EKS → images/VPC/databases stay the same → only the orchestration layer and deploy commands change.

---

## Security, Scripting & Enterprise Governance

> **Required Skill:** "Security: zero-trust networking foundations, workload identity, SCP interpretation, org-level policy review"
> **Required Skill:** "Python or Go for infrastructure tooling and operational scripting"
> **Required Skill:** "Experience working within enterprise AWS organizations with centralized governance and approval gates"

---

### 1. Zero-Trust Networking

**Old model (castle-and-moat):** Everything inside the network is trusted. Once you're "in", you can access anything.

**Zero-trust:** Trust nothing, verify everything. Even services INSIDE the same VPC must prove who they are before communicating.

```
Castle-and-moat (bad):
  Attacker gets inside VPC → can reach ALL services freely

Zero-trust (good):
  Attacker gets inside VPC → still can't do anything
  Every service-to-service call must prove:
    1. WHO are you? (identity — mTLS certificate, IRSA token)
    2. Are you ALLOWED to call me? (authorization policy)
    3. Is this connection ENCRYPTED? (TLS required)
  If any answer is "no" → request denied
```

**Implementation in this platform:**
```
Layer 1: Network level
  • Security groups: only allow specific SG → SG communication
  • NetworkPolicies (EKS): default-deny, whitelist only needed paths

Layer 2: Identity level
  • mTLS between services (Istio or app-level)
  • IRSA/Task Roles: each service has unique identity
  • No shared credentials, no shared roles

Layer 3: Authorization level
  • Service A can call Service B on POST /api/v1/analyze ONLY
  • Everything else = denied
```

---

### 2. Workload Identity

Every container/pod gets its own unique identity — like an employee badge. Uses this identity to prove who it is when accessing AWS services or other microservices.

```
Without workload identity (bad):
  All pods share one set of AWS credentials →
  if one pod is compromised, attacker has access to everything

With workload identity (good):
  Pod "orchestrator" → can ONLY call Bedrock + Redis
  Pod "connector-A"  → can ONLY read from S3
  Pod "agent-B"      → can ONLY write to Neptune
  
  If connector-A is compromised → attacker can only read S3, nothing else
```

**Implementation:**
```
On ECS Fargate: Task Role (IAM role attached per task definition)
On EKS: IRSA (ServiceAccount annotated with IAM Role ARN)
Both give temporary credentials that expire. No stored keys.
```

---

### 3. SCP Interpretation

SCPs (Service Control Policies) are guardrails set by the organization (BBY). They are HARD limits — even account admins cannot override them. Your job is to read, understand, and work within these policies.

```
Example SCPs BBY might have:

SCP 1: "Deny all actions in regions other than us-east-1 and us-west-2"
  → Your Terraform must target only allowed regions

SCP 2: "Deny bedrock:InvokeModel unless model is Claude Haiku, Sonnet, or Titan"
  → Your app must only call approved models

SCP 3: "Deny ec2:RunInstances if not tagged with CostCenter"
  → Your Terraform modules must enforce tagging

SCP 4: "Deny iam:CreateUser"
  → No IAM users allowed — must use roles/SSO only

Your job: READ these SCPs → understand what's allowed → design within them
```

---

### 4. Org-Level Policy Review

Before you build anything, review organizational policies:

```
Day 1 tasks:
  1. Get access to AWS Organizations console
  2. Read all SCPs attached to your OU
  3. Understand what's allowed vs blocked
  4. Check if Bedrock model access is enabled in required regions
  5. Verify Transit Gateway sharing permissions
  6. Confirm CI/CD role trust policies align with SCPs
  
  Then design your Terraform modules to comply with ALL of these.
```

---

### 5. Python for Infrastructure Tooling

Python scripts for automation that Terraform alone can't handle:

```
Examples in this platform:
  1. EKS migration readiness checker → checks trigger criteria, reports status
  2. Cost reporting → queries Cost Explorer API → posts to Slack
  3. Secret rotation → rotates Redis password → updates Secrets Manager
  4. Health check / smoke tests → part of Jenkins pipeline
  5. ADOT config generator → reads service list → generates config per service
  6. Tag compliance checker → scans resources → flags missing tags

Why Python (not Bash):
  • boto3 (AWS SDK) — proper API calls with error handling
  • JSON parsing — API responses, config files
  • Complex logic — conditionals, retries, error handling
  • Testable — pytest for infra scripts
```

---

### 6. Enterprise AWS Organizations with Centralized Governance

You work inside a large company's AWS Organization with strict rules, approvals, and shared infrastructure.

```
Enterprise reality (BBY):
  • Can't create accounts yourself → request via ticket
  • Can't choose any region → SCPs restrict to approved regions
  • Can't open ports freely → network team reviews SG changes
  • Can't deploy to prod without approvals → pipeline gates / CAB
  • Must tag everything → cost allocation, ownership, compliance
  • Logs go to central account → security team has access
  • Changes are audited → CloudTrail + AWS Config

YOUR SKILL: Navigate this system efficiently.
Know what to request, when to escalate, how to comply.
```

**Your proof points:**
- Project 4: 15-account Landing Zone with SCPs, OIDC, centralized logging
- SOC2 passed first attempt (proves governance knowledge)
- Pipeline approval gates (proves controlled release understanding)

---

### One-Line Summary

Zero-trust = verify every call even inside the network → Workload identity = each container has its own unique credential (IRSA/Task Role) → SCPs = hard organizational guardrails you must work within → Python for automation beyond Terraform → Enterprise governance = approvals, tagging, auditing, and centralized control at every step.

---
