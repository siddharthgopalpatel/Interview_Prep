# Interview Q&A Bank: Enterprise OS Patching Automation

## Project: 18-Step Zero-Touch Lifecycle with Automatic Rollback & ITIL Compliance

**Technologies:** Ansible, Ansible Automation Platform (AAP), ServiceNow, AWS (ALB, EC2), Alertmanager, CloudWatch, PagerDuty, Slack, Python, dnf/yum

---

## Section 1: Project Story

### Q1: Walk me through the OS patching project in 2 minutes.

**Answer:**
Built a fully automated, zero-touch OS patching system with 18 discrete steps covering the complete lifecycle on Ansible Automation Platform. Opens ServiceNow Change Request with documented MOP, silences Alertmanager + CloudWatch + PagerDuty, drains traffic from ALB (serial 20% of fleet), patches servers with package exclusions, validates across 7 dimensions (packages, kernel, services, logs, disk, network, application health) with automatic rollback on any failure, verifies file integrity via checksums for 8 components, validates TLS certificates, tests connectivity to all dependencies, re-registers to ALB, re-enables monitoring, closes CR with evidence, and sends multi-channel notifications. Zero human intervention for pass/fail decisions. Runs 500+ servers monthly with zero downtime.

---

### Q2: Why 18 steps? Why not fewer?

**Answer:**
Each step is:
1. **Independently testable** — can run step 11 (integrity check) alone without running steps 1-10
2. **Independently rerunnable** — if playbook fails at step 10, restart from step 10 (not from scratch)
3. **Clear responsibility** — "which step failed?" gives immediate context for debugging
4. **Evidence generator** — each step produces audit evidence (for compliance/ServiceNow)
5. **Separable ownership** — network team reviews step 14 (connectivity), security reviews step 12 (certs)

Fewer steps = monolithic playbook. If it fails, you don't know where. Can't restart from middle. Can't test pieces independently.

---

## Section 2: Technical Deep-Dive

### Q3: How does the 7-dimension zero-touch validation work?

**Answer:**
After patching, these 7 checks decide PASS or FAIL automatically:

| # | Dimension | Check | PASS if |
|---|---|---|---|
| 1 | Package compliance | `dnf check-update` | Zero pending updates (minus exclusions) |
| 2 | Kernel verification | `uname -r` vs latest installed | Running kernel = latest (or kernel not pending) |
| 3 | Service health | `systemctl is-active` for critical services | All critical services active |
| 4 | Log errors | `journalctl --since "1 hour ago" -p err` | Zero critical errors |
| 5 | Disk space | `df -h` all mounts | All below 90% |
| 6 | Network connectivity | `nc -z` to all dependency endpoints | All critical endpoints reachable |
| 7 | Application health | HTTP GET to health endpoint | 200/301/302 response |

**ALL pass → proceed.** ANY fail → auto-rollback triggered. No human decides.

---

### Q4: How does serial batching (20%) with max_fail_percentage work?

**Answer:**
```yaml
serial: "20%"              # Patch 20% of fleet at a time
max_fail_percentage: 10    # Abort if >10% of batch fails
```

**Example with 10 servers:**
- Batch 1: web-01, web-02 → patch → validate → PASS ✅. Other 8 serve traffic.
- Batch 2: web-03, web-04 → patch → validate → PASS ✅.
- Batch 3: web-05, web-06 → patch → web-05 FAILS validation ❌
  - 1 failure out of 2 = 50% failure rate > 10% threshold → **STOP EVERYTHING**
  - web-05 auto-rollback. web-06 if already started, also rollback.
  - web-07, 08, 09, 10: NEVER TOUCHED (safe)
  - Maximum blast radius: 2 servers (20%)

**Why 20%?** Balance between speed (more parallel = faster) and safety (less parallel = less blast radius). At 20%, we always have 80% healthy capacity serving users.

---

### Q5: How does the 3-level service health check work?

**Answer:**
```
Level 1: systemd — is the process managed and active?
Level 2: Port — is it actually bound and listening?
Level 3: HTTP — is the application serving correct responses?
```

**Why all 3?**
- Process running ≠ healthy (could be zombie, deadlocked)
- Port listening ≠ healthy (could return 500s)
- HTTP 200 = strongest proof (app alive + serving correctly)

```yaml
# Level 1
- systemd: name=httpd  # Check ActiveState == "active"

# Level 2
- shell: "ss -tlnp | grep ':80 ' | wc -l"  # Expect > 0

# Level 3
- uri: url="http://localhost:80/" status_code=[200,301,302]  # Expect success
```

Used BEFORE patching (record state), AFTER stopping (verify stopped), and AFTER starting (verify recovered).

---

### Q6: How does ALB traffic management achieve zero downtime?

**Answer:**
Per-server flow:
```
1. Deregister from ALB target group (new requests stop coming)
2. Wait 60s for connection draining (in-flight requests finish)
3. Verify zero active connections (ss -tn | grep ':80' | wc -l == 0)
4. NOW safe to patch (zero traffic on this server)
5. After validation passes:
6. Re-register to ALB target group
7. Wait for ALB health check to pass (GET /health → 200)
8. Server is back in rotation
```

**Key:** Other servers in the fleet serve 100% of traffic while one is being patched. Users never notice.

---

### Q7: How do monitoring silences work and why auto-expire?

**Answer:**
Silences 3 systems before patching:
- Alertmanager: silence by hostname matcher
- CloudWatch: disable alarm actions
- PagerDuty: maintenance window

**All set to auto-expire in 4 hours.**

**Why auto-expire?** If playbook crashes at step 8 (mid-patch) and step 16 (re-enable monitoring) never runs → server could be broken AND unmonitored. With auto-expire: after 4 hours, silences expire → alerts fire → team discovers the server needs attention. Maximum unmonitored window: 4 hours (not forever).

---

### Q8: How does the integrity check (checksum + tag) work?

**Answer:**
After patching, OS package updates can overwrite custom config files with package defaults.

**Checksum verification:**
```yaml
checksum_files:
  - { path: "/etc/httpd/conf/httpd.conf", expected: "a4f8b3c2d1e5..." }
  - { path: "/opt/app/bin/app-server", expected: "f7e6d5c4b3a2..." }
  # 8 critical files total
```
Lambda computes `md5sum` of each file → compares to known-good baseline. Mismatch = patching overwrote something.

**Tag verification:**
```yaml
tag_files:
  - { path: "/opt/app/VERSION", expected_content: "v3.2.1" }
```
Confirms correct application version is still deployed after OS update.

---

## Section 3: Troubleshooting

### Q9: Patching succeeded but 7-dimension validation fails on "service health." What do you investigate?

**Answer:**
1. Which service failed? Check validation report — shows per-service PASS/FAIL.
2. **Common causes after patching:**
   - Shared library updated → app fails to start (incompatible .so version). Check `journalctl -u httpd` for errors.
   - Config file overwritten by package default → app starts but wrong configuration. Integrity check (step 11) should catch this.
   - Port conflict → new package installed a service on same port. Check `ss -tlnp | grep ':80'`.
   - Insufficient memory → kernel update changed memory allocation. Check `free -m`.
3. **Action:** Auto-rollback triggered (revert to AMI snapshot). Server stays out of ALB. CR updated as FAILED. Team investigates root cause.

---

### Q10: Server was patched successfully but can't re-register to ALB. Health check keeps failing. Why?

**Answer:**
1. **App is running** (Level 1 + Level 2 pass) **but ALB health check uses different endpoint than our check.**
   - Our check: `localhost:80/` — works (local)
   - ALB check: `GET /health` from ALB IP — might fail if security group changed, or if app needs warm-up time
2. **Common causes:**
   - Package update changed Security Group rules (unlikely but check)
   - App needs warm-up (loading caches, JIT compilation) — add delay before re-registering
   - DNS resolution changed after reboot — app can't reach dependency, returns 500 on health endpoint
3. **Fix:** Increase wait time between service start and ALB re-registration. Add retries (12 × 10s = 2 min).

---

### Q11: Playbook fails mid-way at step 8 (patching). Server is unpatched, traffic is drained. What happens?

**Answer:**
Current state: Server is OUT of ALB (step 5 completed), services are stopped (step 6), but patch failed.

**What we do:**
1. Monitoring silence still active (auto-expires in 4 hours — safety net)
2. Server is not serving traffic (ALB drained) — no user impact
3. Other servers are handling 100% traffic — no capacity issue (serial 20%)
4. **Recovery:** Start services (step 9) → verify running (step 10) → re-register to ALB (step 15) → re-enable monitoring (step 16)
5. CR updated: "ABORTED — patching failed on web-05, server recovered to pre-patch state"
6. Investigate WHY patching failed (disk full? repo unreachable? package conflict?)

**Key design:** Each step is independently rerunnable. We can run steps 9, 10, 15, 16 to recover without re-running 1-8.

---

## Section 4: System Design

### Q12: How would you scale this to 1000+ servers across multiple regions?

**Answer:**
1. **Dynamic inventory:** Replace static host lists with `aws_ec2` plugin (auto-discovers by tag)
2. **Regional execution:** AAP Instance Groups per region (controller in each region for low latency)
3. **Increase serial:** 20% of 1000 = 200 per batch. Maybe reduce to 10% (100) for extra safety.
4. **Parallel workflows:** Different server roles patch independently (web servers Monday, DB Tuesday, workers Wednesday)
5. **Smarter scheduling:** Patch by business criticality — internal tools first (less risk), customer-facing last
6. **Centralized reporting:** All results flow to single dashboard (Grafana) showing patch status across all regions

Architecture stays the same. Scale changes: inventory (dynamic), batch size (adjust serial), execution (distributed AAP).

---

### Q13: How would you add pre-approval workflow for critical servers?

**Answer:**
For standard servers: fully automated (current flow).
For critical production servers: add human approval gate.

**Using AAP Workflow Template:**
```
[Open CR] → [Silence] → [Pre-Check] → [Drain] → 
  [APPROVAL NODE] ← Authorized person clicks "Approve" in AAP UI
    → [Patch] → [Validate] → [Re-register] → [Close CR]
```

AAP approval node pauses workflow until authorized user approves. Notifications sent to Slack + email.

**Who approves:** Only `release-mgr` or `platform-lead` AAP roles can approve production patches. RBAC enforced.

---

## Section 5: Comparison & Decisions

### Q14: Why Ansible over shell scripts for patching?

**Answer:**
| | Shell Scripts | Ansible (Our Choice) |
|---|---|---|
| Idempotent | ❌ Must handle manually | ✅ Built-in (safe to re-run) |
| Error handling | Basic (set -e, trap) | ✅ block/rescue/always, ignore_errors |
| Parallel execution | Manual (background &) | ✅ forks, serial, async built-in |
| Reporting | DIY (echo to file) | ✅ Structured facts, registered variables |
| Secrets | Hardcoded or env vars | ✅ Ansible Vault (encrypted) |
| Reusability | Copy-paste | ✅ Roles (12 reusable roles) |
| Audit trail | Log files (maybe) | ✅ AAP logs every task, every host |
| RBAC | None | ✅ AAP teams/roles/permissions |

Shell scripts at 10 servers = OK. At 500 servers with compliance requirements = unmanageable.

---

### Q15: Why AAP (Tower) over raw ansible-playbook on cron?

**Answer:**
| | Cron + ansible-playbook | AAP (Our Choice) |
|---|---|---|
| Visibility | Log file on controller (hope you saved it) | Web UI — see every run, every task, every host |
| RBAC | Everyone is root | Teams, roles, who can run what against where |
| Scheduling | cron (no calendar view, easy to conflict) | Visual scheduler + calendar |
| Credentials | Vault file on disk | Credential types, injected at runtime, never exposed |
| Approval | None (just runs) | Workflow approval nodes (human gate) |
| Survey (runtime vars) | Command-line args (error-prone) | Interactive form (hostname, patch_type, exclude_packages) |
| Retry | Manual re-run | Click "Retry" on failed host |
| Notifications | DIY in playbook | Built-in notification templates |

**Key:** Non-Ansible engineers (team leads) can trigger patching by filling a form in AAP. No CLI knowledge required.

---

### Q16: Config-driven connectivity tests vs hardcoded checks — why config-driven?

**Answer:**
**Hardcoded:**
```yaml
- shell: "nc -z database.internal 5432"
- shell: "nc -z redis.internal 6379"
- shell: "nc -z api-gateway.internal 443"
```
Adding a new dependency = code change + PR + review + deploy.

**Config-driven (our approach):**
```yaml
# vars/main.yml
network_endpoints:
  - { host: "database.internal", port: 5432, name: "Database", critical: true }
  - { host: "redis.internal", port: 6379, name: "Redis", critical: true }
  - { host: "api-gateway.internal", port: 443, name: "API Gateway", critical: true }
  - { host: "prometheus.internal", port: 9090, name: "Monitoring", critical: false }
```
Adding a new dependency = add one line to YAML. No code change. No PR for the role.

**Also:** `critical: true/false` means monitoring being unreachable doesn't block patching (non-critical), but database unreachable DOES block (critical).

---

## Section 6: Behavioral

### Q17: How did you get buy-in for building this automation?

**Answer (STAR):**
- **Situation:** Team patched servers manually — SSH, run commands, check results, update tickets. Took 2 hours per server. 500 servers = impossible to keep up with monthly patching cadence.
- **Task:** Convince management to invest 4 weeks in building automation.
- **Action:** Showed data: "We have 500 servers. At 2 hours each, that's 1000 hours/month = 6 FTEs just for patching. Automation: 1 engineer built it in 4 weeks, now runs unattended. Also: last quarter we had 3 incidents from missed validations during manual patching — automation catches those every time."
- **Result:** Approved. Built in 4 weeks. First month: patched 500 servers with zero incidents. Previous month (manual): 2 post-patch issues. ROI proven immediately.

---

### Q18: Tell me about a time the automatic rollback saved production.

**Answer (STAR):**
- **Situation:** Monthly patching cycle. Batch 3 (web-05, web-06). After patching web-05, 7-dimension validation ran.
- **What happened:** Dimension 3 (service health) FAILED. httpd wouldn't start because openssl update changed a cipher configuration → SSL handshake error in httpd.conf.
- **Impact:** Zero user impact (web-05 was already drained from ALB before patching).
- **Auto-rollback:** AMI snapshot reverted. Server restored to pre-patch state in 3 minutes. CR updated: "FAILED — web-05, openssl cipher compatibility issue."
- **Fix:** Updated httpd.conf to use new cipher suite. Added to integrity check baseline. Re-ran patching on web-05 → PASS.
- **Result:** Without automation: users would've seen errors (server would've been patched under live traffic). With automation: zero user impact, self-healed, team notified, root cause found next business day.

---

## Section 7: Future & Improvements

### Q19: What would you add next to this system?

**Answer:**
| Priority | Improvement | Why |
|---|---|---|
| 1 | Dynamic inventory (aws_ec2 plugin) | Replace static host lists with auto-discovery by tags |
| 2 | Molecule testing for all 12 roles | Automated role testing before using in production |
| 3 | Master orchestrator playbook | Single entry point for all 18 steps (currently separate playbooks) |
| 4 | Vulnerability-driven patching | Integrate with Qualys/Nessus — only patch servers with actual vulnerabilities, not entire fleet |
| 5 | ServiceNow report attachment via API | Attach HTML reports directly to CR (currently text in close notes) |
| 6 | Compliance dashboard (Grafana) | Patch status across fleet: which servers are compliant, which are overdue |

---

### Q20: How does this differ from a simple "dnf update in a for loop"?

**Answer:**
| | For Loop (`for server in ...; ssh $server dnf update`) | Our 18-Step Automation |
|---|---|---|
| Traffic management | ❌ Patches under live traffic | ✅ ALB drain before, re-register after |
| Validation | ❌ None — hope it works | ✅ 7-dimension automated PASS/FAIL |
| Rollback | ❌ Manual (SSH in, figure it out) | ✅ Automatic (AMI snapshot restore) |
| Monitoring | ❌ Alerts fire during maintenance | ✅ Silence before, re-enable after |
| ITIL/Audit | ❌ No ticket, no evidence | ✅ CR with MOP, evidence, closure |
| Blast radius | ❌ All servers at once | ✅ Serial 20%, max_fail stops spread |
| Integrity | ❌ Might overwrite configs | ✅ Checksum verification catches it |
| Certificates | ❌ Might break TLS | ✅ 4-check cert validation |
| Notification | ❌ Nobody knows until users report | ✅ Slack + Email + PagerDuty (failure only) |

**One-liner:** "A for loop patches. Our system patches SAFELY — with traffic management, validation, rollback, compliance, and zero human decision-making."

---

*End of Q&A Bank — 20 questions covering all 7 dimensions*
