# Deployment Strategies — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 17, 52

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 17: Deployment Strategies

---

### Q: What deployment strategy are you using?

**Project Reference:** P1 (DevSecOps Pipeline — Canary with Argo Rollouts)

**Answer:**

> "**Canary deployment** for production, **Rolling update** for lower environments.
>
> | Environment | Strategy | Why |
> |---|---|---|
> | Dev | Rolling update | Fast iteration, don't need safety gates |
> | Staging | Rolling update | Full test suite validates after deploy |
> | Production | Canary (5% → 25% → 50% → 100%) | Minimize blast radius, Prometheus-driven auto-rollback |
>
> **How our canary works (Argo Rollouts):**
> 1. New version gets 5% of traffic
> 2. Prometheus checks error rate and latency for 5 minutes
> 3. Metrics healthy → promote to 25% → wait → 50% → 100%
> 4. If error rate > 1% or P99 latency > 500ms at any step → automatic rollback to previous version in 2 minutes
>
> **Result:** Deployment blast radius reduced from 100% to 5%. If something's wrong, only 5% of users are briefly affected before auto-rollback kicks in."

---

### Q: Can you explain how the Blue-Green deployment strategy works?

**Project Reference:** P1 (aware of strategy, we chose Canary instead)

**Answer:**

> "**Blue-Green = two identical environments. Switch traffic atomically.**
>
> - **Blue** = current production (running)
> - **Green** = new version (deployed but not receiving traffic)
>
> **Process:**
> 1. Deploy new version to Green environment
> 2. Run smoke tests against Green (internal traffic only)
> 3. Tests pass → switch load balancer/DNS to point to Green
> 4. Green becomes the new production. Blue becomes standby.
> 5. If something goes wrong → switch back to Blue instantly (still running)
>
> **Pros:**
> - Instant rollback (just switch back to Blue)
> - Zero downtime (traffic switches atomically)
> - Full production environment for testing before going live
>
> **Cons:**
> - **Double the infrastructure cost** — two full environments running during deployment
> - **Database schema changes are tricky** — both Blue and Green must be compatible with the DB
> - All-or-nothing switch — no gradual traffic shift (unlike Canary)
>
> **Why we chose Canary over Blue-Green:** Canary gives us gradual validation with metrics-driven promotion. Blue-Green is all-or-nothing. At 15+ microservices, maintaining two full environments is expensive. Canary gives us safety with lower cost."

---

### Q: How do you manage secrets in your deployment (Kubernetes)?

**Project Reference:** P3 (Kubernetes), P1 (DevSecOps)

**Note:** Pipeline secrets covered in existing Q. This answer focuses on DEPLOYMENT/runtime secrets.

**Answer:**

> "For Kubernetes runtime secrets specifically:
>
> 1. **Kubernetes Secrets** — Created via Helm chart or sealed-secrets. Mounted as env vars or volume files into pods.
>
> 2. **Encryption at rest** — EKS encrypts secrets in etcd using AWS KMS envelope encryption. Not just base64.
>
> 3. **External Secrets Operator (ESO)** — Syncs secrets FROM AWS Secrets Manager INTO Kubernetes Secrets automatically. Source of truth is Secrets Manager. ESO creates/updates K8s secrets when the external value changes.
>
> 4. **IRSA for access** — Pods that need AWS resources don't use secrets at all — they use IAM Roles for Service Accounts. No credentials to manage.
>
> **What we avoid:**
> - Hardcoded secrets in Helm values files (even in private repos)
> - Plain K8s secrets without encryption at rest
> - ConfigMaps for sensitive data (ConfigMaps are not encrypted)"

---

### Q: How do you sync secrets with the deployment in different environments (Dev/UAT/Prod)?

**Project Reference:** P1 (Multi-environment), P3 (Kubernetes)

**Answer:**

> "Each environment has its own secrets — they're NOT copied between environments. Here's how:
>
> **Using External Secrets Operator (ESO):**
>
> ```yaml
> # Same ExternalSecret manifest per env, different SecretStore pointing to different AWS account
> apiVersion: external-secrets.io/v1beta1
> kind: ExternalSecret
> spec:
>   secretStoreRef:
>     name: aws-secrets-manager  # Different store per env
>   target:
>     name: db-credentials
>   data:
>     - secretKey: password
>       remoteRef:
>         key: /dev/db/password    # /staging/db/password, /prod/db/password
> ```
>
> **Strategy:**
> - **Dev:** Secrets Manager in dev account → ESO syncs to dev cluster
> - **Staging:** Secrets Manager in staging account → ESO syncs to staging cluster
> - **Prod:** Secrets Manager in prod account → ESO syncs to prod cluster
>
> **Key principles:**
> - Prod secrets NEVER exist in dev/staging (separate AWS accounts, separate Secrets Manager)
> - Secret rotation in Secrets Manager → ESO auto-updates K8s secret → pods pick up new value (with volume mount, not env var — env vars need pod restart)
> - Developers can manage dev secrets. Only platform team manages prod secrets."

---
---

# ~~SECTION 18: Terraform Commands & Scenarios~~ → Moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)

---
---

# SECTION 52: Architecture, Security Response & Terraform Performance

---

### Q: Can you give me an example of event-driven architecture used in AWS?

**Project Reference:** P5 (Serverless Event-Driven — Security Remediation)

**Answer:**

> "Our **security auto-remediation engine (P5)** is a textbook event-driven architecture:
>
> ```
> AWS Config (detects violation)
>     → EventBridge (routes event by type)
>         → SQS (buffer + retry)
>             → Lambda (remediates)
>                 → DynamoDB (audit log)
>                 → SNS (notification)
> ```
>
> **How it's event-driven:**
> - No polling. No cron. No always-running service.
> - An EVENT (security violation) triggers the chain. If no violations → nothing runs → $0 cost.
> - Each component reacts to an event from the previous component.
> - Loose coupling: Lambda doesn't know/care what produced the event. It just processes whatever lands in SQS.
>
> **Benefits:**
> - **Scale to zero:** No violations = no compute running = $0.07/month total cost
> - **Auto-scales:** 100 violations at once → 100 Lambda invocations concurrently
> - **Resilient:** SQS provides retry + DLQ. If Lambda fails, message retries 3x then goes to dead-letter queue for manual review
>
> **Other event-driven patterns we use:**
> - S3 upload → Lambda (image resize, virus scan)
> - CloudTrail event → EventBridge → Lambda (detect root login → alert)
> - EKS pod failure → Prometheus alert → AlertManager → Slack/PagerDuty"

---

### Q: What happens if you find out that certain credentials have been leaked in AWS? What do you do?

**Project Reference:** P4 (Landing Zone — security), P1 (DevSecOps)

**Answer:**

> "**Immediate response (within minutes):**
>
> 1. **Revoke immediately** — Disable the leaked access key: `aws iam update-access-key --access-key-id AKIA... --status Inactive`. If it's a password → force password reset. If it's a service account → rotate credentials.
>
> 2. **Assess blast radius** — CloudTrail: What did this credential do? `aws cloudtrail lookup-events --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKIA...`. Look for: resource creation, data access, IAM changes, unusual regions.
>
> 3. **Contain damage** — If unauthorized actions found: revert changes, terminate unknown instances, remove unknown IAM users/roles, block suspicious IPs via WAF/NACL.
>
> 4. **Identify source of leak** — Where was it leaked? GitHub public repo? Slack message? Docker image layer? Laptop compromised?
>
> 5. **Rotate ALL potentially affected credentials** — Not just the leaked one. If an IAM user was compromised, rotate everything that user had access to.
>
> **Prevention measures (already in place):**
> - Pipeline Stage 1: secret scanning (trufflehog/git-secrets) — catches before it hits Git
> - AWS GuardDuty: detects credential usage from unusual locations/IPs
> - GitHub secret scanning: auto-alerts if AWS keys appear in any push
> - No long-lived credentials: we use IRSA + OIDC — nothing to leak
>
> **Post-incident:** Blameless postmortem. Action item: how did this credential exist as a static key in the first place? Migrate to short-lived credentials (OIDC/IRSA)."

---

### Q: Can you explain three vulnerabilities you found through SonarQube/Trivy scanning?

**Project Reference:** P1 (DevSecOps Pipeline — real findings)

**Answer:**

> "Three real findings from our pipeline:
>
> **1. SQL Injection pattern (SonarQube — SAST):**
> - Finding: String concatenation in database query: `query = \"SELECT * FROM users WHERE id=\" + user_input`
> - Risk: Attacker can inject `1; DROP TABLE users--`
> - Fix: Parameterized query: `cursor.execute(\"SELECT * FROM users WHERE id=%s\", (user_input,))`
> - Pipeline: BLOCKED deployment until fixed.
>
> **2. Critical CVE in base image (Trivy — Container Scan):**
> - Finding: `python:3.9-slim` base image had CVE-2024-3094 (OpenSSL vulnerability — remote code execution)
> - Risk: Attacker could exploit OpenSSL flaw to execute code inside container
> - Fix: Rebuild image with updated base: `python:3.9-slim` newer digest with patched OpenSSL
> - Pipeline: BLOCKED — Critical CVE = automatic fail.
>
> **3. Hardcoded secret (trufflehog — Secret Scanning):**
> - Finding: AWS access key (`AKIA...`) committed in a config file by a developer
> - Risk: Anyone with repo access could use the key to access AWS resources
> - Fix: Removed from code, rotated the key, moved to Jenkins Credentials Store, added to `.gitignore`
> - Pipeline: BLOCKED at Stage 1 (earliest possible detection).
>
> **Key point:** These aren't hypothetical — they're real things our pipeline catches weekly. Without automated scanning, they'd reach production."

---

### Q: Our company uses GitHub Actions. Are you comfortable with that?

**Project Reference:** P1 (CI/CD — Jenkins primary, GitHub Actions familiar)

**Answer:**

> "Yes, comfortable. I've used GitHub Actions for smaller projects and understand the model well:
>
> - **Workflow files** (`.github/workflows/`) — YAML-based, triggered by events (push, PR, schedule)
> - **Jobs and steps** — parallel jobs, sequential steps, matrix builds
> - **Actions marketplace** — reusable actions for common tasks (checkout, setup-node, docker build)
> - **Secrets** — repository/organization secrets, environment-scoped secrets
> - **Environments** — with protection rules (approvals, wait timers) for production
> - **Self-hosted runners** — for private network access or custom tooling
>
> **My advantage:** CI/CD concepts are the same regardless of tool. Pipeline stages, security gates, artifact management, environment promotion, GitOps — all translate directly. The difference is syntax (Groovy → YAML) and platform features.
>
> **What I'd bring:** My experience with pipeline design (18-stage security pipeline, canary deployments, multi-environment promotion) applies 1:1 to GitHub Actions. I'd structure workflows the same way — just different YAML syntax.
>
> Transition from Jenkins to GitHub Actions would take me 1-2 weeks to be fully productive."

---

*Note: "Terraform plan/apply takes 30 minutes" and "Team X creates EC2 instances, Team B also creates EC2" have been moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)*

---

---
---

