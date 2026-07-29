# SIDDHARTH PATEL

## Senior DevOps & Cloud Engineer

📧 Siddharthgopalpatel825@gmail.com | 📱 +91-7899449988 | 📍 New Delhi, India
🔗 https://www.linkedin.com/in/siddharth-patel-1b090169/ | 💻 github.com/siddharthpatel1993

---

## Professional Summary

Senior DevOps & Cloud Infrastructure Engineer with 10+ years of experience building secure, scalable, and automated cloud infrastructure with embedded DevSecOps practices across the SDLC. Expert in Terraform-driven IaC (modules, state management, multi-environment), Kubernetes platform operations (EKS, self-managed, OpenShift — transferable to AKS), and CI/CD pipeline engineering with integrated security controls (SAST, SCA, container scanning, image signing). Delivered enterprise platforms supporting 500+ servers and 15+ production microservices with 99.95% uptime. Reduced deployment failures by 95%, cloud costs by 35% ($180K/year), and achieved <5-minute disaster recovery RTO. Strong alignment with CIS, NIST, and ISO 27001 compliance frameworks. Passionate about platform engineering, security automation, and building governed, self-healing cloud-native systems.

---

## Technical Skills

| Domain | Technologies |
|---|---|
| **Cloud Platforms** | AWS (VPC, EKS, ALB, RDS Aurora, Lambda, Route53, CloudFront, IAM, Organizations, SCPs), Azure (AKS, Azure Defender, Azure DevOps — working knowledge), Multi-Cloud Architecture |
| **Infrastructure as Code** | Terraform (modules, state management, workspaces, tfsec, Checkov, OPA/Conftest, Infracost), Ansible (AAP, roles, vault), CloudFormation |
| **Containers & Kubernetes** | Kubernetes (EKS, self-managed, OpenShift, AKS-transferable), Docker, Helm Charts, Kustomize, Karpenter, Argo Rollouts |
| **CI/CD & GitOps** | Jenkins, ArgoCD, GitHub Actions, GitLab CI, Azure DevOps Pipelines, Cosign, Kyverno |
| **Security & Compliance** | DevSecOps, Trivy, Snyk, SonarQube, WAF, IRSA, SCPs, mTLS (Istio), Vault, CIS Benchmarks, NIST, ISO 27001, CSPM, Threat Modelling |
| **Monitoring & Observability** | Prometheus, Grafana, AlertManager, CloudWatch, EFK/ELK, Jaeger, X-Ray, Kubecost, App Dynamics (familiar) |
| **Networking** | VPCs, Subnets, VPNs, Firewalls, Security Groups, NACLs, Istio Service Mesh, Calico, VPC CNI, Route53, ALB/NLB, NetworkPolicies |
| **Scripting & Languages** | Python, Bash, PowerShell, Groovy (Jenkinsfile), HCL (Terraform), YAML |
| **Practices & Frameworks** | Platform Engineering, DevSecOps, GitOps, FinOps, SRE, ITIL, Zero-Trust Security, Canary Deployments, Policy-as-Code |

---

## Professional Experience

### Senior DevOps Engineer | Ericsson Global India Private Limited | Noida, India | June 2025 – Present

Led platform engineering initiatives across a 10+ person cross-functional team spanning DevOps, Security, and Development — driving infrastructure automation, security-first CI/CD, and self-healing systems for enterprise telecom platform.

**Infrastructure & Automation:**

- Built production 3-tier infrastructure (VPC across 3 AZs, ALB + WAF + CloudFront, EC2 ASG, Aurora Multi-AZ) entirely with Terraform modules — handles 5,000 req/s with auto-scaling 3→20 instances and <30-second database failover
- Implemented comprehensive IaC security scanning (tfsec + Checkov + OPA/Conftest) integrated in CI pipeline — zero misconfigurations reach production, cost impact visible on every Terraform PR via Infracost
- Architected multi-account AWS Landing Zone (15 accounts, 5 OUs) with Terraform — SCPs, OIDC federation for CI/CD, IAM Identity Center (SSO), account provisioning reduced from 2 weeks to 30 minutes

**Kubernetes & Helm Deployments:**

- Operated Kubernetes platforms across 3 environments: self-managed (kubeadm + Calico + Ansible bootstrap), EKS (Karpenter + IRSA + VPC CNI), and OpenShift (SCCs + built-in monitoring) — serving 15+ microservices in production with standardized Helm chart-based deployments
- Deployed and managed Kubernetes workloads using Helm charts for repeatable, version-controlled application delivery with automated rollbacks and environment-specific value overrides
- Implemented Karpenter for intelligent node autoscaling with Spot instance optimization — achieving 60-70% savings on worker nodes while maintaining performance SLAs

**Security & Compliance (DevSecOps):**

- Designed 18-stage DevSecOps CI/CD pipeline (Jenkins + ArgoCD + Argo Rollouts) with 6 integrated security layers (secret scanning, SCA, SAST, container scanning, Cosign image signing, DAST) — reduced deployment blast radius from 100% to 5% with Prometheus-driven canary auto-rollback
- Implemented supply chain security with Cosign image signing + Kyverno admission control — only signed, scanned images from trusted registries can run in production (SOC2 audit passed first attempt)
- Built serverless security remediation engine (EventBridge + Lambda + SQS + DynamoDB) — auto-fixes AWS Config/CIS violations within 90 seconds, zero human intervention
- Enforced security policies aligned with CIS benchmarks, NIST framework — Pod Security Standards (restricted profile), NetworkPolicies (default-deny), mTLS enforcement via Istio, RBAC least-privilege

**Monitoring & Incident Response:**

- Deployed full observability stack (Prometheus + Grafana + AlertManager + EFK) on Kubernetes with ServiceMonitor auto-discovery and structured alerting (P1→PagerDuty, P2→Slack, P3→Jira)
- Implemented Istio service mesh with strict mTLS, zero-trust AuthorizationPolicies, and Jaeger distributed tracing across 15 microservices — reduced MTTR from 2+ hours to under 5 minutes
- Implemented multi-region active-passive DR (Route53 failover + Aurora Global Database + S3 CRR) — achieved 3-minute RTO and <1-second RPO, validated quarterly through automated DR drills

**Collaboration & Documentation:**

- Documented infrastructure designs, Terraform module specifications, and CI/CD pipeline processes — enabling team onboarding and cross-functional collaboration
- Created architecture decision records (ADRs) and runbooks for incident response, DR procedures, and platform operations

---

### DevOps Engineer | Ericsson Inc | Dallas, US | Dec 2018 – June 2025

**Infrastructure Automation & IaC:**

- Designed and maintained Terraform module library (VPC, EKS, RDS, IAM) for multi-region, multi-account deployments — reusable across all environments with separate state per account, CI/CD-driven plan/apply workflow
- Built enterprise OS patching automation (18-step zero-touch lifecycle) on Ansible Automation Platform — patches 500+ RHEL servers/month with zero downtime via ALB traffic drain, 7-dimension automated validation, and ServiceNow ITIL integration
- Developed 12 production-ready Ansible roles (connectivity, pre_backup, patch, integrity_check, cert_check, post_connectivity, notification) — each independently testable, idempotent, and rerunnable

**Cost Optimization & Performance:**

- Reduced AWS cloud spend by 35% ($180K/year) through FinOps automation: non-prod auto-stop (65% compute savings), Karpenter node consolidation, VPA rightsizing, VPC endpoints, and Compute Savings Plans
- Implemented Kubecost for per-namespace cost visibility with team-level chargeback reporting and mandatory tagging enforcement via SCPs

**Security & Governance:**

- Integrated tfsec, Checkov, and OPA in Terraform CI pipeline — blocking non-compliant infrastructure at PR stage (open Security Groups, unencrypted storage, overly permissive IAM)
- Managed IAM strategy with least-privilege principles: IRSA for pod-level access, OIDC federation for CI/CD, short-lived credentials, Access Analyzer for permission auditing

---

### System/Linux Administrator | Ericsson Global India Private Limited | Bangalore, India | Aug 2015 – Dec 2018

- Administered 100+ Linux servers (RHEL/CentOS) — patching, security hardening (CIS benchmarks), user management, and performance tuning
- Automated routine operations with Bash/Python scripts and early Ansible adoption — reduced manual SSH operations by 80%, established foundation for infrastructure-as-code practices
- Managed CI/CD pipelines (Jenkins) for 20+ services with automated build, test, and deployment to staging/production environments
- Implemented centralized monitoring (Nagios → Prometheus migration) and logging (ELK Stack) — first infrastructure-wide observability across dev, staging, and production
- Managed backup strategies, disaster recovery procedures, and incident response for production infrastructure

---

## Key Achievements

| Achievement | Impact |
|---|---|
| **95% reduction in deployment blast radius** | 18-stage DevSecOps pipeline with canary auto-rollback (5% vs 100% exposure) |
| **Zero IaC misconfigurations in production** | tfsec + Checkov + OPA integrated in Terraform CI pipeline |
| **35% AWS cost reduction ($180K/year)** | Automated FinOps: auto-stop, Karpenter consolidation, rightsizing, Savings Plans |
| **3-minute disaster recovery RTO** | Multi-region active-passive architecture, validated quarterly |
| **500+ servers patched/month zero downtime** | 18-step automated lifecycle, 7-dimension validation, ITIL integration |
| **MTTR reduced from 2+ hours to <5 minutes** | Istio distributed tracing (Jaeger) + structured observability |
| **SOC2 audit passed first attempt** | Multi-account Landing Zone with SCPs, OIDC, centralized logging |
| **90-second auto-remediation** | Serverless engine auto-fixes CIS/security violations without human intervention |

---

## Key Projects (with code)

| Project | Highlights | Link |
|---|---|---|
| **DevSecOps CI/CD Pipeline** | 18 stages, 6 security layers, canary + auto-rollback, GitOps (ArgoCD) | github.com/siddharthpatel1993/dcp_devsecops |
| **Enterprise OS Patching** | 18-step lifecycle, ServiceNow ITIL, 7-dimension validation, AAP | gitlab.com/patelsiddharthnids993/ansible_os_automation |
| **Full DevOps Platform (IaC)** | Terraform + Ansible + kubeadm K8s + Prometheus stack from scratch | gitlab.com/patelsiddharthnids993/devopsproject |

---

## Certifications

- **CKA:** Certified Kubernetes Administrator (Linux Foundation)
- **CKAD:** Certified Kubernetes Application Developer (Linux Foundation)
- **AWS Solutions Architect – Associate** (Amazon Web Services)
- **Certified SAFe® 5 DevOps Practitioner** (Scaled Agile)
- Delivery Champion — Kubernetes 2022
- MANA ADM Champion — DevSecOps 2023-2024

---

## Professional Contributions

- **Knowledge Sharing:** Delivered internal tech talks on DevSecOps pipelines, Kubernetes platform design, Terraform modules, and security automation
- **Documentation:** Created comprehensive project documentation, architecture guides, and runbooks for team onboarding
- **Mentoring:** Guided junior engineers through CKA certification preparation and Ansible/Terraform adoption
- **Open Source:** 3 public repositories with production-ready CI/CD pipelines, Ansible automation, and K8s infrastructure
- **Continuous Learning:** Active in Kubernetes, security tooling, and platform engineering communities; exploring Agentic AI applications in DevOps

---

## Education

**Bachelor of Engineering** (Computer Science / IT) | KIIT University | 2015
