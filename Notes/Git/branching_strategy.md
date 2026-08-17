# Git Branching Strategies & DevSecOps Pipeline

---

## Part 1: DevSecOps Pipeline with Git Flow — How Each Branch Triggers What

This shows exactly how an 18-stage DevSecOps pipeline maps to each branch in Git Flow.

---

### The Complete Picture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                        GIT FLOW + DevSecOps PIPELINE                        │
│                                                                             │
│  BRANCHES:          WHAT PIPELINE RUNS:              DEPLOYS TO:            │
│                                                                             │
│  feature/*    →     Phase 1 (Build)              →   Nowhere               │
│  develop      →     Phase 1 + Phase 2            →   DEV environment       │
│  release/*    →     Phase 1 + Phase 2 + Partial 3→   STAGING environment   │
│  main         →     Phase 1 + Phase 2 + Full 3   →   PRODUCTION (canary)   │
│  hotfix/*     →     Phase 1 + Phase 2 + Fast 3   →   PRODUCTION (fast)     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### The 18 Stages in 3 Phases

```
PHASE 1 (BUILD):
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ Secret  │→│  Unit   │→│  SCA    │→│  SAST   │→│ Quality │
│  Scan   │ │  Tests  │ │ (Snyk)  │ │(Sonar)  │ │  Gate   │
└─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘

PHASE 2 (PACKAGE):
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ Docker  │→│  Trivy  │→│  Push   │→│ Cosign  │→│   S3    │
│  Build  │ │  Scan   │ │ (clean) │ │  Sign   │ │ Reports │
└─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘

PHASE 3 (DEPLOY):
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ Deploy  │→│ DAST +  │→│Promote  │→│ Manual  │→│ Deploy  │
│  DEV    │ │ Smoke   │ │STAGING  │ │Approval │ │  PROD   │
└─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘
                                                       │
                                                       ▼
                                        Canary: 5%→20%→50%→80%→100%
                                        Prometheus checks at each step
                                        FAIL → Auto-rollback
```

---

### Scenario: Adding a "Wishlist" Feature — Full Lifecycle

---

#### STEP 1: Developer creates feature branch and pushes code

```bash
git checkout develop
git checkout -b feature/wishlist
# write code...
git push origin feature/wishlist
```

**What triggers:**

```
feature/wishlist push
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│ PIPELINE FOR feature/* — PHASE 1 ONLY (Build Phase)         │
│                                                              │
│ ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌──────────┐ │
│ │ 1. Secret │→ │ 2. Unit   │→ │ 3. SCA    │→ │ 4. SAST  │ │
│ │    Scan   │  │   Tests   │  │   (Snyk)  │  │  (Sonar) │ │
│ └───────────┘  └───────────┘  └───────────┘  └──────────┘ │
│                                                      │       │
│                                                      ▼       │
│                                              ┌──────────┐   │
│                                              │ 5.Quality│   │
│                                              │   Gate   │   │
│                                              └──────────┘   │
│                                                              │
│ ✅ Pass → Ready for code review                             │
│ ❌ Fail → Developer fixes and pushes again                  │
│                                                              │
│ 🚫 NO Docker build                                          │
│ 🚫 NO deployment                                            │
│ 🚫 NO image scanning                                        │
└─────────────────────────────────────────────────────────────┘
```

**Why only Phase 1?**
- It's just a feature branch — no need to build a Docker image yet
- We only care: Is the code secure? Does it pass tests? Is it quality?
- Fast feedback (2-5 minutes) while developer is still working

**Result shown in Merge Request:**

```
┌─────────────────────────────────────────┐
│ Merge Request: feature/wishlist → develop│
│                                          │
│ Pipeline: ✅ Passed                      │
│   ✓ Secret Scan — no secrets found      │
│   ✓ Unit Tests — 156/156 passed         │
│   ✓ SCA — no critical vulnerabilities   │
│   ✓ SAST — no security issues           │
│   ✓ Quality Gate — passed               │
│                                          │
│ Reviewers: @teammate                     │
│ [Merge] [Close]                          │
└─────────────────────────────────────────┘
```

---

#### STEP 2: Feature merged into develop

Teammate reviews, approves, and merges.

```
feature/wishlist merged → develop
```

**What triggers:**

```
develop branch updated
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PIPELINE FOR develop — PHASE 1 + PHASE 2 + DEPLOY TO DEV               │
│                                                                          │
│ PHASE 1 (Build):                                                        │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐               │
│ │Secret  │→│ Unit   │→│  SCA   │→│  SAST  │→│Quality │               │
│ │Scan    │ │ Tests  │ │(Snyk)  │ │(Sonar) │ │ Gate   │               │
│ └────────┘ └────────┘ └────────┘ └────────┘ └───┬────┘               │
│                                                   │ ✅                  │
│ PHASE 2 (Package):                                ▼                     │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐               │
│ │Docker  │→│ Trivy  │→│Push to │→│Cosign  │→│Upload  │               │
│ │Build   │ │ Scan   │ │Registry│ │ Sign   │ │Reports │               │
│ └────────┘ └────────┘ └────────┘ └────────┘ └───┬────┘               │
│                                                   │ ✅                  │
│ PHASE 3 (Partial):                                ▼                     │
│ ┌─────────────────┐ ┌─────────────────┐                               │
│ │ Deploy to DEV   │→│ DAST + Smoke    │                               │
│ │ dev.myshop.com  │ │ Tests on DEV    │                               │
│ └─────────────────┘ └─────────────────┘                               │
│                                                                          │
│ 🚫 NO staging deployment                                               │
│ 🚫 NO manual approval                                                  │
│ 🚫 NO production deployment                                            │
└─────────────────────────────────────────────────────────────────────────┘
```

**Why Phase 1 + 2 + partial 3?**
- Code is now integrated with other features — need full security scan
- Build the Docker image now so it's ready to promote later
- Deploy to DEV so team can see all features working together
- Run DAST against DEV to find runtime security issues early

**Result:**
- `dev.myshop.com` now has the wishlist feature
- A signed Docker image `myshop:build-247` exists in the registry
- Security reports uploaded to S3

---

#### STEP 3: Release branch created

You've accumulated several features in `develop`. Time to release.

```bash
git checkout develop
git checkout -b release/2.0
git push origin release/2.0
```

**What triggers:**

```
release/2.0 branch created
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PIPELINE FOR release/* — PHASE 1 + PHASE 2 + DEPLOY TO STAGING         │
│                                                                          │
│ PHASE 1 (Build):                                                        │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐               │
│ │Secret  │→│ Unit   │→│  SCA   │→│  SAST  │→│Quality │               │
│ │Scan    │ │ Tests  │ │(Snyk)  │ │(Sonar) │ │ Gate   │               │
│ └────────┘ └────────┘ └────────┘ └────────┘ └───┬────┘               │
│                                                   │ ✅                  │
│ PHASE 2 (Package):                                ▼                     │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐               │
│ │Docker  │→│ Trivy  │→│Push to │→│Cosign  │→│Upload  │               │
│ │Build   │ │ Scan   │ │Registry│ │ Sign   │ │Reports │               │
│ └────────┘ └────────┘ └────────┘ └────────┘ └───┬────┘               │
│                                                   │ ✅                  │
│ PHASE 3 (Up to Staging):                         ▼                     │
│ ┌────────────┐ ┌──────────────┐ ┌─────────────────────┐               │
│ │ Deploy to  │→│ DAST + Smoke │→│ Promote to STAGING  │               │
│ │    DEV     │ │  Tests       │ │ staging.myshop.com  │               │
│ └────────────┘ └──────────────┘ └─────────────────────┘               │
│                                                                          │
│ 🚫 NO manual approval (QA needs to test first)                         │
│ 🚫 NO production deployment                                            │
│                                                                          │
│ ➡️  QA team starts testing at staging.myshop.com                        │
└─────────────────────────────────────────────────────────────────────────┘
```

**Why stop at staging?**
- QA team needs to manually test the release candidate
- Business team needs to verify features look correct
- This is the "prove it works" environment before going live

**If QA finds a bug:**

```bash
git checkout release/2.0
# fix the bug
git commit -m "fix: wishlist crash on mobile"
git push origin release/2.0
```

Same pipeline runs again → re-deploys to staging → QA tests again

---

#### STEP 4: Merge release to main (Go to Production)

QA approves. You merge `release/2.0` → `main`.

```bash
git checkout main
git merge release/2.0
git push origin main
```

**What triggers:**

```
main branch updated
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PIPELINE FOR main — FULL PIPELINE (ALL 18 STAGES)                       │
│                                                                          │
│ PHASE 1 (Build):                                                        │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐               │
│ │Secret  │→│ Unit   │→│  SCA   │→│  SAST  │→│Quality │               │
│ │Scan    │ │ Tests  │ │(Snyk)  │ │(Sonar) │ │ Gate   │               │
│ └────────┘ └────────┘ └────────┘ └────────┘ └───┬────┘               │
│                                                   │ ✅                  │
│ PHASE 2 (Package):                                ▼                     │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐               │
│ │Docker  │→│ Trivy  │→│Push to │→│Cosign  │→│Upload  │               │
│ │Build   │ │ Scan   │ │Registry│ │ Sign   │ │Reports │               │
│ └────────┘ └────────┘ └────────┘ └────────┘ └───┬────┘               │
│                                                   │ ✅                  │
│ PHASE 3 (Full Deploy):                           ▼                     │
│ ┌────────────┐ ┌──────────────┐ ┌────────────────┐ ┌───────────────┐ │
│ │ Deploy to  │→│ DAST + Smoke │→│ Promote to     │→│   MANUAL      │ │
│ │    DEV     │ │  Tests       │ │   STAGING      │ │  APPROVAL     │ │
│ └────────────┘ └──────────────┘ └────────────────┘ └───────┬───────┘ │
│                                                             │ 👤       │
│                                                             ▼ Approved │
│ ┌─────────────────────────────────────────────────────────────────┐   │
│ │                    CANARY DEPLOYMENT                              │   │
│ │                                                                   │   │
│ │   5% ──monitor──► 20% ──monitor──► 50% ──monitor──►             │   │
│ │                                                                   │   │
│ │   80% ──monitor──► 100% 🎉                                       │   │
│ │                                                                   │   │
│ │   Prometheus checks at EVERY step:                               │   │
│ │     • Error rate < 1%                                            │   │
│ │     • Latency p99 < 500ms                                       │   │
│ │     • No OOM kills                                               │   │
│ │                                                                   │   │
│ │   ANY check fails → AUTO-ROLLBACK to previous version            │   │
│ └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**This is the only branch that:**
- Has a manual approval gate (human says "go")
- Does canary deployment (gradual rollout)
- Touches production traffic

---

#### STEP 5: Hotfix (Emergency in production)

Production is crashing. Can't wait for the full flow.

```bash
git checkout main
git checkout -b hotfix/payment-crash
# fix the bug
git push origin hotfix/payment-crash
```

**What triggers:**

```
hotfix/* push
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PIPELINE FOR hotfix/* — FAST TRACK TO PRODUCTION                        │
│                                                                          │
│ PHASE 1 (Build — same as always):                                       │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐               │
│ │Secret  │→│ Unit   │→│  SCA   │→│  SAST  │→│Quality │               │
│ │Scan    │ │ Tests  │ │(Snyk)  │ │(Sonar) │ │ Gate   │               │
│ └────────┘ └────────┘ └────────┘ └────────┘ └───┬────┘               │
│                                                   │ ✅                  │
│ PHASE 2 (Package — same as always):              ▼                     │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐               │
│ │Docker  │→│ Trivy  │→│Push to │→│Cosign  │→│Upload  │               │
│ │Build   │ │ Scan   │ │Registry│ │ Sign   │ │Reports │               │
│ └────────┘ └────────┘ └────────┘ └────────┘ └───┬────┘               │
│                                                   │ ✅                  │
│ PHASE 3 (SHORTENED — Skip staging, fast canary): ▼                     │
│ ┌────────────┐ ┌──────────────┐ ┌───────────────┐ ┌────────────────┐ │
│ │ Deploy to  │→│ DAST + Smoke │→│    MANUAL     │→│ FAST CANARY    │ │
│ │    DEV     │ │ (quick only) │ │   APPROVAL    │ │ to PRODUCTION  │ │
│ └────────────┘ └──────────────┘ │ (auto-approve │ │                │ │
│                                  │  after 15min) │ │ 20%→50%→100%  │ │
│                                  └───────────────┘ │ (skip 5%)     │ │
│                                                     └────────────────┘ │
│                                                                          │
│ 🚫 SKIPS staging (too slow for emergencies)                            │
│ ⚡ Faster canary (fewer steps, shorter wait)                            │
│ ⏰ Auto-approve after 15 minutes if no rejection                       │
└─────────────────────────────────────────────────────────────────────────┘
```

**Why is hotfix different?**
- Production is broken NOW — users are affected
- We still run ALL security scans (never skip security)
- But we skip staging and speed up the canary
- The trade-off: slightly more risk for much faster recovery

**After hotfix is deployed, you merge it back:**

```bash
git checkout develop
git merge hotfix/payment-crash    # so develop has the fix too

git checkout main
git merge hotfix/payment-crash    # so main has the fix too
```

---

### Visual Timeline: Complete Feature Lifecycle

```
TIME ──────────────────────────────────────────────────────────────────►

Day 1          Day 2          Day 3         Day 4        Day 5
  │              │              │             │            │
  ▼              ▼              ▼             ▼            ▼

feature/wishlist created
  │
  ├─ push → [Phase 1] ✅
  ├─ push → [Phase 1] ✅
  ├─ push → [Phase 1] ❌ (SAST found XSS)
  ├─ fix  → [Phase 1] ✅
  │
  ▼
Merge to develop
  │
  ├─ [Phase 1 + 2 + Deploy DEV] ✅
  │   └─ dev.myshop.com updated
  │
  ▼
Create release/2.0
  │
  ├─ [Phase 1 + 2 + Deploy STAGING] ✅
  │   └─ staging.myshop.com updated
  │   └─ QA testing...
  ├─ QA bug found → fix → push → [re-deploy staging] ✅
  │   └─ QA re-tests... approved!
  │
  ▼
Merge to main
  │
  ├─ [FULL PIPELINE - All 18 stages]
  │   ├─ Phase 1 ✅
  │   ├─ Phase 2 ✅
  │   ├─ Deploy DEV ✅
  │   ├─ DAST ✅
  │   ├─ Promote Staging ✅
  │   ├─ Manual Approval ✅ (release manager clicks approve)
  │   └─ Canary: 5% ✅ → 20% ✅ → 50% ✅ → 80% ✅ → 100% 🎉
  │
  ▼
www.myshop.com has wishlist feature! 🎉
```

---

### Summary Table

| Branch | Phase 1 | Phase 2 | DEV | DAST | STAGING | Approval | PROD Canary |
|--------|---------|---------|-----|------|---------|----------|-------------|
| `feature/*` | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| `develop` | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| `release/*` | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| `main` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ (slow) |
| `hotfix/*` | ✅ | ✅ | ✅ | ⚡quick | ❌ | ⚡auto | ✅ (fast) |

**The rule:** The closer you get to production, the more gates you pass through. Security scans run EVERYWHERE — deployment gates increase as you move right.

---
---

## Part 2: Other Branching Strategies (In Brief)

---

### 1. Trunk-Based Development

**Definition:** Everyone commits directly to a single main branch (`main` or `trunk`). No long-lived branches. Features that aren't ready are hidden behind feature flags.

**Visual Flow:**

```
main: ──A──B──C──D──E──F──G──H──I──
         │     │     │     │     │
       dev1  dev2  dev1  dev3  dev2
      (small commits, multiple times a day)
```

**CI/CD with it:**

```
Every push to main
       │
       ▼
┌─────────────────────────────────┐
│ Build → Test → Deploy to PROD   │
│ (fully automated, no gates)     │
└─────────────────────────────────┘
```

**Key points:**
- Simplest strategy — one branch, no merging pain
- Requires feature flags to hide incomplete work
- Requires high test coverage and mature CI/CD
- Used by Google, Facebook, Netflix
- Best for: Small teams, high-trust teams, deploying many times per day

---

### 2. GitHub Flow

**Definition:** Branch off `main` for a feature → work on it → open a Pull Request → review → merge back to `main`. Main is always deployable.

**Visual Flow:**

```
main:    ──A──────B──────────C──────D──────E──
               \              /        \     /
feature-1:      └──x──y──z──┘          \   /
                                         \ /
feature-2:                                └──a──b──┘
```

**CI/CD with it:**

```
Push to feature branch          Merge to main
       │                              │
       ▼                              ▼
┌──────────────────┐          ┌──────────────────┐
│ CI: Build + Test │          │ CD: Deploy PROD  │
│ (validates PR)   │          │ (auto on merge)  │
└──────────────────┘          └──────────────────┘
```

**Key points:**
- Only 2 branch types: `main` and short-lived feature branches
- PR = the quality gate (code review + CI passing)
- `main` is always in a deployable state
- No release branches, no develop branch
- Best for: Most web teams, SaaS products, daily deployments

---

### 3. Git Flow (Detailed above in Part 1)

**Definition:** Structured approach with designated branch types: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`. Each branch type has a specific role in the release process.

**Visual Flow:**

```
main:      ──A─────────────────────M──────────H──
              \                   /            /
release:       \        ──R1──R2─┘            /
                \      /                     /
develop:   ──────B──C──D──E──────────────F──/───
                  \      /                  /
feature:           └──x─┘                  /
                                          /
hotfix:                                 ──h──
```

**Key points:**
- Most structured strategy — clear roles for each branch
- Good for teams with scheduled releases
- Overhead can be too much for small/fast teams
- Best for: Larger teams, regulated industries, bi-weekly/monthly releases

---

### 4. Release Branching

**Definition:** When you're ready to release, you cut a branch like `release/2.3`. That branch gets stabilized while `main` keeps moving forward. Multiple release branches can coexist to support different versions.

**Visual Flow:**

```
main:         ──A──B──C──D──E──F──G──H──I──
                      \           \
release/1.0:           └──fix──fix──fix (support v1.x users)
                                   \
release/2.0:                        └──fix──fix (support v2.x users)
```

**CI/CD with it:**

```
Push to main               Push to release/1.0         Push to release/2.0
     │                           │                           │
     ▼                           ▼                           ▼
┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│ Build + Test │          │ Build + Test │          │ Build + Test │
│ Deploy to    │          │ Ship v1.0.x  │          │ Ship v2.0.x  │
│ DEV/nightly  │          │ to users     │          │ to users     │
└──────────────┘          └──────────────┘          └──────────────┘
```

**Key points:**
- Multiple versions in production simultaneously
- Bug fixes go to specific release branches (backporting)
- Main branch keeps moving forward with new features
- Each release branch has its own CI/CD pipeline and versioning
- Best for: Mobile apps, desktop software, libraries, APIs with versioned clients

---

### 5. Environment Branching

**Definition:** Branches map directly to environments: `dev`, `staging`, `production`. Code is promoted by merging from one branch to the next.

**Visual Flow:**

```
dev:         ──A──B──C──D──E──F──G──
                      │        │
staging:     ─────────C──D─────E────
                         │     │
production:  ────────────D─────E────
```

**CI/CD with it:**

```
Push to dev              Merge dev→staging         Merge staging→production
     │                        │                          │
     ▼                        ▼                          ▼
┌────────────────┐    ┌────────────────┐        ┌────────────────┐
│ Build + Test   │    │ Build + Test   │        │ Build + Test   │
│ Deploy to      │──► │ Deploy to      │──►     │ Deploy to      │
│ dev.myapp.com  │    │ stg.myapp.com  │        │ www.myapp.com  │
└────────────────┘    └────────────────┘        └────────────────┘
```

**Key points:**
- Simple to understand — branch name = environment
- Branches drift apart over time → painful merges
- What you test in staging might differ from what goes to production
- Generally considered an **anti-pattern** today
- Best for: Legacy setups, simple internal tools (avoid for new projects)

---

### 6. GitLab Flow (Bonus — combines the best of others)

**Definition:** A middle ground between GitHub Flow (too simple for some) and Git Flow (too complex for some). Uses `main` as the source of truth + environment branches OR release branches.

**Option A: With environment branches**

```
main:        ──A──B──C──D──E──     (development happens here)
                      │     │
pre-prod:    ─────────C─────E──    (cherry-pick/merge what's ready)
                            │
production:  ───────────────E──    (merge from pre-prod when verified)
```

**Option B: With release branches**

```
main:         ──A──B──C──D──E──F──G──
                   \        \
release/1.0:        └──fix   \
                              \
release/2.0:                   └──fix
```

**CI/CD with it:**

```
Push to main              Merge to pre-prod         Merge to production
     │                         │                          │
     ▼                         ▼                          ▼
┌────────────────┐     ┌────────────────┐        ┌────────────────┐
│ Build + Test   │     │ Deploy to      │        │ Deploy to      │
│ (CI only)      │     │ pre-prod env   │        │ PRODUCTION     │
└────────────────┘     └────────────────┘        └────────────────┘
```

**Key points:**
- Simpler than Git Flow — no `develop` branch
- `main` is the single source of truth
- Merges flow downstream (main → pre-prod → production)
- Unlike environment branching, main is where development happens (no drift)
- Best for: Teams wanting structure without Git Flow's complexity

---

## Quick Comparison: All Strategies

| Strategy | Complexity | Branches | Best For | Deploy Frequency |
|----------|-----------|----------|----------|-----------------|
| Trunk-Based | Very Low | 1 (main) | Small teams, high trust | Many times/day |
| GitHub Flow | Low | main + features | Most web teams | Daily |
| Git Flow | High | main, develop, feature, release, hotfix | Scheduled releases | Weekly/monthly |
| Release Branching | Medium | main + release/* | Multiple live versions | Per version |
| Environment Branching | Medium | dev, staging, prod | Legacy orgs (avoid) | Varies |
| GitLab Flow | Medium | main + env or release branches | Middle-ground teams | Daily/weekly |

---

## How to Choose

```
                            How often do you deploy?
                                     │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
              Many times/day      Daily/Weekly       Monthly+
                    │                 │                 │
                    ▼                 ▼                 ▼
             Trunk-Based        GitHub Flow         Git Flow
                                     │
                                     │
                         Do you have multiple
                         versions in production?
                                     │
                              ┌──────┴──────┐
                              │             │
                             Yes           No
                              │             │
                              ▼             ▼
                      Release Branching  GitHub Flow
                                        or GitLab Flow
```

---

## Key Principles (Apply to ALL strategies)

1. **Shift Left** — Find problems as early as possible (in Phase 1, not in production)
2. **Never rebuild** — Same image moves through all environments
3. **Sign everything** — Prove the image came from your pipeline
4. **Gate progression** — Code only moves forward if it passes checks
5. **Deploy gradually** — Canary/blue-green protects users from bad releases
6. **Auto-rollback** — Machines detect problems faster than humans
7. **Audit everything** — Reports stored for compliance (SOC2, ISO 27001, HIPAA)
