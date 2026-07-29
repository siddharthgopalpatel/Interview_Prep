## CATEGORY 11: REAL-WORLD SCENARIO QUESTIONS (10 Questions)

---

### Q23: A production server is suddenly very slow. Walk me through your troubleshooting.
**Project Reference:** Project 9 (OS Patching — post-patch performance issues) / Project 7 (Monitoring)
**Expected Depth:** Structured methodology (USE method), systematic not random, show experience-based intuition

**Answer:**

**My approach follows the USE method: Utilization, Saturation, Errors — for each resource.**

**First 30 seconds — situational awareness:**
```bash
# 1. Who reported it? What's "slow"? (latency? throughput? specific function?)
# 2. Did anything change? (deployment, patch, scaling event, traffic spike?)

# Quick system overview (these are muscle memory)
uptime                    # Load average — first signal
dmesg | tail -20          # Kernel messages (OOM kills, hardware errors?)
vmstat 1 3                # CPU, memory, swap, I/O at a glance
free -h                   # Memory state
df -h                     # Disk full?
top -bn1 | head -20       # What's consuming resources
```

**Layer 1 — CPU:**
```bash
top -bn1 | head -5        # us% vs sy% vs wa% vs st%
# High us% → application consuming CPU (expected or runaway?)
# High sy% → kernel overhead (too many syscalls, bad I/O pattern)
# High wa% → waiting for disk → jump to I/O investigation
# High st% → hypervisor steal → cloud issue (rightsize or migrate)

# If CPU is the problem:
pidstat -u 1 5            # Which process?
```

**Layer 2 — Memory:**
```bash
free -h                   # Is available low?
vmstat 1 3                # si/so non-zero? (swapping = death for performance)
# If swapping:
ps aux --sort=-%mem | head -5    # Memory hog?
```

**Layer 3 — Disk I/O:**
```bash
iostat -xz 1 3            # %util, await, r/s, w/s
# %util > 90% → disk saturated
# High await → requests queuing
iotop -o                  # Which process?
```

**Layer 4 — Network:**
```bash
ss -s                     # Connection summary
ss -tnp | wc -l          # Total established connections
dmesg | grep -i 'nf_conntrack: table full'   # Connection tracking overflow
sar -n DEV 1 3            # Network throughput and errors
```

**Layer 5 — Application-specific:**
```bash
# Check application logs
journalctl -u myapp --since "10 min ago" | tail -50
# Look for: connection pool exhausted, timeout, queue full, OOM

# Check application health endpoint
curl -w "%{time_total}\n" http://localhost:8080/health
# If slow → internal bottleneck (DB connection, downstream service)
```

**Common root causes I've seen (12 YOE perspective):**
1. **Most common:** Disk full → app can't write logs/temp → hangs
2. **Second most:** Memory leak → swapping → everything slow
3. **Third:** Upstream dependency slow (DB query, external API) → threads pool exhausted → all requests queue
4. **Fourth:** DNS resolution failing → 5-second timeout per request → appears slow
5. **Fifth:** After patching (Project 9) — new kernel has regression, or service didn't restart cleanly

---

### Q24: Application returns 502 after deployment. What do you check on the Linux host?
**Project Reference:** Project 2 (3-Tier AWS — ALB + EC2) / Project 1 (CI/CD deployment)
**Expected Depth:** 502 means proxy got bad/no response from upstream. Check application is running and reachable.

**Answer:**

**502 Bad Gateway means:** The load balancer (ALB/nginx) connected to the backend but got an invalid response or connection refused. The problem is on the backend host.

**Systematic check on the Linux host:**

**Step 1: Is the application process running?**
```bash
systemctl status myapp
# Active: failed (Result: exit-code) ← didn't start!
# Active: activating (auto-restart) ← crash-looping!

# If not running, why?
journalctl -u myapp --since "5 min ago" -n 50
# Look for: "Address already in use", "permission denied", "config file not found",
# "cannot connect to database", missing dependency
```

**Step 2: Is it listening on the expected port?**
```bash
ss -tlnp | grep ':8080'
# If empty → app isn't listening (crashed, wrong port, binding to wrong interface)

# Verify it's the RIGHT process
ss -tlnp | grep ':8080'
# LISTEN  0  128  *:8080  users:(("java",pid=12345,fd=55))
```

**Step 3: Can I reach it locally?**
```bash
curl -v http://localhost:8080/health
# If this works → problem is between ALB and instance (security group, network)
# If this fails → problem is the application itself
# If connection refused → not listening
# If timeout → process is hung (deadlock, thread pool exhausted)
```

**Step 4: Check for port conflicts (after deployment):**
```bash
# Old process still holding the port?
ss -tlnp | grep ':8080'
# If PID belongs to old version → graceful shutdown failed
kill -TERM <old_pid>; sleep 5; systemctl start myapp
```

**Step 5: Check resources:**
```bash
# Disk full? (can't write PID file, temp files, logs)
df -h
# Memory? (OOM killed immediately on start?)
dmesg | grep -i oom | tail -5
# Permissions? (new binary has wrong ownership)
ls -la /opt/myapp/
```

**Step 6: Deployment-specific checks:**
```bash
# Did the new artifact actually deploy?
ls -la /opt/myapp/app.jar    # Correct version? Recent timestamp?
md5sum /opt/myapp/app.jar    # Match expected hash?

# Config file correct? (new version might need new config keys)
cat /opt/myapp/config.yml | grep -i database

# Environment variables set?
cat /etc/systemd/system/myapp.service | grep Environment
systemctl show myapp --property=Environment
```

**In Project 2 (ALB context):**
- ALB health check hits `/health` on port 8080
- If app returns non-200 (or times out in 5s) for 3 consecutive checks → marked unhealthy → 502
- Check ALB target group health: `aws elbv2 describe-target-health --target-group-arn <arn>`
- Common after deployment: app takes 30s to initialize but health check timeout is 5s → always fails → 502

**Fix for slow startup:** Add `initialDelaySeconds` equivalent — in ALB, increase health check `interval` and `unhealthy_threshold` during deployments. Or add readiness endpoint that returns 503 until fully initialized.

---

### Q25: Server ran out of disk space at 3 AM. What happened and how do you prevent it?
**Project Reference:** Project 9 (OS Patching — disk validation) / Project 7 (Monitoring)
**Expected Depth:** Common culprits, diagnosis, immediate fix, long-term prevention

**Answer:**

**Immediate triage:**
```bash
# Which filesystem is full?
df -h
# /dev/nvme0n1p1  50G   50G    0  100% /
# /dev/nvme1n1   200G  198G  2.0G  99% /data

# What's consuming space? (find large directories)
du -sh /* 2>/dev/null | sort -rh | head -10
du -sh /var/log/* | sort -rh | head -10

# Find files created/modified recently (the culprit)
find / -xdev -type f -mtime -1 -size +100M -exec ls -lh {} + 2>/dev/null
```

**Top culprits (in my 12 years of experience):**

| Rank | Culprit | How to verify |
|------|---------|---------------|
| 1 | Application logs not rotated | `du -sh /var/log/myapp/` → 40GB |
| 2 | Deleted file still held open | `lsof +D /var/log \| grep deleted` |
| 3 | /tmp or /var/tmp explosion | `du -sh /tmp/` |
| 4 | Core dumps | `find / -name "core.*" -size +1G` |
| 5 | Docker/container images/layers | `docker system df` |
| 6 | Journal logs unbounded | `journalctl --disk-usage` |
| 7 | Old kernels filling /boot | `du -sh /boot/` |
| 8 | Package manager cache | `du -sh /var/cache/dnf/` |

**The "deleted file still open" trap:**
```bash
# A process has a 30GB log file open, you deleted it, but space isn't freed!
lsof | grep deleted
# java   12345 appuser  4w REG 259,1 32000000000 /var/log/app.log (deleted)

# Fix: Truncate instead of delete (if still needed):
: > /var/log/app.log    # Truncates to 0 without closing the fd

# Or kill/restart the process:
systemctl restart myapp   # Releases the file descriptor
```

**Immediate relief:**
```bash
# Clear package cache
dnf clean all

# Truncate large logs (don't delete if process has it open!)
truncate -s 0 /var/log/huge-app.log

# Clear old journal logs
journalctl --vacuum-size=500M

# Remove old kernels (keep last 2)
dnf remove --oldinstallonly --setopt installonly_limit=2 kernel

# Clear Docker
docker system prune -af --volumes
```

**Long-term prevention (what I implemented in Projects 7/9):**

```bash
# 1. Log rotation (logrotate)
cat /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    maxsize 500M
    missingok
    notifempty
    copytruncate        # Don't rename — truncate (avoids deleted-but-open issue)
}

# 2. Journal size limit
cat /etc/systemd/journald.conf
SystemMaxUse=1G
SystemKeepFree=2G

# 3. Monitoring alert (before it's too late)
# Alert at 80%, critical at 90%
# In Project 9 validation: df check fails if any mount > 90%

# 4. Auto-cleanup cron
0 2 * * * find /tmp -mtime +7 -delete
0 3 * * 0 docker system prune -f

# 5. Separate partitions for /var/log, /tmp, /data
# So a log explosion doesn't kill root filesystem
```

**In Project 9:** Our post-patch validation checks `df -h` on all mounts. If any exceed 90%, it's flagged. We also verify `/boot` has space before installing kernel updates (full /boot = failed kernel install = partial patch = bad state).

---

### Q26: SSH connection drops after 30 seconds. What's wrong?
**Project Reference:** Project 9 (OS Patching — SSH is the connection method) / Project 2 (EC2 access)
**Expected Depth:** Multiple causes, systematic diagnosis, not just "it's the firewall"

**Answer:**

**Common causes (ordered by frequency in my experience):**

**1. TCP keepalive / idle timeout (most common in cloud):**
```bash
# NAT gateways, firewalls, and ALBs drop idle TCP connections (typically 350s for AWS NAT, but some are aggressive)
# If nothing traverses the connection for X seconds → stateful firewall drops it → next packet = RST or blackhole

# Fix (client side — ~/.ssh/config):
Host *
    ServerAliveInterval 15    # Send keepalive every 15s
    ServerAliveCountMax 3     # Disconnect after 3 missed responses

# Fix (server side — /etc/ssh/sshd_config):
ClientAliveInterval 15
ClientAliveCountMax 3
```

**2. Security Group or NACL issue:**
```bash
# NACLs are stateless — need explicit ALLOW for return traffic (ephemeral ports)
# If outbound rule doesn't allow responses on ephemeral ports → connection appears to work then drops

# Check: Does the NACL allow outbound on ports 1024-65535?
aws ec2 describe-network-acls --filters "Name=association.subnet-id,Values=subnet-xxx"
```

**3. TCP timeout (network path issue):**
```bash
# MTU mismatch — jumbo frames not supported across path
# Packets > path MTU get silently dropped (if DF bit set and ICMP blocked)
ping -M do -s 1472 target_host    # Test MTU (1472+28=1500 standard)
# If this fails, try lower sizes

# Fix: Reduce MTU on instance
ip link set dev eth0 mtu 1400
```

**4. DNS resolution timeout (if using hostname):**
```bash
# SSH tries reverse DNS on connecting IP — if DNS is slow, connection feels hung
# Then sshd does UseDNS which causes delay

# Fix (server — /etc/ssh/sshd_config):
UseDNS no

# Also check /etc/resolv.conf on the server — broken DNS can stall SSH
```

**5. Resource exhaustion on server:**
```bash
# Too many open files
cat /proc/sys/fs/file-nr    # allocated  free  maximum
# If allocated near maximum → sshd can't open new fds

# PAM issues (slow LDAP/SSSD lookup)
grep -i "pam" /var/log/secure    # Timeout connecting to LDAP?

# MaxSessions/MaxStartups in sshd_config
grep -i "max" /etc/ssh/sshd_config
# MaxStartups 10:30:60    # Start dropping at 10 unauthenticated connections
```

**6. After patching (Project 9 specific):**
```bash
# sshd_config got replaced by package update
diff /etc/ssh/sshd_config /etc/ssh/sshd_config.rpmnew

# SSH host keys regenerated → client rejects (strict host checking)
# Client sees: WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!

# New firewalld rules activated (if firewalld was updated)
firewall-cmd --list-all
```

**Diagnosis approach:**
```bash
# From client — verbose SSH shows where it hangs:
ssh -vvv user@host
# Watch for: "debug1: expecting SSH2_MSG_KEX_ECDH_REPLY" (network/timeout issue)
# Or: "debug1: Authentications that can continue" (auth phase issue)

# If connection establishes then drops while idle:
# → Keepalive issue (firewall dropping idle connection)

# If connection drops during transfer:
# → MTU issue or rate limiting

# If connection drops immediately after auth:
# → Shell issue (/bin/bash missing, quota exceeded, /etc/nologin exists)
```

---

### Q27: A cron job ran successfully for months but suddenly stopped. How do you troubleshoot?
**Project Reference:** Project 9 (OS Patching — scheduled execution) / Project 7 (Scheduled Lambda/cron tasks)
**Expected Depth:** Systematic cron debugging, common traps, environment differences

**Answer:**

**Step 1: Verify it's actually not running:**
```bash
# Check cron logs
grep CRON /var/log/cron         # RHEL/CentOS
grep cron /var/log/syslog       # Ubuntu
journalctl -u crond --since "24 hours ago"

# Is crond itself running?
systemctl status crond
# Did cron daemon crash after patching?
```

**Step 2: Is the crontab still there?**
```bash
# User crontab
crontab -l -u appuser

# System crontab
cat /etc/crontab
ls -la /etc/cron.d/

# Was it accidentally deleted? (common after user account changes)
# Check if file permissions changed
ls -la /var/spool/cron/appuser
```

**Step 3: Was it recently modified?**
```bash
# Check crontab modification time
stat /var/spool/cron/appuser

# Check audit log (if auditd configured)
ausearch -f /var/spool/cron/
```

**Step 4: The classic cron environment trap:**
```bash
# Cron runs with MINIMAL environment — not your login shell!
# It does NOT source ~/.bashrc, ~/.profile, /etc/profile.d/*

# What cron sees:
# PATH=/usr/bin:/bin (NOT /usr/local/bin, NOT /opt/app/bin)
# No AWS credentials, no JAVA_HOME, no custom vars

# The job probably uses a command in /usr/local/bin or needs env vars

# Fix: Set full path and environment in crontab
PATH=/usr/local/bin:/usr/bin:/bin:/opt/app/bin
JAVA_HOME=/usr/lib/jvm/java-17
AWS_REGION=us-east-1

0 2 * * * /usr/local/bin/python3 /opt/scripts/backup.py

# Or source profile in the command:
0 2 * * * source /etc/profile.d/app.sh && /opt/scripts/backup.sh
```

**Step 5: Check why it's failing (not just "not running"):**
```bash
# Cron emails output to the user — but email is often not configured
# Add explicit logging:
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup_cron.log 2>&1

# Check disk space (can't write temp files/output)
df -h

# Check if the script's dependencies changed:
# - Database credentials rotated?
# - SSL certificate expired?
# - Remote API changed endpoint?
# - Package updated removed a utility?
which aws      # Still exists?
aws sts get-caller-identity    # Still authenticated?
```

**Step 6: Common "suddenly stopped" causes:**

| Cause | How to verify | Fix |
|-------|---------------|-----|
| PATH not set in cron | `which <command>` vs what's in PATH | Use absolute paths |
| Disk full | `df -h` | Clear space, add monitoring |
| Permission changed | `ls -la /opt/scripts/backup.sh` | `chmod +x` |
| SSL/cert expired | `curl -v https://api.com` | Renew certificate |
| Credential rotation | Test authentication | Update credentials |
| After patching (selinux relabel) | `getenforce` + audit.log | `restorecon` or fix policy |
| User account locked/expired | `chage -l appuser` | Unlock account |
| Resource limits hit | `ulimit -a` (cron's limits from PAM) | Adjust /etc/security/limits.d/ |
| DNS changed | `dig api.internal` | Update /etc/hosts or fix DNS |

**Best practice (what I do in Project 9 for scheduled automation):**
- Always use absolute paths in cron entries
- Always redirect stdout AND stderr to a log file
- Include a health check: the cron job writes a timestamp to a file, monitoring alerts if file is >25h old
- Use `flock` to prevent overlapping runs: `flock -n /tmp/backup.lock /opt/scripts/backup.sh`

---
