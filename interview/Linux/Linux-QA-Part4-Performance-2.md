## CATEGORY 10: PERFORMANCE TROUBLESHOOTING & MONITORING — Continued (Questions 19-22)

---

### Q19: How do you interpret /proc/meminfo and /proc/cpuinfo? What key fields matter for operations?
**Project Reference:** Project 7 (Cost Optimization — rightsizing analysis) / Project 9 (Post-patch validation)
**Expected Depth:** Practical fields that drive decisions, not just listing everything

**Answer:**

**/proc/cpuinfo — understanding your compute:**
```bash
# How many logical CPUs (cores × threads)?
grep -c ^processor /proc/cpuinfo    # Quick count
nproc                                # Same result

# What CPU model?
grep "model name" /proc/cpuinfo | head -1
# model name : Intel(R) Xeon(R) Platinum 8275CL CPU @ 3.00GHz (m5 family)
# model name : AWS Graviton3 (c7g family)

# Physical cores vs hyperthreads
grep "cpu cores" /proc/cpuinfo | head -1      # Physical cores per socket
grep "siblings" /proc/cpuinfo | head -1       # Logical CPUs per socket (includes HT)
# If siblings > cpu cores → Hyperthreading is ON

# CPU flags that matter:
grep flags /proc/cpuinfo | head -1
# aes         → AES-NI hardware encryption (TLS performance)
# avx, avx2   → Vector instructions (ML/computation workloads)
# vmx/svm     → Virtualization support
# nx          → No-Execute bit (security)

# Cache size (affects performance characteristics)
grep "cache size" /proc/cpuinfo | head -1
```

**In Project 7 (rightsizing):**
- `model name` tells us instance family. If it's Intel 8124M → old gen, consider Graviton migration (20% cheaper, better perf)
- Thread count helps interpret load average correctly
- Cache size matters for compute-heavy workloads

**/proc/meminfo — understanding memory state:**
```bash
cat /proc/meminfo

# Critical fields for operations:
MemTotal:       16384000 kB   # Physical RAM (verify instance type match)
MemFree:          524288 kB   # Completely unused (misleading alone!)
MemAvailable:    9437184 kB   # Actually available for applications ← THIS ONE MATTERS
Buffers:          204800 kB   # Block device metadata cache
Cached:          9011200 kB   # Page cache (file content)
SwapTotal:       2097152 kB   # Swap space configured
SwapFree:        1843200 kB   # Swap available
Dirty:             12288 kB   # Pages modified in cache, not yet written to disk
Writeback:             0 kB   # Pages being actively written to disk
AnonPages:       5800000 kB   # Memory used by applications (heap, stack, mmap)
Mapped:           800000 kB   # Memory-mapped files (shared libraries, mmap)
Shmem:            131072 kB   # Shared memory (tmpfs, IPC shared segments)
Slab:             450000 kB   # Kernel data structures cache
SReclaimable:     350000 kB   # Slab that can be reclaimed (dentry/inode cache)
SUnreclaim:       100000 kB   # Slab that cannot be reclaimed
CommitLimit:    10000000 kB   # Total memory available for allocation (RAM+swap × overcommit ratio)
Committed_AS:    8500000 kB   # Memory requested by all processes (virtual)
HugePages_Total:       0      # Huge pages allocated (databases often use these)
```

**Key operational decisions from /proc/meminfo:**
```bash
# Is the machine memory-pressured?
awk '/MemAvailable/{avail=$2} /MemTotal/{total=$2} END{printf "%.0f%% available\n", avail/total*100}' /proc/meminfo
# Below 20% → alert. Below 10% → critical.

# Is swap being used?
awk '/SwapTotal/{t=$2} /SwapFree/{f=$2} END{printf "Swap used: %.0f%%\n", (t-f)/t*100}' /proc/meminfo

# High Dirty pages? (data at risk if crash)
awk '/Dirty/{print "Dirty pages:", $2/1024, "MB"}' /proc/meminfo
# High dirty → lots of writes pending. If VM crashes, this data is lost.

# High Committed_AS vs CommitLimit?
# If Committed_AS > CommitLimit → potential OOM even with free RAM (over-committed)
```

---

### Q20: How do you read and interpret `vmstat` output? What story does each column tell?
**Project Reference:** Project 9 (Post-patch health validation) / Project 7 (Rightsizing)
**Expected Depth:** Column-by-column interpretation, recognizing patterns, combining with other tools

**Answer:**

```bash
vmstat 1 5    # Sample every 1 second, 5 times
# First line is since-boot average (ignore it). Lines 2+ are real-time.

# procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
#  r  b   swpd    free    buff   cache   si   so    bi    bo    in   cs  us sy id wa st
#  2  0  51200  524288  204800 9011200    0    0   120   450  3200 5500  45 10 40  5  0
#  1  5  51200  520000  204800 9011200    0    0  4500  2200  1500 3000   5  3 12 80  0
```

**Column-by-column:**

| Section | Column | Meaning | Healthy | Problem |
|---------|--------|---------|---------|---------|
| procs | r | Runnable processes (waiting for CPU) | < nproc | >> nproc |
| procs | b | Blocked processes (waiting for I/O) | 0-1 | >4 sustained |
| memory | swpd | Swap used (KB) | 0 | Growing over time |
| memory | free | Free RAM (KB) | Low is OK if cache high | — |
| memory | buff | Buffer cache | Stable | — |
| memory | cache | Page cache | High = good | — |
| swap | si | Swap in (KB/s from disk to RAM) | 0 | >0 sustained = BAD |
| swap | so | Swap out (KB/s from RAM to disk) | 0 | >0 sustained = BAD |
| io | bi | Blocks read from disk/s | Varies | Very high + high wa |
| io | bo | Blocks written to disk/s | Varies | Very high + high wa |
| system | in | Interrupts/second | Varies | Sudden spike |
| system | cs | Context switches/second | <10000 | >50000 sustained |
| cpu | us | User CPU % | Normal workload | — |
| cpu | sy | System CPU % | <15% | >30% = kernel overhead |
| cpu | id | Idle % | >20% | — |
| cpu | wa | I/O wait % | <5% | >20% = I/O bottleneck |
| cpu | st | Steal % | 0% | >5% = hypervisor issue |

**Pattern recognition:**

```bash
# Pattern 1: CPU-bound
# r=8 (high), b=0, us=90%, id=2%, wa=0%
# → Application consuming all CPU. Normal if expected. Rightsize if not.

# Pattern 2: I/O-bound
# r=1, b=6 (blocked!), wa=75%, bi/bo=high
# → Processes waiting for disk. Upgrade storage or fix I/O pattern.

# Pattern 3: Memory-starved (swapping)
# si=5000, so=8000 (actively swapping!), wa=high
# → System thrashing. Add RAM or kill memory hog.

# Pattern 4: Noisy neighbor (cloud)
# st=15% (CPU stolen!), us/sy/wa all low
# → Hypervisor giving your CPU to others. Resize or dedicated instance.

# Pattern 5: Context switch storm
# cs=80000, sy=25%
# → Too many threads fighting for CPU. Application design issue or wrong scheduler.
```

**In Project 9 (post-patch validation):**
```bash
# Our script captures vmstat snapshot after reboot
vmstat 1 10 > /tmp/vmstat_postpatch.txt

# Automated checks:
# - si/so must be 0 (no swap activity)
# - wa must be < 10% (no I/O issues introduced by patch)
# - b must be < 3 (no processes stuck)
# - Any anomaly = flag for review (doesn't auto-fail, but reported)
```

---

### Q21: How do you detect a memory leak in a running production process?
**Project Reference:** Project 7 (Cost Optimization — identifying inefficient processes) / Project 3 (K8s pod OOMKilled)
**Expected Depth:** Observation methods, tools, /proc/PID/smaps, trend analysis

**Answer:**

**What a memory leak looks like:**
- Process RSS (Resident Set Size) grows continuously over time
- Never plateaus even when workload is constant
- Eventually triggers OOM Killer or swap thrashing

**Step 1: Confirm the leak — trending RSS:**
```bash
# Watch a specific process over time
while true; do
    ps -p <PID> -o pid,rss,vsz,comm --no-headers
    sleep 60
done >> /tmp/mem_trend.log

# Or use pidstat for periodic sampling
pidstat -r -p <PID> 60 1440    # Every 60s for 24 hours
# Columns: minflt/s  majflt/s  VSZ      RSS     %MEM  Command
# If RSS grows consistently → leak confirmed

# Quick check: compare over hours
ps -p <PID> -o rss=    # Now: 500MB
# ... 4 hours later ...
ps -p <PID> -o rss=    # Now: 1.2GB  ← Leak!
```

**Step 2: Understand memory composition:**
```bash
# /proc/PID/status — high level
cat /proc/<PID>/status | grep -E 'VmRSS|VmSize|VmSwap|VmData|VmStk'
# VmSize: 2048000 kB    ← Virtual memory (what it requested)
# VmRSS:   800000 kB    ← Physical memory actually used
# VmData:  750000 kB    ← Heap size (malloc'd memory)
# VmStk:     8192 kB    ← Stack size
# VmSwap:       0 kB    ← Swapped out

# If VmData keeps growing → heap leak (most common)

# /proc/PID/smaps — detailed per-mapping breakdown
cat /proc/<PID>/smaps | head -50
# Shows each memory region: heap, stack, shared libraries, mmap'd files
# Look for [heap] size growing

# Summarized view
cat /proc/<PID>/smaps_rollup
# Or: pmap -x <PID>
```

**Step 3: Tools for deeper analysis:**
```bash
# valgrind (if you can restart the process — dev/staging)
valgrind --leak-check=full --track-origins=yes ./application
# NOT for production (10-50x slowdown)

# For Java:
jmap -heap <PID>                    # Heap summary
jmap -histo <PID> | head -20       # Object count by class
jcmd <PID> GC.heap_info            # Heap regions

# For Python:
# tracemalloc module (must be enabled in code)
# objgraph library for reference tracking

# For any process — gdb (if symbols available)
gdb -p <PID>
(gdb) info proc mappings            # Memory map

# strace — watch allocation patterns
strace -e trace=mmap,brk,munmap -p <PID> 2>&1 | head -100
# If lots of mmap() with no corresponding munmap() → leak
```

**Step 4: System-level detection (without knowing the leaker):**
```bash
# Find top memory consumers
ps aux --sort=-%mem | head -10

# Historical trend (if you have node_exporter/Prometheus)
# Query: process_resident_memory_bytes{job="myapp"} over 7 days
# Increasing linear trend = leak

# In Kubernetes (Project 3):
kubectl top pods --sort-by=memory
# Pod approaching its memory limit + OOMKilled history = leak
kubectl get events | grep OOMKilled
```

**In Project 3 (K8s):**
- Pod keeps getting OOMKilled → container reaches memory limit → kernel kills it
- Check: `kubectl describe pod <pod>` → Last State: OOMKilled
- Fix: Either fix the leak OR increase limits (band-aid) + file a bug
- Prevention: Set memory requests = limits (guaranteed QoS), monitor with Prometheus

**In Project 7 (rightsizing):**
- Before recommending a smaller instance, check if high memory usage is a leak
- If RSS = 14GB on a 16GB instance but usage pattern is "grows over days" → it's a leak, not genuine need
- Fix the leak first, THEN rightsize

---

### Q22: What is CPU steal time? When does it happen and how do you diagnose it?
**Project Reference:** Project 7 (Cost Optimization) / Project 3 (K8s node performance)
**Expected Depth:** Hypervisor explanation, when it's expected, when it's a problem, cloud-specific remediation

**Answer:**

**What CPU steal time is:**
Steal time (`st%` in top/vmstat) represents time the virtual CPU was **ready to run** but the hypervisor didn't give it physical CPU time — because the physical core was busy serving another VM on the same host.

```bash
# Visible in:
top       # st% in CPU line
vmstat    # st column
mpstat 1  # %steal per CPU

# Example:
# %Cpu(s): 30.0 us, 5.0 sy, 0.0 ni, 45.0 id, 0.0 wa, 0.0 hi, 0.0 si, 20.0 st
#                                                                             ↑
#                                                                 20% of CPU time STOLEN
```

**When steal happens:**
1. **Burstable instances (T-family):** After exhausting CPU credits, the instance is throttled. What looks like steal is actually credit-based throttling.
2. **Overcommitted hosts:** The physical host has more vCPUs allocated across VMs than physical cores. Under load, VMs compete.
3. **Noisy neighbor:** Another VM on the same physical host is consuming excessive CPU, leaving less for your VM.
4. **Spot instances:** During reclamation preparation, you may see brief steal spikes.

**How to diagnose:**

```bash
# Step 1: Confirm steal is happening
mpstat -P ALL 1 5    # Per-CPU steal breakdown
# If ALL CPUs show similar steal → host-level issue
# If one CPU shows steal → possible CPU pinning issue

# Step 2: For T-family (burstable) — check CPU credits
aws cloudwatch get-metric-statistics \
    --namespace AWS/EC2 \
    --metric-name CPUCreditBalance \
    --dimensions Name=InstanceId,Value=i-xxx \
    --start-time $(date -u -d '-2 hours' +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300 --statistics Average

# CPUCreditBalance = 0 → throttled! That's your "steal"
# CPUSurplusCreditBalance → using surplus credits (will be charged)

# Step 3: Correlate with CloudWatch
# CPUUtilization shows capped value when credits depleted
# If CPUUtilization = 20% flat for hours → T instance throttled to baseline
```

**When steal is expected (not a problem):**
- T3/T3a instances at baseline (e.g., t3.medium baseline = 20% of 2 vCPUs)
- Brief spikes during host maintenance
- <2% on non-burstable instances (normal hypervisor overhead)

**When steal IS a problem:**
- >5% sustained on m5/c5/r5 (fixed-performance) instances
- >10% on any instance = significant performance impact
- Correlates with latency spikes in your application

**Remediation:**

| Situation | Fix |
|-----------|-----|
| T-family credits depleted | Switch to T3 unlimited (pay for burst) or upgrade to m5 (fixed performance) |
| Sustained steal on fixed instance | Stop/start instance (migrates to different host). Or use dedicated instances/hosts |
| Persistent across stop/start | File AWS support case. Or use placement groups to spread |
| On-prem KVM/VMware | Talk to virtualization team — host is overcommitted |

**In Project 7 (cost optimization context):**
- T3 instances are CHEAP but steal when bursting beyond credits
- For steady 60% CPU workload → m-family is better (no throttling, predictable performance)
- For <20% average with occasional spikes → T3 unlimited is most cost-effective
- Decision: `If avg_cpu > baseline_percent → recommend fixed-performance family`

**In Project 3 (K8s context):**
- K8s worker nodes should NEVER be T-family in production (unpredictable steal = unpredictable pod latency)
- We use m5/m6i for worker nodes (Project 3) — consistent performance
- Monitoring: node_exporter exposes `node_cpu_seconds_total{mode="steal"}` → Prometheus alert if >5% for 10min

---
