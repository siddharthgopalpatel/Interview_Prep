# CI/CD Pipeline Differences: Self-managed K8s vs ECS vs EKS

**Context:** Project 1's 18-stage DevSecOps pipeline is container-native. ~80-95% of it applies directly to ECS and EKS. This document covers ONLY the differences.

---

## Deploy Mechanism

| | Project 1 (K8s) | ECS Fargate | EKS |
|---|---|---|---|
| **Deploy command** | `git push` → ArgoCD syncs | `aws ecs update-service --task-definition new-revision` | `helm upgrade --install` or ArgoCD (same as Project 1) |
| **Deployment unit** | Pod (via Deployment/Rollout YAML) | Task (via Task Definition JSON) | Pod (via Helm chart) |
| **Config format** | Kustomize overlays (YAML) | Task Definition (JSON — CPU, memory, image, IAM role, secrets) | Helm values files (YAML) |

---

## Canary / Traffic Shifting

| | Project 1 | ECS | EKS |
|---|---|---|---|
| **Tool** | Argo Rollouts | AWS CodeDeploy (Blue/Green or Linear) OR ALB weighted target groups | Argo Rollouts (same as Project 1) |
| **Granularity** | 5%→20%→50%→80%→100% (custom steps) | Linear10PercentEvery1Minute OR Canary10Percent5Minutes (predefined) | Same custom steps as Project 1 |
| **Metrics gate** | Prometheus AnalysisTemplate | CloudWatch Alarms (CPU, error rate, latency) | Prometheus AnalysisTemplate |

---

## Auto-Rollback

| | Project 1 | ECS | EKS |
|---|---|---|---|
| **Trigger** | Prometheus query fails → Argo Rollouts aborts | CloudWatch Alarm breaches → CodeDeploy rolls back | Same as Project 1 |
| **Mechanism** | Canary weight → 0%, stable serves all | CodeDeploy shifts traffic back to original task set | Same as Project 1 |
| **Manual rollback** | `git revert` → ArgoCD syncs previous | `aws ecs update-service` with previous task def revision | `helm rollback` or `git revert` |

---

## Scaling

| | Project 1 | ECS | EKS |
|---|---|---|---|
| **Tool** | HPA (Horizontal Pod Autoscaler) | ECS Service Auto Scaling (Application Auto Scaling) | HPA + Karpenter (node scaling) |
| **Config** | `HorizontalPodAutoscaler` YAML | Target Tracking Policy (CPU/Memory) via Terraform/Console | HPA YAML (same as Project 1) + Karpenter for nodes |
| **Node scaling** | Cluster Autoscaler (you managed) | Not needed — Fargate is serverless | Karpenter (auto-provisions EC2 nodes) |

---

## Admission Control / Image Trust

| | Project 1 | ECS | EKS |
|---|---|---|---|
| **Tool** | Kyverno (rejects unsigned images) | No admission controller — trust is at pipeline level only | Kyverno (same as Project 1) |
| **Enforcement** | Cluster rejects pod if Cosign signature missing | You rely on pipeline never pushing unsigned + ECR immutable tags | Same as Project 1 |

---

## Service Identity (IAM/Credentials)

| | Project 1 | ECS | EKS |
|---|---|---|---|
| **Mechanism** | Kubeconfig + ServiceAccount (manual) | Task Role (automatic — attached to task definition) | IRSA (ServiceAccount annotated with IAM Role ARN) |
| **Granularity** | Per ServiceAccount | Per Task Definition (per service) | Per Pod (via ServiceAccount) |
| **Setup effort** | You configure OIDC provider + role trust | Just attach role ARN to task def — simplest | Configure OIDC provider + IRSA annotation |

---

## Networking / Ingress

| | Project 1 | ECS | EKS |
|---|---|---|---|
| **Ingress** | Nginx Ingress Controller (self-managed) | ALB (AWS-managed, no controller needed) | AWS Load Balancer Controller (installs in cluster) |
| **TLS** | cert-manager + Let's Encrypt | ACM certificate (free, auto-renewed by AWS) | ACM certificate via ALB Controller |
| **Service discovery** | CoreDNS (K8s internal) | ECS Service Connect OR Cloud Map | CoreDNS (same as Project 1) |

---

## Observability / Sidecar

| | Project 1 | ECS | EKS |
|---|---|---|---|
| **Metrics** | Prometheus (in-cluster) | CloudWatch Container Insights OR Prometheus (self-hosted) | Prometheus (same as Project 1) OR AMP |
| **Tracing sidecar** | Not in Project 1 | ADOT sidecar per task (extra container in task def) | ADOT DaemonSet (one per node, shared) |
| **Sidecar cost** | N/A | 11 services × 1 sidecar each = 11 extra containers | 1 DaemonSet shared by all pods on that node |

---

## GitOps / Drift Detection

| | Project 1 | ECS | EKS |
|---|---|---|---|
| **GitOps tool** | ArgoCD (selfHeal, prune) | No native GitOps — you'd need Terraform + pipeline | ArgoCD (same as Project 1) |
| **Drift detection** | ArgoCD auto-reverts manual `kubectl` changes | No built-in — manual ECS changes persist | ArgoCD auto-reverts (same as Project 1) |

---

## Resource Limits

| | Project 1 | ECS Fargate | EKS |
|---|---|---|---|
| **Max per container** | Node capacity (you control) | **4 vCPU / 30 GB RAM hard limit** | Node capacity (you control) |
| **GPU support** | Yes (with GPU nodes) | **No** | Yes (GPU node groups) |
| **DaemonSets** | Yes | **No** (sidecar per task only) | Yes |
| **Privileged containers** | Yes | **No** | Yes |

---

## One-Liner Summary

The pipeline is the same. The differences are: **how you deploy** (ArgoCD vs `ecs update-service` vs Helm), **how you do canary** (Argo Rollouts vs CodeDeploy), **how services get IAM access** (ServiceAccount vs Task Role vs IRSA), and **Fargate's hard limits** (4 vCPU, no DaemonSets, no GPU).

---

# ECR vs DockerHub

Both are container image registries — they store and serve Docker images. Core function is the same: `docker build → docker tag → docker push → docker pull`

## Key Differences

| Aspect | DockerHub | ECR (Elastic Container Registry) |
|--------|-----------|----------------------------------|
| **Hosted by** | Docker Inc (public internet) | AWS (inside your account, your region) |
| **Access** | Username/password or token | IAM roles/policies (no passwords) |
| **Private repos** | Paid plan for private | Private by default — always |
| **Authentication** | `docker login -u user -p token` | `aws ecr get-login-password \| docker login` (token expires 12 hours) |
| **Pricing** | Free (rate-limited) / paid plans | Pay per GB stored + data transfer |
| **Pull limits** | 100 pulls/6hr (anonymous), 200 (free account) | **No pull limits** within AWS |
| **Network** | Over public internet always | Stays within AWS (VPC endpoint = no internet needed) |
| **Scanning** | Docker Scout (paid) | Built-in scanning (Basic free, Enhanced = Inspector) |
| **IAM integration** | None — separate credentials | Native — ECS/EKS pull with Task Role/IRSA (no credentials) |
| **Immutable tags** | No (`:latest` can be overwritten) | Yes — can enable immutable tags |
| **Lifecycle policies** | Manual cleanup | Auto-delete untagged/old images (policy-based) |
| **Replication** | No | Cross-region and cross-account replication |
| **Encryption** | Docker manages | KMS encryption (you control the key) |

## Commands Comparison

```bash
# DockerHub
docker build -t myuser/myapp:v1.0 .
docker push myuser/myapp:v1.0
docker pull myuser/myapp:v1.0

# ECR (same commands, different registry URL)
docker build -t 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1.0 .
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1.0
docker pull 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1.0
```

## ECR Login

```bash
# DockerHub login (permanent credentials)
docker login -u siddharthpatel -p <token>

# ECR login (temporary — expires in 12 hours)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com
```

In CI/CD, this login happens automatically via IAM role — no stored passwords.

## Why Companies Use ECR Over DockerHub

1. No pull rate limits — DockerHub throttles in CI/CD (100 pulls/6hr). ECR = unlimited within AWS.
2. No credentials to manage — ECS/EKS pull using IAM roles automatically.
3. Stays in your network — VPC endpoint means images never traverse the internet.
4. Compliance — images stay in your account, your region, encrypted with your KMS key.
5. Lifecycle cleanup — auto-delete images older than 30 days or keep only last N tags.

---

# ECR Authentication: How CI/CD Pushes and ECS/EKS Pulls (All via IAM Roles)

## Push (CI/CD → ECR)

| CI/CD Tool | How It Authenticates to ECR |
|---|---|
| **Jenkins (on EC2)** | EC2 Instance Role — has `ecr:PutImage` permission |
| **GitHub Actions** | OIDC → assumes IAM Role — has `ecr:PutImage` permission |
| **ECS/Fargate task running CI** | Task Role — has `ecr:PutImage` permission |

```
Jenkins/GitHub Actions → assumes IAM Role → calls `aws ecr get-login-password` → pushes image
                         (no passwords stored)
```

## Pull (ECS/EKS ← ECR)

| Platform | How It Authenticates to ECR |
|---|---|
| **ECS Fargate** | Task Execution Role (auto-pulls image — attached to task def) |
| **EKS** | Node IAM Role OR IRSA — has `ecr:GetDownloadUrlForLayer`, `ecr:BatchGetImage` |

```
ECS Task starts → Task Execution Role → automatically pulls image from ECR
                  (no docker login needed at runtime)
```

## The Two ECS Roles (Important Distinction)

| Role | Purpose | Used For |
|---|---|---|
| **Task Execution Role** | Lets ECS **pull image** + fetch secrets at startup | ECR pull, Secrets Manager read, CloudWatch logs |
| **Task Role** | Lets your **application code** access AWS services at runtime | S3, DynamoDB, Bedrock, SQS — whatever your app needs |

```
ECS Task starts:
  1. Task Execution Role → pulls image from ECR ✅
  2. Task Execution Role → fetches secrets from Secrets Manager ✅
  3. Container running → Task Role → app calls S3, Bedrock, etc. ✅
```

## Summary

```
CI/CD (push):     Jenkins/GitHub → IAM Role → ecr:PutImage → pushes to ECR
ECS (pull):       Task Execution Role → ecr:BatchGetImage → pulls from ECR
EKS (pull):       Node Role or IRSA → ecr:BatchGetImage → pulls from ECR
App (runtime):    Task Role (ECS) or IRSA (EKS) → calls S3, Bedrock, etc.
```

**Zero passwords, zero docker login credentials stored anywhere. Everything is IAM roles with temporary credentials.**

---

# ECR Complete Settings (Beyond Push & Pull)

## What We Configure at Creation

| Setting | Value | Why |
|---------|-------|-----|
| Repository name | `my-demo-app` | Identifier |
| Tag immutability | IMMUTABLE | Prevents overwriting tags (security) |
| Scan on push | Enabled | Auto CVE scan on every push |
| Encryption | AES-256 (default) | Free, AWS-managed |

---

## Production Settings (After Creation)

### 1. Lifecycle Policy (Auto-cleanup old images)

Without this, images pile up forever and you pay $0.10/GB/month.

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep only last 10 tagged images",
      "selection": {
        "tagStatus": "tagged",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": { "type": "expire" }
    },
    {
      "rulePriority": 2,
      "description": "Delete untagged images after 1 day",
      "selection": {
        "tagStatus": "untagged",
        "countType": "sinceImagePushed",
        "countUnit": "days",
        "countNumber": 1
      },
      "action": { "type": "expire" }
    }
  ]
}
```

### 2. Repository Policy (Cross-account access)

By default, only your account can access. If another AWS account needs to pull:

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::999888777666:root"
      },
      "Action": [
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ]
    }
  ]
}
```

**Use case:** Prod account pulls images built in Shared Services account.

### 3. Replication (Cross-region / Cross-account)

Auto-copies images to another region or account when pushed.
- DR: image available in `us-west-2` if `us-east-1` goes down
- Multi-region: ECS in both regions pulls locally (faster)

### 4. Pull-through Cache

ECR caches public images (DockerHub, GitHub) so you don't hit pull rate limits.

```bash
# Instead of DockerHub (rate limited):
docker pull nginx:latest

# Pull through ECR cache (no limits):
docker pull <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/docker-hub/nginx:latest
```

### 5. Enhanced Scanning (Inspector)

| | Basic Scan (free) | Enhanced Scan (paid) |
|---|---|---|
| **What** | OS package CVEs only | OS + app language libs (pip, npm, maven) |
| **When** | On push only | Continuous (re-scans when new CVEs published) |
| **Cost** | Free | ~$0.09 per image scan |

---

## All ECR Settings Summary

| Setting | Required? | When |
|---------|-----------|------|
| Repo name | ✅ Yes | Create time |
| Tag immutability | ✅ Recommended | Create time |
| Scan on push | ✅ Recommended | Create time |
| Encryption (AES/KMS) | Optional | Create time |
| Lifecycle policy | ✅ Production must-have | After creation |
| Repository policy | Only if cross-account | After creation |
| Replication | Only if multi-region/DR | Registry level |
| Pull-through cache | Only if using public images | Registry level |
| Enhanced scanning | Only if compliance requires | Registry level |

**Interview tip:** Mention Lifecycle Policy and Repository Policy — those show you understand real-world ECR management beyond just "push and pull."
