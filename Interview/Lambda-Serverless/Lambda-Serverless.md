# Lambda Serverless — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 33

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

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

