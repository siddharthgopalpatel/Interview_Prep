# Behavioral SoftSkills — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 10, 26, 29

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 10: Closing & Soft Skills

---

### Q: As technologies are changing every day, how do you cope with learning?

**Project Reference:** General — learning approach

**Answer:**

> "Three things I do consistently:
>
> 1. **Build, don't just read** — I maintain a personal knowledge base (Obsidian) where I document every technology I learn with hands-on labs. Reading docs ≠ knowing it. I spin up clusters, break things, fix them.
>
> 2. **Follow the problem, not the hype** — I don't chase every new tool. When a real problem arises (e.g., 'how do we do canary deployments?'), I research options, evaluate, and implement. That's how I learned Argo Rollouts, Karpenter, and Istio.
>
> 3. **Community + certifications** — I follow KubeCon talks, AWS re:Invent sessions, and maintain my skills through practical projects on GitHub. Certifications give structure when learning something new end-to-end.
>
> The key: I learn by doing, and I only invest time in technologies that solve real production problems."

---

### Q: Convince me to hire you — based on the most challenging project you've done.

**Project Reference:** P9 (OS Patching Automation)

**Answer:**

> "The most challenging project was building the **OS Patching Automation for 500+ servers**.
>
> **Why it was hard:** You're touching 500+ production Linux servers that run a carrier-grade voice platform. One wrong package, one missed validation — and millions of calls drop. The client had zero tolerance for downtime.
>
> **What I did:**
> - Designed an 18-step zero-touch lifecycle — from ServiceNow CR creation to post-patch validation
> - Built 12 Ansible roles — each independently testable, idempotent, and re-runnable
> - Implemented 7-dimension automated validation (services, ports, connectivity, disk, certs, integrity, logs)
> - ALB traffic drain with serial 20% rolling updates — no user impact
> - One-click rollback if any validation dimension fails
>
> **Result:** 500+ servers patched monthly with ZERO downtime, ZERO human intervention after initiation. Passed every ITIL audit. Reduced patching from a 3-day manual exercise with 2 incidents/month to a 4-hour automated run with zero incidents for 18 months straight.
>
> **Why hire me:** I don't just automate — I build systems that are safe to run without humans watching. I think about failure modes, rollback, and validation before writing a single line of code. That's 10 years of production experience talking."

---

---
---

# SECTION 26: Team Management

---

### Q: Tell me about the management required for your team of six — was it infrastructure or application-oriented?

**Project Reference:** General — team leadership at Ericsson

**Answer:**

> "**Both.** Our team of 10+ spans infrastructure AND application delivery:
>
> - **Infrastructure side (my primary focus):** Terraform modules, Kubernetes platform operations, CI/CD pipeline maintenance, monitoring, patching, DR drills. I own the platform that applications run on.
>
> - **Application support:** We don't write application code, but we own the deployment lifecycle. We help dev teams containerize their apps, write Helm charts, configure pipelines, debug deployment issues, and optimize resource usage.
>
> **My management approach:**
> - I lead technically — architecture decisions, code reviews on Terraform/Ansible PRs, design docs
> - Distribute work by expertise — 2 engineers focused on K8s, 2 on Terraform/AWS, 1 on CI/CD, 1 on monitoring
> - Cross-train so no single point of failure — everyone can handle basic tasks across domains
> - Weekly knowledge sharing sessions — one person demos what they built
>
> **Key insight at senior level:** You're not just managing tasks. You're building a team's capability. I invest in documentation, runbooks, and automation so the team isn't dependent on any one person (including me)."

---
---

# SECTION 29: Behavioral & Leadership

---

### Q: Can you describe a time when you handled a crisis?

**Project Reference:** P9 (OS Patching — before automation), P8 (DR)

**Answer:**

> "**The crisis:** Saturday night, 11 PM. A kernel security patch (CVE with active exploit) was flagged as critical by Red Hat. Client demanded emergency patching of 50 production servers within 24 hours — normally a 3-day planned activity.
>
> **What I did:**
> 1. **Assessed risk** — Read the CVE. Confirmed: remote code execution, public exploit available. Patching couldn't wait.
> 2. **Formed the plan** — Emergency CR in ServiceNow. Identified the 50 highest-risk servers (internet-facing). Decided on serial 10% batches (5 servers at a time).
> 3. **Communicated** — Called the on-call team lead, briefed the client NOC. Set up a bridge call for real-time status updates.
> 4. **Executed** — Used our existing Ansible automation (thank god it was already built). Ran patching in controlled batches. Each batch: drain traffic → patch → reboot → validate → return traffic.
> 5. **Monitored** — Watched Prometheus dashboards for error rate spikes after each batch. Zero issues.
>
> **Result:** 50 servers patched in 6 hours with zero incidents. Client commended the response speed. This validated the investment in automation — without it, this would have been a 48-hour manual marathon with high risk.
>
> **Lesson I applied:** After this, I added an 'emergency fast-track' mode to our automation — fewer validation checks for speed but retains critical ones (service health, connectivity)."

---

### Q: In a crisis where a primary application is down, how would you communicate with stakeholders and distribute work?

**Project Reference:** General — incident management

**Answer:**

> "**Structured incident response:**
>
> **First 5 minutes:**
> 1. Declare incident severity (P1 — customer impacting)
> 2. Open a war room (Zoom/Slack channel) — all relevant people join
> 3. Assign roles: **Incident Commander** (me — coordinates, doesn't debug), **Tech Lead** (hands-on debugging), **Comms lead** (updates stakeholders)
>
> **Communication cadence:**
> - Stakeholders (management/client): Update every 15 minutes — even if the update is 'still investigating, no change.' Silence breeds panic.
> - Format: 'Impact: [what's broken]. Status: [investigating/identified/fixing]. ETA: [if known]. Next update: [time].'
>
> **Work distribution:**
> - Person A: Check application logs, recent deployments, code changes
> - Person B: Check infrastructure — nodes, pods, database, network connectivity
> - Person C: Check external dependencies — third-party APIs, DNS, certificates
> - Me: Coordinate, eliminate duplicate effort, escalate if needed, decide on rollback
>
> **Key principles:**
> - Don't have 5 people troubleshooting the same thing — parallelize
> - If not resolved in 15 min → rollback the last change (most incidents are caused by recent changes)
> - Document actions in real-time (Slack thread) — for postmortem later
> - After resolution: blameless postmortem within 48 hours"

---

### Q: Tell me about a time you influenced your team to do something new or brought a change that helped them.

**Project Reference:** P9 (OS Patching — shift from manual to automation)

**Answer:**

> "**The change:** Moving from manual SSH-based patching to full Ansible Automation Platform.
>
> **The resistance:** Team was comfortable with manual patching. 'We know our servers. Automation is risky — what if it patches the wrong thing?' The fear was valid — automation touching 500+ production servers is scary.
>
> **How I influenced:**
>
> 1. **Started small** — Didn't propose automating everything at once. First automated just the PRE-CHECK (connectivity, disk space, service status). Non-destructive. 'Let's just automate the checks — no actual patching yet.'
>
> 2. **Showed the data** — Tracked manual patching incidents: 2 per month, average 45-min recovery. Showed the team: 'This is what manual errors cost us.'
>
> 3. **Built with the team** — Didn't build it alone and hand it over. Paired with team members on each role. They owned roles they were expert in (cert checks, service validation).
>
> 4. **Ran parallel** — First 3 months: automation ran alongside manual process. Same servers, same window. Compared results. Automation caught 3 things humans missed.
>
> 5. **Celebrated wins** — First fully automated patch cycle with zero issues → team dinner. Made it a team achievement, not 'my automation replacing their jobs.'
>
> **Result:** Team went from resistant to proud. They now present the automation at internal tech talks. Two team members learned Ansible deeply and now contribute roles independently."

---

---
---

