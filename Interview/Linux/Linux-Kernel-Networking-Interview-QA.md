# Interview Q&A Bank: Linux Kernel, Cgroups, Namespaces & Networking

## CATEGORY 3: LINUX KERNEL — CGROUPS, NAMESPACES & CONTAINERS

---

### Q1: What are Linux namespaces? Name all 7 types and their purpose.
**Project Reference:** Project 3 (Kubernetes Platform — kubeadm cluster bootstrap), Project 6 (Istio Service Mesh — sidecar network namespaces)
**Expected Depth:** Name all 7, explain how they combine to create containers, relate to Kubernetes/Istio

**Answer:**

Linux namespaces provide process-level isolation by giving each process (or group of processes) its own view of system resources. They're the fundamental building block of containers.

**The 7 namespace types:**

| # | Namespace | Flag | Isolates | K8s/Container Relevance |
|---|-----------|------|----------|------------------------|
| 1 | **PID** | CLONE_NEWPID | Process IDs | Container sees PID 1 as its init process. Can't see/kill host processes |
| 2 | **NET** | CLONE_NEWNET | Network stack (interfaces, routes, iptables, ports) | Each pod gets its own network namespace. Istio sidecar shares pod's net ns |
| 3 | **MNT** | CLONE_NEWNS | Mount points, filesystem view | Container sees its own rootfs (overlay). Can't see host mounts |
| 4 | **UTS** | CLONE_NEWUTS | Hostname, domain name | Container has its own hostname (pod name in K8s) |
| 5 | **IPC** | CLONE_NEWIPC | Inter-process communication (shared memory, semaphores) | Containers in same pod share IPC ns (can use shared memory) |
| 6 | **USER** | CLONE_NEWUSER | UID/GID mappings | Root inside container maps to unprivileged user on host (rootless containers) |
| 7 | **CGROUP** | CLONE_NEWCGROUP | Cgroup root view | Container sees only its own cgroup hierarchy, can't see host cgroups |

**How Kubernetes uses namespaces:**
- **Pod = shared NET + IPC + UTS namespace** — all containers in a pod share the same IP, can communicate via localhost, share hostname
- **Each container gets its own PID + MNT namespace** — separate filesystem, separate process tree
- **Istio sidecar (Project 6):** Envoy runs in the SAME network namespace as the app container (that's why iptables rules in the pod's net ns redirect all traffic through Envoy)

**Inspecting namespaces:**
```bash
# List namespaces of a process
ls -la /proc/<PID>/ns/

# Enter a container's network namespace
nsenter -t <PID> -n ip addr

# List all network namespaces (used by container runtimes)
ip netns list
```

**Key insight at 12 YOE:** Namespaces provide isolation but NOT security by themselves. A process with CAP_SYS_ADMIN can escape namespaces. True container security requires namespaces + cgroups + seccomp + capabilities drop + user namespace mapping.

---

### Q2: What are cgroups? What's the difference between v1 and v2?
**Project Reference:** Project 3 (Kubernetes — resource requests/limits enforced via cgroups), Project 1 (DevSecOps — container resource management)
**Expected Depth:** Explain hierarchy, controllers, how K8s uses them, v1 vs v2 practical differences

**Answer:**

**Cgroups (Control Groups)** are a Linux kernel feature that limits, accounts for, and isolates resource usage (CPU, memory, I/O, network) of process groups. While namespaces provide visibility isolation, cgroups provide resource isolation.

**How cgroups work:**
```
/sys/fs/cgroup/
├── cpu/
│   ├── kubepods/
│   │   ├── burstable/
│   │   │   └── pod<uid>/
│   │   │       └── <container-id>/
│   │   │           ├── cpu.cfs_quota_us    (CPU limit)
│   │   │           └── cpu.cfs_period_us   (100000 = 100ms)
│   │   └── guaranteed/
│   └── system.slice/
├── memory/
│   └── kubepods/
│       └── burstable/
│           └── pod<uid>/
│               └── <container-id>/
│                   ├── memory.limit_in_bytes  (memory limit)
│                   └── memory.usage_in_bytes  (current usage)
└── pids/
```

**Cgroups v1 vs v2:**

| Aspect | Cgroups v1 | Cgroups v2 |
|--------|-----------|-----------|
| **Hierarchy** | Multiple hierarchies (one per controller) | Single unified hierarchy |
| **Mount point** | `/sys/fs/cgroup/<controller>/` | `/sys/fs/cgroup/` (unified) |
| **Controller attachment** | Each controller independent tree | All controllers in one tree |
| **Memory accounting** | Inaccurate (doesn't track kernel memory well) | Accurate (tracks PSI — Pressure Stall Information) |
| **OOM handling** | Kills single process (unpredictable) | Kills entire cgroup (predictable) |
| **CPU distribution** | `cpu.shares` (relative weight) | `cpu.weight` (1-10000 scale) |
| **Thread-level control** | Limited | Full thread-granularity (threaded cgroups) |
| **K8s support** | Default until K8s 1.25 | Default from K8s 1.25+ (with systemd cgroup driver) |

**How Kubernetes uses cgroups (directly from Project 3):**
```yaml
resources:
  requests:
    cpu: "200m"      # → cpu.shares = 204 (v1) or cpu.weight = 8 (v2)
    memory: "512Mi"  # → used for scheduling only (not enforced)
  limits:
    cpu: "1"         # → cpu.cfs_quota_us = 100000 (v1) or cpu.max = "100000 100000" (v2)
    memory: "1Gi"    # → memory.limit_in_bytes = 1073741824 (OOMKill if exceeded)
```

**K8s QoS classes map to cgroup hierarchy:**
- **Guaranteed** (requests == limits): `/kubepods/guaranteed/pod<uid>/`
- **Burstable** (requests < limits): `/kubepods/burstable/pod<uid>/`
- **BestEffort** (no requests/limits): `/kubepods/besteffort/pod<uid>/`

**Why v2 matters in production:**
- PSI (Pressure Stall Information) gives accurate "how starved is this cgroup" metrics
- Better memory accounting prevents surprise OOM kills
- eBPF programs can attach to cgroup v2 for fine-grained observability
- Kubernetes MemoryQoS feature (alpha) uses cgroup v2's `memory.min` for guaranteed memory

**Checking which version is active:**
```bash
# Check cgroup version
stat -fc %T /sys/fs/cgroup/
# "cgroup2fs" = v2, "tmpfs" = v1

# For containerd, check config
grep SystemdCgroup /etc/containerd/config.toml
# SystemdCgroup = true → uses systemd cgroup driver (required for v2)
```

---

### Q3: How does a container actually work at the kernel level?
**Project Reference:** Project 3 (Kubernetes — containerd runtime), Project 1 (DevSecOps — Docker containers)
**Expected Depth:** Combine namespaces + cgroups + rootfs into coherent explanation, mention OCI spec

**Answer:**

A container is NOT a VM. It's a regular Linux process with three kernel features applied:

**1. Namespaces (Isolation)** — what the process can SEE
```
Process gets its own:
- PID namespace: sees itself as PID 1
- NET namespace: own IP, own iptables, own ports
- MNT namespace: own filesystem tree
- UTS namespace: own hostname
- IPC namespace: own shared memory
- USER namespace: own UID mapping
```

**2. Cgroups (Resource Control)** — what the process can USE
```
Process is limited to:
- X CPU cores (cfs_quota)
- Y MB memory (memory.limit)
- Z IO bandwidth (blkio)
- N max PIDs (pids.max)
```

**3. Root Filesystem (Union/Overlay FS)** — what the process can ACCESS
```
OverlayFS layers:
┌─────────────────────────────┐
│ Container Layer (writable)  │  ← Container's writes go here
├─────────────────────────────┤
│ Image Layer 3 (read-only)   │  ← App code
├─────────────────────────────┤
│ Image Layer 2 (read-only)   │  ← Dependencies
├─────────────────────────────┤
│ Image Layer 1 (read-only)   │  ← Base OS (ubuntu:22.04)
└─────────────────────────────┘
```

**What happens when you `docker run nginx`:**

1. **Pull image** → download layers, assemble overlay filesystem
2. **Create namespaces** → `unshare()` or `clone()` with namespace flags
3. **Set up cgroup** → create `/sys/fs/cgroup/.../container-id/`, write limits
4. **Pivot root** → `pivot_root()` to overlay filesystem (container can't see host fs)
5. **Set up networking** → create veth pair, move one end into container's net ns
6. **Apply seccomp profile** → restrict syscalls (no `reboot`, no `mount`, etc.)
7. **Drop capabilities** → remove dangerous Linux capabilities
8. **exec** → replace init process with container entrypoint (nginx)

**The actual syscall sequence (simplified):**
```c
clone(CLONE_NEWPID | CLONE_NEWNET | CLONE_NEWNS | CLONE_NEWUTS | CLONE_NEWIPC)
// Child process:
mount("overlay", "/merged", "overlay", ...)  // Set up overlay fs
pivot_root("/merged", "/merged/.pivot")       // Change root
umount2("/.pivot", MNT_DETACH)               // Hide host fs
prctl(PR_SET_SECCOMP, SECCOMP_MODE_FILTER)   // Apply seccomp
capset(...)                                   // Drop capabilities
execve("/usr/sbin/nginx", ...)               // Run the actual app
```

**OCI Runtime Specification:**
This entire process is standardized by the OCI (Open Container Initiative). The `config.json` in an OCI bundle specifies namespaces, cgroups, mounts, seccomp, and capabilities. `runc` (the default OCI runtime) reads this spec and creates the container.

**Key interview insight:** There is no "container" kernel object. The kernel only knows about processes, namespaces, cgroups, and mount points. "Container" is a userspace concept that combines these primitives.

---

### Q4: Why does Kubernetes require swap disabled? What happens if swap is on?
**Project Reference:** Project 3 (Kubernetes Platform — kubeadm prerequisites: `swapoff -a`)
**Expected Depth:** Explain scheduler implications, QoS guarantees, memory accounting, K8s 1.28+ changes

**Answer:**

**Why Kubernetes historically requires swap disabled (`swapoff -a`):**

In our Project 3 kubeadm bootstrap (Ansible playbook `k8s/first.yml`), we run `swapoff -a` and remove swap entries from `/etc/fstab` as a prerequisite. Here's why:

**1. Scheduler Accuracy:**
The kube-scheduler makes placement decisions based on memory requests. If a node reports 8GB available and schedules pods requesting 8GB total, it expects all that memory is RAM. With swap, the node might have 4GB RAM + 4GB swap — pods "fit" mathematically but performance is 100x worse for swapped processes.

**2. QoS Guarantees Break:**
```
Guaranteed pod (requests == limits == 1Gi):
- Without swap: Gets 1Gi RAM. If it exceeds → OOMKill (predictable, fast failure)
- With swap: Exceeds 1Gi → swaps to disk → process runs at disk speed → 
  appears "healthy" to K8s but serving requests in 10+ seconds → SLA violated
  but no restart because it's not OOMKilled
```

**3. Memory Accounting:**
Cgroups `memory.limit_in_bytes` controls RSS (physical RAM). With swap, a process can use limit + swap — breaking the isolation contract. The kubelet can't accurately report memory usage.

**4. Latency Unpredictability:**
Swap introduces non-deterministic latency. A pod might respond in 5ms when in RAM, 500ms when partially swapped. This makes SLO compliance impossible and autoscaling useless (HPA sees "pod is fine" but it's actually degraded).

**What happens if swap is on (pre-1.22):**
```bash
# kubeadm preflight check
[preflight] Running pre-flight checks
[ERROR Swap]: running with swap on is not supported. Please disable swap
# kubeadm refuses to initialize
```

**Kubernetes 1.28+ (swap support — alpha/beta):**
Kubernetes now supports swap with `NodeSwap` feature gate:
```yaml
# kubelet config
featureGates:
  NodeSwap: true
memorySwap:
  swapBehavior: LimitedSwap  # Only BestEffort/Burstable pods can use swap
```

- **NoSwap** (default): Same as before — no swap usage
- **LimitedSwap**: Only Burstable pods can use swap (up to their limit - request)
- Guaranteed pods NEVER swap (preserves QoS contract)

**Practical stance at 12 YOE:**
In production, I still disable swap. The LimitedSwap feature is useful for batch workloads (AI/ML training that can tolerate latency) but not for latency-sensitive web services. Our Project 3 kubeadm playbook continues to enforce `swapoff -a`.

---

### Q5: What kernel modules does Kubernetes need (overlay, br_netfilter) and why?
**Project Reference:** Project 3 (Kubernetes Platform — kubeadm node preparation via Ansible)
**Expected Depth:** Explain each module's purpose, what breaks without them, how to verify

**Answer:**

In Project 3, our Ansible playbook loads these kernel modules on every K8s node before `kubeadm init`:

```bash
# /etc/modules-load.d/k8s.conf
overlay
br_netfilter
```

**Module 1: `overlay`**
```
Purpose: Enables OverlayFS — the filesystem driver used by containerd/Docker 
         to layer container images efficiently.

How it works:
┌─UpperDir (writable layer — container changes) ─┐
│ /var/lib/containerd/.../diff/                    │
├─ LowerDir (read-only layers — image layers) ────┤
│ /var/lib/containerd/.../committed/               │
├─ MergedDir (unified view — what container sees) ─┤
│ /var/lib/containerd/.../merged/                  │
└─ WorkDir (internal bookkeeping) ────────────────┘

Without overlay module:
- containerd can't create container filesystems
- Pods fail to start: "overlay mount failed: no such device"
- Fall back to vfs driver (copies entire image per container — wastes disk, slow)
```

**Module 2: `br_netfilter`**
```
Purpose: Enables iptables/netfilter to see BRIDGED traffic (Layer 2).

Problem without it:
- Kubernetes networking uses Linux bridges (cbr0, docker0, cni0)
- Pods on the SAME node communicate via the bridge (Layer 2, no routing)
- By default, iptables only processes ROUTED traffic (Layer 3)
- Without br_netfilter: kube-proxy iptables rules DON'T apply to 
  pod-to-pod traffic on same node
- Result: NetworkPolicies don't work for same-node pods, 
  Service routing breaks for same-node traffic

With br_netfilter:
- Bridge traffic is passed through iptables chains
- kube-proxy rules apply uniformly regardless of whether pods 
  are on same or different nodes
- NetworkPolicies (Calico) work correctly for all traffic paths
```

**Additional modules sometimes needed:**
```bash
# For IPVS mode kube-proxy (better performance than iptables at scale)
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack

# For certain CNIs
vxlan          # For Flannel/Calico VXLAN mode
ip_tables      # Base iptables support
```

**Verification commands:**
```bash
# Check if modules are loaded
lsmod | grep overlay
lsmod | grep br_netfilter

# Load immediately (doesn't persist)
modprobe overlay
modprobe br_netfilter

# Persist across reboots
echo "overlay" >> /etc/modules-load.d/k8s.conf
echo "br_netfilter" >> /etc/modules-load.d/k8s.conf

# Verify bridge traffic goes through iptables
cat /proc/sys/net/bridge/bridge-nf-call-iptables   # Must be 1
cat /proc/sys/net/bridge/bridge-nf-call-ip6tables  # Must be 1
```

**What happens if you forget (real incident pattern):**
1. Cluster bootstraps fine (kubeadm doesn't always check modules)
2. Pods start and run
3. NetworkPolicy is applied — "deny all except from frontend namespace"
4. Same-node pod-to-pod traffic BYPASSES the policy (goes through bridge, not iptables)
5. Security audit finds pods can communicate despite deny-all policy
6. Root cause: `br_netfilter` not loaded → bridge traffic skips netfilter

This is why our Ansible playbook in Project 3 loads these modules BEFORE kubeadm init — not after.


---

### Q6: What sysctl parameters does Kubernetes need and why?
**Project Reference:** Project 3 (Kubernetes Platform — kubeadm node preparation)
**Expected Depth:** Explain each parameter's purpose, what breaks without it, how it relates to pod networking

**Answer:**

In Project 3's Ansible playbook, we configure these sysctl parameters on every Kubernetes node:

```bash
# /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
```

**Parameter 1: `net.bridge.bridge-nf-call-iptables = 1`**
```
What it does: Forces bridged IPv4 traffic through iptables rules.

Why K8s needs it:
- Pods on the same node communicate via a Linux bridge (cni0/cbr0)
- Bridge traffic is Layer 2 — normally bypasses iptables (Layer 3)
- kube-proxy programs iptables rules for Service → Pod routing
- Without this: Service ClusterIP resolution FAILS for same-node pod-to-pod calls
- Without this: Calico NetworkPolicies DON'T APPLY to same-node traffic

Prerequisite: br_netfilter kernel module must be loaded FIRST
(this sysctl parameter doesn't exist until the module is loaded)
```

**Parameter 2: `net.bridge.bridge-nf-call-ip6tables = 1`**
```
Same as above but for IPv6 traffic.
Even if you're not using IPv6 for pods, some CNI health checks
and dual-stack configurations need this.
```

**Parameter 3: `net.ipv4.ip_forward = 1`**
```
What it does: Enables the Linux kernel to forward packets between interfaces
(act as a router).

Why K8s needs it:
- Each pod has its own network namespace with a veth pair
- Pod traffic: pod-veth → host-veth → bridge → host routing → destination
- If ip_forward=0: kernel DROPS packets not destined for itself
- Every K8s node IS a router — forwarding packets between pod networks

Without this:
- Pods can't communicate across nodes
- Pod → external traffic fails (pod tries to reach internet, 
  node refuses to forward)
- Only same-node, same-bridge traffic works
```

**Additional production sysctl tuning (not K8s-required but best practice):**
```bash
# Increase conntrack table for high-traffic clusters
net.netfilter.nf_conntrack_max = 1000000

# Prevent conntrack table overflow (causes random packet drops)
net.netfilter.nf_conntrack_tcp_timeout_established = 86400

# Increase ARP cache for large clusters (many pod IPs)
net.ipv4.neigh.default.gc_thresh1 = 4096
net.ipv4.neigh.default.gc_thresh2 = 8192
net.ipv4.neigh.default.gc_thresh3 = 16384

# Increase max socket buffers (high throughput pods)
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216

# Increase local port range (many outbound connections)
net.ipv4.ip_local_port_range = 1024 65535

# Reuse TIME_WAIT sockets (high connection rate services)
net.ipv4.tcp_tw_reuse = 1
```

**Applying sysctls:**
```bash
# Apply immediately
sysctl --system

# Verify
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.ipv4.ip_forward

# In Ansible (Project 3 approach):
- name: Set sysctl parameters for K8s
  sysctl:
    name: "{{ item.key }}"
    value: "{{ item.value }}"
    sysctl_file: /etc/sysctl.d/k8s.conf
    reload: yes
  loop:
    - { key: net.bridge.bridge-nf-call-iptables, value: 1 }
    - { key: net.bridge.bridge-nf-call-ip6tables, value: 1 }
    - { key: net.ipv4.ip_forward, value: 1 }
```

**Debugging tip:** If `sysctl: net.bridge.bridge-nf-call-iptables: No such file or directory` → the `br_netfilter` module isn't loaded. Load the module first, THEN set the sysctl.

---

### Q7: How does containerd work? What's its relationship to runc and CRI?
**Project Reference:** Project 3 (Kubernetes Platform — containerd as CRI runtime), Project 1 (DevSecOps — container builds)
**Expected Depth:** Explain the full stack from kubelet to process, CRI interface, OCI spec, when runc runs

**Answer:**

**The container runtime stack (from kubelet to Linux process):**

```
┌─────────────────────────────────────────────────────────────┐
│ kubelet                                                      │
│ (Kubernetes agent — manages pod lifecycle)                   │
└─────────────────────┬───────────────────────────────────────┘
                      │ CRI (Container Runtime Interface) — gRPC
                      │ /run/containerd/containerd.sock
┌─────────────────────▼───────────────────────────────────────┐
│ containerd                                                   │
│ (High-level runtime — manages images, containers, snapshots)│
│ - Pulls images from registry (ECR, DockerHub)               │
│ - Manages image layers (overlay snapshotter)                │
│ - Creates OCI bundle (config.json + rootfs)                 │
│ - Manages container lifecycle (create/start/stop/delete)    │
└─────────────────────┬───────────────────────────────────────┘
                      │ OCI Runtime Spec (exec binary)
                      │ /usr/bin/runc
┌─────────────────────▼───────────────────────────────────────┐
│ runc                                                         │
│ (Low-level runtime — creates the actual container process)  │
│ - Reads config.json (namespaces, cgroups, seccomp, caps)    │
│ - Makes syscalls: clone(), pivot_root(), seccomp(), exec()  │
│ - Sets up namespaces, cgroups, mounts                       │
│ - Execs container entrypoint                                │
│ - Exits after container starts (no daemon)                  │
└─────────────────────────────────────────────────────────────┘
```

**CRI (Container Runtime Interface):**
- gRPC protocol defined by Kubernetes
- Two services: `RuntimeService` (create/start/stop containers) + `ImageService` (pull/list/remove images)
- Allows kubelet to work with ANY CRI-compliant runtime (containerd, CRI-O)
- Replaced the old dockershim (Docker → containerd migration in K8s 1.24)

**containerd responsibilities:**
```
1. Image management:
   - Pull from registry (ECR in our Project 1)
   - Store layers in content store
   - Create overlay snapshots (read-only layers + writable layer)

2. Container lifecycle:
   - Create OCI bundle from image
   - Call runc to create container
   - Manage stdin/stdout/stderr streams
   - Report status back to kubelet via CRI

3. Snapshot management:
   - Overlay filesystem management
   - Garbage collection of unused layers

4. Networking (delegated):
   - containerd calls CNI plugins for network setup
   - Doesn't do networking itself — calls Calico/VPC-CNI
```

**runc responsibilities (short-lived, not a daemon):**
```
1. Read OCI config.json
2. Create Linux namespaces (clone syscalls)
3. Set up cgroup limits
4. Configure seccomp filter
5. Drop capabilities
6. pivot_root to overlay filesystem
7. exec() container entrypoint
8. EXIT (runc process gone — container is just a regular process now)
```

**containerd configuration (Project 3):**
```toml
# /etc/containerd/config.toml
version = 2
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.k8s.io/pause:3.9"
  [plugins."io.containerd.grpc.v1.cri".containerd]
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      runtime_type = "io.containerd.runc.v2"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
        SystemdCgroup = true  # Critical for K8s — use systemd cgroup driver
```

**Why `SystemdCgroup = true` matters:**
- systemd is PID 1 on modern Linux — it manages cgroup hierarchy
- If containerd uses cgroupfs driver while kubelet uses systemd → two cgroup managers fight over hierarchy → resource tracking breaks → pods randomly OOMKilled
- Both MUST agree: kubelet (`--cgroup-driver=systemd`) + containerd (`SystemdCgroup = true`)

**The pause container:**
- Every pod has a hidden "pause" container (sandbox)
- It holds the pod's network namespace open (even if all app containers restart)
- It's the PID 1 of the pod's PID namespace — reaps zombie processes

**Debugging commands:**
```bash
# containerd status
systemctl status containerd
crictl info

# List containers via CRI
crictl ps

# List images
crictl images

# Inspect container
crictl inspect <container-id>

# Check runc is available
runc --version
```

---

### Q8: What is seccomp and how does it restrict containers?
**Project Reference:** Project 3 (Kubernetes — pod security), Project 1 (DevSecOps — container hardening)
**Expected Depth:** Explain mechanism, default profile, how to apply in K8s, practical examples

**Answer:**

**Seccomp (Secure Computing Mode)** is a Linux kernel feature that restricts which syscalls a process can make. It's the last line of defense — even if an attacker escapes namespaces, seccomp blocks dangerous kernel interactions.

**How it works:**
```
Application code
    ↓ (makes syscall, e.g., "mount()")
Kernel seccomp filter (BPF program)
    ↓ (checks syscall against allowed list)
    ├── ALLOW → syscall proceeds normally
    ├── ERRNO → returns error to process (EPERM)
    ├── KILL → kills the process immediately (SIGSYS)
    ├── TRAP → sends SIGSYS signal (can be caught)
    └── LOG → allows but logs the syscall
```

**Docker/containerd default seccomp profile blocks ~44 syscalls including:**
```
❌ reboot         - Can't reboot the host
❌ mount/umount   - Can't mount filesystems (escape overlay)
❌ kexec_load     - Can't load new kernel
❌ init_module    - Can't load kernel modules
❌ delete_module  - Can't remove kernel modules
❌ pivot_root     - Can't change root filesystem
❌ swapon/swapoff - Can't affect host swap
❌ clock_settime  - Can't change system clock
❌ bpf            - Can't load eBPF programs
❌ unshare        - Can't create new namespaces (prevent container-in-container escape)
❌ keyctl         - Can't access kernel keyring
```

**Applying seccomp in Kubernetes:**

```yaml
# Pod-level seccomp (Kubernetes 1.19+)
apiVersion: v1
kind: Pod
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault  # Use containerd's default profile (recommended)
  containers:
  - name: app
    image: myapp:latest
```

**Seccomp profile types:**
| Type | Effect |
|------|--------|
| `Unconfined` | No seccomp filtering (dangerous, only for debugging) |
| `RuntimeDefault` | Container runtime's default profile (~44 syscalls blocked) |
| `Localhost` | Custom profile from node filesystem |

**Custom seccomp profile example (restrict further):**
```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": ["read", "write", "open", "close", "stat", "fstat",
                "mmap", "mprotect", "munmap", "brk", "rt_sigaction",
                "accept", "bind", "listen", "connect", "socket",
                "sendto", "recvfrom", "epoll_wait", "epoll_ctl",
                "clone", "execve", "exit_group", "futex", "nanosleep"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

**Kubernetes Pod Security Standards (PSS) and seccomp:**
- **Privileged:** No seccomp requirement
- **Baseline:** No seccomp requirement (but recommended)
- **Restricted:** REQUIRES `RuntimeDefault` or `Localhost` — `Unconfined` rejected

**Real-world debugging scenario:**
```bash
# App crashes with SIGSYS inside container
# Check audit log for blocked syscalls:
journalctl -k | grep SECCOMP
# audit: type=1326 ... syscall=315 ... (io_uring_enter blocked)

# Solution: Either remove the syscall from app, or add it to allowed list
```

**Production recommendation (from our projects):**
1. Run with `RuntimeDefault` on all workloads (baseline security, zero app changes)
2. For high-security pods: generate custom profile using tools like `seccomp-bpf-tracer` or `oci-seccomp-bpf-hook`
3. Never run `Unconfined` in production (we enforce this via Kyverno in Project 1)

---

### Q9: What are Linux capabilities? Which ones do containers get by default?
**Project Reference:** Project 3 (Kubernetes — security context), Project 1 (DevSecOps — production hardening checklist)
**Expected Depth:** Explain capability model, default set, what to drop, practical K8s usage

**Answer:**

**Linux capabilities** split the all-powerful root (UID 0) into ~40 individual privileges. Instead of binary "root or not," you can grant specific powers.

**Why capabilities exist:**
```
Traditional UNIX:     root (UID 0) = GOD MODE (all privileges)
                      non-root = LIMITED

With capabilities:    Any process can have specific privileges WITHOUT being root
                      root can have capabilities REMOVED
```

**Default capabilities granted to containers (Docker/containerd):**
```
CAP_CHOWN           - Change file ownership
CAP_DAC_OVERRIDE    - Bypass file read/write/execute permission checks
CAP_FSETID          - Don't clear set-user-ID/set-group-ID bits when modifying files
CAP_FOWNER          - Bypass permission checks for operations that require UID match
CAP_MKNOD           - Create special files
CAP_NET_RAW         - Use RAW/PACKET sockets (ping, tcpdump)
CAP_SETGID          - Set GID
CAP_SETUID          - Set UID
CAP_SETFCAP         - Set file capabilities
CAP_SETPCAP         - Modify process capabilities
CAP_NET_BIND_SERVICE - Bind to ports < 1024
CAP_SYS_CHROOT     - Use chroot()
CAP_KILL            - Send signals to any process
CAP_AUDIT_WRITE     - Write to audit log
```

**Dangerous capabilities NOT given by default (but sometimes added):**
```
CAP_SYS_ADMIN    - Mount filesystems, configure namespaces, load BPF (basically root)
CAP_NET_ADMIN    - Configure network interfaces, iptables, routes
CAP_SYS_PTRACE   - Trace/debug any process (container escape vector)
CAP_SYS_RAWIO    - Raw I/O access (can corrupt disk)
CAP_SYS_MODULE   - Load/unload kernel modules
```

**Kubernetes security context (from our production checklist):**
```yaml
# Our standard production security context (Project 1 & 3)
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL        # Drop everything first
    add:
      - NET_BIND_SERVICE  # Only if app needs port < 1024
```

**Why `drop: ALL` is best practice:**
- Most applications need ZERO capabilities
- Default set includes 14 capabilities — most are unnecessary for web apps
- `CAP_NET_RAW` allows packet crafting (ARP spoofing, network sniffing in pod network)
- `CAP_DAC_OVERRIDE` bypasses ALL file permissions
- Each capability is a potential attack surface

**When you NEED to add capabilities back:**
| Capability | When Needed |
|-----------|-------------|
| NET_BIND_SERVICE | Nginx binding to port 80 as non-root |
| NET_ADMIN | Istio init container (sets up iptables rules for sidecar) |
| NET_RAW | Pod needs ping/traceroute (debugging) |
| SYS_PTRACE | Debugging containers, some APM agents |

**Istio init container (Project 6) needs NET_ADMIN + NET_RAW:**
```yaml
# Istio's init container configures iptables in pod's netns
initContainers:
- name: istio-init
  securityContext:
    capabilities:
      add: [NET_ADMIN, NET_RAW]  # Needed to set up iptables redirect
      drop: [ALL]
    runAsNonRoot: false  # Must be root to modify iptables
    runAsUser: 0
```

**Checking capabilities of a running container:**
```bash
# From host, find container PID
PID=$(crictl inspect <container-id> | jq .info.pid)

# Check effective capabilities
cat /proc/$PID/status | grep Cap
# CapEff: 00000000a80425fb (bitmask of active capabilities)

# Decode the bitmask
capsh --decode=00000000a80425fb

# From inside container
cat /proc/1/status | grep Cap
```

**Enforcement via Kyverno (our Project 1 approach):**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: drop-all-capabilities
spec:
  validationFailureAction: Enforce
  rules:
  - name: require-drop-all
    match:
      resources:
        kinds: [Pod]
    validate:
      message: "Containers must drop ALL capabilities"
      pattern:
        spec:
          containers:
          - securityContext:
              capabilities:
                drop: ["ALL"]
```

---

### Q10: How does the OOM killer work? How does Kubernetes interact with it?
**Project Reference:** Project 3 (Kubernetes — resource limits, QoS classes), Project 1 (DevSecOps — container stability)
**Expected Depth:** Explain oom_score_adj, K8s QoS mapping, debugging OOMKills, prevention strategies

**Answer:**

**The Linux OOM (Out of Memory) Killer:**

When the system (or a cgroup) runs out of memory and can't reclaim enough via page cache eviction, the kernel's OOM killer selects and kills a process to free memory.

**Process selection — oom_score (0-1000):**
```
oom_score = (% of memory used by process) + oom_score_adj

Higher score = killed first
- oom_score_adj ranges from -1000 (never kill) to +1000 (kill first)
- Each process has /proc/<PID>/oom_score_adj
```

**How Kubernetes sets oom_score_adj based on QoS:**

| QoS Class | oom_score_adj | Kill Priority | How to Get It |
|-----------|---------------|---------------|---------------|
| **Guaranteed** | -997 | Last killed (protected) | requests == limits for ALL resources |
| **Burstable** | 2 to 999 (scaled by memory request ratio) | Middle | requests < limits (or only one set) |
| **BestEffort** | 1000 | First killed (sacrificed) | No requests or limits set |

**The OOM kill chain in Kubernetes:**

```
1. Pod memory usage approaches cgroup limit
   └── /sys/fs/cgroup/memory/kubepods/.../memory.limit_in_bytes

2. Kernel tries to reclaim (page cache, buffers)
   └── If reclaimable memory frees enough → no kill

3. Kernel OOM killer triggers within the cgroup
   └── Selects process with highest oom_score in that cgroup
   └── Sends SIGKILL (unblockable, no graceful shutdown)

4. Container exits with code 137 (128 + 9 = SIGKILL)
   └── kubectl: "OOMKilled" in container status

5. kubelet reports to API server
   └── Pod restart policy applies:
       - Always → restart (CrashLoopBackOff if repeated)
       - OnFailure → restart
       - Never → stays dead
```

**Two types of OOMKill in Kubernetes context:**

**Type 1: Container exceeds its memory LIMIT (cgroup OOM)**
```bash
# Container limit: 512Mi
# Container uses: 520Mi
# Result: Kernel OOM kills within that container's cgroup
# Only this container is killed — other containers on the node are fine

# Diagnosis:
kubectl describe pod <name>
# Last State: Terminated
#   Reason: OOMKilled
#   Exit Code: 137
```

**Type 2: Node-level OOM (system OOM)**
```bash
# All pods together exhaust node memory (beyond kubelet reserved)
# kubelet's eviction manager kicks in BEFORE kernel OOM:
#   - memory.available < 100Mi (default eviction threshold)
#   - kubelet evicts pods: BestEffort first, then Burstable
#   - If eviction is too slow → kernel OOM killer fires on node level
#   - oom_score_adj determines which pod dies

# Diagnosis:
kubectl describe node <name>
# Conditions:
#   MemoryPressure: True
# Events:
#   "evicting pod due to memory pressure"
```

**kubelet eviction vs kernel OOM:**
```
kubelet eviction (graceful):
- Monitors memory.available every 10s
- Evicts pods respecting QoS priority
- Pod gets SIGTERM → gracefulTermination period → SIGKILL
- Shows as "Evicted" in pod status

Kernel OOM (violent):
- Fires when cgroup/system literally at 0 free pages
- No grace period — immediate SIGKILL
- Shows as "OOMKilled" in container status
- kubelet eviction should prevent this from happening node-wide
```

**Preventing OOMKill (production strategies from our projects):**

1. **Set accurate memory limits (not too tight):**
```yaml
resources:
  requests:
    memory: "512Mi"   # What app normally uses
  limits:
    memory: "1Gi"     # 1.5-2x headroom for spikes
```

2. **Monitor before setting limits:**
```bash
# Use VPA in recommend mode to find actual usage
kubectl top pod <name> --containers
# Check Grafana: container_memory_working_set_bytes
```

3. **Understand what's measured:**
```
container_memory_usage_bytes     = RSS + cache (misleading — includes page cache)
container_memory_working_set_bytes = RSS + active cache (what K8s uses for OOM decision)
container_memory_rss             = actual heap/stack usage
```

4. **Java/JVM-specific (common OOMKill cause):**
```bash
# JVM doesn't see cgroup limits by default (older versions)
# Allocates heap based on HOST memory, not container limit
# Fix: Use -XX:+UseContainerSupport (default since JDK 10)
# Set: -XX:MaxRAMPercentage=75 (use 75% of limit, leave room for native memory)
```

5. **Debug a recurring OOMKill:**
```bash
# Check actual memory usage over time
kubectl top pod --containers | grep <pod>

# Check node-level
dmesg | grep -i "oom\|killed process"
journalctl -k | grep oom

# Check cgroup stats directly on node
cat /sys/fs/cgroup/memory/kubepods/burstable/pod<uid>/<cid>/memory.stat
# Look at: rss, cache, mapped_file, pgfault, pgmajfault
```

**Key interview insight:** CPU limit exceeded → process is THROTTLED (slow but alive). Memory limit exceeded → process is KILLED (dead). This asymmetry is because CPU is compressible (can be taken away temporarily) while memory is incompressible (can't take allocated pages back without killing the process).



---

## CATEGORY 4: NETWORKING — IPTABLES, DNS, TROUBLESHOOTING

---

### Q11: Explain iptables tables and chains. How does a packet traverse them?
**Project Reference:** Project 3 (Kubernetes — kube-proxy iptables rules, Calico), Project 6 (Istio — iptables NAT redirect for sidecar)
**Expected Depth:** Name all tables, chains, packet flow order, relate to K8s service routing

**Answer:**

**iptables** is the Linux kernel's packet filtering and NAT framework. It's the backbone of Kubernetes service routing (kube-proxy) and Istio traffic interception.

**Tables (each contains chains with rules):**

| Table | Purpose | When Used in K8s |
|-------|---------|-----------------|
| **raw** | Bypass connection tracking (conntrack) | Rarely — some CNI performance optimizations |
| **mangle** | Modify packet headers (TTL, TOS, marks) | Calico marks packets for routing decisions |
| **nat** | Network Address Translation (SNAT, DNAT) | kube-proxy: DNAT ClusterIP → PodIP. Istio: redirect to Envoy |
| **filter** | Allow/drop/reject packets | Calico NetworkPolicies (ACCEPT/DROP) |

**Chains (hook points in the kernel network stack):**

| Chain | When | Used For |
|-------|------|----------|
| **PREROUTING** | Packet arrives at interface (before routing decision) | DNAT (change destination — kube-proxy) |
| **INPUT** | Packet destined for this host | Filtering incoming to node |
| **FORWARD** | Packet being routed through this host | Pod-to-pod traffic crossing the node |
| **OUTPUT** | Packet originated from this host | Node-originated traffic, local pod → service |
| **POSTROUTING** | Packet leaving interface (after routing) | SNAT/Masquerade (hide pod IP behind node IP) |

**Complete packet traversal order:**

```
INCOMING PACKET (e.g., external → pod):
┌─────────────────────────────────────────────────────────────┐
│ Network Interface (eth0)                                     │
└───────────┬─────────────────────────────────────────────────┘
            ↓
┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐
│ raw.PREROUTING    │ → │ mangle.PREROUTING │ → │ nat.PREROUTING    │
└───────────────────┘   └───────────────────┘   └───────────────────┘
            ↓                                            ↓
     [Routing Decision: for this host or forwarding?]
            ↓                                    ↓
     (Local delivery)                     (Forwarding)
            ↓                                    ↓
┌────────────────────┐              ┌────────────────────┐
│ mangle.INPUT       │              │ mangle.FORWARD     │
│ filter.INPUT       │              │ filter.FORWARD     │ ← NetworkPolicies here
└────────────────────┘              └────────────────────┘
            ↓                                    ↓
     [Local Process]                 ┌───────────────────────┐
                                     │ mangle.POSTROUTING    │
                                     │ nat.POSTROUTING       │ ← Masquerade
                                     └───────────────────────┘
                                              ↓
                                     [Out via interface]

OUTGOING PACKET (e.g., pod → external):
┌─────────────────────────────────────────────────────────────┐
│ Local Process (pod via veth)                                 │
└───────────┬─────────────────────────────────────────────────┘
            ↓
┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐
│ raw.OUTPUT        │ → │ mangle.OUTPUT     │ → │ nat.OUTPUT        │
└───────────────────┘   └───────────────────┘   └───────────────────┘
            ↓
     [Routing Decision]
            ↓
┌───────────────────┐   ┌───────────────────────┐
│ filter.OUTPUT     │ → │ mangle.POSTROUTING    │
└───────────────────┘   │ nat.POSTROUTING       │ ← SNAT here
                        └───────────────────────┘
```

**Viewing K8s-related iptables rules:**
```bash
# See all nat rules (kube-proxy service routing)
iptables -t nat -L -n -v | grep KUBE

# See all filter rules (NetworkPolicies via Calico)
iptables -t filter -L -n -v | grep cali

# Count rules (large clusters can have 10,000+ rules)
iptables -t nat -L | wc -l
```

---

### Q12: How does kube-proxy use iptables for Kubernetes service routing?
**Project Reference:** Project 3 (Kubernetes — ClusterIP/NodePort services with Calico CNI)
**Expected Depth:** Show actual rule chains, explain DNAT, load balancing, and performance implications

**Answer:**

kube-proxy watches the Kubernetes API for Service and Endpoint changes, then programs iptables rules on every node to implement Service → Pod routing.

**Example: Service `web-svc` (ClusterIP 10.96.0.100:80) → 3 pods:**

```bash
# Step 1: Pod calls web-svc → packet has dst=10.96.0.100:80
# nat.PREROUTING or nat.OUTPUT (depending on source) catches it

# KUBE-SERVICES chain (top-level dispatch)
-A KUBE-SERVICES -d 10.96.0.100/32 -p tcp --dport 80 \
    -j KUBE-SVC-XXXXXX

# KUBE-SVC chain (load balancing — random selection)
-A KUBE-SVC-XXXXXX -m statistic --mode random --probability 0.33333 \
    -j KUBE-SEP-AAAA    # → Pod 1
-A KUBE-SVC-XXXXXX -m statistic --mode random --probability 0.50000 \
    -j KUBE-SEP-BBBB    # → Pod 2
-A KUBE-SVC-XXXXXX \
    -j KUBE-SEP-CCCC    # → Pod 3 (default/fallback)

# KUBE-SEP chain (Service EndPoint — actual DNAT)
-A KUBE-SEP-AAAA -p tcp -j DNAT --to-destination 10.244.1.5:8080
-A KUBE-SEP-BBBB -p tcp -j DNAT --to-destination 10.244.2.8:8080
-A KUBE-SEP-CCCC -p tcp -j DNAT --to-destination 10.244.3.12:8080
```

**What happens step by step:**
```
1. Pod sends packet: src=10.244.1.20, dst=10.96.0.100:80 (ClusterIP)
2. Packet hits nat.OUTPUT (locally originated) or nat.PREROUTING (forwarded)
3. KUBE-SERVICES matches destination ClusterIP → jumps to KUBE-SVC chain
4. KUBE-SVC uses iptables `statistic` module for random load balancing
   (probability math gives equal distribution across endpoints)
5. Selected KUBE-SEP does DNAT: dst changes from 10.96.0.100 → 10.244.2.8:8080
6. Conntrack records the translation (reply packets get reverse-NATed)
7. Packet is routed to destination pod
8. Reply: src=10.244.2.8 → conntrack reverses → src appears as 10.96.0.100 to caller
```

**NodePort adds another layer:**
```bash
# External traffic hits node on port 30080
-A KUBE-NODEPORTS -p tcp --dport 30080 -j KUBE-SVC-XXXXXX
# Same KUBE-SVC chain → same random selection → same DNAT to pod
# Plus SNAT (masquerade) so reply comes back through this node
```

**Performance problem at scale:**
```
100 services × 10 endpoints each = 1,000+ iptables rules in KUBE-SERVICES
Rules are evaluated LINEARLY (O(n)) for every packet
At 10,000+ services → noticeable latency increase

Solution: IPVS mode kube-proxy
- Uses kernel hash table (O(1) lookup)
- Supports more load-balancing algorithms (rr, lc, wrr, sh)
- Our Project 3 uses iptables mode (sufficient for <100 services)
```

**Debugging kube-proxy iptables:**
```bash
# Check if kube-proxy is running and mode
kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode

# Verify rules exist for a service
iptables -t nat -L KUBE-SERVICES -n | grep <ClusterIP>

# Watch rule updates in real-time
watch 'iptables -t nat -L KUBE-SVC-XXXXXX -n'

# If service isn't working — check endpoints exist
kubectl get endpoints <service-name>
# If endpoints empty → no pods matching selector, or pods not ready
```

---

### Q13: How does Istio intercept traffic using iptables?
**Project Reference:** Project 6 (Istio Service Mesh — Zero-Trust Networking)
**Expected Depth:** Explain init container iptables rules, traffic flow, why transparent to app

**Answer:**

In Project 6, Istio uses an init container (`istio-init`) that configures iptables rules in the pod's network namespace to transparently redirect ALL traffic through the Envoy sidecar.

**The iptables rules set by istio-init:**
```bash
# Redirect ALL inbound traffic to Envoy's inbound port (15006)
iptables -t nat -A PREROUTING -p tcp -j ISTIO_INBOUND

# ISTIO_INBOUND: redirect to port 15006 (Envoy inbound listener)
iptables -t nat -A ISTIO_INBOUND -p tcp --dport <app-port> \
    -j REDIRECT --to-port 15006

# Redirect ALL outbound traffic to Envoy's outbound port (15001)
iptables -t nat -A OUTPUT -p tcp -j ISTIO_OUTPUT

# ISTIO_OUTPUT: redirect to port 15001 (Envoy outbound listener)
# EXCEPT traffic from Envoy itself (UID 1337) — prevents infinite loop
iptables -t nat -A ISTIO_OUTPUT -m owner --uid-owner 1337 -j RETURN
iptables -t nat -A ISTIO_OUTPUT -p tcp -j REDIRECT --to-port 15001
```

**Complete traffic flow (Service A → Service B):**
```
Service A Pod:
┌──────────────────────────────────────────────────────────┐
│ App Container                                             │
│ curl http://service-b:8080/api                           │
│   → dst=service-b:8080                                   │
│   → OUTPUT chain → ISTIO_OUTPUT → REDIRECT to :15001    │
│                                                          │
│ Envoy Sidecar (port 15001 — outbound)                   │
│   → Resolves service-b via Pilot/xDS                     │
│   → Picks endpoint (load balance)                        │
│   → Encrypts with mTLS                                   │
│   → Applies retry/timeout/circuit-breaker policies       │
│   → Sends to Service B pod's IP:15006                    │
└──────────────────────────────────────────────────────────┘
                         ↓ (encrypted mTLS)
Service B Pod:
┌──────────────────────────────────────────────────────────┐
│ Envoy Sidecar (port 15006 — inbound)                     │
│   → PREROUTING chain → ISTIO_INBOUND → REDIRECT :15006  │
│   → Decrypts mTLS                                        │
│   → Verifies source identity (AuthorizationPolicy)       │
│   → Collects metrics (latency, status code)              │
│   → Forwards to localhost:8080 (app container)           │
│                                                          │
│ App Container (port 8080)                                │
│   → Receives plain HTTP request (doesn't know about TLS) │
└──────────────────────────────────────────────────────────┘
```

**Why this requires NET_ADMIN + NET_RAW capabilities:**
```yaml
initContainers:
- name: istio-init
  image: docker.io/istio/proxyv2
  securityContext:
    capabilities:
      add: [NET_ADMIN, NET_RAW]  # Modify iptables in pod's netns
    runAsUser: 0                  # Must be root for iptables
```

**Key exclusions (what ISN'T redirected):**
```bash
# Envoy's own traffic (prevents redirect loop)
-A ISTIO_OUTPUT -m owner --uid-owner 1337 -j RETURN

# Kubernetes probe traffic (kubelet health checks)
# Port 15020 (Istio healthcheck port) excluded from redirect

# Explicitly excluded ports (configured via annotation)
# traffic.sidecar.istio.io/excludeOutboundPorts: "3306,6379"
```

**Istio ambient mesh (the future — no sidecars):**
Instead of per-pod iptables + sidecar, ambient mode uses:
- **ztunnel** (per-node DaemonSet): handles mTLS at node level
- **waypoint proxy** (per-namespace): handles L7 policies
- No init container needed, no per-pod iptables modification
- This is where Project 6 would evolve to (mentioned in our Istio Q&A)

---

### Q14: How do you use the `ss` command? What information does it provide?
**Project Reference:** Project 9 (Infrastructure Validation — network connectivity checks), Project 3 (Kubernetes — debugging service connectivity)
**Expected Depth:** Show practical usage for troubleshooting, socket states, comparing to netstat

**Answer:**

`ss` (socket statistics) is the modern replacement for `netstat`. It's faster (reads directly from kernel via netlink) and provides more detail.

**Common usage patterns:**

```bash
# 1. List all listening TCP ports
ss -tlnp
# -t = TCP, -l = listening, -n = numeric (no DNS resolve), -p = show process
# Output:
# State  Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
# LISTEN 0      128    0.0.0.0:22          0.0.0.0:*         users:(("sshd",pid=1234))
# LISTEN 0      128    0.0.0.0:6443        0.0.0.0:*         users:(("kube-apiserver",pid=5678))

# 2. List all established connections to kube-apiserver
ss -tnp state established '( dport = :6443 )'

# 3. Count connections by state (troubleshoot connection leaks)
ss -s
# TCP:   540 (estab 320, closed 50, orphaned 12, timewait 100)

# 4. Find who's using port 8080
ss -tlnp | grep :8080
# LISTEN  0  128  *:8080  *:*  users:(("containerd-shim",pid=45678))

# 5. Show all connections to a specific remote (e.g., RDS endpoint)
ss -tn dst 10.0.21.45

# 6. Show connections in TIME_WAIT state (leak indicator)
ss -tn state time-wait | wc -l

# 7. Show CLOSE_WAIT connections (app not closing sockets properly)
ss -tnp state close-wait
# If many CLOSE_WAIT → YOUR app received FIN but hasn't closed the socket

# 8. Show UDP sockets (DNS, metrics)
ss -ulnp
# State  Recv-Q Send-Q Local:Port  Peer:Port  Process
# UNCONN 0      0      127.0.0.53%lo:53  0.0.0.0:*  users:(("systemd-resolve"))

# 9. Show socket memory usage (buffer sizes)
ss -tm
# skmem:(r0,rb131072,t0,tb16384,f0,w0,o0,bl0,d0)

# 10. Filter by process name
ss -tlnp | grep envoy    # All Envoy sidecar listeners
ss -tlnp | grep kubelet  # All kubelet ports
```

**Key socket states and what they mean:**
| State | Meaning | Action |
|-------|---------|--------|
| LISTEN | Waiting for connections | Normal for servers |
| ESTABLISHED | Active connection | Normal |
| TIME_WAIT | Connection closed, waiting for stale packets | Normal (2× MSL timeout) |
| CLOSE_WAIT | Remote closed, local hasn't closed yet | BUG in your app — not closing socket |
| FIN_WAIT1/2 | Local closed, waiting for remote acknowledgment | Brief — if stuck, remote is misbehaving |
| SYN_SENT | Connecting, waiting for SYN-ACK | If stuck — firewall/SG blocking, route wrong |
| SYN_RECV | Received SYN, sent SYN-ACK | If many — possible SYN flood attack |

**Practical K8s troubleshooting with ss:**
```bash
# "Service timeout" → check if pods are actually listening
kubectl exec -it <pod> -- ss -tlnp
# If port not in LISTEN → app crashed or wrong port configured

# "Connection refused" → port listening but too many connections?
kubectl exec -it <pod> -- ss -s
# Check Recv-Q in LISTEN state — if non-zero, accept queue is filling up

# "Intermittent 504" → connection pool exhausted?
kubectl exec -it <pod> -- ss -tn | wc -l
# If thousands of connections → connection pool leak, add connection timeout
```

---

### Q15: How do you use tcpdump for troubleshooting? How would you prove mTLS is working?
**Project Reference:** Project 6 (Istio — prove mTLS encryption), Project 9 (Infrastructure Validation — network debugging)
**Expected Depth:** Practical capture commands, filter expressions, prove encryption, real debugging scenarios

**Answer:**

**Basic tcpdump patterns for DevOps troubleshooting:**

```bash
# 1. Capture all traffic on an interface
tcpdump -i eth0 -nn    # -nn = don't resolve hostnames or ports

# 2. Capture traffic to/from specific host
tcpdump -i any host 10.244.1.5

# 3. Capture only TCP traffic on port 8080
tcpdump -i any tcp port 8080

# 4. Capture DNS queries (port 53)
tcpdump -i any port 53 -nn
# See: 10.244.1.5 > 10.96.0.10: A? web-svc.default.svc.cluster.local

# 5. Capture and write to file (analyze later in Wireshark)
tcpdump -i eth0 -w /tmp/capture.pcap -c 1000    # Stop after 1000 packets

# 6. Show packet contents (ASCII + hex)
tcpdump -i any -A port 80    # -A = ASCII, -X = hex+ASCII

# 7. Capture between two specific hosts
tcpdump -i any host 10.244.1.5 and host 10.244.2.8
```

**Proving mTLS is working (Project 6 — Istio):**

```bash
# Step 1: Capture traffic between two pods WITHOUT Istio
kubectl exec -it debug-pod -- tcpdump -i eth0 -A port 8080 -c 10
# You'll see: GET /api HTTP/1.1\r\nHost: service-b\r\n...
# PLAIN TEXT readable → NOT encrypted

# Step 2: Enable Istio STRICT mTLS, capture again
kubectl exec -it debug-pod -- tcpdump -i eth0 -A port 15006 -c 10
# You'll see: .....#.E.z..........TLS handshake followed by encrypted gibberish
# BINARY/ENCRYPTED → mTLS is working

# Step 3: More specifically, look for TLS ClientHello
tcpdump -i eth0 -nn 'tcp[((tcp[12:1] & 0xf0) >> 2)] = 0x16'
# 0x16 = TLS handshake record type
# If you see this between pod IPs → TLS is active

# Step 4: Verify with openssl (from inside pod)
kubectl exec -it app-pod -c istio-proxy -- \
  openssl s_client -connect service-b:8080 -CAfile /etc/certs/root-cert.pem
# Shows: Certificate chain, SPIFFE URI, TLS version
```

**Debugging scenario: Pod can't connect to external service**
```bash
# 1. Capture on pod's interface
kubectl exec -it <pod> -- tcpdump -i eth0 -nn host <external-ip> -c 20

# What to look for:
# SYN →         (outgoing connection attempt)
# SYN,ACK ←    (response received — connection works)
# SYN → ... SYN → ... SYN →  (retransmits — no response — blocked by SG/firewall)

# 2. If SYN goes out but no response:
#    → Security Group not allowing outbound
#    → NACL blocking return traffic
#    → Route table missing (no NAT Gateway for private subnet)

# 3. If RST comes back immediately:
#    → Port is closed on remote side
#    → Firewall actively rejecting (vs silently dropping)
```

**Debugging scenario: Intermittent connection resets**
```bash
# Capture RST packets
tcpdump -i any 'tcp[tcpflags] & (tcp-rst) != 0' -nn

# If you see RSTs from a load balancer IP:
# → Idle timeout exceeded (ALB default 60s)
# → Fix: Enable TCP keepalive in app (30s interval)
```

**tcpdump inside Kubernetes (various methods):**
```bash
# Method 1: exec into pod (if tcpdump installed)
kubectl exec -it <pod> -- tcpdump -i eth0

# Method 2: Use debug container (ephemeral container)
kubectl debug -it <pod> --image=nicolaka/netshoot --target=app -- tcpdump -i eth0

# Method 3: Use nsenter from node (access pod's netns)
# Find pod's PID on the node:
crictl inspect <container-id> | jq .info.pid
nsenter -t <PID> -n tcpdump -i eth0

# Method 4: Capture on node's veth (pod's virtual interface on host side)
# Find veth for pod:
ip link | grep <pod-if-index>
tcpdump -i vethXXXX
```

---

### Q16: How does DNS resolution work on Linux? Explain /etc/resolv.conf, nsswitch.conf, and systemd-resolved.
**Project Reference:** Project 2 (3-Tier AWS — EC2 DNS within VPC), Project 3 (Kubernetes — CoreDNS)
**Expected Depth:** Full resolution chain, caching layers, how VPC DNS works, troubleshooting

**Answer:**

**DNS resolution chain on a Linux system:**

```
Application calls: getaddrinfo("web-service.example.com")
       ↓
1. /etc/nsswitch.conf — determines resolution ORDER
   hosts: files dns mymachines
   (try /etc/hosts first, then DNS, then other sources)
       ↓
2. /etc/hosts — static lookups (checked first if nsswitch says "files" before "dns")
   127.0.0.1    localhost
   10.0.1.50    db-primary
       ↓ (if not found in /etc/hosts)
3. /etc/resolv.conf — tells system WHERE to query DNS
   nameserver 10.0.0.2        (VPC DNS resolver in AWS — .2 of CIDR)
   nameserver 127.0.0.53      (systemd-resolved local stub)
   search us-east-1.compute.internal  (append domain for short names)
   options ndots:5             (important for K8s — explained below)
       ↓
4. DNS query sent to nameserver
   → VPC DNS (10.0.0.2) or systemd-resolved cache
   → Recursive resolution if not cached
       ↓
5. Response returned to application
```

**systemd-resolved (modern Ubuntu/RHEL):**
```
Application → stub resolver (127.0.0.53) → systemd-resolved → upstream DNS

Benefits:
- Local DNS cache (faster repeated lookups)
- Per-link DNS configuration (different DNS per interface)
- DNSSEC validation
- mDNS/LLMNR for local discovery

Configuration:
/etc/systemd/resolved.conf
[Resolve]
DNS=10.0.0.2                    # Primary DNS
FallbackDNS=8.8.8.8            # Fallback
Domains=~.                      # Route all queries
Cache=yes
DNSStubListener=yes             # Listen on 127.0.0.53

Status:
resolvectl status               # Show current DNS config per interface
resolvectl query example.com    # Test resolution with details
```

**AWS VPC DNS specifics (Project 2):**
```
VPC CIDR: 10.0.0.0/16
VPC DNS Resolver: 10.0.0.2 (always base + 2)

Features:
- Resolves internal hostnames (ip-10-0-1-50.ec2.internal)
- Resolves private hosted zones (internal.company.com)
- Forwards external queries to public DNS
- Route53 Resolver for hybrid (on-prem ↔ AWS)

EC2 instances get:
/etc/resolv.conf:
  nameserver 10.0.0.2
  search us-east-1.compute.internal
```

**Troubleshooting DNS:**
```bash
# Test resolution with specific server
dig @10.0.0.2 web-service.example.com

# Check which server is being used
dig +trace example.com

# Check if caching issue (bypass cache)
dig +nocache example.com
resolvectl flush-caches

# Check systemd-resolved statistics
resolvectl statistics

# Common issues:
# 1. /etc/resolv.conf symlinked wrong (not pointing to resolved)
ls -la /etc/resolv.conf
# Should be → /run/systemd/resolve/stub-resolv.conf

# 2. DNS over VPN — split DNS not configured
resolvectl domain tun0          # Check if VPN interface has search domain

# 3. MTU issues causing DNS failures (large responses truncated)
dig +tcp example.com            # Force TCP (bypass UDP size limit)
```



---

### Q17: How does CoreDNS work in Kubernetes? What's in a pod's /etc/resolv.conf?
**Project Reference:** Project 3 (Kubernetes — service discovery via DNS), Project 1 (DevSecOps — microservice communication)
**Expected Depth:** CoreDNS architecture, pod DNS config, ndots optimization, debugging DNS in K8s

**Answer:**

**CoreDNS in Kubernetes:**

CoreDNS runs as a Deployment in `kube-system` namespace (typically 2 replicas) and is the cluster's DNS server. Every pod uses it for service discovery.

```
Pod makes DNS query: "web-svc.app-prod.svc.cluster.local"
       ↓
Pod's /etc/resolv.conf → nameserver 10.96.0.10 (CoreDNS ClusterIP)
       ↓
CoreDNS receives query
       ↓
CoreDNS checks: Is this a cluster domain (.cluster.local)?
  YES → Look up in kubernetes plugin (watches API server for Services/Endpoints)
       → Returns ClusterIP: 10.96.50.100
  NO  → Forward to upstream DNS (/etc/resolv.conf of CoreDNS pod → VPC DNS 10.0.0.2)
```

**Pod's /etc/resolv.conf (auto-generated by kubelet):**
```bash
# Inside any pod:
cat /etc/resolv.conf

nameserver 10.96.0.10          # CoreDNS ClusterIP
search app-prod.svc.cluster.local svc.cluster.local cluster.local us-east-1.compute.internal
options ndots:5
```

**DNS resolution with search domains and ndots:**
```
ndots:5 means: if the query has FEWER than 5 dots, try search domains first.

Example: Pod calls "web-svc" (0 dots < 5):
  Try: web-svc.app-prod.svc.cluster.local  ← FOUND (same namespace)
  
Example: Pod calls "web-svc.other-ns" (1 dot < 5):
  Try: web-svc.other-ns.app-prod.svc.cluster.local  ← NOT FOUND
  Try: web-svc.other-ns.svc.cluster.local  ← FOUND (cross-namespace)

Example: Pod calls "api.external.com" (2 dots < 5):
  Try: api.external.com.app-prod.svc.cluster.local  ← NOT FOUND
  Try: api.external.com.svc.cluster.local  ← NOT FOUND
  Try: api.external.com.cluster.local  ← NOT FOUND
  Try: api.external.com.us-east-1.compute.internal  ← NOT FOUND
  Try: api.external.com.  ← FOUND (absolute, forwarded to upstream)
  
  5 failed queries before the real one! (DNS amplification problem)
```

**Optimizing DNS (production):**
```yaml
# Option 1: Use FQDN with trailing dot (skips search domains)
url: "http://api.external.com."   # Trailing dot = absolute, no search

# Option 2: Reduce ndots for pods making many external calls
apiVersion: v1
kind: Pod
spec:
  dnsConfig:
    options:
    - name: ndots
      value: "2"     # Only try search domains if < 2 dots

# Option 3: NodeLocal DNSCache (reduces CoreDNS load)
# DaemonSet running dns cache on each node at 169.254.20.10
# Pod → local cache → CoreDNS (only on cache miss)
```

**CoreDNS Corefile (configuration):**
```
.:53 {
    errors
    health { lameduck 5s }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
        ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf { max_concurrent 1000 }
    cache 30
    loop
    reload
    loadbalance
}
```

**Debugging DNS in Kubernetes:**
```bash
# 1. Test from inside a pod
kubectl exec -it debug-pod -- nslookup web-svc.app-prod.svc.cluster.local
kubectl exec -it debug-pod -- dig @10.96.0.10 web-svc.app-prod.svc.cluster.local

# 2. Check CoreDNS pods are running
kubectl get pods -n kube-system -l k8s-app=kube-dns

# 3. Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# 4. Check if Service has endpoints (no endpoints = no DNS A record for headless)
kubectl get endpoints web-svc -n app-prod

# 5. Verify pod's resolv.conf
kubectl exec -it <pod> -- cat /etc/resolv.conf
```

---

### Q18: Troubleshooting: EC2 instance can't reach the internet from a private subnet.
**Project Reference:** Project 2 (3-Tier AWS Architecture — private subnet EC2 instances)
**Expected Depth:** Systematic debugging approach, all possible causes, how you'd fix each

**Answer:**

**Systematic troubleshooting checklist (in order):**

```
Step 1: Verify the symptom
$ curl -v https://google.com
$ ping 8.8.8.8
$ dig google.com

→ If ping fails but DNS works: routing/firewall issue
→ If DNS fails: DNS misconfiguration (separate problem)
→ If everything times out: no outbound path
```

**Step 2: Check route table**
```bash
# On the instance
ip route show
# Look for: default via 10.0.1.1 dev eth0 (gateway should be subnet router)

# In AWS Console/Terraform:
# Private subnet route table must have:
0.0.0.0/0 → nat-gateway-id (NOT internet-gateway!)

# Common mistakes:
# ❌ Route to IGW (only works for public subnets with public IPs)
# ❌ No default route at all (instance has no way out)
# ❌ NAT Gateway in SAME private subnet (must be in PUBLIC subnet)
# ❌ Route table not associated with the correct subnet
```

**Step 3: Check NAT Gateway**
```bash
# NAT Gateway requirements:
# 1. Must be in a PUBLIC subnet (has route to IGW)
# 2. Must have an Elastic IP attached
# 3. Must be in state "Available" (not "Failed" or "Pending")
# 4. The PUBLIC subnet's route table must have: 0.0.0.0/0 → IGW

# Check in AWS:
aws ec2 describe-nat-gateways --filter Name=state,Values=available
```

**Step 4: Check Security Groups (stateful)**
```bash
# SG is stateful — if outbound is allowed, return traffic auto-allowed
# Default SG: allows ALL outbound
# Check: Is there a restrictive outbound rule?

# Common mistake: Custom SG with only specific outbound rules
# e.g., only allows port 443 outbound → can't do yum update (needs port 80 too)
# e.g., only allows traffic to specific CIDR → can't reach internet
```

**Step 5: Check NACLs (stateless)**
```bash
# NACLs are STATELESS — must explicitly allow BOTH directions

# Outbound: Allow TCP 80, 443 to 0.0.0.0/0  ✓
# Inbound: Allow TCP 1024-65535 from 0.0.0.0/0  ← RETURN traffic (ephemeral ports)

# Common mistake: NACL allows outbound 443 but BLOCKS inbound ephemeral ports
# Result: Connection hangs (SYN goes out, SYN-ACK comes back but NACL drops it)

# Check:
aws ec2 describe-network-acls --filters Name=association.subnet-id,Values=<subnet-id>
```

**Step 6: Check DNS resolution**
```bash
# If ping by IP works but curl by hostname fails → DNS issue
cat /etc/resolv.conf
# Should show: nameserver 10.0.0.2 (VPC DNS)

# VPC must have:
# enableDnsSupport: true
# enableDnsHostnames: true
```

**Step 7: Check instance networking**
```bash
# Is the ENI attached properly?
ip addr show
# Should have private IP in subnet range

# Is there a public IP? (shouldn't be needed in private subnet with NAT)
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

**Decision tree summary:**
```
Can't reach internet from private subnet:
├── ping 8.8.8.8 times out?
│   ├── Route table has 0.0.0.0/0 → NAT-GW?
│   │   ├── NO → Add route to NAT Gateway
│   │   └── YES → NAT Gateway in Available state?
│   │       ├── NO → NAT-GW failed (check EIP, check public subnet)
│   │       └── YES → Check SG outbound rules
│   │           └── Check NACL (both directions!)
│   └── Is it a NAT instance (not gateway)? Check src/dest check disabled
├── ping works but HTTPS doesn't?
│   └── SG/NACL allows ICMP but blocks TCP 443
└── DNS fails?
    └── VPC DNS settings, /etc/resolv.conf, SG allows UDP 53
```

**In Project 2:** Our Terraform creates NAT Gateways in public subnets with proper routes. App-tier EC2 instances in private subnets use the NAT Gateway for outbound (yum updates, API calls). We've hit this issue when a NAT Gateway hit a bandwidth limit during peak traffic — switched to multiple NAT Gateways (one per AZ) for redundancy and throughput.



---

### Q19: Troubleshooting: Pod can't resolve DNS. Walk through your debugging process.
**Project Reference:** Project 3 (Kubernetes — CoreDNS issues), Project 1 (DevSecOps — application connectivity)
**Expected Depth:** Systematic approach, all layers, common root causes, how to fix each

**Answer:**

**Systematic debugging approach (layered):**

**Layer 1: Verify the symptom from inside the pod**
```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
# If this fails → fundamental DNS broken
# If this works but app DNS fails → app-specific issue

kubectl exec -it <pod> -- cat /etc/resolv.conf
# Verify: nameserver points to CoreDNS ClusterIP (10.96.0.10)
# Verify: search domains present
# Verify: ndots value
```

**Layer 2: Is CoreDNS running?**
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
# Are pods Running? Are they Ready (1/1)?

kubectl get svc -n kube-system kube-dns
# Does the service have endpoints?
kubectl get endpoints -n kube-system kube-dns
# If endpoints empty → CoreDNS pods not ready, label mismatch, or crashed
```

**Layer 3: Can pod reach CoreDNS?**
```bash
# Test direct connectivity to CoreDNS pod IP
kubectl exec -it <pod> -- wget -O- -T5 <coredns-pod-ip>:8080/health
# If times out → NetworkPolicy blocking DNS traffic

# Check for NetworkPolicy blocking DNS
kubectl get networkpolicy -n <pod-namespace>
# Common mistake: default-deny policy without exception for DNS (port 53)
# Fix: Add egress rule allowing UDP/TCP 53 to kube-system namespace
```

**Layer 4: CoreDNS logs**
```bash
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=50
# Look for:
# "SERVFAIL" — upstream DNS not responding
# "connection refused" — can't reach upstream
# "i/o timeout" — network issue to upstream DNS
# "NXDOMAIN" — normal (name doesn't exist)
# "loop detected" — CoreDNS querying itself (misconfigured forward)
```

**Layer 5: Node-level DNS**
```bash
# SSH to the node where pod is running
# Check if node itself can resolve DNS
dig @10.0.0.2 google.com    # VPC DNS
# If node DNS broken → bigger problem (VPC DNS, DHCP options)

# Check iptables (kube-proxy must have rules for CoreDNS service)
iptables -t nat -L KUBE-SERVICES | grep dns
```

**Common root causes and fixes:**

| Root Cause | Symptom | Fix |
|-----------|---------|-----|
| CoreDNS pods OOMKilled | Intermittent DNS failures | Increase memory limit (default 170Mi often insufficient at scale) |
| NetworkPolicy blocks DNS | Pod-specific DNS failure | Add egress allow: UDP 53 to kube-system |
| CoreDNS overwhelmed | Slow DNS, timeouts | Add NodeLocal DNSCache, increase replicas |
| Upstream DNS unreachable | External names fail, internal works | Check node's /etc/resolv.conf, VPC DNS settings |
| ndots:5 + many external calls | Slow resolution, 5× queries per lookup | Reduce ndots or use FQDNs with trailing dot |
| conntrack table full | Random DNS failures across cluster | Increase `nf_conntrack_max`, add NodeLocal DNSCache (uses TCP) |
| CoreDNS loop | All DNS fails, CoreDNS crash-loops | Remove loop in Corefile, check `forward . /etc/resolv.conf` doesn't point back to itself |

**The conntrack DNS issue (common at scale):**
```
Problem: DNS uses UDP → conntrack entry per query
High-traffic cluster: 1000 pods × 10 queries/sec = 10,000 conntrack entries/sec
If nf_conntrack_max reached → NEW packets dropped → DNS randomly fails

Fix 1: Increase conntrack table
sysctl -w net.netfilter.nf_conntrack_max=1000000

Fix 2: NodeLocal DNSCache (recommended)
- Runs on each node at 169.254.20.10
- Pod → local cache (no conntrack for cached responses)
- Cache miss → CoreDNS (TCP, single conntrack entry)
- Reduces CoreDNS load by 80%+
```

**Quick diagnostic one-liner:**
```bash
# Deploy a debug pod and test everything
kubectl run dns-debug --image=busybox --restart=Never -- sleep 3600
kubectl exec dns-debug -- nslookup kubernetes.default
kubectl exec dns-debug -- nslookup google.com
kubectl exec dns-debug -- wget -O- -T5 http://web-svc.app-prod:8080/health
```



---

### Q20: Explain `ip` command usage: ip addr, ip route, ip link, ip netns.
**Project Reference:** Project 3 (Kubernetes — debugging pod networking, Calico CNI), Project 6 (Istio — network namespace inspection)
**Expected Depth:** Practical usage for container/K8s networking debugging, relate to veth pairs and pod networking

**Answer:**

The `ip` command (from iproute2) is the modern replacement for `ifconfig`, `route`, and other legacy tools. It's essential for debugging container and Kubernetes networking.

**ip addr — Interface addresses:**
```bash
# Show all interfaces and their IPs
ip addr show
# or: ip a

# Key output on a K8s node:
# 1: lo: <LOOPBACK> mtu 65536
#     inet 127.0.0.1/8 scope host lo
# 2: eth0: <BROADCAST,MULTICAST,UP>  mtu 9001  (EC2 instance)
#     inet 10.0.1.50/24 brd 10.0.1.255 scope global dynamic eth0
# 4: cali123abc@if3: <BROADCAST,MULTICAST,UP>  (veth to pod)
#     link/ether ee:ee:ee:ee:ee:ee
# 5: cali456def@if3: <BROADCAST,MULTICAST,UP>  (another pod's veth)

# Show specific interface
ip addr show eth0

# Add IP to interface (debugging/testing)
ip addr add 192.168.1.100/24 dev eth0

# Useful for K8s:
# - Verify node has correct IP
# - See all veth interfaces (one per pod on this node)
# - Check if interface is UP
```

**ip route — Routing table:**
```bash
# Show routing table
ip route show
# or: ip r

# K8s node typical output:
# default via 10.0.1.1 dev eth0               ← Default gateway (to internet via NAT-GW)
# 10.0.1.0/24 dev eth0 proto kernel scope link ← Local subnet (direct)
# 10.244.0.0/26 dev cali123 scope link         ← Pod subnet on THIS node (Calico)
# 10.244.1.0/26 via 10.0.1.51 dev eth0         ← Pod subnet on OTHER node (routed via node IP)
# blackhole 10.244.0.0/26 proto bird            ← Calico BGP blackhole for local pod CIDR

# Add a route (e.g., for debugging)
ip route add 10.244.5.0/24 via 10.0.1.52

# Delete a route
ip route del 10.244.5.0/24

# Show route for specific destination (which path will a packet take?)
ip route get 10.244.1.5
# 10.244.1.5 via 10.0.1.51 dev eth0 src 10.0.1.50
# This tells you: to reach pod 10.244.1.5, go via node 10.0.1.51
```

**ip link — Interface state and properties:**
```bash
# Show all interfaces (link layer info)
ip link show
# or: ip l

# Key info: MTU, state (UP/DOWN), MAC address, type
# 4: cali123@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
#     link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-xxx

# Bring interface up/down
ip link set eth0 up
ip link set eth0 down

# Change MTU (debugging MTU issues — common in overlay networks)
ip link set eth0 mtu 1450

# Create a veth pair (how CNI plugins create pod networking)
ip link add veth-host type veth peer name veth-pod
ip link set veth-pod netns <pod-netns-name>

# Useful for K8s debugging:
# - Check if veth is UP (link-down = container crashed)
# - Verify MTU (overlay networks often need lower MTU)
# - See which netns a veth belongs to (link-netns field)
```

**ip netns — Network namespace management:**
```bash
# List all named network namespaces
ip netns list
# Output: cni-xxxx-yyyy-zzzz (one per pod, created by CNI)

# Execute command inside a network namespace
ip netns exec cni-xxxx ip addr show
# Shows: pod's interfaces (eth0 with pod IP, lo)

ip netns exec cni-xxxx ip route show
# Shows: pod's routing table (default via 169.254.1.1 — Calico)

ip netns exec cni-xxxx ss -tlnp
# Shows: what's listening inside that pod's network

# Identify which netns belongs to which pod:
# 1. Get pod's interface index
kubectl exec <pod> -- cat /sys/class/net/eth0/iflink
# Output: 42

# 2. On the node, find interface with that index
ip link | grep "^42:"
# 42: caliXXXXX@if3 → this is the host-side veth for that pod
```

**Practical K8s networking debug flow:**
```bash
# "Pod can't reach another pod" — full debugging:

# 1. Check pod has IP
kubectl exec <pod> -- ip addr show eth0

# 2. Check pod's route table
kubectl exec <pod> -- ip route show
# Should have: default via 169.254.1.1 dev eth0 (Calico)

# 3. On the node, check route to destination pod
ip route get <dest-pod-ip>
# Shows next hop (should be direct if same node, via other node if remote)

# 4. Check the veth is up on host side
ip link show | grep <veth-for-pod>

# 5. If cross-node: check BGP routes (Calico)
calicoctl node status
# Shows: BGP peers established, routes exchanged
```



---

### Q21: What is a veth pair and how does container networking use them?
**Project Reference:** Project 3 (Kubernetes — Calico CNI pod networking), Project 6 (Istio — sidecar shares pod network namespace)
**Expected Depth:** Explain the mechanism, how CNI creates them, how pods communicate, diagram the flow

**Answer:**

**veth (Virtual Ethernet)** pairs are like a virtual network cable with two ends. Whatever enters one end comes out the other. They're the fundamental mechanism connecting containers to the host network.

**How a veth pair works:**
```
┌─────────────────────────────────────────────────────────────┐
│ Host Network Namespace                                       │
│                                                              │
│  ┌─────────┐    veth pair    ┌─────────────────────────┐   │
│  │ cali123 │◄══════════════►│ Pod Network Namespace     │   │
│  │ (host   │  (virtual      │                           │   │
│  │  end)   │   cable)       │   eth0 (pod end)          │   │
│  └────┬────┘                │   IP: 10.244.0.5/32       │   │
│       │                     └─────────────────────────────┘   │
│       │                                                      │
│  ┌────▼────────────────────────────┐                        │
│  │ Routing table / Bridge / iptables│                        │
│  │ Routes pod traffic to correct    │                        │
│  │ destination (local or remote)    │                        │
│  └─────────────────────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

**How CNI (Calico) sets up pod networking in Project 3:**

```bash
# Step 1: kubelet tells CNI "create network for this pod"
# Step 2: Calico CNI plugin executes:

# Create veth pair
ip link add cali123 type veth peer name eth0

# Move one end into pod's network namespace
ip link set eth0 netns /proc/<pod-pid>/ns/net

# Configure host end
ip link set cali123 up
ip route add 10.244.0.5/32 dev cali123  # Route pod IP to this veth

# Configure pod end (inside pod's netns)
ip netns exec <pod-ns> ip addr add 10.244.0.5/32 dev eth0
ip netns exec <pod-ns> ip link set eth0 up
ip netns exec <pod-ns> ip route add default via 169.254.1.1 dev eth0
# 169.254.1.1 is Calico's proxy-ARP trick — no bridge needed

# Step 3: Calico announces 10.244.0.5/32 via BGP to other nodes
```

**Traffic flow — Pod A (node1) → Pod B (node2):**
```
Pod A (10.244.0.5, node1)
│
├── eth0 (pod end of veth) → packet: src=10.244.0.5, dst=10.244.1.8
│
├── cali123 (host end of veth) — packet enters host namespace
│
├── Host routing table:
│   10.244.1.0/26 via 10.0.1.51 dev eth0  (learned via BGP from node2)
│
├── eth0 (node1's real NIC) → packet forwarded to node2 (10.0.1.51)
│   (In AWS: VPC routing handles this — no BGP needed if using VPC CNI)
│
├── eth0 (node2's real NIC) ← packet arrives
│
├── Host routing table on node2:
│   10.244.1.8/32 dev cali456  (local pod, route to its veth)
│
├── cali456 (host end of veth on node2)
│
└── eth0 (pod end of veth) → Pod B receives packet
```

**Traffic flow — Pod A → Pod B (SAME node):**
```
Pod A eth0 → cali123 (host ns) → routing table → cali456 → Pod B eth0
(Never leaves the node — no bridge needed with Calico's routing approach)
```

**Why Kubernetes pods in the same pod share a network namespace:**
```
Pod with 2 containers (app + sidecar):
┌─────────────────────────────────────────┐
│ Pod Network Namespace (shared)           │
│   eth0 → veth → host                    │
│   IP: 10.244.0.5                        │
│                                          │
│   ┌──────────┐    ┌──────────────┐      │
│   │ App      │    │ Envoy Sidecar│      │
│   │ :8080    │    │ :15001,15006 │      │
│   └──────────┘    └──────────────┘      │
│                                          │
│   Both containers share eth0, same IP    │
│   Can reach each other via localhost     │
│   Istio iptables in THIS netns redirect  │
│   app traffic through Envoy             │
└─────────────────────────────────────────┘
```

**Debugging veth pairs:**
```bash
# Find which veth on host belongs to which pod:
# Inside pod:
cat /sys/class/net/eth0/iflink    # Returns: 42 (interface index on host)

# On host:
ip link | grep "^42:"             # Shows: 42: caliXXXX@if3

# Verify veth is passing traffic:
ip -s link show cali123
# Shows TX/RX byte counts — if increasing, traffic is flowing

# Check for packet drops on veth:
ip -s link show cali123 | grep -i drop
```



---

### Q22: Explain TCP connection states (ESTABLISHED, TIME_WAIT, CLOSE_WAIT). How do you troubleshoot connection leaks?
**Project Reference:** Project 3 (Kubernetes — intermittent 504 debugging), Project 2 (3-Tier AWS — RDS connection pool issues)
**Expected Depth:** Full TCP state diagram understanding, identify leak patterns, practical remediation

**Answer:**

**TCP Connection States — The Full Lifecycle:**

```
CLIENT                                  SERVER
  |                                        |
  |--- SYN ------->  [SYN_SENT]          |
  |                   [SYN_RECV] <--- SYN+ACK ---|
  |--- ACK ------->  [ESTABLISHED] <----> [ESTABLISHED]
  |                                        |
  |   ... data transfer ...                |
  |                                        |
  |--- FIN ------->  [FIN_WAIT_1]        [CLOSE_WAIT]
  |                   [FIN_WAIT_2] <--- ACK ---|
  |                   [TIME_WAIT]  <--- FIN ---|
  |--- ACK ------->                       [LAST_ACK]
  |                                        [CLOSED]
  | (waits 2×MSL)                          |
  | [CLOSED]                               |
```

**Key states and what they mean for troubleshooting:**

| State | Who | Meaning | Concern |
|-------|-----|---------|---------|
| **ESTABLISHED** | Both | Active connection, data flowing | Normal — but too many = connection pool not releasing |
| **TIME_WAIT** | Closer | Connection closed, waiting for stale packets (2×MSL = 60s default) | Normal after close — but thousands = high connection churn |
| **CLOSE_WAIT** | Receiver | Remote closed (sent FIN), local hasn't closed socket yet | **BUG in YOUR code** — not calling close() |
| **FIN_WAIT_1** | Closer | Sent FIN, waiting for ACK | Brief — if stuck, remote not responding |
| **FIN_WAIT_2** | Closer | Got ACK for FIN, waiting for remote's FIN | If stuck — remote app hung (not closing) |
| **SYN_SENT** | Client | Sent SYN, no response | Firewall blocking, wrong IP, host down |
| **SYN_RECV** | Server | Got SYN, sent SYN+ACK, waiting | Many = SYN flood attack |
| **LAST_ACK** | Receiver | Sent FIN, waiting for final ACK | Brief — if stuck, network issue |

**Troubleshooting patterns:**

**Pattern 1: Many CLOSE_WAIT (APPLICATION BUG)**
```bash
ss -tnp state close-wait | wc -l
# 5000+ CLOSE_WAIT connections

# This means: Remote side closed connections, but YOUR app isn't closing its end
# Common causes:
# - HTTP client not calling response.Body.Close() (Go)
# - JDBC connection not returned to pool (Java)
# - Python requests session not closed
# - File descriptor leak (open sockets never closed)

# Diagnosis:
ss -tnp state close-wait
# Look at the process column — which process is holding these?
# users:(("java",pid=12345,fd=4567))

# Fix: Fix the code — ensure sockets are closed in finally/defer blocks
# Temporary: Restart the pod (connections will be cleaned up)
# Monitor: Alert when CLOSE_WAIT count > threshold
```

**Pattern 2: Many TIME_WAIT (HIGH CHURN — usually OK)**
```bash
ss -tn state time-wait | wc -l
# 20000+ TIME_WAIT connections

# This means: Your app is creating and closing MANY short-lived connections
# Common in: Microservices without connection pooling, PHP apps, batch processors

# Why TIME_WAIT exists:
# - Prevents old delayed packets from being interpreted by new connections on same port
# - Lasts 2×MSL (2×30s = 60s on Linux)

# When it's a problem:
# - Exhausts local port range (only 64K ephemeral ports)
# - Error: "Cannot assign requested address" (no ports available)

# Fixes:
# 1. Enable connection reuse
sysctl -w net.ipv4.tcp_tw_reuse=1    # Reuse TIME_WAIT for new outbound connections

# 2. Increase port range
sysctl -w net.ipv4.ip_local_port_range="1024 65535"

# 3. Use connection pooling (BEST fix — avoid creating connections per request)
# HTTP: Keep-Alive, connection pool size 20-50 per upstream
# DB: Connection pool (HikariCP, pgbouncer)

# 4. In K8s: If service mesh (Istio) handles connections, it pools by default
```

**Pattern 3: Many ESTABLISHED (CONNECTION POOL LEAK)**
```bash
ss -tn state established | wc -l
# 10000+ ESTABLISHED connections (growing over time)

# This means: Connections opened but never closed (leaked)
# Different from CLOSE_WAIT — here neither side has initiated close

# Common causes:
# - Connection pool max size too high or no timeout
# - Background connections that are "forgotten"
# - Load balancer/proxy keeping connections that backends dropped

# Diagnosis:
ss -tn state established dst :5432 | wc -l    # Connections to DB
# Compare with connection pool config — if actual > max pool → leak

# Fix:
# - Set connection max lifetime (e.g., 5 minutes — force recycle)
# - Set idle timeout (close connections idle for > 60s)
# - Monitor: connection_pool_active vs connection_pool_max → alert if approaching
```

**Real Kubernetes scenario (from Project 3 — intermittent 504s):**
```
Symptom: Random 504 Gateway Timeout from Ingress Controller
App logs: Clean (no errors)
CPU/Memory: Fine

Investigation:
1. kubectl exec -it nginx-ingress-pod -- ss -tn state close-wait | wc -l
   → 2000+ CLOSE_WAIT connections to backend pods

2. Root cause: Backend pods were restarting (rolling update).
   When pod terminates → sends FIN to ingress.
   Ingress receives FIN but holds CLOSE_WAIT (connection pool keeps reference).
   Next request on that "dead" connection → 504 timeout.

3. Fix:
   - Ingress: Set proxy_next_upstream (retry on another pod)
   - Backend: Add preStop hook with sleep 5s (allow de-registration before shutdown)
   - Connection pool: Set max idle timeout to 30s (release dead connections)
```

**Monitoring commands summary:**
```bash
# Quick health check — count by state
ss -s
# TCP:   1500 (estab 800, closed 100, orphaned 5, timewait 400, close-wait 50)

# Per-state detailed count
ss -tan | awk '{print $1}' | sort | uniq -c | sort -rn
#   800 ESTAB
#   400 TIME-WAIT
#    50 CLOSE-WAIT  ← Investigate if growing
#    15 FIN-WAIT-2
#     5 SYN-SENT   ← Investigate if stuck

# Watch connections to specific destination over time
watch 'ss -tn dst :8080 | wc -l'

# In Kubernetes — expose connection metrics via Prometheus
# node_netstat_Tcp_CurrEstab (node-exporter metric)
# Track over time — growing ESTABLISHED = leak
```

---

*End of Linux Kernel & Networking Q&A Bank — 22 questions covering Categories 3 & 4*

