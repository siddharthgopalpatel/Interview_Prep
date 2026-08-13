# 🚀 AWS Notes — Production-Grade Study Guide

> **Audience:** Cloud Engineer / Architect with 12+ YOE  
> **Focus:** VMs, Serverless, Containers — Production Deployments  
> **Style:** Simple English | Examples | Diagrams

---

## 📌 Topic Relevance Guide

| Symbol | Meaning |
|--------|---------|
| ✅ MUST HAVE | Critical for 12 YOE — expect deep questions |
| ⚡ GOOD TO KNOW | Adds depth, quick review |
| ⏭️ SKIP | Too basic for your level |

---

---

# 🏗️ CATEGORY: AWS FUNDAMENTALS & BUILDING BLOCKS

---

## 1. AWS Global Infrastructure ⚡ GOOD TO KNOW

> At 12 YOE, you should know this cold — but interviewers still ask "Why multi-AZ?" or "When do you use Edge Locations?"

### Core Concepts

| Component | What It Is | Count (approx) |
|-----------|-----------|-----------------|
| **Region** | A physical location with 2+ AZs | 30+ worldwide |
| **Availability Zone (AZ)** | One or more data centers with independent power, networking, cooling | 90+ globally |
| **Edge Location** | CDN endpoint for caching (CloudFront) | 400+ globally |

### How They Relate — Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     AWS GLOBAL INFRASTRUCTURE                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──── Region: us-east-1 (N. Virginia) ────────────────┐   │
│  │                                                      │   │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────┐          │   │
│  │  │  AZ-1a  │   │  AZ-1b  │   │  AZ-1c  │          │   │
│  │  │ (DC+DC) │   │ (DC+DC) │   │ (DC+DC) │          │   │
│  │  └─────────┘   └─────────┘   └─────────┘          │   │
│  │       ↕ Low-latency links ↕                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──── Region: eu-west-1 (Ireland) ───────────────────┐    │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────┐         │    │
│  │  │  AZ-1a  │   │  AZ-1b  │   │  AZ-1c  │         │    │
│  │  └─────────┘   └─────────┘   └─────────┘         │    │
│  └────────────────────────────────────────────────────┘    │
│                                                             │
│  ☁️ Edge Locations (400+) — Scattered globally for CDN     │
│     ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐                  │
│     │EL │ │EL │ │EL │ │EL │ │EL │ │EL │  ...              │
│     └───┘ └───┘ └───┘ └───┘ └───┘ └───┘                  │
└─────────────────────────────────────────────────────────────┘
```

### Key Points for Production

| Question | Answer |
|----------|--------|
| Why multi-AZ? | High availability — if one AZ goes down, traffic shifts to another |
| Why multi-Region? | Disaster recovery + low latency for global users |
| Edge Location vs AZ? | Edge = caching only (CloudFront, Route 53). AZ = full compute/storage |

### Example — Production Setup

```
Your App (Multi-AZ, us-east-1)
├── AZ-1a: EC2 instances + RDS Primary
├── AZ-1b: EC2 instances + RDS Standby (failover)
├── AZ-1c: EC2 instances (extra capacity)
└── CloudFront (Edge Locations) → Cache static assets globally
```

---

## 2. Shared Responsibility Model ✅ MUST HAVE

> This comes up in EVERY AWS interview. Know the boundary clearly.

### The Simple Rule

```
┌──────────────────────────────────────────────────────────┐
│           "CAN YOU DO IT IN THE AWS CONSOLE?"            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   YES → YOU are responsible                              │
│   NO  → AWS is responsible                               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Responsibility Split — Diagram

```
┌──────────────────────────────────────────────────────┐
│                CUSTOMER RESPONSIBILITY                │
│              ("Security IN the Cloud")                │
├──────────────────────────────────────────────────────┤
│  • IAM users, roles, policies                        │
│  • Security Groups / NACLs                           │
│  • OS patching (EC2)                                 │
│  • Application code & data                           │
│  • Encryption configuration                          │
│  • Network configuration                             │
│  • Firewall rules                                    │
├──────────────────────────────────────────────────────┤
│            🤝 SHARED RESPONSIBILITY                   │
│  • Encryption (you enable, AWS provides tools)       │
│  • Patch Management (depends on service)             │
├──────────────────────────────────────────────────────┤
│                 AWS RESPONSIBILITY                    │
│              ("Security OF the Cloud")               │
├──────────────────────────────────────────────────────┤
│  • Physical data center security                     │
│  • Hardware maintenance                              │
│  • Network infrastructure                            │
│  • Hypervisor                                        │
│  • Managed service patching (RDS OS, Lambda runtime) │
│  • Power, cooling, cabling                           │
└──────────────────────────────────────────────────────┘
```

### Production Example — EC2 vs RDS vs Lambda

| Task | EC2 (IaaS) | RDS (PaaS) | Lambda (FaaS) |
|------|------------|------------|----------------|
| OS Patching | **You** | AWS | AWS |
| App Code | **You** | **You** | **You** |
| Scaling | **You** (ASG) | **You** (config) | AWS (auto) |
| HA Setup | **You** (Multi-AZ) | **You** (enable it) | AWS (built-in) |
| Physical Security | AWS | AWS | AWS |

> 💡 **Pro Tip:** The more "managed" the service, the less you manage — but you ALWAYS own your data and access control.

---

## 3. Key Services Overview ⚡ GOOD TO KNOW

> Quick reference map — we'll deep-dive each in their respective category sections.

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS KEY SERVICES MAP                      │
├─────────────┬───────────────────────────────────────────────┤
│  COMPUTE    │  EC2 | Lambda | Elastic Beanstalk             │
│             │  ECS | EKS | Fargate                          │
├─────────────┼───────────────────────────────────────────────┤
│  STORAGE    │  S3 | EBS | EFS | FSx | Storage Gateway       │
├─────────────┼───────────────────────────────────────────────┤
│  DATABASE   │  RDS | DynamoDB | Redshift | Aurora            │
│             │  ElastiCache | Neptune                         │
├─────────────┼───────────────────────────────────────────────┤
│  NETWORKING │  VPC | Direct Connect | Route 53               │
│             │  API Gateway | Global Accelerator | ELB        │
├─────────────┼───────────────────────────────────────────────┤
│  SECURITY   │  IAM | KMS | WAF | Shield | GuardDuty         │
│             │  Secrets Manager | Certificate Manager         │
├─────────────┼───────────────────────────────────────────────┤
│  SERVERLESS │  Lambda | API Gateway | DynamoDB | S3          │
│             │  Step Functions | EventBridge | SQS/SNS        │
├─────────────┼───────────────────────────────────────────────┤
│  CONTAINERS │  ECS | EKS | Fargate | ECR | App Runner       │
├─────────────┼───────────────────────────────────────────────┤
│  MONITORING │  CloudWatch | CloudTrail | X-Ray               │
│  & LOGGING  │  Config | Trusted Advisor                      │
├─────────────┼───────────────────────────────────────────────┤
│  DECOUPLING │  SQS | SNS | EventBridge | Kinesis            │
│             │  Step Functions | MQ                            │
└─────────────┴───────────────────────────────────────────────┘
```

---

## 4. Well-Architected Framework (6 Pillars) ✅ MUST HAVE

> Interviewers LOVE asking "Which pillar does this fall under?" — Know all 6.

### The 6 Pillars — At a Glance

```
                    ┌─────────────────────┐
                    │   WELL-ARCHITECTED  │
                    │     FRAMEWORK       │
                    └────────┬────────────┘
                             │
        ┌────────┬───────┬──┴───┬─────────┬──────────┐
        ▼        ▼       ▼      ▼         ▼          ▼
   ┌────────┐┌───────┐┌─────┐┌──────┐┌────────┐┌────────────┐
   │OPERAT- ││SECUR- ││RELI-││PERF- ││COST    ││SUSTAIN-    │
   │IONAL   ││ITY    ││ABI- ││ORMA- ││OPTIMI- ││ABILITY     │
   │EXCELL- ││       ││LITY ││NCE   ││ZATION  ││            │
   │ENCE    ││       ││     ││EFFIC.││        ││            │
   └────────┘└───────┘└─────┘└──────┘└────────┘└────────────┘
```

### Deep Dive — Each Pillar

| # | Pillar | One-Liner | Production Example |
|---|--------|-----------|-------------------|
| 1 | **Operational Excellence** | Run & monitor systems, keep improving | CI/CD pipelines, runbooks, CloudWatch dashboards |
| 2 | **Security** | Protect information & systems | IAM least privilege, encryption at rest/transit, WAF |
| 3 | **Reliability** | Workload performs correctly & consistently | Multi-AZ, auto-scaling, health checks, backups |
| 4 | **Performance Efficiency** | Use resources efficiently | Right-sizing EC2, caching (ElastiCache), CDN |
| 5 | **Cost Optimization** | Avoid unnecessary costs | Reserved Instances, spot fleet, S3 lifecycle policies |
| 6 | **Sustainability** | Minimize environmental impact | Right-size, use managed services, Graviton instances |

### Memory Trick 🧠

> **"OSRPCS"** — Think: **O**ur **S**ecurity **R**equires **P**erfect **C**ost **S**avings

---

## 5. My Recommendation for Your Level (12 YOE)

| Topic from Your Notes | Verdict | Why |
|----------------------|---------|-----|
| Global Infrastructure (Region/AZ/Edge) | ⚡ Quick Review | You know this — but know the "why" for design decisions |
| Shared Responsibility Model | ✅ MUST Revise | Comes up in every interview, especially for senior roles |
| Key Services List | ⚡ Reference Only | You'll deep-dive each service in their category |
| Well-Architected Framework | ✅ MUST Know | Architects are expected to design around these pillars |

---

> 📝 **Next:** Share your next set of rough notes and I'll add them under the appropriate category (Compute, Storage, Networking, etc.) in this same file!


---

---

# 🔐 CATEGORY: SECURITY

---

## 1. IAM (Identity and Access Management) ✅ MUST HAVE

> At 12 YOE, you're expected to DESIGN IAM strategies, not just create users. Think: least privilege, cross-account access, federation, SCPs.

### What is IAM?

IAM is AWS's **global service** (not region-specific) that controls **WHO** can access **WHAT** in your AWS account.

```
┌─────────────────────────────────────────────────────────────┐
│                        IAM OVERVIEW                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   WHO (Identity)          WHAT (Permissions)                │
│   ┌──────────┐           ┌──────────────────┐              │
│   │  Users   │──────────▶│  IAM Policy      │              │
│   │  Groups  │──────────▶│  (JSON Document) │──▶ AWS       │
│   │  Roles   │──────────▶│                  │   Resources  │
│   └──────────┘           └──────────────────┘              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### 🔒 4 Steps to Secure Your Root Account

```
Step 1 ──▶ Enable MFA on Root Account (use hardware MFA in production)
              │
Step 2 ──▶ Create an Admin Group with AdministratorAccess policy
              │
Step 3 ──▶ Create IAM User accounts for your admins
              │
Step 4 ──▶ Add admin users to the Admin Group
              │
Result ──▶ 🚫 NEVER use Root Account for daily tasks again
```

> 💡 **Production Rule:** Root account should ONLY be used for billing and account-level tasks. Lock it with hardware MFA and store credentials in a vault.

---

### IAM Building Blocks — Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     IAM STRUCTURE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                    AWS ACCOUNT                       │   │
│  │                                                     │   │
│  │   👑 Root User (God Mode — NEVER use daily)         │   │
│  │                                                     │   │
│  │   ┌──── Groups ─────────────────────────────┐      │   │
│  │   │                                          │      │   │
│  │   │  ┌─────────┐  ┌──────────┐  ┌────────┐ │      │   │
│  │   │  │  Admin  │  │Developers│  │  QA    │ │      │   │
│  │   │  │  Group  │  │  Group   │  │ Group  │ │      │   │
│  │   │  └────┬────┘  └────┬─────┘  └───┬────┘ │      │   │
│  │   │       │             │            │       │      │   │
│  │   └───────┼─────────────┼────────────┼───────┘      │   │
│  │           ▼             ▼            ▼              │   │
│  │     ┌─────────┐  ┌─────────┐  ┌─────────┐         │   │
│  │     │ User A  │  │ User B  │  │ User C  │         │   │
│  │     └─────────┘  └─────────┘  └─────────┘         │   │
│  │                                                     │   │
│  │   ┌──── Roles ──────────────────────────────┐      │   │
│  │   │  • EC2-S3-Access-Role                   │      │   │
│  │   │  • Lambda-DynamoDB-Role                 │      │   │
│  │   │  • Cross-Account-Role                   │      │   │
│  │   └─────────────────────────────────────────┘      │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

### IAM Policy Document (JSON) — Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-production-bucket/*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::my-production-bucket/*"
    }
  ]
}
```

**Policy can be attached to:** Users, Groups, or Roles

```
         Policy
           │
     ┌─────┼──────┐
     ▼     ▼      ▼
   User  Group   Role
```

---

### IAM Key Facts — Quick Reference

| Fact | Detail |
|------|--------|
| **IAM is Universal** | NOT region-specific — applies globally across all regions |
| **New Users = Zero Permissions** | By default, new users have NO access. You must explicitly grant |
| **Access Keys ≠ Password** | Access Key ID + Secret Key are for CLI/API only, not console login |
| **Access Keys — One Shot** | You can only view/download secret key ONCE at creation. Lose it = regenerate |
| **Password Rotation** | Always configure password policies (min length, rotation, complexity) |
| **MFA** | Enable everywhere — especially root and admin users |

---

### IAM Federation — Diagram

> Lets users log into AWS using corporate credentials (Active Directory, Okta, etc.)

```
┌─────────────────┐          ┌──────────────┐         ┌─────────────┐
│  Corporate IdP  │──SAML──▶│   AWS IAM    │────────▶│ AWS Console │
│  (Active Dir /  │  Trust   │  Federation  │  Temp   │ / Resources │
│   Okta / Azure  │          │              │  Creds  │             │
│   AD)           │          │              │         │             │
└─────────────────┘          └──────────────┘         └─────────────┘

Flow:
1. User logs into Corporate IdP (e.g., Active Directory)
2. IdP sends SAML assertion to AWS
3. AWS STS issues temporary security credentials
4. User accesses AWS Console/API — no separate AWS password needed!
```

| Federation Type | Protocol | Use Case |
|----------------|----------|----------|
| SAML 2.0 | XML-based | Enterprise SSO (Active Directory, Okta) |
| Web Identity | OIDC (OpenID Connect) | Mobile apps (Google, Facebook, Amazon login) |
| AWS SSO | Built-in | Multi-account access via AWS Organizations |

---

### Production Best Practices (12 YOE Level)

```
┌─────────────────────────────────────────────────────────────┐
│            IAM PRODUCTION CHECKLIST                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Least Privilege — grant ONLY what's needed              │
│  ✅ Use Roles for services (EC2, Lambda, ECS) — NOT keys   │
│  ✅ Enable MFA for all human users                         │
│  ✅ Use IAM Access Analyzer to find unused permissions     │
│  ✅ Rotate access keys every 90 days                       │
│  ✅ Use SCPs (Service Control Policies) in Organizations   │
│  ✅ Never embed access keys in code — use Roles or         │
│     Secrets Manager                                         │
│  ✅ Use Conditions in policies (IP, MFA, time-based)       │
│  ✅ Separate accounts: Dev / Staging / Prod                │
│  ✅ CloudTrail for audit trail of ALL IAM actions          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Interview-Level Example — Cross-Account Access

```
┌──────────────┐                    ┌──────────────┐
│  Account A   │                    │  Account B   │
│  (Dev Team)  │                    │  (Prod Data) │
│              │                    │              │
│  User/Role ──┼── AssumeRole ────▶│  Role with   │
│              │   (STS)           │  S3 access   │
└──────────────┘                    └──────────────┘

How:
1. Account B creates a Role with trust policy allowing Account A
2. Account A user calls sts:AssumeRole
3. Gets temporary credentials to access Account B resources
4. No need to create IAM users in Account B!
```

---

### My Recommendation for Your Level

| IAM Topic | Verdict | Why |
|-----------|---------|-----|
| Root account security | ⚡ Quick review | You should know this already |
| Policy documents (JSON) | ✅ Practice writing them | Interviews ask you to write/debug policies |
| Federation (SAML/SSO) | ✅ Must know flow | Enterprise environments = federation everywhere |
| Cross-account roles | ✅ Must know | Multi-account is standard in production |
| SCPs & Organizations | ✅ Must know | Expected at senior/architect level |
| Basic user/group creation | ⏭️ Skip drilling | Too basic — just know the concepts |


---

---

# 📦 CATEGORY: STORAGE

---

## 1. S3 (Simple Storage Service) ✅ MUST HAVE

> S3 is THE most-asked storage service. At 12 YOE, know storage classes, lifecycle, encryption, replication, and access patterns cold.

### What is S3?

S3 is **object-based storage** — think of it as a flat file system in the cloud with unlimited capacity.

```
┌─────────────────────────────────────────────────────────────┐
│                      S3 AT A GLANCE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Object-based (files, images, videos, backups)           │
│  ❌ NOT for OS installation or databases                    │
│  📁 File size: 0 bytes → 5 TB                              │
│  ♾️  Unlimited total storage                                │
│  🌍 Universal namespace (bucket name globally unique)       │
│  📊 HTTP 200 = successful upload                            │
│  🔒 Private by default                                      │
│  📈 Auto-scales with demand                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### S3 Object Structure — Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    S3 OBJECT                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────┬──────────────────────────────────────┐  │
│   │  Key     │  Object name (e.g., photos/cat.jpg)  │  │
│   ├──────────┼──────────────────────────────────────┤  │
│   │  Value   │  The actual file data (bytes)        │  │
│   ├──────────┼──────────────────────────────────────┤  │
│   │ Version  │  Unique ID for each version          │  │
│   │   ID     │  (when versioning enabled)           │  │
│   ├──────────┼──────────────────────────────────────┤  │
│   │ Metadata │  Content-type, last-modified,        │  │
│   │          │  custom tags, etc.                   │  │
│   └──────────┴──────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘

URL Pattern:
https://<bucket-name>.s3.<region>.amazonaws.com/<key>
https://my-app-prod.s3.us-east-1.amazonaws.com/photos/cat.jpg
```

---

### S3 Access Control — Diagram

```
┌─────────────────────────────────────────────────────────────┐
│              S3 ACCESS CONTROL LAYERS                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  By Default: EVERYTHING IS PRIVATE 🔒                       │
│                                                             │
│  To make things public, you need:                           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Level 1: BUCKET POLICY                              │   │
│  │  → Controls access to the ENTIRE bucket              │   │
│  │  → JSON-based (like IAM policy)                      │   │
│  │  → Example: Make all objects in bucket public        │   │
│  └─────────────────────────────────────────────────────┘   │
│                           +                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Level 2: OBJECT ACL                                 │   │
│  │  → Controls access to INDIVIDUAL objects             │   │
│  │  → Example: Make just one file public                │   │
│  └─────────────────────────────────────────────────────┘   │
│                           +                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Level 3: BLOCK PUBLIC ACCESS (Account/Bucket)       │   │
│  │  → Override that blocks ALL public access            │   │
│  │  → Must be DISABLED to allow any public access       │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### S3 for Static Website Hosting

```
┌────────────┐        ┌────────────────┐        ┌────────────┐
│   User     │──GET──▶│   CloudFront   │──────▶│  S3 Bucket │
│  Browser   │        │   (CDN/Edge)   │        │  (Static)  │
└────────────┘        └────────────────┘        └────────────┘

Rules:
✅ Static content only (HTML, CSS, JS, images)
❌ No dynamic server-side code (no PHP, no Node.js backend)
📈 S3 scales automatically — no provisioning needed
💰 Extremely cheap for hosting static sites
```

---

### S3 Versioning ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                    S3 VERSIONING                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Upload v1: report.pdf (Version ID: abc111)                 │
│  Upload v2: report.pdf (Version ID: abc222) ← Current      │
│  Upload v3: report.pdf (Version ID: abc333) ← Current      │
│                                                             │
│  DELETE report.pdf → Adds "Delete Marker" (not truly gone!) │
│  Previous versions STILL exist — recoverable!               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

| Key Fact | Detail |
|----------|--------|
| All versions stored | Every write creates a new version (costs storage!) |
| Great backup tool | Accidentally deleted? Restore previous version |
| Cannot be disabled | Once enabled → can only be **suspended**, not turned off |
| Works with Lifecycle | Move old versions to cheaper storage automatically |
| MFA Delete | Require MFA to permanently delete versions (extra protection) |

---

### S3 Storage Classes ✅ MUST HAVE

> Know when to use each — interviewers ask "Which class for X scenario?"

```
┌────────────────────────────────────────────────────────────────────────┐
│                    S3 STORAGE CLASSES — COMPARISON                      │
├────────────────────┬────────┬───────────┬──────────┬───────────────────┤
│ Class              │ Avail. │ Min Store │ Retrieval│ Use Case          │
├────────────────────┼────────┼───────────┼──────────┼───────────────────┤
│ S3 Standard        │ 99.99% │ None      │ Instant  │ Frequently        │
│                    │        │           │          │ accessed data     │
├────────────────────┼────────┼───────────┼──────────┼───────────────────┤
│ S3 Intelligent-    │ 99.9%  │ 30 days   │ Instant  │ Unknown or        │
│ Tiering            │        │           │          │ changing access   │
├────────────────────┼────────┼───────────┼──────────┼───────────────────┤
│ S3 Standard-IA     │ 99.9%  │ 30 days   │ Instant  │ Infrequent but    │
│ (Infrequent Access)│        │           │ (per-GB) │ needs fast access │
├────────────────────┼────────┼───────────┼──────────┼───────────────────┤
│ S3 One Zone-IA     │ 99.5%  │ 30 days   │ Instant  │ Non-critical,     │
│                    │        │           │ (per-GB) │ re-creatable data │
├────────────────────┼────────┼───────────┼──────────┼───────────────────┤
│ Glacier Instant    │ 99.9%  │ 90 days   │ Instant  │ Archive needing   │
│ Retrieval          │        │           │ (per-GB) │ millisec access   │
├────────────────────┼────────┼───────────┼──────────┼───────────────────┤
│ Glacier Flexible   │ 99.9%  │ 90 days   │ 1min to  │ Archive, okay     │
│ Retrieval          │        │           │ 12hrs    │ to wait minutes   │
├────────────────────┼────────┼───────────┼──────────┼───────────────────┤
│ Glacier Deep       │ 99.99% │ 180 days  │ 12-48hrs │ Compliance,       │
│ Archive            │        │           │          │ 7-10yr retention  │
└────────────────────┴────────┴───────────┴──────────┴───────────────────┘
```

### Storage Classes — Simple Explanations

| Class | Think of it as... | When to Use |
|-------|------------------|-------------|
| **S3 Standard** | Your everyday hard drive | Active app data, images serving to users, frequently read files. Default choice for most workloads. |
| **S3 Intelligent-Tiering** | Auto-pilot storage manager | When you don't know access patterns. AWS auto-moves objects between tiers. Small monitoring fee, but saves money on unpredictable workloads. |
| **S3 Standard-IA** | Filing cabinet in the next room | Backups, disaster recovery data, older reports. Accessed maybe once a month but need it fast when you do. Cheaper storage, retrieval fee applies. |
| **S3 One Zone-IA** | Cheap filing cabinet (no backup copy) | Thumbnails, re-processable data. Same as Standard-IA but stored in ONE AZ only. If that AZ goes down, data is gone. 20% cheaper. |
| **Glacier Instant Retrieval** | Archive vault with quick access | Medical images, news archives. Rarely accessed (once per quarter) but when needed, you need it NOW in milliseconds. |
| **Glacier Flexible Retrieval** | Deep archive — can wait minutes/hours | Yearly audit data, old backups. Retrieval options: Expedited (1-5 min), Standard (3-5 hrs), Bulk (5-12 hrs). |
| **Glacier Deep Archive** | Cold storage bunker | Compliance data (financial, healthcare) that regulations require keeping 7-10 years. Cheapest storage. 12-48 hour retrieval. |

### Visual — Cost vs Access Speed

```
  COST ($/GB/month)
  HIGH │  Standard ($0.023)
       │      ↓
       │  Intelligent-Tiering ($0.023 → auto-optimizes)
       │      ↓
       │  Standard-IA ($0.0125)
       │      ↓
       │  One Zone-IA ($0.01)
       │      ↓
       │  Glacier Instant ($0.004)
       │      ↓
       │  Glacier Flexible ($0.0036)
       │      ↓
  LOW  │  Glacier Deep Archive ($0.00099)
       └───────────────────────────────────────▶
             RETRIEVAL TIME (ms → hours)
```

---

### S3 Lifecycle Management ✅ MUST HAVE

> Automate moving data to cheaper tiers as it ages — saves serious money in production.

```
┌──────────────────────────────────────────────────────────────┐
│              LIFECYCLE RULE EXAMPLE                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Day 0          Day 30           Day 90           Day 365   │
│    │               │                │                │       │
│    ▼               ▼                ▼                ▼       │
│ ┌──────┐      ┌─────────┐     ┌──────────┐    ┌─────────┐ │
│ │ S3   │─────▶│Standard │────▶│ Glacier  │───▶│ Glacier │ │
│ │Stand.│      │   IA    │     │ Flexible │    │  Deep   │ │
│ └──────┘      └─────────┘     └──────────┘    └─────────┘ │
│                                                              │
│  • Applies to current AND previous versions                  │
│  • Works with versioning enabled                             │
│  • Can also auto-DELETE after X days                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Production Example:**
```
Lifecycle Rule for logs/:
- After 30 days  → Move to Standard-IA (still need quick access)
- After 90 days  → Move to Glacier Flexible (audit purposes)
- After 365 days → Move to Deep Archive (compliance)
- After 7 years  → DELETE (retention met)
```

---

### S3 Object Lock & Glacier Vault Lock ✅ MUST HAVE

> For compliance (HIPAA, SEC, financial regulations) — WORM model.

```
┌─────────────────────────────────────────────────────────────┐
│         S3 OBJECT LOCK (WORM: Write Once, Read Many)        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  GOVERNANCE MODE                                     │   │
│  │  • Most users can't delete/overwrite                 │   │
│  │  • Users with SPECIAL permissions CAN override       │   │
│  │  • Use: Prevent accidental deletes, allow admin fix  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  COMPLIANCE MODE                                     │   │
│  │  • NOBODY can delete — not even ROOT user!           │   │
│  │  • Cannot be shortened or removed during retention   │   │
│  │  • Use: Regulatory compliance (SEC 17a-4, HIPAA)     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Can apply to: Individual objects OR entire bucket           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

> 💡 **Glacier Vault Lock** = same concept but for Glacier archives. Once locked, the policy can NEVER be changed.

---

### S3 Encryption ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                S3 ENCRYPTION OPTIONS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── IN TRANSIT ───────────────────────────────────────┐  │
│  │  SSL/TLS (HTTPS) — data encrypted while moving       │  │
│  │  Always use HTTPS endpoints for S3                    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─── AT REST (Server-Side Encryption) ─────────────────┐  │
│  │                                                       │  │
│  │  SSE-S3    → AWS manages keys (AES-256)              │  │
│  │              Simplest. Default for new buckets.       │  │
│  │                                                       │  │
│  │  SSE-KMS   → AWS KMS manages keys                    │  │
│  │              Audit trail + key rotation + control     │  │
│  │              ⚠️ KMS API rate limits apply             │  │
│  │                                                       │  │
│  │  SSE-C     → YOU provide the key, AWS encrypts       │  │
│  │              You manage key lifecycle entirely        │  │
│  │              Must use HTTPS                           │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─── CLIENT-SIDE ENCRYPTION ───────────────────────────┐  │
│  │  YOU encrypt before uploading to S3                   │  │
│  │  YOU decrypt after downloading                        │  │
│  │  AWS never sees unencrypted data                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Enforcing Encryption via Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    }
  ]
}
```
> This denies any upload that doesn't use KMS encryption. Production standard!

---

### S3 Replication ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                  S3 REPLICATION                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    Replication     ┌─────────────┐       │
│  │  Source      │─────────────────▶│ Destination │       │
│  │  Bucket      │    (async)        │   Bucket    │       │
│  │  us-east-1   │                   │  eu-west-1  │       │
│  └─────────────┘                    └─────────────┘       │
│                                                             │
│  Types:                                                     │
│  • CRR (Cross-Region Replication) — DR & compliance        │
│  • SRR (Same-Region Replication) — log aggregation         │
│                                                             │
│  Key Rules:                                                 │
│  ⚠️ Existing objects NOT replicated automatically           │
│     (use S3 Batch Replication for existing objects)         │
│  ⚠️ Delete markers NOT replicated by default                │
│     (can enable, but permanent deletes never replicate)     │
│  ✅ Versioning MUST be enabled on both buckets              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Production Best Practices (12 YOE Level)

```
┌─────────────────────────────────────────────────────────────┐
│            S3 PRODUCTION CHECKLIST                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Enable versioning on all important buckets              │
│  ✅ Enable server-side encryption (SSE-KMS for sensitive)   │
│  ✅ Block public access at account level (unless needed)    │
│  ✅ Use Lifecycle rules to optimize storage costs           │
│  ✅ Enable access logging to audit bucket                   │
│  ✅ Use CRR for disaster recovery across regions            │
│  ✅ Use VPC Endpoints (Gateway) to avoid internet traffic   │
│  ✅ Use pre-signed URLs for temporary secure access         │
│  ✅ Enable MFA Delete for critical data buckets             │
│  ✅ Use S3 Event Notifications → Lambda for automation      │
│  ✅ Monitor with CloudWatch metrics + S3 Storage Lens       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| S3 Topic | Verdict | Why |
|----------|---------|-----|
| Object basics (key, value, metadata) | ⏭️ Skip drilling | You know this already |
| Storage Classes | ✅ Must memorize tradeoffs | Scenario-based questions guaranteed |
| Lifecycle Management | ✅ Must know | Cost optimization is key at senior level |
| Object Lock / Vault Lock | ✅ Must know | Compliance questions for production systems |
| Encryption (all types) | ✅ Must know | Security pillar — always asked |
| Replication (CRR/SRR) | ✅ Must know | DR strategy questions |
| Static website hosting | ⚡ Quick review | Simple concept, occasionally asked |
| Bucket policies vs ACLs | ✅ Know the difference | Access control design questions |


---

---

# 💻 CATEGORY: COMPUTE

---

## 1. EC2 (Elastic Compute Cloud) ✅ MUST HAVE

> EC2 is the backbone of AWS Compute. At 12 YOE, you must know pricing strategies, networking options, placement groups, instance types, and production-grade deployment patterns.

### What is EC2?

A **virtual machine** hosted in AWS — you select capacity, scale up/down on demand, and pay only for what you use.

```
┌─────────────────────────────────────────────────────────────┐
│                     EC2 AT A GLANCE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🖥️  VM in the cloud (replaces on-prem servers)             │
│  📐 Select capacity you need RIGHT NOW                      │
│  📈 Grow & shrink based on demand (Auto Scaling)            │
│  💰 Pay for what you use (per hour or per second)           │
│  ⏱️  Launch in minutes, not months                           │
│  🐧 Supports Linux, Windows, macOS                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### EC2 Pricing Models ✅ MUST HAVE

> This is THE most-asked EC2 topic. Know when to use each model.

```
┌─────────────────────────────────────────────────────────────────┐
│                  EC2 PRICING MODELS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │ ON-DEMAND │  │ RESERVED  │  │   SPOT    │  │ DEDICATED │  │
│  │           │  │           │  │           │  │           │  │
│  │ 💰 Full   │  │ 💰 Up to  │  │ 💰 Up to  │  │ 💰 Most   │  │
│  │   Price   │  │   72% off │  │   90% off │  │  Expensive│  │
│  │           │  │           │  │           │  │           │  │
│  │ ⏱️ By hr/ │  │ ⏱️ 1 or 3 │  │ ⏱️ Varies │  │ ⏱️ By hr  │  │
│  │   second  │  │   years   │  │   (bid)   │  │           │  │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Pricing Comparison Table

| Model | Discount | Commitment | Best For | Can Be Interrupted? |
|-------|----------|-----------|----------|-------------------|
| **On-Demand** | 0% (full price) | None | Unpredictable workloads, testing, dev environments | ❌ No |
| **Reserved (RI)** | Up to 72% | 1 or 3 years | Steady-state apps (databases, core APIs) | ❌ No |
| **Spot** | Up to 90% | None (can be reclaimed) | Batch processing, CI/CD, data analysis, stateless workers | ✅ Yes (2-min warning) |
| **Dedicated Host** | Varies | None or Reserved | Compliance, licensing (Windows Server, Oracle, SQL Server) | ❌ No |

### When to Use What — Decision Flow

```
                    ┌─────────────────────┐
                    │ Do you have a fixed, │
                    │ predictable workload?│
                    └─────────┬───────────┘
                         YES/  \NO
                         /      \
              ┌─────────▼┐    ┌─▼──────────────────┐
              │ RESERVED │    │ Can your app handle │
              │ INSTANCE │    │ interruptions?      │
              └──────────┘    └──────────┬──────────┘
                                    YES/  \NO
                                    /      \
                         ┌─────────▼┐    ┌─▼──────────┐
                         │   SPOT   │    │ ON-DEMAND  │
                         │ INSTANCE │    │            │
                         └──────────┘    └────────────┘
                         
              ┌──────────────────────────────────────┐
              │ Do you have licensing or compliance   │
              │ requiring a physical server?          │
              │           YES → DEDICATED HOST        │
              └──────────────────────────────────────┘
```

---

### Spot Instances — Deep Dive ✅ MUST HAVE

> Biggest cost saver in AWS. At senior level, know Spot Fleet strategies.

```
┌─────────────────────────────────────────────────────────────┐
│                   SPOT INSTANCES                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  How it works:                                              │
│  1. You set a max price you're willing to pay               │
│  2. As long as spot price < your max → instance runs        │
│  3. If spot price > your max → 2-minute warning → terminate │
│                                                             │
│  ┌─── SPOT FLEET ──────────────────────────────────────┐   │
│  │  A collection of Spot + (optionally) On-Demand       │   │
│  │                                                      │   │
│  │  Strategies:                                         │   │
│  │  • capacityOptimized → pick pool with most capacity  │   │
│  │  • lowestPrice → pick cheapest pool                  │   │
│  │  • diversified → spread across all pools             │   │
│  │  • InstancePoolsToUseCount → spread across N pools   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ⚠️ Spot Block: Reserve spot for 1-6 hours (no interrupt)   │
│  ✅ Great for: Batch jobs, CI/CD, big data, rendering       │
│  ❌ Bad for: Databases, critical stateful services          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Production Example — Spot Fleet

```
┌──────────────────────────────────────────────────────────────┐
│             SPOT FLEET — WEB CRAWLER EXAMPLE                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Target capacity: 100 vCPUs                                  │
│                                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐   │
│  │  Spot   │  │  Spot   │  │  Spot   │  │  On-Demand  │   │
│  │ m5.2xl  │  │ c5.2xl  │  │ r5.2xl  │  │  (fallback) │   │
│  │ (40%)   │  │ (30%)   │  │ (20%)   │  │   (10%)     │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────────┘   │
│                                                              │
│  Strategy: diversified (spread across instance types/AZs)    │
│  Result: If one pool gets reclaimed, others keep running     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### Security Groups ✅ MUST HAVE

> Think of Security Groups as a virtual firewall around your EC2 instance.

```
┌─────────────────────────────────────────────────────────────┐
│                  SECURITY GROUPS                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│              INTERNET                                        │
│                 │                                            │
│                 ▼                                            │
│  ┌──────────────────────────────┐                          │
│  │      SECURITY GROUP          │                          │
│  │  ┌────────────────────────┐  │                          │
│  │  │ INBOUND RULES          │  │                          │
│  │  │ • Port 22 (SSH) ✅     │  │                          │
│  │  │ • Port 80 (HTTP) ✅    │  │                          │
│  │  │ • Port 443 (HTTPS) ✅  │  │                          │
│  │  │ • Everything else ❌    │  │                          │
│  │  └────────────────────────┘  │                          │
│  │                              │                          │
│  │  ┌────────────────────────┐  │                          │
│  │  │ OUTBOUND RULES         │  │                          │
│  │  │ • ALL traffic ✅       │  │                          │
│  │  └────────────────────────┘  │                          │
│  │                              │                          │
│  │       ┌──────────────┐       │                          │
│  │       │  EC2 Instance │      │                          │
│  │       └──────────────┘       │                          │
│  └──────────────────────────────┘                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Security Group Key Facts

| Fact | Detail |
|------|--------|
| **Changes = Immediate** | No waiting — rules apply instantly |
| **Stateful** | If you allow inbound traffic, the response is automatically allowed out |
| **Multiple SGs per instance** | You can attach multiple security groups to one EC2 |
| **Multiple instances per SG** | One SG can protect many EC2 instances |
| **All inbound BLOCKED** by default | You must explicitly open ports |
| **All outbound ALLOWED** by default | Instances can reach the internet |
| **No DENY rules** | You can only ALLOW — what's not allowed is implicitly denied |
| **Can reference other SGs** | Allow traffic from another SG (e.g., "allow from ALB SG") |

### Production Pattern — SG Chaining

```
┌──────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────┐
│  Users   │─────▶│   ALB SG     │─────▶│  App SG      │─────▶│  DB SG   │
│(Internet)│      │ Allow 80/443 │      │ Allow from   │      │Allow from│
│          │      │ from 0.0.0.0 │      │ ALB SG only  │      │App SG    │
└──────────┘      └──────────────┘      └──────────────┘      │only:3306 │
                                                                └──────────┘

Result: DB only reachable from App tier. App only reachable from ALB.
        No direct internet access to App or DB!
```

---

### Bootstrap Scripts (User Data) ⚡ GOOD TO KNOW

> A script that runs ONCE when an EC2 instance first launches. Used to automate setup.

```
┌─────────────────────────────────────────────────────────────┐
│                 BOOTSTRAP SCRIPT FLOW                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  EC2 Instance Launches                                      │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────────────────────────────────┐              │
│  │  USER DATA (Bootstrap Script) executes:  │              │
│  │                                           │              │
│  │  #!/bin/bash                              │              │
│  │  yum update -y                            │              │
│  │  yum install -y httpd                     │              │
│  │  systemctl start httpd                    │              │
│  │  systemctl enable httpd                   │              │
│  │  echo "<h1>Hello World</h1>" > \          │              │
│  │       /var/www/html/index.html            │              │
│  └──────────────────────────────────────────┘              │
│         │                                                   │
│         ▼                                                   │
│  Instance is READY with web server running!                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### User Data vs Metadata

| Concept | What It Is | How to Access |
|---------|-----------|---------------|
| **User Data** | Your bootstrap script (the commands to run) | `http://169.254.169.254/latest/user-data` |
| **Metadata** | Info ABOUT the instance (IP, instance-id, AMI, etc.) | `http://169.254.169.254/latest/meta-data/` |

> 💡 You can use User Data scripts to query Metadata — e.g., get the instance's own IP at boot time and register it somewhere.

---

### EC2 Networking ✅ MUST HAVE

> Know the 3 networking options — interview scenarios love EFA questions.

```
┌─────────────────────────────────────────────────────────────────┐
│              EC2 NETWORKING OPTIONS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─── ENI (Elastic Network Interface) ──────────────────────┐  │
│  │  • Basic networking (default)                             │  │
│  │  • Low cost, separate management/prod/logging networks    │  │
│  │  • Can attach multiple ENIs to one instance               │  │
│  │  • Use: Separate subnets, dual-homed instances            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌─── Enhanced Networking (EN) ─────────────────────────────┐  │
│  │  • High performance: 10 Gbps — 100 Gbps                  │  │
│  │  • Uses SR-IOV (Single Root I/O Virtualization)           │  │
│  │  • Lower CPU utilization, higher PPS (packets/sec)        │  │
│  │  • No extra charge (supported instance types only)        │  │
│  │  • Use: High throughput, low latency between instances    │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌─── EFA (Elastic Fabric Adapter) ─────────────────────────┐  │
│  │  • Highest performance networking in AWS                  │  │
│  │  • OS-bypass: App talks directly to NIC (skips kernel)    │  │
│  │  • Ultra-low latency, massive parallel compute            │  │
│  │  • Use: HPC, Machine Learning, MPI workloads             │  │
│  │  • 🧠 Interview Tip: See HPC/ML → pick EFA               │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Quick Decision

```
Basic networking?           → ENI
High speed (10-100 Gbps)?   → Enhanced Networking
HPC / ML / OS-bypass?       → EFA
```

---

### IAM Roles for EC2 ✅ MUST HAVE

> NEVER put access keys on EC2. Always use IAM Roles.

```
┌─────────────────────────────────────────────────────────────┐
│          IAM ROLES FOR EC2 — THE RIGHT WAY                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ❌ BAD (Hard-coded credentials):                           │
│  ┌──────────────────────────────────┐                      │
│  │  EC2 Instance                    │                      │
│  │  ~/.aws/credentials              │                      │
│  │  aws_access_key_id = AKIAXXXX    │  ← SECURITY RISK!   │
│  │  aws_secret_access_key = xxxxx   │                      │
│  └──────────────────────────────────┘                      │
│                                                             │
│  ✅ GOOD (IAM Role attached):                               │
│  ┌──────────────────────────────────┐                      │
│  │  EC2 Instance                    │                      │
│  │  ┌───────────────────────┐       │                      │
│  │  │ IAM Role: S3ReadRole  │       │                      │
│  │  │ Policy: s3:GetObject  │───────┼──▶ S3 Bucket        │
│  │  └───────────────────────┘       │    (Secure access)   │
│  └──────────────────────────────────┘                      │
│                                                             │
│  Benefits:                                                  │
│  • No keys to rotate or leak                               │
│  • Can attach/detach to RUNNING instances (no restart!)     │
│  • Policy changes take effect IMMEDIATELY                   │
│  • Temporary credentials auto-rotated by AWS               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### AWS CLI Essentials ⚡ GOOD TO KNOW

| Best Practice | Detail |
|---------------|--------|
| **Least Privilege** | Give minimum access needed — no wildcard `*` in production |
| **Use Groups** | Assign permissions via groups, not individual users |
| **Secret Key = One view only** | Lose it → regenerate → re-run `aws configure` |
| **Don't share key pairs** | Each developer gets their own access key |
| **Prefer Roles over Keys** | On EC2? Use Role. In Lambda? Use Role. Always Role first. |
| **Supports all OS** | Linux, Windows, macOS + EC2 instances |

---

### AWS Outposts ⚡ GOOD TO KNOW

> Extends AWS infrastructure to your on-premises data center.

```
┌──────────────────────────────────────────────────────────────┐
│                    AWS OUTPOSTS                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─── Your Data Center ────────┐     ┌── AWS Cloud ──────┐ │
│  │                              │     │                    │ │
│  │  ┌────────────────────────┐ │     │  AWS Region        │ │
│  │  │  AWS Outpost Rack      │ │◄───▶│  (Full services)   │ │
│  │  │  (Same AWS APIs)       │ │     │                    │ │
│  │  │  • EC2                 │ │     │                    │ │
│  │  │  • EBS                 │ │     │                    │ │
│  │  │  • ECS/EKS            │ │     │                    │ │
│  │  │  • RDS                 │ │     │                    │ │
│  │  └────────────────────────┘ │     └────────────────────┘ │
│  └──────────────────────────────┘                            │
│                                                              │
│  Use when:                                                   │
│  • Data residency requirements (data must stay on-prem)      │
│  • Ultra-low latency to on-prem systems                      │
│  • Hybrid workloads needing consistent AWS APIs              │
│                                                              │
│  🧠 Interview Tip: "Extend AWS to data center" → Outposts   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### Production Best Practices (12 YOE Level)

```
┌─────────────────────────────────────────────────────────────┐
│              EC2 PRODUCTION CHECKLIST                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Use IAM Roles — NEVER hard-code credentials            │
│  ✅ Use Auto Scaling Groups (ASG) for high availability    │
│  ✅ Deploy across multiple AZs behind an ALB               │
│  ✅ Use Reserved Instances for baseline capacity           │
│  ✅ Use Spot Instances for burst/batch workloads           │
│  ✅ Security Group chaining (ALB → App → DB)              │
│  ✅ Use SSM Session Manager (no SSH port 22 needed!)       │
│  ✅ Enable detailed monitoring (CloudWatch 1-min)          │
│  ✅ Use Launch Templates (not Launch Configs)              │
│  ✅ Golden AMI + User Data for fast, consistent deploys    │
│  ✅ Enable EBS encryption by default in account settings   │
│  ✅ Use Placement Groups for performance-critical apps     │
│  ✅ Tag everything (cost allocation, automation)           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| EC2 Topic | Verdict | Why |
|-----------|---------|-----|
| Pricing Models (On-Demand/Reserved/Spot/Dedicated) | ✅ Must know cold | Scenario-based "which pricing?" questions guaranteed |
| Spot Instances & Spot Fleet strategies | ✅ Must know | Cost optimization at scale — senior-level topic |
| Security Groups (stateful, chaining) | ✅ Must know | Network design is core at your level |
| IAM Roles for EC2 | ✅ Must know | Security fundamentals — always asked |
| Networking (ENI/EN/EFA) | ✅ Know the differences | HPC/ML scenario questions |
| Bootstrap Scripts (User Data) | ⚡ Quick review | Simple concept, occasionally useful |
| Metadata endpoint | ⚡ Quick review | Know it exists, rarely deep-dived |
| AWS Outposts | ⚡ Know the concept | 1-2 questions max, just know when to pick it |
| Basic instance launch steps | ⏭️ Skip | Too basic for 12 YOE |


---

## 2. EBS (Elastic Block Store) ✅ MUST HAVE

> EBS is the "virtual hard disk" for EC2. At 12 YOE, know volume types by heart, encryption workflows, and snapshot strategies for production.

### What is EBS?

**Block-level storage** that attaches to EC2 instances — like plugging a hard drive into your VM.

```
┌─────────────────────────────────────────────────────────────┐
│                      EBS AT A GLANCE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  💾 Block storage (not object storage like S3)              │
│  🔗 Attaches to ONE EC2 instance at a time (except io2)    │
│  🏢 Lives within a SINGLE AZ                                │
│  📸 Snapshots stored in S3 (cross-region copy possible)     │
│  📈 Resize & change type ON THE FLY (no downtime!)          │
│  🔒 Encryption available (AES-256)                          │
│  📊 Highly available (replicated within AZ)                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### EBS Volume Types ✅ MUST HAVE

> Interview staple: "Which volume type for X workload?" — memorize IOPS and use cases.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          EBS VOLUME TYPES                                │
├──────────────────────── SSD (IOPS-focused) ─────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  gp2 — General Purpose SSD                                      │   │
│  │  • Up to 16,000 IOPS                                           │   │
│  │  • IOPS scales with volume size (3 IOPS/GiB)                   │   │
│  │  • 99.9% durability                                            │   │
│  │  • Use: Boot volumes, dev/test, general apps                    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  gp3 — General Purpose SSD (NEWER, PREFERRED)                   │   │
│  │  • Baseline: 3,000 IOPS + 125 MiB/s (regardless of size!)      │   │
│  │  • Up to 16,000 IOPS (independently configurable)              │   │
│  │  • 99.9% durability                                            │   │
│  │  • 20% cheaper than gp2                                        │   │
│  │  • Use: Most workloads — DEFAULT choice for new deployments     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  io1 — Provisioned IOPS SSD                                     │   │
│  │  • Up to 64,000 IOPS per volume                                 │   │
│  │  • 50 IOPS per GiB ratio                                       │   │
│  │  • 99.9% durability                                            │   │
│  │  • Use: Databases (Oracle, SQL Server), latency-sensitive       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  io2 — Provisioned IOPS SSD (LATEST GEN)                       │   │
│  │  • Up to 64,000 IOPS per volume                                 │   │
│  │  • 500 IOPS per GiB ratio (10x more than io1!)                 │   │
│  │  • 99.999% durability (5 nines!)                               │   │
│  │  • Multi-attach support (attach to multiple EC2)               │   │
│  │  • Use: Mission-critical databases, highest durability needed   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
├──────────────────────── HDD (Throughput-focused) ───────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  st1 — Throughput Optimized HDD                                 │   │
│  │  • Max throughput: 500 MB/s per volume                          │   │
│  │  • ❌ CANNOT be a boot volume                                   │   │
│  │  • 99.9% durability                                            │   │
│  │  • Use: Big data, data warehouses, ETL, log processing          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  sc1 — Cold HDD (Cheapest)                                      │   │
│  │  • Max throughput: 250 MB/s per volume                          │   │
│  │  • ❌ CANNOT be a boot volume                                   │   │
│  │  • Lowest cost of all EBS volumes                               │   │
│  │  • 99.9% durability                                            │   │
│  │  • Use: Infrequently accessed data, archival, cold storage      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Quick Comparison Table

| Type | Category | Max IOPS | Max Throughput | Boot? | Use Case |
|------|----------|----------|----------------|-------|----------|
| **gp3** | SSD | 16,000 | 1,000 MiB/s | ✅ | Default for most workloads |
| **gp2** | SSD | 16,000 | 250 MiB/s | ✅ | Legacy general purpose |
| **io1** | SSD | 64,000 | 1,000 MiB/s | ✅ | High-perf databases |
| **io2** | SSD | 64,000 | 1,000 MiB/s | ✅ | Mission-critical DB (5 nines) |
| **st1** | HDD | 500 | 500 MB/s | ❌ | Big data, ETL, warehouses |
| **sc1** | HDD | 250 | 250 MB/s | ❌ | Cold data, cheapest |

### Decision Flow

```
Need a BOOT volume?
├── YES → Must use SSD (gp2, gp3, io1, io2)
│         ├── General workload? → gp3 (default choice)
│         ├── Need > 16K IOPS? → io1 or io2
│         └── Need 99.999% durability? → io2
│
└── NO → Can use HDD
          ├── High throughput (big data/ETL)? → st1
          └── Lowest cost (cold data)? → sc1
```

---

### EBS Snapshots ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                   EBS SNAPSHOTS                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  EBS Volume (AZ-specific)          S3 (Region-level)        │
│  ┌──────────┐                     ┌──────────────┐         │
│  │ /dev/sda │───── Snapshot ────▶│  Snapshot    │         │
│  │ (us-e-1a)│     (point-in-time) │  (us-east-1) │         │
│  └──────────┘                     └──────┬───────┘         │
│                                          │                  │
│                              ┌───────────┼────────────┐     │
│                              ▼           ▼            ▼     │
│                        ┌────────┐  ┌────────┐  ┌────────┐ │
│                        │New Vol │  │Copy to │  │Share   │ │
│                        │(any AZ)│  │another │  │with    │ │
│                        │        │  │Region  │  │account │ │
│                        └────────┘  └────────┘  └────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Snapshot Key Facts

| Fact | Detail |
|------|--------|
| **Stored in S3** | Snapshots live in S3 (you don't see them in your buckets) |
| **Incremental** | Only changed blocks are saved after first snapshot |
| **First snapshot = slow** | Full copy of the volume — takes longer |
| **Consistent snapshots** | Best practice: Stop instance → detach volume → snapshot |
| **Cross-region copy** | Copy snapshot to another region for DR |
| **Cross-account sharing** | Share with other AWS accounts (must copy first) |
| **Resize on the fly** | Can change volume size & type without downtime |

### Production Example — DR with Snapshots

```
┌─── us-east-1 ────────────┐          ┌─── eu-west-1 ────────────┐
│                           │          │                           │
│  EC2 + EBS Volume         │          │  (Disaster Recovery)      │
│       │                   │          │                           │
│       ▼                   │          │                           │
│  Snapshot (automated,     │──Copy──▶│  Snapshot Copy            │
│  daily via AWS Backup)    │          │       │                   │
│                           │          │       ▼                   │
│                           │          │  Launch EC2 from          │
│                           │          │  snapshot (if DR needed)  │
└───────────────────────────┘          └───────────────────────────┘
```

---

### EBS vs Instance Store ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────┐
│              EBS vs INSTANCE STORE COMPARISON                     │
├────────────────────┬─────────────────────┬───────────────────────┤
│                    │      EBS            │   INSTANCE STORE      │
├────────────────────┼─────────────────────┼───────────────────────┤
│ Persistence        │ ✅ Persistent       │ ❌ Ephemeral          │
│ Survives stop?     │ ✅ Yes              │ ❌ No (data LOST)     │
│ Survives reboot?   │ ✅ Yes              │ ✅ Yes                │
│ Survives terminate?│ ❌ (unless config'd)│ ❌ No                 │
│ Host failure?      │ ✅ Data safe        │ ❌ Data LOST          │
│ Can stop instance? │ ✅ Yes              │ ❌ No (only terminate)│
│ Snapshot support?  │ ✅ Yes              │ ❌ No                 │
│ Root vol on term?  │ Deleted by default* │ Always deleted        │
│ Performance        │ Network-attached    │ Physically on host    │
│                    │                     │ (very fast I/O)       │
├────────────────────┴─────────────────────┴───────────────────────┤
│ * EBS: Can set "DeleteOnTermination = false" to keep root vol    │
└──────────────────────────────────────────────────────────────────┘
```

### When to Use Instance Store

```
✅ Temporary data (buffers, caches, scratch data)
✅ Data that's replicated at app level (HDFS, Cassandra)
✅ Need extremely high I/O performance (NVMe local SSDs)

❌ NEVER for databases (unless replicated)
❌ NEVER for data you can't afford to lose
```

---

### EBS Encryption ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│              EBS ENCRYPTION — WHAT'S ENCRYPTED               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  When you encrypt an EBS volume, you get:                   │
│                                                             │
│  ✅ Data at rest inside the volume — encrypted              │
│  ✅ Data in transit (instance ↔ volume) — encrypted         │
│  ✅ All snapshots from this volume — encrypted              │
│  ✅ All volumes created from those snapshots — encrypted    │
│                                                             │
│  Uses AWS KMS (AES-256) — minimal performance impact        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### How to Encrypt an Existing Unencrypted Volume

```
Step 1: Create snapshot of unencrypted volume
           │
Step 2: Copy the snapshot → select "Encrypt" option
           │
Step 3: Create AMI from encrypted snapshot
           │
Step 4: Launch new EC2 instance from encrypted AMI
           │
Result: New instance has fully encrypted root volume! 🔒
```

> 💡 **Pro Tip:** In production, enable "EBS encryption by default" at the account level. All new volumes will be automatically encrypted — no extra steps needed.

---

### AWS Backup ⚡ GOOD TO KNOW

> Centralized backup service — one place to manage backups across all AWS services.

```
┌─────────────────────────────────────────────────────────────┐
│                      AWS BACKUP                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              AWS BACKUP (Central Console)             │   │
│  │                                                      │   │
│  │  Backup Plans → Schedules → Lifecycle → Vault        │   │
│  └───────────────────────┬──────────────────────────────┘   │
│                          │                                   │
│          ┌───────┬───────┼───────┬───────┬───────┐          │
│          ▼       ▼       ▼       ▼       ▼       ▼          │
│       ┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐┌──────────┐    │
│       │ EC2 ││ EBS ││ EFS ││ RDS ││ FSx ││ Storage  │    │
│       │     ││     ││     ││     ││     ││ Gateway  │    │
│       └─────┘└─────┘└─────┘└─────┘└─────┘└──────────┘    │
│                                                             │
│  Key Features:                                              │
│  • Centralized control across services & accounts           │
│  • Works with AWS Organizations (multi-account)             │
│  • Define lifecycle policies (move to cold, delete after)   │
│  • Enforce encryption on all backups                        │
│  • Compliance auditing built-in                             │
│  • Cross-region backup for DR                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| EBS Topic | Verdict | Why |
|-----------|---------|-----|
| Volume Types (gp2/gp3/io1/io2/st1/sc1) | ✅ Must memorize | "Which volume for X?" is guaranteed in interviews |
| gp3 vs gp2 difference | ✅ Must know | gp3 is newer/cheaper — know why it's preferred |
| io2 (99.999% durability) | ✅ Must know | Mission-critical DB scenario questions |
| Snapshots (cross-region, cross-account) | ✅ Must know | DR strategy is core at senior level |
| EBS vs Instance Store | ✅ Must know | Classic comparison question |
| Encryption workflow | ✅ Must know | "How to encrypt existing unencrypted volume?" |
| AWS Backup | ⚡ Know it exists | Centralized backup — occasionally asked |
| Basic volume attach/detach | ⏭️ Skip | Too basic for 12 YOE |


---

---

# 🗄️ CATEGORY: DATABASE

---

## 1. RDS (Relational Database Service) ✅ MUST HAVE

> At 12 YOE, know Multi-AZ vs Read Replica cold. Expect questions like "How do you scale reads?" and "What happens during failover?"

### What is RDS?

A **managed relational database** — AWS handles patching, backups, failover. You focus on schema and queries.

```
┌─────────────────────────────────────────────────────────────┐
│                      RDS AT A GLANCE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Supported Engines:                                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│  │SQL Server│ │  Oracle  │ │  MySQL   │ │PostgreSQL│     │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘     │
│  ┌──────────┐ ┌──────────┐                                 │
│  │ MariaDB  │ │  Aurora  │                                 │
│  └──────────┘ └──────────┘                                 │
│                                                             │
│  ✅ OLTP (Online Transaction Processing)                    │
│     → Small, frequent transactions (orders, payments)       │
│                                                             │
│  ❌ NOT for OLAP (Online Analytical Processing)             │
│     → Use Redshift for analytics, reporting, warehousing    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### OLTP vs OLAP — Know the Difference

```
┌─────────────────────────────┬────────────────────────────────┐
│          OLTP               │           OLAP                 │
│    (RDS / Aurora)           │       (Redshift)               │
├─────────────────────────────┼────────────────────────────────┤
│ • Many small transactions   │ • Few complex queries          │
│ • INSERT, UPDATE, DELETE    │ • SELECT with aggregations     │
│ • Customer orders, banking  │ • Sales forecasting, BI        │
│ • Low latency per query     │ • Scans millions of rows       │
│ • Row-based storage         │ • Columnar storage             │
│                             │                                │
│ Example:                    │ Example:                       │
│ "Get order #12345"          │ "Total revenue by region       │
│ "Insert new payment"        │  for last 3 years"            │
└─────────────────────────────┴────────────────────────────────┘
```

> 🧠 **Interview Tip:** See "reporting" or "analytics" → Redshift. See "transactions" → RDS/Aurora.

---

### Multi-AZ vs Read Replicas ✅ MUST HAVE

> The #1 most-asked RDS question. Know the difference instantly.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MULTI-AZ vs READ REPLICA                              │
├──────────────────────────────────┬──────────────────────────────────────┤
│          MULTI-AZ                │         READ REPLICA                  │
│     (Disaster Recovery)          │      (Performance / Scaling)          │
├──────────────────────────────────┼──────────────────────────────────────┤
│                                  │                                      │
│  ┌─────────┐    ┌─────────┐    │  ┌─────────┐    ┌─────────┐         │
│  │ Primary │    │ Standby │    │  │ Primary │    │ Replica │         │
│  │  (R/W)  │───▶│  (Sync) │    │  │  (R/W)  │───▶│  (R/O)  │         │
│  │  AZ-1a  │    │  AZ-1b  │    │  │  AZ-1a  │    │  AZ-1b  │         │
│  └─────────┘    └─────────┘    │  └─────────┘    └─────────┘         │
│                                  │                                      │
│  • Synchronous replication       │  • Asynchronous replication          │
│  • Auto failover (DNS switch)    │  • NO auto failover (manual promote)│
│  • Standby NOT readable          │  • Replica IS readable              │
│  • Same region, different AZ     │  • Same AZ, cross-AZ, cross-region │
│  • Purpose: HIGH AVAILABILITY    │  • Purpose: SCALE READ PERFORMANCE  │
│                                  │                                      │
└──────────────────────────────────┴──────────────────────────────────────┘
```

### Comparison Table

| Feature | Multi-AZ | Read Replica |
|---------|----------|--------------|
| **Purpose** | Disaster Recovery (HA) | Scale read performance |
| **Replication** | Synchronous | Asynchronous |
| **Readable?** | ❌ Standby is NOT readable | ✅ Replica IS readable |
| **Failover** | ✅ Automatic (DNS flip) | ❌ Manual promotion |
| **Location** | Same region, different AZ | Same AZ / Cross-AZ / Cross-Region |
| **Number** | 1 standby | Up to 5 read replicas |
| **Use case** | "Keep my DB available" | "Offload BI/reporting reads" |

### Production Architecture — Both Together

```
                    ┌──────────────────────────────────────────┐
                    │           APPLICATION LAYER              │
                    └────────────────┬─────────────────────────┘
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
                    WRITES (R/W)           READS (R/O)
                         │                     │
                         ▼                     ▼
┌─── us-east-1a ────────────────┐   ┌─── us-east-1b ────────────────┐
│  ┌─────────────────────────┐  │   │  ┌─────────────────────────┐  │
│  │   PRIMARY (R/W)         │  │   │  │   READ REPLICA (R/O)    │  │
│  │   MySQL 8.0             │──┼───┼─▶│   (async replication)   │  │
│  └─────────────────────────┘  │   │  └─────────────────────────┘  │
│              │                 │   │                                │
│              │ Sync            │   └────────────────────────────────┘
│              ▼                 │
│  ┌─────────────────────────┐  │   ┌─── eu-west-1 (Cross-Region) ──┐
│  │   STANDBY (Multi-AZ)   │  │   │  ┌─────────────────────────┐  │
│  │   (auto failover)      │  │   │  │   READ REPLICA (DR)     │  │
│  └─────────────────────────┘  │   │  │   (can promote to primary)│ │
└───────────────────────────────┘   │  └─────────────────────────┘  │
                                     └───────────────────────────────┘

Best Practice: Multi-AZ for HA + Read Replicas for scale + Cross-region for DR
```

---

## 2. Amazon Aurora ✅ MUST HAVE

> Aurora is AWS's flagship database — faster, more durable, and auto-scales. Know the 6-copy architecture.

### What is Aurora?

AWS's MySQL/PostgreSQL-compatible database — 5x faster than MySQL, 3x faster than PostgreSQL.

```
┌─────────────────────────────────────────────────────────────┐
│                    AURORA ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Data Replication: 6 COPIES across 3 AZs (minimum)         │
│                                                             │
│  ┌─── AZ-1 ───┐   ┌─── AZ-2 ───┐   ┌─── AZ-3 ───┐      │
│  │  Copy 1    │   │  Copy 3    │   │  Copy 5    │      │
│  │  Copy 2    │   │  Copy 4    │   │  Copy 6    │      │
│  └────────────┘   └────────────┘   └────────────┘      │
│                                                             │
│  Fault Tolerance:                                           │
│  • Can lose 2 copies → still WRITE                          │
│  • Can lose 3 copies → still READ                           │
│  • Self-healing (auto-detects & repairs corrupt data)       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Aurora Key Features

| Feature | Detail |
|---------|--------|
| **6 copies of data** | 2 copies per AZ, minimum 3 AZs |
| **Up to 15 Aurora replicas** | Automated failover (priority tiers) |
| **Share snapshots** | Cross-account sharing supported |
| **Automated backups** | ON by default + manual snapshots |
| **Compatible** | MySQL and PostgreSQL |
| **Storage auto-scales** | 10 GB → 128 TB automatically |
| **Aurora Serverless** | Auto-scales compute for unpredictable workloads |

### Aurora Replica Types

| Replica Type | Max Count | Auto Failover? | Use Case |
|-------------|-----------|----------------|----------|
| **Aurora Replicas** | 15 | ✅ Yes | Production (preferred) |
| MySQL Replicas | 5 | ❌ No | Migration from MySQL |
| PostgreSQL Replicas | 5 | ❌ No | Migration from PostgreSQL |

> 🧠 **Interview Tip:** "Automated failover" → only Aurora Replicas. MySQL/PostgreSQL replicas do NOT support auto failover.

---

### Aurora Serverless ⚡ GOOD TO KNOW

```
┌─────────────────────────────────────────────────────────────┐
│                  AURORA SERVERLESS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── Traditional Aurora ──────┐  ┌─── Aurora Serverless ─┐│
│  │  You pick instance size     │  │  AWS auto-scales       ││
│  │  Pay even when idle         │  │  Scales to 0 (pauses)  ││
│  │  Manual scaling             │  │  Pay per second of use  ││
│  └─────────────────────────────┘  └────────────────────────┘│
│                                                             │
│  Best for:                                                  │
│  • Infrequent / intermittent / unpredictable workloads      │
│  • Dev/test databases (pauses when not in use)              │
│  • New apps with unknown demand                             │
│  • Variable traffic (spiky)                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. DynamoDB ✅ MUST HAVE

> DynamoDB is THE NoSQL answer on AWS. At 12 YOE, know when to choose DynamoDB over RDS, and understand provisioned vs on-demand.

### What is DynamoDB?

A **fully managed NoSQL** database — key-value + document store with single-digit millisecond latency at any scale.

```
┌─────────────────────────────────────────────────────────────┐
│                   DYNAMODB AT A GLANCE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ⚡ Single-digit millisecond latency at ANY scale            │
│  📊 Supports key-value AND document data models             │
│  ♾️  Virtually unlimited throughput and storage              │
│  🏗️  Fully managed (no servers, no patching)                │
│  📈 Auto-scales with on-demand mode                         │
│  🌍 Multi-region with Global Tables                          │
│  🔒 Encryption at rest by default                           │
│                                                             │
│  Great for:                                                 │
│  • Mobile & web apps       • IoT data                       │
│  • Gaming leaderboards     • Session stores                 │
│  • Ad-tech (real-time)     • Shopping carts                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### RDS vs DynamoDB — When to Use What

```
┌─────────────────────────────────┬─────────────────────────────────┐
│           RDS / Aurora          │          DynamoDB                │
├─────────────────────────────────┼─────────────────────────────────┤
│ Structured data (schema)        │ Semi-structured / flexible       │
│ Complex joins & queries         │ Simple key-based lookups         │
│ ACID transactions               │ Single-digit ms at any scale     │
│ Known relationships             │ Massive read/write throughput    │
│ Reporting with complex SQL      │ Unpredictable/spiky traffic      │
│                                 │                                  │
│ Example: Banking, ERP, CRM      │ Example: Gaming, IoT, sessions  │
└─────────────────────────────────┴─────────────────────────────────┘

Quick Rule:
• Need JOINs / complex queries? → RDS / Aurora
• Need speed + scale + flexible schema? → DynamoDB
```

---

## 4. Amazon Neptune ⚡ GOOD TO KNOW

> Specialized graph database. Only comes up if you see "graph" or "relationships" in scenarios.

```
┌─────────────────────────────────────────────────────────────┐
│                    AMAZON NEPTUNE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Fully managed GRAPH database                         │
│                                                             │
│  Data Model:                                                │
│  ┌──────┐──follows──▶┌──────┐──likes──▶┌──────────┐       │
│  │User A│            │User B│          │Product X │       │
│  └──────┘◀──friends──└──────┘          └──────────┘       │
│                                                             │
│  Use cases:                                                 │
│  • Social networks (friends, followers)                     │
│  • Fraud detection (find connected suspicious accounts)     │
│  • Knowledge graphs                                         │
│  • Recommendation engines                                   │
│  • Network topology mapping                                 │
│                                                             │
│  Supports: Gremlin (Apache TinkerPop) + SPARQL (RDF)        │
│                                                             │
│  🧠 Interview Tip: See "graph" or "relationships" → Neptune │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. Amazon Redshift ⚡ GOOD TO KNOW

> Already mentioned in RDS section — this is the OLAP warehouse. Quick reference:

```
┌─────────────────────────────────────────────────────────────┐
│                    AMAZON REDSHIFT                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Fully managed data WAREHOUSE (OLAP)                  │
│  Storage: Columnar (not row-based like RDS)                 │
│  Scale: Petabyte-scale                                      │
│                                                             │
│  Use when:                                                  │
│  • Analytical queries across huge datasets                  │
│  • BI reporting, sales forecasting                          │
│  • Historical data analysis                                 │
│  • Complex aggregations on millions of rows                 │
│                                                             │
│  🧠 Interview Tip: See "warehouse" or "OLAP" → Redshift    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Database Selection — Quick Decision Map

```
┌──────────────────────────────────────────────────────────────┐
│           WHICH DATABASE FOR WHICH SCENARIO?                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "Relational data, transactions"         → RDS              │
│  "MySQL/PostgreSQL but faster & HA"      → Aurora           │
│  "Serverless, unpredictable DB traffic"  → Aurora Serverless│
│  "NoSQL, key-value, millisecond speed"   → DynamoDB         │
│  "Analytics, BI, data warehouse"         → Redshift         │
│  "Graph data, social networks, fraud"    → Neptune          │
│  "In-memory caching"                     → ElastiCache      │
│  "Document database (MongoDB compat)"    → DocumentDB       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Database Topic | Verdict | Why |
|---------------|---------|-----|
| Multi-AZ vs Read Replica | ✅ Must know cold | #1 most-asked RDS question |
| Aurora architecture (6 copies, 3 AZs) | ✅ Must know | AWS flagship — expect deep questions |
| Aurora Serverless | ⚡ Know when to use | "Unpredictable workload" = Aurora Serverless |
| DynamoDB vs RDS decision | ✅ Must know | "Which DB?" scenario questions |
| DynamoDB deep features (DAX, Streams, Global Tables) | ✅ Must know | Will add when you share more notes |
| Redshift | ⚡ Know it's for OLAP | "Analytics/warehouse" → Redshift |
| Neptune | ⚡ Know it's for graphs | Only if you see "graph" in the question |
| OLTP vs OLAP distinction | ✅ Must know | Fundamental concept, always tested |


---

---

# 🌐 CATEGORY: NETWORKING

---

## 1. VPC (Virtual Private Cloud) ✅ MUST HAVE

> VPC is the foundation of ALL AWS networking. At 12 YOE, you must design VPCs, subnetting strategies, connectivity patterns, and hybrid architectures. Expect deep questions.

### What is a VPC?

A **logically isolated virtual data center** in AWS — your own private network where you control everything.

```
┌─────────────────────────────────────────────────────────────┐
│                      VPC AT A GLANCE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🏢 Think: "Your own data center in the cloud"              │
│  🧩 Components:                                             │
│     • Internet Gateway (IGW)                                │
│     • Virtual Private Gateway (VGW)                         │
│     • Route Tables                                          │
│     • NACLs (Network ACLs)                                  │
│     • Subnets                                               │
│     • Security Groups                                       │
│                                                             │
│  📍 1 Subnet = 1 AZ (always!)                               │
│  🔗 Can extend to on-prem via VPN or Direct Connect         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### VPC Architecture — Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                           VPC (10.0.0.0/16)                          │
│                                                                      │
│  ┌─── AZ-1a ─────────────────────┐  ┌─── AZ-1b ─────────────────┐ │
│  │                                │  │                            │ │
│  │  ┌─── Public Subnet ───────┐  │  │  ┌─── Public Subnet ────┐ │ │
│  │  │  10.0.1.0/24            │  │  │  │  10.0.3.0/24         │ │ │
│  │  │  • ALB                  │  │  │  │  • ALB               │ │ │
│  │  │  • NAT Gateway          │  │  │  │  • NAT Gateway       │ │ │
│  │  └─────────────────────────┘  │  │  └──────────────────────┘ │ │
│  │                                │  │                            │ │
│  │  ┌─── Private Subnet ──────┐  │  │  ┌─── Private Subnet ──┐ │ │
│  │  │  10.0.2.0/24            │  │  │  │  10.0.4.0/24        │ │ │
│  │  │  • EC2 App Servers      │  │  │  │  • EC2 App Servers  │ │ │
│  │  │  • RDS (primary)        │  │  │  │  • RDS (standby)    │ │ │
│  │  └─────────────────────────┘  │  │  └──────────────────────┘ │ │
│  │                                │  │                            │ │
│  └────────────────────────────────┘  └────────────────────────────┘ │
│                                                                      │
│  ┌───────────────┐                          ┌───────────────┐       │
│  │ Internet GW   │ (public traffic)         │ Virtual Priv  │       │
│  │ (IGW)         │                          │ Gateway (VGW) │       │
│  └───────┬───────┘                          └───────┬───────┘       │
└──────────┼──────────────────────────────────────────┼────────────────┘
           │                                          │
           ▼                                          ▼
      INTERNET                              ON-PREM DATA CENTER
                                            (via VPN/Direct Connect)
```

---

### NAT Gateway ✅ MUST HAVE

> Allows instances in PRIVATE subnets to reach the internet (for updates, patches) without being publicly accessible.

```
┌─────────────────────────────────────────────────────────────┐
│                    NAT GATEWAY                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  INTERNET                                                   │
│     ▲                                                       │
│     │                                                       │
│  ┌──┴──────────────┐                                       │
│  │ Internet Gateway │                                       │
│  └──┬──────────────┘                                       │
│     │                                                       │
│  ┌──▼──────────────────────┐  (Public Subnet)              │
│  │      NAT Gateway        │                                │
│  │  • Auto-assigned public IP                               │
│  │  • 5 Gbps → scales to 45 Gbps                           │
│  │  • Redundant within AZ                                   │
│  │  • No patching needed                                    │
│  │  • No security group association                         │
│  └──┬──────────────────────┘                                │
│     │                                                       │
│  ┌──▼──────────────────────┐  (Private Subnet)             │
│  │  EC2 Instance           │                                │
│  │  (can reach internet    │                                │
│  │   but NOT reachable     │                                │
│  │   from internet)        │                                │
│  └─────────────────────────┘                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### NAT Gateway — High Availability Pattern

```
⚠️ PROBLEM: NAT Gateway is AZ-specific. If that AZ goes down,
            instances in other AZs lose internet access!

✅ SOLUTION: One NAT Gateway PER AZ

┌─── AZ-1a ─────────────┐    ┌─── AZ-1b ─────────────┐
│  ┌────────────────┐   │    │  ┌────────────────┐   │
│  │  NAT Gateway A │   │    │  │  NAT Gateway B │   │
│  └───────┬────────┘   │    │  └───────┬────────┘   │
│          │             │    │          │             │
│  ┌───────▼────────┐   │    │  ┌───────▼────────┐   │
│  │  Private       │   │    │  │  Private       │   │
│  │  Instances     │   │    │  │  Instances     │   │
│  └────────────────┘   │    │  └────────────────┘   │
└────────────────────────┘    └────────────────────────┘

Route table AZ-1a: 0.0.0.0/0 → NAT-GW-A
Route table AZ-1b: 0.0.0.0/0 → NAT-GW-B
```

---

### Security Groups vs NACLs ✅ MUST HAVE

> Classic interview question: "What's the difference?" — Know this table cold.

```
┌──────────────────────────────────────────────────────────────────────┐
│            SECURITY GROUPS vs NETWORK ACLs (NACLs)                   │
├───────────────────────┬──────────────────────┬───────────────────────┤
│                       │   SECURITY GROUP     │        NACL           │
├───────────────────────┼──────────────────────┼───────────────────────┤
│ Level                 │ Instance level       │ Subnet level          │
│ State                 │ STATEFUL             │ STATELESS             │
│ Rules                 │ ALLOW only           │ ALLOW and DENY        │
│ Rule evaluation       │ All rules evaluated  │ Rules in NUMBER order │
│ Default inbound       │ DENY all             │ ALLOW all (default)   │
│                       │                      │ DENY all (custom)     │
│ Default outbound      │ ALLOW all            │ ALLOW all (default)   │
│                       │                      │ DENY all (custom)     │
│ Block specific IP?    │ ❌ Cannot            │ ✅ Can (use DENY rule)│
│ Applies to            │ Only if assigned     │ All instances in subnet│
│ Association           │ Multiple per instance│ 1 NACL per subnet     │
│                       │ Multiple instances   │ 1 NACL → many subnets │
└───────────────────────┴──────────────────────┴───────────────────────┘
```

### Stateful vs Stateless — Visual

```
STATEFUL (Security Group):
┌────────────────────────────────────────┐
│  Inbound: Allow port 80 from 0.0.0.0  │
│  Outbound: (doesn't matter)           │
│                                        │
│  Request IN on port 80 → ✅ Allowed    │
│  Response OUT → ✅ Auto-allowed!       │
│  (because it "remembers" the request)  │
└────────────────────────────────────────┘

STATELESS (NACL):
┌────────────────────────────────────────┐
│  Inbound: Allow port 80               │
│  Outbound: ??? (must explicitly allow!)│
│                                        │
│  Request IN on port 80 → ✅ Allowed    │
│  Response OUT → ❌ BLOCKED unless you  │
│  also allow ephemeral ports outbound!  │
└────────────────────────────────────────┘
```

> 🧠 **Interview Tip:** "Block a specific IP address" → use NACL (not Security Group). Security Groups have no DENY rule.

---

### VPC Endpoints ✅ MUST HAVE

> Access AWS services (S3, DynamoDB, etc.) from private subnets WITHOUT going through the internet.

```
┌─────────────────────────────────────────────────────────────┐
│                    VPC ENDPOINTS                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  WITHOUT Endpoint:                                          │
│  EC2 (private) → NAT GW → IGW → Internet → S3             │
│  (traffic leaves AWS network! 💸 costs more, less secure)   │
│                                                             │
│  WITH Endpoint:                                             │
│  EC2 (private) → VPC Endpoint → S3                         │
│  (stays within AWS network! ✅ free, fast, secure)          │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  TWO TYPES:                                                 │
│                                                             │
│  ┌─── Gateway Endpoint ────────────────────────────────┐   │
│  │  • Supports: S3 and DynamoDB ONLY                    │   │
│  │  • Added to route table (like a route entry)         │   │
│  │  • FREE!                                             │   │
│  │  • 🧠 S3 or DynamoDB? → Gateway Endpoint            │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─── Interface Endpoint (PrivateLink) ────────────────┐   │
│  │  • Supports: MOST other AWS services                 │   │
│  │  • Uses ENI (private IP in your subnet)              │   │
│  │  • Costs $ (hourly + per-GB)                         │   │
│  │  • Example: CloudWatch, SQS, SNS, KMS, etc.         │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### VPC Peering ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                     VPC PEERING                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Connect 2 VPCs using PRIVATE IP addresses                  │
│                                                             │
│    ┌─────────┐              ┌─────────┐                    │
│    │  VPC A  │── Peering ──│  VPC B  │                    │
│    │10.0.0/16│  Connection  │10.1.0/16│                    │
│    └─────────┘              └─────────┘                    │
│                                                             │
│  Rules:                                                     │
│  ✅ Can peer across regions (inter-region peering)          │
│  ✅ Can peer across accounts                               │
│  ❌ NO transitive peering!                                 │
│  ❌ NO overlapping CIDR ranges                             │
│                                                             │
│  ⚠️ NO TRANSITIVE PEERING:                                 │
│                                                             │
│    VPC-A ←──peer──→ VPC-B ←──peer──→ VPC-C                │
│    VPC-A ✘ CANNOT reach VPC-C through VPC-B!               │
│    Must create direct peering: VPC-A ←──peer──→ VPC-C      │
│                                                             │
│  This creates a "hub-and-spoke" or "full mesh" mess        │
│  at scale → Solution: TRANSIT GATEWAY                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### AWS PrivateLink ✅ MUST HAVE

> Expose your service to THOUSANDS of customer VPCs securely — without VPC peering, route tables, or NAT.

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS PRIVATELINK                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── Service Provider VPC ──┐    ┌─── Customer VPC ────┐ │
│  │                            │    │                      │ │
│  │  ┌────────────────────┐   │    │  ┌──────────────┐   │ │
│  │  │  Your Application  │   │    │  │     ENI      │   │ │
│  │  └────────┬───────────┘   │    │  │ (private IP) │   │ │
│  │           │               │    │  └──────┬───────┘   │ │
│  │  ┌────────▼───────────┐   │    │         │           │ │
│  │  │        NLB         │───┼────┼─────────┘           │ │
│  │  │(Network Load Bal.) │   │    │                      │ │
│  │  └────────────────────┘   │    │  Customer accesses   │ │
│  │                            │    │  via private IP!     │ │
│  └────────────────────────────┘    └──────────────────────┘ │
│                                                             │
│  Requirements:                                              │
│  • Service side: Network Load Balancer (NLB)                │
│  • Customer side: ENI (Elastic Network Interface)           │
│                                                             │
│  🧠 Interview Tip: "Expose service to thousands of VPCs"   │
│     → PrivateLink (NOT VPC Peering)                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Direct Connect vs VPN ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────────┐
│                  DIRECT CONNECT vs VPN                                │
├─────────────────────────────────┬────────────────────────────────────┤
│          VPN (Site-to-Site)     │       DIRECT CONNECT               │
├─────────────────────────────────┼────────────────────────────────────┤
│                                 │                                    │
│  ┌────────┐  ~~~Internet~~~     │  ┌────────┐                       │
│  │On-Prem │──encrypted tunnel──▶│  │On-Prem │──dedicated fiber────▶│
│  └────────┘  (IPsec over web)   │  └────────┘  (private line)      │
│                                 │                                    │
│  • Over PUBLIC internet         │  • Dedicated PRIVATE connection   │
│  • Encrypted (IPsec)            │  • Consistent latency            │
│  • Quick to set up (minutes)    │  • Takes weeks/months to set up  │
│  • Bandwidth limited by ISP     │  • 1 Gbps or 10 Gbps or 100 Gbps│
│  • Variable latency             │  • Great for massive throughput  │
│  • Cheaper                      │  • More expensive                │
│  • Good for: quick/temp needs   │  • Good for: production hybrid  │
│                                 │                                    │
└─────────────────────────────────┴────────────────────────────────────┘

🧠 Interview Tip:
• "Needs to be set up quickly" → VPN
• "Needs stable, high throughput, low latency" → Direct Connect
• "Encrypted private connection" → Direct Connect + VPN (DX doesn't encrypt by default)
```

---

### AWS VPN CloudHub ⚡ GOOD TO KNOW

```
┌─────────────────────────────────────────────────────────────┐
│                   AWS VPN CLOUDHUB                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Connect MULTIPLE sites together via hub-and-spoke          │
│                                                             │
│       Site A ─────┐                                        │
│                    ▼                                        │
│       Site B ──▶ AWS VPN ◀── Site C                        │
│                 CloudHub                                     │
│       Site D ─────┘                                        │
│                                                             │
│  • All traffic encrypted                                    │
│  • Operates over public internet                            │
│  • Low cost, easy to manage                                 │
│  • Hub-and-spoke model                                      │
│  • Each site has its own VPN connection                     │
│                                                             │
│  Use when: Multiple branch offices need to communicate      │
│  via AWS as the hub                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Transit Gateway ✅ MUST HAVE

> The solution to VPC Peering's "no transitive routing" problem. Central hub for all networking.

```
┌─────────────────────────────────────────────────────────────────────┐
│                      TRANSIT GATEWAY                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  WITHOUT Transit Gateway (nightmare at scale):                      │
│                                                                     │
│   VPC-A ──── VPC-B                                                  │
│     │  \   /  │        Full mesh peering = N*(N-1)/2 connections!   │
│     │   \ /   │        10 VPCs = 45 peering connections 😱          │
│     │    X    │                                                     │
│     │   / \   │                                                     │
│   VPC-C ──── VPC-D                                                  │
│                                                                     │
│  WITH Transit Gateway (simple, scalable):                           │
│                                                                     │
│        VPC-A   VPC-B   VPC-C   VPC-D                               │
│          │       │       │       │                                  │
│          └───────┴───┬───┴───────┘                                  │
│                      │                                              │
│              ┌───────▼───────┐                                      │
│              │   TRANSIT     │                                      │
│              │   GATEWAY     │                                      │
│              │  (Cloud       │                                      │
│              │   Router)     │                                      │
│              └───────┬───────┘                                      │
│                      │                                              │
│          ┌───────────┼───────────┐                                  │
│          ▼           ▼           ▼                                  │
│     On-Prem      VPN Conn.   Direct                                │
│     DC (VPN)                 Connect                                │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│  Key Features:                                                      │
│  ✅ Transitive routing (VPC-A can reach VPC-C through TGW)          │
│  ✅ Hub-and-spoke model                                             │
│  ✅ Works with VPN + Direct Connect                                 │
│  ✅ Cross-region (inter-region peering)                             │
│  ✅ Cross-account via RAM (Resource Access Manager)                 │
│  ✅ Route tables to control VPC-to-VPC communication                │
│  ✅ Supports IP multicast (unique — no other AWS service does!)     │
│  ✅ Thousands of VPCs + on-prem connections                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

> 🧠 **Interview Tip:** "Simplify complex VPC peering" or "transitive routing" or "connect thousands of VPCs" → Transit Gateway

---

### VPC Networking — Complete Decision Map

```
┌──────────────────────────────────────────────────────────────────┐
│          VPC CONNECTIVITY — WHICH SERVICE TO USE?                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Connect 2 VPCs privately"                                      │
│       → VPC Peering (simple, 1-to-1)                            │
│                                                                  │
│  "Connect many VPCs + on-prem (transitive)"                      │
│       → Transit Gateway                                          │
│                                                                  │
│  "Expose service to thousands of customer VPCs"                  │
│       → AWS PrivateLink (NLB + ENI)                             │
│                                                                  │
│  "Quick encrypted connection to on-prem"                         │
│       → Site-to-Site VPN                                        │
│                                                                  │
│  "High throughput, stable, dedicated connection"                  │
│       → Direct Connect                                          │
│                                                                  │
│  "Connect multiple branch offices via AWS"                       │
│       → VPN CloudHub                                            │
│                                                                  │
│  "Access S3/DynamoDB from private subnet"                        │
│       → Gateway Endpoint (free!)                                │
│                                                                  │
│  "Access other AWS services from private subnet"                 │
│       → Interface Endpoint (PrivateLink)                        │
│                                                                  │
│  "Block a specific IP"                                           │
│       → NACL (not Security Group!)                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| VPC Topic | Verdict | Why |
|-----------|---------|-----|
| VPC architecture (subnets, route tables, IGW) | ✅ Must design them | Foundation of everything |
| NAT Gateway (HA pattern) | ✅ Must know | "One per AZ" pattern always asked |
| Security Groups vs NACLs | ✅ Must know cold | Stateful vs Stateless, block IP = NACL |
| VPC Endpoints (Gateway vs Interface) | ✅ Must know | "Private access to S3" = Gateway Endpoint |
| VPC Peering (no transitive!) | ✅ Must know | Leads to Transit Gateway question |
| Transit Gateway | ✅ Must know | Senior-level networking, scalable design |
| AWS PrivateLink | ✅ Must know | "Expose to thousands of VPCs" = PrivateLink |
| Direct Connect vs VPN | ✅ Must know | Hybrid architecture is expected at your level |
| VPN CloudHub | ⚡ Know concept | Multi-site VPN hub — occasionally asked |
| Basic subnet/CIDR creation | ⏭️ Skip drilling | You know this already |


---

## 2. Route 53 (DNS Service) ✅ MUST HAVE

> At 12 YOE, know all routing policies and when to use each. Interviewers love "Which routing policy for X scenario?" questions.

### What is Route 53?

Amazon's **DNS service** — register domains, create hosted zones, and route users to your infrastructure.

```
┌─────────────────────────────────────────────────────────────┐
│                    ROUTE 53 AT A GLANCE                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📛 Named after Route 66 + DNS port 53                      │
│  🌐 Register domain names                                   │
│  📋 Create hosted zones (public & private)                  │
│  🔀 Route traffic with various routing policies             │
│  ❤️ Health checks on endpoints                              │
│  🔔 SNS alerts on health check failures                     │
│  🌍 100% SLA (the only AWS service with this!)              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Route 53 Routing Policies ✅ MUST HAVE

> 7 routing policies — know the use case for each one.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ROUTE 53 ROUTING POLICIES                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │   SIMPLE    │  │  WEIGHTED   │  │  LATENCY    │  │  FAILOVER   │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │
│                                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                    │
│  │GEOLOCATION  │  │GEOPROXIMITY │  │ MULTIVALUE  │                    │
│  └─────────────┘  └─────────────┘  └─────────────┘                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Each Policy Explained

---

#### 1. Simple Routing

```
┌─────────────────────────────────────────────────┐
│              SIMPLE ROUTING                      │
├─────────────────────────────────────────────────┤
│                                                 │
│  User ──▶ Route 53 ──▶ Returns ALL IPs randomly│
│                                                 │
│  • 1 record with multiple IP addresses          │
│  • Route 53 returns all values randomly         │
│  • NO health checks                            │
│  • Simplest policy                              │
│                                                 │
│  Use: Single resource, no fancy routing needed  │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

#### 2. Weighted Routing

```
┌─────────────────────────────────────────────────┐
│             WEIGHTED ROUTING                     │
├─────────────────────────────────────────────────┤
│                                                 │
│                    Route 53                      │
│                       │                         │
│              ┌────────┴─────────┐               │
│              │                  │               │
│         80% traffic        20% traffic          │
│              │                  │               │
│              ▼                  ▼               │
│       ┌──────────┐      ┌──────────┐           │
│       │us-east-1 │      │us-west-1 │           │
│       └──────────┘      └──────────┘           │
│                                                 │
│  Use: A/B testing, gradual deployment (canary)  │
│       Blue/Green deployments                    │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

#### 3. Latency-Based Routing

```
┌─────────────────────────────────────────────────┐
│           LATENCY-BASED ROUTING                  │
├─────────────────────────────────────────────────┤
│                                                 │
│  User (London) ──▶ Route 53                     │
│                       │                         │
│         Measures latency to all regions         │
│              │                  │               │
│         50ms to              280ms to           │
│         eu-west-2           ap-southeast-2      │
│              │                                  │
│              ▼                                  │
│       ┌──────────┐                             │
│       │eu-west-2 │  ← Lowest latency WINS     │
│       └──────────┘                             │
│                                                 │
│  Use: Global apps needing best performance      │
│       Route users to nearest/fastest region     │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

#### 4. Failover Routing

```
┌─────────────────────────────────────────────────┐
│            FAILOVER ROUTING                      │
├─────────────────────────────────────────────────┤
│                                                 │
│  Route 53 + Health Check                        │
│       │                                         │
│       ├── Primary healthy? ──▶ eu-west-2 ✅     │
│       │                                         │
│       └── Primary FAILED? ──▶ ap-southeast-2    │
│                                (DR site)        │
│                                                 │
│  ┌────────────────────────────────────────┐    │
│  │  ACTIVE          PASSIVE               │    │
│  │  (Primary)       (DR / Secondary)      │    │
│  │  eu-west-2  ──▶  ap-southeast-2       │    │
│  │  Health ✅        Standby              │    │
│  └────────────────────────────────────────┘    │
│                                                 │
│  Use: Active/passive DR setup                   │
│       Automatic failover to backup site         │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

#### 5. Geolocation Routing

```
┌─────────────────────────────────────────────────┐
│           GEOLOCATION ROUTING                    │
├─────────────────────────────────────────────────┤
│                                                 │
│  Route based on USER'S PHYSICAL LOCATION        │
│                                                 │
│  🇪🇺 European user ──▶ eu-west-1 (Ireland)      │
│  🇺🇸 US user ──────────▶ us-east-1 (Virginia)   │
│  🇦🇺 Australia user ──▶ ap-southeast-2 (Sydney) │
│                                                 │
│  Use:                                           │
│  • Content localization (language, currency)    │
│  • Compliance (data must stay in region)        │
│  • Restrict access by country                   │
│                                                 │
│  ⚠️ Based on WHERE user IS                      │
│     (not where latency is lowest!)              │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

#### 6. Geoproximity Routing

```
┌─────────────────────────────────────────────────┐
│          GEOPROXIMITY ROUTING                    │
├─────────────────────────────────────────────────┤
│                                                 │
│  Routes based on user location + resource       │
│  location + BIAS (expand/shrink regions)        │
│                                                 │
│  Without bias:          With bias (+25 on US):  │
│  ┌─────┬─────┐         ┌────────┬──┐          │
│  │ EU  │ US  │         │   US   │EU│          │
│  │     │     │         │(bigger)│  │          │
│  └─────┴─────┘         └────────┴──┘          │
│                                                 │
│  Bias > 0: Expand region (attract more traffic) │
│  Bias < 0: Shrink region (repel traffic)        │
│                                                 │
│  ⚠️ Requires Route 53 TRAFFIC FLOW             │
│                                                 │
│  Use: Fine-tune traffic distribution between    │
│       regions beyond simple geolocation         │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

#### 7. Multivalue Answer Routing

```
┌─────────────────────────────────────────────────┐
│         MULTIVALUE ANSWER ROUTING                │
├─────────────────────────────────────────────────┤
│                                                 │
│  Like Simple Routing... BUT with HEALTH CHECKS! │
│                                                 │
│  Route 53 returns up to 8 healthy IPs randomly  │
│                                                 │
│  ┌────────────────────────────────────────┐    │
│  │  IP-1  Health ✅  → returned           │    │
│  │  IP-2  Health ✅  → returned           │    │
│  │  IP-3  Health ❌  → NOT returned       │    │
│  │  IP-4  Health ✅  → returned           │    │
│  └────────────────────────────────────────┘    │
│                                                 │
│  Use: Simple load distribution with health      │
│       (not a substitute for real load balancer) │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

### Quick Comparison Table

| Policy | Routes Based On | Health Check? | Use Case |
|--------|----------------|---------------|----------|
| **Simple** | Random (all IPs) | ❌ No | Single resource, basic |
| **Weighted** | % split you define | ✅ Yes | A/B testing, canary deploys |
| **Latency** | Lowest latency to user | ✅ Yes | Global apps, best performance |
| **Failover** | Primary vs DR | ✅ Yes (required) | Active/passive DR |
| **Geolocation** | User's physical location | ✅ Yes | Compliance, localization |
| **Geoproximity** | Location + bias | ✅ Yes | Fine-tuned traffic steering |
| **Multivalue** | Random (healthy only) | ✅ Yes | Simple routing + health |

---

### Health Checks ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                  ROUTE 53 HEALTH CHECKS                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Route 53 monitors your endpoints:                          │
│                                                             │
│  ┌────────────────┐         ┌──────────────────────┐       │
│  │   Route 53     │──ping──▶│   Your Endpoint      │       │
│  │  Health Check  │  (every │   (EC2, ALB, IP)     │       │
│  │                │  30s or │                      │       │
│  │                │  10s)   │   Response: 2xx/3xx? │       │
│  └────────────────┘         └──────────────────────┘       │
│         │                                                   │
│         ├── Healthy ✅ → Include in DNS responses           │
│         │                                                   │
│         └── Unhealthy ❌ → Remove from DNS responses        │
│                           → Trigger SNS notification 🔔     │
│                                                             │
│  Types:                                                     │
│  • Endpoint checks (HTTP/HTTPS/TCP)                         │
│  • Calculated checks (combine child checks)                 │
│  • CloudWatch alarm checks                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Production Example — Global App with Route 53

```
┌──────────────────────────────────────────────────────────────────────┐
│          PRODUCTION: GLOBAL APP ROUTING STRATEGY                     │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Domain: app.example.com                                             │
│                                                                      │
│  Strategy: Latency-based routing + Failover per region               │
│                                                                      │
│  User (London) ──▶ Route 53 (Latency) ──▶ eu-west-1 (50ms)        │
│  User (NYC)    ──▶ Route 53 (Latency) ──▶ us-east-1 (20ms)        │
│                                                                      │
│  Within each region: Failover routing                                │
│                                                                      │
│  eu-west-1:                                                          │
│    Primary: ALB-EU (Health ✅) ──▶ traffic goes here                │
│    Secondary: ALB-US (Failover) ──▶ used if EU fails                │
│                                                                      │
│  Health checks on every ALB endpoint                                 │
│  SNS alerts if any region goes unhealthy                             │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Route 53 Topic | Verdict | Why |
|---------------|---------|-----|
| All 7 routing policies | ✅ Must know use case for each | "Which policy for X?" is guaranteed |
| Failover routing | ✅ Must know deeply | DR architecture design |
| Latency vs Geolocation vs Geoproximity | ✅ Know the difference | Tricky — they sound similar but differ |
| Health checks | ✅ Must know | Foundation of all advanced routing |
| Weighted (canary/blue-green) | ✅ Must know | Deployment strategy questions |
| Simple vs Multivalue | ⚡ Quick review | Just know Multivalue = Simple + health checks |
| Geoproximity + Traffic Flow | ⚡ Know concept | Rarely deep-dived, but know bias concept |


---

## 3. ELB (Elastic Load Balancing) ✅ MUST HAVE

> At 12 YOE, know ALB vs NLB cold — when to use each, listener/target group architecture, and sticky sessions. Classic LB is legacy but still asked.

### What is ELB?

Distributes incoming traffic across multiple targets (EC2, containers, IPs, Lambda) to ensure high availability.

```
┌─────────────────────────────────────────────────────────────┐
│                      ELB AT A GLANCE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  3 Types of Load Balancers:                                 │
│                                                             │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────┐ │
│  │    ALB           │  │      NLB         │  │   CLB    │ │
│  │ (Application)    │  │   (Network)      │  │(Classic) │ │
│  │  Layer 7         │  │   Layer 4        │  │ Legacy   │ │
│  │  HTTP/HTTPS      │  │   TCP/UDP/TLS    │  │ L4 + L7  │ │
│  └──────────────────┘  └──────────────────┘  └──────────┘ │
│                                                             │
│  All ELBs support:                                          │
│  ✅ Health checks on targets                                │
│  ✅ Cross-zone load balancing                               │
│  ✅ CloudWatch metrics                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### ALB vs NLB vs CLB — Comparison ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    ELB TYPE COMPARISON                                    │
├───────────────────┬─────────────────────┬──────────────────┬─────────────┤
│                   │   ALB (Application) │  NLB (Network)   │ CLB(Classic)│
├───────────────────┼─────────────────────┼──────────────────┼─────────────┤
│ OSI Layer         │ Layer 7 (HTTP)      │ Layer 4 (TCP)    │ Layer 4 + 7 │
│ Protocols         │ HTTP, HTTPS         │ TCP, UDP, TLS    │ TCP, HTTP   │
│ Performance       │ Good                │ Extreme (millions│ Basic       │
│                   │                     │ req/s, ultra-low)│             │
│ Static IP         │ ❌ No (use DNS)     │ ✅ Yes (Elastic  │ ❌ No       │
│                   │                     │    IP per AZ)    │             │
│ Path-based routing│ ✅ Yes              │ ❌ No            │ ❌ No       │
│ Host-based routing│ ✅ Yes              │ ❌ No            │ ❌ No       │
│ WebSockets        │ ✅ Yes              │ ✅ Yes           │ ❌ No       │
│ SSL Termination   │ ✅ Yes              │ ✅ Yes           │ ✅ Yes      │
│ Sticky Sessions   │ ✅ (target group)   │ ❌ No            │ ✅ (instance)│
│ Target Types      │ Instance, IP,       │ Instance, IP,    │ Instance    │
│                   │ Lambda              │ ALB              │ only        │
│ Use Case          │ Web apps, micro-    │ Extreme perf,    │ Legacy apps │
│                   │ services, API       │ gaming, IoT,     │ (avoid new) │
│                   │ routing             │ non-HTTP protos  │             │
├───────────────────┴─────────────────────┴──────────────────┴─────────────┤
│ 🧠 DEFAULT CHOICE: ALB for most web workloads                            │
│    NLB only when you need: extreme speed, static IP, or non-HTTP         │
│    CLB: NEVER use for new projects (legacy only)                         │
└──────────────────────────────────────────────────────────────────────────┘
```

---

### Application Load Balancer (ALB) — Deep Dive ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   ALB ARCHITECTURE                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Users                                                                  │
│    │                                                                    │
│    ▼                                                                    │
│  ┌────────────────────────────────────────────────────────────────┐    │
│  │                    ALB (Layer 7)                                │    │
│  │                                                                 │    │
│  │  ┌──── LISTENER (port 443/HTTPS) ──────────────────────────┐  │    │
│  │  │                                                          │  │    │
│  │  │  ┌─── RULES ────────────────────────────────────────┐   │  │    │
│  │  │  │                                                   │   │  │    │
│  │  │  │  IF path = /api/*    → Target Group: API Servers  │   │  │    │
│  │  │  │  IF path = /images/* → Target Group: Static CDN   │   │  │    │
│  │  │  │  IF host = admin.*   → Target Group: Admin App    │   │  │    │
│  │  │  │  DEFAULT             → Target Group: Web Servers  │   │  │    │
│  │  │  │                                                   │   │  │    │
│  │  │  └───────────────────────────────────────────────────┘   │  │    │
│  │  └──────────────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────────────┘    │
│                          │          │          │          │             │
│                          ▼          ▼          ▼          ▼             │
│                    ┌─────────┐┌─────────┐┌─────────┐┌─────────┐       │
│  TARGET GROUPS:    │API TG   ││Static TG││Admin TG ││Web TG   │       │
│                    │(EC2×3)  ││(EC2×2)  ││(EC2×1)  ││(EC2×4)  │       │
│                    └─────────┘└─────────┘└─────────┘└─────────┘       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### ALB Key Concepts

| Component | What It Does |
|-----------|-------------|
| **Listener** | Checks for incoming connections on a specific port/protocol (e.g., HTTPS:443) |
| **Rules** | Conditions that decide WHERE to route (path, host, headers, query strings) |
| **Target Groups** | Collection of targets (EC2, IPs, Lambda) that receive the routed traffic |
| **SSL/TLS** | Must deploy certificate (ACM) on ALB for HTTPS. ALB terminates SSL. |

### ALB Limitations

```
⚠️ ALB supports HTTP and HTTPS ONLY
   Need TCP/UDP? → Use NLB
   Need non-HTTP protocols (MQTT, gaming)? → Use NLB
```

---

### Network Load Balancer (NLB) — Deep Dive ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                 NLB (Layer 4) — WHEN TO USE                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Extreme performance (millions of requests/sec)          │
│  ✅ Ultra-low latency (~100 microseconds)                   │
│  ✅ Static IP / Elastic IP required                         │
│  ✅ Non-HTTP protocols (TCP, UDP, TLS)                      │
│  ✅ Whitelisting by IP (ALB IPs change, NLB IPs don't)     │
│  ✅ PrivateLink (NLB required for VPC endpoint services)    │
│                                                             │
│  Can decrypt TLS traffic (install cert on NLB)              │
│                                                             │
│  Use cases:                                                 │
│  • Gaming servers                                           │
│  • IoT (millions of devices)                                │
│  • Financial trading (ultra-low latency)                    │
│  • VoIP, streaming                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Classic Load Balancer (CLB) — Legacy ⚡ GOOD TO KNOW

```
┌─────────────────────────────────────────────────────────────┐
│              CLB — LEGACY (Avoid for new projects)           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  • Supports both Layer 4 (TCP) and Layer 7 (HTTP/HTTPS)     │
│  • NO target groups, NO path-based routing                  │
│  • Sticky sessions at INSTANCE level                        │
│  • X-Forwarded-For header for client IP                     │
│                                                             │
│  Common Error:                                              │
│  ┌──────────────────────────────────────────────────┐      │
│  │  504 Gateway Timeout                              │      │
│  │  = Application not responding within timeout      │      │
│  │  Troubleshoot: Is it web server or DB server?     │      │
│  └──────────────────────────────────────────────────┘      │
│                                                             │
│  X-Forwarded-For Header:                                    │
│  ┌──────────┐      ┌──────┐      ┌──────────┐             │
│  │  Client  │─────▶│ CLB  │─────▶│   EC2    │             │
│  │12.34.56.7│      │      │      │          │             │
│  └──────────┘      └──────┘      └──────────┘             │
│                                   Sees CLB IP, but         │
│                                   X-Forwarded-For:          │
│                                   12.34.56.78              │
│                                   (original client IP)     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Sticky Sessions ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                   STICKY SESSIONS                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: User always routed to SAME instance                  │
│                                                             │
│  Without Sticky:           With Sticky:                     │
│  User → Instance 1        User → Instance 2 (always!)     │
│  User → Instance 3                                          │
│  User → Instance 2        (cookie binds user to target)    │
│  (random distribution)                                      │
│                                                             │
│  Where it applies:                                          │
│  • CLB: Sticky at instance level                            │
│  • ALB: Sticky at TARGET GROUP level                        │
│  • NLB: N/A                                                 │
│                                                             │
│  Use when: App stores session data locally on instance      │
│                                                             │
│  ⚠️ COMMON PROBLEM:                                        │
│  "EC2 removed from pool but still gets traffic"             │
│  SOLUTION → Disable sticky sessions!                        │
│                                                             │
│  💡 BETTER APPROACH: Don't use sticky sessions.             │
│  Store sessions in ElastiCache/DynamoDB instead.            │
│  (Stateless architecture = easier scaling)                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Deregistration Delay (Connection Draining) ⚡ GOOD TO KNOW

```
┌─────────────────────────────────────────────────────────────┐
│              DEREGISTRATION DELAY                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What happens when an instance is marked unhealthy          │
│  or being removed from the target group?                    │
│                                                             │
│  ┌─── ENABLED (default: 300 seconds) ─────────────────┐   │
│  │  • Existing connections stay OPEN until they finish  │   │
│  │  • No NEW connections sent to this instance          │   │
│  │  • Graceful shutdown                                 │   │
│  │  • Use: Long-running requests (file uploads, etc.)   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─── DISABLED ───────────────────────────────────────┐    │
│  │  • ALL connections closed IMMEDIATELY                │    │
│  │  • Instant removal                                   │    │
│  │  • Use: Short-lived requests, rapid deployments      │    │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### ELB Decision Flow

```
┌──────────────────────────────────────────────────────────────┐
│              WHICH LOAD BALANCER TO USE?                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "Web app, HTTP/HTTPS, path/host routing"                    │
│       → ALB ✅                                              │
│                                                              │
│  "Microservices with different URL paths"                     │
│       → ALB (path-based routing to different target groups) │
│                                                              │
│  "Extreme performance, millions req/s, ultra-low latency"    │
│       → NLB                                                 │
│                                                              │
│  "Need static IP or Elastic IP"                              │
│       → NLB                                                 │
│                                                              │
│  "TCP/UDP/non-HTTP protocols"                                │
│       → NLB                                                 │
│                                                              │
│  "VPC Endpoint Service (PrivateLink)"                        │
│       → NLB (required!)                                     │
│                                                              │
│  "Legacy app, can't migrate"                                 │
│       → CLB (but plan migration to ALB/NLB)                 │
│                                                              │
│  "504 error"                                                 │
│       → App not responding. Check web server / DB.           │
│                                                              │
│  "Need client's real IP"                                     │
│       → X-Forwarded-For header (ALB/CLB)                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| ELB Topic | Verdict | Why |
|-----------|---------|-----|
| ALB vs NLB (when to use each) | ✅ Must know cold | Most common ELB question |
| ALB architecture (Listener → Rules → Target Groups) | ✅ Must know | Production microservice design |
| NLB use cases (static IP, extreme perf, PrivateLink) | ✅ Must know | Key differentiator from ALB |
| Sticky Sessions + problems | ✅ Must know | "Traffic stuck to one instance" scenario |
| Health Checks | ✅ Must know | Foundation of HA |
| Deregistration Delay | ⚡ Know concept | Graceful shutdown during deployments |
| CLB (504, X-Forwarded-For) | ⚡ Quick review | Legacy, but still in question banks |
| SSL termination on ALB/NLB | ✅ Must know | Security + architecture questions |


---

---

# 📊 CATEGORY: MONITORING & LOGGING

---

## 1. CloudWatch ✅ MUST HAVE

> At 12 YOE, CloudWatch is your observability backbone. Know metrics vs logs vs alarms, agent setup, and when to use CloudWatch vs third-party tools (Grafana, Prometheus).

### What is CloudWatch?

AWS's **monitoring and observability platform** — metrics, logs, alarms, dashboards all in one place.

```
┌─────────────────────────────────────────────────────────────┐
│                   CLOUDWATCH AT A GLANCE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🧠 "Anytime monitoring comes up → think CloudWatch"        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                  CLOUDWATCH                          │   │
│  │                                                      │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │   │
│  │  │ METRICS  │  │  LOGS    │  │  ALARMS  │         │   │
│  │  │(numbers) │  │(text data)│  │(triggers)│         │   │
│  │  └──────────┘  └──────────┘  └──────────┘         │   │
│  │                                                      │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │   │
│  │  │DASHBOARDS│  │  EVENTS  │  │INSIGHTS  │         │   │
│  │  │(visuals) │  │(triggers)│  │(query SQL)│         │   │
│  │  └──────────┘  └──────────┘  └──────────┘         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### CloudWatch Metrics — Two Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CLOUDWATCH METRICS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─── SYSTEM METRICS (out of the box) ─────────────────────────┐   │
│  │                                                              │   │
│  │  What AWS can see FROM OUTSIDE the instance (hypervisor):    │   │
│  │  • CPU Utilization                                           │   │
│  │  • Network In/Out                                            │   │
│  │  • Disk Read/Write (EBS)                                     │   │
│  │  • Status Checks                                             │   │
│  │                                                              │   │
│  │  ⚠️ AWS CANNOT see past the hypervisor!                     │   │
│  │     No memory usage, no disk space %, no process info        │   │
│  │                                                              │   │
│  │  Interval: Standard = 5 min | Detailed = 1 min ($)          │   │
│  │                                                              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─── APPLICATION METRICS (requires CloudWatch Agent) ─────────┐   │
│  │                                                              │   │
│  │  What you get by INSTALLING the agent on EC2:                │   │
│  │  • Memory utilization (RAM %)                                │   │
│  │  • Disk space usage (% full)                                 │   │
│  │  • Number of running processes                               │   │
│  │  • Custom application metrics                                │   │
│  │  • Log files from inside the instance                        │   │
│  │                                                              │   │
│  │  ⚠️ Agent is NOT automatic — must install & configure!      │   │
│  │                                                              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Rule: More MANAGED the service → More metrics out of the box       │
│  (Lambda gives you everything; EC2 needs agent for most)            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### What You Get vs What You Don't (EC2)

| ✅ Out of the Box (System) | ❌ Needs Agent (Application) |
|---------------------------|------------------------------|
| CPU utilization | **Memory usage** |
| Network In/Out | **Disk space %** |
| Disk I/O (EBS) | Process count |
| Status checks | Custom app metrics |
| — | Log files |

> 🧠 **Interview Tip:** "How to monitor memory on EC2?" → Install CloudWatch Agent. It's NOT available by default!

---

### CloudWatch Alarms ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                   CLOUDWATCH ALARMS                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ⚠️ NO default alarms exist — you must CREATE every one!   │
│                                                             │
│  Flow:                                                      │
│                                                             │
│  Metric (e.g., CPU > 80%)                                   │
│       │                                                     │
│       ▼                                                     │
│  ┌──────────┐    Threshold     ┌──────────────────────┐    │
│  │  ALARM   │────breached────▶│  ACTION               │    │
│  │          │                  │  • SNS notification   │    │
│  │          │                  │  • Auto Scaling       │    │
│  │          │                  │  • EC2 action (stop/  │    │
│  │          │                  │    terminate/reboot)  │    │
│  └──────────┘                  └──────────────────────┘    │
│                                                             │
│  Alarm States:                                              │
│  • OK — metric is within threshold                          │
│  • ALARM — metric breached threshold                        │
│  • INSUFFICIENT_DATA — not enough data yet                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Metric Intervals

| Type | Interval | Cost | Use Case |
|------|----------|------|----------|
| **Standard** | Every 5 minutes | Free | Most workloads |
| **Detailed** | Every 1 minute | $ | Auto Scaling, critical apps |
| **High Resolution** | Every 1 second | $$ | Custom metrics, ultra-sensitive |

---

### CloudWatch Logs ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                   CLOUDWATCH LOGS                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Monitor, store, and query log files from:                  │
│                                                             │
│  Sources:                                                   │
│  ┌───────────────────────────────────────────────────────┐ │
│  │  • EC2 instances (via agent)                           │ │
│  │  • Lambda functions (automatic!)                       │ │
│  │  • CloudTrail (API audit logs)                        │ │
│  │  • Route 53 (DNS query logs)                          │ │
│  │  • VPC Flow Logs                                      │ │
│  │  • ECS / EKS containers                               │ │
│  │  • API Gateway                                        │ │
│  │  • RDS (slow query logs, error logs)                  │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                             │
│  Structure:                                                 │
│  Log Group → Log Stream → Log Events                        │
│  (container)  (instance)   (individual log lines)           │
│                                                             │
│  Where should logs go?                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Need to PROCESS/QUERY/ALERT? → CloudWatch Logs     │   │
│  │  Just need to STORE (no processing)? → S3 directly  │   │
│  │  Need REAL-TIME processing? → Kinesis Data Streams  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### CloudWatch Logs Insights

```
┌─────────────────────────────────────────────────────────────┐
│              CLOUDWATCH LOGS INSIGHTS                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SQL-like query language to search & analyze logs           │
│                                                             │
│  Example Query:                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ fields @timestamp, @message                          │  │
│  │ | filter @message like /ERROR/                       │  │
│  │ | sort @timestamp desc                              │  │
│  │ | limit 20                                          │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  🧠 Interview Tip: "SQL" + "logs" → CloudWatch Logs Insights│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Decision Flow — Where Do Logs/Metrics Go?

```
┌──────────────────────────────────────────────────────────────┐
│           MONITORING DECISION MAP                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "Monitor AWS resource metrics"                              │
│       → CloudWatch Metrics                                  │
│                                                              │
│  "Alert when metric breaches threshold"                      │
│       → CloudWatch Alarms → SNS / Auto Scaling              │
│                                                              │
│  "Collect & query application logs"                          │
│       → CloudWatch Logs                                     │
│                                                              │
│  "SQL-like log analysis"                                     │
│       → CloudWatch Logs Insights                            │
│                                                              │
│  "Just store logs cheaply (no processing)"                   │
│       → S3 directly                                         │
│                                                              │
│  "Real-time log/event streaming"                             │
│       → Kinesis Data Streams                                │
│                                                              │
│  "Monitor memory / disk on EC2"                              │
│       → CloudWatch Agent (must install!)                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Amazon Managed Grafana ⚡ GOOD TO KNOW

> AWS's managed Grafana for visualization. Know when to pick it over CloudWatch dashboards.

```
┌─────────────────────────────────────────────────────────────┐
│              AMAZON MANAGED GRAFANA                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Fully managed data VISUALIZATION service             │
│                                                             │
│  ┌────────── Data Sources ──────────────────────┐          │
│  │  • Amazon CloudWatch                         │          │
│  │  • Amazon Managed Prometheus                 │          │
│  │  • AWS X-Ray                                 │          │
│  │  • Amazon Elasticsearch                      │          │
│  │  • Amazon Timestream                         │          │
│  │  • Many more (30+ built-in)                  │          │
│  └──────────────────────────────────────────────┘          │
│           │                                                 │
│           ▼                                                 │
│  ┌──────────────────────────────────────────────┐          │
│  │         Beautiful Dashboards                  │          │
│  │   (query, correlate, visualize across         │          │
│  │    metrics, logs, and traces in one place)    │          │
│  └──────────────────────────────────────────────┘          │
│                                                             │
│  Use cases:                                                 │
│  • Container metrics (EKS, ECS, self-hosted K8s)            │
│  • IoT edge device monitoring                               │
│  • Operational teams needing rich dashboards                 │
│  • Multi-source correlation                                 │
│                                                             │
│  ✅ Fully managed (HA, scaling handled by AWS)              │
│  ✅ VPC endpoints for secure access                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Amazon Managed Service for Prometheus ⚡ GOOD TO KNOW

> Know this if you're running containers (EKS/K8s). It's the metric collection layer that feeds Grafana.

```
┌─────────────────────────────────────────────────────────────┐
│           AMAZON MANAGED PROMETHEUS                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Serverless, Prometheus-compatible monitoring          │
│        for container metrics at scale                        │
│                                                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │                                                     │    │
│  │  EKS Cluster ──scrape──▶ Managed Prometheus         │    │
│  │  (or self-managed K8s)     │                        │    │
│  │                            │ (stores metrics)       │    │
│  │                            ▼                        │    │
│  │                     Managed Grafana                  │    │
│  │                     (visualize)                      │    │
│  │                                                     │    │
│  └────────────────────────────────────────────────────┘    │
│                                                             │
│  Key points:                                                │
│  • Serverless (no infrastructure to manage)                 │
│  • Auto-scales                                              │
│  • Compatible with PromQL (standard Prometheus queries)     │
│  • Works with EKS, ECS, self-managed K8s                    │
│  • Secure via VPC endpoints                                 │
│                                                             │
│  🧠 Interview Tip:                                          │
│  "Container monitoring at scale" → Managed Prometheus       │
│  "Visualize container metrics" → Managed Grafana            │
│  Together they're the AWS "Prometheus + Grafana" stack      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Monitoring Stack — How They Fit Together

```
┌──────────────────────────────────────────────────────────────────────┐
│            AWS MONITORING ECOSYSTEM                                   │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─── COLLECT ───────────────────────────────────────────────────┐  │
│  │  CloudWatch Agent → EC2 metrics/logs                           │  │
│  │  CloudWatch (built-in) → AWS service metrics                   │  │
│  │  Managed Prometheus → Container metrics (EKS/K8s)              │  │
│  │  X-Ray → Distributed tracing                                   │  │
│  │  CloudTrail → API audit logs (WHO did WHAT)                    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                           │                                          │
│                           ▼                                          │
│  ┌─── ANALYZE ──────────────────────────────────────────────────┐   │
│  │  CloudWatch Logs Insights → SQL-like log queries              │   │
│  │  CloudWatch Metrics → Dashboards & graphs                     │   │
│  │  Managed Grafana → Rich multi-source visualization            │   │
│  └───────────────────────────────────────────────────────────────┘  │
│                           │                                          │
│                           ▼                                          │
│  ┌─── ACT ──────────────────────────────────────────────────────┐   │
│  │  CloudWatch Alarms → SNS / Auto Scaling / Lambda              │   │
│  │  EventBridge → Trigger workflows on events                    │   │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Monitoring Topic | Verdict | Why |
|-----------------|---------|-----|
| CloudWatch Metrics (system vs app) | ✅ Must know | "How to monitor memory?" = agent |
| CloudWatch Alarms | ✅ Must know | Foundation of automated response |
| CloudWatch Logs + Insights | ✅ Must know | Log analysis and troubleshooting |
| CloudWatch Agent | ✅ Must know | Required for EC2 OS-level metrics |
| Standard vs Detailed monitoring | ✅ Must know | 5 min vs 1 min — affects scaling speed |
| Managed Grafana | ⚡ Know when to use | Container visualization, multi-source |
| Managed Prometheus | ⚡ Know when to use | EKS/K8s metric collection at scale |
| CloudWatch vs Kinesis for logs | ✅ Must know | Real-time = Kinesis, normal = CW Logs |
| "Logs to S3 vs CloudWatch" decision | ✅ Must know | Process = CW Logs, Store only = S3 |


---

## 2. Auto Scaling ✅ MUST HAVE

> At 12 YOE, you're expected to DESIGN scaling strategies — scaling policies, launch templates, warm pools, predictive scaling. This is core production knowledge.

### Horizontal vs Vertical Scaling

```
┌─────────────────────────────────────────────────────────────────────┐
│           SCALING APPROACHES                                        │
├────────────────────────────────┬────────────────────────────────────┤
│     VERTICAL (Scale Up)        │     HORIZONTAL (Scale Out)         │
├────────────────────────────────┼────────────────────────────────────┤
│                                │                                    │
│  ┌──────────┐                  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐    │
│  │          │                  │  │ EC2│ │ EC2│ │ EC2│ │ EC2│    │
│  │  BIGGER  │                  │  └────┘ └────┘ └────┘ └────┘    │
│  │ INSTANCE │                  │     Add MORE instances             │
│  │          │                  │                                    │
│  │ (t2.micro│                  │  ┌────┐ ┌────┐                    │
│  │  → c5.4xl│                  │  │ EC2│ │ EC2│                    │
│  │ arge)    │                  │  └────┘ └────┘                    │
│  └──────────┘                  │     Remove instances               │
│                                │                                    │
│  • Has limits (max size)       │  • Virtually unlimited             │
│  • Requires restart            │  • No downtime                     │
│  • Single point of failure     │  • Fault tolerant                  │
│                                │  • AWS recommended ✅               │
└────────────────────────────────┴────────────────────────────────────┘

Production Rule: Always prefer HORIZONTAL scaling (Auto Scaling Groups)
```

---

### Launch Templates ✅ MUST HAVE

> Blueprint for EC2 instances — what AMI, what size, what security groups, etc.

```
┌─────────────────────────────────────────────────────────────┐
│                   LAUNCH TEMPLATE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What's inside a Launch Template:                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  • AMI (Amazon Machine Image)                        │   │
│  │  • EC2 Instance Type (size: t3.medium, c5.xlarge)    │   │
│  │  • Security Groups                                   │   │
│  │  • Key Pair                                          │   │
│  │  • User Data (bootstrap scripts)                     │   │
│  │  • Networking (subnet, VPC) — optional               │   │
│  │  • IAM Instance Profile (role)                       │   │
│  │  • EBS volumes                                       │   │
│  │  • Tags                                              │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Think: "Everything the EC2 wizard asks = Launch Template"  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Launch Template vs Launch Configuration

```
┌──────────────────────────────────────────────────────────────────────┐
│          LAUNCH TEMPLATE vs LAUNCH CONFIGURATION                     │
├───────────────────────────────┬──────────────────────────────────────┤
│       LAUNCH TEMPLATE ✅      │     LAUNCH CONFIGURATION ❌          │
│       (USE THIS)              │     (LEGACY — avoid)                 │
├───────────────────────────────┼──────────────────────────────────────┤
│ AWS recommended               │ Older version                        │
│ Supports VERSIONING           │ Immutable (can't edit, must recreate)│
│ More granular settings        │ Limited options                      │
│ Includes networking info      │ NO networking info                   │
│ Used for: Auto Scaling +      │ Used for: Auto Scaling ONLY          │
│   Spot Fleet + manual launch  │                                      │
│ Can mix instance types        │ Single instance type                 │
│ Supports T2/T3 Unlimited      │ Doesn't support                     │
└───────────────────────────────┴──────────────────────────────────────┘

🧠 Interview Tip: ALWAYS pick Launch Template over Launch Configuration.
   If answer choices have both, pick Template.
```

---

### Auto Scaling Groups (ASG) ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   AUTO SCALING GROUP (ASG)                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─── Configuration ──────────────────────────────────────────────┐    │
│  │                                                                 │    │
│  │  Launch Template: what to launch                                │    │
│  │  VPC + Subnets: WHERE to launch (multi-AZ!)                     │    │
│  │  Load Balancer: instances register to target group              │    │
│  │  Health Checks: EC2 or ELB health checks                       │    │
│  │  SNS Notifications: alert on scale events                       │    │
│  │                                                                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─── The 3 Critical Settings ────────────────────────────────────┐    │
│  │                                                                 │    │
│  │     MINIMUM          DESIRED          MAXIMUM                   │    │
│  │        │                │                │                      │    │
│  │        ▼                ▼                ▼                      │    │
│  │   ┌────────┐      ┌────────┐      ┌────────┐                  │    │
│  │   │  Min: 2 │      │ Des: 4 │      │Max: 10 │                  │    │
│  │   └────────┘      └────────┘      └────────┘                  │    │
│  │                                                                 │    │
│  │   "Never go       "Try to keep    "Never exceed                │    │
│  │    below this"     this many"      this count"                 │    │
│  │                                                                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ASG auto-balances instances across ALL configured AZs                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### ASG Architecture — Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│                        ┌──────────────┐                              │
│  Users ──────────────▶│     ALB      │                              │
│                        └──────┬───────┘                              │
│                               │                                      │
│                    ┌──────────┴──────────┐                           │
│                    ▼                     ▼                            │
│  ┌─── AZ-1a ─────────────┐  ┌─── AZ-1b ─────────────┐             │
│  │                        │  │                        │             │
│  │  ┌────┐  ┌────┐       │  │  ┌────┐  ┌────┐       │             │
│  │  │EC2 │  │EC2 │       │  │  │EC2 │  │EC2 │       │             │
│  │  └────┘  └────┘       │  │  └────┘  └────┘       │             │
│  │                        │  │                        │             │
│  └────────────────────────┘  └────────────────────────┘             │
│                                                                      │
│  ◄─────────── AUTO SCALING GROUP ──────────────────────▶            │
│                                                                      │
│  CloudWatch Alarm (CPU > 70%) ──▶ SCALE OUT (add instances)         │
│  CloudWatch Alarm (CPU < 30%) ──▶ SCALE IN (remove instances)       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Auto Scaling Policies ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────────────┐
│                   SCALING POLICY TYPES                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─── Target Tracking (Simplest, Recommended) ─────────────────┐   │
│  │  "Keep CPU at 50%"                                           │   │
│  │  ASG auto-adds/removes instances to maintain target          │   │
│  │  Like a thermostat — set it and forget it                    │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─── Step Scaling ────────────────────────────────────────────┐   │
│  │  "If CPU > 60% → add 2 | If CPU > 80% → add 4"             │   │
│  │  Different actions for different severity levels             │   │
│  │  More control, more configuration                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─── Simple Scaling ──────────────────────────────────────────┐   │
│  │  "If CPU > 70% → add 1 instance, then wait (cooldown)"      │   │
│  │  Oldest type, waits for cooldown before next action          │   │
│  │  Less responsive — avoid for production                      │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─── Scheduled Scaling ───────────────────────────────────────┐   │
│  │  "Every Monday 9am → scale to 10 instances"                  │   │
│  │  For predictable load patterns (office hours, events)        │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─── Predictive Scaling ──────────────────────────────────────┐   │
│  │  ML-based: AWS learns your traffic pattern                   │   │
│  │  Pre-provisions instances BEFORE traffic arrives             │   │
│  │  Best for recurring, predictable patterns                    │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Production Scaling Best Practices ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────────────┐
│            AUTO SCALING — PRODUCTION RULES                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  🚀 SCALE OUT AGGRESSIVELY                                    │ │
│  │  • Don't wait too long — get ahead of the workload            │ │
│  │  • Better to over-provision briefly than to drop requests     │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  🐢 SCALE IN CONSERVATIVELY                                   │ │
│  │  • Once instances are up, slowly roll them back               │ │
│  │  • Avoid thrashing (scale out → in → out → in)               │ │
│  │  • Use longer cooldown periods for scale-in                   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  🍳 BAKE YOUR AMIs                                            │ │
│  │  • Pre-install software in AMI (Golden AMI)                   │ │
│  │  • Minimize bootstrap time (don't install everything at boot) │ │
│  │  • Faster provisioning = faster response to load              │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  💰 OPTIMIZE COSTS                                            │ │
│  │  • Use Reserved Instances for MINIMUM count                   │ │
│  │  • Use Spot/On-Demand for instances above minimum             │ │
│  │  • Example: Min=2 (Reserved), scales to Max=10 (Spot/OD)     │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  📊 CLOUDWATCH IS YOUR TRIGGER                                │ │
│  │  • CloudWatch Alarms trigger scaling policies                 │ │
│  │  • Use detailed monitoring (1-min) for faster response        │ │
│  │  • Custom metrics (request queue depth, etc.) for smarter     │ │
│  │    scaling                                                     │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Cost-Optimized Scaling — Diagram

```
┌──────────────────────────────────────────────────────────────┐
│         PRODUCTION COST STRATEGY                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Instances: │████████████████████████████████│               │
│             │  Reserved   │ On-Demand/Spot   │               │
│             │  (Min: 2)   │ (Scales to Max)  │               │
│             │  (cheapest) │ (pay as needed)  │               │
│             │             │                  │               │
│             ├─────────────┼──────────────────┤               │
│             0     Min=2   Desired=4    Max=10                │
│                                                              │
│  Reserved Instances cover your BASELINE                      │
│  On-Demand/Spot covers your BURST                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Auto Scaling Topic | Verdict | Why |
|-------------------|---------|-----|
| Launch Template (what's inside) | ✅ Must know | Foundation for ASG — always asked |
| Template vs Configuration | ✅ Must know | "Which one?" → always Template |
| ASG (Min/Desired/Max) | ✅ Must know | Core concept |
| Scaling policies (Target/Step/Scheduled/Predictive) | ✅ Must know | Design scalable architecture |
| Scale Out Aggressively, Scale In Conservatively | ✅ Must know | Production mindset question |
| Golden AMI + fast provisioning | ✅ Must know | Expected at senior level |
| Cost optimization (RI for min + Spot for burst) | ✅ Must know | Cost-aware architecture |
| CloudWatch as trigger | ✅ Must know | Alarms → Scaling |
| Horizontal vs Vertical | ⚡ Quick review | Simple concept, know to prefer horizontal |


---

---

# 🔗 CATEGORY: DECOUPLING & MESSAGING

---

## 1. SQS (Simple Queue Service) ✅ MUST HAVE

> At 12 YOE, know SQS inside-out — it's the foundation of decoupled architectures. Queue depth for auto-scaling, DLQs for error handling, FIFO for ordering.

### What is SQS?

A **fully managed message queue** — decouple components by letting one service write messages and another read them asynchronously.

```
┌─────────────────────────────────────────────────────────────┐
│                      SQS AT A GLANCE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐       ┌──────────────┐       ┌──────────┐   │
│  │ Producer │──────▶│  SQS QUEUE   │──────▶│ Consumer │   │
│  │(writes)  │       │  (messages)  │       │ (reads)  │   │
│  └──────────┘       └──────────────┘       └──────────┘   │
│                                                             │
│  • Asynchronous processing                                  │
│  • Decouples components (Producer doesn't wait for Consumer)│
│  • Pull-based (Consumer POLLS the queue)                    │
│  • At-least-once delivery (Standard)                        │
│  • Exactly-once delivery (FIFO)                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### SQS Settings — Quick Reference

| Setting | Default | Range | Notes |
|---------|---------|-------|-------|
| **Delivery Delay** | 0 seconds | 0 → 15 minutes | Delay before message becomes visible |
| **Message Size** | — | Up to 256 KB | Any text format (JSON, XML, etc.) |
| **Encryption** | In-transit ✅ | + At-rest (KMS) optional | Enable SSE-KMS for at-rest |
| **Retention** | 4 days | 1 minute → 14 days | After this, message is deleted |
| **Visibility Timeout** | 30 seconds | 0 → 12 hours | Time consumer has to process |
| **Long Polling** | OFF (short) | 1 → 20 seconds | **Always use long polling!** (saves $) |

### Long Polling vs Short Polling

```
┌─────────────────────────────────────────────────────────────┐
│         LONG POLLING vs SHORT POLLING                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SHORT POLLING (default — bad):                             │
│  Consumer: "Any messages?" → Empty response                 │
│  Consumer: "Any messages?" → Empty response                 │
│  Consumer: "Any messages?" → Empty response                 │
│  Consumer: "Any messages?" → "Here's one!"                  │
│  💸 You PAY for all those empty responses!                  │
│                                                             │
│  LONG POLLING (recommended — good):                         │
│  Consumer: "Any messages? I'll wait up to 20 sec..."        │
│  ...(waits)...                                              │
│  → "Here's a message!" (responds when available)            │
│  💰 FEWER API calls = LESS cost + FASTER response           │
│                                                             │
│  🧠 Always enable Long Polling (set WaitTimeSeconds > 0)   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### SQS Standard vs FIFO ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────────┐
│                 SQS STANDARD vs FIFO                                  │
├───────────────────────────────┬──────────────────────────────────────┤
│       STANDARD                │           FIFO                       │
├───────────────────────────────┼──────────────────────────────────────┤
│ Unlimited throughput          │ 300 TPS (or 3000 with batching)      │
│ At-least-once delivery        │ Exactly-once delivery                │
│ Best-effort ordering          │ GUARANTEED order (first in, first out│
│ May get duplicates            │ No duplicates (deduplication)        │
│ Cheaper                       │ More expensive (dedup compute)       │
│                               │ Queue name must end in .fifo         │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│ 🧠 Interview Tip:                                                    │
│ "Message ordering" → FIFO                                            │
│ "Exactly-once processing" → FIFO                                     │
│ "Maximum throughput" → Standard                                      │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Dead-Letter Queue (DLQ) ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│                   DEAD-LETTER QUEUE (DLQ)                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: A separate SQS queue that receives FAILED messages   │
│                                                             │
│  ┌──────────┐     ┌───────────┐     ┌──────────┐          │
│  │ Producer │────▶│ Main Queue│────▶│ Consumer │          │
│  └──────────┘     └─────┬─────┘     └──────────┘          │
│                         │                                   │
│                         │ (after X failed attempts)         │
│                         ▼                                   │
│                   ┌──────────┐                              │
│                   │   DLQ    │ ← Just a regular SQS queue! │
│                   │(inspect, │                              │
│                   │ debug,   │                              │
│                   │ retry)   │                              │
│                   └──────────┘                              │
│                                                             │
│  Key facts:                                                 │
│  • DLQ is just a normal SQS queue (nothing special)         │
│  • Same 14-day max retention as any queue                   │
│  • Set up CloudWatch Alarm on DLQ depth! 🔔                │
│  • Works with SNS topics too                                │
│  • Use for debugging: inspect why messages failed           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### SQS + Auto Scaling (Queue Depth Trigger)

```
┌──────────────────────────────────────────────────────────────┐
│         SQS QUEUE DEPTH → AUTO SCALING                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐     ┌─────────────────┐     ┌─────────────┐  │
│  │Producers │────▶│   SQS Queue     │────▶│  ASG (EC2)  │  │
│  │(100 msgs/│     │   Depth: 5000   │     │  Workers    │  │
│  │ second)  │     └────────┬────────┘     └──────┬──────┘  │
│  └──────────┘              │                     │          │
│                            ▼                     │          │
│                   CloudWatch Alarm               │          │
│                   (depth > 1000)                  │          │
│                            │                     │          │
│                            └─── Scale OUT ───────┘          │
│                                                              │
│  🧠 Queue Depth is an excellent custom metric for scaling!  │
│     More messages waiting = need more workers               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. SNS (Simple Notification Service) ✅ MUST HAVE

> SNS is the PUSH counterpart to SQS (pull). Know the subscriber types and how SNS + SQS work together (Fan-Out pattern).

### What is SNS?

A **push-based pub/sub messaging service** — publishes messages to all subscribed endpoints simultaneously.

```
┌─────────────────────────────────────────────────────────────┐
│                      SNS AT A GLANCE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│               ┌──────────┐                                  │
│               │SNS TOPIC │                                  │
│               └─────┬────┘                                  │
│                     │ (PUSH to all subscribers)              │
│          ┌──────┬───┼───┬──────┬──────┐                    │
│          ▼      ▼   ▼   ▼      ▼      ▼                    │
│       ┌─────┐┌───┐┌───┐┌────┐┌─────┐┌─────┐              │
│       │ SQS ││λ  ││HTTP││Email││SMS ││Kinesis│             │
│       │     ││   ││(S) ││    ││    ││Firehose│            │
│       └─────┘└───┘└───┘└────┘└─────┘└─────┘              │
│                                                             │
│  Key: ONE message → MANY receivers simultaneously           │
│  (unlike SQS where ONE consumer reads each message)         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### SNS Settings

| Setting | Detail |
|---------|--------|
| **Subscribers** | SQS, Lambda, HTTP(S), Email, SMS, Kinesis Data Firehose, Platform endpoint |
| **Message Size** | Up to 256 KB |
| **DLQ Support** | Failed deliveries → SQS DLQ |
| **FIFO Topics** | Supported — but ONLY SQS FIFO as subscriber |
| **Encryption** | In-transit (default) + at-rest (KMS optional) |
| **Access Policy** | Resource-based policy (like S3 bucket policy) |
| **Retry** | Only retries HTTP(S) endpoints. Others: no retry! |

### SNS Key Facts

```
┌─────────────────────────────────────────────────────────────┐
│  🧠 SNS INTERVIEW TIPS                                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  "Alert someone about an AWS event" → SNS                   │
│  "Send notification" → SNS                                  │
│  "CloudWatch Alarm needs to notify" → SNS                   │
│  SNS + CloudWatch = best friends 🤝                         │
│  "Push-based" → SNS                                         │
│  "Pull-based" → SQS                                         │
│  Retry only for HTTP(S), NOT for email/SMS/SQS/Lambda       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### SQS vs SNS — Know the Difference

```
┌──────────────────────────────────────────────────────────────────────┐
│                    SQS vs SNS                                        │
├───────────────────────────────┬──────────────────────────────────────┤
│           SQS                 │           SNS                        │
├───────────────────────────────┼──────────────────────────────────────┤
│ Messaging QUEUE               │ Messaging TOPIC (Pub/Sub)            │
│ PULL-based (consumer polls)   │ PUSH-based (delivers to subscribers) │
│ 1 message → 1 consumer        │ 1 message → MANY subscribers         │
│ Decouples processing          │ Fan-out notifications                │
│ Message persists until read   │ No persistence (deliver or fail)     │
│ Async task processing         │ Alerts, notifications, broadcasting  │
└───────────────────────────────┴──────────────────────────────────────┘
```

### Fan-Out Pattern (SNS + SQS) ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────────┐
│              FAN-OUT PATTERN (SNS → Multiple SQS)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐     ┌───────────┐     ┌──────────────────────┐  │
│  │ Producer │────▶│ SNS Topic │────▶│ SQS: Order Processing│  │
│  │(1 publish│     │           │     └──────────────────────┘  │
│  │ action)  │     │           │                                │
│  └──────────┘     │           │────▶┌──────────────────────┐  │
│                    │           │     │ SQS: Analytics        │  │
│                    │           │     └──────────────────────┘  │
│                    │           │                                │
│                    │           │────▶┌──────────────────────┐  │
│                    │           │     │ SQS: Email Service    │  │
│                    └───────────┘     └──────────────────────┘  │
│                                                                 │
│  Result: ONE event → processed by MULTIPLE independent systems │
│  Each SQS queue processes at its own pace, independently       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. API Gateway ✅ MUST HAVE

> At 12 YOE, API Gateway is YOUR front door for serverless + microservices. Know it's the preferred way to expose APIs (not hard-coded keys).

### What is API Gateway?

A **fully managed service** to create, publish, monitor, and secure APIs — the safe "front door" to your application.

```
┌─────────────────────────────────────────────────────────────┐
│                 API GATEWAY AT A GLANCE                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────┐      ┌─────────────┐      ┌─────────────────┐   │
│  │Client│─────▶│ API Gateway │─────▶│ Backend          │   │
│  │      │      │             │      │ • Lambda         │   │
│  │      │◀─────│ (front door)│◀─────│ • EC2            │   │
│  └──────┘      └─────────────┘      │ • DynamoDB       │   │
│                                      │ • Step Functions │   │
│                                      │ • Any HTTP       │   │
│                                      └─────────────────┘   │
│                                                             │
│  Features:                                                  │
│  ✅ WAF integration (DDoS protection, rate limiting)        │
│  ✅ API versioning                                          │
│  ✅ Authentication (Cognito, IAM, Lambda Authorizers)       │
│  ✅ Throttling & rate limiting                              │
│  ✅ Request/response transformation                         │
│  ✅ Caching (reduce backend calls)                          │
│  ✅ SDK generation                                          │
│                                                             │
│  🧠 "Creating or managing an API" → API Gateway             │
│  🧠 "Secure API without baking credentials" → API Gateway   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### API Gateway Types

| Type | Protocol | Use Case |
|------|----------|----------|
| **REST API** | HTTP (RESTful) | Most common — full features, caching, WAF |
| **HTTP API** | HTTP | Simpler, cheaper, faster — good for Lambda proxy |
| **WebSocket API** | WebSocket | Real-time: chat apps, dashboards, gaming |

---

### When EC2 is Better Than Fargate (for Batch/Compute)

> Not everything fits into serverless. Know the limits.

```
┌─────────────────────────────────────────────────────────────┐
│         WHEN EC2 BEATS FARGATE/SERVERLESS                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Choose EC2 over Fargate/Lambda when:                       │
│                                                             │
│  • Custom AMIs required                                     │
│  • Need > 4 vCPUs                                          │
│  • Need > 30 GiB memory                                    │
│  • Need GPU (ML training, rendering)                        │
│  • Arm-based Graviton CPU (AWS Batch)                       │
│  • Using linuxParameters in container config                │
│  • High job volume (EC2 dispatches at higher concurrency)   │
│                                                             │
│  🧠 Default to serverless, fall back to EC2 for these cases │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. AWS Step Functions ✅ MUST HAVE

> Orchestration for serverless workflows. At 12 YOE, know when to use Step Functions vs SQS/EventBridge.

### What is Step Functions?

A **serverless orchestration service** — coordinate multiple AWS services into visual workflows (state machines).

```
┌─────────────────────────────────────────────────────────────┐
│                 STEP FUNCTIONS AT A GLANCE                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Visual workflow (state machine):                           │
│                                                             │
│  ┌───────┐    ┌───────────┐    ┌─────────┐    ┌────────┐ │
│  │ START │───▶│ Validate  │───▶│ Process │───▶│  Save  │ │
│  │       │    │ (Lambda)  │    │(Lambda) │    │ (DDB)  │ │
│  └───────┘    └─────┬─────┘    └─────────┘    └────────┘ │
│                     │                                      │
│                     │ (if invalid)                          │
│                     ▼                                      │
│               ┌──────────┐                                 │
│               │  FAIL    │                                 │
│               │ (notify) │                                 │
│               └──────────┘                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Execution Types

| Type | Use Case | Duration | Rate |
|------|----------|----------|------|
| **Standard** | Long-running, auditable | Up to 1 year | Lower event rate |
| **Express** | High-volume, short-lived | Up to 5 minutes | Millions of events/sec |

### State Types (8 Types)

| State | What It Does |
|-------|-------------|
| **Task** | Execute work (Lambda, ECS, API call) |
| **Choice** | Branching logic (if/else) |
| **Parallel** | Run branches simultaneously |
| **Wait** | Pause for X seconds or until timestamp |
| **Map** | Loop over items (forEach) |
| **Pass** | Pass input to output (transform data) |
| **Succeed** | End successfully |
| **Fail** | End with error |

### When to Use Step Functions

```
┌──────────────────────────────────────────────────────────────┐
│        STEP FUNCTIONS DECISION                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "Orchestrate multiple Lambda/services in sequence"          │
│       → Step Functions ✅                                    │
│                                                              │
│  "Visual workflow with branching/parallel/retry"             │
│       → Step Functions ✅                                    │
│                                                              │
│  "Long-running process with auditing"                        │
│       → Step Functions (Standard) ✅                         │
│                                                              │
│  "Simple queue → process one by one"                         │
│       → SQS (not Step Functions)                            │
│                                                              │
│  "React to events, trigger downstream"                       │
│       → EventBridge (not Step Functions)                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Decoupling Topic | Verdict | Why |
|-----------------|---------|-----|
| SQS (Standard vs FIFO) | ✅ Must know | "Message ordering" = FIFO, guaranteed question |
| SQS Queue Depth + Auto Scaling | ✅ Must know | Production scaling pattern |
| Dead-Letter Queues | ✅ Must know | Error handling in decoupled systems |
| Long vs Short Polling | ✅ Must know | Always pick long polling |
| SNS (pub/sub, subscribers) | ✅ Must know | "Alert/notify" = SNS |
| Fan-Out (SNS → SQS) | ✅ Must know | Classic architecture pattern |
| SQS vs SNS | ✅ Must know | Pull vs Push, 1:1 vs 1:many |
| API Gateway | ✅ Must know | Front door for all APIs |
| Step Functions | ✅ Must know | Orchestration of serverless workflows |
| Step Functions vs SQS vs EventBridge | ✅ Must know | "When to use which" at senior level |


---

---

# 📈 CATEGORY: ANALYTICS & BIG DATA

---

## 1. Amazon Athena ✅ MUST HAVE

> "Serverless SQL on S3" — one of the most-asked scenario questions. Know when to pick Athena.

### What is Athena?

An **interactive query service** that lets you run SQL directly on data stored in S3 — no database, no loading, no servers.

```
┌─────────────────────────────────────────────────────────────┐
│                    ATHENA AT A GLANCE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐   │
│  │   YOU    │────▶│    ATHENA    │────▶│   S3 BUCKET  │   │
│  │(write SQL│     │ (serverless  │     │  (your data) │   │
│  │ query)   │     │  query engine│     │  CSV, JSON,  │   │
│  └──────────┘     └──────────────┘     │  Parquet,ORC │   │
│                                         └──────────────┘   │
│                                                             │
│  • Serverless — no infrastructure to manage                 │
│  • Pay per query (per TB scanned)                           │
│  • Standard SQL (Presto engine under the hood)              │
│  • No data loading — query S3 DIRECTLY                      │
│  • Works with CSV, JSON, Parquet, ORC, Avro                 │
│                                                             │
│  🧠 "Serverless SQL on S3" → Athena (always!)              │
│  🧠 "Query data without loading into database" → Athena     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. AWS Glue ⚡ GOOD TO KNOW

> The ETL (Extract, Transform, Load) service. Know it pairs with Athena for data discovery.

### What is Glue?

A **serverless data integration service** — discover, prepare, transform, and combine data from various sources.

```
┌─────────────────────────────────────────────────────────────┐
│                      AWS GLUE                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── Data Sources ──┐     ┌───────┐     ┌─── Targets ──┐│
│  │ • S3              │     │       │     │ • S3          ││
│  │ • RDS             │────▶│ GLUE  │────▶│ • Redshift    ││
│  │ • DynamoDB        │     │ (ETL) │     │ • Athena      ││
│  │ • On-prem DB      │     │       │     │ • Data Lake   ││
│  └───────────────────┘     └───────┘     └───────────────┘│
│                                                             │
│  Key Components:                                            │
│  • Glue Crawler — discovers data & creates catalog          │
│  • Glue Data Catalog — metadata store (schema registry)     │
│  • Glue ETL Jobs — transform data (Spark-based)             │
│                                                             │
│  🧠 "ETL + serverless" → Glue                              │
│  🧠 Glue Catalog + Athena = query S3 as a data lake        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Amazon OpenSearch (Elasticsearch) ⚡ GOOD TO KNOW

> Know it for LOG ANALYTICS and VISUALIZATION scenarios. Successor to Elasticsearch Service.

```
┌─────────────────────────────────────────────────────────────┐
│                  AMAZON OPENSEARCH                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Managed search & analytics engine                    │
│  (Successor to Amazon Elasticsearch Service)                │
│                                                             │
│  ┌─── Common Architecture ──────────────────────────────┐  │
│  │                                                       │  │
│  │  CloudWatch Logs ─┐                                   │  │
│  │  VPC Flow Logs ───┼──▶ OpenSearch ──▶ Kibana/         │  │
│  │  CloudTrail ──────┘    (index &      Dashboards       │  │
│  │  App Logs ────────┘     search)      (visualize)      │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  Use cases:                                                 │
│  • Log analytics & visualization                            │
│  • Full-text search                                         │
│  • BI reports on log data                                   │
│  • Security analytics (SIEM)                                │
│  • Application monitoring                                   │
│                                                             │
│  🧠 "Log visualization" or "log analytics" → OpenSearch     │
│  🧠 "Search engine within your app" → OpenSearch            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Analytics — Quick Decision Map

```
┌──────────────────────────────────────────────────────────────┐
│            ANALYTICS — WHICH SERVICE?                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "Serverless SQL directly on S3"       → Athena             │
│  "ETL jobs (transform data)"           → Glue              │
│  "Data warehouse (OLAP, petabytes)"    → Redshift          │
│  "Log analytics & visualization"       → OpenSearch         │
│  "Real-time streaming analytics"       → Kinesis           │
│  "Discover data schema in S3"          → Glue Crawler      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Analytics Topic | Verdict | Why |
|----------------|---------|-----|
| Athena (serverless SQL on S3) | ✅ Must know | "Query S3 without loading" = Athena. Always. |
| Glue (ETL + Crawler + Catalog) | ⚡ Know concept | Pairs with Athena, data lake architectures |
| OpenSearch (log visualization) | ⚡ Know when to use | "Log analytics + dashboard" scenarios |
| Athena vs Redshift | ✅ Must know difference | Athena = serverless/ad-hoc. Redshift = warehouse |

---

---

# ⚡ CATEGORY: SERVERLESS

---

## 1. AWS Lambda ✅ MUST HAVE

> Lambda is THE serverless compute service. At 12 YOE, know Lambda deeply — triggers, permissions, VPC networking, limits, and patterns. "Lambda is the answer" to most automation scenarios.

### What is Lambda?

Run code **without provisioning servers** — just upload your function, set a trigger, and AWS handles everything else.

```
┌─────────────────────────────────────────────────────────────┐
│                    LAMBDA AT A GLANCE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ⚡ Serverless compute — no servers to manage                │
│  💰 Pay per invocation + execution time (per ms)            │
│  📈 Auto-scales (0 → thousands of concurrent executions)    │
│  ⏱️  Max execution time: 15 minutes                          │
│  💾 Memory: 128 MB → 10,240 MB (CPU scales with memory)    │
│  📦 Deployment package: 50 MB (zipped) / 250 MB (unzipped) │
│  🌐 Can run inside VPC (for private resources access)       │
│                                                             │
│  🧠 "Add features to AWS" → Lambda                          │
│  🧠 "Automate anything" → Lambda                            │
│  🧠 "Serverless compute" → Lambda                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Building a Lambda Function — 5 Key Components

```
┌─────────────────────────────────────────────────────────────────┐
│              LAMBDA FUNCTION ANATOMY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─── 1. RUNTIME ──────────────────────────────────────────┐   │
│  │  Language: Python, Node.js, Java, Go, .NET, Ruby        │   │
│  │  Or bring your own (custom runtime via Lambda Layers)    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─── 2. PERMISSIONS (IAM Role) ───────────────────────────┐   │
│  │  Lambda needs a ROLE to access other AWS services        │   │
│  │  E.g., "Allow Lambda to write to DynamoDB"              │   │
│  │  Called the "Execution Role"                             │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─── 3. NETWORKING (Optional) ────────────────────────────┐   │
│  │  Run Lambda inside a VPC to access private resources     │   │
│  │  Define: VPC, Subnet(s), Security Group(s)              │   │
│  │  ⚠️ VPC Lambda needs NAT Gateway for internet access    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─── 4. RESOURCES ────────────────────────────────────────┐   │
│  │  Memory: 128 MB → 10 GB                                 │   │
│  │  CPU: Scales proportionally with memory                  │   │
│  │  More memory = more CPU = faster execution               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─── 5. TRIGGER ──────────────────────────────────────────┐   │
│  │  What event invokes this function?                       │   │
│  │  • API Gateway (HTTP request)                            │   │
│  │  • S3 (object uploaded)                                  │   │
│  │  • DynamoDB Streams (table change)                       │   │
│  │  • SQS (message in queue)                                │   │
│  │  • CloudWatch Events / EventBridge (schedule/event)      │   │
│  │  • SNS (notification)                                    │   │
│  │  • Kinesis (stream data)                                 │   │
│  │  • ALB (load balanced)                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Lambda — Common Production Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│            LAMBDA COMMON USE CASES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Pattern 1: API Backend                                         │
│  Client → API Gateway → Lambda → DynamoDB                       │
│                                                                 │
│  Pattern 2: Event Processing                                    │
│  S3 Upload → Lambda → Thumbnail generation → S3                │
│                                                                 │
│  Pattern 3: Automation                                          │
│  CloudWatch Event (cron) → Lambda → Stop EC2 at night          │
│                                                                 │
│  Pattern 4: Stream Processing                                   │
│  Kinesis Stream → Lambda → DynamoDB / S3                       │
│                                                                 │
│  Pattern 5: Queue Consumer                                      │
│  SQS → Lambda → Process message → Store result                 │
│                                                                 │
│  Pattern 6: Security Automation                                 │
│  CloudTrail → EventBridge → Lambda → Remediate                 │
│  (e.g., auto-remove public S3 bucket access)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Lambda Limits — Know These

| Limit | Value |
|-------|-------|
| Max execution time | **15 minutes** |
| Memory | 128 MB → 10,240 MB |
| Deployment package (zipped) | 50 MB |
| Deployment package (unzipped) | 250 MB |
| Environment variables | 4 KB |
| /tmp storage | 512 MB (up to 10 GB) |
| Concurrent executions | 1,000 (default, can request increase) |

> 🧠 **Interview Tip:** If task takes > 15 minutes → Lambda is NOT the answer. Use ECS/Fargate, Step Functions, or EC2.

---

### Serverless Mindset for Interviews

```
┌─────────────────────────────────────────────────────────────┐
│          THE SERVERLESS INTERVIEW RULE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  When you see answer choices:                               │
│                                                             │
│  ❌ "Deploy on EC2, manage servers, install software..."    │
│  ✅ "Use Lambda / Fargate / managed service..."             │
│                                                             │
│  FAVOR SERVERLESS unless there's a specific reason not to   │
│  (GPU, >15min execution, >10GB memory, custom OS, etc.)     │
│                                                             │
│  "Lambda is the answer" for:                                │
│  • Automating AWS tasks                                     │
│  • Processing events/triggers                               │
│  • API backends                                             │
│  • Glue logic between services                              │
│  • Security remediation                                     │
│  • Scheduled tasks (cron)                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Serverless Topic | Verdict | Why |
|-----------------|---------|-----|
| Lambda (triggers, role, VPC, limits) | ✅ Must know deeply | Core serverless service — used everywhere |
| Lambda patterns (API, events, automation) | ✅ Must know | Design questions at senior level |
| Lambda limits (15 min, memory, package size) | ✅ Must know | "When NOT to use Lambda" scenarios |
| Lambda + VPC networking | ✅ Must know | "Access private RDS from Lambda" = VPC + NAT |
| Athena (serverless SQL on S3) | ✅ Must know | Big data scenario staple |
| Glue (ETL) | ⚡ Know concept | Data engineering scenarios |
| OpenSearch (log analytics) | ⚡ Know when to use | "Visualize logs" → OpenSearch |


---

---

# 🐳 CATEGORY: CONTAINERS

---

## 1. ECS (Elastic Container Service) ✅ MUST HAVE

> At 12 YOE, containers are production bread and butter. Know ECS vs EKS decision, Fargate vs EC2 launch types, and how they all connect.

### What is ECS?

AWS's **proprietary container orchestration service** — manages containers at scale with deep AWS integration.

```
┌─────────────────────────────────────────────────────────────┐
│                      ECS AT A GLANCE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🐳 Manages 1 to thousands of containers                    │
│  ⚖️  ELB integration (auto-register/deregister containers)  │
│  🔐 Individual IAM roles per container (task role)          │
│  📈 Easy to set up and scale                                │
│  🏗️  Deep AWS integration (CloudWatch, VPC, ALB, etc.)      │
│                                                             │
│  Two Launch Types:                                          │
│  ┌──────────────────┐    ┌──────────────────┐             │
│  │   EC2 Launch     │    │  FARGATE Launch  │             │
│  │  (you manage     │    │  (serverless —   │             │
│  │   the hosts)     │    │   AWS manages)   │             │
│  └──────────────────┘    └──────────────────┘             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### ECS Architecture — Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                        ECS CLUSTER                                    │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐                                                    │
│  │   SERVICE    │ (desired count: 4, load-balanced)                  │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ├── Task Definition (blueprint: image, CPU, memory, role)    │
│         │                                                            │
│  ┌──────▼────────────────────────────────────────────────────────┐  │
│  │                         ALB                                    │  │
│  └──────┬─────────────────────┬──────────────────────────────────┘  │
│         │                     │                                      │
│  ┌──────▼──────┐       ┌─────▼───────┐                             │
│  │  AZ-1a      │       │   AZ-1b     │                             │
│  │ ┌────┐┌────┐│       │ ┌────┐┌────┐│                             │
│  │ │Task││Task││       │ │Task││Task││                             │
│  │ └────┘└────┘│       │ └────┘└────┘│                             │
│  └─────────────┘       └─────────────┘                             │
│                                                                      │
│  Running on: EC2 instances (you manage)                              │
│         OR: Fargate (AWS manages — serverless)                       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2. EKS (Elastic Kubernetes Service) ✅ MUST HAVE

### What is EKS?

AWS's **managed Kubernetes service** — runs the open-source Kubernetes control plane so you don't have to install, operate, or maintain it yourself.

```
┌─────────────────────────────────────────────────────────────┐
│                      EKS AT A GLANCE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ☸️  Managed Kubernetes (open-source orchestration)          │
│  🌍 Portable — same K8s API works on-prem, GCP, Azure      │
│  🔧 More complex to configure than ECS                      │
│  🏗️  Large ecosystem (Helm, Istio, ArgoCD, etc.)            │
│  📦 Runs on EC2 or Fargate (same as ECS)                    │
│                                                             │
│  EKS manages:                                               │
│  • Kubernetes control plane (API server, etcd, scheduler)   │
│  • High availability (multi-AZ control plane)               │
│  • Patching & upgrades of K8s version                       │
│                                                             │
│  YOU manage:                                                │
│  • Worker nodes (EC2 or Fargate)                            │
│  • Pods, Deployments, Services                              │
│  • Networking (VPC CNI plugin)                              │
│  • Monitoring (Prometheus, CloudWatch Container Insights)   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### ECS vs EKS — Decision ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────────┐
│                      ECS vs EKS                                      │
├────────────────────────────────┬─────────────────────────────────────┤
│          ECS                   │           EKS                       │
├────────────────────────────────┼─────────────────────────────────────┤
│ AWS proprietary                │ Open-source Kubernetes              │
│ Simpler to set up             │ More complex configuration          │
│ Deep AWS integration          │ Portable (multi-cloud, on-prem)     │
│ Best if all-in on AWS         │ Best if multi-cloud or open-source  │
│ Task Definitions              │ Pod specs (YAML manifests)          │
│ AWS-specific tooling          │ Huge K8s ecosystem (Helm, Istio)    │
│ Less operational overhead     │ More operational overhead           │
│ Great for simpler apps        │ Great for complex microservices     │
├────────────────────────────────┴─────────────────────────────────────┤
│                                                                      │
│  🧠 Decision Rule:                                                   │
│  "Containers on AWS, simple"         → ECS                          │
│  "Open-source / Kubernetes / on-prem / multi-cloud" → EKS           │
│  "Portability required"              → EKS                          │
│  Default exam answer for containers  → ECS (unless K8s mentioned)   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3. AWS Fargate ✅ MUST HAVE

> Serverless containers — no EC2 instances to manage. Know Fargate vs EC2 launch type and Fargate vs Lambda.

### What is Fargate?

A **serverless compute engine for containers** — works with BOTH ECS and EKS. AWS manages all infrastructure.

```
┌─────────────────────────────────────────────────────────────┐
│                    FARGATE AT A GLANCE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ⚡ Serverless — no EC2 instances to manage                  │
│  🐳 Works with ECS AND EKS                                  │
│  🐧 Linux-only workloads                                    │
│  💰 Pay for resources allocated + time running              │
│  🔒 Each task runs in its own isolation boundary            │
│  📦 No OS access (no SSH into the container host)           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Fargate vs EC2 Launch Type

```
┌──────────────────────────────────────────────────────────────────────┐
│                  FARGATE vs EC2 LAUNCH TYPE                           │
├───────────────────────────────┬──────────────────────────────────────┤
│        FARGATE                │         EC2                          │
├───────────────────────────────┼──────────────────────────────────────┤
│ No OS access                  │ Full OS access (SSH, install agents) │
│ Pay per task (resources+time) │ Pay per EC2 instance (always on)     │
│ AWS manages infrastructure    │ YOU manage instances (patch, scale)  │
│ Isolated environments         │ Multiple containers share host       │
│ Short-running tasks ideal     │ Long-running containers ideal        │
│ Auto-scales tasks             │ You manage ASG for hosts             │
│ Linux only                    │ Linux + Windows                      │
│ Simpler operations            │ More control, more responsibility    │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  Choose FARGATE when: "I don't want to manage servers"              │
│  Choose EC2 when: "I need OS access, GPUs, Windows, or cost control"│
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Fargate vs Lambda ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────────┐
│                    FARGATE vs LAMBDA                                  │
├───────────────────────────────┬──────────────────────────────────────┤
│        FARGATE                │         LAMBDA                       │
├───────────────────────────────┼──────────────────────────────────────┤
│ Container-based (Docker)      │ Function-based (single function)     │
│ Consistent workloads          │ Unpredictable / inconsistent loads   │
│ Longer running (hours/days)   │ Short (max 15 minutes)              │
│ More developer control        │ Simpler, less control               │
│ Docker ecosystem tools        │ AWS-managed runtimes                │
│ Up to 4 vCPU / 30 GB RAM     │ Up to 10 GB RAM                    │
│ Full application packages     │ Single functions                    │
│ Pay: resources + time         │ Pay: invocations + duration (ms)   │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 Decision:                                                        │
│  "Single function, event-driven, short"  → Lambda                   │
│  "Dockerized app, consistent load"       → Fargate                  │
│  "Need full Docker control + longer run" → Fargate                  │
│  "Quick glue code, automation"           → Lambda                   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### The Complete Compute Decision Map ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────┐
│         COMPUTE — EC2 vs FARGATE vs LAMBDA                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Need GPU, custom AMI, Windows containers, >30GB RAM"           │
│       → EC2                                                     │
│                                                                  │
│  "Dockerized app, don't want to manage servers, Linux"           │
│       → Fargate                                                 │
│                                                                  │
│  "Single function, event-driven, < 15 min, < 10GB RAM"          │
│       → Lambda                                                  │
│                                                                  │
│  "Kubernetes / open-source / multi-cloud"                        │
│       → EKS (+ Fargate or EC2 nodes)                           │
│                                                                  │
│  "Simple container orchestration, all-in on AWS"                 │
│       → ECS (+ Fargate or EC2)                                 │
│                                                                  │
│  "Long-running, multiple containers sharing host, cost control"  │
│       → EC2 launch type                                         │
│                                                                  │
│  "Short tasks, isolated, no OS management"                       │
│       → Fargate                                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. Amazon ECR (Elastic Container Registry) ⚡ GOOD TO KNOW

> Where you store your Docker images. Think: "Docker Hub but private and on AWS."

```
┌─────────────────────────────────────────────────────────────┐
│                    AMAZON ECR                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Private Docker image registry on AWS                 │
│                                                             │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐   │
│  │Developer │────▶│     ECR      │────▶│  ECS / EKS   │   │
│  │(push img)│     │ (store img)  │     │  (pull & run)│   │
│  └──────────┘     └──────────────┘     └──────────────┘   │
│                                                             │
│  Components:                                                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Registry     → One per AWS account (private)         │  │
│  │  Repository   → Holds images (like a Git repo)        │  │
│  │  Image        → Docker/OCI image (tagged versions)    │  │
│  │  Auth Token   → Required to push/pull (expires 12hr)  │  │
│  │  Repo Policy  → IAM-based access control              │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Features:                                                  │
│  ✅ Image scanning (vulnerabilities)                        │
│  ✅ Encryption at rest                                      │
│  ✅ Lifecycle policies (auto-delete old images)             │
│  ✅ Cross-region replication                                │
│  ✅ ECR Public (for public open-source images)              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. Amazon EventBridge ✅ MUST HAVE

> The "glue" of serverless architectures. At 12 YOE, know EventBridge as the event router that connects everything.

### What is EventBridge?

A **serverless event bus** — routes events from sources to targets. Formerly CloudWatch Events.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EVENTBRIDGE AT A GLANCE                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─── Sources ────────┐     ┌───────────────┐     ┌── Targets ──┐ │
│  │ • AWS Services     │     │               │     │ • Lambda    │ │
│  │   (EC2, S3, IAM)   │     │  EVENTBRIDGE  │     │ • SQS       │ │
│  │ • Custom Apps      │────▶│   (Event Bus) │────▶│ • SNS       │ │
│  │ • SaaS Partners    │     │               │     │ • Step Func │ │
│  │   (Zendesk, etc.)  │     │    RULES      │     │ • Kinesis   │ │
│  │ • Scheduled (cron) │     │   (filter &   │     │ • ECS Task  │ │
│  └────────────────────┘     │    route)     │     │ • API dest. │ │
│                              └───────────────┘     └─────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating an EventBridge Rule

```
┌──────────────────────────────────────────────────────────────┐
│          EVENTBRIDGE RULE — 5 STEPS                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: DEFINE PATTERN                                      │
│          • Event-driven: "When EC2 instance state changes"   │
│          • Scheduled: "Every day at 6 PM UTC"               │
│                                                              │
│  Step 2: SELECT EVENT BUS                                    │
│          • Default (AWS events)                              │
│          • Custom (your app events)                          │
│          • Partner (SaaS integrations)                        │
│                                                              │
│  Step 3: SELECT TARGET                                       │
│          • Lambda function                                   │
│          • SQS queue                                         │
│          • Step Function                                     │
│          • SNS topic                                         │
│                                                              │
│  Step 4: TAG IT                                              │
│                                                              │
│  Step 5: DONE — wait for event or test it!                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### EventBridge Key Facts

```
┌─────────────────────────────────────────────────────────────┐
│  🧠 EVENTBRIDGE INTERVIEW TIPS                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  "Trigger action based on AWS event"     → EventBridge      │
│  "Serverless cron/scheduled task"        → EventBridge      │
│  "Any API call in AWS can trigger..."    → EventBridge      │
│  "Glue for serverless architecture"      → EventBridge      │
│  "React to EC2 state change"             → EventBridge      │
│  "Route events from SaaS partners"       → EventBridge      │
│                                                             │
│  EventBridge = CloudWatch Events (evolved, more features)   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. AWS X-Ray ✅ MUST HAVE

> Distributed tracing for microservices/serverless. At 12 YOE with containers, this is essential for debugging production issues.

### What is X-Ray?

A **distributed tracing service** — helps you analyze and debug production microservices by tracking requests as they flow through your entire application.

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS X-RAY AT A GLANCE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Problem: In microservices, one request touches MANY        │
│  services. When something is slow or broken, WHERE is it?   │
│                                                             │
│  Solution: X-Ray traces the ENTIRE request journey          │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  User Request                                         │  │
│  │    │                                                  │  │
│  │    ▼                                                  │  │
│  │  API Gateway (12ms) ──▶ Lambda (45ms) ──▶ DynamoDB   │  │
│  │                              │              (8ms)     │  │
│  │                              ▼                        │  │
│  │                         SQS (2ms) ──▶ Lambda (120ms) │  │
│  │                                          │           │  │
│  │                                          ▼           │  │
│  │                                      S3 (15ms) ❌ SLOW│  │
│  │                                                       │  │
│  │  X-Ray shows: Total: 202ms. Bottleneck: 2nd Lambda   │  │
│  │                                                       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### How X-Ray Works

```
┌─────────────────────────────────────────────────────────────┐
│                 X-RAY COMPONENTS                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── X-Ray SDK ──────────────────────────────────────┐    │
│  │  Install in your code (Lambda, EC2, ECS)           │    │
│  │  Automatically traces: HTTP calls, AWS SDK calls,  │    │
│  │  SQL queries, custom subsegments                    │    │
│  └────────────────────────────────────────────────────┘    │
│                    │                                        │
│                    ▼                                        │
│  ┌─── X-Ray Daemon ──────────────────────────────────┐    │
│  │  Runs on EC2/ECS (collects & sends traces to API)  │    │
│  │  (Lambda has it built-in — no daemon needed!)      │    │
│  └────────────────────────────────────────────────────┘    │
│                    │                                        │
│                    ▼                                        │
│  ┌─── X-Ray Console ─────────────────────────────────┐    │
│  │  • Service Map (visual graph of architecture)      │    │
│  │  • Traces (individual request journeys)            │    │
│  │  • Analytics (filter, group, find patterns)        │    │
│  └────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### X-Ray Key Concepts

| Concept | What It Means |
|---------|--------------|
| **Trace** | End-to-end journey of a single request through all services |
| **Segment** | Data about work done by ONE service (e.g., Lambda segment) |
| **Subsegment** | Detailed breakdown within a segment (e.g., DynamoDB call within Lambda) |
| **Service Map** | Visual graph showing all services and their connections + latency |
| **Annotations** | Key-value pairs for FILTERING traces (indexed, searchable) |
| **Metadata** | Additional data NOT indexed (for context only) |

### X-Ray Production Example — Debugging Latency

```
┌──────────────────────────────────────────────────────────────┐
│          X-RAY SERVICE MAP (Visual in Console)               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐      ┌──────────┐      ┌──────────────┐      │
│  │  Client  │─────▶│API G/W   │─────▶│  Lambda A    │      │
│  │          │      │  (5ms)   │      │  (200ms) 🔴  │      │
│  └──────────┘      └──────────┘      └──────┬───────┘      │
│                                              │              │
│                                    ┌─────────┴────────┐     │
│                                    ▼                  ▼     │
│                              ┌──────────┐      ┌─────────┐ │
│                              │DynamoDB  │      │  SQS    │ │
│                              │ (3ms) 🟢 │      │ (2ms) 🟢│ │
│                              └──────────┘      └────┬────┘ │
│                                                     ▼      │
│                                               ┌──────────┐ │
│                                               │Lambda B  │ │
│                                               │(500ms) 🔴│ │
│                                               └──────────┘ │
│                                                              │
│  X-Ray reveals: Lambda B is the bottleneck (500ms)          │
│  Drill into subsegments: It's a slow external API call!     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Where X-Ray Integrates

| Service | X-Ray Support |
|---------|--------------|
| Lambda | ✅ Built-in (just enable "Active Tracing") |
| API Gateway | ✅ Enable tracing in stage settings |
| ECS / Fargate | ✅ Run X-Ray daemon as sidecar container |
| EC2 | ✅ Install X-Ray daemon + SDK |
| ELB (ALB) | ✅ Passes trace headers |
| SQS / SNS | ✅ Propagates trace context |
| Elastic Beanstalk | ✅ Built-in option |

### X-Ray vs CloudWatch

```
┌──────────────────────────────────────────────────────────────┐
│         X-RAY vs CLOUDWATCH — NOT competitors!               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CloudWatch = METRICS + LOGS + ALARMS                        │
│  "How much CPU? How many errors? Alert me when..."          │
│                                                              │
│  X-Ray = TRACES (request journey)                            │
│  "Why is this request slow? Where did it fail?"             │
│                                                              │
│  USE TOGETHER:                                               │
│  • CloudWatch tells you SOMETHING is wrong                  │
│  • X-Ray tells you WHERE and WHY                            │
│                                                              │
│  🧠 "Distributed tracing" or "find bottleneck" → X-Ray     │
│  🧠 "Monitor metrics/logs" → CloudWatch                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Container Topic | Verdict | Why |
|----------------|---------|-----|
| ECS (architecture, task definitions, services) | ✅ Must know | Default container answer on AWS |
| EKS (when to use, vs ECS) | ✅ Must know | "Kubernetes / open-source / multi-cloud" = EKS |
| Fargate vs EC2 launch type | ✅ Must know | "Serverless containers" decision |
| Fargate vs Lambda | ✅ Must know cold | THE most common compute decision question |
| ECR (store Docker images) | ⚡ Know concept | "Where do images go?" = ECR |
| EventBridge (event routing) | ✅ Must know | Glue of serverless + event-driven |
| X-Ray (distributed tracing) | ✅ Must know | Debug microservices in production |
| X-Ray vs CloudWatch | ✅ Must know | "Tracing" = X-Ray. "Metrics/Logs" = CloudWatch |
| EC2 vs Fargate vs Lambda decision | ✅ Must know cold | Core architecture decision at senior level |


---

## 2. CloudTrail (API Auditing) ✅ MUST HAVE

> Think of CloudTrail as **CCTV for your AWS account** — records WHO did WHAT, WHEN, and from WHERE.

### What is CloudTrail?

Records **every API call** made in your AWS account — Console actions, CLI commands, SDK calls, service-to-service.

```
┌─────────────────────────────────────────────────────────────┐
│                   CLOUDTRAIL AT A GLANCE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🎥 "CCTV for your AWS account"                             │
│                                                             │
│  Records:                                                   │
│  • WHO made the call (IAM user, role, service)              │
│  • WHAT was called (API action: RunInstances, PutObject)    │
│  • WHEN it happened (timestamp)                             │
│  • WHERE from (source IP address)                           │
│  • WHAT changed (request/response parameters)               │
│                                                             │
│  Stores logs in: S3 bucket (and optionally CloudWatch Logs) │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  User/Service ──API Call──▶ AWS                      │   │
│  │        │                                             │   │
│  │        └──▶ CloudTrail ──▶ S3 Bucket (logs)         │   │
│  │                       └──▶ CloudWatch Logs           │   │
│  │                       └──▶ EventBridge (trigger)     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  🧠 "Who did this?" or "audit" or "API logging" → CloudTrail│
│  🧠 CloudTrail ≠ CloudWatch                                 │
│     CloudTrail = WHO did WHAT (audit)                       │
│     CloudWatch = HOW is it performing (metrics/logs)        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### CloudTrail vs CloudWatch vs X-Ray

| Service | Purpose | Think... |
|---------|---------|----------|
| **CloudTrail** | WHO did WHAT (API audit) | CCTV camera |
| **CloudWatch** | HOW is it performing (metrics, logs, alarms) | Health monitor |
| **X-Ray** | WHERE is the bottleneck (distributed tracing) | Request detective |

---

## 3. AWS Shield (DDoS Protection) ✅ MUST HAVE

> Know the two tiers and what each protects. "DDoS" in a question → Shield.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        AWS SHIELD                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─── SHIELD STANDARD (FREE) ──────────────────────────────────────┐   │
│  │                                                                  │   │
│  │  • Automatically enabled for ALL AWS customers                   │   │
│  │  • Protects: ELB, CloudFront, Route 53                          │   │
│  │  • Defends against: Layer 3 & 4 attacks                         │   │
│  │    (SYN/UDP floods, reflection attacks)                          │   │
│  │  • No cost, no setup needed                                     │   │
│  │                                                                  │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─── SHIELD ADVANCED ($3,000/month) ─────────────────────────────┐    │
│  │                                                                  │    │
│  │  • Enhanced protection (larger, sophisticated attacks)           │    │
│  │  • Protects: ELB, CloudFront, Route 53, Elastic IPs            │    │
│  │  • 24/7 DDoS Response Team (DRT) access                        │    │
│  │  • Near real-time attack notifications                          │    │
│  │  • Always-on flow-based monitoring                              │    │
│  │  • Cost protection: AWS won't bill you for DDoS-caused          │    │
│  │    usage spikes on protected resources!                          │    │
│  │  • WAF included at no extra cost                                │    │
│  │                                                                  │    │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  🧠 "DDoS protection" → Shield                                         │
│  🧠 "Layer 3/4 attacks" → Shield                                       │
│  🧠 "Bill protection during DDoS" → Shield Advanced                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. AWS WAF (Web Application Firewall) ✅ MUST HAVE

> Layer 7 protection. "Block SQL injection / XSS / specific IPs / countries" → WAF.

```
┌─────────────────────────────────────────────────────────────┐
│                      AWS WAF                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Layer 7 (HTTP/HTTPS) web application firewall        │
│                                                             │
│  Sits in front of:                                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐              │
│  │CloudFront│   │   ALB    │   │API G/W   │              │
│  └──────────┘   └──────────┘   └──────────┘              │
│                                                             │
│  3 Behaviors:                                               │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  1. ALLOW all — except what you block                │  │
│  │  2. BLOCK all — except what you allow                │  │
│  │  3. COUNT — just count matches (monitoring mode)      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Can filter on:                                             │
│  • IP addresses (block specific IPs/ranges)                 │
│  • Country of origin (geo-blocking)                         │
│  • Request headers / values                                 │
│  • SQL injection patterns                                   │
│  • Cross-site scripting (XSS)                               │
│  • String/regex patterns in requests                        │
│  • Rate-based rules (rate limiting)                         │
│                                                             │
│  Result: Allow (200) or Block (403 Forbidden)               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Shield vs WAF — Know the Difference

```
┌──────────────────────────────────────────────────────────────────────┐
│                  SHIELD vs WAF                                        │
├───────────────────────────────┬──────────────────────────────────────┤
│          SHIELD               │           WAF                        │
├───────────────────────────────┼──────────────────────────────────────┤
│ Layer 3 & 4                   │ Layer 7                              │
│ DDoS protection               │ Application-level filtering          │
│ Network/transport attacks     │ HTTP/HTTPS attacks                   │
│ SYN floods, UDP floods        │ SQL injection, XSS, IP blocking     │
│ Automatic (Standard)          │ Rules you configure                  │
│ "Volumetric attacks"          │ "Application attacks"                │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 "DDoS / Layer 3-4" → Shield                                     │
│  🧠 "SQL injection / XSS / Layer 7 / block IPs" → WAF              │
│  🧠 Use BOTH together for complete protection!                       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 5. AWS Firewall Manager ⚡ GOOD TO KNOW

> Centrally manage WAF rules and Shield Advanced across MULTIPLE accounts.

```
┌─────────────────────────────────────────────────────────────┐
│                AWS FIREWALL MANAGER                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Central security management across multiple          │
│        AWS accounts (via AWS Organizations)                  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           AWS ORGANIZATIONS                          │   │
│  │                                                      │   │
│  │    ┌──────────┐  ┌──────────┐  ┌──────────┐        │   │
│  │    │Account A │  │Account B │  │Account C │        │   │
│  │    │  ALB     │  │  ALB     │  │CloudFront│        │   │
│  │    └──────────┘  └──────────┘  └──────────┘        │   │
│  │         │              │              │              │   │
│  │         └──────────────┼──────────────┘              │   │
│  │                        ▼                             │   │
│  │              ┌──────────────────┐                    │   │
│  │              │ FIREWALL MANAGER │                    │   │
│  │              │ (one pane of     │                    │   │
│  │              │  glass for WAF,  │                    │   │
│  │              │  Shield, SGs)    │                    │   │
│  │              └──────────────────┘                    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Manages: WAF rules, Shield Advanced, Security Groups,     │
│           VPC Network Firewalls                             │
│                                                             │
│  🧠 "Multiple accounts + centralized security rules"       │
│     → Firewall Manager                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Amazon GuardDuty (Threat Detection) ✅ MUST HAVE

> AI-powered threat detection. Learns normal behavior, alerts on anomalies. Know what it monitors and how it responds.

```
┌─────────────────────────────────────────────────────────────┐
│                  AMAZON GUARDDUTY                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Intelligent THREAT DETECTION using AI/ML             │
│  Think: "Smart security guard watching your account 24/7"   │
│                                                             │
│  ┌─── What It Monitors ────────────────────────────────┐   │
│  │                                                      │   │
│  │  • CloudTrail Logs (API activity)                    │   │
│  │  • VPC Flow Logs (network traffic)                   │   │
│  │  • DNS Logs (DNS queries)                            │   │
│  │                                                      │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  ┌─── What It Detects ─────────────────────────────────┐   │
│  │                                                      │   │
│  │  • Unusual API calls                                 │   │
│  │  • Calls from known malicious IPs                    │   │
│  │  • Attempts to disable CloudTrail                    │   │
│  │  • Unauthorized deployments                          │   │
│  │  • Compromised instances (bitcoin mining, etc.)      │   │
│  │  • Reconnaissance (port scanning)                    │   │
│  │  • Failed login attempts                             │   │
│  │                                                      │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  ┌─── How It Responds ─────────────────────────────────┐   │
│  │                                                      │   │
│  │  Findings → GuardDuty Console                        │   │
│  │          → CloudWatch Events / EventBridge           │   │
│  │          → Lambda (automated remediation)            │   │
│  │                                                      │   │
│  │  Example automation:                                 │   │
│  │  GuardDuty detects compromised EC2                   │   │
│  │  → EventBridge → Lambda → isolate instance (SG)     │   │
│  │                                                      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### GuardDuty Key Facts

| Fact | Detail |
|------|--------|
| **AI/ML based** | Learns normal behavior over 7-14 days (baseline) |
| **Threat feeds** | Uses 3rd-party feeds (Proofpoint, CrowdStrike) + AWS data |
| **Multi-account** | Centralize threat detection across all accounts |
| **No agents** | Doesn't need anything installed — just enable it |
| **Automated response** | CloudWatch Events → Lambda → remediate |
| **Findings** | Appear in GuardDuty console + CloudWatch Events |

---

### Security Services — Complete Decision Map

```
┌──────────────────────────────────────────────────────────────────┐
│          SECURITY — WHICH SERVICE FOR WHICH SCENARIO?            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Who did what? API audit trail"                                 │
│       → CloudTrail                                              │
│                                                                  │
│  "DDoS protection (Layer 3/4)"                                   │
│       → Shield (Standard = free, Advanced = $3K/mo)             │
│                                                                  │
│  "Block SQL injection / XSS / specific IPs (Layer 7)"           │
│       → WAF                                                     │
│                                                                  │
│  "Centralize WAF/Shield rules across multiple accounts"          │
│       → Firewall Manager                                        │
│                                                                  │
│  "Detect threats, compromised instances, anomalies"              │
│       → GuardDuty                                               │
│                                                                  │
│  "Control who can access what (identity)"                        │
│       → IAM                                                     │
│                                                                  │
│  "Encrypt data, manage encryption keys"                          │
│       → KMS                                                     │
│                                                                  │
│  "Store secrets (DB passwords, API keys)"                        │
│       → Secrets Manager                                         │
│                                                                  │
│  "Block a specific IP address"                                   │
│       → NACL (network level) or WAF (application level)         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Security Topic | Verdict | Why |
|---------------|---------|-----|
| CloudTrail (API auditing) | ✅ Must know | "Who did this?" = CloudTrail. Every interview. |
| CloudTrail vs CloudWatch vs X-Ray | ✅ Must know | Classic "what's the difference?" question |
| Shield (Standard vs Advanced) | ✅ Must know | "DDoS" = Shield. Know both tiers. |
| WAF (Layer 7 filtering) | ✅ Must know | "SQL injection / XSS / block IPs" = WAF |
| Shield vs WAF | ✅ Must know difference | Layer 3/4 vs Layer 7 |
| GuardDuty (threat detection) | ✅ Must know | AI threat detection + automated response pattern |
| Firewall Manager | ⚡ Know concept | "Multi-account security management" |
| GuardDuty automated remediation | ✅ Must know pattern | GuardDuty → EventBridge → Lambda → fix |


---

## 7. Amazon Macie (Sensitive Data Discovery) ⚡ GOOD TO KNOW

> AI-powered service that finds sensitive data (PII, PHI, financial) in S3. Know it for compliance scenarios.

```
┌─────────────────────────────────────────────────────────────┐
│                    AMAZON MACIE                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: AI/ML service that discovers & protects sensitive     │
│        data stored in S3                                    │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │   S3 Buckets ──▶ Macie (AI Scans) ──▶ Findings       │  │
│  │                                           │           │  │
│  │                                           ▼           │  │
│  │                                     EventBridge       │  │
│  │                                           │           │  │
│  │                                           ▼           │  │
│  │                                  Step Functions /      │  │
│  │                                  Lambda (remediate)    │  │
│  │                                                       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Discovers:                                                 │
│  • PII (names, SSNs, credit cards, addresses)               │
│  • PHI (medical records, health data)                       │
│  • Financial data (bank accounts, tax IDs)                  │
│                                                             │
│  Also alerts on:                                            │
│  • Unencrypted S3 buckets                                   │
│  • Public S3 buckets                                        │
│  • Buckets shared outside your Organization                 │
│                                                             │
│  Great for: HIPAA, GDPR, PCI-DSS compliance                │
│                                                             │
│  🧠 "Find sensitive data in S3" → Macie                     │
│  🧠 "PII/PHI discovery" → Macie                             │
│  🧠 "HIPAA/GDPR compliance for S3" → Macie                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Amazon Inspector (Vulnerability Scanning) ✅ MUST HAVE

> Automated vulnerability assessment for EC2 and containers. Know what it scans and how.

```
┌─────────────────────────────────────────────────────────────┐
│                  AMAZON INSPECTOR                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Automated security assessment service                │
│        Scans for vulnerabilities and deviations from        │
│        best practices                                       │
│                                                             │
│  Two Assessment Types:                                      │
│                                                             │
│  ┌─── HOST ASSESSMENT ─────────────────────────────────┐   │
│  │  • Runs on EC2 instances (requires agent)            │   │
│  │  • Checks: CVEs, CIS benchmarks, security best      │   │
│  │    practices, network reachability                    │   │
│  │  • OS vulnerability scanning                         │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─── NETWORK ASSESSMENT ──────────────────────────────┐   │
│  │  • Scans VPC network configuration                   │   │
│  │  • Checks: Open ports reachable from internet        │   │
│  │  • No agent required for network checks              │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  Also scans:                                                │
│  • ECR container images (vulnerability scan)                │
│  • Lambda functions (code vulnerabilities)                  │
│                                                             │
│  Schedule: Run once OR weekly (automated)                   │
│                                                             │
│  Output: Findings prioritized by severity                   │
│          (Critical → High → Medium → Low)                   │
│                                                             │
│  🧠 "Vulnerability scan on EC2/VPC" → Inspector             │
│  🧠 "Security assessment" → Inspector                       │
│  🧠 "CVE scanning" → Inspector                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 9. AWS KMS (Key Management Service) ✅ MUST HAVE

> Encryption key management. At 12 YOE, know KMS vs CloudHSM, key policies, rotation, and CMK types.

### What is KMS?

A **managed service** to create, manage, and control encryption keys used across AWS services.

```
┌─────────────────────────────────────────────────────────────┐
│                      AWS KMS AT A GLANCE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🔐 Create & control encryption keys                        │
│  🔗 Integrated with: EBS, S3, RDS, Redshift, Lambda, etc.  │
│  📋 Centralized control over key lifecycle & permissions    │
│  🔄 Automatic key rotation (yearly, if generated in KMS)   │
│  📊 CloudTrail logs every key usage (audit trail)          │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Your Data ──▶ Encrypted with CMK ──▶ Stored Safely  │  │
│  │                                                       │  │
│  │  CMK = Customer Master Key (now called "KMS Key")     │  │
│  │  Contains: Key ID, creation date, description,        │  │
│  │            key state, key material                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3 Ways to Generate a CMK (KMS Key)

```
┌─────────────────────────────────────────────────────────────┐
│           3 WAYS TO CREATE A KMS KEY                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── Option 1: AWS-Generated ─────────────────────────┐   │
│  │  AWS creates key material in KMS-managed HSMs        │   │
│  │  Simplest. Most common. Supports auto-rotation.      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─── Option 2: Import Your Own Key Material ──────────┐   │
│  │  Bring key from your own infrastructure              │   │
│  │  YOU manage key material lifecycle                   │   │
│  │  ⚠️ NO automatic rotation (must rotate manually)    │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─── Option 3: CloudHSM Custom Key Store ─────────────┐   │
│  │  Key generated & stored in YOUR CloudHSM cluster     │   │
│  │  Full control over HSM hardware                      │   │
│  │  ⚠️ NO automatic rotation                           │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3 Ways to Control KMS Key Permissions

```
┌─────────────────────────────────────────────────────────────┐
│        3 WAYS TO CONTROL KMS ACCESS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. KEY POLICY (resource-based policy on the key itself)    │
│     → Full scope of access in one document                  │
│     → Required: every KMS key MUST have a key policy        │
│                                                             │
│  2. IAM POLICY + KEY POLICY (combination)                   │
│     → Manage all IAM permissions in IAM                     │
│     → Key policy must allow IAM policies to work            │
│                                                             │
│  3. GRANTS + KEY POLICY                                     │
│     → Temporary, delegated access                           │
│     → User can grant their own access to others             │
│     → Programmatic (no policy change needed)                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Key Rotation

| Key Type | Auto-Rotation | Detail |
|----------|--------------|--------|
| AWS-managed keys | ✅ Every year (automatic) | Cannot disable |
| Customer-managed (AWS-generated) | ✅ Optional (every year) | Enable it! |
| Imported key material | ❌ Not supported | Must rotate manually |
| CloudHSM custom key store | ❌ Not supported | Must rotate manually |
| Asymmetric keys | ❌ Not supported | Must rotate manually |

---

## 10. AWS CloudHSM ⚡ GOOD TO KNOW

> Dedicated hardware for encryption keys. Know the difference from KMS.

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS CLOUDHSM                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Dedicated Hardware Security Module (HSM)             │
│        YOUR own physical encryption device in AWS           │
│                                                             │
│  🔐 FIPS 140-2 Level 3 compliance                          │
│  🏢 Single-tenant (dedicated to YOU — not shared)           │
│  🔑 YOU manage users, keys, groups                          │
│  ❌ NO automatic key rotation (you manage everything)       │
│  🏗️  Deploy in HA (cluster across AZs)                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### KMS vs CloudHSM ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────────┐
│                    KMS vs CloudHSM                                    │
├───────────────────────────────┬──────────────────────────────────────┤
│           KMS                 │         CloudHSM                     │
├───────────────────────────────┼──────────────────────────────────────┤
│ Shared tenancy (multi-tenant) │ Dedicated HSM (single-tenant)        │
│ AWS manages hardware          │ YOU manage hardware config           │
│ Automatic key rotation ✅     │ No automatic rotation ❌             │
│ Automatic key generation ✅   │ You generate keys                    │
│ Integrated with AWS services  │ Custom applications                  │
│ FIPS 140-2 Level 2           │ FIPS 140-2 Level 3 (higher!)        │
│ Free tier available           │ Expensive (~$1.50/hr per HSM)        │
│ Most use cases               │ Strict compliance / contractual      │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 Decision:                                                        │
│  "Encryption, managed, easy" → KMS                                  │
│  "Dedicated hardware, full control, FIPS 140-2 Level 3,             │
│   regulatory requirement for single-tenant HSM" → CloudHSM          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Updated Security Decision Map

```
┌──────────────────────────────────────────────────────────────────┐
│      SECURITY — COMPLETE SERVICE DECISION MAP                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Who can access what?" (identity/permissions)                   │
│       → IAM                                                     │
│                                                                  │
│  "Who did what?" (API audit)                                     │
│       → CloudTrail                                              │
│                                                                  │
│  "Encrypt data, manage keys"                                     │
│       → KMS (or CloudHSM for dedicated hardware)                │
│                                                                  │
│  "DDoS protection"                                               │
│       → Shield                                                  │
│                                                                  │
│  "Block SQL injection / XSS / Layer 7"                           │
│       → WAF                                                     │
│                                                                  │
│  "Centralize security across accounts"                           │
│       → Firewall Manager                                        │
│                                                                  │
│  "Detect threats / anomalies / compromised instances"            │
│       → GuardDuty                                               │
│                                                                  │
│  "Find sensitive data (PII/PHI) in S3"                           │
│       → Macie                                                   │
│                                                                  │
│  "Vulnerability scan EC2 / containers / code"                    │
│       → Inspector                                               │
│                                                                  │
│  "Store secrets (DB passwords, API keys)"                        │
│       → Secrets Manager                                         │
│                                                                  │
│  "Single-tenant HSM, FIPS 140-2 Level 3"                        │
│       → CloudHSM                                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Security Topic | Verdict | Why |
|---------------|---------|-----|
| KMS (CMK, key policies, rotation) | ✅ Must know | Encryption is core at senior level |
| KMS vs CloudHSM | ✅ Must know | "Dedicated vs shared" decision |
| 3 ways to create CMK | ✅ Must know | Shows depth of understanding |
| Key policy vs IAM policy vs Grants | ✅ Must know | Access control design |
| Inspector (vulnerability scanning) | ✅ Must know | Security assessment for EC2/VPC |
| Macie (PII/PHI in S3) | ⚡ Know concept | "Sensitive data in S3" = Macie |
| CloudHSM (dedicated HSM) | ⚡ Know when to use | "FIPS 140-2 Level 3" or "single-tenant" |
| Auto-rotation rules (which keys support it) | ✅ Must know | Tricky detail question |


---

## 11. Secrets Manager vs Parameter Store ✅ MUST HAVE

> At 12 YOE, know when to use which. Both store secrets — but cost, rotation, and scale differ.

```
┌──────────────────────────────────────────────────────────────────────┐
│              SECRETS MANAGER vs PARAMETER STORE                       │
├───────────────────────────────┬──────────────────────────────────────┤
│      SECRETS MANAGER          │       PARAMETER STORE                │
├───────────────────────────────┼──────────────────────────────────────┤
│ Purpose-built for secrets     │ General config + secrets storage     │
│ Automatic rotation ✅         │ No automatic rotation ❌             │
│ Generates random passwords    │ Cannot generate passwords           │
│ Costs money ($0.40/secret/mo) │ FREE (Standard) / $0.05 (Advanced)  │
│ Up to 65KB per secret         │ Up to 8KB (Advanced) / 4KB (Std)    │
│ 10,000+ secrets supported     │ Standard: 10,000 max                │
│ KMS encryption                │ KMS encryption (optional)           │
│ CloudFormation integration    │ CloudFormation integration          │
│ Cross-account sharing ✅      │ Limited cross-account               │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 Decision:                                                        │
│  "Minimize cost" → Parameter Store (free tier)                       │
│  "Auto key rotation" → Secrets Manager                              │
│  ">10,000 parameters" → Secrets Manager                            │
│  "Generate passwords via CloudFormation" → Secrets Manager          │
│  "Store AMI IDs, config strings, non-sensitive" → Parameter Store   │
│  "Database credentials with auto-rotation" → Secrets Manager         │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Secrets Manager Key Facts

```
┌─────────────────────────────────────────────────────────────┐
│              SECRETS MANAGER TIPS                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Stores: DB credentials, API keys, SSH keys, passwords      │
│                                                             │
│  How apps access:                                           │
│  App ──▶ Secrets Manager API ──▶ Returns secret value      │
│  (never hard-code secrets in code!)                         │
│                                                             │
│  Encryption: KMS (in transit + at rest)                     │
│  Access: IAM policies (fine-grained)                        │
│                                                             │
│  ⚠️ WARNING: When you enable rotation, Secrets Manager     │
│     rotates credentials IMMEDIATELY!                        │
│     → Make sure ALL app instances use Secrets Manager       │
│       BEFORE enabling rotation, or they'll break!           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 12. AWS Certificate Manager (ACM) ✅ MUST HAVE

> Free SSL/TLS certificates with auto-renewal. Know which services it integrates with.

```
┌─────────────────────────────────────────────────────────────┐
│              AWS CERTIFICATE MANAGER (ACM)                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Create, manage, and deploy SSL/TLS certificates      │
│        for FREE                                             │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            ACM Certificate                            │  │
│  │                 │                                     │  │
│  │     ┌───────────┼──────────────┐                     │  │
│  │     ▼           ▼              ▼                     │  │
│  │  ┌──────┐  ┌──────────┐  ┌─────────┐               │  │
│  │  │ ALB  │  │CloudFront│  │API G/W  │               │  │
│  │  └──────┘  └──────────┘  └─────────┘               │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Benefits:                                                  │
│  💰 FREE (you only pay for the resources using the cert)   │
│  🔄 Automated renewal (no manual cert rotation!)           │
│  🚀 Easy setup (few clicks vs manual CSR process)          │
│  🔗 Integrates with: ELB, CloudFront, API Gateway          │
│                                                             │
│  🧠 "SSL certificate for ALB/CloudFront" → ACM             │
│  🧠 "Free SSL with auto-renewal" → ACM                     │
│  🧠 NOT for EC2 directly (install cert on LB, not instance)│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 13. AWS Audit Manager ⚡ GOOD TO KNOW

> Continuous compliance auditing — produces reports for auditors.

```
┌─────────────────────────────────────────────────────────────┐
│               AWS AUDIT MANAGER                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Automated, continuous auditing of your AWS usage     │
│        Produces audit-ready reports for compliance           │
│                                                             │
│  Frameworks supported:                                      │
│  • PCI DSS (payment card industry)                          │
│  • GDPR (EU data protection)                                │
│  • HIPAA (healthcare)                                       │
│  • SOC 2                                                    │
│                                                             │
│  🧠 "Continuous auditing" + "compliance reports" → Audit Manager│
│  🧠 "Automate audit reports for PCI/GDPR/HIPAA" → Audit Manager│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 14. AWS Artifact ⚡ GOOD TO KNOW (Distractor!)

> Simply a download portal for AWS compliance reports. Often used as a WRONG answer choice.

```
┌─────────────────────────────────────────────────────────────┐
│                   AWS ARTIFACT                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Self-service portal to download AWS compliance       │
│        reports and agreements                               │
│                                                             │
│  • Download: SOC reports, PCI reports, ISO certs            │
│  • Accept: AWS agreements (BAA for HIPAA, etc.)             │
│                                                             │
│  ⚠️ This is just a DOWNLOAD portal for AWS's OWN reports   │
│     It does NOT audit YOUR usage (that's Audit Manager)     │
│                                                             │
│  🧠 "Need AWS's compliance reports" → Artifact              │
│  🧠 "Audit MY account's compliance" → Audit Manager        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 15. Amazon Detective ⚡ GOOD TO KNOW

> Root cause analysis of security incidents. Don't confuse with Inspector!

```
┌─────────────────────────────────────────────────────────────┐
│                  AMAZON DETECTIVE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Analyze, investigate, and find ROOT CAUSE of         │
│        security issues or suspicious activities             │
│                                                             │
│  Sources:                                                   │
│  • VPC Flow Logs                                            │
│  • CloudTrail Logs                                          │
│  • EKS Audit Logs                                           │
│  • GuardDuty Findings                                       │
│                                                             │
│  Uses: ML, statistical analysis, graph theory               │
│  → Builds visual map of resource interactions over time     │
│                                                             │
│  ⚠️ DON'T CONFUSE:                                         │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Detective  = INVESTIGATE root cause (after alert)    │  │
│  │  Inspector  = SCAN for vulnerabilities (proactive)    │  │
│  │  GuardDuty  = DETECT threats (alerts you)             │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  🧠 "Investigate root cause of security issue" → Detective  │
│  🧠 "What happened after GuardDuty alert?" → Detective     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 16. AWS Network Firewall ⚡ GOOD TO KNOW

> Physical-level firewall for VPCs. Know it exists for "filter before IGW" and "IPS" scenarios.

```
┌─────────────────────────────────────────────────────────────┐
│               AWS NETWORK FIREWALL                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Managed physical firewall across your VPCs           │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  INTERNET                                             │  │
│  │     │                                                 │  │
│  │     ▼                                                 │  │
│  │  ┌──────────────────┐                                 │  │
│  │  │ NETWORK FIREWALL │ ← Filters BEFORE reaching IGW  │  │
│  │  │ (IPS, DPI, rules)│                                 │  │
│  │  └────────┬─────────┘                                 │  │
│  │           ▼                                            │  │
│  │  ┌──────────────────┐                                 │  │
│  │  │   Your VPC       │                                 │  │
│  │  │   (Protected)    │                                 │  │
│  │  └──────────────────┘                                 │  │
│  │                                                       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Features:                                                  │
│  • Intrusion Prevention System (IPS)                        │
│  • Active traffic flow inspection                           │
│  • Block outbound SMB requests (malware spread)             │
│  • Works with Firewall Manager (multi-account)              │
│  • Physical infrastructure managed by AWS                   │
│                                                             │
│  🧠 "Filter traffic before internet gateway" → Network Firewall│
│  🧠 "IPS / hardware firewall" → Network Firewall           │
│  🧠 NOT the same as Security Groups or NACLs               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 17. AWS Security Hub ⚡ GOOD TO KNOW

> Single pane of glass for ALL security findings across services and accounts.

```
┌─────────────────────────────────────────────────────────────┐
│                 AWS SECURITY HUB                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Centralized view of ALL security alerts              │
│        from multiple AWS security services                  │
│                                                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │                                                     │    │
│  │  GuardDuty ─────┐                                   │    │
│  │  Inspector ─────┤                                   │    │
│  │  Macie ─────────┼──▶  SECURITY HUB  ──▶ Dashboard  │    │
│  │  Firewall Mgr ──┤    (single pane       + Actions   │    │
│  │  IAM Access     │     of glass)                     │    │
│  │  Analyzer ──────┘                                   │    │
│  │                                                     │    │
│  └────────────────────────────────────────────────────┘    │
│                                                             │
│  • Works across multiple accounts                           │
│  • Automated compliance checks (CIS, PCI DSS)              │
│  • Aggregates + prioritizes findings                        │
│                                                             │
│  🧠 "Single place for all security alerts" → Security Hub  │
│  🧠 "View GuardDuty + Inspector + Macie together" → Security Hub│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Security Services — The Full Family (Summary)

```
┌──────────────────────────────────────────────────────────────────────┐
│          AWS SECURITY SERVICES — COMPLETE MAP                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  IDENTITY & ACCESS                                                   │
│  ├── IAM (who can access what)                                      │
│  ├── Secrets Manager (store/rotate credentials)                     │
│  ├── Parameter Store (config + secrets, cheap)                      │
│  └── ACM (free SSL certificates)                                    │
│                                                                      │
│  DETECTION & MONITORING                                              │
│  ├── CloudTrail (API audit — who did what)                          │
│  ├── GuardDuty (AI threat detection)                                │
│  ├── Inspector (vulnerability scanning)                             │
│  ├── Macie (sensitive data in S3)                                   │
│  └── Detective (root cause investigation)                           │
│                                                                      │
│  PROTECTION                                                          │
│  ├── Shield (DDoS — Layer 3/4)                                      │
│  ├── WAF (Layer 7 — SQL injection, XSS, IP block)                  │
│  ├── Network Firewall (IPS, deep packet inspection)                 │
│  └── Firewall Manager (centralize across accounts)                  │
│                                                                      │
│  ENCRYPTION                                                          │
│  ├── KMS (managed key service)                                      │
│  └── CloudHSM (dedicated hardware HSM)                              │
│                                                                      │
│  GOVERNANCE & COMPLIANCE                                             │
│  ├── Audit Manager (continuous compliance reports)                  │
│  ├── Artifact (download AWS's compliance docs)                      │
│  └── Security Hub (single pane of glass)                            │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Security Topic | Verdict | Why |
|---------------|---------|-----|
| Secrets Manager vs Parameter Store | ✅ Must know | "Where to store secrets?" — always asked |
| Secrets Manager rotation warning | ✅ Must know | Rotates IMMEDIATELY — break apps if not ready |
| ACM (free SSL, auto-renew) | ✅ Must know | SSL on ALB/CloudFront = ACM |
| Audit Manager | ⚡ Know trigger | "Continuous audit reports" |
| Artifact | ⚡ Know it's a distractor | "Download AWS compliance docs" — that's it |
| Detective vs Inspector vs GuardDuty | ✅ Must know difference | Often confused — know each role |
| Network Firewall | ⚡ Know trigger | "IPS / filter before IGW" |
| Security Hub | ⚡ Know concept | "Single pane of glass for security" |


---

---

# 🏗️ CATEGORY: INFRASTRUCTURE AS CODE & DEPLOYMENT

---

## 1. AWS CloudFormation ✅ MUST HAVE

> At 12 YOE, you should be designing IaC strategies. Know template structure, immutable architecture, and rollback behavior.

### What is CloudFormation?

**Infrastructure as Code (IaC)** — define your ENTIRE AWS architecture in a JSON/YAML template and deploy it consistently, anywhere, anytime.

```
┌─────────────────────────────────────────────────────────────┐
│                 CLOUDFORMATION AT A GLANCE                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📝 Define infrastructure in code (JSON/YAML template)      │
│  🔁 Repeatable — same template → same infrastructure        │
│  🌍 Cross-region — deploy template in any region            │
│  ↩️  Auto-rollback on error (last known good state)          │
│  🗑️  Easily create AND destroy entire architectures         │
│  🏗️  Immutable architecture — don't modify, replace!        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### CloudFormation Template Structure

```
┌─────────────────────────────────────────────────────────────┐
│          CLOUDFORMATION TEMPLATE SECTIONS                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  AWSTemplateFormatVersion: "2010-09-09"              │   │
│  │                                                      │   │
│  │  Description: "My production stack"                  │   │
│  │                                                      │   │
│  │  Parameters:      ← Input values (instance type,    │   │
│  │                      env name, etc.)                  │   │
│  │                                                      │   │
│  │  Mappings:        ← Static lookup tables             │   │
│  │                     (AMI per region, etc.)            │   │
│  │                                                      │   │
│  │  Conditions:      ← If/else logic                    │   │
│  │                     (prod vs dev settings)            │   │
│  │                                                      │   │
│  │  Resources:       ← THE ONLY REQUIRED SECTION!       │   │
│  │                     (EC2, S3, VPC, RDS, etc.)         │   │
│  │                                                      │   │
│  │  Outputs:         ← Values to export                 │   │
│  │                     (ALB DNS, DB endpoint, etc.)      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  🧠 Only RESOURCES section is required!                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Immutable Architecture — Diagram

```
┌──────────────────────────────────────────────────────────────┐
│          IMMUTABLE ARCHITECTURE (CloudFormation Style)        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ❌ MUTABLE (bad — modify in place):                         │
│  Deploy v1 → SSH in → patch → configure → drift happens     │
│                                                              │
│  ✅ IMMUTABLE (good — replace entirely):                     │
│  Deploy v1 → Need change? → Deploy v2 (new stack)           │
│                           → Destroy v1                       │
│                                                              │
│  CloudFormation enables this:                                │
│  • Same template → consistent environment every time         │
│  • Tear down old → spin up new (no drift, no patches)       │
│  • Cross-region: deploy same template in any region          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### CloudFormation Key Facts

| Fact | Detail |
|------|--------|
| **Error handling** | Auto-rolls back to last known good state |
| **Cross-region** | Same template works in any region (but watch hard-coded values!) |
| **Hard-coded IDs = failure** | Don't hard-code resource IDs or AMI IDs — use Mappings/Parameters |
| **Makes API calls** | CloudFormation makes the same API calls you'd make manually |
| **Stacks** | A collection of resources managed as a single unit |
| **Change Sets** | Preview changes before applying them |
| **Drift Detection** | Identify resources that have been modified outside CloudFormation |

---

## 2. Elastic Beanstalk (PaaS) ⚡ GOOD TO KNOW

> "Bring your code, AWS does the rest." Know it exists but it's not commonly used at senior/architect level — production teams usually need more control.

### What is Elastic Beanstalk?

AWS's **Platform as a Service (PaaS)** — upload your code and Beanstalk handles deployment, scaling, load balancing, and health monitoring automatically.

```
┌─────────────────────────────────────────────────────────────┐
│               ELASTIC BEANSTALK AT A GLANCE                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  YOU provide: Application code                              │
│                                                             │
│  BEANSTALK provides:                                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  • EC2 instances (auto-provisioned)                   │  │
│  │  • Load Balancer (auto-configured)                    │  │
│  │  • Auto Scaling Group                                 │  │
│  │  • Security Groups                                    │  │
│  │  • CloudWatch monitoring                              │  │
│  │  • Deployment management (staging → production)       │  │
│  │  • OS patching                                        │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Supports: Java, .NET, PHP, Node.js, Python, Ruby, Go,     │
│            Docker, Containers, Windows, Linux               │
│                                                             │
│  ⚠️ NOT serverless — creates real EC2 instances underneath  │
│  ⚠️ Good starting point, but limited for complex prod apps  │
│                                                             │
│  🧠 "Simple deployment, bring your code" → Elastic Beanstalk│
│  🧠 "PaaS on AWS" → Elastic Beanstalk                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Beanstalk vs Other Options

```
┌──────────────────────────────────────────────────────────────┐
│          DEPLOYMENT OPTIONS COMPARED                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  More Control ◄──────────────────────────────▶ Less Control  │
│  More Work                                       Less Work   │
│                                                              │
│  ┌────────┐    ┌────────────┐    ┌──────────┐    ┌───────┐ │
│  │  EC2   │    │ ECS/EKS +  │    │ Elastic  │    │Lambda │ │
│  │(manual)│    │ Fargate    │    │Beanstalk │    │(func) │ │
│  └────────┘    └────────────┘    └──────────┘    └───────┘ │
│                                                              │
│  Full control  Container       PaaS (code +    Serverless   │
│  You manage    orchestration   auto-deploy)    Event-driven │
│  everything    + serverless                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. AWS Systems Manager ✅ MUST HAVE

> At 12 YOE, know Systems Manager features by name — especially Session Manager, Patch Manager, and Parameter Store. It's your ops automation toolkit.

### What is Systems Manager?

A **suite of tools** to view, control, and automate BOTH AWS and on-premises infrastructure.

```
┌─────────────────────────────────────────────────────────────┐
│              AWS SYSTEMS MANAGER                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🧰 "Swiss Army knife for operations"                       │
│  ☁️  Works on AWS (EC2) AND on-premises servers             │
│  🤖 Automate patching, configuration, compliance            │
│                                                             │
│  Key Features (know these by name!):                        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  SESSION MANAGER                                     │   │
│  │  • SSH into EC2 WITHOUT opening port 22!             │   │
│  │  • No SSH keys needed, no bastion host               │   │
│  │  • Fully audited (logs in S3/CloudWatch)             │   │
│  │  🧠 "Secure instance access without SSH" → Session Mgr│  │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  PATCH MANAGER                                       │   │
│  │  • Automate OS and application patching              │   │
│  │  • Define patch baselines and schedules              │   │
│  │  • Works on-prem + cloud                             │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  AUTOMATION (Documents / Runbooks)                   │   │
│  │  • Predefined or custom automation workflows         │   │
│  │  • Fix S3 bucket permissions, restart services       │   │
│  │  • Usable by AWS Config for auto-remediation         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  PARAMETER STORE                                     │   │
│  │  • Store config values and secrets                   │   │
│  │  • Hierarchical (e.g., /prod/db/password)            │   │
│  │  • Free tier available                               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  RUN COMMAND                                         │   │
│  │  • Execute commands on fleet of instances            │   │
│  │  • No SSH needed, uses SSM Agent                     │   │
│  │  • Run shell scripts at scale                        │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  INVENTORY                                           │   │
│  │  • Collect metadata about instances                  │   │
│  │  • Installed software, OS details, network config    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Systems Manager Key Facts

| Fact | Detail |
|------|--------|
| **On-prem + Cloud** | Manages BOTH (requires SSM Agent) |
| **Features by name** | Exam uses feature names, not "Systems Manager" |
| **Config integration** | AWS Config uses SSM Automation for remediation |
| **Session Manager** | Replace bastion hosts + SSH keys entirely |
| **Free** | Most features are free (you pay for instances) |

---

### IaC & Deployment — Decision Map

```
┌──────────────────────────────────────────────────────────────┐
│      INFRASTRUCTURE & DEPLOYMENT — WHICH SERVICE?            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "Define infrastructure in code (IaC)"                       │
│       → CloudFormation (or Terraform if multi-cloud)        │
│                                                              │
│  "Immutable architecture, deploy same stack anywhere"        │
│       → CloudFormation                                      │
│                                                              │
│  "Simple deploy — just bring code, auto-manage everything"   │
│       → Elastic Beanstalk                                   │
│                                                              │
│  "Automate patching across fleet"                            │
│       → Systems Manager (Patch Manager)                     │
│                                                              │
│  "SSH into EC2 without port 22 / bastion"                    │
│       → Systems Manager (Session Manager)                   │
│                                                              │
│  "Execute commands on multiple instances"                     │
│       → Systems Manager (Run Command)                       │
│                                                              │
│  "Auto-remediate non-compliant resources"                    │
│       → AWS Config + SSM Automation                         │
│                                                              │
│  "Store config values / secrets cheaply"                     │
│       → Systems Manager (Parameter Store)                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| IaC & Deployment Topic | Verdict | Why |
|-----------------------|---------|-----|
| CloudFormation (template sections, rollback) | ✅ Must know | IaC is expected at senior level |
| Immutable architecture concept | ✅ Must know | Production deployment philosophy |
| Hard-coded values = failure | ✅ Must know | Common trap question |
| Elastic Beanstalk | ⚡ Know what it is | "PaaS / bring code" — occasionally asked |
| Systems Manager — Session Manager | ✅ Must know | "No SSH" = Session Manager |
| Systems Manager — Patch Manager | ✅ Must know | Fleet patching automation |
| Systems Manager — Automation | ⚡ Know concept | Remediation workflows |
| Beanstalk vs Lambda vs ECS | ✅ Must know | "Which deployment model?" |
| CloudFormation Change Sets + Drift | ⚡ Know concepts | Shows depth |


---

---

# 🚀 CATEGORY: CACHING & CONTENT DELIVERY

---

## Overview — Where to Cache in AWS

> AWS loves caches. On the exam and in production: "Can we put a cache here?" → Probably yes.

```
┌─────────────────────────────────────────────────────────────────────┐
│                  AWS CACHING LAYERS                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─── EXTERNAL CACHE (closer to users) ────────────────────────┐   │
│  │  CloudFront (CDN) — static content, images, videos, APIs    │   │
│  │  Global Accelerator — IP caching, routing optimization      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─── INTERNAL CACHE (closer to database) ─────────────────────┐   │
│  │  ElastiCache (Redis/Memcached) — front any database          │   │
│  │  DAX — specific to DynamoDB only                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Full caching architecture:                                         │
│                                                                     │
│  User → CloudFront → ALB → App → ElastiCache → RDS                │
│                                    (or DAX)   (or DynamoDB)         │
│                                                                     │
│  Every layer = less load on the next layer = faster + cheaper       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1. CloudFront (CDN) ✅ MUST HAVE

> At 12 YOE, know CloudFront as your global content delivery layer. Any "slow for global users" problem → CloudFront.

### What is CloudFront?

A **Content Delivery Network (CDN)** — caches content at 400+ edge locations worldwide for low-latency delivery.

```
┌─────────────────────────────────────────────────────────────┐
│                  CLOUDFRONT AT A GLANCE                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌────────────┐                    ┌─────────────────┐     │
│  │   User     │                    │    ORIGIN        │     │
│  │ (Sydney)   │                    │  (S3, ALB, EC2,  │     │
│  └─────┬──────┘                    │   custom HTTP)   │     │
│        │                           └────────┬────────┘     │
│        ▼                                    │              │
│  ┌──────────────┐         Cache miss?       │              │
│  │ Edge Location│◄─────────────────────────►│              │
│  │ (nearest to  │         Fetch from origin │              │
│  │  user)       │                           │              │
│  │              │  Cache hit? → Return       │              │
│  │              │  instantly! (< 10ms)       │              │
│  └──────────────┘                                          │
│                                                             │
│  Result: User in Sydney gets content from Sydney edge,      │
│          NOT from us-east-1 origin (200ms+ saved!)          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### CloudFront Key Settings

| Setting | Detail |
|---------|--------|
| **HTTPS** | Default. Can add custom SSL certificate (via ACM) |
| **Origins** | AWS (S3, ALB, EC2) + non-AWS (any HTTP server) |
| **Distribution** | Global — you can't pick specific edge locations |
| **Geo-Restriction** | Block entire countries (whitelist or blacklist) |
| **TTL (Time to Live)** | How long content stays cached. Can force expiration (invalidation) |
| **Price Class** | Choose All, 200, or 100 edge locations (cost optimization) |
| **OAI/OAC** | Restrict S3 access to CloudFront only (Origin Access) |

### CloudFront Use Cases

```
┌─────────────────────────────────────────────────────────────┐
│  🧠 "Slow performance for global users" → CloudFront        │
│  🧠 "Cache static content (images, CSS, JS)" → CloudFront   │
│  🧠 "Serve S3 content securely + fast" → CloudFront + OAC  │
│  🧠 "HTTPS for custom domain" → CloudFront + ACM           │
│  🧠 "Block country access" → CloudFront Geo-Restriction     │
│     (or WAF for more granular control)                      │
│  🧠 "Any external performance issue" → CloudFront           │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. ElastiCache ✅ MUST HAVE

> Database caching layer. Know Redis vs Memcached — the difference is ALWAYS asked.

### What is ElastiCache?

A **managed in-memory cache** — reduces database load by caching frequently accessed data in memory.

```
┌─────────────────────────────────────────────────────────────┐
│                  ELASTICACHE AT A GLANCE                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Without cache:                                             │
│  App → RDS (every request hits DB → slow, expensive)        │
│                                                             │
│  With cache:                                                │
│  App → ElastiCache → Cache HIT? → Return instantly (μs)     │
│                    → Cache MISS? → Query RDS → Store in cache│
│                                                             │
│  Result: 80-90% of reads served from cache                  │
│          Database load drops dramatically                    │
│          Response time: milliseconds → microseconds          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Redis vs Memcached ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────────┐
│                    REDIS vs MEMCACHED                                 │
├───────────────────────────────┬──────────────────────────────────────┤
│          REDIS                │        MEMCACHED                     │
├───────────────────────────────┼──────────────────────────────────────┤
│ Multi-AZ with failover ✅     │ No Multi-AZ ❌                       │
│ Read replicas (HA) ✅         │ No replication ❌                    │
│ Backups & restore ✅          │ No backups ❌                        │
│ Data persistence ✅           │ No persistence (pure cache)          │
│ Complex data types ✅         │ Simple key-value only                │
│ (lists, sets, sorted sets)    │                                      │
│ Pub/Sub messaging ✅          │ No pub/sub ❌                        │
│ CAN be a standalone DB        │ ONLY a cache                        │
│ Single-threaded               │ Multi-threaded                      │
│ More features                 │ Simpler, slightly faster for basic   │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 Decision:                                                        │
│  "Need HA, backups, persistence, complex data" → Redis              │
│  "Simple cache, multi-threaded, no persistence needed" → Memcached  │
│  "Can be standalone database" → Redis                               │
│  "Just cache, disposable" → Memcached                               │
│                                                                      │
│  DEFAULT CHOICE: Redis (more features, safer, HA)                   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### ElastiCache Use Cases

| Use Case | How |
|----------|-----|
| **Session store** | Store user sessions in Redis (stateless EC2 instances) |
| **Database cache** | Cache RDS query results (reduce DB load) |
| **Leaderboards** | Redis sorted sets (real-time rankings) |
| **Pub/Sub messaging** | Redis pub/sub for real-time notifications |
| **Rate limiting** | Count API calls per user with TTL keys |

---

## 3. DAX (DynamoDB Accelerator) ✅ MUST HAVE

> In-memory cache SPECIFICALLY for DynamoDB. Know it's DynamoDB-only.

```
┌─────────────────────────────────────────────────────────────┐
│                    DAX (DynamoDB Accelerator)                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: In-memory cache that sits in front of DynamoDB       │
│                                                             │
│  ┌──────┐      ┌──────┐      ┌──────────────┐             │
│  │ App  │─────▶│ DAX  │─────▶│  DynamoDB    │             │
│  │      │      │(cache)│      │  (if miss)   │             │
│  └──────┘      └──────┘      └──────────────┘             │
│                                                             │
│  Performance:                                               │
│  • DynamoDB: single-digit milliseconds                      │
│  • DAX: MICROSECONDS (10x faster!)                          │
│                                                             │
│  Key facts:                                                 │
│  • DynamoDB-ONLY (cannot use with RDS/other DBs)            │
│  • Lives inside YOUR VPC                                    │
│  • You control: node size, node count, TTL, maintenance     │
│  • Highly available (multi-node cluster)                    │
│  • API-compatible (drop-in replacement, no code change!)    │
│                                                             │
│  🧠 "Speed up DynamoDB reads" → DAX                         │
│  🧠 "Microsecond latency for DynamoDB" → DAX                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### DAX vs ElastiCache — When to Use Which

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DAX vs ELASTICACHE                                 │
├───────────────────────────────┬──────────────────────────────────────┤
│           DAX                 │        ELASTICACHE                   │
├───────────────────────────────┼──────────────────────────────────────┤
│ DynamoDB ONLY                 │ Any database (RDS, custom, etc.)     │
│ Drop-in (no code change)      │ Requires app-level caching logic     │
│ Lives in VPC                  │ Lives in VPC                         │
│ Table-level cache             │ Application-level cache              │
│ Microsecond response          │ Sub-millisecond response             │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 "Cache DynamoDB" → DAX                                           │
│  🧠 "Cache RDS / general purpose" → ElastiCache (Redis)             │
│  🧠 "Cache session data / custom logic" → ElastiCache (Redis)       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 4. AWS Global Accelerator ✅ MUST HAVE

> Fixes IP caching problems. Routes traffic through AWS backbone (not public internet). Know the difference from CloudFront.

### What is Global Accelerator?

A **networking service** that routes user traffic through AWS's global network, providing 2 static IPs and improving performance by up to 60%.

```
┌─────────────────────────────────────────────────────────────┐
│               GLOBAL ACCELERATOR AT A GLANCE                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  WITHOUT Global Accelerator:                                │
│  User → Public Internet (many hops, variable latency) → App│
│                                                             │
│  WITH Global Accelerator:                                   │
│  User → Nearest Edge → AWS Global Network (fast!) → App    │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                                                       │  │
│  │  User ──▶ Edge Location ──▶ AWS Backbone ──▶ ALB/EC2 │  │
│  │           (nearest)         (private, fast)           │  │
│  │                                                       │  │
│  │  You get: 2 STATIC Anycast IPs                       │  │
│  │  (never change, even if backend changes!)             │  │
│  │                                                       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  3 Key Features:                                            │
│  1. Masks complex architecture (users see same 2 IPs)       │
│  2. Speeds things up (AWS backbone vs public internet)      │
│  3. Weighted pools (test features, handle failover)         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### CloudFront vs Global Accelerator ✅ MUST HAVE

```
┌──────────────────────────────────────────────────────────────────────┐
│              CLOUDFRONT vs GLOBAL ACCELERATOR                         │
├───────────────────────────────┬──────────────────────────────────────┤
│       CLOUDFRONT              │     GLOBAL ACCELERATOR               │
├───────────────────────────────┼──────────────────────────────────────┤
│ Content CACHING (CDN)         │ Network ROUTING (no caching)         │
│ Caches at edge locations      │ Routes to nearest edge, then AWS net │
│ HTTP/HTTPS content             │ TCP/UDP traffic (any protocol)       │
│ Static & dynamic content      │ Gaming, IoT, VoIP, HTTP             │
│ Returns cached response       │ Proxies connection to origin         │
│ Unique domain (d123.cf.net)   │ 2 static Anycast IPs                │
│ Improves: content delivery    │ Improves: connection performance     │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 "Cache static content globally" → CloudFront                    │
│  🧠 "IP caching problem" → Global Accelerator                       │
│  🧠 "Static IPs needed for whitelisting" → Global Accelerator       │
│  🧠 "Non-HTTP (TCP/UDP) performance" → Global Accelerator           │
│  🧠 "Speed up HTTP content" → CloudFront                            │
│  🧠 "Users can't reach new endpoint after failover" →               │
│     Global Accelerator (static IPs solve IP caching)                │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Caching — Complete Decision Map

```
┌──────────────────────────────────────────────────────────────────┐
│          CACHING — WHICH SERVICE FOR WHICH SCENARIO?             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Speed up static content for global users"                      │
│       → CloudFront (CDN)                                        │
│                                                                  │
│  "Cache database queries (RDS)"                                  │
│       → ElastiCache (Redis or Memcached)                        │
│                                                                  │
│  "Cache DynamoDB reads (microsecond)"                            │
│       → DAX                                                     │
│                                                                  │
│  "Users can't reach app after IP change / failover"              │
│       → Global Accelerator (static IPs)                         │
│                                                                  │
│  "Stateless instances (store sessions externally)"               │
│       → ElastiCache Redis (session store)                       │
│                                                                  │
│  "Non-HTTP (gaming/IoT) performance for global users"            │
│       → Global Accelerator                                      │
│                                                                  │
│  "Need HA cache with backups"                                    │
│       → ElastiCache Redis (not Memcached)                       │
│                                                                  │
│  "Simple disposable cache, multi-threaded"                       │
│       → ElastiCache Memcached                                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Caching Topic | Verdict | Why |
|--------------|---------|-----|
| CloudFront (CDN, settings, origins) | ✅ Must know | "Slow globally" = CloudFront |
| CloudFront vs Global Accelerator | ✅ Must know | Classic confusion — know the difference |
| ElastiCache Redis vs Memcached | ✅ Must know cold | Always asked — Redis = HA, Memcached = simple |
| DAX (DynamoDB only) | ✅ Must know | "Speed up DynamoDB" = DAX |
| DAX vs ElastiCache | ✅ Must know | DynamoDB = DAX, everything else = ElastiCache |
| Global Accelerator (static IPs) | ✅ Must know | "IP caching" or "static IPs" = GA |
| Caching strategy (where to cache) | ✅ Must know | External (CDN) + Internal (DB cache) |
| CloudFront Geo-Restriction | ⚡ Know concept | "Block countries" = Geo-Restriction (or WAF) |


---

---

# 🏛️ CATEGORY: GOVERNANCE & MULTI-ACCOUNT

---

## 1. AWS Organizations ✅ MUST HAVE

> At 12 YOE, multi-account strategy is expected. Know SCPs, consolidated billing, and centralized logging.

### What is AWS Organizations?

A **free governance tool** to create, manage, and control multiple AWS accounts from a single location.

```
┌─────────────────────────────────────────────────────────────┐
│                  AWS ORGANIZATIONS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── Management Account (Root) ───────────────────────┐   │
│  │  • Pays all the bills (Consolidated Billing)         │   │
│  │  • Creates/destroys accounts programmatically        │   │
│  │  • Applies Service Control Policies (SCPs)           │   │
│  └──────────────────────────────────────────────────────┘   │
│         │                                                   │
│         ├── OU: Production                                  │
│         │    ├── Account: prod-app                          │
│         │    ├── Account: prod-data                         │
│         │    └── SCP: "No delete on S3"                     │
│         │                                                   │
│         ├── OU: Development                                 │
│         │    ├── Account: dev-team-1                        │
│         │    ├── Account: dev-team-2                        │
│         │    └── SCP: "us-east-1 only"                      │
│         │                                                   │
│         ├── OU: Security / Logging                          │
│         │    └── Account: central-logging                   │
│         │         (CloudTrail logs aggregated here)         │
│         │                                                   │
│         └── OU: Sandbox                                     │
│              └── Account: experiments                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Key Features

| Feature | What It Does |
|---------|-------------|
| **Consolidated Billing** | One bill for all accounts. Volume discounts apply across all. |
| **Service Control Policies (SCPs)** | Restrict permissions across entire accounts/OUs |
| **Programmatic Account Creation** | Create/destroy accounts via API |
| **Centralized Logging** | Dedicated logging account (CloudTrail aggregation) |
| **Reserved Instance Sharing** | RIs shared across all accounts in the org |
| **Organizational Units (OUs)** | Group accounts logically (Prod, Dev, Security) |

---

### Service Control Policies (SCPs) ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────────┐
│              SERVICE CONTROL POLICIES (SCPs)                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: The ULTIMATE permission boundary                     │
│  Applied to: Every resource in the account(s)               │
│  Override: Even restricts the ROOT user!                     │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  IAM Policy says: "Allow *"                           │  │
│  │  SCP says: "Deny s3:DeleteBucket"                     │  │
│  │                                                       │  │
│  │  Result: User CANNOT delete S3 buckets                │  │
│  │          (SCP wins over IAM — it's a guardrail)       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  SCPs do NOT grant permissions — they only RESTRICT         │
│  (You still need IAM policies to allow actions)             │
│                                                             │
│  Common SCP use cases:                                      │
│  • Prevent leaving the organization                         │
│  • Restrict regions (only allow us-east-1)                  │
│  • Prevent disabling CloudTrail                             │
│  • Deny ability to delete logs                              │
│  • Require encryption on all S3 buckets                     │
│                                                             │
│  🧠 "Centralize logs + prevent anyone from editing/deleting"│
│     → Organizations + SCPs on logging account               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. AWS RAM (Resource Access Manager) ⚡ GOOD TO KNOW

> Share AWS resources across accounts without duplicating them.

```
┌─────────────────────────────────────────────────────────────┐
│                      AWS RAM                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Share resources across accounts (free!)              │
│                                                             │
│  Shareable Resources:                                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  • VPC Subnets (most common!)                         │  │
│  │  • Transit Gateways                                   │  │
│  │  • License Manager configurations                     │  │
│  │  • Dedicated Hosts                                    │  │
│  │  • Route 53 Resolver rules                            │  │
│  │  • Aurora DB clusters                                 │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─── Account A ────────┐    ┌─── Account B ────────┐     │
│  │                       │    │                       │     │
│  │  VPC Subnet           │    │  EC2 instances        │     │
│  │  (shared via RAM) ────┼───▶│  (launched in         │     │
│  │                       │    │   shared subnet)      │     │
│  └───────────────────────┘    └───────────────────────┘     │
│                                                             │
│  Benefits:                                                  │
│  • No duplication — share existing resources                │
│  • Free service (consumer pays for usage)                   │
│  • Works within Organizations                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### RAM vs VPC Peering

```
┌──────────────────────────────────────────────────────────────┐
│               RAM vs VPC PEERING                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "Share resources within SAME region" → RAM                  │
│  "Connect networks ACROSS regions"    → VPC Peering          │
│  "Share a subnet with another account" → RAM                 │
│  "Both available?" → RAM is simpler for same-region sharing │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Cross-Account Role Access ✅ MUST HAVE

> At 12 YOE, this is standard practice. NEVER duplicate IAM users across accounts.

```
┌─────────────────────────────────────────────────────────────┐
│            CROSS-ACCOUNT ROLE ACCESS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ❌ BAD: Create IAM user in every account                   │
│     (duplicate credentials = security vulnerability)        │
│                                                             │
│  ✅ GOOD: Assume a Role in the target account               │
│     (temporary credentials, auditable, revokable)           │
│                                                             │
│  ┌─── Account A (Dev) ───┐    ┌─── Account B (Prod) ───┐  │
│  │                        │    │                         │  │
│  │  User: Alice           │    │  Role: ProdReadOnly     │  │
│  │    │                   │    │    (trust: Account A)   │  │
│  │    │ sts:AssumeRole ───┼───▶│    │                    │  │
│  │    │                   │    │    ▼                    │  │
│  │    │                   │    │  Temp credentials       │  │
│  │    │◀──────────────────┼────│  (1 hour, renewable)   │  │
│  │    │                   │    │                         │  │
│  │  Alice accesses Prod   │    │                         │  │
│  │  with temp creds       │    │                         │  │
│  └────────────────────────┘    └─────────────────────────┘  │
│                                                             │
│  Key facts:                                                 │
│  • Role assumption is TEMPORARY (not permanent)             │
│  • No credentials stored in target account                  │
│  • Fully auditable (CloudTrail logs the AssumeRole)         │
│  • Temporary employees → role access only, no IAM user      │
│                                                             │
│  🧠 "Credentials mentioned" → scan for ROLE answers         │
│  🧠 "Cross-account access" → AssumeRole (not IAM users)    │
│  🧠 "Temporary access for contractors" → Cross-account role │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. AWS Config ✅ MUST HAVE

> Infrastructure compliance and inventory management. Know it enforces STANDARDS.

### What is AWS Config?

An **inventory management and compliance tool** — tracks resource configuration history and evaluates against rules you define.

```
┌─────────────────────────────────────────────────────────────┐
│                      AWS CONFIG                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What it does:                                              │
│                                                             │
│  ┌─── RECORD ──────────────────────────────────────────┐   │
│  │  • Tracks ALL configuration changes to resources     │   │
│  │  • Maintains configuration HISTORY                   │   │
│  │  • Can track DELETED resources                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─── EVALUATE ────────────────────────────────────────┐   │
│  │  • Config Rules check if resources are COMPLIANT     │   │
│  │  • Pre-built rules OR custom Lambda rules            │   │
│  │  • Examples:                                         │   │
│  │    - "Are S3 buckets public?" (should be NO)         │   │
│  │    - "Are EC2 using approved AMIs?"                  │   │
│  │    - "Is EBS encryption enabled?"                    │   │
│  │    - "Are Security Groups too open?"                 │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─── REMEDIATE ───────────────────────────────────────┐   │
│  │  • SSM Automation Documents (auto-fix)               │   │
│  │  • Lambda functions (custom remediation)             │   │
│  │  • Example: Auto-remove public access from S3        │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  Consolidation: Roll up results to single region            │
│  Multi-account: Works with Organizations (aggregator)       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Config Flow — Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                  AWS CONFIG FLOW                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Resource changes ──▶ Config records it                      │
│                            │                                 │
│                            ▼                                 │
│                    Config Rule evaluates                      │
│                            │                                 │
│                    ┌───────┴───────┐                         │
│                    ▼               ▼                         │
│              COMPLIANT ✅    NON-COMPLIANT ❌                │
│                                    │                         │
│                                    ▼                         │
│                            ┌──────────────┐                 │
│                            │ Remediation  │                 │
│                            │ (SSM / Lambda│                 │
│                            │  auto-fix)   │                 │
│                            └──────────────┘                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Config Key Facts

| Fact | Detail |
|------|--------|
| **Standards enforcement** | If a "standard" is mentioned → think Config |
| **History** | Full history of resource configuration changes |
| **Deleted resources** | Can track previously deleted AWS resources |
| **Remediation** | SSM Automation docs or Lambda to auto-fix |
| **Multi-account** | Aggregator consolidates across accounts/regions |
| **Not preventive** | Config DETECTS drift, doesn't prevent it (SCPs prevent) |

### Config vs CloudTrail

```
┌──────────────────────────────────────────────────────────────┐
│           CONFIG vs CLOUDTRAIL                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CloudTrail: WHO did WHAT (API calls — audit trail)          │
│  Config:     WHAT changed and is it COMPLIANT? (state)       │
│                                                              │
│  They complement each other:                                 │
│  CloudTrail → "Alice called DeleteBucket at 3pm"            │
│  Config → "S3 bucket was public (non-compliant) since 2pm"  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### Governance — Complete Decision Map

```
┌──────────────────────────────────────────────────────────────────┐
│       GOVERNANCE — WHICH SERVICE FOR WHICH SCENARIO?             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Manage multiple AWS accounts centrally"                        │
│       → AWS Organizations                                       │
│                                                                  │
│  "Restrict permissions across entire account (even root)"        │
│       → Service Control Policies (SCPs)                         │
│                                                                  │
│  "Centralize logs, prevent anyone from deleting them"            │
│       → Organizations + SCPs + dedicated logging account        │
│                                                                  │
│  "Share Reserved Instances across accounts"                      │
│       → Organizations (Consolidated Billing)                    │
│                                                                  │
│  "Share VPC subnets / Transit GW across accounts"                │
│       → AWS RAM                                                 │
│                                                                  │
│  "Cross-account access without duplicating users"                │
│       → Cross-Account Roles (AssumeRole)                        │
│                                                                  │
│  "Enforce standards / check compliance"                          │
│       → AWS Config (rules + remediation)                        │
│                                                                  │
│  "Track deleted resources / config history"                      │
│       → AWS Config                                              │
│                                                                  │
│  "Auto-fix non-compliant resources"                              │
│       → AWS Config + SSM Automation                             │
│                                                                  │
│  "Temporary access for external contractors"                     │
│       → Cross-Account Roles (temporary credentials)             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Governance Topic | Verdict | Why |
|-----------------|---------|-----|
| Organizations (SCPs, OUs, consolidated billing) | ✅ Must know | Multi-account is standard at senior level |
| SCPs (restrict even root, guardrails) | ✅ Must know | "Ultimate permission boundary" |
| Centralized logging pattern | ✅ Must know | Logging account + SCP protection |
| Cross-Account Roles | ✅ Must know | "Never duplicate IAM users" |
| AWS Config (rules, compliance, remediation) | ✅ Must know | "Enforce standards" = Config |
| Config vs CloudTrail | ✅ Must know | What changed vs who did it |
| AWS RAM | ⚡ Know concept | "Share resources same-region" = RAM |
| RAM vs VPC Peering | ⚡ Know decision | Same region = RAM, cross-region = Peering |
| Programmatic account creation | ⏭️ Skip details | Just know it's possible |


---

---

# 💰 CATEGORY: COST MANAGEMENT & OPTIMIZATION

---

## 1. AWS Cost Explorer ⚡ GOOD TO KNOW

> Visualize and analyze your cloud spend. Know it exists for "budgeting / controlling spend" scenarios.

```
┌─────────────────────────────────────────────────────────────┐
│                  AWS COST EXPLORER                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Visual tool to analyze and forecast AWS spending     │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            COST EXPLORER REPORTS                       │  │
│  │                                                       │  │
│  │  📊 By SERVICE — break down cost per AWS service      │  │
│  │  🏷️  By TAG — filter by project, team, environment    │  │
│  │  📅 By TIME — daily, monthly, custom range            │  │
│  │  🔮 FORECAST — predict next month's spend             │  │
│  │  📈 By CATEGORY — data transfer, compute, storage     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Key points:                                                │
│  • Can predict/estimate upcoming month's costs              │
│  • Tags are crucial for tracking spend by project/team      │
│  • Works with Budgets (set alerts based on Explorer data)   │
│                                                             │
│  🧠 "Visualize costs" or "budgeting" → Cost Explorer        │
│  🧠 "Where is the spend coming from?" → Cost Explorer       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. AWS Budgets ⚡ GOOD TO KNOW

> Set spending limits and get alerted BEFORE you overspend. Proactive cost control.

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS BUDGETS                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Set budget thresholds and get alerted when close     │
│                                                             │
│  4 Budget Types:                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  1. COST Budget      — "How much are we spending?"    │  │
│  │  2. USAGE Budget     — "How much are we using?"       │  │
│  │  3. RESERVATION Budget — "Are RIs efficient?"         │  │
│  │  4. SAVINGS PLANS Budget — "Savings Plan utilization?"│  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Features:                                                  │
│  • 2 free budgets per month                                 │
│  • Alert on CURRENT or PROJECTED spend                      │
│  • Filter by tags for specific budgets                      │
│  • Integrates with Cost Explorer                            │
│                                                             │
│  🧠 "Alert before overspending" → Budgets                   │
│  🧠 "Proactive cost control" → Budgets                      │
│  🧠 Cost Explorer = analyze PAST. Budgets = alert FUTURE.  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. AWS Cost and Usage Reports (CUR) ⚡ GOOD TO KNOW

> Most comprehensive and detailed cost data. Know it for "detailed breakdown" scenarios.

```
┌─────────────────────────────────────────────────────────────┐
│          AWS COST AND USAGE REPORTS (CUR)                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Most COMPREHENSIVE view of your AWS spending         │
│        Daily CSV reports published to S3                     │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  CUR → S3 Bucket → Analyze with:                     │  │
│  │                                                       │  │
│  │      ┌──────────┐  ┌──────────┐  ┌──────────────┐   │  │
│  │      │  Athena  │  │ Redshift │  │  QuickSight  │   │  │
│  │      │  (SQL)   │  │ (DW)     │  │ (Visualize)  │   │  │
│  │      └──────────┘  └──────────┘  └──────────────┘   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Breaks down by:                                            │
│  • Time span (hour, day, month)                             │
│  • Service and resource                                     │
│  • Tags (cost allocation)                                   │
│  • Data transfer charges (external + inter-region)          │
│                                                             │
│  Use cases:                                                 │
│  • Track Savings Plans utilization                          │
│  • Monitor On-Demand capacity reservations                  │
│  • Organization-level reporting (by OU or member account)   │
│  • Deep-dive into data transfer costs                       │
│                                                             │
│  🧠 "Most detailed/comprehensive cost data" → CUR          │
│  🧠 "Daily usage reports to S3" → CUR                       │
│  🧠 "Query billing data with SQL" → CUR + Athena           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Savings Plans ✅ MUST HAVE

> Flexible pricing for up to 72% savings. Know the 3 types and how they differ from Reserved Instances.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       SAVINGS PLANS                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  What: Commit to $/hour for 1 or 3 years → get up to 72% discount     │
│                                                                         │
│  3 Types:                                                               │
│                                                                         │
│  ┌─── Compute Savings Plan (Most Flexible) ────────────────────────┐   │
│  │  • Applies to: EC2, Lambda, AND Fargate                          │   │
│  │  • Any instance family, size, OS, tenancy, Region                │   │
│  │  • Up to 66% savings                                            │   │
│  │  • Best for: Teams using mixed compute                           │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─── EC2 Instance Savings Plan (Stricter) ────────────────────────┐   │
│  │  • Applies to: Specific EC2 instance family in specific Region   │   │
│  │  • Example: "m5 family in us-east-1"                             │   │
│  │  • Up to 72% savings (highest discount!)                         │   │
│  │  • Best for: Predictable, stable EC2 workloads                   │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─── SageMaker Savings Plan ──────────────────────────────────────┐   │
│  │  • Applies to: SageMaker instances                               │   │
│  │  • Any instance family, size, Region                             │   │
│  │  • Up to 64% savings                                            │   │
│  │  • Best for: ML workloads                                        │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  Payment Options: All Upfront | Partial Upfront | No Upfront           │
│  Commitment: 1 year or 3 years                                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Savings Plans vs Reserved Instances

```
┌──────────────────────────────────────────────────────────────────────┐
│              SAVINGS PLANS vs RESERVED INSTANCES                      │
├───────────────────────────────┬──────────────────────────────────────┤
│      SAVINGS PLANS            │      RESERVED INSTANCES              │
├───────────────────────────────┼──────────────────────────────────────┤
│ Commit to $/hour spend        │ Commit to specific instance type     │
│ Flexible (cross-service)      │ Tied to specific configuration      │
│ Covers EC2 + Lambda + Fargate │ EC2 only (or specific service RI)   │
│ Easier to manage              │ More complex to right-size          │
│ Newer model (AWS recommended) │ Older model (still valid)           │
│ Up to 72% savings             │ Up to 72% savings                   │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 DEFAULT CHOICE: Savings Plans (more flexible)                    │
│  RIs still valid for specific, predictable workloads                 │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 5. AWS Compute Optimizer ⚡ GOOD TO KNOW

> ML-based recommendations to right-size your compute resources.

```
┌─────────────────────────────────────────────────────────────┐
│              AWS COMPUTE OPTIMIZER                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: ML-powered service that recommends optimal           │
│        compute resources based on actual usage              │
│                                                             │
│  Resources it optimizes:                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  • Amazon EC2 instances (right-size up or down)       │  │
│  │  • Auto Scaling Groups (optimal configuration)        │  │
│  │  • Amazon EBS volumes (type and size)                 │  │
│  │  • AWS Lambda (memory allocation)                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Provides:                                                  │
│  • Current usage vs optimal (over/under-provisioned)        │
│  • Graphical history + projected utilization                │
│  • Specific resize recommendations                          │
│  • Works across Organizations (management or member level)  │
│                                                             │
│  🧠 "Right-size EC2/EBS/Lambda" → Compute Optimizer         │
│  🧠 "Reduce compute spend with recommendations" → Compute Optimizer│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Cost Management — Complete Decision Map

```
┌──────────────────────────────────────────────────────────────────┐
│        COST — WHICH SERVICE FOR WHICH SCENARIO?                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Visualize past spending / forecast future costs"               │
│       → Cost Explorer                                           │
│                                                                  │
│  "Alert before overspending"                                     │
│       → AWS Budgets                                             │
│                                                                  │
│  "Most detailed/comprehensive cost breakdown (CSV)"              │
│       → Cost and Usage Reports (CUR)                            │
│                                                                  │
│  "Query billing data with SQL"                                   │
│       → CUR + Athena                                            │
│                                                                  │
│  "Save money on compute (EC2 + Lambda + Fargate)"                │
│       → Savings Plans (Compute)                                 │
│                                                                  │
│  "Save on specific EC2 instance family (highest discount)"       │
│       → EC2 Instance Savings Plan (or RI)                       │
│                                                                  │
│  "Right-size over/under-provisioned resources"                   │
│       → Compute Optimizer                                       │
│                                                                  │
│  "Share RI discounts across accounts"                            │
│       → AWS Organizations (Consolidated Billing)                │
│                                                                  │
│  "Tag-based cost tracking by project/team"                       │
│       → Cost Allocation Tags + Cost Explorer                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### Production Cost Strategy (12 YOE Level)

```
┌─────────────────────────────────────────────────────────────────────┐
│         PRODUCTION COST OPTIMIZATION PLAYBOOK                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1️⃣  TAG EVERYTHING                                                 │
│     → Cost allocation tags for project, team, environment           │
│     → Filter in Cost Explorer and Budgets                           │
│                                                                     │
│  2️⃣  RIGHT-SIZE                                                     │
│     → Compute Optimizer recommendations                             │
│     → Review monthly, resize underutilized instances                │
│                                                                     │
│  3️⃣  COMMIT (for predictable workloads)                             │
│     → Savings Plans (flexible) or RIs (specific)                   │
│     → Cover baseline with commitments                               │
│                                                                     │
│  4️⃣  SPOT (for burst/fault-tolerant)                                │
│     → Spot Instances for batch, CI/CD, stateless workers           │
│     → Up to 90% savings                                            │
│                                                                     │
│  5️⃣  LIFECYCLE (for storage)                                        │
│     → S3 Lifecycle policies (Standard → IA → Glacier)              │
│     → EBS snapshot cleanup                                          │
│                                                                     │
│  6️⃣  SERVERLESS (when possible)                                     │
│     → Lambda, Fargate (pay only when running)                       │
│     → No idle cost                                                  │
│                                                                     │
│  7️⃣  MONITOR & ALERT                                                │
│     → Budgets (proactive alerts)                                    │
│     → Cost Explorer (monthly review)                                │
│     → CUR + Athena (deep analysis)                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Cost Topic | Verdict | Why |
|-----------|---------|-----|
| Savings Plans (3 types, vs RIs) | ✅ Must know | "Save on compute" — know which plan for which scenario |
| Cost Explorer (visualize, forecast) | ⚡ Know concept | "Where is spend?" = Cost Explorer |
| Budgets (alert before overspending) | ⚡ Know concept | Proactive alerts |
| CUR (most comprehensive data) | ⚡ Know concept | "Detailed CSV to S3" = CUR |
| Compute Optimizer (right-sizing) | ⚡ Know concept | "Right-size EC2/Lambda" |
| Tags for cost allocation | ✅ Must know | Foundation of cost tracking |
| Production cost strategy | ✅ Must internalize | Expected to design cost-optimized architectures |
| Savings Plans vs Reserved Instances | ✅ Must know | Savings Plans are now preferred (more flexible) |


---

## 6. AWS Trusted Advisor ✅ MUST HAVE

> Best-practice auditing — scans 5 areas. Know the 5 pillars and the automation pattern.

```
┌─────────────────────────────────────────────────────────────┐
│                  AWS TRUSTED ADVISOR                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Fully managed best-practice auditing tool            │
│        Scans your account and recommends improvements       │
│                                                             │
│  5 Categories It Checks:                                    │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  1. 💰 COST OPTIMIZATION                              │  │
│  │     "Spending money on unused resources?"             │  │
│  │                                                       │  │
│  │  2. 🚀 PERFORMANCE                                    │  │
│  │     "Services configured properly?"                   │  │
│  │                                                       │  │
│  │  3. 🔐 SECURITY                                       │  │
│  │     "Architecture full of vulnerabilities?"           │  │
│  │                                                       │  │
│  │  4. 🛡️  FAULT TOLERANCE                               │  │
│  │     "Protected when something fails?"                 │  │
│  │                                                       │  │
│  │  5. 📊 SERVICE LIMITS                                  │  │
│  │     "Room to scale?"                                  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Important:                                                 │
│  • Basic checks: FREE (all accounts)                        │
│  • Full checks: Business or Enterprise Support plan         │
│  • Does NOT fix problems — only identifies them             │
│  • Use SNS to alert users about findings                    │
│  • Use EventBridge + Lambda to AUTOMATE remediation         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Trusted Advisor Automation Pattern

```
┌──────────────────────────────────────────────────────────────┐
│       TRUSTED ADVISOR AUTOMATION                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Trusted Advisor ──▶ EventBridge ──▶ Lambda (auto-fix)      │
│                         │                                    │
│                         └──▶ SNS (alert humans)              │
│                                                              │
│  Example:                                                    │
│  TA finds open Security Group (0.0.0.0/0 on port 22)        │
│  → EventBridge triggers Lambda                               │
│  → Lambda removes the offending rule                         │
│  → SNS notifies the security team                           │
│                                                              │
│  🧠 "Best practice checks + automation" → Trusted Advisor   │
│  🧠 Doesn't fix things itself → use EventBridge + Lambda    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 7. AWS Control Tower ⚡ GOOD TO KNOW

> Fastest way to set up a secure, governed multi-account environment. Know the key terms.

```
┌─────────────────────────────────────────────────────────────┐
│                 AWS CONTROL TOWER                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Automated multi-account governance                   │
│        Sets up a "landing zone" with best practices         │
│                                                             │
│  Think: "Organizations + Config + SCPs + CloudFormation     │
│          all pre-configured and automated"                  │
│                                                             │
│  Key Terms:                                                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  LANDING ZONE                                         │  │
│  │  • Pre-built multi-account environment                │  │
│  │  • Security + compliance out of the box               │  │
│  │                                                       │  │
│  │  GUARDRAILS (2 types)                                 │  │
│  │  • Preventive: SCPs — block non-compliant actions     │  │
│  │  • Detective: Config rules — alert on violations      │  │
│  │                                                       │  │
│  │  ACCOUNT FACTORY                                      │  │
│  │  • Template for creating new accounts                 │  │
│  │  • Pre-approved configurations                        │  │
│  │  • Standardized setup                                 │  │
│  │                                                       │  │
│  │  SHARED ACCOUNTS (3)                                  │  │
│  │  • Management account                                 │  │
│  │  • Log archive account                                │  │
│  │  • Audit account                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  🧠 "Automated multi-account governance" → Control Tower    │
│  🧠 "Landing zone" → Control Tower                          │
│  🧠 "Guardrails (preventive + detective)" → Control Tower   │
│  🧠 "Quickest multi-account setup" → Control Tower          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. AWS Well-Architected Tool ⚡ GOOD TO KNOW

> Review your workloads against the 6 pillars. Mostly documentation and assessment.

```
┌─────────────────────────────────────────────────────────────┐
│            AWS WELL-ARCHITECTED TOOL                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Self-service tool to evaluate your architecture      │
│        against the 6 Well-Architected Framework pillars     │
│                                                             │
│  What it does:                                              │
│  • Consistent process for measuring architectures           │
│  • Documents workload decisions                             │
│  • Identifies high-risk issues (HRIs)                       │
│  • Provides improvement plan                                │
│  • Compares against years of AWS best practices             │
│                                                             │
│  6 Pillars reviewed:                                        │
│  Operational Excellence | Security | Reliability |           │
│  Performance Efficiency | Cost Optimization | Sustainability│
│                                                             │
│  🧠 "Review/audit architecture against best practices"      │
│     → Well-Architected Tool                                 │
│  🧠 "Document workload decisions" → Well-Architected Tool   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

---

# 🚚 CATEGORY: MIGRATION & HYBRID

---

## 1. AWS Storage Gateway ✅ MUST HAVE

> Hybrid cloud storage. Know it's for ONGOING hybrid (not one-time migration).

### What is Storage Gateway?

A **hybrid cloud storage service** — bridges on-premises storage with AWS cloud storage. Can be one-time or long-term pairing.

```
┌─────────────────────────────────────────────────────────────┐
│                 STORAGE GATEWAY                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─── On-Premises ────────┐      ┌─── AWS Cloud ────────┐ │
│  │                         │      │                       │ │
│  │  Existing apps/servers  │      │  S3 / EBS / Glacier  │ │
│  │         │               │      │       ▲              │ │
│  │         ▼               │      │       │              │ │
│  │  ┌─────────────────┐   │      │       │              │ │
│  │  │ Storage Gateway │───┼──────┼───────┘              │ │
│  │  │ (VM or hardware)│   │      │                       │ │
│  │  └─────────────────┘   │      │                       │ │
│  └─────────────────────────┘      └───────────────────────┘ │
│                                                             │
│  3 Types:                                                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  FILE GATEWAY                                         │  │
│  │  • NFS/SMB interface → stores in S3                   │  │
│  │  • Files as objects in S3                             │  │
│  │  • Use: File shares backed by S3                      │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  VOLUME GATEWAY                                       │  │
│  │  • iSCSI block storage → backed by S3/EBS            │  │
│  │  • Cached mode: frequent data local, rest in S3       │  │
│  │  • Stored mode: all data local, async backup to S3    │  │
│  │  • Use: Block storage with cloud backup               │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  TAPE GATEWAY                                         │  │
│  │  • Virtual tape library backed by S3/Glacier          │  │
│  │  • Use: Replace physical tapes for backup             │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  🧠 "On-premises + cloud hybrid storage" → Storage Gateway  │
│  🧠 "Continuous sync / long-term hybrid" → Storage Gateway  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. AWS DataSync ✅ MUST HAVE

> One-time migration of on-prem storage to AWS. Know it's AGENT-BASED.

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS DATASYNC                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  What: Agent-based data migration service                   │
│        Move data from on-prem NFS/SMB → AWS                │
│                                                             │
│  ┌─── On-Premises ────────┐      ┌─── AWS ──────────────┐ │
│  │                         │      │                       │ │
│  │  NFS/SMB Storage        │      │  Destinations:        │ │
│  │         │               │      │  • Amazon S3          │ │
│  │         ▼               │      │  • Amazon EFS         │ │
│  │  ┌─────────────────┐   │      │  • Amazon FSx         │ │
│  │  │ DataSync Agent  │───┼─────▶│                       │ │
│  │  │ (installed on   │   │      │                       │ │
│  │  │  your server)   │   │      │                       │ │
│  │  └─────────────────┘   │      │                       │ │
│  └─────────────────────────┘      └───────────────────────┘ │
│                                                             │
│  Key facts:                                                 │
│  • Agent-based (install on your on-prem server)             │
│  • One-time or scheduled migration                          │
│  • Supports S3, EFS, FSx as destinations                    │
│  • Automatic encryption in transit                          │
│  • Data validation (integrity checks)                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### DataSync vs Storage Gateway — Decision

```
┌──────────────────────────────────────────────────────────────────────┐
│              DATASYNC vs STORAGE GATEWAY                              │
├───────────────────────────────┬──────────────────────────────────────┤
│        DATASYNC               │       STORAGE GATEWAY                │
├───────────────────────────────┼──────────────────────────────────────┤
│ One-time migration            │ Ongoing hybrid (continuous)          │
│ Move data TO AWS              │ Bridge on-prem ↔ cloud              │
│ Agent-based                   │ VM or hardware appliance            │
│ "I want to migrate my data"  │ "I want to keep using on-prem +     │
│                               │  extend to cloud long-term"         │
├───────────────────────────────┴──────────────────────────────────────┤
│                                                                      │
│  🧠 "One-time data migration to AWS" → DataSync                     │
│  🧠 "Hybrid storage (on-prem + cloud, long-term)" → Storage Gateway │
│  🧠 "On-premises mentioned" → think Storage Gateway                 │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Migration & Governance — Updated Decision Map

```
┌──────────────────────────────────────────────────────────────────┐
│     GOVERNANCE & MIGRATION — WHICH SERVICE?                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Best-practice audit (cost, perf, security, fault tol, limits)" │
│       → Trusted Advisor                                         │
│                                                                  │
│  "Automate response to Trusted Advisor finding"                  │
│       → EventBridge + Lambda (TA doesn't fix things itself)     │
│                                                                  │
│  "Set up secure multi-account environment quickly"               │
│       → Control Tower (landing zone + guardrails)               │
│                                                                  │
│  "Preventive guardrail (block action)"                           │
│       → SCP (via Control Tower or Organizations)                │
│                                                                  │
│  "Detective guardrail (alert on violation)"                      │
│       → AWS Config rule (via Control Tower)                     │
│                                                                  │
│  "Review architecture against Well-Architected"                  │
│       → Well-Architected Tool                                   │
│                                                                  │
│  "One-time data migration (NFS/SMB → S3/EFS)"                   │
│       → DataSync                                                │
│                                                                  │
│  "Hybrid storage (on-prem + cloud, ongoing)"                     │
│       → Storage Gateway                                         │
│                                                                  │
│  "Replace physical tape backups with cloud"                      │
│       → Storage Gateway (Tape Gateway)                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### My Recommendation for Your Level

| Topic | Verdict | Why |
|-------|---------|-----|
| Trusted Advisor (5 categories) | ✅ Must know | "Best practice audit" = TA. Know the 5 areas. |
| TA automation (EventBridge + Lambda) | ✅ Must know | Automation pattern is always tested |
| Control Tower (landing zone, guardrails) | ⚡ Know concepts | "Automated multi-account setup" |
| Control Tower guardrail types | ⚡ Know | Preventive = SCP, Detective = Config |
| Well-Architected Tool | ⚡ Know it exists | "Review architecture" |
| Storage Gateway (3 types) | ✅ Must know | "On-prem hybrid storage" = Storage Gateway |
| DataSync | ✅ Must know | "One-time migration" = DataSync |
| DataSync vs Storage Gateway | ✅ Must know | Migration vs ongoing hybrid — always asked |
| Full support plan needed for TA | ⚡ Know | Business/Enterprise for full checks |
