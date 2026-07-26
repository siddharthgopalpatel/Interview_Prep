# SIDDHARTH PATEL

## Senior DevOps & Cloud Engineer

📧 Siddharthgopalpatel825@gmail.com | 📱 +91-7899449988 | 📍 [New Delhi, India]
🔗 linkedin.com/in/[your-linkedin] | 💻 github.com/siddharthpatel1993 

---

## Professional Summary

Senior DevOps & Cloud Engineer with 10+ years of experience designing and operating production-grade infrastructure at scale. Built enterprise CI/CD platforms (18-stage DevSecOps pipeline with canary deployments and automated rollback), multi-account AWS Landing Zones (15 accounts with SCPs, OIDC, SSO), and Kubernetes platforms (EKS, self-managed, OpenShift) supporting 500+ servers and 15+ production microservices. Reduced deployment failures by 95%, cloud costs by 35% ($180K/year saved), and achieved <5-minute disaster recovery RTO with <1-second RPO. Passionate about platform engineering, AI-driven observability, and building self-healing systems that scale without 3 AM pages.

---

## Technical Skills

| Domain | Technologies |
|---|---|
| **Cloud & Platforms** | AWS (VPC, EKS, ALB, RDS Aurora, Lambda, Route53, CloudFront, IAM, Organizations, SCPs) |
| **Containers & Orchestration** | Kubernetes, AWS EKS, OpenShift, Docker, Helm, Kustomize, Karpenter, Argo Rollouts |
| **Infrastructure as Code** | Terraform (modules, state, workspaces), Ansible (AAP, roles, vault), CloudFormation |
| **CI/CD & GitOps** | Jenkins, ArgoCD, GitHub Actions, GitLab CI, Cosign, Kyverno |
| **Security & Compliance** | DevSecOps, Trivy, Snyk, SonarQube, WAF, IRSA, SCPs, mTLS (Istio), Vault |
| **Monitoring & Observability** | Prometheus, Grafana, AlertManager, CloudWatch, EFK/ELK, Jaeger, X-Ray, Kubecost, AI-Driven Anomaly Detection |
| **Networking** | Istio Service Mesh, Calico, VPC CNI, Route53, ALB/NLB, NetworkPolicies |
| **Scripting & Languages** | Python, Bash, Groovy (Jenkinsfile), HCL (Terraform), YAML |
| **Practices** | Platform Engineering, DevSecOps, GitOps, FinOps, SRE, ITIL, Canary Deployments, Zero-Trust |

---

## Professional Experience

### Senior DevOps Engineer | Ericsson Global India Private Limited | Noida, India | June 2025 – Present

Led platform engineering initiatives across a 10+ person cross-functional team spanning DevOps, Security, and Development — driving infrastructure automation, security-first CI/CD, and self-healing systems for enterprise telecom platform.

**DevSecOps CI/CD Platform (Project 1):**
- Designed 18-stage DevSecOps CI/CD pipeline (Jenkins + ArgoCD + Argo Rollouts) with 6 security layers (secret scanning, SCA, SAST, container scanning, image signing, DAST) — reduced deployment blast radius from 100% to 5% with Prometheus-driven canary auto-rollback in 2 minutes
- Implemented supply chain security with Cosign image signing + Kyverno admission control — only signed, scanned images from trusted registry can run in production cluster

**Multi-Account AWS Landing Zone (Project 4):**
- Architected multi-account AWS Landing Zone (15 accounts, 5 OUs) with SCPs, OIDC federation for CI/CD, and IAM Identity Center (SSO) — passed SOC2 audit first attempt, reduced account provisioning from 2 weeks to 30 minutes

**Multi-Region Disaster Recovery (Project 8):**
- Implemented multi-region active-passive DR (Route53 failover + Aurora Global Database + S3 CRR) — achieved 3-minute RTO and <1-second RPO, validated quarterly through automated DR drills using AWS FIS

**Service Mesh & Zero-Trust (Project 6):**
- Implemented Istio service mesh with strict mTLS, zero-trust AuthorizationPolicies, and Jaeger distributed tracing across 15 microservices — reduced mean time to root cause (MTTR) from 2+ hours to under 5 minutes

**Serverless Security Automation (Project 5):**
- Built serverless security remediation engine (EventBridge + Lambda + SQS + DynamoDB) — auto-fixes AWS Config violations within 90 seconds, zero human intervention, $0.07/month operating cost

---

### DevOps Engineer | Ericsson Inc | Dallas, US | Dec 2018 – June 2025

**3-Tier AWS Architecture (Project 2):**
- Designed production 3-tier architecture (VPC across 3 AZs, ALB + WAF + CloudFront, EC2 ASG, Aurora Multi-AZ) with Terraform modules — handles 5000 req/s with auto-scaling 3→20 instances and <30-second database failover
- Implemented comprehensive IaC scanning (tfsec + Checkov + OPA) in CI pipeline — zero misconfigurations reach production, cost impact visible on every Terraform PR via Infracost

**Kubernetes Platform (Project 3):**
- Operated Kubernetes across 3 environments: self-managed (kubeadm + Calico + Ansible bootstrap), EKS (Karpenter + IRSA + VPC CNI with prefix delegation), and OpenShift (SCCs + built-in monitoring) — serving 15+ microservices in production
- Deployed full observability stack (Prometheus + Grafana + AlertManager + EFK) on Kubernetes with ServiceMonitor auto-discovery and structured alerting (P1→PagerDuty, P2→Slack, P3→Jira)

**Enterprise OS Patching (Project 9):**
- Built enterprise OS patching automation (18-step zero-touch lifecycle) on Ansible Automation Platform — patches 500+ RHEL servers/month with zero downtime via ALB traffic drain (serial 20%), 7-dimension automated validation, and ServiceNow ITIL integration (CR open → close)
- Developed 12 production-ready Ansible roles (connectivity, pre_backup, patch, integrity_check, cert_check, post_connectivity, notification) — each independently testable, idempotent, and rerunnable

**Cost Optimization (Project 7):**
- Reduced AWS cloud spend by 35% ($180K/year) through FinOps automation: non-prod auto-stop (65% compute savings), Karpenter node consolidation, VPA rightsizing, VPC endpoints (eliminated NAT transfer costs), and Compute Savings Plans
- Implemented Kubecost for per-namespace cost visibility with team-level chargeback reporting and mandatory tagging enforcement via SCPs

---

### System/Linux Administrator | Ericsson Global India Private Limited | Bangalore, India | Aug 2015 – Dec 2018

- Administered 100+ Linux servers (RHEL/CentOS) — patching, security hardening (CIS benchmarks), user management, and performance tuning
- Automated routine operations with Bash/Python scripts and early Ansible adoption — reduced manual SSH operations by 80%, established foundation for infrastructure-as-code practices
- Managed CI/CD pipelines (Jenkins) for 20+ services with automated build, test, and deployment to staging/production environments
- Implemented centralized monitoring (Nagios → Prometheus migration) and logging (ELK Stack) — provided first infrastructure visibility across dev, staging, and production
- Managed backup strategies, disaster recovery procedures, and incident response for production infrastructure

---

## Key Achievements

- **95% reduction in deployment blast radius** — 18-stage DevSecOps pipeline with Prometheus-driven canary auto-rollback (5% exposure vs 100% previously)
- **35% AWS cost reduction ($180K/year saved)** — automated FinOps: non-prod auto-stop, Karpenter consolidation, VPA rightsizing, Savings Plans
- **3-minute disaster recovery RTO** — multi-region active-passive architecture with Route53 failover + Aurora Global, validated quarterly
- **500+ servers patched/month with zero downtime** — 18-step automated lifecycle, 7-dimension validation, zero human intervention
- **MTTR reduced from 2+ hours to under 5 minutes** — Istio distributed tracing (Jaeger) + structured observability across 15 microservices
- **SOC2 audit passed first attempt** — multi-account Landing Zone with SCPs, OIDC, centralized logging, automated compliance

---

## Key Projects (with code)

| Project | Highlights | Link |
|---|---|---|
| **DevSecOps CI/CD Pipeline** | 18 stages, 6 security layers, canary + auto-rollback, GitOps (ArgoCD) | github.com/siddharthpatel1993/dcp_devsecops |
| **Enterprise OS Patching** | 18-step lifecycle, ServiceNow ITIL, 7-dimension validation, AAP | gitlab.com/patelsiddharthnids993/ansible_os_automation |
| **Full DevOps Platform (IaC)** | Terraform + Ansible + kubeadm K8s + Prometheus stack from scratch | gitlab.com/patelsiddharthnids993/devopsproject |

---

## Certifications

- CKA: Certified Kubernetes Administrator (Linux Foundation)
- CKAD: Certified Kubernetes Application Developer (Linux Foundation)
- AWS Solutions Architect – Associate (Amazon Web Services)
- Certified SAFe® 5 DevOps Practitioner (Scaled Agile)
- Delivery Champion — Kubernetes 2022
- MANA ADM Champion — DevSecOps 2023-2024

---

## Professional Contributions

- **Knowledge Sharing:** Delivered internal tech talks on DevSecOps pipelines, Kubernetes platform design, and Infrastructure as Code best practices
- **Documentation:** Created comprehensive project documentation and architecture guides for team onboarding and knowledge transfer
- **Mentoring:** Guided junior engineers through CKA certification preparation and Ansible/Terraform adoption
- **Open Source:** 3 public repositories with production-ready CI/CD pipelines, Ansible automation, and K8s infrastructure (GitHub + GitLab)

---

## Education

**Bachelor of Engineering** (Computer Science / IT) | [University Name] | [Year]

---

---

## CUSTOMIZATION NOTES (Remove this section before sending)

### How to customize per job application:

**If job emphasizes CI/CD + Security:**
→ Move DevSecOps bullet points to top of experience section

**If job emphasizes AWS + Infrastructure:**
→ Move Landing Zone + 3-Tier + DR bullet points to top

**If job emphasizes Kubernetes:**
→ Move K8s platform + Istio bullet points to top

**If job emphasizes Ansible + Operations:**
→ Move OS Patching automation to top

**If job emphasizes Cost/FinOps:**
→ Move cost optimization bullet point higher + expand with specific numbers

### Keywords to mirror from job descriptions:
- If JD says "Platform Engineering" → add to summary
- If JD says "SRE" → change title to "Staff SRE / DevOps Engineer"
- If JD says "Go/Golang" → mention if you have any Go experience
- If JD says "GCP/Azure" → mention multi-cloud awareness from Project 3

### Numbers to remember (use in cover letter / interviews):
- 18-stage pipeline, 6 security layers
- 15 AWS accounts, 5 OUs
- 35% cost reduction = $180K/year
- 3-minute RTO, <1-second RPO
- 500+ servers patched/month
- 5000 req/s handled
- 99.95% uptime
- 90-second auto-remediation
- 2-minute canary rollback
