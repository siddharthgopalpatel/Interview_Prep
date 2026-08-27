# General Interview Q&A Bank

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)
- **DevOps Phase:** CI/CD pipelines, 3-tier architecture (VMs + Containers), OS Patch Automation, Serverless, Kubernetes
- **DevSecOps Phase:** DevSecOps pipeline, Multi-Account Landing Zone, Cost Optimization
- **Awards/Certs:** Delivery Champion (Kubernetes), CKA, CKAD, AWS SAA, SAFe DevOps

---
---

# SECTION 1: Introduction & Experience

---

## Question Types — How to Identify

| Signal in Question | Type | Duration |
|---|---|---|
| "brief" / "start by" / "introduce yourself" | Type 1: Quick Intro | 30-60s |
| "skills" / "day-to-day" / "responsibilities" | Type 2: Skills + Daily Work | 90s |
| "projects" / "professional background" / "what you've done" | Type 3: Career Arc + Projects | 2 min |

**Pro tip:** If unsure which type → start with Type 1 (short). They'll ask follow-ups if they want more. Never open with a 3-minute monologue unless explicitly asked.

---

### Q: Can you provide a brief introduction about yourself? / Could you start by introducing yourself?

**Type:** Quick Intro (30-60 seconds)

**Answer:**

> "I'm Siddharth Patel, a DevSecOps Engineer at Ericsson with about 10 years of experience. I started as a Linux Admin, spent 7 years as a DevOps Engineer building CI/CD pipelines, 3-tier architectures, container platforms, and OS patching automation — earned CKA, CKAD, and AWS Solutions Architect along the way. Currently I own our DevSecOps pipeline, multi-account AWS Landing Zone, and cloud cost optimization — the security and governance layer for the platform."

---

### Q: Can you brief me about yourself, your skills, experience, and day-to-day responsibilities?

**Type:** Skills + Daily Work (90 seconds)

**Answer:**

> "Siddharth Patel, 10 years at Ericsson. Started as a Linux Admin managing RHEL servers — patching, troubleshooting, monitoring. Then 7 years as a DevOps Engineer where I handled CI/CD pipelines with Jenkins and ArgoCD, built 3-tier AWS architectures for VMs and containers, automated OS patching for 500+ servers with zero downtime, and managed serverless workloads. During that phase I got certified — CKA, CKAD, AWS Solutions Architect, SAFe DevOps — and was recognized as Delivery Champion for Kubernetes.
>
> Currently I'm a DevSecOps Engineer. Day-to-day I own three things: our 18-stage DevSecOps pipeline with 6 security gates, a 15-account multi-account AWS Landing Zone with SCPs and centralized compliance, and cloud cost optimization — which has saved $180K a year so far.
>
> **Tech stack:** AWS (EKS, Lambda, Organizations, Security Hub), Terraform, Ansible, Kubernetes, Docker, Jenkins, ArgoCD, Prometheus/Grafana, Istio.
>
> The Linux foundation still helps daily — when a container is OOMKilled or a network policy isn't working, I debug at the kernel level, not just the YAML level."

---

### Q: Please introduce yourself and explain your projects and experience. / Can you provide an introduction of your professional background?

**Type:** Career Arc + Projects (2 minutes)

**Answer:**

> "Siddharth Patel, DevSecOps Engineer at Ericsson with 10 years of experience. I've grown through three distinct phases here.
>
> **Phase 1 — Linux Admin (2015–2018, Bangalore):** Managed 100+ RHEL servers. Patching, monitoring, troubleshooting, capacity planning. This is where I built the deep OS-level understanding. But I kept automating repetitive work with Bash and Ansible — reduced manual SSH by 80% — and that pulled me into DevOps.
>
> **Phase 2 — DevOps Engineer (2018–2025):** This is where I built most of the platform from scratch. CI/CD pipelines with Jenkins and ArgoCD. 3-tier AWS architectures handling VMs and containers — 5,000 req/s with auto-scaling. OS patching automation across 500+ servers with zero downtime using Ansible Automation Platform. Serverless event-driven workloads on Lambda. Kubernetes across EKS, OpenShift, and self-managed clusters running 15+ microservices. During this time I earned CKA, CKAD, AWS Solutions Architect, SAFe DevOps, and was awarded Delivery Champion for Kubernetes.
>
> **Phase 3 — DevSecOps Engineer (2025–Present):** Now I own three pillars. First, the DevSecOps pipeline — 18 stages with 6 security layers from secret scanning through DAST, canary deployments with auto-rollback, blast radius reduced from 100% to 5%. Second, multi-account AWS Landing Zone — 15 accounts, 5 OUs, SCPs, centralized compliance with GuardDuty, Security Hub, Config — SOC2 passed first attempt. Third, cost optimization — FinOps automation that saved $180K a year through auto-stop, Karpenter consolidation, rightsizing, and Savings Plans.
>
> Common thread across all three phases: I take manual, risky processes and make them automated, secure, and self-healing. That's what I want to keep doing at scale."

---

### Q: Have you worked directly with clients to solve problems or implement solutions?

**What they're really asking:** Are you just a backend executor, or do you interface with stakeholders?

**Answer:**

> "Yes, extensively. At Ericsson, I worked directly with Verizon's operations and engineering teams for 10 years. It wasn't just 'take a ticket, execute.' I'd join bridge calls during production incidents, understand their SLA impact — for a contact center, even 30 seconds of downtime means dropped calls and revenue loss. I'd propose solutions, get their sign-off, and implement.
>
> For example, when we built the OS patching automation — the requirement came directly from the client's change management team. They needed zero-downtime patching with ITIL compliance. I worked with their ServiceNow admins to design the CR workflow, with their app teams to define health check criteria, and then built the 18-step automation. It wasn't thrown over a wall — it was collaborative from requirements to production."

**Project Reference:** P9 (OS Patching), P8 (DR — quarterly DR drills with client stakeholders)

---

### Q: Do you understand that different customers have different requirements?

**What they're really asking:** Can you adapt or do you force one-size-fits-all?

**Answer:**

> "Absolutely. Even within Verizon, different teams had different priorities. The voice platform team cared about latency and MOS scores — they'd reject any change that added even 10ms. The compliance team cared about audit trails and CIS benchmarks. The cost team wanted savings without touching SLAs.
>
> My approach: I build modular, configurable solutions — not rigid ones. Our Terraform modules have environment-specific variable files. Our CI/CD pipeline has configurable security gates — a fintech customer might need all 6 layers mandatory, while an internal tool might skip DAST. The Ansible patching roles take parameters — one customer wants serial 20% rolling updates, another needs maintenance windows with full-fleet parallel.
>
> The architecture stays consistent, but the policy is customer-driven."

**Project Reference:** P1 (configurable pipeline gates), P9 (parameterized patching roles), P4 (Landing Zone — different OUs for different team requirements)

---

### Q: What is your strong zone, or what do you work on regularly?

**What they're really asking:** Where do you add the MOST value?

**Answer:**

> "My strong zone is **infrastructure automation and platform reliability** — specifically three areas:
>
> 1. **Terraform-driven IaC** — Designing reusable module libraries, multi-account/multi-region deployments, with security scanning baked in. I do this daily.
>
> 2. **Kubernetes platform operations** — EKS, self-managed clusters, Helm deployments, troubleshooting pod/node issues, autoscaling. This is my bread and butter.
>
> 3. **CI/CD pipeline engineering** — Building secure, reliable delivery pipelines. Not just 'code to deploy' but the full chain — security gates, canary rollouts, automated rollback.
>
> The common thread: I make infrastructure self-healing and deployment safe. My Linux/System Admin background means I can troubleshoot at any layer — from kernel to application."

**Project Reference:** All 9 projects, but strongest in P1 (CI/CD), P3 (Kubernetes), P2 (Terraform)

---

## Key Points to Remember

- **Ericsson = Employer** | **Verizon = Customer** — never say "I worked at Verizon"
- Mention Verizon once for brand credibility, then switch to "the platform"
- Always end with a forward-looking strength (automation, security, self-healing)
- Numbers stick: 500+ servers, 95% fewer failures, $180K saved, 3-min RTO
- Your unique edge: Linux System Admin foundation → understands things from kernel up

---
---

# SECTION 2: CI/CD & Jenkins

---

### Q: What triggers a build job in Jenkins when code is merged to a central repository?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "We use **webhooks**. GitHub/GitLab is configured with a webhook pointing to our Jenkins URL. When a developer pushes code or merges a PR to the central repository, GitHub sends an HTTP POST to Jenkins (e.g., `http://jenkins-url/github-webhook/`). Jenkins receives the payload, matches it to the configured job (based on repo URL and branch), and triggers the pipeline automatically.
>
> So the flow is: Developer pushes → GitHub fires webhook → Jenkins receives notification → Pipeline starts.
>
> In our 18-stage DevSecOps pipeline, every push to `main` triggers the full pipeline. For feature branches, we trigger a lighter pipeline (lint + unit test + SAST only) to give fast feedback."

**Key facts:**
- Webhook = push-based (GitHub notifies Jenkins). No polling needed.
- Alternative: Jenkins Poll SCM (`H/5 * * * *`) — checks every X minutes. Wasteful, adds delay. We don't use this.
- Webhook is near-instant (seconds after push).

---


### Q: If a build job fails, what actions do you take?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "First, I check **which stage failed** — our 18-stage pipeline has clear stage boundaries, so I know immediately if it's a compilation error, test failure, security scan failure, or deployment issue.
>
> **My approach:**
> 1. **Check the console log** — Jenkins Blue Ocean or classic console output. 90% of the time, the error is right there (dependency not found, test assertion failed, Trivy found critical CVE).
> 2. **Identify if it's code or infra** — Is it a flaky test? A network timeout pulling dependencies? Docker daemon issue on the agent? This determines who fixes it.
> 3. **Notify the developer** — Our pipeline sends Slack notifications on failure with the stage name, error snippet, and commit author. Developer owns code fixes.
> 4. **If it's infra** — I fix it. Common issues: agent disk full, Docker cache corrupted, SonarQube down, registry unreachable.
> 5. **If it's a security gate failure** — Trivy found a critical CVE or SonarQube quality gate failed. This is NOT bypassed. Developer must fix the vulnerability or get a documented exception.
>
> We also track build failure trends — if the same stage fails repeatedly, that's a systemic issue (flaky test, unreliable dependency mirror) that needs a permanent fix, not just a re-run."

**Key facts:**
- Never blindly re-run without understanding the failure
- Security gate failures are NEVER bypassed without documented exception
- Infrastructure issues (agent, registry, tools) = DevOps owns it
- Code/test failures = Developer owns it, DevOps assists

---

### Q: How does Maven understand how to build code with the required libraries?

**Project Reference:** P1 (DevSecOps Pipeline — build stage) | No direct Maven project, but concept applies

**Answer:**

> "Everything is defined in `pom.xml` — Maven's Project Object Model file. It's the single source of truth for the build.
>
> **How it works:**
> 1. **`pom.xml` declares dependencies** — groupId, artifactId, version. Maven knows exactly which libraries are needed.
> 2. **Maven downloads from repositories** — First checks local cache (`~/.m2/repository`), then remote repos (Maven Central, or a private Nexus/Artifactory mirror).
> 3. **Build lifecycle** — Maven has predefined phases: `validate → compile → test → package → install → deploy`. Each phase knows what to do (compile uses `javac`, test uses JUnit, package creates JAR/WAR).
> 4. **Transitive dependencies** — If your code needs Library A, and Library A needs Library B, Maven resolves the entire dependency tree automatically.
>
> It's declarative — you tell Maven WHAT you need, not HOW to get it. The `pom.xml` is the contract."

**Key facts:**
- `pom.xml` = the brain of the build (dependencies, plugins, build config)
- Local cache: `~/.m2/repository` — avoids re-downloading
- `mvn clean package` = most common command (clean old build → compile → test → create JAR)
- In CI/CD, we point Maven to a private Nexus/Artifactory mirror to avoid external dependency on Maven Central

**Note:** In our pipeline (P1), the app is Python/Django (uses `pip install -r requirements.txt`), but the concept is the same — a manifest file declares dependencies, a package manager resolves and downloads them. Maven = Java's equivalent.

---

### Q: How do you handle sensitive information (secrets/credentials) for your pipelines?

**Project Reference:** P1 (DevSecOps Pipeline), P9 (OS Patching)

**Answer:**

> "We handle secrets at three layers depending on where they're consumed:
>
> 1. **Build-time secrets (CI)** — Jenkins Credentials Store. API tokens, registry passwords, AWS access keys stored as Jenkins secrets. Referenced in Jenkinsfile as `credentials('secret-id')` — never hardcoded, never in logs (masked automatically).
>
> 2. **Runtime secrets (Kubernetes)** — Kubernetes Secrets (base64-encoded, but we also enforce encryption at rest via AWS KMS on EKS). Mounted as environment variables or volume files into pods.
>
> 3. **Infrastructure secrets (Ansible)** — Ansible Vault for encrypting sensitive variables (DB passwords, cert keys). Encrypted at rest in Git, decrypted at runtime with vault password file.
>
> Additionally, our pipeline has a **secret scanning stage** (Stage 1 — using `trufflehog` or `git-secrets`) that fails the build if anyone accidentally commits credentials to the repo."

**Key facts:**
- Never store secrets in code, environment files, or pipeline definitions
- Jenkins masks secrets in console output automatically
- K8s Secrets + IRSA (no long-lived AWS credentials in pods)
- Ansible Vault = encrypted at rest, safe to commit to Git

---

### Q: How did you integrate Vault with your pipeline?

**Project Reference:** P9 (OS Patching — Ansible Vault)

**Answer:**

> "If you're referring to **Ansible Vault** — we encrypt sensitive variable files locally on the dev machine using `ansible-vault encrypt`, push the encrypted file to GitHub. When Jenkins triggers the Ansible playbook, it passes the vault password file (`--vault-password-file`) at runtime to decrypt and use the secrets. The vault password itself is stored in Jenkins Credentials Store — so it's secrets protecting secrets.
>
> If you're referring to **HashiCorp Vault** — I haven't used it directly in my pipelines, but the concept is similar: pipeline authenticates to Vault (via AppRole or OIDC), fetches secrets dynamically at runtime, and secrets are never stored in Git or Jenkins."

**Key facts:**
- Ansible Vault: encrypt → push to Git → decrypt at runtime via Jenkins
- Vault password stored securely in Jenkins Credentials (not in repo)
- This approach keeps secrets version-controlled (encrypted) without exposure

---

### Q: Are these pipelines common for all environments (Dev, Staging, Prod), or are they different?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "Same pipeline, different behavior per environment. One Jenkinsfile handles all three — Dev, Staging, Production. The difference is controlled by:
>
> 1. **Branch-based triggering** — `develop` branch deploys to Dev, `release/*` deploys to Staging, `main` deploys to Prod.
> 2. **Environment-specific gates** — Dev: auto-deploy after tests pass. Staging: auto-deploy + run integration tests. Prod: requires manual approval + canary rollout (5% → 25% → 100%).
> 3. **Environment-specific Helm values** — Same chart, different `values-dev.yaml`, `values-staging.yaml`, `values-prod.yaml` (replica counts, resource limits, feature flags).
>
> One pipeline definition, different execution paths. DRY principle — no maintaining 3 separate pipelines."

---

### Q: Do you run Production from a release branch?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "Yes. Our flow is:
> - `feature/*` → Dev (auto-deploy)
> - `release/*` → Staging (auto-deploy + integration tests)
> - `main` → Production (after release branch is validated and merged)
>
> The release branch acts as a stabilization branch — only bug fixes go in, no new features. Once QA signs off on Staging, we merge release to main, which triggers the production pipeline with canary deployment.
>
> This gives us a clear audit trail: every production deployment maps to a specific release branch that was tested in Staging."

---

### Q: Do you use self-hosted runners or dynamic runners for GitLab?

**Project Reference:** P1 (DevSecOps Pipeline — Jenkins)

**Answer:**

> "We use Jenkins, not GitLab CI. Our Jenkins agents are **self-hosted and containerized** — a custom Docker image (Ubuntu + Sonar Scanner + Trivy + Java + AWS CLI) running as a persistent agent connected via SSH.
>
> The advantage: full control over tools installed, no cold-start delay, and consistent build environment. We also run the agent inside Docker Compose alongside Jenkins Master and SonarQube — the entire CI platform is reproducible in under 5 minutes.
>
> If I were to use GitLab, I'd prefer **dynamic runners** (autoscaling with Kubernetes executor) for cost efficiency — spin up per-job, scale to zero when idle. But self-hosted gives more control for security-sensitive builds where you need fixed egress IPs and pre-authenticated tool access."

---

### Q: How do you deploy a service into Kubernetes?

**Project Reference:** P1 (DevSecOps Pipeline — ArgoCD GitOps)

**Answer:**

> "We use **GitOps with ArgoCD**. The flow: CI pipeline (Jenkins) builds the image, pushes to ECR, updates the image tag in the Helm values file in a separate GitOps repo using `sed`. ArgoCD watches that repo and auto-syncs to the cluster.
>
> **Why GitOps over Ansible for K8s deployments:**
>
> 1. **Automatic sync** — ArgoCD auto-syncs desired state to cluster. No manual `kubectl apply` or Ansible playbook trigger needed.
> 2. **Git as audit trail** — Every change is a Git commit. Who changed what, when, and why — fully traceable.
> 3. **Drift detection** — If someone does `kubectl edit` manually in production, ArgoCD detects the drift and either alerts or auto-corrects. With Ansible, you'd never know until something breaks.
>
> We tried Ansible for K8s deployments initially — it works, but it's imperative (push-based). You run the playbook, it applies, done. If someone changes something manually after, Ansible doesn't know. Plus, rolling updates need extra handling in Ansible, whereas Kubernetes handles it natively with deployment strategy.
>
> GitOps is declarative and self-healing — the cluster always converges to what's in Git."

**Key facts:**
- GitOps = pull-based (ArgoCD pulls from Git). Ansible = push-based (you trigger it)
- ArgoCD gives: auto-sync, drift detection, rollback (just revert the Git commit), UI visibility
- Deployment strategy (RollingUpdate/Canary) is handled by Kubernetes + Argo Rollouts, not the deployment tool

---

### Q: How does your Jenkins communicate with EKS?

**Project Reference:** P1 (DevSecOps Pipeline), P4 (Landing Zone — OIDC federation)

**Answer:**

> "Jenkins doesn't talk to EKS directly for deployments — that's ArgoCD's job. But Jenkins DOES interact with EKS for stages like smoke tests, namespace setup, or secret creation.
>
> **The mechanism:**
> 1. Jenkins agent has `kubectl` and `aws` CLI installed.
> 2. Pipeline runs `aws eks update-kubeconfig --name cluster-name --region us-east-1` — this generates a kubeconfig that uses IAM for authentication.
> 3. The Jenkins agent uses an **IAM role (via OIDC federation)** — not long-lived access keys. Jenkins authenticates to AWS using OIDC token → assumes an IAM role → gets short-lived credentials → EKS validates via aws-iam-authenticator.
>
> So the chain is: Jenkins (OIDC) → AWS STS (assume role) → short-lived token → EKS API server.
>
> No static credentials stored anywhere. Tokens expire in 1 hour. If Jenkins is compromised, the blast radius is limited to what that IAM role can do (least-privilege)."

**Key facts:**
- No long-lived AWS keys — OIDC federation for CI/CD (P4 Landing Zone design)
- `aws eks update-kubeconfig` = the bridge between Jenkins and EKS
- Jenkins talks to EKS API server only for non-deployment tasks (smoke tests, validation)
- Actual deployment = ArgoCD watches Git, pulls changes, applies to cluster

---

### Q: Tell me five best practices for using CI/CD pipelines.

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> 1. **Pipeline as Code** — Jenkinsfile in Git, version-controlled, reviewed like application code. No click-ops.
> 2. **Fail fast** — Run cheap checks first (lint, unit tests), expensive ones later (DAST, integration). Developer gets feedback in 2 minutes, not 20.
> 3. **Security gates, not speed bumps** — Integrate security scanning (SAST, SCA, container scan) as pipeline stages. Automated, not an afterthought.
> 4. **Immutable artifacts** — Build once, promote everywhere. Same Docker image goes Dev → Staging → Prod. Never rebuild per environment.
> 5. **Automated rollback** — If production deployment fails health checks, auto-rollback. Don't rely on humans at 3 AM. We use Prometheus-driven canary auto-rollback.

---

### Q: What is your holistic view of DevOps and why was it introduced?

**Project Reference:** No direct project — general philosophy

**Answer:**

> "DevOps was introduced to break the wall between Development (who wants fast releases) and Operations (who wants stability). Traditional model: Dev throws code over the wall → Ops deploys → something breaks → blame game.
>
> DevOps fixes this by making both teams jointly responsible for the full lifecycle — build, deploy, run, monitor. The result: faster delivery WITH stability.
>
> My holistic view: DevOps = Automation + Culture + Feedback loops. Automate everything repeatable (CI/CD, IaC, patching). Build a culture of shared ownership. Close feedback loops (monitoring → alerts → fix → deploy). It's not a tool — it's how teams work together."

---

### Q: What do you mean by the statement that "DevOps is a cultural shift"?

**Project Reference:** No direct project — philosophy

**Answer:**

> "It means tools alone don't make DevOps. You can have Jenkins, Terraform, Kubernetes — but if developers don't own their deployments, if ops doesn't collaborate during design, if teams blame each other during incidents — you're just doing automation, not DevOps.
>
> Cultural shift means: developers run their own code in production (you build it, you own it), ops is involved early in design (not just handed a JAR to deploy), incidents are blameless postmortems (not finger-pointing), and everyone shares the pager.
>
> In my team, developers write Helm values, review Terraform PRs, and join incident bridges. That's the cultural shift — shared ownership, not silos with a CI/CD tool in between."

---

### Q: How does the continuous feedback loop from operations to development happen?

**Project Reference:** P1 (Prometheus-driven canary), P3 (Observability stack)

**Answer:**

> "Through monitoring and alerting feeding back into the development cycle:
>
> 1. **Prometheus + Grafana** — Dashboards show real-time error rates, latency, resource usage per service. Developers can see their code's behavior in production.
> 2. **Canary metrics** — During deployment, Prometheus metrics (5xx rate, latency P99) determine if new code is healthy. Bad metrics → auto-rollback → developer gets notified with the exact metric that failed.
> 3. **Alerting → Jira** — P3 alerts auto-create Jira tickets assigned to the owning dev team. Ops doesn't just absorb the pain.
> 4. **Distributed tracing (Jaeger)** — When MTTR matters, developers can trace a request across 15 microservices and see where it slowed down.
>
> The loop: Deploy → Monitor → Detect issue → Alert developer → Fix → Deploy again. Cycle time measured in hours, not weeks."

---

### Q: What types of CI/CD pipelines have you built and what challenges did you face?

**Project Reference:** P1 (DevSecOps Pipeline), P9 (OS Patching)

**Answer:**

> "Two main types:
>
> 1. **Container-based (K8s deployment)** — 18-stage DevSecOps pipeline. Build → Scan → Test → Push image → Update GitOps repo → ArgoCD deploys → Canary rollout.
> 2. **VM-based (Ansible deployment)** — OS patching pipeline. Jenkins triggers Ansible AAP → rolling update across 500+ servers with traffic drain and validation.
>
> **Challenges faced:**
> - **Long pipeline execution time** — Solved by parallelizing independent stages (SAST and SCA run together, not sequentially).
> - **Flaky security scans** — Trivy DB download timeout. Solved by pre-caching vulnerability DB on the agent.
> - **Environment drift** — Staging config didn't match Prod. Solved by GitOps — same Git repo, different value files.
> - **Rollback complexity** — Initial pipelines had no auto-rollback. Added Prometheus-driven canary analysis with Argo Rollouts."

---

### Q: Do you have any understanding of the multi-branch concept in Jenkins?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "Yes, we use it. A **Multibranch Pipeline** in Jenkins automatically discovers branches (and PRs) in your Git repo and creates a pipeline job for each branch. Each branch runs the same Jenkinsfile but can behave differently based on the branch name.
>
> In our setup:
> - `feature/*` branches → run lint + unit tests + SAST only (fast feedback)
> - `develop` → full pipeline, deploy to Dev
> - `release/*` → full pipeline, deploy to Staging
> - `main` → full pipeline + manual approval + canary production deployment
>
> Jenkins scans the repo periodically (or via webhook), creates/deletes jobs as branches are created/merged. No manual job creation needed."

---

### Q: Scenario: If you want a web application deployed to multiple environments automatically after passing test cases, what tools would you use?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "My tool chain:
>
> | Layer | Tool | Why |
> |---|---|---|
> | Source control | GitHub/GitLab | Branching strategy, webhooks |
> | CI | Jenkins | Pipeline as code, extensible, mature |
> | Build | Docker | Immutable container image |
> | Security scan | Trivy + SonarQube | Container + code quality gates |
> | Artifact store | ECR | Container registry |
> | CD | ArgoCD | GitOps, auto-sync per environment |
> | Orchestration | Kubernetes (EKS) | Multi-env namespaces or separate clusters |
> | Deployment strategy | Argo Rollouts | Canary for prod, rolling for lower envs |
> | Monitoring | Prometheus + Grafana | Validate deployment health |
>
> **Configuration:** One Jenkinsfile with branch-based logic. Helm chart with per-environment values. ArgoCD Application per environment pointing to same chart, different values file. Tests pass → image promoted → GitOps repo updated → ArgoCD syncs."

---

### Q: Is there any monitoring tool for security analysis of containers or images?

**Project Reference:** P1 (DevSecOps Pipeline — Stage 10: Container Scanning)

**Answer:**

> "Yes, several. We use **Trivy** — it scans container images for OS vulnerabilities (CVEs) and application library vulnerabilities. Runs in our pipeline: `trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:latest`.
>
> Other options:
> - **Trivy** — Open source, fast, CI-friendly (what we use)
> - **Snyk Container** — SaaS, good developer experience, integrates with registries
> - **AWS ECR Image Scanning** — Built into ECR, basic scanning on push
> - **Aqua Security / Sysdig** — Runtime container security (not just build-time)
>
> We scan at two points: build-time (pipeline gate) and runtime (continuous scanning in registry). A critical CVE blocks deployment. A medium CVE creates a Jira ticket with SLA to fix."

---

### Q: Is Jenkins the right tool for 10-12+ applications?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "Yes, Jenkins handles it fine — we run pipelines for 15+ microservices on a single Jenkins instance. But it depends on HOW you set it up:
>
> **What makes it work at scale:**
> - Shared libraries — Common pipeline logic in a shared Groovy library. Don't duplicate Jenkinsfiles across 12 repos.
> - Multiple agents — Parallel builds across containerized agents. One master, multiple executors.
> - Folder organization — Group by team/project.
>
> **When Jenkins becomes painful:**
> - Plugin management hell (100+ plugins, compatibility issues)
> - Groovy debugging is terrible
> - No native GitOps support (need ArgoCD separately)
>
> **Would I choose Jenkins today for a greenfield project?** Probably GitHub Actions or GitLab CI for simpler setups. But for enterprise with complex pipelines, shared libraries, and existing investment — Jenkins still works. It's mature, flexible, and battle-tested."

---

### Q: Have you explored alternatives to Jenkins?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "Yes:
>
> | Tool | My Experience | When I'd Choose It |
> |---|---|---|
> | **GitHub Actions** | Used for smaller projects | Cloud-native, YAML-based, great for GitHub repos |
> | **GitLab CI** | Familiar, used in some teams | Tight Git integration, built-in registry, good for all-in-one |
> | **ArgoCD** | Use actively for CD | GitOps-native, but it's CD only — not CI |
> | **Tekton** | Explored | K8s-native CI/CD, good if you're all-in on K8s |
>
> **Why we still use Jenkins:** Existing investment, complex shared libraries (Groovy), integration with Ansible AAP, and the flexibility to handle both container AND VM pipelines. Migration cost isn't justified yet. But for new projects, I'd lean toward GitHub Actions (CI) + ArgoCD (CD)."

---

---
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

# SECTION 4: Docker & Containers

---

### Q: Any top two or three findings when people create Docker containers?

**Project Reference:** P1 (DevSecOps Pipeline — Docker best practices)

**Answer:**

> "Top 3 mistakes I see repeatedly:
>
> 1. **Running as root** — Default Docker containers run as root (UID 0). If container is compromised, attacker has root inside. Fix: add `USER 1001` in Dockerfile. We enforce this via Kyverno admission policy — no root containers allowed in production.
>
> 2. **Using `latest` tag** — Developers use `FROM python:latest`. Breaks reproducibility — image changes without notice. Fix: always pin versions (`FROM python:3.9-slim@sha256:...`). Our pipeline rejects `latest` tags.
>
> 3. **Fat images with unnecessary packages** — Installing `vim`, `curl`, `gcc` in production image. Increases attack surface and image size. Fix: multi-stage builds — build in a fat image, copy only the binary/artifact to a slim runtime image. Our production images are <100MB."

---

### Q: By default, what is the user that a Docker container runs as?

**Project Reference:** P1 (DevSecOps Pipeline — Dockerfile)

**Answer:**

> "**Root (UID 0)**. Unless you explicitly specify a `USER` directive in the Dockerfile, the container process runs as root.
>
> This is a security risk — if an attacker escapes the container, they're root on the host (unless user namespaces are configured).
>
> In our Dockerfile, we explicitly set:
> ```dockerfile
> RUN adduser --disabled-password --no-create-home appuser
> USER appuser  # UID 1001
> ```
>
> And in Kubernetes, we enforce it via Pod Security Standards (restricted profile) — `runAsNonRoot: true`, `allowPrivilegeEscalation: false`. Kyverno blocks any pod trying to run as root."

---

### Q: Do you follow any standards for containers? Do you enforce anything?

**Project Reference:** P1 (DevSecOps Pipeline), P3 (Kubernetes — Pod Security Standards)

**Answer:**

> "Yes, we enforce at multiple levels:
>
> | Level | What We Enforce | How |
> |---|---|---|
> | **Build time** | No root user, pinned base image, multi-stage build, no secrets in layers | Dockerfile linting + pipeline checks |
> | **Registry** | Only signed images (Cosign) | Kyverno admission policy — unsigned = rejected |
> | **Runtime** | Non-root, no privilege escalation, read-only rootfs, drop all capabilities | Pod Security Standards (restricted profile) |
> | **Scanning** | No HIGH/CRITICAL CVEs | Trivy gate in pipeline + continuous ECR scanning |
>
> It's not just guidelines — it's enforced. A developer can't bypass it because Kyverno admission controller rejects non-compliant pods at the API server level, and the pipeline fails before the image even reaches the cluster."

---
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

# SECTION 6: Role Targeting & Closing

---

### Q: What are you looking at in terms of your next role or what type of role are you targeting?

**Project Reference:** General — career direction

**Answer:**

> "I'm looking for a **Senior/Lead DevOps or Platform Engineering role** where I can:
>
> 1. **Design and own the platform** — Not just execute tickets, but architect the CI/CD, infrastructure, and deployment strategy for the organization.
> 2. **Work at scale** — Large production environments, multi-region, high availability. That's where my 10+ years of carrier-grade telecom experience adds the most value.
> 3. **Drive DevSecOps adoption** — Security integrated into pipelines from day one, not bolted on later.
>
> I want to be in a role where I'm building systems that teams depend on — not maintaining someone else's legacy. I want ownership, impact, and the ability to mentor junior engineers."

---

### Q: Tell me about the few projects that you have done?

**Project Reference:** All 9 projects (pick top 3-4 based on relevance to the role)

**Note:** This is a shorter version of the Type 3 intro. Don't repeat full career arc — jump straight to projects.

**Answer:**

> "I'll highlight my top four:
>
> 1. **DevSecOps CI/CD Pipeline (P1)** — 18-stage Jenkins pipeline with 6 security layers, GitOps with ArgoCD, canary deployments with auto-rollback. Reduced deployment failures by 95%.
>
> 2. **OS Patching Automation (P9)** — Zero-touch patching for 500+ RHEL servers using Ansible AAP. 18-step lifecycle with automated validation, ServiceNow ITIL integration, zero downtime.
>
> 3. **Kubernetes Platform (P3)** — Operated K8s across 3 environments (EKS, self-managed, OpenShift) serving 15+ microservices with full observability stack.
>
> 4. **Multi-Region DR (P8)** — Active-passive disaster recovery with Route53 failover, Aurora Global DB — achieved 3-minute RTO, validated quarterly through automated drills.
>
> Each project solved a real business problem — not just tech for tech's sake."

---

### Q: As technologies are changing every day, how do you cope with learning?

**Project Reference:** General — learning approach

**Answer:**

> "Three things I do consistently:
>
> 1. **Build, don't just read** — I maintain a personal knowledge base (Obsidian) where I document every technology I learn with hands-on labs. Reading docs ≠ knowing it. I spin up clusters, break things, fix them.
>
> 2. **Follow the problem, not the hype** — I don't chase every new tool. When a real problem arises (e.g., 'how do we do canary deployments?'), I research options, evaluate, and implement. That's how I learned Argo Rollouts, Karpenter, and Istio.
>
> 3. **Community + certifications** — I follow KubeCon talks, AWS re:Invent sessions, and maintain my skills through practical projects on GitHub. Certifications give structure when learning something new end-to-end.
>
> The key: I learn by doing, and I only invest time in technologies that solve real production problems."

---

### Q: Convince me to hire you — based on the most challenging project you've done.

**Project Reference:** P9 (OS Patching Automation) or P1 (DevSecOps Pipeline)

**Answer:**

> "The most challenging project was building the **OS Patching Automation for 500+ servers**.
>
> **Why it was hard:** You're touching 500+ production Linux servers that run a carrier-grade voice platform. One wrong package, one missed validation — and millions of calls drop. The client had zero tolerance for downtime.
>
> **What I did:**
> - Designed an 18-step zero-touch lifecycle — from ServiceNow CR creation to post-patch validation
> - Built 12 Ansible roles — each independently testable, idempotent, and re-runnable
> - Implemented 7-dimension automated validation (services, ports, connectivity, disk, certs, integrity, logs)
> - ALB traffic drain with serial 20% rolling updates — no user impact
> - One-click rollback if any validation dimension fails
>
> **Result:** 500+ servers patched monthly with ZERO downtime, ZERO human intervention after initiation. Passed every ITIL audit. Reduced patching from a 3-day manual exercise with 2 incidents/month to a 4-hour automated run with zero incidents for 18 months straight.
>
> **Why hire me:** I don't just automate — I build systems that are safe to run without humans watching. I think about failure modes, rollback, and validation before writing a single line of code. That's 10 years of production experience talking."

---
---

# SECTION 7: Technical Architecture & Concepts

---

### Q: If you are to design a highly available and fault tolerant architecture on AWS, what are the key things you'll look at?

**Project Reference:** P2 (3-Tier AWS Architecture), P8 (Multi-Region HA/DR)

**Answer:**

> "Five key pillars:
>
> 1. **Multi-AZ everything** — VPC across 3 AZs. ALB distributes traffic across AZs. EC2 ASG spans all 3. Aurora Multi-AZ with automatic failover (<30 seconds). No single AZ dependency.
>
> 2. **Auto-scaling** — ASG with target tracking (CPU-based or request-based). Scale 3→20 instances. Karpenter for Kubernetes node scaling.
>
> 3. **Stateless compute** — Application instances are disposable. Session data in ElastiCache/DynamoDB, not local disk. Any instance can serve any request.
>
> 4. **Database resilience** — Aurora Multi-AZ (same region HA) + Aurora Global Database (cross-region DR). Read replicas for read-heavy workloads.
>
> 5. **Health checks + circuit breakers** — ALB health checks remove unhealthy targets in seconds. Route53 health checks trigger DNS failover for regional outages.
>
> For fault tolerance specifically: assume anything can fail. Design so that when it does, traffic automatically routes to healthy components without human intervention."

---

### Q: How would you manage the database if it's a multi-region setup?

**Project Reference:** P8 (Multi-Region HA/DR)

**Answer:**

> "We use **Aurora Global Database**:
>
> - **Primary region** (us-east-1): handles all writes. Replication to secondary region in <1 second (typically ~100ms).
> - **Secondary region** (us-west-2): read-only replica. Promotes to read-write in ~1 minute during failover.
> - **RPO: <1 second** — minimal data loss during regional failure.
>
> **Failover process:**
> 1. Route53 health check detects primary region down
> 2. DNS failover routes traffic to secondary region
> 3. Aurora secondary promotes to primary (detach + promote, ~60 seconds)
> 4. Application in DR region starts receiving write traffic
>
> **Key consideration:** After failover, the old primary must be re-created as a secondary. It's NOT automatic re-join. We handle this in our DR runbook.
>
> **Alternative for non-Aurora:** DynamoDB Global Tables (active-active, multi-region writes, no manual failover needed). We'd use this for session stores or metadata — not transactional data."

---

### Q: Explain when a user hits a website from the internet, what are the different steps that happen in the background?

**Project Reference:** P2 (3-Tier AWS Architecture)

**Answer:**

> "Full request flow in our architecture:
>
> 1. **DNS resolution** — User types `app.example.com`. Browser queries DNS → Route53 returns CloudFront distribution IP (or ALB IP if no CDN).
>
> 2. **TLS handshake** — Browser establishes HTTPS connection. SSL terminates at CloudFront or ALB (ACM certificate).
>
> 3. **CDN cache check** — CloudFront checks if static content is cached at edge. If yes → serve directly (fast). If no → forward to origin.
>
> 4. **WAF inspection** — Request passes through AWS WAF rules (SQL injection, XSS, rate limiting, geo-blocking). Bad request → blocked.
>
> 5. **Load balancer** — ALB receives request, routes to healthy target (EC2/pod) based on path rules and health checks.
>
> 6. **Application processing** — App server handles business logic, queries database (Aurora), returns response.
>
> 7. **Database query** — App connects to Aurora reader/writer endpoint. Connection pooling, query execution, response.
>
> 8. **Response back** — Reverse path: App → ALB → CloudFront (caches if cacheable) → User.
>
> Total time for a cached request: ~20-50ms. Uncached with DB: ~100-300ms."

---

### Q: Explain the concept of stateful and stateless infrastructure?

**Project Reference:** P2 (3-Tier AWS), P3 (Kubernetes — StatefulSets vs Deployments)

**Answer:**

> "**Stateless:** Instance doesn't store any data locally. Kill it, replace it — no data loss. All shared state lives externally (database, cache, object store). Example: our EC2 web servers in ASG, Kubernetes Deployments. Any instance serves any request.
>
> **Stateful:** Instance holds data that must persist. You can't just kill and replace it without handling the data. Example: databases, Kafka brokers, etcd nodes. In Kubernetes, these run as StatefulSets with Persistent Volumes.
>
> **Why it matters for DevOps:**
> - Stateless = easy to scale, easy to patch (just terminate and launch new). Our ASG rolling update works because instances are stateless.
> - Stateful = hard to scale, hard to patch (must drain data, failover, then patch). Our database patching needs Aurora failover, not just 'terminate instance.'
>
> **Our rule:** Keep compute stateless wherever possible. Push state to managed services (Aurora, ElastiCache, S3). This is why we use EBS-backed PVCs in K8s only for databases, not for application pods."

---

### Q: How would you debug a slow performing application where the web page takes a lot of time to render?

**Project Reference:** P3 (Observability stack), P6 (Istio — Jaeger tracing)

**Answer:**

> "I follow a **top-down approach** — start broad, narrow down:
>
> 1. **Where is the time spent?** — Check browser dev tools (Network tab). Is it DNS? TLS? TTFB (time to first byte)? Large payload? If TTFB is high → backend problem. If content download is slow → payload/network.
>
> 2. **Backend tracing** — Use Jaeger distributed tracing. It shows exactly which microservice call took how long. 'Order service: 50ms, Payment service: 2.5 seconds' — found the bottleneck.
>
> 3. **Database slow queries** — Check Aurora Performance Insights or slow query log. A missing index on a 10M row table = 3-second query.
>
> 4. **Resource saturation** — Check Prometheus/Grafana: Is the pod CPU-throttled? Memory swapping? Is the node overloaded? Check `kubectl top pods`.
>
> 5. **Network latency** — Check if cross-AZ calls are adding latency. Istio metrics show inter-service latency per hop.
>
> 6. **External dependencies** — Third-party API slow? Check timeout settings, add circuit breakers.
>
> Key: Don't guess. Use traces to pinpoint the exact bottleneck, then fix surgically."

---

### Q: In terms of monitoring, tell me the difference between metrics and traces?

**Project Reference:** P3 (Observability stack — Prometheus + Jaeger)

**Answer:**

> "**Metrics** = WHAT is happening (aggregated numbers).
> - 'HTTP 500 errors increased by 40% in the last 5 minutes'
> - 'Average response time is 350ms'
> - 'CPU usage is at 85%'
> - Good for: alerting, dashboards, trends. Tools: Prometheus, CloudWatch.
>
> **Traces** = WHY is it happening (per-request journey).
> - 'This specific request took 3.2 seconds because the payment-service DB query took 2.8 seconds'
> - Shows the full call chain across microservices with timing per hop
> - Good for: debugging specific slow requests, finding bottlenecks. Tools: Jaeger, X-Ray.
>
> **How they work together:** Metrics tell you THERE IS a problem (alert: latency spike). Traces tell you WHERE the problem is (this specific DB call in this specific service). You need both — metrics for detection, traces for diagnosis."

---

### Q: Tell me the difference between a monolithic and microservices architecture?

**Project Reference:** P1 (15+ microservices), P3 (Kubernetes platform)

**Answer:**

> | Aspect | Monolith | Microservices |
> |---|---|---|
> | **Codebase** | Single deployable unit | Many independent services |
> | **Deployment** | Deploy everything together | Deploy services independently |
> | **Scaling** | Scale the whole app | Scale only the hot service |
> | **Failure** | One bug can crash everything | Failure isolated to one service |
> | **Complexity** | Simple to start, hard to maintain at scale | Complex infra, but each service is simple |
>
> **In our environment:** We run 15+ microservices on Kubernetes. Each team owns their service, deploys independently via ArgoCD, scales independently via HPA. A bug in the notification service doesn't take down the payment service.
>
> **The trade-off:** Microservices need more infrastructure — service mesh (Istio), distributed tracing (Jaeger), centralized logging (EFK). A monolith is simpler if your team is small and app is straightforward."

---

### Q: How would you enable the data of one microservice to reach another microservice if they are in different clusters?

**Project Reference:** P6 (Istio Service Mesh), P8 (Multi-Region)

**Answer:**

> "Three common approaches depending on the use case:
>
> 1. **API Gateway / External service** — Service A in Cluster 1 calls Service B in Cluster 2 via an external API Gateway (Kong, AWS API Gateway) or Ingress endpoint. Simple, works across any cluster. Downside: goes through the internet/ALB, adds latency.
>
> 2. **Istio Multi-Cluster Mesh** — Both clusters are part of the same Istio mesh. Service discovery works transparently across clusters. Service A calls Service B by Kubernetes service name — Istio routes it cross-cluster over mTLS. Seamless, secure, no code changes.
>
> 3. **Event-driven (async)** — Services communicate via a shared message broker (Kafka, SQS, EventBridge). Service A publishes an event, Service B in another cluster consumes it. No direct coupling, eventual consistency.
>
> **What we'd use:** For synchronous real-time calls → Istio multi-cluster or API Gateway. For async data sharing → Kafka/EventBridge. Never direct pod-to-pod across clusters without encryption and service identity."

---

### Q: What do you know about API gateways and APIs?

**Project Reference:** P2 (ALB + WAF), P5 (Serverless — API Gateway + Lambda)

**Answer:**

> "An API Gateway is a single entry point that sits in front of your backend services. It handles:
>
> - **Routing** — `/users` → user service, `/orders` → order service
> - **Authentication/Authorization** — Validate JWT tokens, API keys before traffic hits backend
> - **Rate limiting** — Prevent abuse (100 requests/min per client)
> - **SSL termination** — Handle TLS at the gateway, backends get plain HTTP
> - **Request transformation** — Add headers, modify payloads
> - **Caching** — Cache GET responses to reduce backend load
>
> **In our work:**
> - AWS API Gateway + Lambda (P5) — Serverless APIs, pay per request, built-in throttling
> - ALB as a basic API gateway (P2) — Path-based routing to different target groups
> - Istio Ingress Gateway (P6) — K8s-native, mTLS, AuthorizationPolicies, traffic management
>
> For microservices, the API gateway decouples clients from internal service topology. Clients call one URL, gateway routes to the right service."

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

# SECTION 9: Deployment Incidents & Lessons

*Note: Terraform-specific Q&A has been moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)*

---

### Q: Can you recall from your experience where you were deploying something and it caused an outage or didn't go as planned?

**Project Reference:** P9 (OS Patching), P8 (DR drills)

**Answer:**

> "Yes. Early in the OS patching project, before we had full automation:
>
> **What happened:** We were patching a batch of RHEL servers manually. A kernel update required a reboot. After reboot, the application service started — but a shared library (.so file) had been updated by the patch, and the application binary was linked against the old version. Service came up, passed basic systemctl checks, but started throwing segfaults under load.
>
> **Impact:** 15-minute degraded performance on the contact center platform. Calls were routing but with audio quality issues.
>
> **Root cause:** We checked 'is the service running?' but not 'is the service actually healthy under traffic?' No deep validation.
>
> **What I built after this:**
> - 7-dimension automated validation in P9 (services, ports, connectivity, disk, certs, integrity checks, log errors)
> - **Integrity check dimension** — md5sum of 8 critical application files compared against baseline. If a library changes unexpectedly, automation catches it before traffic returns.
> - **Traffic drain before patching** — ALB removes the server from rotation BEFORE we patch. Traffic only returns AFTER all 7 validations pass.
>
> That incident is why our patching system has zero incidents for 18 months since automation went live."

---
---

# SECTION 10: Closing & Soft Skills

---

### Q: As technologies are changing every day, how do you cope with learning?

**Project Reference:** General — learning approach

**Answer:**

> "Three things I do consistently:
>
> 1. **Build, don't just read** — I maintain a personal knowledge base (Obsidian) where I document every technology I learn with hands-on labs. Reading docs ≠ knowing it. I spin up clusters, break things, fix them.
>
> 2. **Follow the problem, not the hype** — I don't chase every new tool. When a real problem arises (e.g., 'how do we do canary deployments?'), I research options, evaluate, and implement. That's how I learned Argo Rollouts, Karpenter, and Istio.
>
> 3. **Community + certifications** — I follow KubeCon talks, AWS re:Invent sessions, and maintain my skills through practical projects on GitHub. Certifications give structure when learning something new end-to-end.
>
> The key: I learn by doing, and I only invest time in technologies that solve real production problems."

---

### Q: Convince me to hire you — based on the most challenging project you've done.

**Project Reference:** P9 (OS Patching Automation)

**Answer:**

> "The most challenging project was building the **OS Patching Automation for 500+ servers**.
>
> **Why it was hard:** You're touching 500+ production Linux servers that run a carrier-grade voice platform. One wrong package, one missed validation — and millions of calls drop. The client had zero tolerance for downtime.
>
> **What I did:**
> - Designed an 18-step zero-touch lifecycle — from ServiceNow CR creation to post-patch validation
> - Built 12 Ansible roles — each independently testable, idempotent, and re-runnable
> - Implemented 7-dimension automated validation (services, ports, connectivity, disk, certs, integrity, logs)
> - ALB traffic drain with serial 20% rolling updates — no user impact
> - One-click rollback if any validation dimension fails
>
> **Result:** 500+ servers patched monthly with ZERO downtime, ZERO human intervention after initiation. Passed every ITIL audit. Reduced patching from a 3-day manual exercise with 2 incidents/month to a 4-hour automated run with zero incidents for 18 months straight.
>
> **Why hire me:** I don't just automate — I build systems that are safe to run without humans watching. I think about failure modes, rollback, and validation before writing a single line of code. That's 10 years of production experience talking."

---

---
---

# SECTION 11: AWS Architectural Design (E-Commerce System Design)

---

### Q: Design a highly scalable, robust, and available international e-commerce application using AWS services.

**Project Reference:** P2 (3-Tier AWS Architecture), P8 (Multi-Region HA/DR)

**Answer:**

> "Here's the architecture I'd design:
>
> **Frontend/CDN Layer:**
> - CloudFront (global CDN) — static assets cached at 400+ edge locations worldwide. International users get <50ms response for static content.
> - Route53 — latency-based routing to nearest regional deployment. Geo-restriction if needed.
> - S3 — static website hosting (React/Angular SPA).
>
> **Application Layer:**
> - EKS (Kubernetes) — microservices: product catalog, cart, orders, payments, inventory, notifications. Each scales independently.
> - ALB + WAF — L7 load balancing with DDoS and bot protection.
> - API Gateway — for external partner APIs, rate limiting, API key management.
>
> **Data Layer:**
> - Aurora Global Database — primary in us-east-1, read replicas in eu-west-1 and ap-southeast-1 for low-latency reads. <1s cross-region replication.
> - ElastiCache (Redis) — session store + product catalog cache. Reduces DB load by 80%.
> - DynamoDB Global Tables — cart data (active-active multi-region, no failover needed).
> - S3 — product images, user uploads, order documents.
>
> **Async/Event Layer:**
> - SQS — order processing queue (decouple order placement from fulfillment).
> - EventBridge — event-driven: 'order placed' triggers inventory update, notification, analytics.
> - Lambda — lightweight processing (email notifications, image resizing).
>
> **Security:**
> - WAF + Shield Advanced — DDoS protection for e-commerce (peak traffic during sales).
> - ACM certificates, mTLS between services (Istio).
> - Secrets Manager for DB credentials, API keys.
>
> **Key design decisions:**
> - Stateless compute → easy horizontal scaling during flash sales
> - Async processing → order page returns fast, fulfillment happens in background
> - Multi-region with DynamoDB Global Tables for cart → user never loses their cart regardless of region
> - Aurora Global DB for orders → strong consistency for financial transactions"

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

# SECTION 12: Kubernetes & EKS

---

### Q: What are some common Kubernetes troubleshooting scenarios you have faced in production?

**Project Reference:** P3 (Kubernetes Platform — EKS, self-managed, OpenShift)

**Answer:**

> "Top scenarios I've dealt with:
>
> 1. **Pod stuck in CrashLoopBackOff** — App failing on startup. Check: `kubectl logs <pod> --previous`. Common causes: missing config/secret, wrong image tag, DB not reachable, OOMKilled (check `kubectl describe pod` → Last State → Reason).
>
> 2. **Pod stuck in Pending** — No node can schedule it. Check: `kubectl describe pod` → Events. Causes: insufficient CPU/memory on nodes (Karpenter not scaling?), PVC not bound (storage class issue), node affinity/taint mismatch.
>
> 3. **OOMKilled** — Container hit memory limit. Fix: increase limits OR fix the memory leak. Check: `kubectl describe pod` → `OOMKilled`, then `dmesg | grep -i oom` on the node.
>
> 4. **Service unreachable** — Pod is running but can't be reached. Check: selector mismatch between Service and Pod labels, endpoints (`kubectl get endpoints`), NetworkPolicy blocking traffic, kube-proxy iptables rules not propagated.
>
> 5. **Node NotReady** — kubelet stopped or disk pressure. Check: `kubectl describe node` → Conditions. SSH to node: `systemctl status kubelet`, `journalctl -u kubelet`, `df -h`.
>
> 6. **ImagePullBackOff** — Can't pull image. Causes: wrong image name, ECR auth expired (`aws ecr get-login-token`), registry unreachable, image doesn't exist.
>
> **My approach:** Always start with `kubectl describe` → Events section tells you 80% of the story."

---

### Q: How can you provide EKS pods access to DynamoDB and S3 without using static access/secret keys?

**Project Reference:** P4 (Landing Zone — OIDC), P3 (EKS — IRSA)

**Answer:**

> "**IRSA — IAM Roles for Service Accounts.**
>
> How it works:
> 1. Create an IAM role with the permissions needed (DynamoDB read/write, S3 access)
> 2. Create a trust policy that trusts the EKS OIDC provider — scoped to a specific Kubernetes ServiceAccount in a specific namespace
> 3. Annotate the K8s ServiceAccount: `eks.amazonaws.com/role-arn: arn:aws:iam::123:role/my-role`
> 4. Pod uses that ServiceAccount → EKS injects a web identity token → AWS SDK exchanges it for temporary IAM credentials automatically
>
> **Result:** Pod gets short-lived credentials (15 min–12 hr). No static keys. No secrets to rotate. If pod is compromised, credentials expire automatically.
>
> **Key security points:**
> - Scoped to namespace + ServiceAccount (not cluster-wide)
> - Least privilege — each pod gets ONLY what it needs
> - Auditable via CloudTrail — you see which pod assumed which role
>
> We use this for everything: ECR pull, S3 access, DynamoDB, Secrets Manager. Zero static credentials in our cluster."

---

### Q: What version of EKS are you currently on?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "We're on **EKS 1.31** currently, with an upgrade to 1.32 planned next quarter. We stay one version behind latest to let the community shake out bugs. EKS supports N-3 versions, so we have runway.
>
> We upgrade every 3-4 months to stay within support. Each upgrade follows our tested runbook — control plane first, then managed node groups, then self-managed components (CoreDNS, kube-proxy, VPC CNI add-ons)."

**Key fact:** Don't say "latest" — shows you don't track versions. Give a specific version and show you have an upgrade cadence.

---

### Q: How would you plan and carry out a migration of an EKS cluster from version 1.33 to 1.34?

**Project Reference:** P3 (Kubernetes Platform — upgrades)

**Answer:**

> "It's a sequential, tested process:
>
> **Pre-upgrade:**
> 1. Read release notes — check deprecated APIs (`kubectl convert` or `kubent` to find deprecated resources in our manifests)
> 2. Check add-on compatibility — VPC CNI, CoreDNS, kube-proxy, Karpenter, ArgoCD, Istio versions against K8s 1.34 support matrix
> 3. Test in non-prod first — upgrade Dev cluster, run full integration test suite
> 4. Backup — etcd snapshot (self-managed) or just ensure Terraform state is current (EKS managed)
>
> **Upgrade sequence:**
> 1. **Control plane** — `aws eks update-cluster-version --name cluster --kubernetes-version 1.34`. Takes ~20 min. Zero downtime (EKS manages it).
> 2. **Add-ons** — Update CoreDNS, kube-proxy, VPC CNI to compatible versions.
> 3. **Node groups** — Rolling update. Launch new 1.34 nodes → drain old nodes → terminate. Pods migrate gracefully via PodDisruptionBudgets.
> 4. **Validate** — All pods running, services reachable, monitoring healthy, no deprecated API warnings.
>
> **Key risks:** Deprecated APIs (moved to GA in new version), webhook compatibility, CNI plugin version mismatch. That's why we test in Dev first."

---

### Q: Which tool would you use to deploy your Docker image onto an EKS cluster?

**Project Reference:** P1 (DevSecOps Pipeline — ArgoCD)

**Answer:**

> "**ArgoCD** — GitOps-based. CI pipeline (Jenkins) pushes the image to ECR, updates the Helm values file in the GitOps repo, ArgoCD auto-syncs it to EKS.
>
> For the Helm chart itself: **Helm** packages the Kubernetes manifests with templating and version control."

**Note:** Short answer as asked. If they probe deeper, reference the existing "How do you deploy a service into Kubernetes" answer.

---

### Q: Have you worked with cloud-native tools like Cert-Manager or Nginx Ingress?

**Project Reference:** P3 (Kubernetes Platform), P10 (Ingress Controllers)

**Answer:**

> "Yes, both:
>
> **Cert-Manager** — Automates TLS certificate lifecycle in Kubernetes. We use it with Let's Encrypt (for non-prod) and AWS ACM PCA (for prod). It creates Certificate resources, requests from the CA, stores in K8s Secrets, and auto-renews before expiry. No manual cert rotation.
>
> **Nginx Ingress Controller** — Our L7 ingress in self-managed clusters. Handles path-based routing, TLS termination, rate limiting, custom headers. On EKS, we use AWS ALB Ingress Controller instead (native ALB integration), but Nginx is used in our self-managed kubeadm clusters.
>
> Both are deployed via Helm with ArgoCD managing their lifecycle."

---

### Q: If release notes state that your Nginx ingress controller is not compatible with the new Kubernetes version, what would you do?

**Project Reference:** P3 (Kubernetes — upgrade planning)

**Answer:**

> "Three options in priority order:
>
> 1. **Upgrade Nginx Ingress first** — Check if a newer version of Nginx Ingress IS compatible with K8s 1.34. Usually, the ingress controller releases a compatible version within weeks. Upgrade Nginx Ingress → then upgrade K8s. This is the normal path.
>
> 2. **If no compatible version exists yet** — Wait. Don't upgrade K8s until ingress controller supports it. Ingress is critical path — all traffic flows through it. Breaking it = full outage.
>
> 3. **If wait isn't an option (security patch urgency)** — Evaluate switching to AWS ALB Ingress Controller (aws-load-balancer-controller) which is maintained by AWS and always compatible with latest EKS. Migrate ingress resources, test, then upgrade K8s.
>
> **Key principle:** Never upgrade K8s if a critical cluster component (ingress, CNI, CSI driver) isn't compatible. Test the full stack in Dev first. The upgrade is only safe when ALL components work together."

---

### Q: What happens when a user clicks on a public DNS (abc.com) until the request reaches the pod in your EKS cluster?

**Project Reference:** P2 (3-Tier), P3 (Kubernetes networking)

**Answer:**

> "Full flow:
>
> 1. **DNS resolution** — Browser queries DNS. Route53 returns the ALB's IP address (or CloudFront if CDN is in front).
>
> 2. **TLS handshake** — Browser establishes HTTPS with ALB. SSL terminates at ALB (ACM certificate).
>
> 3. **ALB routing** — ALB checks Ingress rules (path/host-based). Forwards to the correct Target Group.
>
> 4. **Target Group → Node** — ALB sends traffic to a NodePort on one of the EKS worker nodes (or directly to pod IP if using IP-mode target groups with VPC CNI).
>
> 5. **kube-proxy / iptables** — If NodePort mode: kube-proxy's iptables rules DNAT the request to the actual pod IP (could be on same node or different node).
>
> 6. **Pod receives request** — Traffic enters the pod's network namespace, hits the container port, application processes it.
>
> **With Istio (P6):** Between step 5 and 6, the Envoy sidecar intercepts traffic (iptables redirect in pod's network namespace), applies mTLS, AuthorizationPolicy, then forwards to the app container on localhost.
>
> **IP-mode (what we use):** ALB sends directly to pod IP (VPC CNI assigns routable IPs to pods). Skips NodePort/kube-proxy. Lower latency, better load distribution."

---

### Q: What parameters, besides CPU, memory, and disk, would you alert on to monitor a Kubernetes cluster?

**Project Reference:** P3 (Observability stack)

**Answer:**

> "Beyond the obvious CPU/memory/disk:
>
> 1. **Pod restart count** — Pods restarting = CrashLoopBackOff, OOMKill, or liveness probe failure. Alert if restarts > 3 in 5 minutes.
>
> 2. **Pending pods** — Pods stuck in Pending = scheduling failure (resource exhaustion, node issues). Should be 0 in production.
>
> 3. **Node conditions** — DiskPressure, MemoryPressure, PIDPressure, NetworkUnavailable. Any True = alert.
>
> 4. **API server latency** — If etcd or API server is slow (>500ms), the whole cluster suffers. Alert on apiserver_request_duration_seconds.
>
> 5. **Certificate expiry** — Kubelet, API server, etcd certs. Alert 30 days before expiry (cert-manager handles this for app certs).
>
> 6. **Endpoint readiness** — `kube_endpoint_address_not_ready` — services with no ready endpoints = user-facing outage.
>
> 7. **HPA at max** — If HPA is at max replicas for >10 minutes, it can't scale further. Need bigger nodes or higher max.
>
> 8. **PVC usage** — Persistent volumes approaching capacity (>85%). Especially critical for stateful workloads.
>
> 9. **Network errors** — Pod network drops, DNS resolution failures (CoreDNS errors).
>
> 10. **Image pull failures** — ECR token expired, rate limiting."

---

### Q: What steps would you take to secure a default Kubernetes cluster?

**Project Reference:** P3 (Kubernetes security), P1 (DevSecOps)

**Answer:**

> "Default K8s is INSECURE out of the box. Here's what I harden:
>
> 1. **RBAC** — Disable anonymous access. Create specific Roles/ClusterRoles per team. No one gets cluster-admin except break-glass.
>
> 2. **Pod Security Standards** — Enforce `restricted` profile. No root containers, no privilege escalation, no hostNetwork, no hostPID.
>
> 3. **Network Policies** — Default-deny all traffic. Explicitly allow only required paths. Ingress → frontend → backend → database. Nothing else.
>
> 4. **Secrets encryption** — Enable encryption at rest for etcd (KMS provider on EKS). Secrets are encrypted, not just base64.
>
> 5. **Image policies** — Kyverno: only images from trusted ECR registry, only signed images (Cosign). Block `latest` tag, block Docker Hub in prod.
>
> 6. **API server access** — Private endpoint (no public access). Access only via VPN/bastion. Audit logging enabled → CloudTrail.
>
> 7. **Node security** — Minimal AMI (Bottlerocket), no SSH to nodes in prod, IMDSv2 enforced, regular patching.
>
> 8. **Service accounts** — `automountServiceAccountToken: false` by default. Only mount when needed.
>
> 9. **Resource limits** — LimitRange per namespace. Prevent one pod from starving others.
>
> 10. **Runtime security** — Falco for anomaly detection, read-only root filesystem on all containers."

---

### Q: What are the security constraints called in the Kubernetes world?

**Project Reference:** P3 (Kubernetes security)

**Answer:**

> "They've evolved over time:
>
> - **Pod Security Policies (PSP)** — Deprecated since K8s 1.21, removed in 1.25. Was the original way to restrict what pods can do.
>
> - **Pod Security Standards (PSS) + Pod Security Admission (PSA)** — The replacement. Three levels:
>   - `privileged` — no restrictions (only for system components)
>   - `baseline` — prevents known privilege escalations (no hostNetwork, no hostPID)
>   - `restricted` — full hardening (non-root, drop all capabilities, read-only rootfs, no privilege escalation)
>
> - **External policy engines (what we use):**
>   - **Kyverno** — Kubernetes-native policy engine. We write ClusterPolicies that enforce: no root, require resource limits, require labels, only signed images, block NodePort services.
>   - **OPA/Gatekeeper** — Alternative to Kyverno. Rego-based policies.
>
> We use PSA (baseline at namespace level) + Kyverno (for granular enforcement beyond what PSA offers). Belt and suspenders."

---

---
---

# ~~SECTION 13: Terraform Advanced~~ → Moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)

---
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

# SECTION 16: Git & Branching Strategy

---

### Q: Can you explain your branching strategy and how you handle releases?

**Project Reference:** P1 (DevSecOps Pipeline — branch-based deployment)

**Answer:**

> "We use **Git Flow variant** tailored for CI/CD:
>
> ```
> main          ← production (always deployable, protected)
> └── release/* ← stabilization for next release (staging)
>     └── develop    ← integration branch (dev environment)
>         └── feature/* ← individual developer work
> ```
>
> **Flow:**
> 1. Developer creates `feature/add-payment` from `develop`
> 2. PR → code review → merge to `develop` → auto-deploy to Dev
> 3. When ready for release: create `release/v2.1` from `develop`
> 4. Release branch → auto-deploy to Staging. Only bug fixes allowed here.
> 5. QA signs off → merge `release/v2.1` to `main` → triggers production pipeline (canary deployment)
> 6. Tag `v2.1.0` on main for tracking
>
> **Hotfix path:** `hotfix/*` branch from `main` → fix → PR to `main` directly → emergency pipeline → also merge back to `develop` to keep branches in sync.
>
> **Key rules:**
> - `main` is protected — no direct pushes, requires PR + approval
> - Release branches are short-lived (1-2 weeks max)
> - Feature branches are even shorter (2-3 days ideally)"

---

### Q: Three team members are working on the same file with different changes. How do you merge this into a higher-level branch, and what issues might arise?

**Project Reference:** P1 (team collaboration)

**Answer:**

> "This is a **merge conflict** scenario. Here's how it plays out:
>
> **Process:**
> 1. First developer merges their PR — no issues (clean merge)
> 2. Second developer's PR now shows conflict — Git can't auto-merge because the same lines were modified
> 3. Second developer must: `git pull origin develop` → resolve conflicts manually → push → re-request review
> 4. Third developer — same process, conflicts with both previous changes
>
> **Issues that arise:**
> - **Merge conflicts** — Most common. Same file, same lines. Must be resolved manually.
> - **Semantic conflicts** — No Git conflict, but code is logically incompatible. Tests catch this (one person renamed a function, another person calls the old name).
> - **Lost changes** — If someone resolves conflict carelessly and accepts 'theirs' without reading.
>
> **How we minimize this:**
> - **Small, short-lived branches** — Merge daily, not weekly. Less divergence = fewer conflicts.
> - **File ownership** — In microservices, each team owns their service files. Rarely do 3 people edit the same file.
> - **Rebase before merge** — `git pull --rebase origin develop` keeps history clean and surfaces conflicts early.
> - **CI runs on PR** — After conflict resolution, pipeline re-runs to verify nothing broke."

---

### Q: Did you use CLI-based or UI-based Git?

**Project Reference:** General — daily workflow

**Answer:**

> "**Primarily CLI.** For daily work — `git add`, `commit`, `push`, `pull`, `rebase`, `log`, `diff` — all CLI. It's faster and scriptable.
>
> **UI for:** Pull request reviews (GitHub/GitLab web UI), merge conflict visualization (VS Code GitLens), and browsing history graphically.
>
> **In CI/CD:** Everything is CLI — Jenkins pipeline runs `git rev-parse --short HEAD` for image tags, `sed` to update GitOps repo, `git push` to trigger ArgoCD. No UI possible in automation.
>
> At 12 years experience, CLI is muscle memory. But I don't gatekeep — junior devs using VS Code Git integration or GitKraken is fine as long as the commits are clean."

---

### Q: What is the first command you use to start working on a repository, and what command to push your finished work?

**Project Reference:** General — Git fundamentals

**Answer:**

> "**Start:**
> ```bash
> git clone git@github.com:org/repo.git    # First time — get the repo
> git checkout -b feature/my-feature       # Create feature branch
> ```
>
> Or if repo already cloned:
> ```bash
> git pull origin develop                   # Get latest changes
> git checkout -b feature/my-feature       # Branch from latest
> ```
>
> **Finish:**
> ```bash
> git add .                                 # Stage changes (or specific files)
> git commit -m \"feat: add payment service\"  # Commit with conventional message
> git push -u origin feature/my-feature    # Push to remote, set upstream
> ```
> Then create a PR via GitHub/GitLab UI.
>
> **Key point:** I never push to `main` or `develop` directly. Always feature branch → PR → review → merge."

---

### Q: What is the difference between git pull and git fetch?

**Project Reference:** General — Git fundamentals

**Answer:**

> "**`git fetch`** — Downloads changes from remote but does NOT apply them to your working branch. Your local code stays untouched. It just updates your remote tracking references (`origin/main`, `origin/develop`).
>
> **`git pull`** — Does `git fetch` + `git merge` (or `git rebase` if configured). Downloads AND applies changes to your current branch immediately.
>
> ```bash
> git fetch origin        # Safe — just downloads. You can review before merging.
> git pull origin main    # Downloads AND merges into your branch immediately.
> ```
>
> **When I use which:**
> - `git fetch` — When I want to check what changed on remote before merging. `git log origin/main..HEAD` shows what's different.
> - `git pull --rebase` — My daily workflow. Rebase my local commits on top of latest remote. Keeps history linear, avoids unnecessary merge commits.
>
> **Key insight:** `git pull` can cause unexpected merge conflicts if you have uncommitted work. `git fetch` is always safe."

---

### Q: What is the difference between a central repository and a fork?

**Project Reference:** General — Git workflow

**Answer:**

> "**Central repository** — One shared repo that everyone pushes to directly (via branches). All developers have write access. This is what we use internally at Ericsson — everyone works on feature branches within the same repo.
>
> **Fork** — A complete copy of the repo in YOUR account. You push to your fork, then create a PR from your fork to the original repo. You DON'T have write access to the original.
>
> | Aspect | Central Repo | Fork |
> |---|---|---|
> | **Access** | All team members have write access | Only maintainers have write to original |
> | **Branches** | Feature branches in same repo | Feature branches in your copy |
> | **Use case** | Internal team collaboration | Open source, untrusted contributors |
> | **PR flow** | branch → main (same repo) | fork/branch → upstream/main (cross-repo) |
>
> **When to use which:**
> - Internal team (trusted devs) → Central repo with branch protection
> - Open source / external contributors → Fork model (don't give strangers write access to your repo)
>
> We use central repo internally. Fork model only when contributing to open-source projects (e.g., submitting a fix to a Helm chart upstream)."

---

---
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

# SECTION 19: Cloud Experience & Domain

---

### Q: Can you detail your experience across AWS and GCP?

**Project Reference:** All projects (AWS-focused)

**Answer:**

> "**AWS — Deep, production experience (10+ years):**
> - All 9 projects are on AWS. VPC, EKS, RDS Aurora, Lambda, Organizations, Route53, CloudFront, IAM, S3, DynamoDB, EventBridge — all in production at scale.
> - Multi-account architecture (15 accounts), multi-region DR, 500+ EC2 instances managed via IaC.
> - Certifications: familiar with AWS Well-Architected Framework across all 6 pillars.
>
> **GCP — Conceptual knowledge, not production:**
> - Understand the service mappings: GKE (=EKS), Cloud Run (=Fargate), Cloud Storage (=S3), BigQuery, MIG (=ASG), VPC, IAM.
> - Haven't managed production GCP infrastructure, but the concepts transfer: networking, IAM, IaC (Terraform works the same), Kubernetes is Kubernetes.
>
> **My position:** I'm strongest in AWS, but Terraform + Kubernetes skills are cloud-agnostic. Give me a GCP project — the learning curve is the service names and console, not the architecture patterns. Those are universal.
>
> If this role requires GCP, I'll ramp up quickly because the fundamentals (networking, security, IaC, containers, CI/CD) are identical — only the service APIs differ."

---

---
---

# SECTION 20: Infrastructure, Scripting & Monitoring

---

### Q: How did you configure the EKS or Kubernetes setup?

**Project Reference:** P3 (Kubernetes Platform — EKS, self-managed, OpenShift)

**Answer:**

> "We have EKS provisioned entirely via Terraform:
>
> **EKS cluster (Terraform):**
> - VPC with private subnets (3 AZs) — pods run in private subnets only
> - EKS control plane (managed by AWS) + managed node groups (or Karpenter for dynamic nodes)
> - VPC CNI add-on with prefix delegation (high pod density — 110+ pods/node)
> - CoreDNS, kube-proxy as managed add-ons
> - OIDC provider enabled (for IRSA — pod-level IAM)
> - Private endpoint (API server not public-facing)
>
> **Post-cluster setup (ArgoCD bootstraps everything):**
> - Karpenter (node autoscaling — replaces managed node groups for worker nodes)
> - AWS Load Balancer Controller (ALB ingress)
> - External Secrets Operator (sync secrets from Secrets Manager)
> - Prometheus + Grafana stack (monitoring)
> - Istio (service mesh — if needed)
> - Kyverno (policy enforcement)
> - Cert-Manager (TLS automation)
>
> **Self-managed (kubeadm) setup (P3):**
> - Ansible bootstrap: disable swap, load kernel modules, install containerd, kubeadm init, join workers, install Calico CNI
> - More operational overhead but full control (used for on-prem/edge deployments)
>
> **Key point:** Cluster creation is Terraform. Everything that runs ON the cluster is GitOps (ArgoCD). Clean separation."

---

### Q: What is one of the challenges you faced while doing these integrations?

**Project Reference:** P3 (Kubernetes Platform), P1 (DevSecOps Pipeline)

**Answer:**

> "One major challenge: **IRSA (IAM Roles for Service Accounts) not working after EKS cluster recreation.**
>
> **What happened:** We rebuilt the EKS cluster (Terraform destroy + apply for a version upgrade in Dev). After recreation, pods with IRSA stopped working — getting `AccessDenied` from AWS APIs.
>
> **Root cause:** The OIDC provider URL changed (new cluster = new OIDC endpoint). But our IAM role trust policies still referenced the OLD OIDC provider ARN. Terraform created the new OIDC provider but didn't update all the IAM roles that trusted it (they were in a separate Terraform state/module).
>
> **Fix:**
> 1. Updated IAM role trust policies to reference the new OIDC provider ARN
> 2. Restructured Terraform so OIDC provider ARN is exported as an output and consumed by IAM roles module via `terraform_remote_state` — so they always stay in sync
>
> **Lesson:** Cross-module dependencies need explicit data passing. Don't hardcode ARNs across Terraform modules — use outputs and remote state references.
>
> This is the kind of challenge that only surfaces in real production — docs don't warn you about it."

---

*Note: "Why do you need Python if you are using Terraform?" has been moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)*

---

### Q: How about shell scripting — what do you use it for?

**Project Reference:** P1 (Jenkinsfile), P9 (OS Patching — Ansible shell modules)

**Answer:**

> "Shell/Bash for quick, system-level tasks that don't need a full language:
>
> 1. **Pipeline glue** — In Jenkinsfile: `sed` to update image tags, `curl` for health checks, `aws` CLI calls, exit code handling. Pipeline stages are essentially shell scripts.
>
> 2. **Ansible shell/command modules** — When no Ansible module exists for what I need. E.g., custom validation checks: `ss -tlnp | grep :8080`, `md5sum /etc/app/config.yml`.
>
> 3. **Instance bootstrap (user-data)** — EC2 startup scripts: install packages, configure CloudWatch agent, mount EBS volumes. Runs once at launch.
>
> 4. **Quick debugging on servers** — `for` loops to check something across multiple files, `grep` + `awk` for log parsing, disk space checks.
>
> **When I DON'T use shell:**
> - Anything >50 lines → Python (shell gets unreadable fast)
> - Anything with error handling complexity → Python (bash error handling is fragile)
> - Anything that needs JSON/API processing → Python (jq in bash is painful vs Python dict)
>
> **Rule:** Shell for glue and one-liners. Python for logic. Terraform for infra. Right tool for the right job."

---

### Q: Why not use Ansible for these automation tasks (instead of Python/Shell)?

**Project Reference:** P9 (Ansible AAP), P1 (Jenkins)

**Answer:**

> "Ansible is great for server configuration but wrong tool for some tasks:
>
> | Task | Right Tool | Why Not Ansible |
> |---|---|---|
> | Lambda function logic | Python | Ansible doesn't run inside Lambda |
> | CI/CD pipeline stages | Shell (in Jenkinsfile) | Ansible overhead for a 2-line command is overkill |
> | API calls (Jira, Slack) | Python | Ansible URI module works but Python is cleaner for complex logic |
> | One-shot data processing | Python | Ansible is for idempotent server config, not ETL/data tasks |
> | Quick health check in pipeline | Shell (`curl`) | Ansible requires inventory, playbook, YAML — too heavy for one curl |
>
> **When I DO use Ansible:**
> - Server configuration (install packages, configure services, manage files)
> - OS patching (P9 — 500+ servers, idempotent roles)
> - Multi-server orchestration (serial rolling updates with traffic drain)
> - Anything that needs idempotency + inventory-based targeting
>
> **Key insight:** Ansible's power is idempotency and inventory. If the task isn't about 'configure this group of servers to a desired state' — Ansible is the wrong tool. Use Python/Shell for logic, Terraform for infra, Ansible for configuration."

---

### Q: Are you using any other resources like CloudWatch?

**Project Reference:** P3 (Observability), P2 (3-Tier AWS)

**Answer:**

> "Yes, CloudWatch is part of our stack but not the primary monitoring tool for Kubernetes:
>
> **What we use CloudWatch for:**
> - **AWS service metrics** — RDS (CPU, connections, replica lag), ALB (5xx count, response time, healthy hosts), Lambda (invocations, errors, duration)
> - **CloudWatch Alarms** — ALB unhealthy host count > 0 → SNS → PagerDuty
> - **CloudWatch Logs** — VPC Flow Logs, CloudTrail logs, Lambda logs (these are AWS-native, can't easily redirect)
> - **CloudWatch Agent on EC2** — Custom metrics (memory, disk — not available by default)
>
> **What we use Prometheus/Grafana for (primary):**
> - Kubernetes metrics (pod, node, container level)
> - Application metrics (custom business metrics via ServiceMonitor)
> - Alerting (AlertManager — richer routing than CloudWatch Alarms)
>
> **Why both:** CloudWatch is unavoidable for AWS services (RDS, ALB have no Prometheus-native metrics without exporters). Prometheus is better for K8s and custom application metrics. We federate: CloudWatch → YACE exporter → Prometheus → Grafana dashboard shows everything in one place."

---

---
---

# SECTION 21: Microservices Concepts

---

### Q: What do you mean by 'loosely coupled'? Can you give an example?

**Project Reference:** P1 (15+ microservices), P5 (Event-driven architecture)

**Answer:**

> "**Loosely coupled** means services can change, deploy, scale, and fail independently without affecting each other.
>
> **Example from our system:**
> - **Order Service** places an order → publishes an event to SQS/EventBridge: `OrderPlaced`
> - **Notification Service** listens for `OrderPlaced` → sends email confirmation
> - **Inventory Service** listens for `OrderPlaced` → reduces stock count
>
> Order Service doesn't know or care about Notification Service. It just fires the event and moves on. If Notification Service crashes — orders still work. If we replace the Notification Service with a completely new implementation — Order Service doesn't change.
>
> **Tightly coupled (bad):** Order Service directly calls Notification Service via HTTP synchronously. Notification down = order fails. Can't change Notification without coordinating with Order team.
>
> **How we achieve loose coupling:**
> - Async communication (SQS, EventBridge) — not direct HTTP calls
> - API contracts (OpenAPI spec) — agreed interface, implementation is independent
> - Separate databases per service — no shared DB schema
> - Independent deployment via ArgoCD — one service deploys without touching others"

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

# SECTION 24: Ansible

---

### Q: How do you download modules in Ansible? Which modules have you used?

**Project Reference:** P9 (OS Patching — 12 Ansible roles)

**Answer:**

> "**Terminology clarification:** Ansible has 'modules' (task-level plugins like `yum`, `copy`, `service`) and 'collections' (packages of modules + roles from Ansible Galaxy).
>
> **Downloading collections:**
> ```bash
> # From Ansible Galaxy
> ansible-galaxy collection install amazon.aws
> ansible-galaxy collection install community.general
>
> # From requirements file (what we use in CI)
> ansible-galaxy install -r requirements.yml
> ```
>
> `requirements.yml`:
> ```yaml
> collections:
>   - name: amazon.aws
>     version: \">=6.0.0\"
>   - name: community.general
>     version: \">=7.0.0\"
> ```
>
> **Modules I use regularly (P9 OS Patching):**
>
> | Module | Use Case |
> |---|---|
> | `ansible.builtin.dnf` | Install/update packages (patching) |
> | `ansible.builtin.service` | Start/stop/restart services |
> | `ansible.builtin.systemd` | Systemd service management |
> | `ansible.builtin.copy` | Deploy config files |
> | `ansible.builtin.template` | Jinja2 templated configs |
> | `ansible.builtin.shell` | Custom commands (ss, md5sum checks) |
> | `ansible.builtin.stat` | Check file existence/permissions |
> | `ansible.builtin.reboot` | Controlled reboot with wait |
> | `ansible.builtin.wait_for` | Wait for port/service availability |
> | `ansible.builtin.uri` | HTTP health checks |
> | `amazon.aws.ec2_instance` | EC2 management |
> | `community.general.nmcli` | Network configuration |
>
> All modules are idempotent — run them 10 times, same result. That's why Ansible works for patching: re-run is safe."

---

---
---

# SECTION 25: Kubernetes Fundamentals

---

### Q: Why would you use Kubernetes, what are its advantages, and what are its different components?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "**Why Kubernetes:**
> You have 15+ microservices in containers. You need them to self-heal, scale, deploy without downtime, and communicate securely. Doing this manually on VMs is impossible at scale. Kubernetes automates container orchestration.
>
> **Key advantages:**
> 1. **Self-healing** — Pod crashes → K8s restarts it. Node dies → pods reschedule to healthy nodes.
> 2. **Auto-scaling** — HPA scales pods on CPU/memory/custom metrics. Karpenter scales nodes.
> 3. **Rolling updates + rollback** — Zero-downtime deployments. One command rollback.
> 4. **Service discovery** — Pods find each other via DNS (CoreDNS). No hardcoded IPs.
> 5. **Declarative** — Describe desired state in YAML. K8s converges to it.
>
> **Components:**
>
> | Control Plane | Worker Node |
> |---|---|
> | API Server (entry point for all commands) | kubelet (manages pods on node) |
> | etcd (key-value store — cluster state) | kube-proxy (network routing/iptables) |
> | Scheduler (assigns pods to nodes) | Container runtime (containerd) |
> | Controller Manager (ensures desired state) | Pods (your application) |
>
> Plus: CoreDNS (service discovery), CNI plugin (Calico/VPC CNI — pod networking), Ingress Controller (external traffic entry)."

---

### Q: What was the size of the infrastructure you managed using Kubernetes?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "Across our three Kubernetes environments:
>
> - **EKS (primary production):** 3 clusters (dev/staging/prod), 15+ microservices, ~50-80 pods in production, 10-20 worker nodes (Karpenter scales dynamically), multi-AZ across 3 availability zones.
>
> - **Self-managed (kubeadm):** 1 cluster, 5 masters (HA etcd), 15 workers. Used for specific workloads requiring more control.
>
> - **OpenShift:** 1 cluster, managed by client. We deploy workloads to it.
>
> **Total:** ~500+ pods across environments, serving the contact center platform with 99.95% uptime SLA. Supporting 15+ microservices with full observability stack (Prometheus, Grafana, Jaeger, EFK).
>
> The Kubernetes platform supports the same Verizon contact center infrastructure that previously ran on 500+ VMs — we're progressively containerizing workloads."

---

### Q: What are the three types of services in Kubernetes?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "Four actually (3 main + 1 headless):
>
> | Type | What It Does | Use Case |
> |---|---|---|
> | **ClusterIP** (default) | Internal-only IP. Reachable only within cluster | Service-to-service communication (backend → database) |
> | **NodePort** | Exposes on each node's IP at a static port (30000-32767) | Development/testing, legacy access |
> | **LoadBalancer** | Provisions external cloud load balancer (ALB/NLB on AWS) | Exposing service to internet |
> | **Headless** (`clusterIP: None`) | No cluster IP. Returns pod IPs directly via DNS | StatefulSets (each pod individually addressable — Kafka, databases) |
>
> **What we use:**
> - **ClusterIP** — 90% of services. Internal communication.
> - **LoadBalancer** — NOT used directly. Instead, we use Ingress Controller (ALB) which is more flexible (path/host routing, SSL, WAF).
> - **NodePort** — Avoided in production (security risk — exposes ports on all nodes).
>
> **Key point:** In production, we expose services via **Ingress** (ALB controller) not raw LoadBalancer type. Ingress gives you path-based routing, SSL termination, WAF — one ALB for multiple services instead of one NLB per service."

---
---

# SECTION 26: Team Management

---

### Q: Tell me about the management required for your team of six — was it infrastructure or application-oriented?

**Project Reference:** General — team leadership at Ericsson

**Answer:**

> "**Both.** Our team of 10+ spans infrastructure AND application delivery:
>
> - **Infrastructure side (my primary focus):** Terraform modules, Kubernetes platform operations, CI/CD pipeline maintenance, monitoring, patching, DR drills. I own the platform that applications run on.
>
> - **Application support:** We don't write application code, but we own the deployment lifecycle. We help dev teams containerize their apps, write Helm charts, configure pipelines, debug deployment issues, and optimize resource usage.
>
> **My management approach:**
> - I lead technically — architecture decisions, code reviews on Terraform/Ansible PRs, design docs
> - Distribute work by expertise — 2 engineers focused on K8s, 2 on Terraform/AWS, 1 on CI/CD, 1 on monitoring
> - Cross-train so no single point of failure — everyone can handle basic tasks across domains
> - Weekly knowledge sharing sessions — one person demos what they built
>
> **Key insight at senior level:** You're not just managing tasks. You're building a team's capability. I invest in documentation, runbooks, and automation so the team isn't dependent on any one person (including me)."

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

# SECTION 29: Behavioral & Leadership

---

### Q: Can you describe a time when you handled a crisis?

**Project Reference:** P9 (OS Patching — before automation), P8 (DR)

**Answer:**

> "**The crisis:** Saturday night, 11 PM. A kernel security patch (CVE with active exploit) was flagged as critical by Red Hat. Client demanded emergency patching of 50 production servers within 24 hours — normally a 3-day planned activity.
>
> **What I did:**
> 1. **Assessed risk** — Read the CVE. Confirmed: remote code execution, public exploit available. Patching couldn't wait.
> 2. **Formed the plan** — Emergency CR in ServiceNow. Identified the 50 highest-risk servers (internet-facing). Decided on serial 10% batches (5 servers at a time).
> 3. **Communicated** — Called the on-call team lead, briefed the client NOC. Set up a bridge call for real-time status updates.
> 4. **Executed** — Used our existing Ansible automation (thank god it was already built). Ran patching in controlled batches. Each batch: drain traffic → patch → reboot → validate → return traffic.
> 5. **Monitored** — Watched Prometheus dashboards for error rate spikes after each batch. Zero issues.
>
> **Result:** 50 servers patched in 6 hours with zero incidents. Client commended the response speed. This validated the investment in automation — without it, this would have been a 48-hour manual marathon with high risk.
>
> **Lesson I applied:** After this, I added an 'emergency fast-track' mode to our automation — fewer validation checks for speed but retains critical ones (service health, connectivity)."

---

### Q: In a crisis where a primary application is down, how would you communicate with stakeholders and distribute work?

**Project Reference:** General — incident management

**Answer:**

> "**Structured incident response:**
>
> **First 5 minutes:**
> 1. Declare incident severity (P1 — customer impacting)
> 2. Open a war room (Zoom/Slack channel) — all relevant people join
> 3. Assign roles: **Incident Commander** (me — coordinates, doesn't debug), **Tech Lead** (hands-on debugging), **Comms lead** (updates stakeholders)
>
> **Communication cadence:**
> - Stakeholders (management/client): Update every 15 minutes — even if the update is 'still investigating, no change.' Silence breeds panic.
> - Format: 'Impact: [what's broken]. Status: [investigating/identified/fixing]. ETA: [if known]. Next update: [time].'
>
> **Work distribution:**
> - Person A: Check application logs, recent deployments, code changes
> - Person B: Check infrastructure — nodes, pods, database, network connectivity
> - Person C: Check external dependencies — third-party APIs, DNS, certificates
> - Me: Coordinate, eliminate duplicate effort, escalate if needed, decide on rollback
>
> **Key principles:**
> - Don't have 5 people troubleshooting the same thing — parallelize
> - If not resolved in 15 min → rollback the last change (most incidents are caused by recent changes)
> - Document actions in real-time (Slack thread) — for postmortem later
> - After resolution: blameless postmortem within 48 hours"

---

### Q: Tell me about a time you influenced your team to do something new or brought a change that helped them.

**Project Reference:** P9 (OS Patching — shift from manual to automation)

**Answer:**

> "**The change:** Moving from manual SSH-based patching to full Ansible Automation Platform.
>
> **The resistance:** Team was comfortable with manual patching. 'We know our servers. Automation is risky — what if it patches the wrong thing?' The fear was valid — automation touching 500+ production servers is scary.
>
> **How I influenced:**
>
> 1. **Started small** — Didn't propose automating everything at once. First automated just the PRE-CHECK (connectivity, disk space, service status). Non-destructive. 'Let's just automate the checks — no actual patching yet.'
>
> 2. **Showed the data** — Tracked manual patching incidents: 2 per month, average 45-min recovery. Showed the team: 'This is what manual errors cost us.'
>
> 3. **Built with the team** — Didn't build it alone and hand it over. Paired with team members on each role. They owned roles they were expert in (cert checks, service validation).
>
> 4. **Ran parallel** — First 3 months: automation ran alongside manual process. Same servers, same window. Compared results. Automation caught 3 things humans missed.
>
> 5. **Celebrated wins** — First fully automated patch cycle with zero issues → team dinner. Made it a team achievement, not 'my automation replacing their jobs.'
>
> **Result:** Team went from resistant to proud. They now present the automation at internal tech talks. Two team members learned Ansible deeply and now contribute roles independently."

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

# SECTION 33: Lambda & Serverless

---

### Q: When have you needed a serverless (Lambda) function, and what steps did you take?

**Project Reference:** P5 (Serverless Event-Driven Architecture)

**Answer:**

> "**Use case (P5): Automated security remediation.** AWS Config detects a violation (e.g., Security Group opened to 0.0.0.0/0) → EventBridge triggers Lambda → Lambda auto-fixes it within 90 seconds. No human intervention.
>
> **Why Lambda (not EC2/container):**
> - Runs only when triggered — no idle cost ($0.07/month total for our remediation engine)
> - Event-driven — perfect for 'when X happens, do Y'
> - No infrastructure to manage — no patching, no scaling decisions
> - Completes in seconds — short-lived tasks, not long-running services
>
> **Steps I took:**
> 1. **Identified the trigger** — AWS Config rule violation → EventBridge event
> 2. **Wrote the function** — Python (boto3). Logic: receive event → parse violation type → determine remediation → apply fix → log to DynamoDB
> 3. **IAM role (least privilege)** — Lambda role can ONLY modify the specific resource types it remediates (SG rules, S3 bucket policies)
> 4. **Error handling** — Dead Letter Queue (SQS) for failed invocations. Alert if remediation fails.
> 5. **Testing** — SAM CLI (`sam local invoke`) for local testing before deploying
> 6. **Deployment** — Terraform provisions the Lambda + EventBridge rule + IAM role + DLQ
>
> **When NOT to use Lambda:**
> - Long-running tasks (>15 min) → use ECS/Step Functions
> - High-throughput APIs with consistent traffic → use EKS (Lambda cold starts add latency)
> - Stateful workloads → use containers"

---

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

# SECTION 35: Databases

---

### Q: Have you used any NoSQL databases? What are the differences between RDS and DynamoDB?

**Project Reference:** P5 (Serverless — DynamoDB), P2 (3-Tier — Aurora RDS)

**Answer:**

> "Yes, DynamoDB in our serverless security remediation engine (P5).
>
> | Aspect | RDS (Aurora) | DynamoDB |
> |---|---|---|
> | **Type** | Relational (SQL) | NoSQL (key-value / document) |
> | **Schema** | Fixed schema (tables, columns, relations) | Schemaless (flexible attributes per item) |
> | **Query** | SQL (complex joins, aggregations) | Simple key-based queries (partition + sort key) |
> | **Scaling** | Vertical (bigger instance) + read replicas | Horizontal (infinite — AWS manages sharding) |
> | **Transactions** | Full ACID | Limited transactions (TransactWriteItems) |
> | **Cost model** | Pay per instance hour (always on) | Pay per request OR provisioned capacity |
> | **Use case** | Transactional data (orders, financial) | High-throughput, simple access patterns (sessions, events, logs) |
> | **Latency** | ~5-10ms | Single-digit ms at any scale |
>
> **When I use which:**
> - **Aurora (P2):** Order management, user accounts — needs joins, complex queries, ACID guarantees
> - **DynamoDB (P5):** Remediation audit logs — write-heavy, simple lookups by violation ID, global tables for multi-region
>
> **VPC-bound?** RDS = YES (lives in VPC private subnets, accessed via Security Group). DynamoDB = NO (AWS-managed endpoint, accessed via IAM + VPC endpoint for private access)."

---

### Q: Which of these databases are bound by a VPC, or are both?

**Project Reference:** P2, P5

**Answer:**

> "**RDS = VPC-bound.** It lives in your private subnets. You control access via Security Groups. It has a private IP within your VPC.
>
> **DynamoDB = NOT VPC-bound.** It's a fully managed AWS service with a public endpoint (like S3). Access is controlled by IAM policies, not Security Groups.
>
> **However:** To access DynamoDB from a private subnet WITHOUT going through the internet (via NAT Gateway), you use a **VPC Gateway Endpoint** — routes traffic to DynamoDB over AWS's private network. No data transfer costs, more secure.
>
> ```hcl
> # Terraform — DynamoDB VPC endpoint
> resource \"aws_vpc_endpoint\" \"dynamodb\" {
>   vpc_id       = module.vpc.vpc_id
>   service_name = \"com.amazonaws.us-east-1.dynamodb\"
>   route_table_ids = module.vpc.private_route_table_ids
> }
> ```
>
> This is one of our cost optimization measures (P7) — VPC endpoints eliminate NAT Gateway data processing charges ($0.045/GB) for DynamoDB and S3 traffic."

---
---

# SECTION 36: Git (Advanced)

---

### Q: Can you think of a scenario where Git can be used in the continuous feedback loop?

**Project Reference:** P1 (DevSecOps — GitOps), P3 (ArgoCD)

**Answer:**

> "Yes — **GitOps IS the continuous feedback loop in code form.**
>
> **Scenario: Production auto-rollback updates Git:**
>
> 1. Developer merges code → Pipeline builds image → Updates Git repo (Helm values) → ArgoCD deploys canary
> 2. Prometheus detects error rate spike → Argo Rollouts auto-rolls back → Reverts the Git commit in the GitOps repo
> 3. Developer gets notification: 'Your deployment was rolled back. Commit X reverted. Error rate exceeded threshold.'
> 4. Git log now shows: the attempted deployment AND the rollback — full audit trail
> 5. Developer opens the failed commit, sees what changed, fixes the bug, pushes again
>
> **Git is the feedback mechanism:**
> - Deployment state = Git commit. Rollback = Git revert. History = Git log.
> - Every production change is traceable to a Git commit
> - Feedback from production (monitoring) flows BACK into Git (reverted commit)
>
> **Another scenario: Infrastructure drift feedback:**
> - Scheduled `terraform plan` detects drift → creates a Git issue automatically: 'Drift detected: Security Group modified outside Terraform'
> - Team investigates and either codifies the change or reverts it
> - Git issue = feedback from operations to development"

---
---

# SECTION 37: Kubernetes Manifests

---

### Q: Can you explain the contents of deployment.yaml and service.yaml, and why they are different?

**Project Reference:** P3 (Kubernetes Platform), P1 (Helm deployments)

**Answer:**

> "They serve completely different purposes:
>
> **deployment.yaml — WHAT to run:**
> ```yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   name: payment-service
> spec:
>   replicas: 3                    # How many pods
>   selector:
>     matchLabels:
>       app: payment               # Which pods this deployment manages
>   template:
>     spec:
>       containers:
>       - name: payment
>         image: ecr.aws/payment:v2.1   # Container image
>         ports:
>         - containerPort: 8080     # Port app listens on
>         resources:
>           requests:
>             cpu: 100m
>             memory: 256Mi
>         livenessProbe:            # Is it alive?
>           httpGet:
>             path: /health
>             port: 8080
> ```
> **Purpose:** Defines the application — what image, how many replicas, resource limits, health checks, update strategy.
>
> **service.yaml — HOW to reach it:**
> ```yaml
> apiVersion: v1
> kind: Service
> metadata:
>   name: payment-service
> spec:
>   selector:
>     app: payment                  # Routes to pods with this label
>   ports:
>   - port: 80                     # Service port (what callers use)
>     targetPort: 8080             # Pod port (where traffic goes)
>   type: ClusterIP                # Internal only
> ```
> **Purpose:** Provides stable networking — a fixed DNS name and IP that routes traffic to healthy pods regardless of which pod IPs change.
>
> **Why separate:**
> - Deployment = compute concern (what runs, how it scales)
> - Service = networking concern (how traffic reaches it)
> - Pods are ephemeral (IPs change). Service provides a stable endpoint.
> - You can update a Deployment (new image) without touching the Service. Networking stays stable."

---
---

# SECTION 38: Ansible Fundamentals

---

### Q: Why is Ansible used?

**Project Reference:** P9 (OS Patching — 500+ servers)

**Answer:**

> "Ansible is used for **configuration management and orchestration of existing servers** — things Terraform can't do.
>
> **Why Ansible specifically:**
>
> 1. **Agentless** — Uses SSH. No agent to install/maintain on 500+ servers. Just SSH access and Python (already on every Linux server).
>
> 2. **Idempotent** — Run it 10 times, same result. Safe to re-run. If package is already installed, it skips. This is critical for patching — re-running after a failure is safe.
>
> 3. **Declarative + Procedural** — Describe desired state (package: latest) but also support ordered steps (stop service → patch → validate → start service).
>
> 4. **Inventory-based** — Target groups of servers by role, environment, location. 'Patch all web servers in us-east-1 but not databases.'
>
> 5. **Human-readable** — YAML playbooks. Operations team can read and understand what automation does without programming knowledge.
>
> **Our use cases:**
> - OS patching (P9) — 500+ servers, 12 roles, 18-step lifecycle
> - Initial server configuration (post-AMI-launch baseline)
> - Application deployment to VMs (before we migrated to K8s)
> - Certificate rotation across fleet
>
> **Ansible vs Terraform:** Terraform creates infrastructure (VPC, EC2, RDS). Ansible configures what's ON the infrastructure (packages, services, files). They complement, not compete."

---

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

# SECTION 40: Docker & Kubernetes (Advanced)

---

### Q: What is an image pull policy in Kubernetes, and what values does it accept?

**Project Reference:** P3 (Kubernetes Platform), P1 (DevSecOps — image management)

**Answer:**

> "`imagePullPolicy` tells kubelet WHEN to pull the container image from the registry.
>
> | Value | Behavior | Use Case |
> |---|---|---|
> | **Always** | Pull image every time pod starts (checks for new digest) | Production with `latest` tag (we don't use `latest` but if you do) |
> | **IfNotPresent** | Pull only if image not cached on node | Default for tagged images (e.g., `v2.1.3`). Saves pull time. |
> | **Never** | Never pull — use only what's on the node | Air-gapped environments, pre-loaded images |
>
> **Default behavior (if not specified):**
> - Tag is `latest` → defaults to `Always`
> - Tag is specific (e.g., `v2.1`) → defaults to `IfNotPresent`
>
> **Our practice:**
> - We ALWAYS use specific image tags (never `latest`) — so `IfNotPresent` is the default
> - This is safe because our tags are immutable — `v2.1` always points to the same image digest
> - We use image digest pinning in production: `image: ecr.aws/app@sha256:abc123...` — guarantees exact image regardless of tag
>
> **Why `Always` can be a problem:** If registry is down and policy is `Always`, pods can't start even if image is cached locally. `IfNotPresent` with immutable tags is safer."

---

### Q: What is the difference between a Deployment and a DaemonSet?

**Project Reference:** P3 (Kubernetes Platform — monitoring stack)

**Answer:**

> | Aspect | Deployment | DaemonSet |
> |---|---|---|
> | **Purpose** | Run N replicas of an app | Run exactly ONE pod per node |
> | **Scheduling** | Scheduler decides which nodes | One on EVERY node (or filtered by nodeSelector) |
> | **Scaling** | Scales by changing replicas (HPA) | Scales automatically as nodes are added/removed |
> | **Use case** | Application workloads | Node-level agents (monitoring, logging, networking) |
> | **Example** | payment-service (3 replicas) | Fluentd (log collector), node-exporter (metrics), kube-proxy, Calico |
>
> **In our environment:**
> - **Deployments:** All 15+ microservices. Scaled by HPA, managed by ArgoCD.
> - **DaemonSets:**
>   - `fluent-bit` — collects logs from every node → ships to Elasticsearch
>   - `node-exporter` — exposes node metrics for Prometheus
>   - `aws-node` (VPC CNI) — manages pod networking on each node
>   - `kube-proxy` — iptables rules for service routing
>   - `falco` — runtime security monitoring on each node
>
> **Key difference in behavior:**
> - Add a new node → DaemonSet automatically runs its pod there (no action needed)
> - Add a new node → Deployments don't care unless scheduler places their pods there
> - DaemonSets ignore things like resource requests for scheduling (they MUST run on every node)"

---

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

# SECTION 42: Configuration Management

---

### Q: Tell me a couple of Ansible handlers you have used.

**Project Reference:** P9 (OS Patching — service restart handlers)

**Answer:**

> "Handlers are tasks that run ONLY when notified — typically for service restarts after config changes.
>
> **Handlers I use:**
>
> ```yaml
> handlers:
>   - name: restart httpd
>     ansible.builtin.service:
>       name: httpd
>       state: restarted
>
>   - name: reload nginx
>     ansible.builtin.service:
>       name: nginx
>       state: reloaded
>
>   - name: restart sshd
>     ansible.builtin.service:
>       name: sshd
>       state: restarted
>
>   - name: daemon-reload
>     ansible.builtin.systemd:
>       daemon_reload: yes
> ```
>
> **Usage in tasks:**
> ```yaml
> - name: Update sshd config
>   ansible.builtin.template:
>     src: sshd_config.j2
>     dest: /etc/ssh/sshd_config
>   notify: restart sshd       # Only restarts if file actually changed
> ```
>
> **Why handlers matter:**
> - Handler runs ONCE at the end of the play, even if notified multiple times (efficient — don't restart a service 5 times)
> - Only runs if the task actually CHANGED something (idempotent — if config is already correct, no unnecessary restart)
> - In our patching (P9): after updating packages, handler restarts affected services only if the package was actually updated"

---

### Q: What is the key differentiator between Ansible and other tools like Chef and Puppet?

**Project Reference:** P9 (OS Patching — chose Ansible)

**Answer:**

> "**Ansible is agentless.** That's the #1 differentiator.
>
> | Aspect | Ansible | Chef / Puppet |
> |---|---|---|
> | **Agent** | None — uses SSH | Requires agent on every node |
> | **Architecture** | Push-based (you trigger it) | Pull-based (agent polls server periodically) |
> | **Language** | YAML (simple, readable) | Ruby DSL (Chef) / Puppet DSL (steeper learning curve) |
> | **Setup effort** | Minimal — SSH + Python on target | Install agent, configure master server, certificates |
> | **State** | Stateless (no central DB of node state) | Stateful (master tracks node state) |
> | **Scaling** | Ansible AAP/Tower for enterprise | Chef Server / Puppet Master |
>
> **Why we chose Ansible (P9):**
> - 500+ servers already had SSH access — no agent deployment required
> - Operations team could read YAML playbooks immediately — no Ruby learning curve
> - Push-based fits our patching model (controlled trigger, not continuous enforcement)
> - Ansible Automation Platform (AAP) gave us enterprise features (RBAC, scheduling, audit, API)
>
> **When Chef/Puppet might be better:**
> - Continuous enforcement (desired state every 30 minutes) — large fleet that must ALWAYS match policy
> - Already have Chef/Puppet infrastructure invested
>
> For our use case (orchestrated patching, one-time configuration), push-based Ansible is the natural fit."

---
---

# SECTION 43: Kubernetes Core Concepts

---

### Q: What is the difference between a Pod and a Container?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "**Container** = a single running process in an isolated environment (namespaces + cgroups + rootfs). It's a Docker/containerd unit.
>
> **Pod** = Kubernetes' smallest deployable unit. A wrapper around one or MORE containers that share:
> - Same network namespace (same IP, same localhost, same ports)
> - Same storage volumes
> - Same lifecycle (start together, die together)
>
> **Why the distinction matters:**
>
> | Aspect | Container | Pod |
> |---|---|---|
> | **Scope** | Single process | Group of related containers |
> | **Networking** | Own network stack | SHARED network (containers in same pod talk via localhost) |
> | **Scheduling** | N/A | Scheduled together on same node |
> | **Use case** | One app process | Main app + sidecar (Envoy, log collector) |
>
> **Real example (P6 — Istio):**
> One Pod contains:
> - Container 1: `payment-service` (our app — port 8080)
> - Container 2: `envoy-proxy` (Istio sidecar — intercepts traffic)
>
> They share the same network namespace — Envoy can intercept traffic on `localhost` because they're in the same Pod. If they were separate Pods, this wouldn't work.
>
> **Rule:** One container per Pod (90% of cases). Multiple containers only for sidecar patterns (logging, proxying, init containers)."

---

### Q: What is the difference between a ConfigMap and a Secret?

**Project Reference:** P3 (Kubernetes), P1 (DevSecOps)

**Answer:**

> | Aspect | ConfigMap | Secret |
> |---|---|---|
> | **Purpose** | Non-sensitive configuration (app settings, feature flags) | Sensitive data (passwords, tokens, certificates) |
> | **Storage** | Plain text in etcd | Base64-encoded in etcd (+ encrypted at rest with KMS on EKS) |
> | **Access** | No special restrictions by default | Can restrict via RBAC (limit who can `get secrets`) |
> | **Size limit** | 1 MB | 1 MB |
> | **Mounted as** | Env vars or volume files | Env vars or volume files |
>
> **Example:**
> ```yaml
> # ConfigMap — non-sensitive
> apiVersion: v1
> kind: ConfigMap
> data:
>   LOG_LEVEL: \"info\"
>   MAX_RETRIES: \"3\"
>   APP_MODE: \"production\"
>
> # Secret — sensitive
> apiVersion: v1
> kind: Secret
> type: Opaque
> data:
>   DB_PASSWORD: cGFzc3dvcmQxMjM=    # base64 encoded
>   API_KEY: c2VjcmV0a2V5MTIz
> ```
>
> **Key misconception:** base64 is NOT encryption. Anyone who can read the Secret can decode it. Real security comes from:
> - RBAC (restrict who can `kubectl get secrets`)
> - Encryption at rest (KMS envelope encryption in EKS)
> - External Secrets Operator (secrets fetched from Secrets Manager, not stored in Git)"

---

### Q: What are Persistent Volumes (PV) and Persistent Volume Claims (PVC)?

**Project Reference:** P3 (Kubernetes — stateful workloads)

**Answer:**

> "**PV** = the actual storage resource (the disk). Provisioned by admin or dynamically by StorageClass.
>
> **PVC** = a request for storage by a pod. 'I need 10Gi of SSD storage.'
>
> **Analogy:** PV is a house. PVC is a rental application. Pod is the tenant.
>
> ```yaml
> # PVC — what the pod asks for
> apiVersion: v1
> kind: PersistentVolumeClaim
> metadata:
>   name: data-volume
> spec:
>   accessModes: [ReadWriteOnce]
>   resources:
>     requests:
>       storage: 10Gi
>   storageClassName: gp3
>
> # Pod uses the PVC
> spec:
>   volumes:
>   - name: data
>     persistentVolumeClaim:
>       claimName: data-volume
>   containers:
>   - volumeMounts:
>     - mountPath: /data
>       name: data
> ```
>
> **How it works on EKS:**
> 1. Pod requests PVC (10Gi, gp3)
> 2. StorageClass (EBS CSI driver) dynamically provisions an EBS gp3 volume
> 3. EBS volume attached to the node where pod is scheduled
> 4. Mounted into the container at `/data`
> 5. Pod deleted → PVC retained (data persists). New pod can claim same PVC.
>
> **Key facts:**
> - `ReadWriteOnce` = only one node can mount it (EBS limitation)
> - `ReadWriteMany` = multiple nodes (need EFS, not EBS)
> - `persistentVolumeReclaimPolicy: Retain` = data survives PVC deletion (safety for databases)"

---

### Q: How do you perform scaling over node groups in Kubernetes/EKS?

**Project Reference:** P3 (Kubernetes — Karpenter), P7 (Cost Optimization)

**Answer:**

> "Two approaches — we use Karpenter:
>
> **1. Managed Node Groups + Cluster Autoscaler (traditional):**
> - Define node group with min/max/desired
> - Cluster Autoscaler watches for Pending pods (pods that can't schedule due to no capacity)
> - Pending pods detected → Autoscaler adds nodes from the node group
> - Scale down: nodes with low utilization → drain pods → terminate node
>
> **2. Karpenter (what we use — better):**
> - No pre-defined node groups. Karpenter provisions RIGHT-SIZED instances on demand.
> - Pod goes Pending → Karpenter evaluates pod requirements (CPU, memory, GPU, architecture) → provisions the optimal instance type (could be m5.large, c5.xlarge, or even Spot)
> - Consolidation: detects underutilized nodes → moves pods → terminates extra nodes
>
> **Why Karpenter over Cluster Autoscaler:**
>
> | Aspect | Cluster Autoscaler | Karpenter |
> |---|---|---|
> | Instance types | Pre-defined in node group | Picks from 60+ types dynamically |
> | Speed | ~2 min to scale | ~30 seconds |
> | Efficiency | May over-provision (fixed instance type) | Right-sizes per workload |
> | Spot handling | Basic | Diversifies across instance families, handles interruptions |
> | Consolidation | No (leaves underutilized nodes) | Yes (bin-packs and removes waste) |
>
> **Result (P7):** Karpenter + Spot = 60-70% savings on worker node compute costs."

---

### Q: Scenario — You have an Nginx container running, but you cannot get into it. What are the probable scenarios?

**Project Reference:** P3 (Kubernetes troubleshooting)

**Answer:**

> "Assuming 'can't get into it' means `kubectl exec` fails or you can't access Nginx via its service:
>
> **Can't `kubectl exec` into the container:**
>
> 1. **No shell in image** — Minimal/distroless Nginx image has no `bash` or `sh`. Fix: `kubectl exec -it pod -- /bin/sh` (try sh, not bash). If neither exists → use ephemeral debug container: `kubectl debug -it pod --image=busybox`
>
> 2. **Container is crashing** — It's in CrashLoopBackOff. Can't exec into a crashed container. Check: `kubectl logs pod` or `kubectl logs pod --previous` for last crash output.
>
> 3. **Security policy blocking exec** — Pod Security Standard or Kyverno policy blocking `exec` to production pods (we do this). Check RBAC: do you have `pods/exec` permission?
>
> 4. **Read-only filesystem** — Container runs with `readOnlyRootFilesystem: true`. Exec works but you can't write/install anything inside.
>
> **Can't reach Nginx via network:**
>
> 5. **Pod is running but not Ready** — Readiness probe failing. Pod exists but removed from Service endpoints. Check: `kubectl get endpoints`
>
> 6. **Wrong port** — Nginx listening on 80, but Service targets port 8080. Check `containerPort` vs `targetPort`.
>
> 7. **NetworkPolicy blocking** — Default-deny ingress policy, no rule allowing traffic to Nginx pod.
>
> 8. **Nginx config error** — Nginx started but misconfigured (wrong `server_name`, `listen` directive). Check: `kubectl logs pod` for Nginx error logs.
>
> **My debugging order:** `kubectl describe pod` → `kubectl logs` → `kubectl get endpoints` → `kubectl exec` (if possible) → check NetworkPolicies."

---

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

# SECTION 45: Containers, Security & Misc

---

### Q: How do we ensure container state is maintained?

**Project Reference:** P3 (Kubernetes — StatefulSets + PVCs)

**Answer:**

> "Containers are **ephemeral by default** — when they die, everything inside is lost. To maintain state:
>
> **1. Persistent Volumes (PVC):**
> - Mount external storage (EBS via CSI driver) into the container
> - Container dies → new container mounts the SAME volume → data preserved
> - Used for databases (PostgreSQL, MongoDB) running in Kubernetes
>
> **2. StatefulSets (not Deployments):**
> - Each pod gets a stable identity (pod-0, pod-1, pod-2) and a dedicated PVC
> - Pods are recreated with the same name and reattach to their specific volume
> - Ordered startup/shutdown (important for clustered databases)
>
> **3. External state stores (preferred):**
> - Don't store state IN containers at all. Push state to managed services:
>   - Database → Aurora RDS (managed, outside K8s)
>   - Cache → ElastiCache Redis
>   - Files → S3
> - Container stays stateless = easy to scale, replace, deploy
>
> **Our approach:** Application containers are stateless (Deployments). State lives in Aurora/Redis/S3 (external). Only exception: monitoring stack (Prometheus uses StatefulSet + PVC for metrics storage)."

---

### Q: What are Docker volumes?

**Project Reference:** P1 (DevSecOps — Docker Compose), P3 (Kubernetes storage)

**Answer:**

> "Docker volumes are the mechanism to persist data beyond container lifecycle and share data between containers.
>
> **Three types:**
>
> | Type | Syntax | Use Case |
> |---|---|---|
> | **Named volume** | `docker run -v mydata:/app/data` | Persistent storage, managed by Docker. Survives container removal. |
> | **Bind mount** | `docker run -v /host/path:/container/path` | Mount host directory into container. For development (live code reload). |
> | **tmpfs** | `docker run --tmpfs /tmp` | In-memory only. Fast, not persisted. For sensitive temp data. |
>
> **Why volumes matter:**
> - Container filesystem is layered (overlay) and ephemeral — deleted when container is removed
> - Volumes exist OUTSIDE the container's filesystem — independent lifecycle
> - Multiple containers can share a volume (e.g., sidecar reading logs written by app)
>
> **In our Docker Compose (P1):**
> ```yaml
> services:
>   jenkins:
>     volumes:
>       - jenkins_data:/var/jenkins_home   # Named volume — persists Jenkins config
>   sonarqube:
>     volumes:
>       - sonar_data:/opt/sonarqube/data   # Persists SonarQube analysis data
> ```
>
> **In Kubernetes:** Docker volumes → PersistentVolumeClaims (PVC). Same concept, different abstraction layer."

---

### Q: Any GitLab experience?

**Project Reference:** P1 (CI/CD — primarily Jenkins, aware of GitLab)

**Answer:**

> "I'm familiar with GitLab CI but my primary CI/CD tool is Jenkins.
>
> **What I know about GitLab CI:**
> - `.gitlab-ci.yml` — pipeline as code (equivalent to Jenkinsfile)
> - Stages, jobs, artifacts, environments — similar concepts to Jenkins
> - Built-in container registry, package registry
> - Runners (shared/specific) — equivalent to Jenkins agents
> - Auto DevOps — pre-built pipeline templates
>
> **Comparison from my perspective:**
> | Jenkins | GitLab CI |
> |---|---|
> | Groovy (Jenkinsfile) | YAML (.gitlab-ci.yml) |
> | Plugin ecosystem | Built-in features |
> | Shared libraries | Include templates |
> | Freestyle + Pipeline | Jobs within stages |
>
> **Why we use Jenkins:** Complex enterprise pipelines, shared Groovy libraries, integration with Ansible AAP, and existing team expertise. But for a new project with GitLab as source control — GitLab CI is the natural choice (everything in one platform)."

---

### Q: What is 'egress'?

**Project Reference:** P3 (Kubernetes — NetworkPolicies), P2 (VPC — NAT Gateway)

**Answer:**

> "**Egress = outbound traffic** — traffic LEAVING a resource/network.
>
> **Opposite: Ingress = inbound traffic** — traffic COMING IN.
>
> | Context | Egress Means |
> |---|---|
> | **Security Group** | Outbound rules (what can the instance send OUT) |
> | **Kubernetes NetworkPolicy** | What destinations can a pod talk TO |
> | **VPC/NAT Gateway** | Internet-bound traffic from private subnet |
> | **Firewall** | Rules controlling outgoing connections |
>
> **Why egress matters for security:**
> - If an attacker compromises a container, they want to: exfiltrate data (egress to external IP) or download malware (egress to C2 server)
> - **Default-deny egress** in NetworkPolicies = compromised pod can't phone home
> - We restrict egress: pods can only reach specific external endpoints (DB, APIs they need). Everything else blocked.
>
> **Cost context:** AWS charges for egress data transfer ($0.09/GB to internet). VPC endpoints eliminate egress charges to AWS services (S3, DynamoDB)."

---

### Q: What is a Suricata rule?

**Project Reference:** No direct project — network security IDS/IPS knowledge

**Answer:**

> "**Suricata** is an open-source network threat detection engine — IDS/IPS (Intrusion Detection/Prevention System). It inspects network traffic in real-time.
>
> **A Suricata rule** defines what malicious traffic pattern to detect:
>
> ```
> alert http any any -> any any (msg:\"SQL Injection attempt\"; content:\"UNION SELECT\"; nocase; sid:1000001; rev:1;)
> ```
>
> **Rule breakdown:**
> - `alert` — action (alert, drop, reject, pass)
> - `http` — protocol
> - `any any -> any any` — source/dest IP and port
> - `content:\"UNION SELECT\"` — pattern to match in traffic
> - `msg` — alert description
> - `sid` — unique rule ID
>
> **AWS context:** AWS Network Firewall uses Suricata-compatible rules. You can write custom rules to:
> - Block known malicious IPs/domains
> - Detect SQL injection, XSS in HTTP traffic
> - Block specific TLS SNI patterns (command & control domains)
> - Alert on unusual outbound connections
>
> **In our setup:** We use AWS WAF (Layer 7) for web attacks and Network Firewall (Suricata-based, Layer 3-4) for network-level IDS/IPS at the VPC perimeter. I haven't written custom Suricata rules extensively, but understand the framework."

---

### Q: Where is your source code stored?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "**GitHub** (primary) — application code, Helm charts, Terraform modules, Ansible roles. All in private repositories under our organization.
>
> **Separate repos by concern:**
> - `app-repos/` — One repo per microservice (payment-service, order-service, etc.)
> - `infra-terraform/` — Terraform modules and environment configurations
> - `gitops-manifests/` — Helm values for ArgoCD (separate from app code — GitOps pattern)
> - `ansible-automation/` — Playbooks, roles, inventory for OS patching
>
> **Why separate repos:**
> - App code changes trigger app pipeline (build + deploy)
> - GitOps manifest changes trigger ArgoCD sync (deploy only — no rebuild)
> - Terraform changes trigger infra pipeline (plan + apply)
> - Different teams own different repos with appropriate RBAC
>
> Branch protection on all repos: no direct push to main, require PR + approval + passing CI checks."

---

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

# SECTION 48: Tool Preferences & Containerization

---

### Q: Which tool do you enjoy the most so far?

**Project Reference:** General — shows passion and depth

**Answer:**

> "**Terraform** — because it gives me the most architectural control.
>
> With Terraform, I'm not just running commands — I'm designing systems. Writing a module that defines how our entire VPC, EKS cluster, and security posture works is deeply satisfying. It's infrastructure as architecture, not infrastructure as tickets.
>
> Close second: **Kubernetes**. The declarative model, self-healing, and the richness of the ecosystem (Helm, ArgoCD, Karpenter, Istio) make it endlessly interesting. Every week I learn something new about K8s internals.
>
> **Why these two:** They're the tools where deep knowledge compounds. A surface-level Terraform user creates resources. A deep Terraform user designs reusable module libraries with security baked in, state isolation strategies, and CI/CD-driven workflows. Same with K8s — basic users deploy pods, advanced users understand scheduling, CNI, cgroups, and can troubleshoot at the kernel level.
>
> That depth is what I enjoy."

---

### Q: How do you secure your variables in Azure pipelines?

**Project Reference:** P1 (Jenkins — equivalent pattern), No direct Azure project

**Answer:**

> "I haven't used Azure DevOps pipelines directly — my primary CI tool is Jenkins. But the concept is identical:
>
> **In Azure DevOps (what I know):**
> - **Variable Groups** — store secrets centrally, link to multiple pipelines. Marked as 'secret' = masked in logs.
> - **Azure Key Vault integration** — pipeline fetches secrets from Key Vault at runtime. Secrets never stored in pipeline definition.
> - **Pipeline-level secrets** — set variables as 'secret' in UI or YAML. Masked in all log output.
>
> **Equivalent in our Jenkins setup:**
> - Jenkins Credentials Store = Azure Variable Groups
> - AWS Secrets Manager = Azure Key Vault
> - `credentials('id')` in Jenkinsfile = `$(secretVariable)` in Azure YAML
>
> **The principle is the same regardless of tool:**
> 1. Never hardcode secrets in pipeline code or repo
> 2. Use a vault/secret store as the source of truth
> 3. Fetch at runtime, never persist in logs
> 4. Rotate regularly, scope narrowly (per-pipeline access, not global)
>
> If this role uses Azure DevOps, I'd adapt quickly — the security principles transfer directly."

---

### Q: (Following incident scenario) What would be your starting first few things to investigate?

**Project Reference:** P3 (Observability), General — incident triage

**Answer:**

> "My first 3 checks in order (takes under 2 minutes):
>
> 1. **Monitoring dashboard (Grafana/CloudWatch)** — Is it truly down, or one user reporting? Check: endpoint health (Blackbox exporter), error rate, latency spike. This tells me WHAT is broken and WHEN it started.
>
> 2. **Recent changes** — What happened around the time the issue started?
>    - ArgoCD: Any deployment in last 2 hours?
>    - Terraform: Any infra change?
>    - CloudTrail: Any IAM/security change?
>    - If yes → correlates with issue → rollback candidate
>
> 3. **Basic health of components (top-down):**
>    - DNS resolving? (`dig app.example.com`)
>    - ALB healthy host count > 0?
>    - Pods running? (`kubectl get pods` — any CrashLoopBackOff?)
>    - DB reachable? (connection count, replication lag)
>    - Certificate expired? (sneaky — causes 'app not working' but infra looks fine)
>
> **This 2-minute checklist identifies 90% of issues.** Either it's a recent change (rollback), a crashed component (restart/scale), or an external dependency (DB, DNS, cert). Then I dig deeper into whichever bucket it falls into."

---

### Q: Did you containerize the app yourself, or did you find a Dockerfile somewhere?

**Project Reference:** P1 (DevSecOps — custom Dockerfile for Django app)

**Answer:**

> "**Wrote it myself.** Our Django application's Dockerfile was authored from scratch with production best practices:
>
> ```dockerfile
> # Multi-stage build
> FROM python:3.9-slim AS builder
> WORKDIR /app
> COPY requirements.txt .
> RUN pip install --no-cache-dir -r requirements.txt
>
> FROM python:3.9-slim
> WORKDIR /app
> # Non-root user
> RUN adduser --disabled-password --no-create-home appuser
> COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
> COPY . .
> USER appuser
> EXPOSE 8000
> CMD [\"gunicorn\", \"--bind\", \"0.0.0.0:8000\", \"--workers\", \"3\", \"app.wsgi:application\"]
> ```
>
> **Decisions I made:**
> - **Multi-stage** — build dependencies (gcc, dev headers) in builder stage, only runtime in final image. Image size: ~120MB not 800MB.
> - **Non-root** — `USER appuser` (UID 1001). Enforced by Kyverno in cluster.
> - **Slim base** — `python:3.9-slim` not full `python:3.9` (eliminates hundreds of unused packages = smaller attack surface)
> - **No cache** — `--no-cache-dir` reduces layer size
> - **Pinned version** — `python:3.9-slim` not `:latest`
>
> I don't copy Dockerfiles from the internet for production. I understand every line — because in a security audit, I need to justify why each package exists and why the base image is trusted."

---

---
---

# SECTION 49: Terraform Scenarios (Advanced)

---

*Note: "How do you handle joining a company with 100 manually-created EC2 instances?" and "What is the impact of changing the Terraform version?" have been moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)*

---

### Q: How do you manage a scenario where US users go to one deployment and Australian users go to a different one?

**Project Reference:** P8 (Multi-Region), P2 (Route53)

**Answer:**

> "**Route53 Geolocation Routing** (or Latency-based Routing).
>
> **Geolocation routing:**
> - US users → DNS resolves to US deployment (us-east-1 ALB)
> - Australian users → DNS resolves to Australian deployment (ap-southeast-2 ALB)
> - Rules based on geographic location of the DNS resolver
>
> ```
> Route53 Record: app.example.com
> ├── Geolocation: North America → ALB-us-east-1
> ├── Geolocation: Oceania → ALB-ap-southeast-2
> └── Geolocation: Default → ALB-us-east-1 (fallback)
> ```
>
> **Alternative: Latency-based routing:**
> - Route53 automatically sends users to the region with lowest latency
> - Don't need to define geographic rules — AWS measures latency to each endpoint
> - Better for 'best performance' use case vs 'data residency' use case
>
> **Implementation (Terraform):**
> ```hcl
> resource \"aws_route53_record\" \"us\" {
>   zone_id        = aws_route53_zone.main.zone_id
>   name           = \"app.example.com\"
>   type           = \"A\"
>   set_identifier = \"us\"
>   geolocation_routing_policy {
>     continent = \"NA\"
>   }
>   alias {
>     name    = aws_lb.us.dns_name
>     zone_id = aws_lb.us.zone_id
>   }
> }
> ```
>
> **When to use which:**
> - **Geolocation** → Data residency/compliance (EU data must stay in EU), regulatory requirements
> - **Latency-based** → Pure performance optimization (send users to fastest region)
> - **Failover** → DR (primary/secondary, switch only when primary is down — our P8 setup)
>
> **Key consideration:** Both regions need independent deployments (separate clusters, databases). For the database: Aurora Global Database gives read-local capability. Writes still go to primary region unless active-active (DynamoDB Global Tables)."

---

---
---

# SECTION 50: Cloud Recommendations & Identity

---

### Q: Do you classify yourself more as a developer or more as cloud operations?

**Project Reference:** General — self-positioning

**Answer:**

> "**Neither purely — I'm a Platform Engineer** who bridges both worlds.
>
> - I WRITE code: Terraform modules, Ansible roles, Python Lambda functions, Jenkinsfiles, Helm charts. That's development.
> - I OPERATE infrastructure: Kubernetes clusters, production deployments, incident response, patching. That's operations.
>
> If forced to pick: I lean **operations-heavy with strong development skills**. My value isn't writing application features — it's building the platform, pipelines, and automation that developers USE to ship features safely.
>
> At 12 years experience, the line blurs. I can read application code to troubleshoot it, I write infrastructure code daily, and I design systems that are self-healing. That's the DevOps sweet spot — not pure dev, not pure ops, but the engineer who makes both sides productive."

---

### Q: Can you give an example of the most complex or recent Lambda you have worked with?

**Project Reference:** P5 (Serverless Event-Driven — Security Remediation Engine)

**Answer:**

> "**Security auto-remediation engine (P5)** — not just a simple function, but an event-driven architecture:
>
> **What it does:** AWS Config detects a security violation (e.g., Security Group opened to 0.0.0.0/0) → EventBridge routes the event → Lambda identifies the violation type → applies the correct fix automatically → logs the action to DynamoDB → notifies via SNS.
>
> **Complexity:**
> - **Multiple violation types** — One Lambda handles 10+ different remediation actions (close SG, encrypt bucket, disable public access, remove overly permissive IAM policy). Uses a dispatch pattern based on event type.
> - **Idempotent** — If Lambda is triggered twice for same violation (EventBridge at-least-once delivery), it doesn't break. Checks current state before acting.
> - **Error handling** — DLQ (SQS) for failed remediations. Separate alerting Lambda processes DLQ items and notifies security team.
> - **Cross-account** — Lambda in security account assumes roles into workload accounts to remediate.
>
> **Result:** Violations auto-fixed in <90 seconds. $0.07/month operating cost. Zero human intervention for routine security issues."

---

### Q: What would be some of your recommendations (three things) to customers that are in the cloud?

**Project Reference:** General — cloud advisory

**Answer:**

> "Three things I always recommend:
>
> 1. **Automate everything from day one (IaC)** — Don't ClickOps and promise to codify later. You won't. Start with Terraform for infra, CI/CD for deployments. Manual changes = drift, outages, and unrepeatable environments. The earlier you adopt IaC, the cheaper it is.
>
> 2. **Security is not an afterthought** — Enable CloudTrail, GuardDuty, and SCPs on day one. Not after you get breached. Use IAM least-privilege (not `AdministratorAccess` for developers). Encrypt everything. It's 10x cheaper to build secure than to fix insecure later.
>
> 3. **Monitor cost from day one** — Set up billing alerts and budgets immediately. Enable Cost Explorer. Tag every resource. By month 3, untagged resources become orphans you're paying for but nobody owns. FinOps isn't a 'later' initiative — it's a 'now' habit."

---

### Q: What level of monitoring would you suggest for a production-grade application?

**Project Reference:** P3 (Observability stack)

**Answer:**

> "**Four pillars — all required for production:**
>
> 1. **Metrics** (Prometheus/CloudWatch) — CPU, memory, request rate, error rate, latency P50/P99. Set alerts on SLO thresholds.
>
> 2. **Logs** (EFK/CloudWatch Logs) — Structured JSON logs from all services. Searchable, correlated by request ID. Retention: 30 days hot, 1 year cold (S3).
>
> 3. **Traces** (Jaeger/X-Ray) — End-to-end request path across microservices. Essential for debugging latency and finding bottlenecks.
>
> 4. **Synthetic monitoring** (Blackbox exporter) — External probes hitting your endpoints every 30 seconds. Catches issues invisible to internal monitoring (DNS failure, cert expiry, ingress down).
>
> **Plus:**
> - **Alerting** with tiered routing (P1→PagerDuty, P2→Slack, P3→Jira)
> - **Dashboards** per service + one executive overview (SLO burn rate)
> - **On-call runbooks** linked to every alert (alert fires → engineer knows what to check)
>
> **Missing any one of these = blind spots.** Metrics without traces means you know SOMETHING is slow but not WHY. Traces without logs means you know WHERE it's slow but not WHAT happened."

---

### Q: Any dashboard you have created through log analysis?

**Project Reference:** P3 (Observability — Grafana + EFK)

**Answer:**

> "Yes, several:
>
> 1. **Application Error Dashboard (Grafana):**
>    - Log-based metric: count of `level=error` per service per 5-min window
>    - Parsed from EFK: top 10 error messages with frequency
>    - Trend: error rate over last 24 hours vs previous week
>    - Drill-down: click error → opens Kibana with exact log entries
>
> 2. **Deployment Tracking Dashboard:**
>    - Shows all deployments across environments (ArgoCD webhook → Grafana annotations)
>    - Error rate overlaid with deployment markers — immediately see 'error spike started right after deploy X'
>
> 3. **Security Events Dashboard:**
>    - Failed SSH attempts, unauthorized API calls (from CloudTrail logs)
>    - GuardDuty findings summary
>    - WAF blocked requests by rule
>
> **How I build them:** Logs → Fluentd parses and enriches → Elasticsearch indexes → Grafana/Kibana visualizes. For metrics extracted from logs, I use Fluentd's `prometheus` plugin to expose log-derived metrics directly to Prometheus."

---

### Q: What would you suggest to customers what they should NOT be doing in the cloud?

**Project Reference:** General — anti-patterns

**Answer:**

> "Top 5 things to STOP doing:
>
> 1. **Don't use root account for daily work** — Root account = God mode. No MFA recovery if compromised. Use IAM Identity Center (SSO), give roles with least privilege. Root should be locked in a safe with hardware MFA.
>
> 2. **Don't leave resources untagged** — Untagged = unowned = unknown cost = never cleaned up. We enforce tagging via SCPs — untagged resources get auto-terminated after 7 days (non-prod).
>
> 3. **Don't put everything in one account** — One account for Dev, Staging, Prod = one bad `terraform destroy` away from losing production. Separate accounts per environment.
>
> 4. **Don't hardcode credentials** — No access keys in code, no passwords in env files, no secrets in Git. Use IAM roles (IRSA, instance profiles), Secrets Manager, and short-lived tokens.
>
> 5. **Don't skip backups and DR testing** — 'It's in the cloud so it's safe' is a dangerous assumption. AWS doesn't backup your data automatically. Enable RDS automated backups, S3 versioning, and TEST your restore process quarterly."

---

### Q: How would you assign permissions to an EC2 instance to read an S3 bucket across accounts?

**Project Reference:** P4 (Landing Zone — cross-account access)

**Answer:**

> "**Cross-account S3 access from EC2 — two approaches:**
>
> **Approach 1: S3 Bucket Policy (resource-based):**
>
> Account A (EC2) needs to read S3 bucket in Account B:
>
> On Account B's S3 bucket, add bucket policy:
> ```json
> {
>   \"Effect\": \"Allow\",
>   \"Principal\": {\"AWS\": \"arn:aws:iam::AccountA:role/ec2-role\"},
>   \"Action\": [\"s3:GetObject\", \"s3:ListBucket\"],
>   \"Resource\": [\"arn:aws:s3:::bucket-name\", \"arn:aws:s3:::bucket-name/*\"]
> }
> ```
>
> EC2's instance role in Account A also needs `s3:GetObject` permission (both sides must allow).
>
> **Approach 2: Assume Role (identity-based):**
>
> 1. Create a role in Account B: `cross-account-s3-reader` with S3 read permissions + trust policy trusting Account A's EC2 role.
> 2. EC2's instance role in Account A gets `sts:AssumeRole` permission for Account B's role.
> 3. Application on EC2: `aws sts assume-role --role-arn arn:aws:iam::AccountB:role/cross-account-s3-reader` → gets temporary credentials → reads S3.
>
> **Which to use:**
> - Bucket policy (Approach 1) — simpler for single-bucket access
> - Assume Role (Approach 2) — better for multiple resources in Account B (one role, many permissions). More auditable.
>
> **Our preference (P4):** Assume Role. Clearer trust boundaries, logged in CloudTrail, works for any AWS service (not just S3)."

---

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

# SECTION 53: Kubernetes Scheduling & Observability (Deep Dive)

---

### Q: Explain node affinity, pod affinity/anti-affinity, taints & tolerations, topology spread constraints with production examples.

**Project Reference:** P3 (Kubernetes Platform — scheduling)

**Answer:**

> **Node Affinity:** Schedule on specific nodes (required=hard, preferred=soft). Example: 'only SSD nodes' or 'prefer us-east-1a'.
>
> **Pod Anti-Affinity:** Spread replicas. topologyKey=hostname (across nodes) or zone (across AZs). Example: 'don't put 2 payment pods on same node'.
>
> **Pod Affinity:** Co-locate. Example: 'put cache on same node as app for low latency'.
>
> **Taints & Tolerations:** Repel pods from nodes. Taint node (gpu=true:NoSchedule) → only pods with matching toleration schedule there. Use: GPU nodes, spot instances, dedicated team nodes.
>
> **Effects:** NoSchedule (block new), PreferNoSchedule (soft), NoExecute (evict existing).
>
> **Topology Spread:** Even distribution. maxSkew:1 + topologyKey:zone = perfectly spread across AZs.
>
> **In our setup:** We use pod anti-affinity to spread replicas across AZs (survive AZ failure), Karpenter uses taints for Spot nodes (only fault-tolerant workloads tolerate it), and topology spread ensures no AZ gets more than 1 extra pod vs others.

---

### Q: How to monitor K8s cluster and apps? Prometheus ServiceMonitor, kube-state-metrics, alerting strategy?

**Project Reference:** P3 (Observability stack — Prometheus/Grafana)

**Answer:**

> **Layers:**
> - Infrastructure → node_exporter (CPU, memory, disk per node)
> - Cluster → kube-state-metrics (pod phase, deployment replicas, restarts, node conditions)
> - Application → custom app metrics via ServiceMonitor
> - Autoscaling → metrics-server (for HPA + `kubectl top`)
>
> **metrics-server:** Lightweight, CPU/memory only, for HPA + kubectl top. NOT for alerting.
> **Prometheus:** Full monitoring, custom metrics, alerting, long-term storage.
> **kube-state-metrics:** Cluster-level objects (pod phase, deployment replicas, restarts, node conditions).
>
> **ServiceMonitor:** CRD that auto-configures Prometheus scraping by matching Service labels. No manual prometheus.yml editing. New service with correct labels → automatically scraped.
>
> **Alerting strategy:**
> - P1 (page): service down, node NotReady, error>5%
> - P2 (hours): high latency, disk>80%, HPA at max
> - P3 (ticket): pod restarts, memory trending up, cert expiring
>
> **Dashboards:** Cluster overview, namespace view, application RED (Rate/Errors/Duration), node resources, HPA status.

---

### Q: Explain pod states: ImagePullBackOff, Pending, Evicted, OOMKilled, CreateContainerConfigError. How to debug each?

**Project Reference:** P3 (Kubernetes troubleshooting)

**Answer:**

> | State | Cause | Debug | Fix |
> |---|---|---|---|
> | **ImagePullBackOff** | Can't pull image (typo, no imagePullSecret, DockerHub rate limit, network) | `kubectl describe pod` → Events | Check image name, add secret, use private registry/mirror |
> | **Pending** | Can't schedule (no resources, selector mismatch, taint, PVC unbound) | `kubectl describe pod` → Events (FailedScheduling) | Scale nodes, fix selectors, add toleration, check StorageClass |
> | **Evicted** | Kubelet kicked pod (disk/memory/ephemeral storage pressure) | `kubectl describe node` → Conditions | Clean images, fix leaks, increase disk, set resource requests |
> | **OOMKilled** | Exceeded memory limit (cgroup killed it) | `kubectl describe pod` → Last State: OOMKilled | Increase limit, fix memory leak, set -Xmx for JVM |
> | **CreateContainerConfigError** | Missing ConfigMap or Secret referenced by pod | `kubectl describe pod` → Events | Create the missing resource in same namespace |
>
> **Debug tree:**
> - Pending → check Events section for scheduling reason
> - ImagePull → describe pod, verify image exists and credentials
> - CrashLoop → `kubectl logs --previous` for crash output
> - OOM → check `kubectl top pods`, increase memory limits
> - Evicted → check node conditions, clean up disk

---

### Q: What metrics are you monitoring with Prometheus and Grafana, and did you configure them yourself?

**Project Reference:** P3 (Observability — full stack ownership)

**Answer:**

> "Yes, configured the full stack myself. Deployed via `kube-prometheus-stack` Helm chart (Prometheus Operator, AlertManager, Grafana, node-exporter, kube-state-metrics). Custom ServiceMonitors for application metrics. Grafana dashboards provisioned as code (JSON in Git). AlertManager routing: PagerDuty (P1), Slack (P2), Jira (P3).
>
> **What I monitor:**
>
> **Infrastructure (USE method):**
> - CPU utilization, memory usage, disk I/O, network throughput per node
> - Node conditions (pressure, NotReady), kubelet health, API server latency
>
> **Kubernetes:**
> - Pod restarts, pending pods, OOMKilled count, CrashLoopBackOff
> - HPA current vs desired replicas
> - PVC usage %, node allocatable vs requested
> - Deployment rollout status
>
> **Application (RED method):**
> - Rate: `http_requests_total` (requests per second)
> - Errors: `http_requests_total{status=~'5..'}` (error rate)
> - Duration: `http_request_duration_seconds` (P50, P95, P99)
> - Custom business metrics (orders/min, queue depth)
>
> **Key principle:** RED for services, USE for infrastructure. Alert on symptoms (user impact), not causes. Dashboard per team/namespace."

---
---

# SECTION 54: CI/CD Metrics & Ansible Advanced

---

### Q: CI/CD metrics — DORA metrics? How to identify and fix pipeline bottlenecks?

**Project Reference:** P1 (DevSecOps Pipeline)

**Answer:**

> "DORA metrics measure DevOps performance:
>
> | Metric | Elite | Our Status |
> |---|---|---|
> | **Deployment Frequency** | Multiple/day | Multiple/day (per service via ArgoCD) |
> | **Lead Time for Changes** | <1 hour | ~30 min (commit to prod for small changes) |
> | **Change Failure Rate** | <5% | ~3% (canary catches most before full rollout) |
> | **MTTR** | <1 hour | <5 min (auto-rollback via Argo Rollouts) |
>
> **Pipeline metrics I also track:**
> - Build success rate (target >95%)
> - Pipeline duration (target <10 min for PR feedback)
> - Test pass rate and flaky test ratio
> - Queue wait time (agents available?)
>
> **Finding bottlenecks:** Profile each stage timing. Common fixes:
> - Dependencies slow → cache (Maven .m2, npm node_modules, Docker layers)
> - Tests slow → parallelize + only run affected tests
> - Docker build slow → multi-stage + layer caching
> - Deploy slow → parallel environments
> - Flaky tests → quarantine, track 30 days, fix or remove
>
> **Key principle:** Measure first, optimize the slowest stage, track continuously."

---

### Q: What are Execution Environments (EE) in Ansible? Why introduced? How to build and use?

**Project Reference:** P9 (OS Patching — Ansible AAP)

**Answer:**

> "EE = containerized images packaging ansible-core + Python deps + collections + system libs into a portable container.
>
> **Why introduced:** 'Works on my machine' problem. Different engineers have different Python versions, different collection versions, different system libraries. On AWX, dependency conflicts between jobs were a nightmare.
>
> **How it works:**
> - Same EE image runs everywhere: laptop, CI pipeline, AWX. Identical environment.
> - Built with `ansible-builder`:
>
> ```yaml
> # execution-environment.yml
> version: 3
> dependencies:
>   galaxy: requirements.yml    # collections
>   python: requirements.txt    # Python packages
>   system: bindep.txt          # system packages (gcc, etc.)
> images:
>   base_image: quay.io/ansible/ansible-runner:latest
> ```
>
> - Build: `ansible-builder build --tag my-ee:1.0.0`
> - Push to registry → use in AWX or ansible-navigator
>
> **Usage:**
> ```bash
> ansible-navigator run deploy.yml --execution-environment-image my-ee:1.0.0
> ```
>
> **In AWX/AAP:** Settings → Execution Environments → assign to Job Template. Each job runs in that container.
>
> **Our practice:** Version-tagged EEs (my-ee:2.3.0). CI builds new EE on collection/dependency change. AWX points to specific version. Reproducible across all environments."

---

### Q: Difference between include_* and import_* in Ansible? Gotchas?

**Project Reference:** P9 (OS Patching — complex playbook architecture)

**Answer:**

> | Aspect | `import_*` (static) | `include_*` (dynamic) |
> |---|---|---|
> | **When parsed** | At playbook LOAD time | At RUNTIME (when reached) |
> | **`when:` behavior** | Applies to EACH task inside | Decides whether to include AT ALL |
> | **Tags** | Flow into child tasks ✓ | Do NOT propagate ✗ |
> | **Variable in filename** | Cannot use | CAN use (`include_tasks: "{{ os }}.yml"`) |
> | **Looping** | Cannot loop | CAN loop |
> | **Visibility** | `--list-tasks` shows all tasks | Only shows include line |
>
> **Gotchas:**
> 1. **Tags don't propagate into `include_tasks`** — If you run `--tags deploy` and your deploy tasks are inside an `include_tasks`, they won't run. Use `import_tasks` for tag-based execution.
> 2. **Handlers in includes may not be visible** — If a handler is defined inside `include_tasks`, it might not be found for `notify`. Use `import_tasks` for files with handlers.
> 3. **Variable filenames only with include** — `import_tasks: "{{ ansible_os_family }}.yml"` fails. Must use `include_tasks`.
>
> **Rules of thumb:**
> - Variable filename → `include_tasks`
> - Tags must propagate → `import_tasks`
> - Loop over files → `include_tasks`
> - Default/simple → `import_tasks` (safer, more predictable)
>
> In our P9 patching roles, we use `import_tasks` for the main flow (tags work correctly for running specific steps) and `include_tasks` only when loading OS-specific task files dynamically."

---
