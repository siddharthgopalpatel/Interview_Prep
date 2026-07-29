## CATEGORY 11: REAL-WORLD SCENARIO QUESTIONS — Continued (Questions 28-32)

---

### Q28: After a reboot, one application doesn't start. Full investigation.
**Project Reference:** Project 9 (Enterprise OS Patching — post-reboot validation)
**Expected Depth:** systemd boot chain, dependency failures, common post-reboot issues

**Answer:**

**This is exactly what our 7-dimension validation in Project 9 catches. Here's how I investigate:**

**Step 1: Check systemd status:**
```bash
systemctl status myapp.service
# Possible states:
# failed (exit-code)    → crashed on start
# failed (timeout)      → took too long to start
# inactive (dead)       → never attempted to start (not enabled?)
# activating (auto-restart) → crash-looping

# If "inactive" — is it enabled?
systemctl is-enabled myapp.service
# disabled → it was accidentally disabled (or package upgrade reset it)
systemctl enable --now myapp.service
```

**Step 2: Check why it failed:**
```bash
# Full journal for this boot only
journalctl -u myapp.service -b

# Common failures:
# "Cannot open /var/run/myapp/myapp.pid: No such file or directory"
# → tmpfiles not created. /var/run is tmpfs, cleared on reboot!
# Fix: Add to /etc/tmpfiles.d/myapp.conf: d /var/run/myapp 0755 appuser appuser -

# "Address already in use"
# → Port conflict. Something else took the port.
ss -tlnp | grep ':8080'

# "Permission denied"
# → SELinux context wrong after update
ausearch -m avc --start today | head -20

# "Cannot connect to database"
# → DB starts AFTER app (dependency ordering wrong)
```

**Step 3: Check dependencies:**
```bash
# What does the unit depend on?
systemctl list-dependencies myapp.service

# Does it need network?
grep -E "After|Requires|Wants" /etc/systemd/system/myapp.service
# If After=network.target but needs network-online.target → starts before network ready

# Does it depend on a mount?
grep "RequiresMountsFor" /etc/systemd/system/myapp.service
# If data is on EBS volume that mounts late → app starts before mount
```

**Step 4: Common post-reboot traps:**

| Issue | Symptom | Fix |
|-------|---------|-----|
| tmpfs (/run, /tmp) cleared | PID file, socket, temp dir missing | Use RuntimeDirectory= in unit file |
| Mount ordering | App starts before /data mounted | Add RequiresMountsFor=/data |
| Network timing | App connects before DNS/network up | After=network-online.target + Wants=network-online.target |
| SELinux relabeling | Files have wrong context | `restorecon -Rv /opt/myapp/` |
| Kernel module not loaded | App needs specific kernel module | Add to /etc/modules-load.d/ |
| Environment variables | Env set in .bashrc (not systemd) | Use EnvironmentFile= in unit |
| Resource limits | ulimit differs from interactive | Use LimitNOFILE= in unit |

**Step 5: Fix and verify:**
```bash
# Fix the root cause (example: add proper dependencies)
cat /etc/systemd/system/myapp.service
[Unit]
After=network-online.target postgresql.service
Wants=network-online.target
Requires=postgresql.service

[Service]
RuntimeDirectory=myapp
EnvironmentFile=/etc/myapp/env
ExecStart=/opt/myapp/bin/start.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

# Reload and restart
systemctl daemon-reload
systemctl restart myapp.service
systemctl status myapp.service
```

**In Project 9 context:**
- After kernel reboot, our validation checks all critical services with `systemctl is-active`
- If ANY expected service is not running → FAIL → that server stays out of ALB
- We maintain a list of critical services per server role in Ansible variables:
  ```yaml
  critical_services:
    - httpd
    - redis
    - myapp
    - node_exporter
  ```

---

### Q29: You see high iowait but disk isn't full. What's happening?
**Project Reference:** Project 7 (Cost Optimization — I/O analysis for rightsizing)
**Expected Depth:** iowait ≠ full disk. Understand it's about I/O speed/throughput not capacity.

**Answer:**

**Key clarification:** Disk full (capacity) and disk slow (I/O performance) are completely different problems. You can have a 1TB disk that's 10% full but completely saturated on IOPS.

**What iowait actually means:**
CPU is idle but cannot run the process because the process is waiting for disk I/O to complete. The CPU has nothing else to do, so it reports iowait instead of idle.

**Root causes of high iowait with space available:**

**1. IOPS saturation (most common in cloud):**
```bash
iostat -xz 1 5
# nvme0n1: r/s=3000 w/s=500 await=45ms %util=99%
# The disk is handling maximum IOPS — new requests queue

# In AWS:
# gp3 default: 3000 IOPS, 125 MB/s throughput
# If workload needs 5000 IOPS → requests queue → iowait

# Check EBS CloudWatch:
# VolumeQueueLength > 1 sustained → IOPS saturated
# BurstBalance hitting 0 (gp2) → burst depleted
```

**2. Throughput saturation (large sequential I/O):**
```bash
iostat -xz 1 5
# nvme0n1: rMB/s=125 wMB/s=0 await=25ms
# Hitting the 125 MB/s gp3 throughput limit!

# Common cause: Large backup, log shipping, database dump
# Fix: Increase gp3 throughput (up to 1000 MB/s) or use io2
```

**3. Swapping (memory → disk I/O):**
```bash
vmstat 1 5
# If si/so > 0 → system is swapping
# Swap lives on disk → contributes to iowait
# Fix: Add RAM or kill memory hog. Swap is last resort, not a feature.
```

**4. Filesystem journaling/sync heavy workload:**
```bash
# Applications calling fsync() frequently (databases, message queues)
strace -p <PID> -e trace=fsync,fdatasync -c
# High fsync count → each one waits for disk guarantee

# Fix: Faster disk (io2) or reduce fsync frequency (application config)
# PostgreSQL: synchronous_commit = off (for non-critical data)
```

**5. RAID rebuild or disk degradation:**
```bash
# Check if RAID array is rebuilding
cat /proc/mdstat    # Software RAID
megacli -LDInfo -Lall -aALL    # Hardware RAID

# Rebuild consumes massive I/O → everything else is slow
```

**6. Process creating millions of small files:**
```bash
# Check for runaway temp file creation
iotop -o    # Which process?
strace -p <PID> -e trace=open,write -c    # What's it doing?

# Common: log4j creating per-request temp files, or find scanning entire filesystem
```

**Diagnosis workflow:**
```bash
# Step 1: Confirm iowait
vmstat 1 5    # wa column > 20%

# Step 2: Which disk
iostat -xz 1 3    # %util and await per device

# Step 3: Which process
iotop -o    # Real-time I/O per process
pidstat -d 1 5    # Disk I/O stats per PID

# Step 4: What type of I/O
# Random small I/O (database) vs sequential large I/O (backup)
iostat -xz 1 | watch rrqm/s wrqm/s    # High merge = sequential
# No merge = random

# Step 5: Is the device at its limit?
# Compare actual IOPS vs device maximum
# gp3: check CloudWatch VolumeReadOps, VolumeWriteOps vs provisioned
```

**In Project 7 (rightsizing):**
- Instance with consistent high iowait → storage is the bottleneck, not compute
- Recommendation: upgrade gp3 IOPS/throughput ($0.065 per provisioned IOPS/month) rather than upsizing the instance
- Sometimes cheaper to move to io2 than to add IOPS piecemeal

---

### Q30: A process is consuming 100% CPU. How do you investigate without killing it?
**Project Reference:** Project 7 (Monitoring) / Project 3 (K8s node troubleshooting)
**Expected Depth:** Non-destructive investigation techniques, getting useful data before deciding to kill

**Answer:**

**Why not just kill it?**
In production, you need to understand the root cause BEFORE killing it:
- Will it happen again? (recurring bug vs one-time event)
- Is it doing useful work? (batch processing, GC, compilation)
- Killing might make things worse (corrupt data, trigger restart storm)

**Step 1: Identify and baseline (non-disruptive):**
```bash
# Confirm which process
top -bn1 -o %CPU | head -10
# Or: ps aux --sort=-%cpu | head -5

# How long has it been running?
ps -p <PID> -o pid,etime,pcpu,pmem,comm
# etime shows elapsed time — running for 2 days at 100%? Or just started?

# Is it a single thread or multi-threaded?
ps -p <PID> -T    # Show threads
top -H -p <PID>   # Thread view in top
# If one thread at 100% → likely infinite loop or deadlock-spin
# If all threads at 100% → legitimate heavy computation or runaway
```

**Step 2: What is it doing? (non-destructive profiling):**
```bash
# strace — system calls (slight overhead but non-destructive)
strace -p <PID> -c -t 10    # Summarize syscalls for 10 seconds
# "100% futex" → lock contention (threads spinning on locks)
# "100% read/write" → I/O loop
# "0 syscalls" → pure userspace computation (CPU-bound algorithm)

timeout 10 strace -p <PID> -e trace=write -s 200 2>&1 | head -20
# See what it's writing (are those useful outputs or garbage?)

# perf — CPU profiling (zero-copy, minimal overhead)
perf top -p <PID>    # Live function-level CPU usage
# Shows which FUNCTION is consuming CPU (e.g., regex_match, gc_sweep, sort)

perf record -p <PID> -g -- sleep 30    # Record 30s profile
perf report    # Analyze call stack

# /proc/PID/stack — kernel stack trace (instant snapshot)
cat /proc/<PID>/stack
# Shows if process is stuck in kernel (waiting for lock, I/O, etc.)

# /proc/PID/wchan — what it's waiting on
cat /proc/<PID>/wchan
```

**Step 3: Get application-level context:**
```bash
# For Java — thread dump (non-destructive, app continues)
kill -3 <PID>    # Sends SIGQUIT → Java prints thread dump to stdout
# Or: jstack <PID> > /tmp/thread_dump.txt
# Or: jcmd <PID> Thread.print

# For Python — attach and get traceback
py-spy dump --pid <PID>    # Non-invasive Python profiler

# For any process — GDB backtrace
gdb -batch -ex "thread apply all bt" -p <PID> > /tmp/backtrace.txt
# WARNING: Brief pause (~1s). Non-destructive but not zero-impact.

# Check what files it has open
ls -la /proc/<PID>/fd/ | wc -l    # File descriptor count
lsof -p <PID> | tail -20          # What files/sockets open
```

**Step 4: Reduce impact without killing (if needed):**
```bash
# Lower priority (renice) — give it less CPU
renice +19 -p <PID>    # Lowest priority (other processes get CPU first)
# Process still runs but doesn't starve others

# CPU pinning (if multi-core) — confine to one core
taskset -p -c 0 <PID>    # Pin to CPU 0 only
# Other cores freed for other work

# I/O priority
ionice -c 3 -p <PID>    # Idle I/O class (only gets I/O when nobody else needs it)

# cgroups (most powerful — limit to X% CPU)
# Using systemd-run for ad-hoc limiting:
systemd-run --scope --slice=containment -p CPUQuota=25% -p MemoryMax=1G \
    --uid=$(ps -p <PID> -o uid= | tr -d ' ') -- sleep infinity &
# Then move process to that cgroup
echo <PID> > /sys/fs/cgroup/system.slice/containment.slice/cgroup.procs
```

**Step 5: Decision tree:**
```
Is it doing useful work? (batch job, GC, compilation)
├── YES → Let it finish, just renice/cgroup limit it
└── NO (stuck/looping) →
    ├── Can we get a core dump for debugging?
    │   gcore <PID>    # Generates core dump without killing
    └── Kill gracefully
        kill -TERM <PID>; sleep 10; kill -KILL <PID>
```

---

### Q31: DNS resolution works with `dig` but application still can't connect. Why?
**Project Reference:** Project 9 (Post-patch connectivity validation) / Project 2 (3-Tier networking)
**Expected Depth:** Difference between system resolver and dig, nsswitch.conf, /etc/hosts, library-level DNS

**Answer:**

**The key insight:** `dig` and `nslookup` bypass the system resolver. They directly query DNS servers. Applications use the system resolver (glibc/nsswitch), which is different.

**Common causes:**

**1. /etc/nsswitch.conf ordering — hosts file checked FIRST:**
```bash
cat /etc/nsswitch.conf | grep hosts
# hosts: files dns myhostname
#        ↑     ↑
#        |     └── Then DNS
#        └── /etc/hosts checked FIRST

# If /etc/hosts has a STALE entry:
cat /etc/hosts
# 10.0.1.50  api.internal.com    ← Old IP! DNS returns 10.0.2.100 but app uses this!

# dig queries DNS directly → returns 10.0.2.100 (correct)
# Application uses glibc → reads /etc/hosts first → gets 10.0.1.50 (wrong!)

# Fix: Remove stale /etc/hosts entry
# Prevention: Don't hardcode in /etc/hosts unless necessary
```

**2. /etc/resolv.conf issues:**
```bash
cat /etc/resolv.conf
# nameserver 10.0.0.2
# search us-east-1.compute.internal

# Possible problems:
# - Wrong nameserver (after VPC change or DHCP renewal)
# - search domain causes unexpected resolution
#   "db" resolves as "db.us-east-1.compute.internal" (not what app expects)
# - ndots:5 (default in K8s!) → any name with <5 dots searches all domains first

# Application looks up "api.payments.svc" (2 dots < ndots:5)
# System tries: api.payments.svc.us-east-1.compute.internal → NXDOMAIN
#               api.payments.svc.ec2.internal → NXDOMAIN
# ... finally tries api.payments.svc → SUCCESS (but took 10+ seconds!)

# dig skips all this — queries exactly what you ask
```

**3. DNS caching (systemd-resolved or nscd):**
```bash
# Is there a local caching resolver?
systemctl status systemd-resolved
systemctl status nscd

# Cached stale record:
resolvectl statistics    # Show cache hit/miss
resolvectl flush-caches  # Clear cache

# nscd cache (common on RHEL):
nscd -i hosts    # Invalidate hosts cache

# dig bypasses all caches → shows fresh result
# Application uses cached (stale) result
```

**4. IPv4 vs IPv6 resolution order:**
```bash
# DNS returns both A (IPv4) and AAAA (IPv6) records
dig api.example.com A        # 10.0.1.50
dig api.example.com AAAA     # fd00::1

# Application (glibc) prefers IPv6 by default!
# If IPv6 networking isn't configured → connection fails/times out
# Then falls back to IPv4 (maybe, after timeout)

# dig only queries what you ask (A record) → looks fine
# Application tries AAAA first → times out → appears broken

# Fix (/etc/gai.conf — address selection):
echo "precedence ::ffff:0:0/96 100" >> /etc/gai.conf    # Prefer IPv4

# Or disable IPv6 if not needed:
sysctl -w net.ipv6.conf.all.disable_ipv6=1
```

**5. Application-specific DNS (ignores system resolver):**
```bash
# Java: Uses its own DNS resolver with its own cache
# Default positive cache: 30 seconds (or FOREVER if security manager installed!)
# Check: -Dsun.net.inetaddr.ttl=30 -Dsun.net.inetaddr.negative.ttl=10

# Docker/containers: /etc/resolv.conf inside container differs from host
docker exec container cat /etc/resolv.conf    # Often 127.0.0.11 (Docker DNS)

# curl vs application: curl might use c-ares (different resolver library)
```

**6. Firewall blocking application but not dig:**
```bash
# dig uses UDP/53 from an unprivileged port
# But if your APP is trying to connect to the resolved IP on port 443/80
# The DNS works fine, but the actual CONNECTION is blocked!

# Test actual connectivity (not just DNS):
# This is what the application actually needs:
nc -zv api.example.com 443
curl -v https://api.example.com

# vs just DNS:
dig api.example.com    # Only tests name resolution!
```

**Diagnosis workflow:**
```bash
# 1. What does the system resolver actually return?
getent hosts api.example.com    # Uses nsswitch (same as application)
# Compare with:
dig +short api.example.com      # Direct DNS query

# 2. If different → check /etc/hosts, nsswitch.conf, caches
# 3. If same → problem isn't DNS, it's connectivity to the resolved IP
#    → Check: nc -zv <resolved_ip> <port>
#    → Check security groups, NACLs, firewalls
```

---

### Q32: Patching completed but application health check is failing. Give me your systematic approach.
**Project Reference:** Project 9 (Enterprise OS Patching — this IS the scenario we built for)
**Expected Depth:** Full methodology matching the 7-dimension validation, real troubleshooting, experience-driven

**Answer:**

**Context:** In Project 9, our 7-dimension validation caught this. But let me walk through how I'd debug it systematically if investigating manually.

**Phase 1: Characterize the failure (30 seconds)**
```bash
# What does the health check actually return?
curl -v http://localhost:8080/health
# 200 OK → health check endpoint works locally (problem is between LB and app)
# 503 → app reports unhealthy (check app-specific health)
# Connection refused → app not listening
# Timeout → app hung

# Check from ALB's perspective
aws elbv2 describe-target-health --target-group-arn <arn>
# State: unhealthy, Reason: "Elb.HealthCheckTimeout" or "Target.ResponseCodeMismatch"
```

**Phase 2: What changed during patching? (most probable causes)**

```bash
# 1. Was the application's service restarted?
systemctl status myapp
# If inactive → service didn't start after reboot
# If active but unhealthy → started but broken

# 2. Were any application dependencies updated?
# Check if OpenSSL, glibc, shared libraries changed
rpm -qa --last | head -20    # Recently installed packages

# Library compatibility issue?
ldd /opt/myapp/bin/myapp | grep "not found"
# If a shared library was updated and app links to old version → crash

# 3. Was a config file replaced by package update?
find /etc -name "*.rpmnew" -newer /tmp/patch_start_marker
find /etc -name "*.rpmsave" -newer /tmp/patch_start_marker
# .rpmnew = package wanted to replace your config but left yours intact
# .rpmsave = package DID replace your config, saved old one
# Either way: diff them and merge

# 4. SELinux context issue?
getenforce
ausearch -m avc --start today | audit2why
# Package update changed SELinux policy → app blocked from accessing port/file

# 5. Kernel change affecting application?
uname -r    # New kernel?
# Known issues: Docker overlay2 with certain kernels, iptables changes in new kernel,
# some apps have specific sysctl requirements
```

**Phase 3: Application-level investigation**

```bash
# Check application logs (most informative)
journalctl -u myapp --since "30 min ago" | grep -iE 'error|fail|exception|timeout'

# Common post-patch application failures:
# a) "Cannot connect to database on port 5432"
#    → Was PostgreSQL patched and needs restart? Or config changed?
systemctl status postgresql
pg_isready

# b) "SSL handshake failed"  
#    → OpenSSL updated, cipher suite changed
#    → Certificate path changed
#    → TLS version deprecated in new OpenSSL (TLS 1.0/1.1 removed!)
openssl s_client -connect localhost:8443

# c) "Permission denied: /var/run/myapp/myapp.sock"
#    → Reboot cleared tmpfs
#    → SELinux denial

# d) "Too many open files"
#    → PAM limits changed by package update
cat /proc/<PID>/limits | grep "open files"
```

**Phase 4: Network-level (if health check times out)**

```bash
# Is iptables/firewalld blocking the health check?
iptables -L -n | grep 8080
firewall-cmd --list-all

# Did patching add/change firewall rules?
# (firewalld package update can reset rules!)

# Can the ALB reach the instance?
# Check security group allows ALB's subnet on port 8080
# Check NACL allows return traffic

# Test from instance to itself (rules out app issue):
curl http://$(hostname -I | awk '{print $1}'):8080/health
```

**Phase 5: The systematic checklist (our Project 9 validation dimensions)**

| # | Check | Command | Pass criteria |
|---|-------|---------|---------------|
| 1 | Service running | `systemctl is-active myapp` | active |
| 2 | Port listening | `ss -tlnp \| grep :8080` | LISTEN |
| 3 | Local health | `curl -s localhost:8080/health` | 200 |
| 4 | Dependencies reachable | `nc -zv db.internal 5432` | Connected |
| 5 | Disk space | `df -h \| awk '$5+0>90'` | None |
| 6 | Memory available | `free -m` (available > 10%) | Enough |
| 7 | No critical errors | `journalctl -p err --since "30m ago"` | Empty |
| 8 | Correct file permissions | `ls -la /opt/myapp/` | Expected |
| 9 | SELinux not blocking | `ausearch -m avc --start today` | Clean |
| 10 | Config intact | `diff config current vs backup` | Match |

**Phase 6: Resolution and prevention**
```bash
# Once identified, fix and verify:
# Example: OpenSSL update removed TLS 1.1 → app tried TLS 1.1 to DB → failed
# Fix: Update app config to use TLS 1.2+
# OR: Pin OpenSSL version in patch exclusion list

# In Project 9, we prevent this:
# - exclude_packages list for known-sensitive packages
# - Staging patch first (same AMI) → catch issues before production
# - Rollback: restore from pre-patch AMI snapshot
# - Post-mortem: add new validation check for the scenario we missed
```

**The 12 YOE difference:**
A junior checks if the app is running. A senior checks the entire chain: service → port → local connectivity → dependencies → configuration → library compatibility → security context → network path. And has automation that does all of this in 60 seconds.

---
