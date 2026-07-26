# Interview Q&A Bank: Cloud Cost Optimization (FinOps)

## Project: Automated Cost Governance — 35% Reduction ($180K/year Saved)

**Technologies:** AWS Cost Explorer, Lambda, EventBridge, CloudWatch, Terraform, Kubecost, Grafana, Infracost, Compute Optimizer, Karpenter, Savings Plans

---

## Section 1: Project Story

### Q1: Walk me through the cost optimization project in 2 minutes.

**Answer:**
AWS bill was $45,000/month growing 20% monthly — no visibility, no accountability, no automation. Built an automated FinOps platform with three pillars: Inform (Kubecost per-namespace, Grafana dashboards per team), Optimize (Lambda auto-stop non-prod, Compute Optimizer rightsizing, Karpenter consolidation, VPC endpoints), and Govern (SCPs enforce tagging, Infracost shows cost in every Terraform PR, budget alerts at 80/100/120%). Result: reduced bill from $45K to $29K/month — 35% reduction, $180K+ annual savings. Applied across Projects 1, 2, and 3: EC2 rightsizing (Project 2), Karpenter spot + consolidation (Project 3), ephemeral Jenkins agents (Project 1).

---

### Q2: What was the biggest single cost saving and how did you achieve it?

**Answer:**
**Non-prod auto-stop = $102K/year** (single biggest lever).

10 dev EC2 instances + 5 staging instances running 24/7. Used only 10 hours/day on weekdays. Nights and weekends = pure waste.

**Solution:** EventBridge scheduled rule → Lambda stops all instances tagged `Environment=dev|staging` + `AutoStop=true` at 8 PM weekdays. Another rule starts them at 8 AM. Weekends fully off.

**Math:** 118 idle hours/week out of 168 total = 65% savings on non-prod compute. Zero developer impact — they don't even notice.

---

## Section 2: Technical Deep-Dive

### Q3: How does the auto-stop/start Lambda work technically?

**Answer:**
```
EventBridge Rule: cron(0 20 ? * MON-FRI *)  → Lambda: stop_non_prod
EventBridge Rule: cron(0 8  ? * MON-FRI *)  → Lambda: start_non_prod
```

Lambda logic:
1. Query EC2: find all instances with tag `Environment=dev|staging` AND `AutoStop=true`
2. Stop them (`ec2.stop_instances`)
3. Also: stop RDS dev instances, scale ECS services to 0, scale EKS node groups to 0
4. Log to DynamoDB (what was stopped, when, cost saved)
5. Notify Slack: "🌙 Stopped 15 non-prod instances. Saving $28/hour."

**Safety:** If someone needs to work late → they click "Start" in Slack bot or override tag `AutoStop=false`. Next night it gets stopped again.

---

### Q4: How does Infracost work in the Terraform PR workflow?

**Answer:**
```
Developer creates PR with Terraform change
    → CI pipeline runs infracost
    → Infracost calculates cost diff
    → Posts comment on PR:

    💰 Monthly cost will increase by $150
    
    + aws_nat_gateway.new    $32/month
    + aws_rds_cluster.new    $118/month
    
    Total monthly: $4,520 → $4,670
```

**Why this matters:** Developer sees cost BEFORE merging. Reviewer can ask: "Do we really need a NAT gateway here? Can we use VPC endpoint instead?" Cost becomes part of code review — not a surprise bill 30 days later.

---

### Q5: How does Kubecost provide per-namespace cost visibility?

**Answer:**
Kubecost runs in the K8s cluster, reads:
- Node costs (from AWS pricing API — instance type × hours running)
- Pod resource requests (how much CPU/memory each pod claims)
- Allocates node cost proportionally to pods based on their resource requests

**Result:**
```
Namespace: app-prod     → $1,200/month (5% waste)
Namespace: monitoring   → $400/month (10% waste)
Namespace: app-staging  → $600/month (65% waste!) ← flag
Namespace: ci-runners   → $800/month (20% waste) ← use spot
```

**"Waste" = resources requested but not used.** Pod requests 2 CPU but uses 0.3 = 85% wasted.

Teams see THEIR cost → they reduce it. Visibility alone drops cost 15% without any automation.

---

### Q6: How did you implement mandatory tagging via SCPs?

**Answer:**
```json
{
  "Effect": "Deny",
  "Action": ["ec2:RunInstances", "rds:CreateDBInstance", "s3:CreateBucket"],
  "Condition": {
    "Null": {
      "aws:RequestTag/Team": "true",
      "aws:RequestTag/Environment": "true"
    }
  }
}
```

**What this does:** If you try to create an EC2/RDS/S3 WITHOUT `Team` and `Environment` tags → Access Denied. Even admin can't bypass (SCP > IAM).

**Why:** Without tags → can't attribute cost to a team. With mandatory tags → Cost Explorer shows spend per team, per environment instantly. Monthly chargeback becomes automatic.

---

### Q7: How does Karpenter help with cost optimization specifically?

**Answer:**
Three ways:

1. **Right-sized nodes:** Karpenter picks the cheapest instance type that fits pending pods (not fixed node group with oversized instances)
2. **Spot instances:** `capacity-type: ["spot", "on-demand"]` — Karpenter uses spot (70% cheaper) when available, falls back to on-demand
3. **Consolidation:** `consolidationPolicy: WhenUnderutilized` — if 5 nodes are at 30% utilization, Karpenter repacks pods onto 2 nodes and terminates 3

**Result:** 40% fewer nodes for the same workload. Automatic. No manual intervention.

---

### Q8: Explain the full cost governance pipeline — from PR to production.

**Answer:**
```
1. PREVENT (before deploy):
   - Infracost on PR → shows cost impact before merge
   - SCP → blocks untagged resources
   - Terraform modules → enforce GP3 (not GP2), Graviton, smallest viable instance

2. DETECT (after deploy):
   - Compute Optimizer → weekly rightsizing report (Slack)
   - Kubecost → per-namespace waste percentage
   - AWS Budgets → alert at 80%, 100%, 120% threshold
   - Lambda → daily scan for idle resources (unused EIPs, unattached EBS)

3. ACT (remediate):
   - Auto-stop non-prod (Lambda + EventBridge)
   - Karpenter consolidation (auto, continuous)
   - Monthly review meeting (team leads see their spend)
   - Idle resource cleanup (notify owner → delete after 14 days)
```

---

## Section 3: Troubleshooting

### Q9: AWS bill spiked $5000 unexpectedly this month. How do you investigate?

**Answer:**
1. **Cost Explorer → Group by Service:** Which service increased? (EC2? NAT? RDS? Data Transfer?)
2. **Cost Explorer → Group by Tag (Team):** Which team's spend jumped?
3. **Common causes:**
   - NAT Gateway data transfer: Private subnet pulling from S3 through NAT ($0.045/GB). Fix: S3 VPC Endpoint.
   - ASG max hit: Scaling policy too aggressive → scaled to 20 instances during spike, didn't scale down (cooldown too long).
   - Untagged resources: Someone launched outside Terraform. Can't attribute → investigate CloudTrail.
   - RDS snapshots accumulating: Manual snapshots with no lifecycle.
4. **Fix:** Address root cause + set tighter budget alerts to catch earlier next time.

---

### Q10: Team says "our Kubecost report shows 65% waste but we can't reduce pod requests without risking OOM." How do you help?

**Answer:**
1. **VPA (Vertical Pod Autoscaler) in recommend mode:** Shows actual P95 usage over 7 days. Not guessing — data-driven.
2. **Common pattern:** Dev set `memory: 4Gi` once during a spike. Actual usage: 800Mi steady, 1.2Gi peak.
3. **Safe approach:** Set requests to P95 usage + 20% buffer. Set limits to P99 + 50% buffer.
   - Request: 1.2Gi × 1.2 = ~1.5Gi (scheduler reserves this)
   - Limit: 1.5Gi × 1.5 = ~2.2Gi (OOM kill only above this)
4. **Start with non-prod:** Right-size staging first. Monitor for 1 week. No OOMs? → Apply to prod.
5. **Key insight:** "Waste" doesn't mean "reduce to zero." It means "reduce to actual usage + safety margin."

---

### Q11: Savings Plans were purchased but bill didn't decrease as expected. Why?

**Answer:**
Common causes:
1. **Wrong commitment type:** Purchased EC2 Savings Plan (locked to instance family) but workload moved to Fargate (not covered). Should've bought Compute Savings Plan (covers EC2 + Fargate + Lambda).
2. **Commitment too high:** Bought $10/hour commitment but baseline usage is only $7/hour. Over-committed = paying for unused commitment.
3. **Region mismatch:** Savings Plan covers us-east-1 but most workload moved to us-west-2.
4. **Instance family change:** Committed to m5 family but team migrated to Graviton (m6g). EC2 SP doesn't cover cross-family.

**Prevention:** Start with Compute Savings Plans (most flexible). Buy only 60-70% of steady baseline (leave headroom for changes). Review quarterly.

---

## Section 4: System Design

### Q12: Design a FinOps program for a company spending $500K/month across 50 teams.

**Answer:**
1. **Visibility (Week 1-2):** CUR → Athena → Grafana dashboards per team. Mandatory tagging via SCP. Everyone sees their spend.
2. **Quick wins (Week 3-4):** Auto-stop non-prod, S3 lifecycle policies, reserved instances for steady baseline.
3. **Accountability (Month 2):** Monthly cost review meetings per team. Budget alerts. Chargeback reports to team leads.
4. **Automation (Month 3+):** Idle resource cleanup, rightsizing alerts, Infracost in CI, Karpenter for K8s.
5. **Governance (Ongoing):** FinOps team reviews anomalies. Quarterly Savings Plan adjustments. New team onboarding includes cost awareness training.

**Target:** 30-40% reduction in 6 months = $150-200K/month saved = $1.8-2.4M/year.

---

### Q13: How do you balance cost optimization with reliability?

**Answer:**
**Golden rule:** Never sacrifice production reliability for cost.

| Environment | Cost Strategy | Reliability Compromise? |
|---|---|---|
| Production | Savings Plans + Graviton + right-sized (NOT spot, NOT auto-stop) | ❌ Never |
| Staging | Spot instances + auto-stop nights/weekends | ✅ OK — slower/interrupted acceptable |
| Dev | Spot + auto-stop + smallest instances | ✅ OK — interruptions expected |
| CI/CD runners | 100% spot (fault-tolerant by design) | ✅ OK — retry on interruption |
| Batch/cron jobs | Spot + Karpenter (schedule during cheap hours) | ✅ OK — retryable |

**Cost optimization targets:** Non-prod environments + waste elimination + right-sizing. NOT removing redundancy from production.

---

## Section 5: Comparison & Decisions

### Q14: Savings Plans vs Reserved Instances — when to use which?

**Answer:**
| | Savings Plans | Reserved Instances |
|---|---|---|
| Flexibility | Compute SP covers EC2 + Fargate + Lambda | Locked to specific instance type + AZ |
| Discount | 30-40% (1-year), 50-60% (3-year) | 30-40% (1-year), 60%+ (3-year) |
| Best for | Dynamic workloads (might change instance type) | Stable workloads (same instance for years) |
| Risk | Low (flexible across types/regions) | High (locked — waste if you change) |

**Our choice:** Compute Savings Plans (1-year) covering 60% of steady baseline. Most flexible. If we migrate from m5 to Graviton → SP still applies.

---

### Q15: Kubecost vs native AWS Cost Explorer — why both?

**Answer:**
| | AWS Cost Explorer | Kubecost |
|---|---|---|
| Granularity | Per-service (EC2, RDS, S3) | Per-pod, per-namespace, per-label |
| K8s visibility | Sees "3 EC2 nodes = $630" | Sees "app-prod uses $400 of that $630, monitoring uses $150, rest is waste" |
| Waste detection | ❌ Can't see pod-level waste | ✅ Shows "pod requests 4GB, uses 800MB = 80% waste" |
| Chargeback | Per-account, per-tag | Per-namespace (per-team in K8s) |

**We need both:** Cost Explorer for AWS-level spend (RDS, NAT, S3). Kubecost for inside-the-cluster spend (which team's pods are wasting resources).

---

### Q16: VPC Endpoint vs NAT Gateway for S3 access — the cost decision?

**Answer:**
| | NAT Gateway | S3 VPC Endpoint (Gateway type) |
|---|---|---|
| Cost | $0.045/GB data transfer + $0.045/hour | **FREE** (zero cost) |
| Path | Private subnet → NAT → IGW → S3 (public) | Private subnet → VPC Endpoint → S3 (private) |
| Security | Traffic exits VPC to internet | Traffic stays within AWS network (never touches internet) |

**Our savings:** App servers download 2TB/month from S3 through NAT = $90/month wasted. VPC Endpoint = $0. Plus better security.

**Why didn't we have it from day one?** Oversight. Added VPC endpoints for S3 and Secrets Manager after the cost review. Now it's a default in our Terraform modules — every new VPC gets them automatically.

---

## Section 6: Behavioral

### Q17: How did you make teams care about cloud costs?

**Answer (STAR):**
- **Situation:** Engineers said "cloud cost is someone else's problem." No visibility. No accountability. Bill shock every month.
- **Task:** Make cost a team-level responsibility without creating friction.
- **Action:** Three-step approach:
  1. **Visibility first:** Grafana dashboard showing each team's spend. No blame — just data. "Did you know your team spends $4,400/month?"
  2. **Quick win:** Auto-stop saved $8,500/month. Showed in all-hands: "We saved $100K/year by stopping dev at night. Nobody noticed."
  3. **Gamification:** Monthly "Cost Champion" award for team with biggest % reduction. Teams started competing.
- **Result:** Within 3 months, teams proactively asked: "Can we use spot here? Can we right-size this?" Culture shifted from "not my problem" to "let me check the dashboard."

---

### Q18: Tell me about a time a cost optimization caused a production issue.

**Answer (STAR):**
- **Situation:** Enabled Karpenter consolidation for ALL node pools including the one running stateful monitoring (Prometheus).
- **Task:** Karpenter moved Prometheus pod to a different node during consolidation. PVC was AZ-locked — pod stuck in Pending for 10 minutes. Monitoring gap.
- **Action:** Immediate: manually scheduled Prometheus back to original AZ node. Long-term: Added `karpenter.sh/do-not-disrupt: "true"` annotation to Prometheus pods. Created rule: "Stateful workloads with PVCs get disruption protection."
- **Result:** Never happened again. Created checklist: before enabling consolidation, identify all stateful workloads and add protection annotations.

**Learning:** Cost optimization must respect stateful workload constraints. Not everything can be freely moved.

---

## Section 7: Future & Improvements

### Q19: What's next for your FinOps program?

**Answer:**
| Priority | Improvement | Expected Savings |
|---|---|---|
| 1 | Graviton migration (m5 → m6g) for all EC2/EKS | 20% compute savings |
| 2 | S3 Intelligent-Tiering for all data buckets | ~$500/year (auto-tiers) |
| 3 | Spot for staging EKS (not just CI runners) | 60% on staging compute |
| 4 | Cost anomaly detection (ML-based — AWS native) | Early warning before bill shock |
| 5 | Commitment management tool (review SP utilization quarterly) | Prevent over-commitment waste |

---

### Q20: How would AI/ML change FinOps in the next 2-3 years?

**Answer:**
1. **Predictive cost forecasting:** ML predicts next month's bill based on trends → alerts BEFORE overspend (not after)
2. **Autonomous rightsizing:** AI watches usage patterns → auto-adjusts instance types (not just recommends)
3. **Anomaly detection:** ML spots unusual cost patterns in real-time (not just monthly review). "S3 cost spiked 300% in the last 4 hours — investigate."
4. **Smart scheduling:** ML learns traffic patterns → auto-schedules scaling (don't just react to CPU — predict demand)
5. **Natural language cost queries:** "Why did our bill increase last week?" → AI analyzes CUR and gives plain-English answer

**What we'd adopt first:** AWS Cost Anomaly Detection (already available, ML-based, near-zero effort to enable).

---

*End of Q&A Bank — 20 questions covering all 7 dimensions*
