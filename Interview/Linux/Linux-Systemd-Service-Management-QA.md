# Interview Q&A Bank: Systemd & Service Management

## Context: Projects 2, 3, 8, 9 use systemd. Project 9 does 3-level service verification. Project 3 runs kubelet as systemd service.

---

### Q11: Explain the structure of a systemd unit file. What are the sections and key directives?

**Project Reference:** Project 3 (Kubernetes Environments — kubeadm cluster, kubelet systemd unit)
**Expected Depth:** Not just [Unit][Service][Install] — real directives you'd configure for production services, how kubelet's unit file is structured.

**Answer:**

A systemd unit file has three main sections. Here's how kubelet's unit file looks on our kubeadm cluster (Project 3) — this is a real production example:

```ini
# /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/
Wants=network-online.target
After=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/kubelet \
  --config=/var/lib/kubelet/config.yaml \
  --kubeconfig=/etc/kubernetes/kubelet.conf \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock
Restart=always
RestartSec=10
StartLimitInterval=0
KillMode=process
CPUAccounting=true
MemoryAccounting=true

[Install]
WantedBy=multi-user.target
```

**[Unit] section — metadata and ordering:**
- `Description` — shows in `systemctl status`
- `After` — start AFTER these units (ordering only, not dependency)
- `Wants` — soft dependency (if the wanted unit fails, this still starts)
- `Requires` — hard dependency (if required unit fails, this fails too)
- `Documentation` — man page or URL references

**[Service] section — how to run:**
- `Type` — how systemd determines "started" status:
  - `simple` (default): process starts = service started
  - `notify`: process sends `sd_notify(READY=1)` when truly ready (kubelet uses this)
  - `forking`: process forks and parent exits (legacy daemons)
  - `oneshot`: process completes and exits (for scripts)
- `ExecStart` — the command to run
- `ExecStartPre/Post` — commands before/after main process
- `Restart` — when to restart (always, on-failure, on-abnormal)
- `RestartSec` — delay between restarts
- `KillMode` — what to kill on stop (process, control-group, mixed)
- `Environment/EnvironmentFile` — set env vars

**[Install] section — how to enable:**
- `WantedBy=multi-user.target` — enables at boot (equivalent to old runlevel 3)
- `RequiredBy` — hard dependency in reverse

**Why `Type=notify` matters for kubelet:** With `simple`, systemd considers kubelet "started" the instant the process spawns — even before it connects to the API server. With `notify`, kubelet signals readiness only after initialization is complete. This means `systemctl start kubelet` only returns success when kubelet is actually functioning.

---

### Q12: Explain service dependencies — After, Requires, Wants, BindsTo. When do you use each?

**Project Reference:** Project 9 (OS Patching — service stop/start ordering) and Project 3 (kubelet dependencies)
**Expected Depth:** The difference between ordering (After/Before) and dependency (Requires/Wants), and BindsTo for tightly coupled services.

**Answer:**

The key insight most people miss: **ordering and dependency are SEPARATE concepts in systemd.**

**Ordering (After/Before) — WHEN to start:**
```ini
After=network-online.target postgresql.service
```
"Start me AFTER these units are started." But if postgresql isn't enabled/started at all, this directive is ignored. It only affects sequence IF both units are being started.

**Dependency (Requires/Wants) — WHETHER to start:**

**`Wants=` (soft dependency):**
```ini
Wants=network-online.target
```
"Try to start network-online.target when starting me. If it fails, start me anyway." Use for nice-to-have dependencies. Kubelet wants networking but might still start (degraded) without it.

**`Requires=` (hard dependency):**
```ini
Requires=containerd.service
```
"containerd MUST be running. If it fails or is stopped, stop me too." Use for hard requirements where your service is useless without the dependency.

**`BindsTo=` (tightest coupling):**
```ini
BindsTo=dev-sda1.device
```
"If this unit is stopped, deactivated, OR enters a failed state, stop me immediately." Stronger than Requires — reacts to the dependency entering failed state, not just stopping. Use for: mount points binding to devices, containers binding to their runtime.

**Common pattern in production (Project 9 — app depends on database):**
```ini
[Unit]
Description=My Application
After=network-online.target postgresql.service
Wants=network-online.target
Requires=postgresql.service
```
Translation: "Start after network and postgres. Network is nice-to-have (starts anyway if network slow). Postgres is required (don't start without it, and stop if it stops)."

**Critical mistake I've seen:** Using `After=` without `Requires=` or `Wants=`. `After=postgresql.service` alone does NOT start postgresql. It only says "if both of us are starting, start me second." You need BOTH ordering and dependency.

**In Project 9 (service stop ordering):**
We stop services in reverse dependency order during patching. If app depends on database:
- Stop: app first → then database (so app doesn't crash trying to reach stopped DB)
- Start: database first → then app (so app finds DB ready)

```yaml
# Ansible handles ordering via task sequence, not systemd dependency
- name: Stop application
  systemd: { name: myapp, state: stopped }
- name: Stop database
  systemd: { name: postgresql, state: stopped }
# ... patch ...
- name: Start database
  systemd: { name: postgresql, state: started }
- name: Start application
  systemd: { name: myapp, state: started }
```

---

### Q13: How do you use journalctl effectively? Filtering by time, priority, unit — give real troubleshooting examples.

**Project Reference:** Project 9 (OS Patching — post-patch log verification, dimension 4 of 7-dimension validation)
**Expected Depth:** Advanced filtering, output formats, storage management, and how it integrates with automated validation.

**Answer:**

journalctl is the interface to systemd's journal. In production at scale, knowing the right query saves hours.

**Filtering by unit (most common):**
```bash
journalctl -u kubelet                    # All kubelet logs
journalctl -u kubelet --no-pager -n 50   # Last 50 lines
journalctl -u kubelet -f                 # Follow (like tail -f)
journalctl -u httpd -u nginx             # Multiple units (OR logic)
```

**Filtering by time (incident investigation):**
```bash
journalctl --since "2024-01-15 02:00" --until "2024-01-15 03:00"
journalctl --since "1 hour ago"
journalctl --since "yesterday"
journalctl -u sshd --since "2024-01-15 14:30" --until "2024-01-15 14:45"
```

**Filtering by priority (the killer feature):**
```bash
journalctl -p err                 # Only errors and above (err, crit, alert, emerg)
journalctl -p warning             # Warnings and above
journalctl -p err --since "1 hour ago" -u httpd   # Errors in httpd in last hour
```
Priority levels: emerg(0), alert(1), crit(2), err(3), warning(4), notice(5), info(6), debug(7)

**In Project 9 (7-dimension validation, dimension 4 — log errors):**
```yaml
# Check for critical errors after patching
- name: "Dimension 4 — Check for post-patch errors"
  shell: |
    journalctl --since "1 hour ago" -p err --no-pager -q | wc -l
  register: error_count

- name: "Evaluate log errors"
  set_fact:
    log_check_pass: "{{ error_count.stdout | int == 0 }}"
```
If ANY error-level messages appeared in the hour since patching — validation flags it. Zero errors = PASS.

**Advanced patterns:**
```bash
# Kernel messages only (hardware issues, OOM kills)
journalctl -k
journalctl -k -p err    # Kernel errors only

# By PID (track a specific process)
journalctl _PID=12345

# JSON output (for parsing in scripts)
journalctl -u httpd -o json-pretty --since "5 min ago"

# Disk usage of journal
journalctl --disk-usage
# Vacuum (free space)
journalctl --vacuum-size=500M
journalctl --vacuum-time=7d
```

**Journal persistence config (`/etc/systemd/journald.conf`):**
```ini
[Journal]
Storage=persistent          # Write to /var/log/journal/ (survives reboot)
SystemMaxUse=2G            # Max disk usage
MaxRetentionSec=30day      # Keep logs for 30 days
Compress=yes               # Compress old logs
```

Without `Storage=persistent`, journal is in `/run/log/journal/` (RAM) and lost on reboot. First thing I configure on any new server.

---

### Q14: Explain systemd targets vs SysV runlevels. How do you change the default target?

**Project Reference:** Project 9 (OS Patching — servers must boot to correct target after reboot)
**Expected Depth:** Target equivalents, how to manage them, and the practical difference for server administration.

**Answer:**

Targets are systemd's replacement for runlevels — but they're more flexible (a system can have multiple active targets simultaneously, forming a dependency graph).

**Mapping:**
| SysV Runlevel | systemd Target | Purpose |
|---|---|---|
| 0 | poweroff.target | Shut down |
| 1 | rescue.target | Single-user (root only, minimal services) |
| 3 | multi-user.target | Full multi-user, no GUI (servers) |
| 5 | graphical.target | Multi-user + GUI (desktops) |
| 6 | reboot.target | Reboot |

**Check/change default target:**
```bash
# Current default
systemctl get-default
# multi-user.target (expected for servers)

# Change default
systemctl set-default multi-user.target

# Temporary change (this boot only)
systemctl isolate rescue.target   # Drop to rescue mode NOW
```

**Why targets matter in Project 9:**

After patching + reboot, the server MUST come back to `multi-user.target`. If somehow the default target got changed (misconfigured package, admin error), the server boots to rescue mode or graphical mode — services don't start, ALB health check fails, patching validation fails.

```yaml
# Post-reboot verification in Project 9
- name: Verify system booted to correct target
  command: systemctl get-default
  register: boot_target
  failed_when: boot_target.stdout != "multi-user.target"
```

**How targets work internally:**
`multi-user.target` is just a unit with `Wants=` and `After=` pointing to many other units. When you "boot to multi-user.target," systemd builds the dependency tree from that target and starts everything it needs.

```bash
# See what multi-user.target pulls in
systemctl list-dependencies multi-user.target
```

**`isolate` — the target equivalent of switching runlevels:**
```bash
systemctl isolate rescue.target    # Stop everything not needed by rescue
systemctl isolate multi-user.target # Start everything for multi-user
```

`isolate` stops units that aren't wanted by the new target. It's how you "switch runlevels" live. Most units have `AllowIsolate=no` to prevent accidental isolation.

---

### Q15: How do you troubleshoot a systemd service that won't start? Walk me through your process.

**Project Reference:** Project 9 (OS Patching — post-patch service start failures, 3-level verification)
**Expected Depth:** Systematic debugging approach — not just "check the logs" but the full investigation flow a 12 YOE engineer follows.

**Answer:**

Here's my systematic approach when a service fails to start after patching (happens ~2-3 times per 500 server cycle):

**Step 1: Check the unit status**
```bash
systemctl status httpd
# Shows: Active: failed (Result: exit-code)
# Shows: last few log lines
# Shows: PID, start time, exit code
```
The exit code and status line give the first clue.

**Step 2: Full journal for that unit**
```bash
journalctl -u httpd --since "5 min ago" --no-pager
# Shows the actual error message from the process
```
Common findings: "Address already in use" (port conflict), "Permission denied" (SELinux or file permissions), "No such file or directory" (missing config/binary).

**Step 3: Check syntax/configuration**
```bash
# For httpd
httpd -t
# For nginx
nginx -t
# For generic services - try running the ExecStart command manually
/usr/sbin/httpd -DFOREGROUND
```
Running the binary directly gives you the FULL error without systemd wrapping it.

**Step 4: Check dependencies**
```bash
systemctl list-dependencies httpd
# Is a required dependency failed?
systemctl is-active network-online.target
```

**Step 5: Check resource issues**
```bash
# Port conflict
ss -tlnp | grep ':80'
# If something else grabbed port 80

# Disk space
df -h
# /var full = can't write PID file or logs

# Memory
free -m
# OOM conditions
dmesg | grep -i "oom\|killed"
```

**Step 6: SELinux (the silent killer on RHEL)**
```bash
# Check if SELinux is blocking
ausearch -m avc --start recent
# Or
journalctl -t setroubleshoot --since "5 min ago"
# Or simply
getenforce    # If Enforcing, try Permissive temporarily
setenforce 0  # Temporary test — does service start now?
```

If service starts with SELinux permissive — you found the problem. Fix with proper policy, not by disabling SELinux.

**Step 7: File permissions and ownership**
```bash
# Check if patching changed ownership
ls -la /etc/httpd/conf/httpd.conf
# Check if .rpmsave/.rpmnew situation occurred
find /etc/httpd -name "*.rpmnew" -o -name "*.rpmsave"
```

**In Project 9 — automated diagnosis:**
When our 3-level verification detects a service failure:
- Level 1 (systemd state) fails → capture `systemctl status` output
- Level 2 (port check) fails → capture `ss -tlnp` output  
- Level 3 (HTTP health) fails → capture HTTP response/error

All captured in the validation report attached to ServiceNow CR. The on-call engineer gets a structured report, not "patching failed — figure it out."

---


### Q16: Explain ExecStartPre/ExecStartPost and health check patterns in systemd unit files.

**Project Reference:** Project 9 (OS Patching — 3-level service verification) and Project 2 (Platform provisioning — custom services)
**Expected Depth:** How to build robust startup sequences with pre/post checks, failure handling with `-` prefix, and real patterns.

**Answer:**

`ExecStartPre` and `ExecStartPost` let you run commands before and after the main process starts. This is where you implement health gates.

**Basic structure:**
```ini
[Service]
ExecStartPre=/usr/bin/test -f /etc/myapp/config.yaml
ExecStartPre=/usr/bin/myapp --validate-config
ExecStart=/usr/bin/myapp --config /etc/myapp/config.yaml
ExecStartPost=/usr/bin/curl -sf http://localhost:8080/health
```

**Execution order:**
1. All `ExecStartPre` commands run sequentially
2. If ANY `ExecStartPre` fails (non-zero exit) → service goes to `failed` state, `ExecStart` never runs
3. `ExecStart` runs (the main process)
4. `ExecStartPost` runs after the main process starts

**The `-` prefix (ignore failures):**
```ini
ExecStartPre=-/usr/bin/mkdir -p /var/run/myapp
```
The `-` means: if this command fails, continue anyway. Without `-`, any non-zero exit stops the startup sequence.

**Real production pattern (custom app service):**
```ini
[Service]
Type=notify
# Pre-flight checks
ExecStartPre=/usr/bin/test -f /opt/app/bin/app-server
ExecStartPre=/opt/app/bin/app-server --check-config
ExecStartPre=-/usr/bin/mkdir -p /var/log/myapp
ExecStartPre=-/usr/bin/chown appuser:appgroup /var/log/myapp

# Main process
ExecStart=/opt/app/bin/app-server --config /etc/app/config.yaml

# Post-start health verification
ExecStartPost=/bin/bash -c 'for i in $(seq 1 30); do curl -sf http://localhost:8080/health && exit 0; sleep 1; done; exit 1'

# Graceful shutdown
ExecStop=/bin/kill -SIGTERM $MAINPID
ExecStopPost=/bin/bash -c 'while ss -tlnp | grep -q ":8080"; do sleep 1; done'
```

**ExecStopPost pattern (Project 9 — graceful drain):**
The `ExecStopPost` ensures the port is actually released before systemd considers the service "stopped." This prevents "Address already in use" on restart.

**In Project 9's 3-level verification, we don't use ExecStartPost in the unit file. Instead, Ansible does the verification externally:**
```yaml
# Level 1: systemd says active
- name: Check systemd state
  systemd: { name: httpd }
  register: svc_status

# Level 2: Port listening
- name: Verify port open
  shell: "ss -tlnp | grep ':80 ' | wc -l"
  register: port_check

# Level 3: HTTP responding
- name: Application health check
  uri:
    url: "http://localhost:80/"
    status_code: [200, 301, 302]
    timeout: 10
  retries: 3
  delay: 5
```

**Why external verification vs ExecStartPost?** ExecStartPost runs once. Our Ansible check retries with delay — handles JVM warmup, cache loading, DB connection pool initialization (10-15 seconds). Also, ExecStartPost failure marks the unit as failed — we want to DETECT and REPORT, not crash the service.

---

### Q17: Compare systemd timers vs cron. When would you choose each?

**Project Reference:** Project 9 (OS Patching — scheduled maintenance windows) and Project 8 (monitoring alerting schedules)
**Expected Depth:** Architectural advantages of timers, practical migration examples, and when cron is still appropriate.

**Answer:**

**systemd timers** are the modern replacement for cron, but cron isn't dead. Here's the real comparison:

**systemd timer anatomy (two files needed):**
```ini
# /etc/systemd/system/cleanup.timer
[Unit]
Description=Daily log cleanup

[Timer]
OnCalendar=*-*-* 02:00:00        # Every day at 2 AM
Persistent=true                    # Run immediately if missed (server was off)
RandomizedDelaySec=300             # Random 0-5min offset (avoid thundering herd)
AccuracySec=1min                   # Precision (default 1min is fine)

[Install]
WantedBy=timers.target

# /etc/systemd/system/cleanup.service
[Unit]
Description=Log cleanup task

[Service]
Type=oneshot
ExecStart=/opt/scripts/cleanup.sh
User=root
StandardOutput=journal
StandardError=journal
```

**Advantages of timers over cron:**

| Feature | cron | systemd timer |
|---|---|---|
| Logging | Syslog/mail (easy to miss) | Full journal integration (`journalctl -u cleanup`) |
| Dependency management | None | Can use `After=`, `Requires=` |
| Resource control | None | `CPUQuota=`, `MemoryMax=`, `IOWeight=` via service |
| Missed runs (server was off) | Lost forever | `Persistent=true` runs on next boot |
| Randomized delay | Not built-in | `RandomizedDelaySec=` (prevents fleet stampede) |
| Calendar expressions | Limited (5 fields) | Rich (`OnCalendar=Mon..Fri *-*-* 09:00`) |
| Monitoring | Hard (check cron logs) | `systemctl list-timers`, `is-active`, alerting |
| Concurrency control | None (can overlap) | Only one instance (won't run if previous still running) |

**When I still use cron:**
- Simple one-liners that don't need monitoring or dependency management
- When team isn't comfortable with systemd yet
- Ephemeral containers where systemd isn't PID 1
- User-level cron jobs (`crontab -e`) for non-root tasks that don't need journaling

**When I use systemd timers:**
- Anything that needs resource limits (don't let a log rotation eat all CPU)
- Fleet-wide scheduled tasks where `RandomizedDelaySec` prevents thundering herd
- Tasks that MUST run even if the system was off at scheduled time (`Persistent=true`)
- Tasks where I need to see "when did this last run? did it succeed?" at a glance

**In context of Project 9:** The patching itself runs from AAP (not local timers), but we use systemd timers on target servers for:
- Daily security compliance checks
- Hourly log rotation for heavy-logging applications
- Certificate expiry monitoring (daily check, alert at 30 days)

```bash
# Check all active timers
systemctl list-timers --all
# NEXT                         LEFT     LAST                         PASSED   UNIT
# Mon 2024-01-15 02:00:00 UTC  3h left  Sun 2024-01-14 02:00:00 UTC  21h ago  cleanup.timer
```

---

### Q18: What is socket activation in systemd? When is it useful?

**Project Reference:** Project 3 (Kubernetes — kubelet and containerd socket communication)
**Expected Depth:** How socket activation works, real use cases, and the connection to container runtimes.

**Answer:**

Socket activation is systemd's on-demand service startup pattern. Instead of starting a service at boot and having it listen on a socket, systemd OWNS the socket and starts the service only when something connects to it.

**How it works:**
1. systemd creates and listens on the socket (port, Unix socket, etc.)
2. A connection arrives
3. systemd starts the associated service
4. systemd passes the socket file descriptor to the service
5. Service handles the connection
6. (Optionally) service exits when idle; systemd keeps holding the socket

**Two-file setup:**
```ini
# /etc/systemd/system/myapp.socket
[Unit]
Description=MyApp Socket

[Socket]
ListenStream=/run/myapp.sock        # Unix socket
# OR: ListenStream=8080              # TCP port
Accept=no                            # Service handles all connections (not per-connection spawn)

[Install]
WantedBy=sockets.target

# /etc/systemd/system/myapp.service
[Unit]
Description=MyApp
Requires=myapp.socket

[Service]
ExecStart=/usr/bin/myapp
```

**Real use cases:**

1. **Container runtimes (Project 3):** `containerd.socket` — containerd is socket-activated. When kubelet tries to create a container via `/run/containerd/containerd.sock`, systemd ensures containerd is running. This is why kubelet's unit file has:
```ini
After=containerd.service
Requires=containerd.service
```

2. **SSH (`sshd.socket`):** Instead of sshd running 24/7, systemd holds port 22. First SSH connection triggers sshd start. On rarely-accessed servers, this saves resources.

3. **Docker socket (`docker.socket`):** Docker daemon starts only when something hits `/var/run/docker.sock`. CLI commands or CI agents trigger it on demand.

4. **Cockpit (web management):** `cockpit.socket` holds port 9090. Admin connects → cockpit starts. No resources wasted when nobody is managing the server.

**Advantages:**
- **Boot speed:** Services don't all start at once — only activated when needed
- **Dependency resolution:** Services that depend on each other can start in parallel — systemd holds the socket so connections queue (not fail) while the service initializes
- **Resource efficiency:** Rarely-used services consume zero resources until needed
- **Zero-downtime restart:** systemd holds the socket during service restart — connections queue, nothing is lost

**In Project 9 context:** We don't directly configure socket activation, but we depend on it — containerd's socket activation ensures the container runtime is available when kubelet needs it after a reboot during patching.

---

### Q19: When and why do you need `systemctl daemon-reload`? What happens if you forget it?

**Project Reference:** Project 9 (OS Patching — service unit file changes after package updates)
**Expected Depth:** The systemd manager's in-memory state vs on-disk files, when reload is needed, and automation implications.

**Answer:**

`systemctl daemon-reload` tells systemd to re-read ALL unit files from disk. It refreshes the in-memory representation of unit configurations.

**When you MUST run it:**
1. After creating a new unit file
2. After modifying an existing unit file (edited directives)
3. After adding/removing override files in `.d/` directories
4. After a package update that changes a unit file

**What happens if you forget:**
```bash
vim /etc/systemd/system/myapp.service   # Change RestartSec from 5 to 30
systemctl restart myapp                  # STILL USES OLD RestartSec=5!
# Systemd warns:
# Warning: The unit file, source configuration file or drop-ins of myapp.service changed on disk.
# Run 'systemctl daemon-reload' to reload units.
```

The service restarts with OLD configuration. Your change is on disk but not loaded. This is a common "why isn't my change working?" trap.

**What daemon-reload does NOT do:**
- It does NOT restart any services
- It does NOT stop running services
- It only reloads configuration from disk into systemd manager memory

**After daemon-reload, you still need to restart the service if you want the new config applied to the running process:**
```bash
systemctl daemon-reload    # Load new config
systemctl restart myapp    # Apply it (restart with new config)
```

**In Ansible (Project 9):**
```yaml
- name: Deploy updated service file
  template:
    src: myapp.service.j2
    dest: /etc/systemd/system/myapp.service
  notify: 
    - reload systemd
    - restart myapp

handlers:
  - name: reload systemd
    systemd:
      daemon_reload: yes

  - name: restart myapp
    systemd:
      name: myapp
      state: restarted
```

**The Ansible `systemd` module handles this elegantly:**
```yaml
- name: Ensure service is running with latest config
  systemd:
    name: myapp
    state: restarted
    daemon_reload: yes    # Reload THEN restart — one task
```

**In Project 9 during patching:** Package updates (step 8) can update unit files for services like httpd, sshd, or containerd. Our service restart step (step 9) always does `daemon_reload: yes` to ensure we're not running with stale unit configurations from before the patch.

**`daemon-reexec` — the nuclear option:**
```bash
systemctl daemon-reexec    # Re-execute systemd manager itself (PID 1 restarts)
```
Use only when systemd itself was updated. This is what happens during a systemd package update — it needs to replace its own binary. Rarely needed manually.

---

### Q20: Explain systemd restart policies — Restart, RestartSec, StartLimitBurst, on-failure vs always.

**Project Reference:** Project 3 (kubelet restart policy — must always restart) and Project 9 (service recovery after patching)
**Expected Depth:** How to design resilient restart behavior, prevent crash loops, and configure appropriate backoff.

**Answer:**

Restart policies determine what happens when a service dies unexpectedly. Getting this right is the difference between self-healing and crash-loop hell.

**`Restart=` options:**
| Value | Restarts on | Use case |
|---|---|---|
| `no` | Never | One-shot tasks, intentional exits |
| `always` | Any exit (clean or dirty) | Critical daemons that MUST run (kubelet) |
| `on-failure` | Non-zero exit, signal, timeout | Normal services (don't restart on `systemctl stop`) |
| `on-abnormal` | Signal, timeout, watchdog | Don't restart on clean non-zero exit |
| `on-abort` | Unclean signal only (SEGV, ABRT) | Restart only on crashes, not config errors |

**kubelet's configuration (Project 3):**
```ini
Restart=always
RestartSec=10
StartLimitInterval=0    # DISABLE the start limit entirely
```

**Why `always` + `StartLimitInterval=0` for kubelet?** Kubelet MUST run on every node, period. If it crashes, restart it immediately. If it can't start (API server down during upgrade), keep trying forever. A node without kubelet is a dead node.

**`RestartSec=` (delay between restart attempts):**
```ini
RestartSec=5    # Wait 5 seconds before restarting
```
Without this, a crashing service restarts instantly → crashes → restarts → crashes → rapid-fire loop consuming resources and flooding logs.

**Start rate limiting (`StartLimitBurst` + `StartLimitIntervalSec`):**
```ini
StartLimitBurst=5              # Max 5 starts...
StartLimitIntervalSec=60       # ...within 60 seconds
```
If the service starts (and crashes) 5 times within 60 seconds, systemd STOPS trying and marks it `failed`. This prevents crash loops from consuming system resources indefinitely.

**Designing restart policies for different service types:**

```ini
# Critical infrastructure (kubelet, containerd) — never give up
[Service]
Restart=always
RestartSec=10
StartLimitInterval=0          # No limit

# Normal application service — recover from crashes, respect stop commands
[Service]
Restart=on-failure
RestartSec=5
StartLimitBurst=5
StartLimitIntervalSec=300     # 5 attempts in 5 minutes, then give up

# Application with expensive initialization — slower backoff
[Service]
Restart=on-failure
RestartSec=30                 # 30s delay (let DB connections close, JVM GC)
StartLimitBurst=3
StartLimitIntervalSec=600     # 3 attempts in 10 minutes
```

**In Project 9 (post-patching service recovery):**

After patching and rebooting, services should start cleanly. If a service enters a restart loop (3-level verification catches this), our validation FAILS the dimension and triggers rollback.

```yaml
# Detect crash loop
- name: Check service isn't restart-looping
  shell: |
    systemctl show httpd --property=NRestarts --value
  register: restart_count

- name: Fail if service is crash-looping
  fail:
    msg: "httpd has restarted {{ restart_count.stdout }} times — likely crash loop"
  when: restart_count.stdout | int > 2
```

**`WatchdogSec=` (bonus — detect hung processes):**
```ini
WatchdogSec=30    # Service must send sd_notify(WATCHDOG=1) every 30s
```
If the service process hangs (deadlock, infinite loop), it stops sending watchdog pings → systemd kills and restarts it. This catches the case where the process is "running" but not functioning — exactly what our Level 3 HTTP health check also detects.

---
