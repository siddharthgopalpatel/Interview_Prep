# Git Branching — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 16, 36

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 16: Git & Branching Strategy

---

### Q: Can you explain your branching strategy and how you handle releases?

**Project Reference:** P1 (DevSecOps Pipeline — branch-based deployment)

**Answer:**

> "We use **Git Flow variant** tailored for CI/CD:
>
> ```
> main          ← production (always deployable, protected)
> └── release/* ← stabilization for next release (staging)
>     └── develop    ← integration branch (dev environment)
>         └── feature/* ← individual developer work
> ```
>
> **Flow:**
> 1. Developer creates `feature/add-payment` from `develop`
> 2. PR → code review → merge to `develop` → auto-deploy to Dev
> 3. When ready for release: create `release/v2.1` from `develop`
> 4. Release branch → auto-deploy to Staging. Only bug fixes allowed here.
> 5. QA signs off → merge `release/v2.1` to `main` → triggers production pipeline (canary deployment)
> 6. Tag `v2.1.0` on main for tracking
>
> **Hotfix path:** `hotfix/*` branch from `main` → fix → PR to `main` directly → emergency pipeline → also merge back to `develop` to keep branches in sync.
>
> **Key rules:**
> - `main` is protected — no direct pushes, requires PR + approval
> - Release branches are short-lived (1-2 weeks max)
> - Feature branches are even shorter (2-3 days ideally)"

---

### Q: Three team members are working on the same file with different changes. How do you merge this into a higher-level branch, and what issues might arise?

**Project Reference:** P1 (team collaboration)

**Answer:**

> "This is a **merge conflict** scenario. Here's how it plays out:
>
> **Process:**
> 1. First developer merges their PR — no issues (clean merge)
> 2. Second developer's PR now shows conflict — Git can't auto-merge because the same lines were modified
> 3. Second developer must: `git pull origin develop` → resolve conflicts manually → push → re-request review
> 4. Third developer — same process, conflicts with both previous changes
>
> **Issues that arise:**
> - **Merge conflicts** — Most common. Same file, same lines. Must be resolved manually.
> - **Semantic conflicts** — No Git conflict, but code is logically incompatible. Tests catch this (one person renamed a function, another person calls the old name).
> - **Lost changes** — If someone resolves conflict carelessly and accepts 'theirs' without reading.
>
> **How we minimize this:**
> - **Small, short-lived branches** — Merge daily, not weekly. Less divergence = fewer conflicts.
> - **File ownership** — In microservices, each team owns their service files. Rarely do 3 people edit the same file.
> - **Rebase before merge** — `git pull --rebase origin develop` keeps history clean and surfaces conflicts early.
> - **CI runs on PR** — After conflict resolution, pipeline re-runs to verify nothing broke."

---

### Q: Did you use CLI-based or UI-based Git?

**Project Reference:** General — daily workflow

**Answer:**

> "**Primarily CLI.** For daily work — `git add`, `commit`, `push`, `pull`, `rebase`, `log`, `diff` — all CLI. It's faster and scriptable.
>
> **UI for:** Pull request reviews (GitHub/GitLab web UI), merge conflict visualization (VS Code GitLens), and browsing history graphically.
>
> **In CI/CD:** Everything is CLI — Jenkins pipeline runs `git rev-parse --short HEAD` for image tags, `sed` to update GitOps repo, `git push` to trigger ArgoCD. No UI possible in automation.
>
> At 12 years experience, CLI is muscle memory. But I don't gatekeep — junior devs using VS Code Git integration or GitKraken is fine as long as the commits are clean."

---

### Q: What is the first command you use to start working on a repository, and what command to push your finished work?

**Project Reference:** General — Git fundamentals

**Answer:**

> "**Start:**
> ```bash
> git clone git@github.com:org/repo.git    # First time — get the repo
> git checkout -b feature/my-feature       # Create feature branch
> ```
>
> Or if repo already cloned:
> ```bash
> git pull origin develop                   # Get latest changes
> git checkout -b feature/my-feature       # Branch from latest
> ```
>
> **Finish:**
> ```bash
> git add .                                 # Stage changes (or specific files)
> git commit -m \"feat: add payment service\"  # Commit with conventional message
> git push -u origin feature/my-feature    # Push to remote, set upstream
> ```
> Then create a PR via GitHub/GitLab UI.
>
> **Key point:** I never push to `main` or `develop` directly. Always feature branch → PR → review → merge."

---

### Q: What is the difference between git pull and git fetch?

**Project Reference:** General — Git fundamentals

**Answer:**

> "**`git fetch`** — Downloads changes from remote but does NOT apply them to your working branch. Your local code stays untouched. It just updates your remote tracking references (`origin/main`, `origin/develop`).
>
> **`git pull`** — Does `git fetch` + `git merge` (or `git rebase` if configured). Downloads AND applies changes to your current branch immediately.
>
> ```bash
> git fetch origin        # Safe — just downloads. You can review before merging.
> git pull origin main    # Downloads AND merges into your branch immediately.
> ```
>
> **When I use which:**
> - `git fetch` — When I want to check what changed on remote before merging. `git log origin/main..HEAD` shows what's different.
> - `git pull --rebase` — My daily workflow. Rebase my local commits on top of latest remote. Keeps history linear, avoids unnecessary merge commits.
>
> **Key insight:** `git pull` can cause unexpected merge conflicts if you have uncommitted work. `git fetch` is always safe."

---

### Q: What is the difference between a central repository and a fork?

**Project Reference:** General — Git workflow

**Answer:**

> "**Central repository** — One shared repo that everyone pushes to directly (via branches). All developers have write access. This is what we use internally at Ericsson — everyone works on feature branches within the same repo.
>
> **Fork** — A complete copy of the repo in YOUR account. You push to your fork, then create a PR from your fork to the original repo. You DON'T have write access to the original.
>
> | Aspect | Central Repo | Fork |
> |---|---|---|
> | **Access** | All team members have write access | Only maintainers have write to original |
> | **Branches** | Feature branches in same repo | Feature branches in your copy |
> | **Use case** | Internal team collaboration | Open source, untrusted contributors |
> | **PR flow** | branch → main (same repo) | fork/branch → upstream/main (cross-repo) |
>
> **When to use which:**
> - Internal team (trusted devs) → Central repo with branch protection
> - Open source / external contributors → Fork model (don't give strangers write access to your repo)
>
> We use central repo internally. Fork model only when contributing to open-source projects (e.g., submitting a fix to a Helm chart upstream)."

---

---
---

# SECTION 36: Git (Advanced)

---

### Q: Can you think of a scenario where Git can be used in the continuous feedback loop?

**Project Reference:** P1 (DevSecOps — GitOps), P3 (ArgoCD)

**Answer:**

> "Yes — **GitOps IS the continuous feedback loop in code form.**
>
> **Scenario: Production auto-rollback updates Git:**
>
> 1. Developer merges code → Pipeline builds image → Updates Git repo (Helm values) → ArgoCD deploys canary
> 2. Prometheus detects error rate spike → Argo Rollouts auto-rolls back → Reverts the Git commit in the GitOps repo
> 3. Developer gets notification: 'Your deployment was rolled back. Commit X reverted. Error rate exceeded threshold.'
> 4. Git log now shows: the attempted deployment AND the rollback — full audit trail
> 5. Developer opens the failed commit, sees what changed, fixes the bug, pushes again
>
> **Git is the feedback mechanism:**
> - Deployment state = Git commit. Rollback = Git revert. History = Git log.
> - Every production change is traceable to a Git commit
> - Feedback from production (monitoring) flows BACK into Git (reverted commit)
>
> **Another scenario: Infrastructure drift feedback:**
> - Scheduled `terraform plan` detects drift → creates a Git issue automatically: 'Drift detected: Security Group modified outside Terraform'
> - Team investigates and either codifies the change or reverts it
> - Git issue = feedback from operations to development"

---
---

