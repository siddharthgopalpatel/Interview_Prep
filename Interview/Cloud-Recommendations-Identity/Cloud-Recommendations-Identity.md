# Cloud Recommendations Identity — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 50

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

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

