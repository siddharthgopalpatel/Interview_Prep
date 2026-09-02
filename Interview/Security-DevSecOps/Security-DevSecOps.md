# Security DevSecOps — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 3, 8, 41, 46

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 3: Security & Dependencies

---

### Q: How will a developer know that something downloaded from a public repository doesn't have issues?

**Project Reference:** P1 (DevSecOps Pipeline — Stage 5: SCA, Stage 10: Container Scan)

**Answer:**

> "They won't know manually — that's why we automate it in the pipeline. We have two layers:
>
> 1. **SCA (Software Composition Analysis)** — Stage 5 in our pipeline. Tools like Snyk or `pip audit` / `npm audit` scan every dependency in `requirements.txt` or `package.json` against known CVE databases. If a library has a critical vulnerability, pipeline fails. Developer gets told: 'Library X version Y has CVE-2024-XXXX — upgrade to version Z.'
>
> 2. **Container image scanning** — Stage 10. Trivy scans the base image (e.g., `python:3.9-slim`) for OS-level vulnerabilities. Developers don't manually check if Debian's openssl is patched — Trivy does.
>
> Additionally, we use a **private Nexus/Artifactory mirror** — public packages are proxied and cached. We can block known-malicious packages at the proxy level before they even reach developers.
>
> The point: security is shifted left. Developer doesn't need to 'know' — the pipeline catches it automatically."

---
---

# SECTION 8: Security & DevSecOps

---

### Q: How do you ensure DevSecOps in a service and what are the different components of it?

**Project Reference:** P1 (DevSecOps Pipeline — 6 security layers)

**Answer:**

> "DevSecOps means security is embedded at every stage, not bolted on at the end. Our 6 layers:
>
> | Layer | Stage | Tool | What It Catches |
> |---|---|---|---|
> | 1 | Secret scanning | trufflehog / git-secrets | Hardcoded passwords, API keys in code |
> | 2 | SCA | Snyk / pip audit | Vulnerable third-party libraries |
> | 3 | SAST | SonarQube | Code-level bugs, SQL injection patterns, code smells |
> | 4 | Container scan | Trivy | OS/library CVEs in Docker images |
> | 5 | Image signing | Cosign | Supply chain — only trusted images deploy |
> | 6 | DAST | OWASP ZAP | Runtime vulnerabilities (XSS, injection on running app) |
>
> **Beyond the pipeline:**
> - Kubernetes: Pod Security Standards (restricted), Kyverno policies, NetworkPolicies (default-deny)
> - Infrastructure: tfsec + Checkov + OPA in Terraform CI (no open security groups, no unencrypted storage)
> - Runtime: Falco for anomaly detection, Istio mTLS for zero-trust networking
>
> Security is not a gate at the end — it's embedded in every commit, every build, every deployment."

---

### Q: What are the key things you will do to secure your application?

**Project Reference:** P1, P3, P6 (security across stack)

**Answer:**

> "Defense in depth — secure at every layer:
>
> 1. **Network** — VPC with private subnets, NACLs, Security Groups (least-privilege), WAF for L7 protection, NetworkPolicies in K8s (default-deny)
> 2. **Identity** — IRSA for pods (no static credentials), OIDC for CI/CD, IAM least-privilege, short-lived tokens only
> 3. **Data** — Encryption at rest (KMS), encryption in transit (TLS/mTLS everywhere), secrets in Vault/K8s secrets (not env vars or code)
> 4. **Application** — SAST/DAST in pipeline, input validation, dependency scanning, no root containers, read-only filesystem
> 5. **Supply chain** — Cosign image signing, Kyverno admission control, trusted registry only
> 6. **Monitoring** — Falco for runtime anomalies, GuardDuty for account-level threats, CloudTrail for audit
>
> The principle: assume breach. Minimize blast radius. Detect fast."

---

### Q: What do you understand by SSL termination?

**Project Reference:** P2 (3-Tier AWS — ALB), P6 (Istio — mTLS)

**Answer:**

> "SSL termination is where the TLS/HTTPS encryption is decrypted. The component that terminates SSL handles the certificate, does the CPU-heavy crypto work, and forwards plain HTTP (or re-encrypts) to the backend.
>
> **Where we terminate:**
> - **ALB** — Most common. ALB holds the ACM certificate, terminates TLS, forwards HTTP to targets. Backend doesn't deal with certificates.
> - **CloudFront** — Edge termination for CDN. Client → HTTPS → CloudFront (terminate) → HTTP → ALB → Backend.
> - **Istio Ingress Gateway** — Terminates external TLS, then re-encrypts with mTLS for internal mesh traffic.
>
> **Why it matters:** Offloads crypto from app servers (saves CPU), centralizes certificate management, and allows inspection of traffic (WAF can't inspect encrypted traffic).
>
> **Security consideration:** Traffic between ALB and backend is unencrypted unless you use end-to-end TLS. In our setup, Istio re-encrypts internally with mTLS — so traffic is encrypted at every hop."

---

### Q: Explain the concept of immutable infrastructure?

**Project Reference:** P2 (AMI baking with Packer), P1 (Docker images)

**Answer:**

> "**Immutable = never modify in place. Replace entirely.**
>
> Instead of: SSH into a server → install patch → restart service (mutable, risky, drift-prone)
> We do: Build a new AMI/container image with the patch → deploy new instances → terminate old ones.
>
> **How we implement it:**
> - **Containers (K8s):** New code = new Docker image = new pods rolled out. We never `kubectl exec` and change things inside a running container.
> - **EC2 (ASG):** Packer bakes a new AMI with updated code/patches → ASG instance refresh replaces instances one by one.
>
> **Benefits:**
> - No configuration drift — every instance is identical
> - Rollback = just deploy the previous image/AMI
> - No 'works on that server but not this one' problems
> - Reproducible — same image in Dev, Staging, Prod
>
> **Our exception:** P9 (OS patching) is mutable by nature — you can't rebuild 500+ bare-metal-like servers every month. But for cloud-native workloads, immutable is the standard."

---
---

# SECTION 41: Security Testing

---

### Q: What are the key differences between SAST and DAST?

**Project Reference:** P1 (DevSecOps Pipeline — Stage 7: SAST, Stage 17: DAST)

**Answer:**

> | Aspect | SAST (Static) | DAST (Dynamic) |
> |---|---|---|
> | **When** | During build (before deployment) | After deployment (running application) |
> | **What it scans** | Source code / binary | Running application via HTTP requests |
> | **How** | Analyzes code patterns without executing | Attacks the app like a hacker would |
> | **Finds** | SQL injection patterns, hardcoded secrets, insecure deserialization, buffer overflows | XSS, authentication flaws, misconfigurations, exposed endpoints |
> | **False positives** | Higher (code may never execute that path) | Lower (it's actually exploitable) |
> | **Speed** | Fast (no deployment needed) | Slow (needs running app, crawls endpoints) |
> | **Tool we use** | SonarQube | OWASP ZAP |
> | **Pipeline stage** | Stage 7 (early — fast feedback) | Stage 17 (late — needs deployed app) |
>
> **Why we use BOTH:**
> - SAST catches problems EARLY (before deployment) — cheaper to fix
> - DAST catches problems SAST misses (runtime configs, auth flaws, server misconfigs)
> - Together = defense in depth. Code-level + runtime-level security
>
> **Key insight:** SAST finds 'this code COULD be vulnerable.' DAST proves 'this application IS vulnerable.' Both are needed for comprehensive security coverage."

---
---

# SECTION 46: DevSecOps Workflow & Architecture Concepts

---

### Q: Did you evaluate Checkmarx vs other products? Any specific benefits?

**Project Reference:** P1 (DevSecOps — we use SonarQube + Trivy + Snyk)

**Answer:**

> "We evaluated Checkmarx but chose SonarQube (SAST) + Trivy (container scanning) + Snyk (SCA) instead.
>
> **Checkmarx strengths:**
> - Enterprise-grade SAST — very deep code analysis, supports 25+ languages
> - Low false-positive rate (better than SonarQube for pure security findings)
> - Compliance reporting built-in (SOC2, PCI-DSS mapping)
> - SCA and container scanning in one platform (Checkmarx One)
>
> **Why we chose our stack over Checkmarx:**
> 1. **Cost** — Checkmarx is expensive ($50K+/year). SonarQube Community + Trivy (open source) + Snyk (free tier for SCA) = significantly cheaper for our scale.
> 2. **Pipeline integration** — SonarQube and Trivy integrate natively with Jenkins via CLI. Checkmarx needs plugin + server setup.
> 3. **SonarQube gives BOTH** — code quality AND security. Checkmarx is security-only (still need a separate quality tool).
> 4. **Team familiarity** — Team already knew SonarQube. Switching introduces learning curve.
>
> **When I'd choose Checkmarx:** If the organization has strict compliance requirements (PCI-DSS), needs enterprise support, and budget allows. It's technically superior for SAST — but for our DevSecOps pipeline, the open-source stack gives us 90% of the value at 10% of the cost."

---

### Q: What happens when you receive a security issue in your pipeline? What is the workflow?

**Project Reference:** P1 (DevSecOps Pipeline — security gate workflow)

**Answer:**

> "Depends on severity:
>
> **Critical/High (pipeline FAILS — blocks deployment):**
> 1. Pipeline stops at the security stage (e.g., Trivy finds Critical CVE, SonarQube finds SQL injection)
> 2. Developer gets Slack notification: 'Build failed — 1 Critical vulnerability in `lodash@4.17.20`'
> 3. Developer fixes (upgrade library, patch code) → push → pipeline re-runs
> 4. If can't fix immediately (no patch available): raise a **security exception request** — security team reviews, documents the risk, sets a remediation SLA (7 days for Critical)
> 5. Exception approved → temporary bypass with ticket tracking. Exception expires → pipeline blocks again.
>
> **Medium/Low (pipeline passes — creates ticket):**
> 1. Pipeline continues (doesn't block deployment)
> 2. Automatically creates a Jira ticket: 'Medium: XSS pattern found in input handler'
> 3. SLA: Medium = 30 days, Low = 90 days
> 4. Security team reviews in weekly triage
>
> **Key principles:**
> - Critical/High = NEVER reaches production without fix or documented exception
> - We don't block deployments for Low severity (creates developer fatigue)
> - Every exception has an expiry date — no permanent bypasses
> - Security findings are tracked as technical debt with SLAs"

---

### Q: Do you understand the AWS Well-Architected Framework?

**Project Reference:** All projects — follows WAF pillars

**Answer:**

> "Yes. It's AWS's guidance for building secure, resilient, efficient, cost-effective, and sustainable workloads. **6 pillars:**
>
> | Pillar | What It Addresses | Our Implementation |
> |---|---|---|
> | **Operational Excellence** | Automation, monitoring, incident response | CI/CD pipelines (P1), runbooks, automated DR drills (P8) |
> | **Security** | Protection of data, systems, assets | DevSecOps (P1), IRSA, SCPs, mTLS (P6), encryption everywhere |
> | **Reliability** | Recover from failures, meet demand | Multi-AZ (P2), Multi-Region DR (P8), auto-scaling, self-healing |
> | **Performance Efficiency** | Use resources efficiently | Right-sizing, Karpenter (P7), caching, Aurora read replicas |
> | **Cost Optimization** | Avoid unnecessary costs | FinOps automation (P7), Spot, auto-stop, Savings Plans |
> | **Sustainability** | Minimize environmental impact | Right-sizing reduces compute waste, Graviton (ARM) instances |
>
> **How I use it:** When designing any new architecture, I mentally walk through each pillar. 'Is this reliable? Is it secure? Am I over-provisioning?' It's a checklist for completeness — ensures you don't optimize for one dimension at the expense of others."

---

### Q: Have you designed any system solution?

**Project Reference:** All 9 projects — end-to-end design

**Answer:**

> "Yes, I've designed several end-to-end:
>
> 1. **DevSecOps Pipeline (P1)** — Designed the entire 18-stage pipeline architecture from scratch. Chose tools, defined security gates, designed canary strategy, built the CI platform (Docker Compose: Jenkins + SonarQube + Agent).
>
> 2. **Multi-Account Landing Zone (P4)** — Designed the OU structure, SCP policies, OIDC federation, networking topology (Transit Gateway hub-spoke). From requirements gathering with security team to Terraform implementation.
>
> 3. **OS Patching Automation (P9)** — Designed the entire 18-step lifecycle, role decomposition, validation dimensions, ITIL integration, and rollback strategy.
>
> **My design approach:**
> - Start with requirements (SLAs, constraints, compliance)
> - Identify components and their interactions (draw architecture diagram)
> - Consider failure modes (what breaks, blast radius, rollback)
> - Document trade-offs and decisions (ADRs — Architecture Decision Records)
> - Build incrementally (MVP → iterate based on feedback)
>
> I'm not just an implementer — I design the solution, get stakeholder alignment, and then build it."

---

### Q: What was the approach for migration and your role?

**Project Reference:** P3 (VM → Kubernetes migration), General

**Answer:**

> "We migrated workloads from VM-based deployments to containerized Kubernetes (EKS).
>
> **My role:** Technical lead for the migration strategy and execution.
>
> **Approach (phased):**
>
> 1. **Assessment** — Identified 20+ services running on EC2. Categorized: easy to containerize (stateless APIs), medium (stateful with volumes), hard (legacy monoliths needing refactoring).
>
> 2. **Start with easy wins** — Containerized stateless microservices first. Wrote Dockerfiles, Helm charts. Deployed to EKS Dev alongside existing EC2 instances.
>
> 3. **Parallel running** — Both VM and K8s versions running. Traffic split via ALB weighted routing (90% VM, 10% K8s). Validated performance, error rates.
>
> 4. **Cutover** — Once K8s version stable for 2 weeks → shift 100% traffic → decommission EC2 instances.
>
> 5. **Remaining services** — Medium complexity: added PVCs for stateful data. Hard: refactored into microservices over time (not all migrated — some still on EC2 with Ansible management).
>
> **Key decisions I made:**
> - Lift-and-shift first (same code, just containerized) — not refactor + migrate simultaneously
> - Keep database on RDS (not in K8s) — managed services for stateful components
> - Build observability BEFORE migration — need to compare metrics VM vs K8s"

---

---
---

