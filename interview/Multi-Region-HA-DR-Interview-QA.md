# Interview Q&A Bank: Multi-Region HA & Disaster Recovery

## Project: Production-Grade Cross-Region Failover — Route53, Aurora Global, S3 CRR, Chaos Engineering

**Technologies:** AWS Route53, Aurora Global Database, S3 Cross-Region Replication, Terraform, CloudFront, Auto Scaling, AWS FIS, Lambda, CloudWatch

---

## Section 1: Project Story

### Q1: Walk me through your DR strategy in 2 minutes.

**Answer:**
Built active-passive multi-region DR for our e-commerce platform. Primary in us-east-1 serves all traffic. Route53 health checks detect failure in 30 seconds, automatically failover DNS to us-west-2. Aurora Global Database replicates with <1 second lag — RPO under 1 second. DR region has compute scaled to zero (warm pool) — scales up in 90 seconds on failover. S3 Cross-Region Replication handles static assets. Total measured RTO in our last quarterly drill: 3 minutes 10 seconds. All infrastructure deployed via same Terraform modules with different region parameter. Cost: ~$400/month for DR readiness (vs $760 for active-active). Validated quarterly through automated DR drills using AWS FIS.

---

### Q2: Why active-passive over active-active?

**Answer:**
- **Cost:** Active-active = 2x infrastructure ($760/month × 2). Active-passive = 1.3x ($400/month DR overhead).
- **Complexity:** Active-active requires solving write conflicts (multi-master DB, conflict resolution, session routing). Active-passive = simple failover.
- **Our requirement:** 5-minute RTO is acceptable for $2M/hour revenue. If we were $50M/hour → active-active justified.
- **Trade-off accepted:** ~3 minutes downtime during region failure vs $4,000+/month for instant failover.

**When I'd choose active-active:** Stock trading (zero-second tolerance), global app needing low latency everywhere (users in US + EU + Asia), or compliance requiring data in multiple regions simultaneously.

---

## Section 2: Technical Deep-Dive

### Q3: Explain RTO and RPO in simple terms with your actual numbers.

**Answer:**
- **RPO (Recovery Point Objective) = "How much data can we LOSE?"**
  Aurora replicates every ~1 second. If region dies → we lose max 1 second of data.
  Our RPO: <1 second.

- **RTO (Recovery Time Objective) = "How long are we DOWN?"**
  Route53 detects in 30s + DNS propagates in 60s + compute scales in 90s = ~3 minutes.
  Our RTO: ~3 minutes (measured in drills, not theoretical).

**Real timeline from our last drill:**
```
0:00  — Region fails
0:30  — Route53 health check fails (3 × 10s interval)
0:30  — DNS failover triggers
1:30  — DNS propagation complete (TTL=60s)
3:00  — DR ASG scaled to 3 instances + healthy
3:10  — First request served from DR ✅
```

---

### Q4: How does Route53 failover routing work technically?

**Answer:**
1. Route53 has TWO records for `app.example.com`: PRIMARY (us-east-1 ALB) and SECONDARY (us-west-2 ALB)
2. Health check pings primary ALB every 10 seconds (`GET /health`)
3. After 3 consecutive failures (30 seconds) → primary marked unhealthy
4. Route53 stops returning primary IP in DNS responses
5. All new DNS queries return secondary (DR) IP
6. Client DNS cache expires (TTL=60s) → clients get new IP → traffic goes to DR

**Key configs:** `failure_threshold=3`, `request_interval=10`, alias record TTL=60s, `evaluate_target_health=true`

**What Route53 does NOT do:** It doesn't migrate your data, promote your DB, or scale your compute. That's Lambda's job (triggered by CloudWatch alarm on health check failure).

---

### Q5: How does Aurora Global Database failover work?

**Answer:**
**Normal state:**
- Primary cluster (us-east-1): handles all WRITES
- Secondary cluster (us-west-2): read replica, receives async replication (<1s lag)
- Writer endpoint: `cluster-abc.us-east-1.rds.amazonaws.com` → always points to primary

**Failover (promotion):**
```bash
aws rds remove-from-global-cluster \
  --global-cluster-identifier app-global-db \
  --db-cluster-identifier arn:aws:rds:us-west-2:...:cluster:app-db-dr
```

This DETACHES the DR cluster from global cluster → it becomes an independent writer. Takes <60 seconds.

**After promotion:**
- DR cluster is now an independent primary (accepts writes)
- Application in DR region connects to DR cluster's writer endpoint
- Global cluster relationship is BROKEN (must recreate for failback)

**Failback challenge:** After primary region recovers, must re-establish global replication (recreate global cluster, resync data). Takes 30-45 minutes. This is the hardest part of DR.

---

### Q6: Why warm pool (ASG min=0) instead of keeping instances running in DR?

**Answer:**
| | Instances running in DR | Warm pool (min=0) |
|---|---|---|
| Cost | ~$200/month (3 instances always on) | ~$50/month (stopped instances in pool) |
| RTO impact | 0 extra seconds (already running) | +90 seconds (launch from stopped state) |
| Trade-off | Pay $150/month for 90 seconds faster RTO | Save $150/month, accept 90s slower |

**Our choice:** Warm pool. 3 minutes RTO is acceptable. $150/month × 12 = $1,800/year saved for 90 seconds difference.

**Warm pool vs cold launch:** Warm pool instances are PRE-CREATED and STOPPED. Starting a stopped instance = 30-60s. Launching fresh from AMI = 2-3 minutes. Warm pool is the middle ground.

---

### Q7: How does the automated failover Lambda work?

**Answer:**
**Trigger:** CloudWatch Alarm (Route53 health check status = 0 = unhealthy)
→ EventBridge rule → Lambda

**Lambda does:**
1. Promote Aurora DR replica to standalone primary (`remove-from-global-cluster`)
2. Update ASG desired capacity: 0 → 3 (triggers instance launch from warm pool)
3. Wait for ALB health check to pass (retry loop)
4. Send notifications: PagerDuty (P1) + Slack ("🔴 DR ACTIVATED")
5. Log to DynamoDB (audit trail: when, what triggered, actions taken)

**Why Lambda (not manual):** Region failure at 3 AM. Nobody awake. Lambda promotes DB + scales compute in 2 minutes. Human response = 30+ minutes (wake up, VPN in, assess, act).

---

### Q8: How does S3 Cross-Region Replication fit into the DR strategy?

**Answer:**
S3 CRR replicates objects from primary bucket to DR bucket in real-time.

**What gets replicated:**
- Static assets (images, CSS, JS) — DR CloudFront needs these
- Terraform state backup — can re-provision from DR
- Application config files — needed if DR instances boot

**What S3 CRR does NOT cover:**
- Database data (Aurora Global handles that)
- In-flight requests (those are lost during failover)
- Application secrets (Secrets Manager has its own multi-region replication)

**Key detail:** CRR is eventually consistent (usually seconds, no SLA). Not suitable as the sole data replication mechanism — that's why Aurora Global is primary for the database layer.

---

## Section 3: Troubleshooting

### Q9: DR drill shows RTO of 8 minutes instead of target 5 minutes. Where do you investigate?

**Answer:**
Break down the timeline — which phase took longer?

| Phase | Expected | Actual? | If Slow → Fix |
|---|---|---|---|
| Health check detection | 30s | Check CloudWatch | Reduce `request_interval` to 5s? |
| DNS propagation | 60s | Check client TTL | Reduce TTL from 60s to 30s |
| DB promotion | 30-60s | Check Aurora logs | Usually consistent — check for replica lag |
| Compute scale-up | 90s | Check ASG activity | Warm pool → faster. Check AMI boot time. |
| ALB health check pass | 30s | Check target health | App startup slow? Reduce health check interval |

**Most common cause:** Compute launch time. If AMI has heavy startup (JVM warmup, cache loading) → add user-data warmup script or use pre-baked AMI with warmed caches.

---

### Q10: Aurora replication lag suddenly shows 5 seconds (normally <1s). What's happening?

**Answer:**
1. **Check primary write volume:** Sudden write spike? (bulk import, migration running) → replication can't keep up.
2. **Check network:** Cross-region network latency increased? (rare but AWS has outages)
3. **Check DR instance size:** If DR replica is smaller instance class than primary → can't apply changes as fast.
4. **Impact on RPO:** 5s lag means if we fail over NOW, we lose 5 seconds of data (5 transactions maybe).

**Actions:**
- If temporary (bulk job): wait for it to finish, lag will recover
- If persistent: upgrade DR instance class to match primary
- Alert threshold: alarm at >5s lag → investigate immediately (our RPO target is <5s)

---

### Q11: After DR failover, application connects to DR database but gets "read-only" errors. Why?

**Answer:**
**Cause:** Application is still using the GLOBAL writer endpoint (which now points to the old dead primary) instead of the DR cluster's local endpoint.

**Or:** Aurora promotion hasn't completed yet — cluster is still in reader mode.

**Fix:**
1. Check: Did `remove-from-global-cluster` complete? (`aws rds describe-global-clusters`)
2. Application should use the DR cluster's endpoint (not global). Update application config / environment variable.
3. **Better architecture:** Use Route53 CNAME for DB endpoint (`db.internal.example.com`) → update this CNAME to point to DR cluster endpoint during failover. App doesn't change — DNS changes.

**Prevention:** Never hardcode DB endpoints. Always use a DNS abstraction that can be flipped during failover.

---

## Section 4: System Design

### Q12: Design DR for an application that CANNOT tolerate ANY data loss (RPO=0).

**Answer:**
**RPO=0 means:** Synchronous replication (every write confirmed in BOTH regions before returning success).

**Options:**
1. **DynamoDB Global Tables:** Multi-region active-active. Writes replicated synchronously. RPO=0. But: DynamoDB only (not relational).
2. **Aurora Multi-Master (deprecated) → Aurora Global with write forwarding:** Near-zero RPO but adds latency to every write (cross-region round trip).
3. **Application-level dual-write:** App writes to both regions. Complex — conflict resolution needed.

**Trade-off:** RPO=0 = every write has cross-region latency penalty (50-100ms added). For most apps unacceptable for every transaction.

**Practical approach:** RPO <1 second (Aurora Global async) + accept losing 1 transaction worst case. True RPO=0 is usually only for financial transactions (use DynamoDB Global Tables for those specific tables).

---

### Q13: How would you test DR without impacting production users?

**Answer:**
Our quarterly drill procedure:
1. **Announce:** Stakeholders know it's a drill (not a real failure)
2. **Simulate failure:** Override Route53 health check to return unhealthy (doesn't actually break primary)
3. **Verify failover chain:** DNS flips → DB promotes → compute scales → first request from DR
4. **Run smoke tests** against DR endpoint (not production traffic — just test calls)
5. **Measure RTO/RPO** (timestamps at each phase)
6. **Failback:** Restore health check → traffic returns to primary
7. **Report:** "RTO: 3 min 10 sec. RPO: 0.8 sec. Issues found: [list]"

**Key:** We DON'T send real user traffic to DR during drills. We simulate the infrastructure failover and verify with synthetic tests. Real user traffic only flows to DR during actual failures.

**Alternative (more aggressive):** Chaos engineering with AWS FIS — actually terminate instances/block network in primary → verify real failover. Do this in staging first.

---

## Section 5: Comparison & Decisions

### Q14: Pilot Light vs Warm Standby vs Active-Active — how did you choose?

**Answer:**
| Strategy | RTO | RPO | DR Cost/month | We Chose? |
|---|---|---|---|---|
| Backup & Restore | 2-4 hours | 1 hour | ~$50 | ❌ Too slow |
| **Pilot Light** | ~5 min | <1 sec | ~$150 | ✅ Best balance |
| Warm Standby | 1-2 min | <1 sec | ~$400 | ❌ Not worth $250/month for 3 min faster |
| Active-Active | ~0 | 0 | ~$760 | ❌ Too expensive + complex for our needs |

**Decision logic:** Business loses $2M/hour. 5-minute RTO = $167K max exposure per incident. Incidents happen maybe once per year. Paying $5,400/year (Pilot Light) to protect against $167K potential loss = obvious ROI. Active-Active at $9,120/year saves 5 minutes more — not justified.

---

### Q15: Route53 failover vs Global Accelerator — when to use which?

**Answer:**
| | Route53 Failover | Global Accelerator |
|---|---|---|
| Failover speed | DNS TTL dependent (30-60s) | Instant (anycast IP, no DNS change) |
| Client impact | Clients with cached DNS see old IP for TTL duration | Zero — same IP, routing changes at AWS edge |
| Cost | ~$5/month (health checks) | ~$50/month + data transfer |
| Setup | Simple (health check + failover records) | More complex (endpoint groups, weights) |
| Best for | Cost-sensitive DR with acceptable 60s DNS delay | Mission-critical apps needing instant failover |

**Our choice:** Route53 (cheaper, 60s DNS delay acceptable for our 5-min RTO target). If we needed sub-30-second failover → Global Accelerator.

---

### Q16: Same Terraform modules for both regions — how does that work?

**Answer:**
```hcl
provider "aws" {
  alias  = "primary"
  region = "us-east-1"
}

provider "aws" {
  alias  = "dr"
  region = "us-west-2"
}

# Same VPC module, different region + CIDR
module "vpc_primary" {
  source    = "../../modules/vpc"
  providers = { aws = aws.primary }
  cidr_block = "10.0.0.0/16"
}

module "vpc_dr" {
  source    = "../../modules/vpc"
  providers = { aws = aws.dr }
  cidr_block = "10.1.0.0/16"    # Different CIDR
}
```

**Why same module:** If primary and DR have different code → config drift. Module X deployed in both regions = IDENTICAL infrastructure guaranteed. Only differences: CIDR (for peering), capacity (DR starts at zero).

**Weekly drift detection:** `terraform plan` runs against DR region. If changes detected → alert. Ensures DR is always ready.

---

## Section 6: Behavioral

### Q17: Tell me about your most recent DR drill — what went well and what didn't?

**Answer (STAR):**
- **Situation:** Quarterly DR drill, Saturday 6 AM. Full failover simulation.
- **What went well:** Detection in 28 seconds (target <30). DNS flip in 31 seconds. Aurora promoted in 45 seconds. Total RTO: 3 min 10 sec (target <5 min).
- **What didn't:** Two issues found:
  1. DR region missing 2 environment variables (config drift — someone added vars to primary manually, not via Terraform)
  2. One Lambda function had hardcoded `us-east-1` endpoint (didn't use region-agnostic SDK call)
- **Action:** Added weekly `terraform plan` drift detection for DR. Linted all Lambda code for hardcoded regions. Both fixed same day.
- **Result:** Next drill (following quarter): zero issues. Config drift detection caught 3 more cases before they became problems.

---

### Q18: How did you justify the $400/month DR cost to management?

**Answer (STAR):**
- **Situation:** Management asked: "Why pay $4,800/year for something we might never use?"
- **Task:** Justify DR investment in business terms.
- **Action:** Presented: "Our platform generates $2M/hour. A 4-hour outage (without DR) = $8M lost revenue + customer trust + SLA penalties. DR costs $4,800/year and reduces outage from 4 hours to 5 minutes. That's $8M protection for $4,800 investment — 1,666x ROI."
- **Result:** Approved immediately. Also got budget for quarterly drills.

**Key lesson:** Never justify DR in technical terms ("Aurora Global replicates..."). Always in business terms ("$8M at risk, $4,800 to protect").

---

## Section 7: Future & Improvements

### Q19: What would make you upgrade from Pilot Light to Active-Active?

**Answer:**
1. **Revenue exceeds $10M/hour** — 5 minutes downtime = $833K loss. At that point, $9K/month for zero-downtime failover is justified.
2. **Global user base** — users in EU and Asia need low latency (can't route all through us-east-1). Active-active with latency-based routing.
3. **Compliance requiring zero data loss** — regulated industry mandates RPO=0 for all transactions.
4. **SLA commitment of 99.999%** — allows only 5 min downtime/year. Can't risk even one 5-min failover.

Until then: Pilot Light at $150/month gives us <5-min RTO, <1-sec RPO, validated quarterly. Good enough.

---

### Q20: How would you add chaos engineering to validate DR continuously (not just quarterly)?

**Answer:**
**AWS FIS (Fault Injection Simulator) experiments running monthly:**

| Experiment | What It Tests | Frequency |
|---|---|---|
| Kill random EC2 instance | ASG self-healing (single instance failure) | Weekly |
| Block AZ network | Multi-AZ failover (ALB routes around it) | Monthly |
| Increase API latency 5s | Circuit breakers activate, graceful degradation | Monthly |
| Simulate region health check failure | Full DR chain (Route53 → Lambda → DB promote → scale) | Quarterly |

**Safety nets:**
- Stop condition: If error rate > 10% → abort experiment immediately
- Run during low-traffic window (2 AM Tuesday)
- Staging first → then production (with smaller blast radius)
- All stakeholders notified before experiment

**Goal:** Move from "we THINK DR works" (quarterly drills) to "we KNOW DR works" (continuous validation).

---

*End of Q&A Bank — 20 questions covering all 7 dimensions*
