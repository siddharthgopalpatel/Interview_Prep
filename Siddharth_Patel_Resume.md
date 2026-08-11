# SIDDHARTH PATEL
**Senior DevOps & Cloud Infrastructure Engineer**

siddharthgopalpatel825@gmail.com | +91-7899449988 | New Delhi, India | [LinkedIn](https://www.linkedin.com/in/siddharth-patel-1b090169/) | [GitHub](https://github.com/siddharthpatel1993)

---

## PROFESSIONAL SUMMARY

Senior DevOps & Cloud Infrastructure Engineer with 10+ years at Ericsson — started as a Linux admin, moved into full cloud and DevOps ownership, and now lead platform engineering for enterprise telecom infrastructure. I've built most of what's on this resume from scratch: multi-account AWS Landing Zones, 18-stage DevSecOps pipelines, Kubernetes platforms across EKS and OpenShift, automated OS patching for 500+ servers, and a multi-region DR setup we actually test quarterly. The numbers are real — 35% cost reduction ($180K/year), SOC2 passed on first attempt, disaster recovery RTO under 3 minutes, MTTR cut from 2 hours to under 5 minutes. I'm drawn to the hard problems: making DR actually work when it counts, getting security into pipelines without slowing teams down, and turning a 2 AM incident into a 5-minute auto-recovery.

---

## TECHNICAL SKILLS

**Cloud – AWS:** VPC, EC2 ASG, ALB/NLB, EKS, RDS Aurora, ElastiCache, Lambda, API Gateway, EventBridge, SQS, SNS, DynamoDB, S3, ECR, CloudFront, WAF, Route53, ACM, KMS, Secrets Manager, IAM, IAM Identity Center, Organizations, Control Tower, SCPs, GuardDuty, Security Hub, AWS Config, Inspector, CloudTrail, CloudWatch, Cost Explorer, Compute Optimizer, AWS Budgets, Aurora Global Database, S3 CRR, VPC Endpoints, AWS FIS

**Infrastructure as Code:** Terraform (modules, remote state, workspaces, tfsec, Checkov, OPA/Conftest, Infracost), Ansible (AAP, roles, vault), CloudFormation

**Containers & Kubernetes:** Kubernetes (EKS, self-managed, OpenShift), Docker, Helm, Kustomize, Karpenter, Argo Rollouts, Istio Service Mesh

**CI/CD & GitOps:** Jenkins, ArgoCD, GitHub Actions, GitLab CI, Cosign, Kyverno

**Security & Compliance:** DevSecOps, Trivy, Snyk, SonarQube, OWASP ZAP, mTLS (Istio), Vault, CIS Benchmarks, NIST, ISO 27001, Pod Security Standards, RBAC

**Monitoring & Observability:** Prometheus, Grafana, AlertManager, CloudWatch, EFK/ELK, Jaeger, Kubecost, X-Ray, AppDynamics

**Networking:** VPC design, Subnets, NAT Gateway, Transit Gateway, VPN, Security Groups, NACLs, NetworkPolicies, Calico

**Scripting & Languages:** Python, Bash, PowerShell, Groovy (Jenkinsfile), HCL (Terraform), YAML

**Practices:** Platform Engineering, DevSecOps, GitOps, FinOps, SRE, ITIL, Zero-Trust, Canary Deployments, Policy-as-Code

---

## PROFESSIONAL EXPERIENCE

### Senior DevOps Engineer | Ericsson Global India Pvt. Ltd. | Noida, India | June 2025 – Present

Led platform engineering across 10+ person cross-functional team — infrastructure automation, security-first CI/CD, and self-healing systems for enterprise telecom platform.

**Infrastructure & Automation (Terraform + AWS)**
- Built production 3-tier AWS infrastructure (VPC, 3 AZs, CloudFront + WAF + ALB, EC2 ASG, Aurora Multi-AZ, ElastiCache) with Terraform modules — handles 5,000 req/s, auto-scales 3→20 instances, <30s DB failover
- Implemented IaC security scanning (tfsec + Checkov + OPA) in CI — zero misconfigurations reach production; Infracost shows cost impact on every PR

**Kubernetes & Helm**
- Operated K8s across 3 environments (self-managed, EKS, OpenShift) serving 15+ production microservices with standardised Helm chart deployments and environment-specific value overrides
- Karpenter autoscaling with Spot optimisation — 60-70% node cost reduction; bin-packing consolidation eliminates half-empty nodes

**Security & DevSecOps**
- Designed 18-stage DevSecOps pipeline (Jenkins + ArgoCD + Argo Rollouts) with 6 security layers: secret scan → SCA → SAST → Trivy image scan → Cosign signing → DAST (OWASP ZAP) — blast radius reduced 100% → 5%
- Supply chain security: Cosign + Kyverno admission control — only signed, scanned images from ECR run in production (SOC2 passed first attempt)
- Enforced CIS/NIST-aligned policies: Pod Security Standards (restricted), NetworkPolicies (default-deny), mTLS (Istio), RBAC least-privilege; secrets managed via AWS Secrets Manager + KMS

**Monitoring & Incident Response**
- Deployed Prometheus + Grafana + AlertManager + EFK with ServiceMonitor auto-discovery; structured alerting (P1→PagerDuty, P2→Slack, P3→Jira)
- Istio service mesh + Jaeger distributed tracing across 15 microservices — MTTR reduced from 2+ hours to <5 minutes

---

### DevOps Engineer | Ericsson Inc. | Dallas, US | Dec 2018 – June 2025

End-to-end ownership of cloud infrastructure, platform architecture, and DevOps toolchain across AWS for enterprise-scale product delivery teams.

**Multi-Account AWS Landing Zone**
- Architected 15-account AWS Landing Zone (5 OUs) with Terraform: SCPs, IAM Identity Center SSO, OIDC federation for CI/CD — account provisioning reduced from 2 weeks → 30 minutes
- Centralised security: GuardDuty + Security Hub + AWS Config + CloudTrail across all accounts; immutable log archive; SOC2/ISO27001 compliant

**Multi-Region HA & Disaster Recovery**
- Designed multi-region active-passive DR (Route53 health-check failover + Aurora Global Database + S3 CRR) — measured RTO: 3 minutes, RPO: <1 second; quarterly drills with AWS FIS chaos experiments

**Serverless & Event-Driven Automation**
- Built serverless security remediation engine (EventBridge + SQS + Lambda + DynamoDB + SNS) — auto-fixes AWS Config violations (open SGs, public S3, unencrypted EBS) in 90 seconds; full DynamoDB audit trail; API Gateway compliance dashboard

**FinOps — Cloud Cost Optimisation**
- Reduced AWS spend 35% ($180K/year): non-prod auto-stop (Lambda + EventBridge), Karpenter consolidation, VPA rightsizing, VPC Endpoints (eliminated NAT data transfer cost), Savings Plans, GP2→GP3, S3 lifecycle policies
- Kubecost per-namespace cost visibility with chargeback reporting; mandatory tagging enforced via SCPs; Cost Explorer + Compute Optimizer + AWS Budgets governance

**Terraform Module Library**
- Built reusable Terraform module library (VPC, EKS, RDS, IAM, ALB) for multi-region, multi-account deployments — CI/CD-driven plan/apply with separate state per account; drift detection via scheduled plans

**OS Patching Automation**
- Built enterprise OS patching automation (18-step lifecycle, 7-dimension validation, auto-rollback) on Ansible Automation Platform — 500+ RHEL servers/month, zero downtime, full ServiceNow ITIL CR lifecycle (open → close)
- 12 production-grade, idempotent Ansible roles; multi-system monitoring silence (Alertmanager + CloudWatch + PagerDuty) with auto-expire safety; AES-256 Vault for all credentials

---

### System/Linux Administrator | Ericsson Global India Pvt. Ltd. | Bangalore, India | Aug 2015 – Dec 2018

- Administered 100+ Linux servers (RHEL/CentOS) — CIS hardening, patching, performance tuning, capacity planning
- Automated operations with Bash/Python/Ansible — reduced manual SSH by 80%; built and maintained Jenkins CI/CD for 20+ services
- Implemented monitoring (Nagios → Prometheus) and centralised logging (ELK) — first cross-environment observability platform
- Managed backup strategies, DR runbooks, and incident response for production infrastructure

---

## KEY ACHIEVEMENTS

| Metric | Result | How |
|---|---|---|
| Deployment blast radius | 100% → 5% | 18-stage pipeline + Prometheus canary auto-rollback |
| Zero IaC misconfigs in production | ✅ | tfsec + Checkov + OPA in Terraform CI |
| Cloud cost reduction | 35% ($180K/year) | FinOps automation: auto-stop, Karpenter, rightsizing, Savings Plans |
| Disaster recovery RTO | <3 minutes | Multi-region active-passive, Aurora Global, quarterly validated |
| OS patching scale | 500+ servers/month, zero downtime | 18-step AAP automation, 7-dimension validation, ITIL |
| MTTR improvement | 2+ hours → <5 minutes | Istio + Jaeger distributed tracing across 15 microservices |
| SOC2 — first attempt pass | ✅ | Landing Zone: SCPs, OIDC, centralised logging, automated compliance |

---

## KEY PROJECTS

| Project | Highlights |
|---|---|
| DevSecOps CI/CD Pipeline — [github.com/siddharthpatel1993/dcp_devsecops](https://github.com/siddharthpatel1993/dcp_devsecops) | 18 stages, 6 security layers (secret→SCA→SAST→Trivy→Cosign→DAST), canary auto-rollback in 2 min, 900MB→150MB image, GitOps (ArgoCD) |
| 3-Tier AWS Architecture | VPC (9 subnets, 3 AZs), CloudFront+WAF+ALB, EC2 ASG (3→20), Aurora Multi-AZ, ElastiCache, Terraform modules, 5,000 req/s |
| Kubernetes Platform (EKS) | EKS + OpenShift + self-managed, Helm, Karpenter, Argo Rollouts, Istio mTLS, Kyverno, Kubecost, 15+ microservices |
| Multi-Account Landing Zone | 15 accounts, 5 OUs, SCPs, IAM Identity Center, OIDC, GuardDuty, Security Hub, Config, CloudTrail, account vending <30 min |
| Serverless Remediation Engine | EventBridge→SQS→Lambda→DynamoDB→SNS, auto-fix CIS violations in 90s, API Gateway dashboard, $0.07/month |
| Service Mesh (Istio) | mTLS STRICT, AuthorizationPolicies (zero-trust), VirtualService canary, Jaeger tracing, Kiali topology, circuit breakers |
| FinOps Platform | 35% cost reduction, non-prod auto-stop (Lambda), Karpenter consolidation, VPA, Infracost on PRs, Kubecost chargeback |
| Multi-Region HA & DR | Route53 failover, Aurora Global (<1s RPO), S3 CRR, warm pools, AWS FIS chaos drills, RTO <3 min, quarterly validated |
| OS Patching Automation — [gitlab.com/patelsiddharthnids993/ansible_os_automation](https://gitlab.com/patelsiddharthnids993/ansible_os_automation) | 18-step lifecycle, 7-dimension validation, auto-rollback, ServiceNow ITIL, 500+ RHEL/month, AAP, 12 Ansible roles |

---

## CERTIFICATIONS

- CKA — Certified Kubernetes Administrator (Linux Foundation)
- CKAD — Certified Kubernetes Application Developer (Linux Foundation)
- AWS Solutions Architect – Associate (Amazon Web Services)
- Certified SAFe® 5 DevOps Practitioner (Scaled Agile)

---

## PROFESSIONAL CONTRIBUTIONS

- **Awards:** Delivery Champion — Kubernetes 2022 | MANA ADM Champion — DevSecOps 2023-2024
- **Knowledge Sharing:** Internal tech talks on DevSecOps, K8s platform design, Terraform, and security automation
- **Mentoring:** Guided junior engineers through CKA certification and Ansible/Terraform adoption
- **Open Source:** 3 public repositories — CI/CD pipelines, Ansible automation, K8s infrastructure (GitHub + GitLab)
- **Continuous Learning:** Active in K8s/security/platform engineering communities; exploring Agentic AI in DevOps

---

## EDUCATION

Bachelor of Engineering (Computer Science / IT) — KIIT University, 2015
