# Docker Containers — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 4, 40, 45, 48

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

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

