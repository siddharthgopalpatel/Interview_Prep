# Infrastructure Scripting Monitoring — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 20

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

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

