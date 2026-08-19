# GIT FLOW + DevSecOps Pipeline (With All Environments)

## Environments: DEV → TEST → UAT → PROD

## Branch to Environment Mapping

| Branch | Pipeline Phases | Deploys To |
|--------|----------------|------------|
| `feature/*` | Phase 1 (Build + Unit Test) | Nowhere (just validates) |
| `develop` | Phase 1 + Phase 2 (SAST, SCA, Integration Tests) | DEV environment |
| `develop` (promotion) | Auto/manual trigger after DEV passes | TEST environment (QA testers do testing here) |
| `release/*` | Phase 1 + Phase 2 + Phase 3 (DAST, Perf, Compliance) | UAT environment (Business/stakeholder sign-off) |
| `main` | Phase 1 + Phase 2 + Full Phase 3 (Full security gate) | PRODUCTION (canary/blue-green) |
| `hotfix/*` | Phase 1 + Phase 2 + Fast Phase 3 (Critical fix fast-track) | PRODUCTION (expedited) |

## Visual Flow

```
Developer Work         Merge/Promote           Environments

feature/login ──PR──→ develop ──auto──→  [DEV] ──promote──→ [TEST]
feature/auth  ──PR──┘                      │                   │
                                           │    Testers OK?     │
                                           │        ↓           │
                            Cut release/* ←┘    YES             │
                                  │                             │
                                  ↓                             │
                               [UAT]  ← Business validates here │
                                  │                             │
                            UAT sign-off?                       │
                                  │ YES                         │
                                  ↓                             │
                            Merge to main                       │
                                  │                             │
                                  ↓                             │
                              [PROD]  (canary → full rollout)   │

── HOTFIX PATH (emergency) ──
hotfix/* ──→ fast pipeline ──→ [PROD] ──→ backmerge to develop
```

## Explanation

1. **feature/* → Nowhere** — Developer writes code, pipeline just checks "does it compile and pass unit tests?" No deployment.
2. **develop → DEV** — When feature is merged to develop, it auto-deploys to DEV. Developers verify integration here.
3. **develop → TEST (promotion)** — Once DEV is stable, you promote (same artifact) to TEST. QA testers do manual/automated testing. This is NOT a new branch — it's an artifact promotion with a gate.
4. **release/* → UAT** — When testers give thumbs up, you cut a release branch. This triggers full security scans and deploys to UAT. Business stakeholders validate here.
5. **main → PROD** — After UAT sign-off, merge release to main. Full pipeline runs, deploys to production with canary/blue-green strategy.
6. **hotfix/* → PROD (fast)** — Critical bug in prod? Branch from main, fix it, run abbreviated pipeline, deploy directly. Then backmerge to develop.

**Key point:** DEV → TEST is NOT a branch change. It's the same artifact being promoted through environments with gates (test pass, manual approval). The branch only changes when you go from `develop` to `release/*` to `main`.

---

# Bug Found in TEST - Fix Flow

## Flow

```
Tester finds bug in TEST
      │
      ↓
Raises a ticket (e.g., JIRA-456: "Login button not working")
      │
      ↓
Developer creates branch from `develop`:
      bugfix/JIRA-456-login-button-fix
            │
            ↓
      Developer fixes & commits
            │
            ↓
      Phase 1 pipeline runs (build + unit test)
            │
            ↓
      PR → merge to `develop`
            │
            ↓
      Auto-deploy to DEV (developer verifies)
            │
            ↓
      Promote to TEST (tester re-verifies the fix)
            │
            ↓
      Tester confirms fix ✓ → continue normal flow
```

## Explanation

1. Tester finds bug → raises ticket
2. Developer creates **`bugfix/JIRA-456-description`** branch from `develop`
3. Fixes the code, pushes, pipeline validates
4. Merges back to `develop` via PR
5. It goes through the same flow: DEV → TEST
6. Tester re-tests and confirms fix

## Branch Naming Convention Based on Where Bug is Found

| Scenario | Branch Name | Branch From |
|----------|-------------|-------------|
| New work | `feature/JIRA-123-desc` | `develop` |
| Bug found in TEST | `bugfix/JIRA-456-desc` | `develop` |
| Bug found in UAT | `bugfix/JIRA-789-desc` | `release/*` |
| Bug found in PROD | `hotfix/JIRA-999-desc` | `main` |

**Key point:** A `bugfix/*` follows the exact same flow as a `feature/*`. The only difference is the naming convention — so your team knows from the branch name that it's a fix, not new work. The branch name changes based on **where the bug was found**, because that decides where you branch from and how fast it needs to reach that environment.
