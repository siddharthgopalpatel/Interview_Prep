# Linux Interview Q&A — Category 5: Process Management & Signals

---

### Q1: Explain the Linux process lifecycle — fork, exec, wait, exit. How does a new process actually start?

**Project Reference:** Project 1 (DevSecOps CI/CD Pipeline — Gunicorn workers)
**Expected Depth:** Kernel-level understanding of process creation, not just "a process runs"

**Answer:**

Every process in Linux starts via the fork-exec model:

**1. fork() — Clone the parent**
- Parent process calls `fork()` → kernel creates an exact copy (child)
- Child gets a new PID but inherits: open file descriptors, memory mappings (copy-on-write), environment variables, signal handlers
- Copy-on-Write (COW): memory pages are shared until one process writes — then the page is duplicated. This makes fork() cheap (no immediate memory copy)
- Return values: fork() returns 0 to child, child's PID to parent

**2. exec() — Replace the process image**
- Child calls `execve("/usr/bin/python", args, env)` → replaces its entire memory with the new program
- PID stays the same, file descriptors stay open (unless O_CLOEXEC)
- The new program starts from its `main()` / entry point

**3. wait() — Parent collects exit status**
- Parent calls `wait()` or `waitpid()` → blocks until child exits
- Retrieves child's exit code (0 = success, non-zero = failure)
- This is CRITICAL — without wait(), child becomes a zombie

**4. exit() — Process terminates**
- Process calls `exit(status)` → kernel frees resources (memory, file descriptors)
- Process entry remains in process table until parent calls wait() (zombie state)
- Kernel sends SIGCHLD to parent to notify

**Real-world example — Gunicorn in Project 1:**
```
gunicorn master (PID 1 in container)
  ├── fork() → worker 1 (PID 10) → exec() not needed (same Python)
  ├── fork() → worker 2 (PID 11)
  └── fork() → worker 3 (PID 12)
```
Gunicorn master forks 3 workers. Each worker inherits the loaded application code (COW means shared memory until request handling writes to it). Master calls `waitpid()` with WNOHANG to detect dead workers and respawn them.

**Key interview points:**
- `fork()` without `exec()` = same program, different process (Gunicorn model)
- `fork()` + `exec()` = running a different program (shell launching commands)
- `vfork()` = optimization where child guarantees exec() immediately (avoids COW page table copy)
- `clone()` = Linux-specific, creates threads (shared memory) or processes with fine-grained sharing control — used by containers (namespaces)

---

### Q2: What is PID 1 and why does it matter in containers? What goes wrong if your application runs as PID 1?

**Project Reference:** Project 3 (EKS Kubernetes Platform — containerized workloads)
**Expected Depth:** Init system responsibilities, signal handling, zombie reaping in container context

**Answer:**

**PID 1 in traditional Linux:**
- First process started by kernel after boot (`/sbin/init` or `systemd`)
- Special responsibilities:
  1. **Adopts orphaned processes** — if a parent dies, its children are re-parented to PID 1
  2. **Reaps zombies** — calls `wait()` on adopted children to clean up process table
  3. **Signal handling** — PID 1 has UNIQUE signal behavior: signals without registered handlers are IGNORED (including SIGTERM!)

**Why this matters in containers:**

In Docker/Kubernetes, your application becomes PID 1:
```dockerfile
# This makes gunicorn PID 1 in the container
ENTRYPOINT ["gunicorn", "app:application"]
```

**Problem 1: Signals are ignored**
- Kubernetes sends SIGTERM for graceful shutdown
- If gunicorn is PID 1 and doesn't explicitly register a SIGTERM handler, the signal is IGNORED
- After `terminationGracePeriodSeconds` (default 30s), Kubernetes sends SIGKILL (ungraceful)
- Result: connections dropped, in-flight requests lost

**Problem 2: Zombie accumulation**
- If gunicorn forks workers and a worker crashes, the zombie is reaped by gunicorn (it's the parent)
- But if a subprocess spawns grandchildren that outlive their parent → they're re-parented to PID 1 (gunicorn)
- Gunicorn doesn't call `wait()` on unknown children → zombies accumulate
- In long-running containers, zombie processes consume PID space

**Solutions:**

1. **Use tini (or dumb-init) as PID 1:**
```dockerfile
ENTRYPOINT ["tini", "--", "gunicorn", "app:application"]
# tini is PID 1, gunicorn is PID 2
# tini forwards signals and reaps zombies
```

2. **Docker --init flag:**
```bash
docker run --init myimage  # Injects tini automatically
```

3. **Kubernetes shareProcessNamespace** (for sidecar scenarios):
```yaml
spec:
  shareProcessNamespace: true  # All containers share PID namespace
```

4. **Application handles it properly:**
- Register SIGTERM handler explicitly
- Call `waitpid(-1, WNOHANG)` periodically to reap orphans
- Gunicorn's master process DOES handle this correctly — but only for its own workers

**In our Project 3 (EKS):**
We use `tini` in our Dockerfile as the entrypoint wrapper. This ensures:
- SIGTERM from Kubernetes pod termination is forwarded to gunicorn
- Gunicorn initiates graceful shutdown (finishes in-flight requests)
- Any zombie processes are automatically reaped

---

### Q3: What's the difference between SIGTERM and SIGKILL? Why does Kubernetes send SIGTERM first before SIGKILL?

**Project Reference:** Project 9 (OS Patching Automation — service stop/start), Project 3 (EKS — pod termination)
**Expected Depth:** Signal semantics, graceful shutdown design, Kubernetes termination lifecycle

**Answer:**

**SIGTERM (Signal 15):**
- "Please terminate gracefully"
- CAN be caught, handled, or ignored by the process
- Process can: finish in-flight requests, close DB connections, flush buffers, write state to disk, deregister from service discovery
- Default action if not handled: terminate (but PID 1 is special — ignores unhandled signals)

**SIGKILL (Signal 9):**
- "Terminate immediately — non-negotiable"
- CANNOT be caught, handled, or ignored — kernel enforces it
- Process has zero opportunity to clean up
- Resources freed by kernel (open files, memory) but application-level cleanup doesn't happen
- Data corruption risk: partial writes, unflushed buffers, broken locks

**Why Kubernetes sends SIGTERM first — the Pod Termination Lifecycle:**

```
1. kubectl delete pod / rolling update triggered
2. Pod marked as "Terminating" — removed from Service endpoints
3. preStop hook executes (if defined) — e.g., sleep 5 for connection draining
4. SIGTERM sent to PID 1 in each container
5. terminationGracePeriodSeconds countdown starts (default: 30s)
6. Application handles SIGTERM — graceful shutdown
7. If still alive after grace period → SIGKILL
```

**Why this design:**
- Gives applications time to drain connections (ALB/Ingress stops sending new requests)
- Allows writing final state (checkpointing, releasing distributed locks)
- Prevents data loss in databases/message queues
- SIGKILL is the safety net — guarantees eventual termination even if app is hung

**In Project 9 (OS Patching):**
When we stop services during patching:
```yaml
- name: Stop application service gracefully
  ansible.builtin.systemd:
    name: "{{ app_service }}"
    state: stopped
  # systemd sends SIGTERM first, waits TimeoutStopSec, then SIGKILL
```
We validate the service stopped cleanly before proceeding with patching.

**In Project 3 (EKS) — our Gunicorn setup:**
```python
# Gunicorn handles SIGTERM:
# 1. Stop accepting new connections
# 2. Send SIGTERM to all workers
# 3. Workers finish current request (within graceful_timeout)
# 4. Workers exit
# 5. Master exits
```

**Best practice for Kubernetes:**
```yaml
spec:
  terminationGracePeriodSeconds: 60  # Match your app's drain time
  containers:
  - lifecycle:
      preStop:
        exec:
          command: ["sh", "-c", "sleep 5"]  # Allow endpoints to update
```

---

### Q4: What are zombie processes? How do you find them and how do you fix them?

**Project Reference:** Project 3 (EKS Kubernetes Platform — long-running containers)
**Expected Depth:** Process states, practical detection, root cause analysis

**Answer:**

**What is a zombie process:**
- A process that has exited but its entry still exists in the process table
- The process has released all resources (memory, file descriptors, CPU)
- Only the PID and exit status remain — waiting for the parent to call `wait()`
- Shown as state `Z` (zombie) or `Z+` in `ps` output
- Also called "defunct" processes

**Why they exist:**
- UNIX design: parent must acknowledge child's death to retrieve exit status
- Between child calling `exit()` and parent calling `wait()` → zombie state
- If parent never calls `wait()` → zombie persists until parent itself dies

**How to find zombies:**
```bash
# Count zombies
ps aux | awk '{if ($8 == "Z") print}' | wc -l

# Find zombie processes
ps -eo pid,ppid,stat,comm | grep -w Z
#  PID  PPID STAT COMMAND
# 1234  1200 Z+   [worker] <defunct>

# Top shows zombie count
top  # Look for "zombie" in header line

# /proc shows status
cat /proc/1234/status | grep State
# State: Z (zombie)

# Find the parent responsible
ps -o pid,ppid,stat,comm -p 1234
# Then check what PPID 1200 is doing
```

**How to fix zombies:**

1. **Kill the parent process** — when parent dies, zombies are re-parented to PID 1 (init/systemd), which reaps them immediately
```bash
kill -SIGCHLD 1200  # Remind parent to reap (if it has a handler)
kill 1200           # Kill parent → init adopts and reaps zombies
```

2. **Fix the application** — the parent should:
```c
// Option A: Handle SIGCHLD
signal(SIGCHLD, SIG_IGN);  // Auto-reap children (Linux-specific)

// Option B: Explicit wait
while (waitpid(-1, NULL, WNOHANG) > 0);  // Non-blocking reap loop
```

3. **In containers — use tini/dumb-init** (as discussed in Q2)

**Why zombies are dangerous:**
- Each zombie consumes a PID (limited resource — default 32768, max 4194304)
- Large number of zombies → `fork()` fails with "Cannot allocate memory" (actually out of PIDs)
- Symptom: can't SSH into server, can't start new processes
- Check: `cat /proc/sys/kernel/pid_max` and compare with `ps -e | wc -l`

**Real-world scenario in Project 3:**
A container running a Python web app that shells out to `subprocess.Popen()` without calling `.wait()` — over days, zombies accumulate. Fix: ensure all `Popen` calls use `communicate()` or the `with` context manager, and add `tini` as container entrypoint.

---

### Q5: How does Gunicorn's pre-fork model work on Linux? Why is it designed this way?

**Project Reference:** Project 1 (DevSecOps Pipeline — Django app served by Gunicorn with 3 workers)
**Expected Depth:** Process model, memory sharing, worker management, signal handling

**Answer:**

**Pre-fork model explained:**

"Pre-fork" means the master process forks worker processes BEFORE any requests arrive:

```
Startup sequence:
1. Master starts → loads Python app into memory (imports Django, loads models)
2. Master calls fork() × N (where N = --workers 3)
3. Each worker inherits the loaded app via Copy-on-Write
4. Workers enter request loop — each handles one request at a time (sync) or many (async)
5. Master enters management loop — monitors workers, handles signals
```

**Architecture in our Project 1:**
```
Container PID namespace:
  tini (PID 1) — signal forwarding + zombie reaping
    └── gunicorn master (PID 7) — no request handling, only management
          ├── worker 1 (PID 10) — handles HTTP requests
          ├── worker 2 (PID 11) — handles HTTP requests
          └── worker 3 (PID 12) — handles HTTP requests
```

**Why pre-fork (not threading or on-demand spawning):**

1. **Memory efficiency via Copy-on-Write:**
   - Django app loads ~100MB of code/libraries
   - With fork, 3 workers share those read-only pages
   - Actual memory: ~100MB + 3×(per-request overhead) ≈ 130MB
   - Without fork (3 separate processes): 300MB

2. **Process isolation:**
   - Python GIL prevents true parallelism in threads
   - Processes bypass GIL — true parallel request handling
   - Worker crash doesn't take down other workers or master

3. **Graceful reload:**
   - `kill -HUP <master_pid>` → master forks new workers with new code → old workers finish current requests → old workers exit
   - Zero-downtime deployments without Kubernetes (useful for traditional VMs)

**Worker lifecycle management by master:**
```
Master loop:
  - SIGCHLD received → worker died
    → Log the failure
    → fork() a new worker (maintain pool size)
  - SIGTTIN → increase workers by 1
  - SIGTTOU → decrease workers by 1
  - SIGTERM → graceful shutdown (SIGTERM to all workers, wait, exit)
  - SIGHUP → graceful reload (new workers with fresh code)
  - Worker heartbeat timeout → kill stuck worker (SIGKILL), fork new one
```

**Worker types and their Linux implications:**
- `sync` (default): One request per process. Simple. Blocked on I/O = wasted process.
- `gevent`/`eventlet`: One process handles thousands of connections via cooperative multitasking (epoll under the hood)
- `uvicorn.workers.UvicornWorker`: Async (ASGI). Uses asyncio event loop within each forked process.

**Tuning formula:**
```
workers = (2 × CPU_cores) + 1  # CPU-bound
workers = (4 × CPU_cores) + 1  # I/O-bound (API calls, DB queries)

# In our K8s pod with 500m CPU request:
# We use 3 workers (good balance for 0.5-1 vCPU)
```

---

### Q6: How do you find what process is using a specific port on Linux?

**Project Reference:** Project 9 (OS Patching — post-patch connectivity validation)
**Expected Depth:** Multiple methods, understanding of socket states, troubleshooting methodology

**Answer:**

**Multiple approaches (from most to least common):**

**1. ss (preferred — modern replacement for netstat):**
```bash
# Find process on port 8080
ss -tlnp | grep :8080
# LISTEN  0  128  0.0.0.0:8080  *:*  users:(("gunicorn",pid=1234,fd=5))

# -t = TCP, -l = listening, -n = numeric, -p = process
# Also shows socket queue sizes (Send-Q = backlog)

# All connections (not just listening):
ss -tnp | grep :8080
```

**2. lsof (list open files — everything is a file):**
```bash
# Find what's using port 8080
lsof -i :8080
# COMMAND  PID  USER  FD  TYPE  DEVICE  SIZE/OFF  NODE  NAME
# gunicorn 1234 app   5u  IPv4  12345   0t0       TCP   *:8080 (LISTEN)

# Find all network connections for a specific PID:
lsof -i -a -p 1234
```

**3. netstat (legacy but still on older systems):**
```bash
netstat -tlnp | grep :8080
# tcp  0  0  0.0.0.0:8080  0.0.0.0:*  LISTEN  1234/gunicorn
```

**4. fuser (direct and scriptable):**
```bash
fuser 8080/tcp
# 8080/tcp: 1234 1235 1236

# Kill whatever is using the port:
fuser -k 8080/tcp
```

**5. /proc filesystem (no tools needed):**
```bash
# Find socket inode for port 8080 (0x1F90 = 8080 in hex)
grep "00000000:1F90" /proc/net/tcp
# Get inode number from output
# Then find which PID owns that inode:
find /proc/*/fd -lname "socket:\[<inode>\]" 2>/dev/null
```

**In Project 9 (post-patch validation):**
```yaml
- name: Verify application port is listening after service restart
  ansible.builtin.shell: |
    ss -tlnp | grep ':{{ app_port }}' | grep -q LISTEN
  register: port_check
  failed_when: port_check.rc != 0
```

**Troubleshooting scenarios:**
- "Address already in use" → find and stop the conflicting process
- Port in TIME_WAIT → previous process closed, waiting for TCP timeout (use `ss -tn state time-wait`)
- Port bound but no connections → check firewall/security group, not a Linux process issue

---

### Q7: What information is available in /proc per process? How do you use it for troubleshooting?

**Project Reference:** Project 9 (OS Patching — service validation), Project 3 (EKS — container debugging)
**Expected Depth:** Practical /proc usage for production debugging, not just "it's a virtual filesystem"

**Answer:**

`/proc` is a pseudo-filesystem — kernel exposes runtime information as files. No disk I/O. Reading `/proc/[PID]/` gives you everything about a running process:

**Critical files per process (`/proc/<PID>/`):**

| File | What it shows | Use case |
|------|--------------|----------|
| `status` | Process state, memory, UIDs, threads | Quick overview, memory usage |
| `cmdline` | Full command with arguments | "What exactly is this process running?" |
| `environ` | Environment variables (null-separated) | Debug missing env vars |
| `fd/` | Directory of open file descriptors (symlinks) | Find open files, sockets, pipes |
| `maps` | Memory mappings (shared libs, heap, stack) | Memory analysis, which .so loaded |
| `limits` | Resource limits (ulimits) per process | "Why can't it open more files?" |
| `io` | Read/write bytes (disk I/O) | Which process is hammering disk |
| `stat` | CPU time, priority, threads, start time | Performance analysis |
| `net/` | Network stats (TCP connections, routing) | Network debugging inside containers |
| `cgroup` | Which cgroups the process belongs to | Container resource limits |
| `ns/` | Namespace references | Container isolation verification |
| `oom_score` | OOM killer priority (0-1000) | Predict which process gets killed |
| `mountinfo` | Mount points visible to process | Debug mount namespace issues |

**Practical troubleshooting examples:**

```bash
# Why is a process using so much memory?
cat /proc/1234/status | grep -i vm
# VmRSS: 524288 kB  (actual physical memory)
# VmSize: 1048576 kB (virtual memory — includes shared)

# What files does a process have open?
ls -la /proc/1234/fd | wc -l  # Count open FDs
ls -la /proc/1234/fd | grep deleted  # Find deleted files still held open

# What are the process's resource limits?
cat /proc/1234/limits
# Max open files  1024  1048576  files  ← soft/hard limit

# What environment was passed to the process?
cat /proc/1234/environ | tr '\0' '\n' | grep DB_HOST

# What's the working directory?
readlink /proc/1234/cwd

# What binary is actually running?
readlink /proc/1234/exe

# I/O stats (which process is doing heavy disk I/O):
cat /proc/1234/io
# read_bytes: 1073741824
# write_bytes: 536870912
```

**In containers (Project 3):**
Inside a container, `/proc` shows only that container's view (PID namespace). But from the host, you can see all container processes with their real PIDs. This is how tools like `kubectl top pods` and cAdvisor work — they read `/proc/<PID>/cgroup` and aggregate.

**In Project 9 (post-patch validation):**
We verify services are running correctly by checking `/proc/<PID>/status` for state and reading `/proc/net/tcp` to confirm ports are bound.

---

### Q8: How does Linux handle graceful shutdown of a service? Explain systemd's KillMode and TimeoutStopSec.

**Project Reference:** Project 9 (OS Patching Automation — stop/start services during patching)
**Expected Depth:** systemd service lifecycle, signal propagation, practical configuration

**Answer:**

**systemd service stop sequence:**

```
systemctl stop myapp.service
  │
  ├─ 1. Execute ExecStop= command (if defined)
  │     e.g., ExecStop=/usr/bin/myapp --shutdown
  │
  ├─ 2. Send signal based on KillMode to remaining processes
  │
  ├─ 3. Wait TimeoutStopSec (default: 90s on RHEL 8/9)
  │
  └─ 4. If still alive → send SIGKILL (force kill)
```

**KillMode options (controls WHO gets the signal):**

| KillMode | Behavior | Use case |
|----------|----------|----------|
| `control-group` (default) | Signal ALL processes in the service's cgroup | Services that spawn children (Gunicorn, Apache) |
| `process` | Signal ONLY the main process (ExecStart PID) | When main process manages its own children gracefully |
| `mixed` | SIGTERM to main process, SIGKILL to remaining after timeout | Main process handles shutdown, kill stragglers |
| `none` | Don't send any signal (only run ExecStop) | Application handles its own shutdown via ExecStop command |

**TimeoutStopSec — how long to wait before SIGKILL:**
```ini
[Service]
TimeoutStopSec=30    # Wait 30s for graceful shutdown
# After 30s, systemd sends SIGKILL to everything remaining
```

**Real-world service configuration (Project 9 context):**
```ini
[Unit]
Description=Gunicorn Application Server
After=network.target

[Service]
Type=notify
ExecStart=/usr/bin/gunicorn --workers 3 --bind 0.0.0.0:8080 app:application
ExecStop=/bin/kill -SIGTERM $MAINPID
ExecReload=/bin/kill -HUP $MAINPID

KillMode=mixed
# SIGTERM to gunicorn master (it signals workers)
# SIGKILL to any remaining workers after timeout

TimeoutStopSec=60
# 60s for workers to finish in-flight requests

KillSignal=SIGTERM
# First signal sent (could be SIGQUIT for Gunicorn graceful)

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**In Project 9 (OS Patching):**
```yaml
- name: Stop application gracefully before patching
  ansible.builtin.systemd:
    name: "{{ item }}"
    state: stopped
  loop: "{{ critical_services }}"
  register: stop_result

- name: Verify service actually stopped (not hung)
  ansible.builtin.shell: |
    systemctl is-active {{ item }} | grep -q "^inactive$"
  loop: "{{ critical_services }}"
  retries: 3
  delay: 10

- name: Check for any leftover processes
  ansible.builtin.shell: |
    pgrep -f "{{ item }}" | wc -l
  register: leftover
  failed_when: leftover.stdout | int > 0
```

**Debugging shutdown issues:**
```bash
# Why won't my service stop?
systemctl status myapp.service  # Shows main PID and cgroup
systemd-cgls -u myapp.service  # Shows ALL processes in service cgroup

# Service stuck in "deactivating"
journalctl -u myapp.service --since "5 min ago"  # Check for shutdown errors

# Find what signal systemd sent
journalctl -u myapp.service | grep -i "kill\|signal\|timeout"

# Override timeout temporarily
systemctl stop myapp.service --timeout=120
```

**Key insight at 12 YOE level:**
The combination of `KillMode=mixed` + appropriate `TimeoutStopSec` + proper `ExecStop` ensures: the application gets time to drain connections and flush state, but the system is GUARANTEED to reclaim resources within a bounded time. This is the same philosophy as Kubernetes' `terminationGracePeriodSeconds` — graceful with a hard backstop.
