# CICD Jenkins — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 2, 54

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

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
