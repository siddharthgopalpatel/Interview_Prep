# Databases — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 35

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

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

