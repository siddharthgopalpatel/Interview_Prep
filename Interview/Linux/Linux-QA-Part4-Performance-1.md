## CATEGORY 10: PERFORMANCE TROUBLESHOOTING & MONITORING (8 Questions)

---

### Q15: High CPU usage — how do you identify the culprit process? Walk through top, pidstat, strace.
**Project Reference:** Project 7 (Cost Optimization — CPU monitoring for rightsizing) / Project 3 (K8s node health)
**Expected Depth:** Practical workflow, understanding us/sy/wa/st, drilling down to function level

**Answer:**

**Step 1: High-level view with `top` (or `htop`):**
```bash
top -bn1 | head -20    # Batch mode, one iteration

# Key CPU line:
# %Cpu(s): 85.2 us, 10.1 sy, 0.0 ni, 3.5 id, 0.0 wa, 0.0 hi, 1.0 si, 0.2 st
#           │        │                    │         │                      │
#           │        │                    │         │                      └── steal (VM)
#           │        │                    │         └── iowait (blocked on disk)
#           │        │                    └── idle
#           │        └── system (kernel mode — syscalls, interrupts)
#           └── user (application code)

# Press '1' in top to see per-CPU breakdown
# Press 'P' to sort by CPU usage
# Press 'H' to show threads

# Interpretation:
# High us%: Application doing heavy computation (your code)
# High sy%: Too many syscalls, context switches, or kernel operations
# High wa%: Processes waiting for disk I/O
# High st%: Hypervisor stealing CPU time (noisy neighbor or undersized VM)
```

**Step 2: Identify the specific process with `pidstat`:**
```bash
# Per-process CPU breakdown (every 2 seconds, 5 samples)
pidstat -u 2 5

# Specific process with threads
pidstat -u -t -p <PID> 1

# Which threads are consuming CPU
pidstat -t -p $(pgrep java) 1 5

# Example output:
# PID   %usr  %system  %CPU  Command
# 12345  92.0    5.0    97.0  python3
```

**Step 3: What is the process doing? — `strace`:**
```bash
# Attach to running process (see system calls)
strace -p <PID> -c              # Summary: which syscalls take most time
strace -p <PID> -e trace=read,write  # Filter specific syscalls
strace -p <PID> -f -tt          # Follow forks, with timestamps

# Example output of strace -c:
# % time  seconds   calls  syscall
# 78.32   0.145232  45231  futex       ← Lock contention!
# 15.21   0.028201   2103  read
#  4.12   0.007632    891  write
```

**Step 4: CPU profiling (for deep analysis):**
```bash
# perf — kernel-level CPU profiler
perf top                         # Live view (like top for functions)
perf record -p <PID> -g -- sleep 30   # Record 30s profile
perf report                      # Analyze recorded profile

# Generate flame graph (visual CPU profile)
perf record -F 99 -p <PID> -g -- sleep 60
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg
```

**Real workflow in Project 7 (rightsizing):**
1. CloudWatch shows instance averaging 85% CPU → alert fires
2. SSH in → `top` → find it's the Java app at 90% CPU
3. `pidstat -t -p <PID>` → identify hot threads (GC thread? Worker thread?)
4. If GC: check heap settings (`-Xmx`), consider rightsizing UP or fixing memory leak
5. If worker: the app genuinely needs more CPU → rightsize to c-family (compute-optimized)
6. If `sy%` is high: too many context switches → `pidstat -w` shows voluntary/involuntary switches

**In Project 3 (K8s node):**
- Node at 90% CPU → `kubectl top nodes` → `kubectl top pods` → identify pod
- Check if pod has CPU limits → being throttled (`cat /sys/fs/cgroup/cpu/cpu.stat` → `nr_throttled`)
- Decision: increase limits, add HPA, or rightsize the node

---

### Q16: Explain the `free` command output — what are buffers vs cache, and when should you worry about swap?
**Project Reference:** Project 7 (Cost Optimization — memory monitoring) / Project 9 (Post-patch validation)
**Expected Depth:** Correct interpretation (available != free), buffer/cache reclaimable, swap implications

**Answer:**

**Understanding `free -h` output:**
```
              total    used    free    shared  buff/cache   available
Mem:           16G     6.2G    512M    128M    9.3G         9.1G
Swap:          2G      200M    1.8G
```

**Critical insight: "free" is misleading. Look at "available" instead.**

| Column | What it means |
|--------|--------------|
| total | Physical RAM installed |
| used | RAM actively used by applications (NOT including cache) |
| free | Completely unused RAM (wasteful if high!) |
| shared | Used by tmpfs (shared memory segments) |
| buff/cache | RAM used for disk caching (reclaimable!) |
| **available** | **RAM available for new applications (free + reclaimable cache)** |

**Buffers vs Cache:**
- **Buffers:** Metadata about disk blocks (directory entries, inodes, block device cache). Small, always present.
- **Cache (Page Cache):** File content cached in RAM for faster reads. This is the big one.
- **Both are GOOD** — Linux uses all available RAM for caching. It's not "used up."
- **Both are reclaimable** — if an application needs RAM, kernel drops cache pages instantly.

```bash
# Detailed breakdown
cat /proc/meminfo | head -20

# Key entries:
# MemTotal:       16384000 kB
# MemFree:          524288 kB    ← Truly unused
# MemAvailable:    9437184 kB    ← What you can actually use
# Buffers:          204800 kB    ← Block device metadata cache
# Cached:          9011200 kB    ← File content cache (page cache)
# SwapTotal:       2097152 kB
# SwapFree:        1843200 kB
# Dirty:             8192 kB     ← Cache pages waiting to be written to disk
# Shmem:           131072 kB     ← Shared memory (tmpfs, IPC)
```

**When to worry:**
```bash
# GOOD: Low free, high buff/cache, high available
# Linux is efficiently using RAM for caching. Applications have plenty available.
Mem: 16G used=6G free=512M buff/cache=9.3G available=9.1G  ✅

# BAD: Low free, low available (most RAM truly consumed)
# Applications are using most RAM. New allocations may fail or trigger OOM.
Mem: 16G used=14G free=200M buff/cache=1.8G available=1.5G  ⚠️

# CRITICAL: Swap actively being used + high swap I/O
Swap: 2G used=1.8G  ← System is swapping = SLOW
```

**Swap — when to worry:**
```bash
# Check swap usage
swapon --show
free -h | grep Swap

# Check swap I/O (are we actively swapping in/out?)
vmstat 1 5
# si (swap in) and so (swap out) columns
# si/so = 0: swap exists but not actively used (fine)
# si/so > 0 sustained: actively paging = performance problem!

# What's using swap?
for pid in $(ls /proc/[0-9]*/status); do
    awk '/VmSwap/{if($2>0) print FILENAME, $0}' "$pid" 2>/dev/null
done | sort -k3 -rn | head -10

# Or simpler:
smem -rs swap | head -10
```

**In Project 7 (rightsizing decisions):**
- Instance with `available` < 20% of total → undersized, recommend larger instance
- Instance with `available` > 70% of total → oversized, recommend smaller
- Swap used > 0 consistently → application needs more RAM than available → upsize

**In Project 9 (post-patch validation):**
```bash
# Our validation checks:
free -m | awk '/^Mem:/{if($7/$2*100 < 10) print "WARN: Available memory below 10%"}'
swapon --show | awk 'NR>1{if($4+0 > 50) print "WARN: Swap usage above 50%"}'
```

---

### Q17: How do you identify a disk I/O bottleneck using iostat and iotop?
**Project Reference:** Project 7 (Cost Optimization) / Project 9 (Health validation)
**Expected Depth:** Reading iostat columns, %util vs await, distinguishing random vs sequential, EBS implications

**Answer:**

**Step 1: Is I/O the problem? Check `iowait` in top/vmstat:**
```bash
vmstat 1 5
# procs -----memory----- --io-- -system- ------cpu-----
# r  b   swpd  free  buff  cache  bi   bo  in   cs  us sy id wa st
# 1  8   0   1024000 2048  8192  4500 2200 1500 3000  5  3 12 80  0
#    ↑                                                        ↑
#    8 processes blocked on I/O                               80% iowait!
```

**Step 2: Which disk? — `iostat -xz 1`:**
```bash
iostat -xz 1 5    # Extended stats, skip idle devices, every 1s, 5 samples

# Device    r/s    w/s   rMB/s  wMB/s  rrqm/s wrqm/s await r_await w_await %util
# nvme0n1  1500   800   45.0   12.0    0.0    50.0   8.5   5.2     15.1    95.2
#                                                     ↑                     ↑
#                                                     avg wait time         utilization
```

**Key columns explained:**

| Column | Meaning | Healthy | Problem |
|--------|---------|---------|---------|
| r/s, w/s | IOPS (reads/writes per second) | Within device limit | At device IOPS limit |
| rMB/s, wMB/s | Throughput | Within device limit | At throughput limit |
| await | Average I/O request time (ms) | <5ms (SSD/NVMe) | >20ms sustained |
| r_await, w_await | Read/write latency separately | <5ms | >20ms |
| %util | Device utilization | <70% | >90% sustained |
| avgqu-sz | Average queue length | <2 | >4 (I/Os backing up) |
| rrqm/s, wrqm/s | Merged requests (sequential I/O being batched) | Varies | — |

**Interpretation guide:**
```bash
# Scenario 1: High %util, low IOPS → small random I/O saturating single-threaded disk
# Fix: Use provisioned IOPS (io2), or move to instance store

# Scenario 2: High %util, high IOPS, high await → genuine overload
# Fix: Distribute I/O across volumes, upgrade volume type (gp3→io2)

# Scenario 3: High IOPS but low %util, low await → healthy high-throughput workload
# This is fine — device handling load well

# Scenario 4: Low %util but high await → volume throttled by burst credits
# EBS gp2/gp3 burst depleted. Check CloudWatch: VolumeQueueLength, BurstBalance
```

**Step 3: Which process? — `iotop`:**
```bash
sudo iotop -o    # Show only processes doing I/O

# Output:
# TID   PRIO  USER  DISK READ   DISK WRITE  COMMAND
# 15234 be/4  mysql  45.00 M/s   12.00 M/s  mysqld
# 8921  be/4  root    0.00 B/s   800.00 K/s  rsync

# Alternative if iotop not installed:
pidstat -d 1 5    # Disk I/O per process
```

**EBS-specific (Project 7 context):**
```bash
# Check EBS burst balance (gp2/gp3)
aws cloudwatch get-metric-statistics \
    --namespace AWS/EBS \
    --metric-name BurstBalance \
    --dimensions Name=VolumeId,Value=vol-xxx \
    --start-time $(date -u -d '-1 hour' +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300 --statistics Average

# If BurstBalance hits 0 → latency spikes dramatically
# Fix: Increase gp3 baseline IOPS (default 3000, max 16000)
# Or switch to io2 for consistent performance

# Check volume queue length (backlog)
# VolumeQueueLength > 1 sustained → volume can't keep up
```

**In Project 7 rightsizing:**
- High I/O wait + EBS burst depleted → recommend upgrading to gp3 with provisioned IOPS
- High I/O on temp data → recommend instance store (NVMe local disks) for ephemeral workloads
- Cost calculation: gp3 3000 IOPS = free, each additional 1000 IOPS = $65/month

---

### Q18: What do the three load average numbers mean, and how do they relate to CPU cores?
**Project Reference:** Project 3 (K8s Node Health) / Project 7 (Rightsizing)
**Expected Depth:** Not just "1, 5, 15 minutes" — understand runnable + uninterruptible, relationship to cores, when to act

**Answer:**

**The three numbers:**
```bash
uptime
# 14:32:01 up 45 days,  load average: 4.50, 3.20, 2.80
#                                       ↑     ↑     ↑
#                                      1min  5min  15min
cat /proc/loadavg
# 4.50 3.20 2.80 3/287 45231
#                 ↑     ↑
#        running/total  last PID
```

**What load average actually measures:**
Load average = average number of processes in the **run queue** (R state) + processes in **uninterruptible sleep** (D state) over 1, 5, and 15 minutes.

- **R (runnable):** Waiting for CPU time or currently running
- **D (uninterruptible sleep):** Waiting for I/O (disk, network) — CANNOT be killed

**Critical insight:** Load average includes I/O-waiting processes! A load of 8 on a 4-core system could mean:
- 4 processes using CPU + 4 processes waiting for disk (iowait)
- NOT necessarily CPU saturation

**Relationship to CPU cores:**
```bash
# How many cores do I have?
nproc    # or: grep -c ^processor /proc/cpuinfo

# Rules of thumb (for an N-core system):
# Load = N:     System is fully utilized (no spare capacity)
# Load < N:     System has spare capacity
# Load > N:     Processes are waiting (queued). Some won't get CPU immediately.
# Load > 2*N:   Significant saturation. Performance degradation likely.

# Example: 4-core instance
# Load 2.0:  50% utilized. Comfortable.
# Load 4.0:  100% utilized. No spare capacity. Still responsive.
# Load 8.0:  Overloaded. Processes waiting. Response time increasing.
# Load 16.0: Severely overloaded. System likely sluggish.
```

**Reading the trend (1-min vs 15-min):**
```bash
# load avg: 8.50, 3.20, 2.80
# 1min >> 15min → Load is INCREASING (recent spike, ongoing)
# Interpretation: Something just started consuming resources

# load avg: 2.10, 5.50, 8.20
# 1min << 15min → Load is DECREASING (recovering from past spike)
# Interpretation: Problem may be resolving itself

# load avg: 4.00, 4.10, 3.90
# All three similar → Sustained load (steady state)
# Interpretation: This is the normal operating level
```

**Load average does NOT tell you WHERE the problem is:**
```bash
# High load — is it CPU, I/O, or both?

# Step 1: Check iowait
vmstat 1 3
# If wa% > 20: load is from I/O waiters (D state), not CPU

# Step 2: Count processes in each state
ps aux | awk '{print $8}' | sort | uniq -c | sort -rn
# R = runnable (CPU demand)
# D = uninterruptible (I/O demand)
# S = sleeping (normal, not contributing to load)

# Step 3: Identify D-state processes (I/O waiters)
ps aux | awk '$8=="D" {print}'
# Or with process state info:
cat /proc/[0-9]*/status | grep -B1 "^State.*D"
```

**In Project 3 (K8s node monitoring):**
- We alert when 1-min load > (cores * 1.5) for > 5 minutes
- High load on K8s node → check if it's pod density (too many pods) or a single runaway pod
- Karpenter uses this signal — if node consistently overloaded, launch additional nodes

**In Project 7 (rightsizing):**
- Instance averaging load 0.3 on 4 cores → massively oversized → recommend 1-core instance
- Instance averaging load 3.8 on 4 cores → properly sized (running hot but functional)
- Instance averaging load 6.0 on 4 cores → undersized → recommend 8-core or investigate I/O

---
