# Developer Workflow in DevOps with Agile Scrum

**Author:** Siddharth Patel
**Purpose:** Understand how developers actually work day-to-day in a DevOps team following Scrum Agile
**Style:** Storytelling with real examples from our platform

---

## The Big Picture — How It All Connects

Imagine you're a new developer joining our team. You've heard of Agile, Scrum, DevOps, Git — but how does it all fit together in real life?

Here's the simple truth:

```
Scrum = HOW we plan and organize work (sprints, stories, ceremonies)
Git   = HOW we write and manage code (branches, PRs, merges)
DevOps = HOW we deliver and operate (CI/CD pipelines, deploy, monitor)

Together = A developer picks up a story → writes code on a branch → 
           pipeline tests & deploys it → users get the feature → 
           feedback comes back → next sprint improves it
```

Let me walk you through a real sprint to show how this works.

---

## Part 1: Git Branching Strategy — The Developer's Daily Tool

### Why Do We Need a Branching Strategy?

Think of `main` branch as the **production highway**. Cars (code) on this highway are live — real users are using them. You can't just park your half-built car on the highway while you're still working on it.

So what do you do? You work in a **garage** (feature branch), finish building, get it inspected (code review), and only then merge it onto the highway.

### Our Branching Strategy: Feature Branch + PR Workflow

We use a simple but powerful strategy:

```
main (production-ready, always deployable)
├── feature/IPCC-123-add-call-routing-logic
├── feature/IPCC-456-fix-ivr-timeout
├── bugfix/IPCC-789-memory-leak-agent-service
└── hotfix/IPCC-999-critical-db-connection
```

### Branch Naming Convention

Every branch starts with the **Jira ticket ID**. This is non-negotiable.

```
Format: <type>/<TICKET-ID>-<short-description>

Examples:
  feature/IPCC-234-implement-skill-based-routing
  bugfix/IPCC-567-fix-null-pointer-in-cdr
  hotfix/IPCC-890-patch-ssl-cert-expiry
```

**Why ticket ID?** Because when someone looks at a branch, a PR, or a commit — they can trace it back to the Jira story instantly. Full traceability from idea to production.

### The Developer's Git Workflow (Step by Step)

Let's say you're working on a new feature: "Add skill-based routing for VIP customers"

```bash
# Step 1: Start from latest main
git checkout main
git pull origin main

# Step 2: Create your feature branch
git checkout -b feature/IPCC-234-skill-based-routing

# Step 3: Work on your code (multiple commits are fine)
git add .
git commit -m "feat(routing): add VIP skill detection logic [IPCC-234]"

git add .
git commit -m "feat(routing): implement priority queue for VIP calls [IPCC-234]"

git add .
git commit -m "test(routing): add unit tests for skill routing [IPCC-234]"

# Step 4: Push to remote
git push origin feature/IPCC-234-skill-based-routing

# Step 5: Create Pull Request (PR) on GitHub/GitLab
# → This triggers CI pipeline automatically
# → Request review from 2 team members
```

### Commit Message Convention

We follow **Conventional Commits** — this isn't just neat, it helps generate changelogs and triggers correct CI behavior:

```
feat(component): description [TICKET-ID]    → New feature
fix(component): description [TICKET-ID]     → Bug fix
refactor(component): description [TICKET-ID] → Code cleanup
test(component): description [TICKET-ID]    → Adding tests
docs(component): description [TICKET-ID]    → Documentation
chore(component): description [TICKET-ID]   → Build/config changes
```

### What Happens When You Push? (The CI Magic)

The moment you push your branch, the DevOps pipeline kicks in:

```
Developer pushes feature branch
        ↓
CI Pipeline Triggers (automatically)
        ↓
┌─────────────────────────────────────┐
│ 1. Secret Scan (Trivy)             │ ← Catches leaked passwords
│ 2. Build (compile/install deps)     │ ← Does the code even build?
│ 3. Unit Tests (pytest)              │ ← Does the logic work?
│ 4. SCA (Snyk)                       │ ← Are dependencies safe?
│ 5. SAST (SonarQube)                 │ ← Is YOUR code secure?
│ 6. Quality Gate                     │ ← Pass/fail on thresholds
└─────────────────────────────────────┘
        ↓
Results posted as PR comment
        ↓
✅ All green → Ready for review
❌ Any red → Fix before review
```

**Key insight:** The developer gets feedback in <10 minutes. They don't wait days for someone to find bugs — the pipeline catches them immediately.

### Pull Request (PR) / Merge Request (MR) — The Quality Gate

> **Quick note:** PR and MR are the **same thing** — different platforms, different names.
> - **GitHub** → Pull Request (PR)
> - **GitLab** → Merge Request (MR)
> - Both mean: "I'm requesting my code be reviewed and merged into the target branch."
> 
> In interviews, use whichever term matches the company's platform. If unsure, say "PR or MR" once to show you know both.

A PR/MR is not just "merge my code." It's a conversation:

```
PR: feature/IPCC-234-skill-based-routing → main

Checklist (enforced):
☑ CI pipeline passes (all stages green)
☑ At least 2 reviewers approved
☑ No unresolved comments
☑ Branch is up-to-date with main
☑ Security scan clean (no HIGH/CRITICAL)
☑ Test coverage maintained (≥80%)
```

**Real example of a PR comment:**
> "The VIP detection logic looks good, but I'd suggest using a strategy pattern here instead of if-else chains — it'll be easier to add new skill types later. Also, the timeout value is hardcoded at line 47 — should this come from ConfigMap?"

### After PR Merges → What Happens Automatically

```
PR merged to main
        ↓
Full CI Pipeline runs again (on main)
        ↓
Docker image built + scanned + signed
        ↓
Image pushed to ECR (with git-sha tag)
        ↓
GitOps repo updated (ArgoCD picks up change)
        ↓
Auto-deployed to DEV environment
        ↓
Smoke tests run → If pass → Auto-deploy to STAGING
        ↓
Manual approval required for PRODUCTION
        ↓
Canary deployment (5% → 20% → 50% → 100%)
```

**The developer didn't do any of this manually.** They just merged a PR. The pipeline handles the rest.

---

## Branching Strategies Comparison (Interview Context)

| Strategy | How It Works | Best For | We Use? |
|---|---|---|---|
| **Feature Branching** | One branch per feature, merge via PR | Most teams | ✅ Yes |
| **GitFlow** | develop + release + hotfix branches | Scheduled releases, regulated | Partial (hotfix concept) |
| **Trunk-Based Dev** | Very short branches (<24h), feature flags | High-velocity, mature CI/CD | Evolving toward this |
| **GitHub Flow** | Feature branch → PR → merge to main → deploy | Simple, continuous delivery | ✅ Closest to this |

### Why We Chose Feature Branching (not Trunk-Based yet)

```
Our reality:
- Carrier-grade telecom platform (99.99% SLA)
- Compliance requirements (SOC2, CIS)
- Need code review before anything hits main
- Multiple teams working on same codebase
- Can't afford "whoops" in production

So: Feature branches (1-3 days max) + PR review + automated pipeline

Future direction: Moving toward trunk-based with feature flags
as our CI/CD matures and team confidence grows.
```



---

## Part 2: How Scrum Works in Our DevOps Team

### The Story of Sprint 47

Let me tell you the story of a real sprint to make this concrete.

**Setting:** Our team manages the Verizon IPCC platform. We follow 2-week sprints. The team has 8 people — 1 Scrum Master, 1 Product Owner, 4 developers, 1 QA engineer, 1 DevOps/Platform engineer (me).

---

### Day 0 (Friday before sprint): Sprint Planning

**9:30 AM — Sprint Planning Meeting (2 hours)**

The Product Owner (PO) comes with the **Product Backlog** — a prioritized list of work:

```
Product Backlog (top items):
1. [IPCC-234] Skill-based routing for VIP customers    — 8 story points
2. [IPCC-235] Fix IVR timeout causing dropped calls    — 3 story points
3. [IPCC-236] Add Prometheus metrics for call queue    — 5 story points
4. [IPCC-237] Upgrade base Docker image (CVE fix)      — 2 story points
5. [IPCC-238] Implement retry logic for SIP failures   — 5 story points
6. [IPCC-239] Dashboard for real-time agent status     — 8 story points
```

**How we decide what to take:**

The team's **velocity** (based on last 3 sprints) = 25 story points per sprint.

```
Team discussion:
PO: "The VIP routing is highest priority — customer escalation."
Dev1: "That's complex — touching routing engine + database + API. I'd say 8 points."
Dev2: "The IVR timeout is a bug affecting 200+ calls/day. Quick fix — 3 points."
Me (DevOps): "Docker image upgrade is critical — there's a HIGH CVE. 2 points, I'll handle it."
QA: "We need the Prometheus metrics to validate the routing change. Let's include IPCC-236."

Result — Sprint 47 Backlog:
☐ IPCC-234 (8 pts) — Skill-based routing
☐ IPCC-235 (3 pts) — Fix IVR timeout
☐ IPCC-236 (5 pts) — Prometheus metrics for call queue
☐ IPCC-237 (2 pts) — Docker image CVE fix
☐ IPCC-238 (5 pts) — SIP retry logic
Total: 23 story points (within our velocity of 25)
```

**Sprint Goal:** "Deliver VIP routing capability with observability, fix the IVR timeout bug."

Each developer picks stories and breaks them into **tasks**:

```
IPCC-234 (Skill-based routing) — owned by Dev1 + Dev2:
  Task 1: Design routing algorithm (2h)
  Task 2: Implement VIP detection from ANI/CRM lookup (6h)
  Task 3: Build priority queue logic (4h)
  Task 4: Write unit tests (3h)
  Task 5: Integration test with IVR system (3h)
  Task 6: Update Helm values for new config (1h)

IPCC-236 (Prometheus metrics) — owned by Me:
  Task 1: Add ServiceMonitor for call-queue service (2h)
  Task 2: Create custom metrics (queue_depth, wait_time, agent_available) (4h)
  Task 3: Build Grafana dashboard (3h)
  Task 4: Set up alerting rules (2h)
```

---

### Day 1-10: Sprint Execution (The Daily Work)

#### Daily Standup (Every morning, 9:15 AM, 15 minutes)

```
Scrum Master: "Alright team, let's go around."

Dev1: "Yesterday: Finished VIP detection logic, pushed PR. 
       Today: Starting priority queue implementation. 
       Blockers: None."

Dev2: "Yesterday: Reviewed Dev1's PR, fixed IVR timeout bug (PR merged, deployed to DEV).
       Today: Writing integration tests for routing.
       Blockers: Need access to staging call simulator — can you help, Siddharth?"

Me:   "Yesterday: Upgraded Docker base image, pipeline passed, deployed to all envs.
       Today: Building ServiceMonitor for IPCC-236, will also unblock Dev2's staging access.
       Blockers: None."

QA:   "Yesterday: Wrote test cases for VIP routing.
       Today: Testing the IVR fix in DEV environment.
       Blockers: None."

Scrum Master: "Great. Dev2's blocker — Siddharth will handle by noon. Anything else? No? Done."
```

**Total time: 12 minutes.** Everyone knows what everyone is doing. Blockers get solved same day.

---

#### A Developer's Typical Day (Dev1 working on IPCC-234)

```
9:00 AM  — Check Jira board, see task status
9:15 AM  — Daily standup
9:30 AM  — Pull latest main, continue coding on feature/IPCC-234-skill-based-routing
11:00 AM — Push code, CI runs automatically (takes 8 mins)
11:08 AM — CI passes ✅ — continue working
12:00 PM — Lunch
1:00 PM  — Review teammate's PR (IPCC-235 IVR fix) — approve with comment
1:30 PM  — Continue on priority queue logic
3:00 PM  — Push again, CI runs
3:08 PM  — CI fails ❌ — unit test broken (missed edge case)
3:30 PM  — Fix the test, push again
3:38 PM  — CI passes ✅
4:00 PM  — Create PR for first part of feature (VIP detection)
4:30 PM  — Update Jira task: "In Review"
5:00 PM  — End of day
```

**Notice:** The developer doesn't manually test on servers, doesn't deploy anywhere, doesn't worry about infrastructure. They write code, push, and the pipeline gives feedback in minutes.

---

#### How a Story Moves Through the Board

```
Jira Board (Sprint 47):

TO DO          │  IN PROGRESS    │  IN REVIEW      │  DONE
───────────────┼─────────────────┼─────────────────┼──────────────
IPCC-238       │  IPCC-234       │  IPCC-235       │  IPCC-237
(SIP retry)    │  (VIP routing)  │  (IVR timeout)  │  (Docker CVE)
               │  IPCC-236       │                 │
               │  (Prometheus)   │                 │
```

A story goes through these states:
```
To Do → In Progress → In Review → QA Testing → Done
         (coding)     (PR open)   (verified)   (deployed + accepted)
```



---

### Day 10 (Thursday): Sprint Review (Demo)

**2:00 PM — Sprint Review Meeting (1 hour)**

The team demos completed work to the Product Owner and stakeholders:

```
Demo 1: Dev1 shows VIP routing working in Staging
  → Dials VIP number → Call routes to skilled agent queue
  → PO: "Can we also route based on language preference?"
  → Captured as new story for next sprint

Demo 2: Dev2 shows IVR timeout fix
  → Before: 200 dropped calls/day. After: Zero in last 5 days.
  → PO: "Excellent. This was a major customer complaint."

Demo 3: Me shows Prometheus dashboard
  → Live call queue metrics, alerting when queue > 50 calls
  → Stakeholder: "Can we add agent utilization?" 
  → Added to backlog

Demo 4: QA shows test coverage report
  → New routing logic: 92% coverage
  → No critical bugs in staging
```

**Key point:** The demo shows DEPLOYED, WORKING software — not slides, not "it works on my machine." Because our pipeline auto-deploys to staging, the PO sees real features in a real environment.

---

### Day 10 (Thursday): Sprint Retrospective

**3:30 PM — Retrospective (1 hour, team only)**

```
What went well? ✅
- IVR bug fixed and deployed in 2 days (fast turnaround)
- Pipeline caught a secret leak in Dev2's first commit (saved us)
- New Prometheus dashboard already caught a production anomaly

What didn't go well? ❌
- IPCC-238 (SIP retry) didn't get started — we overcommitted
- Flaky integration test blocked PRs for half a day
- Dev1 waited 4 hours for PR review (only 1 reviewer online that day)

Action items for next sprint:
1. Review SLA: PRs must be reviewed within 2 hours (Scrum Master to track)
2. Fix or quarantine the flaky test (Me — by Day 2 of next sprint)
3. Take fewer points next sprint (22 instead of 25 — we have a holiday)
```

---

### Sprint Velocity & Burndown

```
Sprint 47 Results:
  Committed: 23 story points
  Completed: 18 story points (IPCC-238 carried over)
  Velocity: 18 (this sprint)
  
  3-sprint average velocity: (25 + 22 + 18) / 3 = 21.6 ≈ 22

Burndown Chart:
Story Points
│ 23 ├──╲
│ 20 │   ╲___          ← Ideal line
│ 15 │       ╲___
│ 10 │    ╲       ╲
│  5 │     ╲___    ╲
│  0 │──────────╲───╲──
     Day1   Day5   Day10

(Our actual line was above ideal on Day 3-5 because
 VIP routing was harder than estimated, but we caught up)
```

---

## Part 3: How Scrum Agile Integrates with the DevOps Cycle

This is the crucial part — how the Scrum process and DevOps pipeline form one continuous loop.

### The Infinite Loop

```
         ┌──────── PLAN ────────┐
         │                      │
    FEEDBACK               DEVELOP
    (Monitor +               (Sprint
    Retrospective)            Execution)
         │                      │
         └──── DELIVER ─────────┘
              (CI/CD Pipeline)
```

Let me map each Scrum ceremony to the DevOps cycle:

### Mapping: Scrum Ceremonies → DevOps Activities

| Scrum Ceremony | DevOps Activity | How They Connect |
|---|---|---|
| **Sprint Planning** | Backlog includes infra/pipeline work | DevOps tasks get story points too (CVE fix, dashboard, scaling) |
| **Daily Standup** | Pipeline status, deployment updates | "Pipeline is green", "Prod deploy went smooth", "Alert fired at 2 AM" |
| **Sprint Execution** | CI/CD running on every push | Every commit → automated build → test → scan → deploy |
| **Sprint Review** | Demo on deployed environment | Features shown in STAGING (not localhost) — real deployed software |
| **Retrospective** | Pipeline metrics, incident postmortems | "Pipeline took 25 min — let's optimize", "MTTR was 45 min — too slow" |

### The Complete Sprint-to-Deploy Flow (Story of One Feature)

Let me trace one user story from idea to production:

```
📋 STEP 1: Sprint Planning (Day 0)
   PO creates story: "As a VIP customer, I want my call routed to a skilled agent"
   Team estimates: 8 story points
   Dev1 takes ownership
   
💻 STEP 2: Development (Day 1-4)
   Dev1 creates: feature/IPCC-234-skill-based-routing
   Writes code → commits → pushes
   CI pipeline runs on every push (feedback in 8 min)
   
🔍 STEP 3: Code Review (Day 4-5)
   Dev1 opens PR → 2 reviewers review
   Pipeline results visible on PR (all green ✅)
   Reviewers approve
   
🔀 STEP 4: Merge (Day 5)
   PR merged to main
   Full pipeline triggers:
   Build → Test → Scan → Docker Build → Sign → Push to ECR
   
🚀 STEP 5: Auto-Deploy to DEV (Day 5, automated)
   ArgoCD detects new image in GitOps repo
   Deploys to DEV cluster automatically
   Smoke tests pass ✅
   
🧪 STEP 6: Deploy to STAGING (Day 5-6, automated)
   ArgoCD promotes to STAGING
   Integration tests + DAST run
   QA validates the feature manually
   
✅ STEP 7: Deploy to PRODUCTION (Day 7, manual approval)
   Tech Lead approves production deployment
   Canary rollout: 5% → 20% → 50% → 100%
   Prometheus monitors success rate at each step
   All good → Full rollout complete
   
📊 STEP 8: Monitor (Day 7 onwards)
   Prometheus tracks: call routing success rate, latency, errors
   Grafana dashboard shows VIP calls being routed correctly
   Alert set: if routing failure > 2% → page on-call
   
🔄 STEP 9: Feedback (Next Sprint Planning)
   PO: "Customers love VIP routing. Can we add language-based routing too?"
   New story created → prioritized → picked up in next sprint
```

**This is the DevOps + Scrum loop in action.** The feature goes from a Post-it on the board to running in production within ONE sprint.

---

### How DevOps Tasks Live Inside Sprints

A common question: "Does the DevOps engineer just maintain pipelines, or do they participate in Scrum?"

**Answer: Fully embedded in the sprint.** DevOps work gets story points just like feature work.

```
Typical sprint backlog mix:

Feature work (Dev team):        70% of points
  - New features, bug fixes, refactoring

DevOps/Platform work (Me):      20% of points
  - Pipeline improvements
  - Monitoring dashboards
  - Security fixes (CVEs)
  - Infrastructure changes
  - Performance optimization

Tech debt:                      10% of points
  - Flaky test fixes
  - Documentation
  - Dependency upgrades
```

**Example of my sprint tasks:**
```
IPCC-237: Upgrade Docker base image (CVE-2026-1234)     — 2 pts
IPCC-236: Add Prometheus metrics for call queue          — 5 pts
IPCC-240: Fix flaky integration test (retro action item) — 2 pts
IPCC-241: Set up Karpenter for spot instance savings     — 3 pts
```

I participate in standups, I demo my work in sprint review, and my velocity counts toward the team total.

---

### The Definition of Done — Where DevOps Meets Scrum

In pure Agile, "Done" might mean "code written and tested." In our DevOps world, "Done" means much more:

```
Definition of Done (our team):

□ Code written and compiles
□ Unit tests written and passing (≥80% coverage)
□ Code reviewed and approved (2 reviewers)
□ CI pipeline green (all security scans pass)
□ No HIGH/CRITICAL vulnerabilities
□ Deployed to STAGING and verified
□ QA approved (acceptance criteria met)
□ Deployed to PRODUCTION (canary successful)
□ Monitoring confirms no errors (24h observation)
□ Product Owner accepts in Sprint Review
```

**Notice:** A story isn't "Done" when code is merged. It's done when it's **running in production without issues**. This is the DevOps mindset applied to Scrum.



---

## Part 4: Real Scenarios — When Things Don't Go Smoothly

### Scenario 1: Hotfix During a Sprint

```
Tuesday, 3 PM — Production alert: "SSL certificate expiring in 2 hours"

What happens:
1. On-call (Me) gets paged → immediate investigation
2. Create hotfix branch: hotfix/IPCC-999-patch-ssl-cert-expiry
3. Fix applied → PR with emergency review (1 reviewer sufficient for hotfix)
4. Pipeline runs (fast-track — skip non-critical stages)
5. Deployed to PROD within 30 minutes
6. Create Jira ticket retroactively (for tracking)
7. In next standup: "Handled SSL hotfix yesterday, added cert monitoring to prevent recurrence"
8. Retro action item: "Set up cert expiry alerting at 30 days, not 2 hours"
```

**Key point:** Hotfixes bypass normal sprint flow but still go through Git (branch → PR → pipeline → deploy). We don't SSH into servers and fix things manually.

---

### Scenario 2: Story Spills Over to Next Sprint

```
Sprint 47 ends. IPCC-238 (SIP retry logic) wasn't started.

What happens:
1. In Sprint Review: "We didn't complete IPCC-238 — VIP routing was more complex than estimated."
2. PO decides: Keep it in next sprint backlog (still high priority)
3. Retrospective: "Why did we miss it?"
   → We underestimated IPCC-234 (estimated 8, actually took 13 points of effort)
   → Action: Break large stories into smaller ones (max 5 points each)
4. Next sprint planning: IPCC-238 is first in backlog, taken immediately
```

**Learning:** This is normal in Scrum. We don't extend the sprint. We carry over and improve estimation.

---

### Scenario 3: Pipeline Breaks Mid-Sprint

```
Wednesday morning — All PRs are failing. CI pipeline broken.

What happens:
1. Multiple developers report: "My PR pipeline failed — but my code is fine"
2. DevOps (Me) checks: "SonarQube server is down — affecting all pipelines"
3. Standup update: "Blocker — SonarQube down, I'm fixing it. ETA 1 hour."
4. I fix SonarQube, re-trigger failed pipelines
5. All PRs go green within 2 hours
6. No sprint impact (just a few hours delay)

If it was longer (>1 day):
→ Scrum Master escalates
→ Team re-plans: "Can we work on tasks that don't need pipeline?"
   (documentation, design, local testing)
```

---

### Scenario 4: Security Vulnerability Discovered Mid-Sprint

```
Thursday — Snyk alerts: Critical CVE in Django 4.2.2 (our app's framework)

What happens:
1. Auto-created Jira ticket: IPCC-VULN-001 (Critical — 24h SLA)
2. Scrum Master: "This is unplanned but critical. We need to fit it in."
3. Dev2 takes it (smallest current workload)
4. Branch: bugfix/IPCC-VULN-001-django-upgrade
5. Upgrade Django → run tests → push → pipeline validates → PR → merge
6. Deployed to all environments within 4 hours
7. Sprint capacity adjustment: Dev2's other story might slip

In Retrospective:
"We handled the CVE well (4h turnaround), but it cost us 5 hours of sprint capacity.
 Action: Schedule monthly dependency upgrades as preventive maintenance."
```

---

## Part 5: The Tools That Tie It All Together

### How Jira + Git + Pipeline = Full Traceability

```
Jira Story: IPCC-234 "Skill-based routing for VIP customers"
        ↓
Git Branch: feature/IPCC-234-skill-based-routing
        ↓
Commits: "feat(routing): add VIP detection [IPCC-234]"
        ↓
PR: Links to IPCC-234 automatically (Smart Commits)
        ↓
Pipeline: Build #1247 — triggered by PR merge
        ↓
Deploy: ArgoCD sync — app v2.3.1 deployed to production
        ↓
Jira: Ticket auto-transitions to "Done" when deployed

Result: PO can click IPCC-234 and see:
  - Who worked on it
  - What code changed (PR link)
  - When it was deployed
  - Which build number
  - Current status in production
```

### Tools Map

| Purpose | Tool | Role |
|---|---|---|
| Project Management | Jira | Sprint boards, stories, tracking |
| Source Control | GitHub/GitLab | Code, branches, PRs |
| CI Pipeline | Jenkins | Build, test, scan, package |
| CD/GitOps | ArgoCD | Deploy to Kubernetes |
| Container Registry | ECR | Store Docker images |
| Monitoring | Prometheus + Grafana | Observe production health |
| Alerting | AlertManager + PagerDuty | Notify on issues |
| Communication | Slack | Daily updates, alerts, notifications |
| Documentation | Confluence | Architecture docs, runbooks |

---

## Part 6: Interview Quick Reference

### "Tell me about your development workflow"

> "We follow Scrum with 2-week sprints. The PO maintains a prioritized product backlog, and during sprint planning the team pulls stories based on our velocity (about 22 points/sprint). Developers work on feature branches named with the Jira ticket ID — like `feature/IPCC-234-skill-routing`. Every push triggers our 18-stage CI pipeline that runs security scans, tests, and quality gates. Once the PR is reviewed and merged, ArgoCD auto-deploys to DEV, then STAGING with integration tests, and PRODUCTION requires manual approval with canary deployment. The feature is only 'Done' when it's running in production with no errors for 24 hours."

### "How does DevOps fit into Agile Scrum?"

> "DevOps is fully embedded in our Scrum team — not a separate team. I participate in sprint planning, take DevOps stories with story points (pipeline improvements, monitoring, security fixes), attend standups, and demo my work in sprint review. The pipeline IS what makes Scrum work at speed — without automated CI/CD, we couldn't deliver working software every sprint. And monitoring feeds back into sprint planning — production metrics drive which bugs and improvements we prioritize next."

### "What branching strategy do you use and why?"

> "Feature branching with GitHub Flow — one branch per Jira story, short-lived (1-3 days max), merged via PR with 2 approvals. Branch naming includes ticket ID for traceability. We chose this over GitFlow because we deploy continuously (no scheduled releases) and over trunk-based because we need PR reviews for compliance. Our pipeline validates every branch push in <10 minutes, so developers get fast feedback without blocking."

### "How do you handle unplanned work during a sprint?"

> "We have a rule: Critical production issues and HIGH CVEs get immediate attention — they bypass sprint planning but still go through Git workflow (branch → PR → pipeline → deploy). The Scrum Master tracks unplanned work as sprint disruption. In retrospectives, we measure planned vs unplanned ratio — if it's >20% unplanned, we investigate root cause and add preventive work to future sprints."

---

## Summary: The Complete Picture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SCRUM + DevOps = Continuous Delivery              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  PLAN (Scrum)          │  BUILD (DevOps)        │  RUN (DevOps)    │
│  ──────────────        │  ──────────────        │  ──────────────  │
│  Sprint Planning       │  Git branch + code     │  Deploy (canary) │
│  User Stories          │  CI Pipeline           │  Monitor         │
│  Story Points          │  Security Scans        │  Alert           │
│  Sprint Backlog        │  Docker Build          │  Auto-rollback   │
│                        │  PR Review             │  Feedback        │
│                        │                        │                  │
│  ──── Daily Standup keeps everyone synced across all three ────     │
│                                                                     │
│  Sprint Review: Demo DEPLOYED features (not localhost)              │
│  Retrospective: Improve BOTH dev process AND pipeline/infra         │
│                                                                     │
│  LOOP: Feedback from monitoring → drives next sprint priorities     │
└─────────────────────────────────────────────────────────────────────┘
```

**The golden rule:**
> Ship working software every sprint, deployed to production automatically, monitored continuously, and improved via retrospectives.

---

---

## Part 7: Feature Flags — Deploying Incomplete Code Safely

### The Problem

You're mid-sprint. Your feature (VIP routing) takes 8 days to build, but the sprint is 10 days. Other developers are merging to `main` daily. If you wait until Day 8 to merge, you'll have massive merge conflicts.

**Solution:** Feature Flags (also called Feature Toggles).

### How It Works

```
Concept: Deploy your code to production EVERY DAY — but hide it behind a flag.
         Users don't see it until you flip the switch.
```

```python
# In your routing service
if feature_flags.is_enabled("vip_skill_routing", user=caller):
    # New VIP routing logic (only enabled for test users)
    route_to_skilled_agent(caller)
else:
    # Old routing logic (what everyone still sees)
    route_to_next_available(caller)
```

### Feature Flag Lifecycle

```
Sprint Day 1: Dev creates flag "vip_skill_routing" → disabled by default
Sprint Day 2: Code merged to main with flag OFF → deployed to prod (nobody sees it)
Sprint Day 4: More code merged → still flag OFF in prod
Sprint Day 6: Flag turned ON for internal testers → they validate in prod
Sprint Day 8: Flag turned ON for 5% of users (canary)
Sprint Day 10: Flag ON for 100% → feature launched
Sprint+2 weeks: Remove flag from code (cleanup) → tech debt if forgotten
```

### Types of Feature Flags

| Type | Purpose | Lifespan | Example |
|---|---|---|---|
| **Release Flag** | Hide incomplete work | Days to weeks | `vip_routing_enabled` |
| **Experiment Flag** | A/B testing | Weeks | `new_ivr_menu_v2` |
| **Ops Flag** | Kill switch for emergencies | Permanent | `enable_external_api_calls` |
| **Permission Flag** | Feature for specific users | Permanent | `premium_routing_tier` |

### Tools for Feature Flags

| Tool | Type | Best For |
|---|---|---|
| LaunchDarkly | Enterprise SaaS | Full dashboard, targeting, analytics |
| Unleash | Open source | Self-hosted, privacy-first |
| Flagsmith | Open source + hosted | Flexible, multi-platform |
| AWS AppConfig | AWS-native | Integrates with Lambda/ECS/EKS |
| K8s ConfigMap | Simple | Ops flags, environment toggles |

### How Feature Flags Enable Trunk-Based Development

```
Without flags:
  Developer works on branch for 8 days → massive merge conflict → risky merge

With flags:
  Developer merges daily (code hidden behind flag) → zero merge conflicts
  → flag flipped when ready → instant launch with no deploy needed
```

**Interview insight:** "We're evolving toward trunk-based development using feature flags. Currently we use short-lived feature branches (1-3 days), but for larger features that span multiple days, we use flags to merge daily while keeping the feature hidden."


---

## Part 8: Git Merge vs Rebase — Keeping Branches Up-to-Date

### The Scenario

You created `feature/IPCC-234` from `main` on Monday. It's now Wednesday. Other developers have merged 5 PRs to `main`. Your branch is **behind**.

You have two options:

### Option 1: Merge (what we use)

```bash
git checkout feature/IPCC-234
git merge main
# Creates a merge commit — preserves history
```

```
Result (history):
main:    A──B──C──D──E──────────M (merge commit from your PR)
                    \          /
feature:             F──G──H──┘
```

**Pros:** Safe, preserves full history, never rewrites commits
**Cons:** Merge commits clutter history

### Option 2: Rebase

```bash
git checkout feature/IPCC-234
git rebase main
# Replays your commits on top of latest main — rewrites history
```

```
Result (history):
main:    A──B──C──D──E──F'──G'──H' (linear, clean)
```

**Pros:** Clean linear history, easier to read
**Cons:** Rewrites commit hashes, dangerous if branch is shared

### Our Team Rule

```
Rule: Merge for updating branches. Never force-push shared branches.

Why:
- Multiple people might be working on the same feature branch
- Rebase rewrites history → causes conflicts for others
- In regulated environments, rewriting history = audit nightmare
- Merge commits clearly show when integration happened

Exception: You MAY rebase your OWN local commits before pushing
           (squash messy "WIP" commits into clean ones)
```

### Squash Merge (PR Merge Strategy)

When merging PRs, we use **squash merge**:

```bash
# Instead of bringing all 15 messy commits into main:
# "WIP", "fix typo", "forgot file", "actually fix it"

# Squash merge creates ONE clean commit:
# "feat(routing): implement VIP skill-based routing [IPCC-234]"
```

**Interview answer:** "We use merge to update feature branches from main — it's safe and preserves history. For PR merges into main, we use squash merge to keep main's history clean — one commit per feature/story. We never rebase shared branches because it rewrites history and causes problems in a team environment."


---

## Part 9: Release Management — From Sprint Increment to Production Release

### How Sprints Become Releases

In traditional software, there's a "release day." In our world, **every merged PR is potentially a release.** But we still version things for tracking.

### Semantic Versioning (SemVer)

```
Format: MAJOR.MINOR.PATCH

v2.3.1
│ │ └── PATCH: Bug fix (backward compatible) — e.g., fix IVR timeout
│ └──── MINOR: New feature (backward compatible) — e.g., add VIP routing
└────── MAJOR: Breaking change (not backward compatible) — e.g., API v2

Examples:
v1.0.0 → v1.0.1 (patched a bug)
v1.0.1 → v1.1.0 (added new feature)
v1.1.0 → v2.0.0 (changed API contract — old clients break)
```

### How We Tag Releases

```bash
# After Sprint 47 review, PO says "ship it"
# Tag the current main with the version

git checkout main
git pull origin main
git tag -a v2.4.0 -m "Sprint 47: VIP routing, IVR fix, call queue metrics"
git push origin v2.4.0

# This tag becomes the official "Sprint 47 Release"
```

### Image Tagging Strategy

```
In our pipeline, Docker images get TWO tags:

1. Git SHA (unique, immutable, for tracing):
   ecr.aws/ipcc/routing-service:abc1234

2. Semantic version (human-readable, for releases):
   ecr.aws/ipcc/routing-service:v2.4.0

Rule: NEVER use `:latest` in production. 
      latest is mutable → you can't trace what's running.
```

### Release Notes (Auto-Generated from Commits)

Because we use Conventional Commits, we auto-generate changelogs:

```markdown
## v2.4.0 (Sprint 47) — 2026-07-25

### Features
- feat(routing): implement VIP skill-based routing [IPCC-234]
- feat(monitoring): add Prometheus metrics for call queue [IPCC-236]

### Bug Fixes
- fix(ivr): resolve timeout causing dropped calls [IPCC-235]

### Security
- chore(docker): upgrade base image to fix CVE-2026-1234 [IPCC-237]
```

### Release vs Deploy

```
Important distinction:

Release = Making a version available (tag it, document it, announce it)
Deploy  = Putting it in production (ArgoCD syncs it to the cluster)

In our flow:
- Every PR merge → auto-deploys to DEV/STAGING (continuous deployment to lower envs)
- Sprint end → tag release → manual approval → canary deploy to PROD
- Hotfix → immediate release + deploy (bypasses sprint cadence)
```


---

## Part 10: Environment Strategy — Branches to Environments

### How Environments Map to Our Workflow

```
┌──────────────┬────────────────────┬────────────────────────────────────┐
│ Environment  │ Deployed From      │ Purpose                            │
├──────────────┼────────────────────┼────────────────────────────────────┤
│ LOCAL        │ Feature branch     │ Developer's laptop (docker-compose)│
│ DEV          │ main (auto)        │ Integration testing, team testing  │
│ STAGING      │ main (auto + gate) │ QA validation, DAST, performance   │
│ PRODUCTION   │ main (manual + canary) │ Real users, 99.99% SLA         │
└──────────────┴────────────────────┴────────────────────────────────────┘
```

### The Flow Visualized

```
feature/IPCC-234 → PR merged to main
                          │
                          ▼
                    ┌─── DEV ───┐  (auto-deploy, smoke tests)
                    │  Passes?  │
                    └─── Yes ───┘
                          │
                          ▼
                   ┌── STAGING ──┐  (auto-deploy, DAST + integration tests)
                   │   Passes?   │
                   └──── Yes ────┘
                          │
                          ▼
                   ┌─ PRODUCTION ─┐  (manual approval → canary rollout)
                   │   Healthy?   │
                   └──── Yes ─────┘
                          │
                          ▼
                    Feature is LIVE 🎉
```

### Ephemeral Environments (Per-PR Preview)

For larger features, we spin up **temporary environments per PR**:

```
Developer opens PR for feature/IPCC-234
        ↓
Pipeline creates: dev-ipcc-234.internal.platform.com
        ↓
Developer + QA can test the feature in isolation
        ↓
PR merged → ephemeral environment auto-destroyed

Benefits:
- Test without polluting shared DEV environment
- Multiple features tested simultaneously without interference
- QA can validate before merge (shift-left testing)
```

### Environment Parity

```
Principle: All environments should be as similar as possible.
           Differences should be ONLY in scale, not in architecture.

What's SAME across all environments:
- Same Docker image (built once, promoted)
- Same Kubernetes manifests (Kustomize overlays for differences)
- Same monitoring stack (Prometheus, Grafana)
- Same network architecture (VPC, subnets, security groups)

What DIFFERS (via Kustomize overlays):
- Replicas: DEV=1, STAGING=2, PROD=3
- Resources: DEV=256Mi, STAGING=512Mi, PROD=1Gi
- Domains: dev.internal, staging.internal, app.platform.com
- Database: DEV=single instance, PROD=Multi-AZ Aurora
- Autoscaling: DEV=off, PROD=on (Karpenter + HPA)
```


---

## Part 11: Code Review Best Practices

### Why Code Review Matters in DevOps

Code review isn't just "find bugs." It's:
- Knowledge sharing (team learns from each other)
- Security gate (catch vulnerabilities before they ship)
- Quality gate (maintain codebase standards)
- Compliance requirement (SOC2 requires separation of duties)

### Our Code Review Rules

```
1. Minimum 2 approvals required for any PR/MR
2. Author cannot approve their own PR (obvious, but enforced via settings)
3. Review SLA: Respond within 2 hours (action item from our retro)
4. PR size limit: Max 400 lines changed (split larger work into smaller PRs)
5. CI must pass before review is requested
6. At least 1 reviewer must be senior (for mentoring + architectural review)
```

### What Reviewers Look For

```
Tier 1 — Correctness (does it work?):
  □ Logic is correct
  □ Edge cases handled
  □ Error handling present
  □ Tests cover the change

Tier 2 — Security (is it safe?):
  □ No hardcoded secrets
  □ Input validation present
  □ No SQL injection patterns
  □ Proper authorization checks

Tier 3 — Maintainability (is it clean?):
  □ Code is readable (clear names, comments where needed)
  □ No unnecessary complexity
  □ Follows team conventions
  □ DRY — no copy-paste duplication

Tier 4 — Operational (will it run well?):
  □ Logging added for debugging
  □ Metrics/monitoring considered
  □ Config externalized (not hardcoded)
  □ Backward compatible with existing deployments
```

### Good vs Bad PR Comments

```
❌ Bad: "This is wrong."
✅ Good: "This might cause a null pointer if the caller has no VIP flag set. 
          Consider adding a default: `caller.get('vip_level', 'standard')`"

❌ Bad: "Rewrite this."
✅ Good: "This if-else chain will grow as we add more routing types. 
          A strategy pattern here would make it easier to extend. 
          Happy to pair on this if helpful."

❌ Bad: "LGTM" (with no actual review)
✅ Good: "Reviewed the routing logic — looks solid. One question: 
          is the 30s timeout in line 47 from config or should it be?"
```

### Handling Disagreements in Reviews

```
Scenario: Reviewer says "use Helm", author says "Kustomize is simpler here"

Our process:
1. Discuss in PR comments (async, documented)
2. If no agreement in 2 rounds → take it to a 15-min sync call
3. Decide based on data, not opinions ("show me the PR diff in both approaches")
4. Document the decision (ADR — Architecture Decision Record)
5. Move on. No grudges. The team decided.
```


---

## Part 12: Cross-Team Collaboration — Multiple Scrum Teams, One Platform

### The Challenge

Our platform has 3 Scrum teams working on the same codebase:
- **Team Routing** — Call routing engine, skill-based routing
- **Team IVR** — Interactive Voice Response, call flows
- **Team Platform** (my team) — Infrastructure, CI/CD, monitoring, security

### How We Avoid Chaos

#### 1. Service Ownership (Who Owns What)

```
routing-service/        → Team Routing owns
ivr-service/            → Team IVR owns
call-queue-service/     → Team Routing owns
agent-gateway/          → Team IVR owns
platform-infra/         → Team Platform owns
pipeline-templates/     → Team Platform owns
monitoring-config/      → Team Platform owns

Rule: You OWN your service = you review PRs, you get paged, you fix bugs.
      CODEOWNERS file enforces this in GitHub.
```

#### 2. API Contracts (Don't Break Each Other)

```
Problem: Team Routing changes their API response format → Team IVR's service breaks

Solution: Contract Testing (Pact)

How it works:
1. Team IVR defines: "I expect routing-service to return { agent_id, skill, queue_position }"
2. This contract is stored in a shared Pact Broker
3. Team Routing's CI runs contract verification BEFORE merging
4. If their change breaks the contract → CI fails → they can't merge

Result: Breaking changes are caught BEFORE deployment, not after.
```

#### 3. Shared vs Team-Specific Pipelines

```
Shared (owned by Platform team):
  - Pipeline templates (build, scan, deploy stages)
  - Security scanning configuration
  - ArgoCD deployment patterns
  - Base Docker images

Team-specific (owned by each team):
  - Jenkinsfile/workflow calling shared templates
  - Helm values per service
  - Test configurations
  - Service-specific monitoring rules
```

#### 4. Coordinated Releases (When Needed)

```
Most deploys: Independent — each team deploys when ready (microservices!)

But sometimes: Coordinated — when a feature spans multiple services

Example: "VIP routing" needs changes in routing-service AND ivr-service

How:
1. Feature flag in BOTH services (deployed independently, both hidden)
2. Integration testing in STAGING with both flags ON
3. Coordinated flag flip in production (same time)
4. OR: ArgoCD sync waves (routing deploys first → then IVR)
```

#### 5. Scrum of Scrums (Cross-Team Sync)

```
When: Twice a week (Tuesday + Thursday), 15 minutes
Who: Scrum Master from each team (or a representative)
Format:
  - What did your team accomplish that affects other teams?
  - What are you planning that might affect other teams?
  - Any cross-team blockers?

Example:
  Team Routing SM: "We're changing the routing API response format next sprint.
                    Team IVR — will this affect you?"
  Team IVR SM: "Yes — give us the new contract, we'll update our consumer."
  Team Platform SM: "New pipeline version v3.2 releasing Friday — 
                     please test your builds against it in staging."
```


---

## Part 13: On-Call & Incident Management in Scrum

### How On-Call Works Within a Sprint

```
On-call rotation: 1 week per person, rotating across the team
On-call person: Still in the sprint, but with reduced capacity

Sprint Planning adjustment:
  Normal capacity: 6 productive hours/day × 10 days = 60 hours
  On-call week capacity: 4 productive hours/day × 5 days = 20 hours (that week)
  
  If you're on-call for 1 week of a 2-week sprint:
  Your capacity = 60 (week 1) + 20 (week 2 on-call) = 40 hours total
  Take fewer story points accordingly
```

### When a Production Incident Happens Mid-Sprint

```
Tuesday 2 AM — PagerDuty alert: "Call routing failure rate > 5%"

Incident Timeline:
─────────────────────────────────────────────────────────
2:00 AM  Alert fires → On-call (Dev1) woken up
2:05 AM  Dev1 acknowledges, starts investigating
2:15 AM  Identifies: recent deploy caused memory leak in routing service
2:20 AM  Decision: Rollback immediately (ArgoCD → git revert → auto-sync)
2:22 AM  Rollback complete. Routing restored. Monitoring confirms recovery.
2:30 AM  Incident documented in Slack channel, severity: P1
2:35 AM  Dev1 goes back to sleep

Next morning:
9:15 AM  Standup: "P1 incident last night — routing memory leak after yesterday's 
         deploy. Rolled back. Need to investigate root cause today."
         
Same day:
         Dev1 investigates → finds the bug → creates bugfix PR
         Root cause: unbounded cache in new VIP routing logic
         
Sprint impact:
         Dev1 lost ~6 hours (night + morning investigation)
         Scrum Master adjusts: "Dev1, drop IPCC-238 from this sprint if needed"
─────────────────────────────────────────────────────────
```

### Incident Postmortem → Sprint Action Items

After every P1/P2 incident, we do a **blameless postmortem**:

```
Postmortem: Routing Memory Leak (Sprint 47, Day 6)

What happened: VIP routing feature had unbounded cache → OOM after 4 hours
Root cause: No cache eviction policy, load testing didn't simulate long-running scenario
Impact: 12 minutes of degraded routing (5% failure rate)
Resolution: Rolled back in 2 minutes (ArgoCD)

What went well:
  ✅ Alert fired within 60 seconds
  ✅ Rollback completed in 2 minutes
  ✅ On-call responded in 5 minutes

What needs improvement:
  ❌ Load test didn't catch this (only tested 30-min runs, not 4-hour)
  ❌ No memory limit set on the pod (should have OOMKilled before affecting routing)
  ❌ Cache had no TTL or max-size

Action items (become sprint stories):
  IPCC-250: Add memory limits to all routing pods (2 pts) → Next sprint
  IPCC-251: Extend load test duration to 8 hours (3 pts) → Next sprint  
  IPCC-252: Add cache eviction policy (LRU, max 10K entries) (3 pts) → This sprint (hotfix)
```

### Tracking Unplanned Work

```
Sprint 47 Summary:
  Planned work:    23 story points
  Completed:       18 story points
  Unplanned work:  8 story points (SSL hotfix + incident + CVE fix)
  
  Unplanned ratio: 8 / (18+8) = 30% ← Too high!

Retrospective discussion:
  "30% of our sprint was unplanned. We need to:
   1. Add preventive monitoring (cert expiry alert at 30 days)
   2. Schedule dependency upgrades monthly (prevent surprise CVEs)
   3. Budget 15-20% of sprint capacity for unplanned work"

Next sprint planning:
  Velocity target: 22 points
  Planned capacity: 18 points (leaving 4 points buffer for unplanned)
```


---

## Part 14: DORA Metrics — Measuring Team Performance

### What are DORA Metrics?

DORA (DevOps Research and Assessment) defined 4 key metrics that predict software delivery performance. These connect directly to our Scrum + DevOps workflow:

### The 4 Metrics

| Metric | What It Measures | Elite | High | Medium | Low |
|---|---|---|---|---|---|
| **Deployment Frequency** | How often we deploy to prod | Multiple/day | Weekly | Monthly | Monthly+ |
| **Lead Time for Changes** | Commit to production time | <1 hour | <1 week | <1 month | 6+ months |
| **Change Failure Rate** | % of deploys causing issues | <5% | 5-10% | 10-15% | 15%+ |
| **Mean Time to Recovery** | How fast we fix production issues | <1 hour | <1 day | <1 week | 1+ week |

### Our Team's DORA Metrics

```
Deployment Frequency:  ~3-5 deploys/week to production
                       (multiple/day to DEV, daily to STAGING)
                       → HIGH tier

Lead Time for Changes: ~2-3 days (story picked up → in production)
                       PR review (2h) + pipeline (20min) + canary (1h)
                       → HIGH tier

Change Failure Rate:   ~3% (1 in ~30 deploys needs rollback)
                       Thanks to: canary deployment, security scanning, test coverage
                       → ELITE tier

MTTR:                  ~5 minutes (auto-rollback via canary)
                       ~15 minutes (manual rollback via ArgoCD git revert)
                       → ELITE tier
```

### How DORA Connects to Sprint Retrospectives

```
Sprint 47 Retrospective — DORA Review:

Deployment Frequency: 4 prod deploys this sprint ✅
  "Good — one deploy per completed story."

Lead Time: Average 2.5 days this sprint ✅
  "VIP routing took 5 days (complex), IVR fix took 1 day (simple). Average is fine."

Change Failure Rate: 1 rollback out of 4 deploys = 25% ❌
  "The routing memory leak caused a rollback. Root cause addressed."
  "Action: Extend load testing duration (IPCC-251)"

MTTR: 2 minutes (auto-rollback caught it) ✅
  "Canary worked perfectly — detected the issue at 5% traffic, rolled back automatically."
```

### How to Track DORA Metrics

```
Data sources:
  Deployment Frequency → ArgoCD sync events / Jenkins deploy job count
  Lead Time → Jira (story created) → Git (first commit) → ArgoCD (deployed)
  Change Failure Rate → Rollback events / total deploys
  MTTR → PagerDuty (alert fired) → ArgoCD (rollback complete)

Dashboard:
  Grafana dashboard showing all 4 metrics per sprint
  Reviewed in every retrospective
  Trend over time (are we improving?)
```

### Interview Answer: "How do you measure your team's performance?"

> "We track DORA metrics — Deployment Frequency, Lead Time, Change Failure Rate, and MTTR. We review them every sprint retrospective in Grafana. Currently our Change Failure Rate is ~3% thanks to canary deployments and our MTTR is under 5 minutes because of automated rollback. When a metric degrades, it becomes a sprint action item. For example, when our lead time spiked to 4 days last month, we investigated and found PR reviews were taking too long — we set a 2-hour review SLA and it dropped back to 2 days."

---

## Part 15: Putting It All Together — The Complete Mental Model

### One Picture to Remember Everything

```
┌─────────────────────────── THE INFINITE LOOP ───────────────────────────┐
│                                                                          │
│   ┌─── SCRUM (Plan) ──┐    ┌─── GIT (Build) ──┐    ┌── DevOps (Ship) ─┐│
│   │                    │    │                   │    │                   ││
│   │  Sprint Planning   │───▶│  Feature Branch   │───▶│  CI Pipeline      ││
│   │  User Stories      │    │  Commits          │    │  Security Scans   ││
│   │  Story Points      │    │  PR/MR Review     │    │  Docker Build     ││
│   │  Sprint Backlog    │    │  Squash Merge     │    │  CD (ArgoCD)      ││
│   │                    │    │                   │    │  Canary Deploy    ││
│   └────────────────────┘    └───────────────────┘    └───────────────────┘│
│           ▲                                                    │          │
│           │                                                    ▼          │
│   ┌─── FEEDBACK ──────────────────────────────────── MONITOR ──────┐     │
│   │                                                                 │     │
│   │  Sprint Review (demo deployed features)                         │     │
│   │  Retrospective (improve process + pipeline)                     │     │
│   │  DORA Metrics (measure performance)                             │     │
│   │  Prometheus/Grafana (real-time health)                          │     │
│   │  Incidents → Postmortems → Action Items → Next Sprint           │     │
│   │                                                                 │     │
│   └─────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### The Key Relationships to Remember

```
Scrum gives structure    → "What to build, when, by whom"
Git gives mechanics      → "How to write code together safely"
DevOps gives speed       → "How to ship it automatically and reliably"
Feature Flags give safety → "How to deploy without risk"
DORA Metrics give insight → "How to continuously improve"
Code Review gives quality → "How to maintain standards"
Monitoring gives feedback → "What to fix/improve next"
```

### Quick Interview Cheat Sheet

| Question | Key Points |
|---|---|
| "Your branching strategy?" | Feature branches, Jira ticket ID in name, 1-3 day lifespan, squash merge via PR/MR |
| "Merge or rebase?" | Merge to update branches, squash merge for PRs. Never rebase shared branches. |
| "How does Agile fit DevOps?" | Fully embedded — DevOps tasks in sprint backlog, demo deployed features, retro improves pipeline |
| "Feature flags?" | Deploy incomplete code behind flags, merge daily, flip when ready. Enables trunk-based. |
| "Release process?" | SemVer tags, auto-generated changelog from conventional commits, build once promote everywhere |
| "Environment strategy?" | DEV (auto) → STAGING (auto+gate) → PROD (manual+canary). Same image promoted. |
| "Code review SLA?" | 2 hours response, 2 approvals, max 400 lines per PR, CI must pass first |
| "Multiple teams?" | CODEOWNERS, contract testing (Pact), Scrum of Scrums, independent deployability |
| "On-call in sprints?" | Reduced capacity, budget 15-20% for unplanned, track planned vs unplanned ratio |
| "How you measure?" | DORA metrics in Grafana, reviewed every retro, degradation becomes sprint action item |

---

*Document created: July 2026*
*Last updated: July 31, 2026*
*Part of: DevOps Interview Prep Portfolio*
*Related docs: Project-Documentation.md, DevSecOps-Pipeline-Interview-QA.md, Kubernetes-Environments-Comparison.md*
