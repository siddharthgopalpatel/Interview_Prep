# 🚀 Kubernetes Notes — Production-Grade Study Guide

> **Audience:** DevOps/Cloud Engineer with 12+ YOE  
> **Focus:** Cluster Setup, Architecture, Networking — kubeadm on Linux  
> **Style:** Simple English | Diagrams | Examples | OpenShift Comparison

---

## 📌 Topic Relevance Guide

| Symbol | Meaning |
|--------|---------|
| ✅ MUST HAVE | Critical for 12 YOE — expect deep questions |
| ⚡ GOOD TO KNOW | Adds depth, quick review |
| 🔴 OPENSHIFT DIFF | Different in OpenShift — explained inline |

---

---

# 🏗️ SECTION 1: Kubernetes Cluster Architecture

---

## 1.1 Cluster Topology — What Makes a K8s Cluster ✅ MUST HAVE

A Kubernetes cluster = **Control Plane** (brain) + **Worker Nodes** (muscle).

```
┌──────────────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌─────────── Control Plane (k8s-cp) ──────────────────────┐     │
│  │                                                           │     │
│  │  ┌──────────┐  ┌────────────┐  ┌───────────────────┐    │     │
│  │  │ API      │  │ etcd       │  │ Controller        │    │     │
│  │  │ Server   │  │ (key-value │  │ Manager           │    │     │
│  │  │ :6443    │  │  store)    │  │ :10257            │    │     │
│  │  └──────────┘  └────────────┘  └───────────────────┘    │     │
│  │  ┌──────────┐  ┌────────────┐                            │     │
│  │  │ Scheduler│  │ kubelet    │                            │     │
│  │  │ :10259   │  │ :10250     │                            │     │
│  │  └──────────┘  └────────────┘                            │     │
│  └───────────────────────────────────────────────────────────┘     │
│                           │                                         │
│                      API :6443                                      │
│                           │                                         │
│  ┌──── Worker 01 ─────┐  │  ┌──── Worker 02 ─────┐               │
│  │  ┌───────┐         │  │  │  ┌───────┐         │               │
│  │  │kubelet│         │  │  │  │kubelet│         │               │
│  │  │:10250 │         │  │  │  │:10250 │         │               │
│  │  └───────┘         │◄─┘─►│  └───────┘         │               │
│  │  ┌───────┐         │     │  ┌───────┐         │               │
│  │  │kube-  │         │     │  │kube-  │         │               │
│  │  │proxy  │         │     │  │proxy  │         │               │
│  │  │:10256 │         │     │  │:10256 │         │               │
│  │  └───────┘         │     │  └───────┘         │               │
│  │  ┌───────────────┐ │     │  ┌───────────────┐ │               │
│  │  │  Pods / Apps  │ │     │  │  Pods / Apps  │ │               │
│  │  └───────────────┘ │     │  └───────────────┘ │               │
│  └─────────────────────┘     └─────────────────────┘               │
└──────────────────────────────────────────────────────────────────┘
```

### Component Summary

| Component | Role | Runs On |
|-----------|------|---------|
| **API Server** | Front door — all requests go through it | Control Plane |
| **etcd** | Key-value store — cluster state/config | Control Plane |
| **Controller Manager** | Ensures desired state (replicas, nodes, etc.) | Control Plane |
| **Scheduler** | Picks which node gets a new pod | Control Plane |
| **kubelet** | Node agent — talks to container runtime | Every node |
| **kube-proxy** | Network rules for Service traffic | Every node |

### 🔴 OpenShift Difference

| Aspect | Kubernetes (kubeadm) | OpenShift |
|--------|---------------------|-----------|
| Installer | `kubeadm init` / `kubeadm join` | `openshift-install` (IPI/UPI) |
| Control Plane | Standard K8s components | Same components + **OpenShift API Server** + **OAuth Server** |
| etcd | Managed manually or by kubeadm | Managed by **etcd Operator** (auto backup, scaling) |
| Extra layer | None | **Operators for everything** — cluster self-manages |
| Console | No built-in UI | Full Web Console out of the box |

> **Interview tip:** "OpenShift IS Kubernetes underneath — but it adds opinionated layers (Operators, OAuth, Routes, built-in registry) on top."

---

## 1.2 Lab Planning — Sizing & Network ✅ MUST HAVE

### Minimum Lab Setup

| Hostname | Role | IP | vCPU | RAM |
|----------|------|----|------|-----|
| k8s-cp | Control Plane | 192.168.56.108 | 2 | 8 GiB |
| worker01 | Worker | 192.168.56.109 | 2 | 8 GiB |
| worker02 | Worker | 192.168.56.110 | 2 | 8 GiB |

**Host workstation:** 8 GB min for 2 VMs, 16 GB+ for 3 VMs.

### Network CIDRs — Don't Overlap!

```
┌─────────────────────────────────────────┐
│         Network CIDR Layout             │
├─────────────────────────────────────────┤
│                                         │
│  Node Network:    192.168.56.0/24       │
│  Pod CIDR:        192.168.0.0/16        │  ← Calico uses this
│  Service CIDR:    10.96.0.0/12          │  ← kubeadm default
│                                         │
│  ⚠️  Pod CIDR must NOT overlap with:    │
│     - Node network                      │
│     - Service CIDR                      │
│     - VPN ranges                        │
└─────────────────────────────────────────┘
```

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| Pod CIDR default | You choose (commonly 10.244.0.0/16 or 192.168.0.0/16) | 10.128.0.0/14 |
| Service CIDR default | 10.96.0.0/12 | 172.30.0.0/16 |
| Machine sizing | 2 vCPU / 8 GiB min | 4 vCPU / 16 GiB min (control plane needs more) |

---

## 1.3 Required Ports ⚡ GOOD TO KNOW

### Control Plane Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 6443 | TCP | API Server (most important!) |
| 2379-2380 | TCP | etcd |
| 10250 | TCP | kubelet API |
| 10257 | TCP | Controller Manager |
| 10259 | TCP | Scheduler |

### Worker Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 10250 | TCP | kubelet API |
| 10256 | TCP | kube-proxy health |
| 30000-32767 | TCP/UDP | NodePort Services |

### Calico Overlay Ports

| Encapsulation | Allow |
|--------------|-------|
| VXLAN (default) | UDP 4789 between nodes |
| IPIP | IP protocol 4 |
| BGP | TCP 179 |

---

---

# 🛠️ SECTION 2: Node Preparation (All Nodes)

---

## 2.1 Hostname & /etc/hosts ✅ MUST HAVE

Every node needs a unique hostname. kubelet registers with this name.

```bash
# On control-plane:
hostnamectl set-hostname k8s-cp

# On workers:
hostnamectl set-hostname worker01
```

Add all nodes to `/etc/hosts` on **every machine**:

```bash
cat >> /etc/hosts <<'EOF'
192.168.56.108 k8s-cp
192.168.56.109 worker01
192.168.56.110 worker02
EOF
```

Verify: `hostname -f` → should return the short hostname.

---

## 2.2 Time Sync (NTP/Chrony) ✅ MUST HAVE

**Why?** K8s certificates and token expiry depend on accurate clocks. Skewed clocks = broken bootstrap.

```bash
# Check:
timedatectl status
# Look for: "System clock synchronized: yes" + "NTP service: active"

# Fix if needed (RHEL/Fedora):
dnf install -y chrony && systemctl enable --now chronyd

# Fix if needed (Ubuntu/Debian):
apt-get install -y chrony && systemctl enable --now chrony

# Verify:
chronyc tracking   # → "Leap status: Normal"
```

---

## 2.3 Disable Swap ✅ MUST HAVE

**Why?** kubelet refuses to start with active swap (by default). Swap messes up pod memory limits and scheduling decisions.

```bash
swapoff -a
sed -ri '/^[^#].*[[:space:]]swap[[:space:]]/ s/^/#/' /etc/fstab

# Verify:
swapon --show       # → empty = good
free -h | grep Swap # → Swap: 0B 0B 0B
```

> **Interview Q:** "Why disable swap?"  
> **A:** Kubernetes scheduler assumes pod memory limits are real physical limits. Swap breaks that guarantee — a pod could use more memory than its limit by swapping, causing unpredictable behavior for other pods.

---

## 2.4 Kernel Modules ✅ MUST HAVE

```bash
modprobe overlay && modprobe br_netfilter

# Persist across reboots:
cat > /etc/modules-load.d/k8s.conf <<'EOF'
overlay
br_netfilter
EOF
```

| Module | Why |
|--------|-----|
| `overlay` | Needed by containerd for OverlayFS (container filesystem layers) |
| `br_netfilter` | Lets iptables see bridged traffic (pod-to-pod networking) |

---

## 2.5 Sysctl Parameters ✅ MUST HAVE

```bash
cat > /etc/sysctl.d/99-kubernetes.conf <<'EOF'
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system   # Apply immediately, no reboot needed
```

**What each does:**

| Parameter | Purpose |
|-----------|---------|
| `bridge-nf-call-iptables` | Bridge packets go through iptables rules (needed for kube-proxy) |
| `ip_forward` | Node can route traffic between pod networks |

---

## 2.6 cgroup v2 Check ✅ MUST HAVE

```bash
stat -fc %T /sys/fs/cgroup/
# Expected: cgroup2fs
```

**K8s 1.36+ rejects nodes with cgroup v1.** If output is `tmpfs`, upgrade your distro or enable cgroup v2 in kernel boot params.

```
┌────────────────────────────────────────┐
│        cgroup versions                 │
├────────────────────────────────────────┤
│                                        │
│  cgroup v1 (legacy):  tmpfs            │
│  cgroup v2 (modern):  cgroup2fs  ✅    │
│                                        │
│  K8s 1.36+ = cgroup v2 required       │
│  containerd SystemdCgroup = true       │
└────────────────────────────────────────┘
```

### 🔴 OpenShift Difference

OpenShift 4.x **always uses cgroup v2** on RHCOS (Red Hat CoreOS). You don't configure this manually — the OS is immutable and pre-configured.

---

## 2.7 SELinux, Firewall & NetworkManager ⚡ GOOD TO KNOW

```bash
# RHEL/Fedora — set SELinux to permissive:
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config

# Disable firewalld (RHEL/Fedora):
systemctl disable --now firewalld

# Disable ufw (Ubuntu/Debian):
ufw disable
```

**Why permissive?** kubeadm baseline recommendation. In production, run enforcing with proper policies.

### 🔴 OpenShift Difference

| Aspect | Kubernetes (kubeadm) | OpenShift |
|--------|---------------------|-----------|
| SELinux | Permissive (lab) | **Enforcing always** — OpenShift ships proper SELinux policies |
| Firewall | Disabled for lab | Managed by OpenShift's machine-config-operator |
| Host OS | Any Linux (Ubuntu, RHEL, etc.) | **RHCOS only** for control plane; RHEL for workers |

---

## 2.8 Verify Node IP & Unique UUID ⚡ GOOD TO KNOW

```bash
# Confirm traffic uses the correct interface:
ip route get 192.168.56.109
# → src 192.168.56.108 (control-plane's host-only IP)

# Check unique machine UUID (cloned VMs can share this!):
cat /sys/class/dmi/id/product_uuid
```

> **Gotcha:** Cloned VMs share the same UUID → kubeadm treats them as the same node. Fix: regenerate UUID or use different VM templates.

---

---

# 📦 SECTION 3: Container Runtime — containerd

---

## 3.1 Why containerd? ✅ MUST HAVE

```
┌──────────────────────────────────────────────┐
│     Container Runtime Evolution              │
├──────────────────────────────────────────────┤
│                                              │
│  Docker (dockershim)                         │
│       │                                      │
│       ▼  K8s 1.24: dockershim removed!       │
│                                              │
│  containerd ◄── Now the standard runtime     │
│       │                                      │
│       ▼                                      │
│  kubelet ←→ CRI (Container Runtime           │
│              Interface) ←→ containerd        │
└──────────────────────────────────────────────┘
```

| Key Point | Detail |
|-----------|--------|
| What is it? | Lightweight container runtime (extracted from Docker) |
| Why not Docker? | Docker was removed from K8s in v1.24. containerd is what Docker used underneath anyway. |
| CRI version | Must expose CRI v1 |
| cgroup driver | **Must use `SystemdCgroup = true`** to match kubelet's systemd cgroup driver |

### Key Config — `/etc/containerd/config.toml`

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

**Why SystemdCgroup = true?** kubelet and containerd must use the same cgroup driver. Mismatch = pods randomly crash.

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| Runtime | containerd (you install & configure) | **CRI-O** (pre-installed on RHCOS) |
| Config | Manual `/etc/containerd/config.toml` | Managed by Machine Config Operator |
| Docker | Not supported since K8s 1.24 | Never supported in OCP 4.x |

> **Interview tip:** "OpenShift uses CRI-O, not containerd. CRI-O is purpose-built for Kubernetes — lighter than containerd because it doesn't support non-K8s workloads."

---

---

# ⚙️ SECTION 4: Install Kubernetes Packages

---

## 4.1 kubeadm, kubelet, kubectl ✅ MUST HAVE

| Tool | Purpose |
|------|---------|
| `kubeadm` | Bootstrap the cluster (init, join, upgrade) |
| `kubelet` | Node agent — runs pods via container runtime |
| `kubectl` | CLI to talk to the API server |

### Install from pkgs.k8s.io (v1.36.x)

```bash
# Add K8s repo (example for apt-based):
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | \
  gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' > \
  /etc/apt/sources.list.d/kubernetes.list

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl   # Prevent auto-upgrade
```

**Why hold/pin?** Uncontrolled upgrades can break the cluster. Always upgrade deliberately.

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| Install method | Manual package install | `openshift-install` handles everything |
| Version control | `apt-mark hold` / `yum versionlock` | Cluster Version Operator (CVO) manages upgrades |
| kubectl | Install separately | Ships as `oc` (superset of kubectl) |

---

---

# 🎯 SECTION 5: Initialize Control Plane — kubeadm init

---

## 5.1 The Init Command ✅ MUST HAVE

Run **only on the control-plane node**:

```bash
kubeadm init \
  --apiserver-advertise-address=192.168.56.108 \
  --pod-network-cidr=192.168.0.0/16
```

| Flag | Purpose |
|------|---------|
| `--apiserver-advertise-address` | IP that API server listens on (use stable/host-only IP) |
| `--pod-network-cidr` | CIDR for pod IPs — must match your CNI config (Calico wants 192.168.0.0/16) |

### What kubeadm init does behind the scenes:

```
┌──────────────────────────────────────────────────────────┐
│              kubeadm init — Step by Step                  │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. Pre-flight checks (swap, ports, cgroup, runtime)     │
│  2. Generates CA certificates (/etc/kubernetes/pki/)     │
│  3. Creates kubeconfig files                             │
│  4. Starts static pods: etcd, api-server, controller,    │
│     scheduler (manifests in /etc/kubernetes/manifests/)  │
│  5. Applies RBAC, CoreDNS, kube-proxy as addons          │
│  6. Prints the JOIN command for workers                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### After init — Set up kubectl:

```bash
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

### The Join Token (save this!)

kubeadm prints something like:

```
kubeadm join 192.168.56.108:6443 --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:abc123...
```

> **Token expires in 24h.** Regenerate with: `kubeadm token create --print-join-command`

### 🔴 OpenShift Difference

| Aspect | Kubernetes (kubeadm) | OpenShift |
|--------|---------------------|-----------|
| Bootstrap | `kubeadm init` | `openshift-install create cluster` (or bootstrap node for UPI) |
| Certificates | Generated once by kubeadm | Auto-rotated by cert-manager operator |
| Static pods | Yes (in /etc/kubernetes/manifests/) | Yes on control plane, but managed by operators |
| Join mechanism | Token-based `kubeadm join` | Machine API + ignition configs |

---

---

# 🌐 SECTION 6: Install CNI — Calico

---

## 6.1 Why CNI? Why Calico? ✅ MUST HAVE

**Without a CNI plugin, nodes stay `NotReady`!** Pods can't get IPs and can't communicate.

```
┌────────────────────────────────────────────────────────────┐
│                  CNI — Container Network Interface          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  kubelet creates pod → asks CNI plugin: "Give this pod    │
│  an IP and connect it to the network"                     │
│                                                            │
│  Without CNI:                                              │
│    - Nodes = NotReady                                      │
│    - CoreDNS = Pending                                     │
│    - No pod-to-pod traffic                                 │
│                                                            │
│  Popular CNI plugins:                                      │
│    ┌──────────┐  ┌───────┐  ┌────────┐  ┌───────┐        │
│    │ Calico   │  │Cilium │  │Flannel │  │Weave │        │
│    │(feature  │  │(eBPF  │  │(simple)│  │      │        │
│    │ rich)    │  │based) │  │        │  │      │        │
│    └──────────┘  └───────┘  └────────┘  └───────┘        │
└────────────────────────────────────────────────────────────┘
```

### Install Calico (Operator method — v3.32.x)

```bash
# Install Tigera operator:
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.32.1/manifests/tigera-operator.yaml

# Install custom resources (defines pod CIDR):
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.32.1/manifests/custom-resources.yaml
```

The `custom-resources.yaml` sets:
- **Pod CIDR:** 192.168.0.0/16 (must match `--pod-network-cidr` from init!)
- **Encapsulation:** VXLANCrossSubnet

### Calico Encapsulation Modes

| Mode | How It Works | When to Use |
|------|-------------|-------------|
| **VXLANCrossSubnet** | VXLAN only when crossing subnets | Default — good for most labs |
| **VXLAN (always)** | All pod traffic wrapped in VXLAN | Multi-subnet, cloud VPCs |
| **IPIP** | IP-in-IP tunneling | Legacy, lighter than VXLAN |
| **None (BGP)** | Direct routing via BGP peers | Production — best performance |

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| Default CNI | You choose (Calico, Cilium, Flannel, etc.) | **OVN-Kubernetes** (default since OCP 4.12) |
| Install | Manual `kubectl create -f` | Pre-installed during cluster creation |
| Network Policy | Depends on CNI choice | Always available (OVN-K supports it natively) |
| Alternative | Calico, Cilium, etc. | Can use Calico via operator, but OVN-K is default |

> **Interview tip:** "In OpenShift, you don't install a CNI manually. OVN-Kubernetes is baked in. In vanilla K8s, the cluster is broken until you install one."

---

---

# 🤝 SECTION 7: Join Worker Nodes

---

## 7.1 kubeadm join ✅ MUST HAVE

Run **on each worker node** (not on control-plane):

```bash
kubeadm join 192.168.56.108:6443 \
  --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:abc123...
```

### What join does:

```
┌─────────────────────────────────────────────────────┐
│            kubeadm join — Worker Flow                │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. Contacts API server at 192.168.56.108:6443      │
│  2. Validates token + CA cert hash (TLS bootstrap)  │
│  3. Downloads cluster CA certificate                │
│  4. kubelet gets a signed certificate               │
│  5. kubelet registers node with API server          │
│  6. Node shows up in `kubectl get nodes`            │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Token expired? Regenerate:

```bash
# On control-plane:
kubeadm token create --print-join-command
```

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| Adding workers | `kubeadm join` with token | Machine API creates VMs automatically (IPI) |
| Manual add | Token-based | Generate ignition config → boot node with it (UPI) |
| Scaling | Manual per node | `oc scale machineset` — declarative! |
| Auto-approval | Manual `kubectl certificate approve` (if needed) | Machine approver auto-approves CSRs |

---

---

# ✅ SECTION 8: Verify the Cluster

---

## 8.1 Health Checks ✅ MUST HAVE

```bash
# All nodes should be Ready:
kubectl get nodes
# NAME       STATUS   ROLES           AGE   VERSION
# k8s-cp     Ready    control-plane   10m   v1.36.x
# worker01   Ready    <none>          5m    v1.36.x
# worker02   Ready    <none>          5m    v1.36.x

# All system pods running:
kubectl get pods -n kube-system
# coredns-xxx        Running
# calico-node-xxx    Running
# kube-proxy-xxx     Running
# etcd-k8s-cp       Running
# kube-apiserver     Running
# kube-controller    Running
# kube-scheduler     Running

# Test DNS resolution:
kubectl run test --image=busybox --rm -it --restart=Never -- nslookup kubernetes
# → Should resolve to 10.96.0.1 (Service CIDR's first IP)

# Deploy a test app:
kubectl create deployment nginx --image=nginx --replicas=2
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc nginx
# → Access via http://<worker-ip>:<nodeport>
```

### Troubleshooting Checklist

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Node `NotReady` | CNI not installed | Install Calico |
| CoreDNS `Pending` | No CNI / pod CIDR mismatch | Check `--pod-network-cidr` matches Calico |
| kubelet won't start | Swap on / cgroup mismatch | `swapoff -a` / fix SystemdCgroup |
| Token rejected | Token expired (24h) | `kubeadm token create --print-join-command` |
| Wrong node IP | Multiple default routes | Use `--apiserver-advertise-address` / set kubelet `--node-ip` |

---

---

# 📋 SECTION 9: Quick Reference — Full Setup Flow

---

```
┌────────────────────────────────────────────────────────────────┐
│           COMPLETE kubeadm CLUSTER SETUP — CHEAT SHEET         │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─── ALL NODES ──────────────────────────────────────────┐   │
│  │  1. Set hostname (hostnamectl)                          │   │
│  │  2. Configure /etc/hosts                                │   │
│  │  3. Sync time (chrony)                                  │   │
│  │  4. Disable swap (swapoff -a)                           │   │
│  │  5. Load modules (overlay, br_netfilter)                │   │
│  │  6. Set sysctl (ip_forward, bridge-nf-call)             │   │
│  │  7. Verify cgroup v2                                    │   │
│  │  8. Install containerd (SystemdCgroup = true)           │   │
│  │  9. Install kubeadm, kubelet, kubectl                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          │                                      │
│            ┌─────────────┴──────────────┐                      │
│            ▼                            ▼                       │
│  ┌── CONTROL PLANE ONLY ──┐   ┌── WORKERS ONLY ────────┐      │
│  │  10. kubeadm init       │   │  12. kubeadm join      │      │
│  │  11. Install Calico CNI │   │      (use token from   │      │
│  │  (nodes go Ready)       │   │       init output)     │      │
│  └─────────────────────────┘   └────────────────────────┘      │
│                          │                                      │
│                          ▼                                      │
│  ┌── VERIFY ──────────────────────────────────────────────┐    │
│  │  13. kubectl get nodes → all Ready                      │    │
│  │  14. kubectl get pods -n kube-system → all Running      │    │
│  │  15. Test DNS + deploy sample app                       │    │
│  └─────────────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────────┘
```

---

---

# 🆚 SECTION 10: Kubernetes vs OpenShift — Master Comparison

---

| Aspect | Kubernetes (kubeadm) | OpenShift (OCP 4.x) |
|--------|---------------------|---------------------|
| **Install** | Manual (kubeadm, packages, CNI) | `openshift-install` (automated) |
| **Host OS** | Any Linux | RHCOS (control plane), RHEL (workers) |
| **Container Runtime** | containerd | CRI-O |
| **CNI** | You choose (Calico, Cilium, etc.) | OVN-Kubernetes (default) |
| **Ingress** | You install (NGINX, Traefik, etc.) | HAProxy Router (built-in Routes) |
| **Registry** | You deploy (Harbor, etc.) | Built-in integrated registry |
| **Auth** | You configure (OIDC, certs, etc.) | OAuth server built-in |
| **Upgrades** | `kubeadm upgrade` (manual) | CVO + OLM (operator-managed) |
| **Monitoring** | You install (Prometheus stack) | Built-in (Prometheus + Grafana) |
| **Security** | PSA (Pod Security Admission) | SCC (Security Context Constraints) — stricter |
| **CLI** | `kubectl` | `oc` (superset of kubectl) |
| **Nodes** | Add manually | Machine API (declarative scaling) |
| **Cost** | Free (open-source) | Paid subscription (Red Hat support) |

### When to use which?

```
Choose kubeadm/vanilla K8s when:
  → You want full control
  → Budget-conscious / learning
  → Cloud-managed K8s (EKS, AKS, GKE) handles the rest

Choose OpenShift when:
  → Enterprise needs (compliance, support SLAs)
  → Developer self-service portal needed
  → "Batteries included" — monitoring, logging, CI/CD, registry
  → Red Hat ecosystem (RHEL, Ansible, etc.)
```

---

## 💡 Interview Power Answers

**Q: "Walk me through setting up a K8s cluster from scratch."**
> "I use kubeadm. First, prepare all nodes — hostname, time sync, disable swap, load kernel modules, set sysctl for bridge traffic and IP forwarding. Install containerd with SystemdCgroup=true. Install kubeadm/kubelet/kubectl. On the control plane, run kubeadm init with the API advertise address and pod CIDR. Then install Calico as the CNI. Finally, run kubeadm join on workers with the bootstrap token. Verify with kubectl get nodes — all should be Ready."

**Q: "Why do nodes show NotReady?"**
> "No CNI installed. Without a CNI plugin, pods can't get IPs, CoreDNS stays Pending, and kubelet reports NotReady. Installing Calico or another CNI fixes it within seconds."

**Q: "What's the difference between containerd and CRI-O?"**
> "Both are CRI-compliant runtimes. containerd came from Docker — it's general-purpose and supports non-K8s workloads too. CRI-O is built specifically for Kubernetes — lighter, fewer features outside K8s. OpenShift uses CRI-O; vanilla K8s typically uses containerd."

**Q: "How does OpenShift differ from vanilla Kubernetes?"**
> "OpenShift IS Kubernetes with opinionated additions. It adds: CRI-O runtime, OVN-K networking, OAuth, Routes (instead of just Ingress), built-in registry, Prometheus monitoring, and manages everything through Operators. It's like K8s + all the CNCF add-ons pre-integrated."

---

> 📝 **Note:** This is Part 1 — Cluster Setup. More sections will be added as you share more K8s content (Deployments, Services, Storage, RBAC, Helm, etc.)


---

---

# 🖥️ SECTION 11: kubectl — The Kubernetes CLI Client

---

## 11.1 What Is kubectl? ✅ MUST HAVE

kubectl = CLI that talks to the **API Server** over HTTPS. That's it.

```
┌────────────────────────────────────────────────────┐
│            kubectl Communication Flow              │
├────────────────────────────────────────────────────┤
│                                                    │
│  You (human)                                       │
│    │                                               │
│    ▼                                               │
│  kubectl ──── HTTPS :6443 ────► API Server         │
│    │                              │                │
│    │  (reads kubeconfig           ▼                │
│    │   for auth + endpoint)     etcd / kubelet     │
│    │                                               │
│  kubectl does NOT talk to:                         │
│    ✗ kubelet directly                              │
│    ✗ etcd directly                                 │
│    ✗ container runtime                             │
└────────────────────────────────────────────────────┘
```

| Key Fact | Detail |
|----------|--------|
| What it does | Sends REST requests to API server |
| What it doesn't do | Install clusters, talk to kubelet/etcd directly |
| Where to run it | Control-plane, worker, laptop, jump host — anywhere with network to :6443 |

---

## 11.2 Installing kubectl ⚡ GOOD TO KNOW

Three methods — pick one:

| Method | When to Use | Distro |
|--------|------------|--------|
| **Official binary** | Default choice — works everywhere | Any Linux |
| **dnf repo** | RPM-based systems | RHEL, Rocky, Fedora |
| **apt repo** | DEB-based systems | Ubuntu, Debian |

### Method A: Official Binary (Universal)

```bash
KUBECTL_VERSION=v1.36.3

# Download:
curl -fsSLO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl"

# Verify checksum:
curl -fsSLO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
# → kubectl: OK

# Install system-wide:
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# OR install for your user only:
mkdir -p "$HOME/.local/bin"
install -m 0755 kubectl "$HOME/.local/bin/kubectl"
export PATH="$HOME/.local/bin:$PATH"  # Add to ~/.bashrc
```

### Method B: dnf (RHEL/Rocky/Fedora)

```bash
cat <<'EOF' | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.36/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.36/rpm/repodata/repomd.xml.key
EOF

sudo dnf install -y kubectl
```

### Method C: apt (Ubuntu/Debian)

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update && sudo apt-get install -y kubectl
```

### Verify Install:

```bash
kubectl version --client
# Client Version: v1.36.3
```

---

## 11.3 Version Skew Policy ✅ MUST HAVE

**Rule:** kubectl must be within **±1 minor version** of the API server.

```
┌──────────────────────────────────────────┐
│       Version Skew — What Works          │
├──────────────────────────────────────────┤
│                                          │
│  API Server: v1.36                       │
│                                          │
│  ✅ kubectl v1.35 — works                │
│  ✅ kubectl v1.36 — works (best match)   │
│  ✅ kubectl v1.37 — works                │
│  ❌ kubectl v1.34 — too old              │
│  ❌ kubectl v1.38 — too new              │
│                                          │
│  Patch versions don't matter (v1.36.1    │
│  vs v1.36.3 = fine)                      │
└──────────────────────────────────────────┘
```

> **Interview tip:** "Always match the minor version to your cluster. Don't just grab the newest kubectl binary."

---

## 11.4 kubeconfig — How kubectl Knows Where to Connect ✅ MUST HAVE

### What's Inside a kubeconfig?

```yaml
# ~/.kube/config structure:
clusters:       # API server URL + CA cert
users:          # Client cert / token / exec plugin
contexts:       # cluster + user + (optional) namespace
current-context: # which context is active
```

```
┌─────────────────────────────────────────────────────┐
│              kubeconfig Structure                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Context = Cluster + User + Namespace               │
│                                                     │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐   │
│  │ Cluster  │     │  User    │     │Namespace │   │
│  │ (server  │  +  │ (creds)  │  +  │(optional)│   │
│  │  + CA)   │     │          │     │          │   │
│  └──────────┘     └──────────┘     └──────────┘   │
│        └──────────────┼──────────────┘             │
│                       ▼                             │
│               ┌──────────────┐                     │
│               │   Context    │                     │
│               │  "lab-admin" │                     │
│               └──────────────┘                     │
└─────────────────────────────────────────────────────┘
```

### Precedence (highest → lowest):

| Priority | Source | Scope |
|----------|--------|-------|
| 1 (highest) | `--kubeconfig <path>` | Single command |
| 2 | `KUBECONFIG` env var | Shell session |
| 3 (default) | `$HOME/.kube/config` | Always (fallback) |

---

## 11.5 kubeconfig Methods — Quick Reference ✅ MUST HAVE

### Method 1: Copy admin.conf (Default Setup)

```bash
# On control-plane:
mkdir -p "$HOME/.kube"
sudo install -o "$USER" -g "$(id -gn)" -m 0600 \
  /etc/kubernetes/admin.conf "$HOME/.kube/config"

# From remote workstation:
scp root@192.168.56.108:/etc/kubernetes/admin.conf "$HOME/admin.conf"
sudo install -o "$USER" -g "$(id -gn)" -m 0600 \
  "$HOME/admin.conf" "$HOME/.kube/config"
```

### Method 2: Inline for One Command

```bash
KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes
```

### Method 3: Flag for One Command

```bash
kubectl --kubeconfig "$HOME/lab.conf" get nodes
```

### Method 4: Export for Shell Session

```bash
export KUBECONFIG="$HOME/lab.conf"
# All kubectl commands in this terminal use lab.conf
```

### Method 5: Multiple Clusters (Colon-Separated)

```bash
export KUBECONFIG="$HOME/.kube/config:$HOME/.kube/lab.conf"

# Merge into one file:
( umask 077; KUBECONFIG="$HOME/.kube/config:$HOME/.kube/lab.conf" \
  kubectl config view --raw --flatten > "$HOME/.kube/merged-config" )
```

### Method 6: Switch Context

```bash
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=kube-system
kubectl --context <name> get nodes   # one-off override
```

### Useful Commands:

```bash
kubectl config get-contexts          # list all contexts
kubectl config current-context       # which context is active
kubectl config view                  # show kubeconfig (creds masked)
```

---

## 11.6 Verify Cluster Access ⚡ GOOD TO KNOW

```bash
kubectl version
# Client Version: v1.36.3
# Server Version: v1.36.3   ← confirms API server is reachable

kubectl get nodes
# NAME       STATUS   ROLES           AGE   VERSION
# k8s-cp     Ready    control-plane   21h   v1.36.3
# worker01   Ready    <none>          21h   v1.36.3
```

---

## 11.7 Troubleshooting kubectl ✅ MUST HAVE

| Symptom | Cause | Fix |
|---------|-------|-----|
| `command not found` | Not on PATH | Check install location, `hash -r`, new shell |
| `localhost:8080 refused` | No kubeconfig loaded | Create `~/.kube/config` or set `KUBECONFIG` |
| `connection refused :6443` | API server down / firewall | Check server health, network, port 6443 |
| `no current-context` | Kubeconfig has no active context | `kubectl config use-context <name>` |
| `x509: certificate signed by unknown authority` | CA mismatch | Get correct kubeconfig from admin |
| `Unauthorized` | Expired creds | Refresh certs/tokens |
| Works without sudo, fails with sudo | sudo reads `/root/.kube/config` | Run as the user who owns kubeconfig |
| Version skew warning | Client too old/new | Install within ±1 minor of server |
| Wrong cluster/namespace | Wrong context active | `kubectl config current-context` → switch |

---

## 11.8 Security — kubeconfig Best Practices ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────────┐
│         kubeconfig Security Rules                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. chmod 600 ~/.kube/config (only you can read)        │
│  2. admin.conf = cluster-admin → lab only!              │
│  3. Production → create RBAC-scoped credentials         │
│  4. Never accept kubeconfig from untrusted sources      │
│     (can run arbitrary exec plugins or point to         │
│      malicious API servers)                             │
│  5. --raw --flatten embeds secrets → treat as secret    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 🔴 OpenShift Difference

| Aspect | Kubernetes (kubectl) | OpenShift (oc) |
|--------|---------------------|----------------|
| CLI tool | `kubectl` | `oc` (superset of kubectl — all kubectl commands work) |
| Login | Copy kubeconfig manually | `oc login https://api.cluster:6443 -u user -p pass` |
| Auth | Certs / tokens in kubeconfig | OAuth — `oc login` gets a token automatically |
| Context switch | `kubectl config use-context` | `oc login` to different cluster (simpler UX) |
| kubeconfig location | `~/.kube/config` | Same — `~/.kube/config` |
| Web console | None built-in | Full web UI with terminal |
| Extra commands | None | `oc new-app`, `oc new-project`, `oc adm`, `oc debug` |

> **Interview tip:** "`oc` is kubectl + extras. Any kubectl command works with oc. But oc adds developer-friendly commands (new-app, debug, login) and ties into OpenShift's OAuth. In vanilla K8s, you manage kubeconfig files manually; in OpenShift, `oc login` handles auth for you."

---

## 💡 Interview Power Answers — kubectl

**Q: "What is kubectl and how does it connect to a cluster?"**
> "kubectl is the CLI client for the Kubernetes API server. It reads a kubeconfig file (~/.kube/config by default) which contains the API server URL, CA cert, and user credentials. It sends HTTPS requests to port 6443. It never talks to kubelet or etcd directly."

**Q: "How do you manage multiple clusters?"**
> "I use kubeconfig contexts. Each context maps a cluster + user + namespace. I can merge multiple kubeconfig files using the KUBECONFIG environment variable with colon-separated paths, then switch with `kubectl config use-context`. For quick one-offs, I use `--kubeconfig` or `--context` flags."

**Q: "What's the version skew policy?"**
> "kubectl must be within plus or minus one minor version of the API server. So against a v1.36 server, I can use kubectl v1.35 through v1.37. Patch versions don't need to match."

**Q: "Why shouldn't you use admin.conf in production?"**
> "admin.conf has cluster-admin privileges — full access to everything. In production, you create RBAC-bound service accounts or user certs with limited permissions. admin.conf is fine for labs and initial bootstrap only."


---

---

# 🧠 SECTION 12: Kubernetes Architecture — Deep Dive

---

## 12.1 The Control Loop — How K8s Works ✅ MUST HAVE

Everything in Kubernetes follows one pattern:

```
┌─────────────────────────────────────────────────────┐
│         THE KUBERNETES CONTROL LOOP                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. You DECLARE desired state (YAML → API)          │
│  2. Controllers RECONCILE (make reality match)      │
│  3. Scheduler PLACES Pods on nodes                  │
│  4. kubelet STARTS containers via CRI runtime       │
│                                                     │
│  ┌────────┐    ┌────────────┐    ┌──────────┐     │
│  │ You    │───►│ API Server │───►│  etcd    │     │
│  │(YAML)  │    │            │    │ (state)  │     │
│  └────────┘    └─────┬──────┘    └──────────┘     │
│                      │                              │
│         ┌────────────┼────────────┐                 │
│         ▼            ▼            ▼                 │
│  ┌───────────┐ ┌──────────┐ ┌─────────┐           │
│  │Controller │ │Scheduler │ │kubelet  │           │
│  │Manager    │ │          │ │(on node)│           │
│  └───────────┘ └──────────┘ └─────────┘           │
│       │              │            │                 │
│  "3 replicas     "Put Pod     "Start              │
│   needed,         on node      container           │
│   only 2 exist"   worker01"    via CRI"            │
└─────────────────────────────────────────────────────┘
```

> **Interview one-liner:** "Kubernetes is a declarative system — you tell it WHAT you want, controllers figure out HOW to get there."

---

## 12.2 Components vs Objects vs Workloads ✅ MUST HAVE

Don't confuse these three things:

| Category | Examples | What It Is |
|----------|----------|------------|
| **Cluster Components** | kube-apiserver, scheduler, kubelet | Running processes/binaries |
| **API Objects** | Pod, Deployment, Service, Node | Data stored in etcd |
| **Runtime Workloads** | Containers inside Pods | Actual running processes |
| **Add-ons** | CoreDNS, Metrics Server, CNI | Extra capabilities (not core) |

Key distinctions:
- A **Pod** = API object + smallest deployable unit
- A **Node** = API object + real machine
- A **Container** = created by runtime, not by API server
- A **Deployment** = desired state in etcd; controllers make it real
- **Controllers** watch objects and write new objects — they don't run your app code

---

## 12.3 Control Plane Components ✅ MUST HAVE

### kube-apiserver — The Front Door

```
┌──────────────────────────────────────────────────────────┐
│          API Server Request Flow (Write Path)            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Client (kubectl / controller / kubelet)                 │
│    │                                                     │
│    ▼                                                     │
│  TLS Termination                                         │
│    │                                                     │
│    ▼                                                     │
│  Authentication ("Who are you?")                         │
│    │                                                     │
│    ▼                                                     │
│  Authorization ("Can you do this?" — RBAC)               │
│    │                                                     │
│    ▼                                                     │
│  Mutating Admission ("Let me modify your request")       │
│    │                                                     │
│    ▼                                                     │
│  Object Validation ("Is this YAML valid?")               │
│    │                                                     │
│    ▼                                                     │
│  Validating Admission ("Final policy check")             │
│    │                                                     │
│    ▼                                                     │
│  Persist to etcd ✅                                      │
└──────────────────────────────────────────────────────────┘
```

**Order matters:** Mutating → Validation → Validating admission

| Fact | Detail |
|------|--------|
| Everyone talks to it | kubectl, controllers, kubelets — all use API server |
| It doesn't orchestrate | Doesn't tell scheduler or controllers what to do — they watch independently |
| Only writer to etcd | Under normal operation, only API server writes to etcd |

### etcd — The Brain's Memory

| Fact | Detail |
|------|--------|
| What | Consistent key-value store |
| Stores | All cluster state (Pods, Deployments, Secrets, etc.) |
| Where | Runs on control-plane nodes (static Pod in kubeadm) |
| NOT on | Worker nodes |
| NOT a | Messaging bus between nodes |

### kube-scheduler — The Placement Engine

```
Pod created (no nodeName) → Scheduler picks up
    │
    ▼
Filter: Which nodes CAN run this Pod?
  (resources, selectors, taints, affinity, topology)
    │
    ▼
Score: Which node is BEST?
  (spreading, resource balance)
    │
    ▼
Bind: Set spec.nodeName → kubelet takes over
```

**The scheduler selects a node. It does NOT start containers.**

### kube-controller-manager — The Reconcilers

One binary, many controllers inside:

| Controller | What It Reconciles |
|------------|-------------------|
| Deployment | Creates/updates ReplicaSets |
| ReplicaSet | Creates/deletes Pods to match replica count |
| Job | Creates Pods for batch tasks |
| Node | Marks unhealthy nodes, triggers eviction |
| EndpointSlice | Updates Service backends based on Pod readiness |
| ServiceAccount | Creates default SA in new namespaces |

```
Desired state: "I want 3 replicas"
Current state: "Only 2 Pods exist"
Action: Create 1 more Pod
... repeat forever
```

### cloud-controller-manager (Optional)

| When Present | When Absent |
|--------------|-------------|
| Cloud clusters (EKS, AKS, GKE) | Local kubeadm labs |
| Manages: node lifecycle, routes, LBs | Not needed |

---

## 12.4 Worker Node Components ✅ MUST HAVE

### kubelet — The Node Agent

| Responsibility | Detail |
|----------------|--------|
| Registers node | With API server |
| Watches Pods | Only Pods assigned to THIS node |
| Creates containers | Via CRI → container runtime |
| Mounts volumes | ConfigMaps, Secrets, PVs |
| Runs probes | startup, readiness, liveness |
| Reports status | Pod + Node status to API |

**kubelet does NOT choose which node gets a Pod. Scheduler does that.**

### Container Runtime + CRI

```
kubelet ──► CRI (gRPC) ──► Container Runtime ──► Container
                                │
                          ┌─────┴─────┐
                          │containerd │  (standard)
                          │CRI-O     │  (OpenShift)
                          └───────────┘
```

- Images built with Docker work on containerd/CRI-O (standard OCI format)
- Docker Engine doesn't implement CRI directly (needs cri-dockerd adapter)

### kube-proxy — Service Forwarding

| Does | Doesn't |
|------|---------|
| Programs iptables/IPVS rules for Services | Create Pod network (that's CNI) |
| Watches Service + EndpointSlice objects | Do DNS lookups (that's CoreDNS) |
| Enables ClusterIP/NodePort/LoadBalancer | Route Pod-to-Pod traffic |

Some CNIs (Cilium in kube-proxy replacement mode) handle Service forwarding themselves → kube-proxy not needed.

---

## 12.5 Add-ons (Not Core, But Essential) ⚡ GOOD TO KNOW

| Add-on | Purpose |
|--------|---------|
| **CNI** (Calico/Cilium/Flannel) | Pod IP assignment + Pod-to-Pod networking |
| **CoreDNS** | DNS for Services (`my-svc.default.svc.cluster.local`) |
| **Metrics Server** | CPU/memory metrics for `kubectl top` |
| **Ingress/Gateway Controller** | External HTTP routing into cluster |

---

## 12.6 Deployment → Running Pod (Full Request Flow) ✅ MUST HAVE

```bash
kubectl apply -f deployment.yaml
```

```
┌─────────────────────────────────────────────────────────────────┐
│      FROM kubectl apply TO RUNNING CONTAINERS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. kubectl → sends YAML to API server                           │
│  2. API server → auth → admission → validate → save to etcd     │
│  3. Deployment controller → sees new Deployment → creates        │
│     ReplicaSet                                                   │
│  4. ReplicaSet controller → sees RS needs Pods → creates         │
│     Pod objects                                                   │
│  5. Scheduler → sees Pods with no nodeName → assigns node        │
│  6. kubelet (on assigned node) → sees Pod for its node           │
│  7. kubelet → prepares volumes                                   │
│  8. kubelet → CRI → runtime creates Pod sandbox                  │
│  9. CNI → assigns Pod IP + configures networking                 │
│  10. Runtime → pulls images → starts init containers             │
│  11. Runtime → starts app containers                             │
│  12. kubelet → runs probes → reports status to API               │
│  13. EndpointSlice controller → marks Pod as ready backend       │
│      for its Service                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

> **Interview tip:** Walk through this flow step-by-step. Interviewers love this question: "What happens when you run kubectl apply?"

---

## 12.7 Self-Healing — What It Really Means ✅ MUST HAVE

**Self-healing ≠ Pod moves to another node.**  
**Self-healing = controllers create a REPLACEMENT Pod (new UID).**

| Failure | Who Acts | Result |
|---------|----------|--------|
| Container crashes | kubelet | Restarts container (same Pod, same node) |
| Liveness probe fails | kubelet | Restarts container (same Pod, same node) |
| Pod deleted (under Deployment) | ReplicaSet controller | NEW Pod created (may go to different node) |
| Node dies | Node controller + workload controllers | Node tainted → Pods evicted → NEW Pods on other nodes |
| App not ready | kubelet + EndpointSlice controller | Pod removed from Service endpoints |
| Image changed in Deployment | Deployment controller | Rolling update → new ReplicaSet → new Pods |

**Key facts:**
- A Pod stays on ONE node for its entire lifetime
- Standalone Pods (no controller) are **NOT recreated** if node fails
- Replacement is not instant — there are eviction delays and tolerations

---

## 12.8 Control Plane Availability ⚡ GOOD TO KNOW

| Layout | Use Case | Risk |
|--------|----------|------|
| **Single CP** | Labs, dev, CKAD practice | CP dies → no API changes (existing Pods may keep running) |
| **HA (3+ CP)** | Production | LB in front of API servers, etcd replicated, leader election for scheduler/CM |
| **Managed** (EKS/AKS/GKE) | Production (cloud) | Provider manages CP — you manage workers + objects |

```
┌── Single Control Plane (Lab) ──┐
│  1 API server                   │
│  1 etcd                         │
│  Single point of failure        │
└─────────────────────────────────┘

┌── HA Control Plane (Prod) ─────────────────┐
│  Load Balancer                              │
│    ├── API Server 1 + etcd member           │
│    ├── API Server 2 + etcd member           │
│    └── API Server 3 + etcd member           │
│  Scheduler: leader election (1 active)      │
│  Controller Mgr: leader election (1 active) │
└─────────────────────────────────────────────┘
```

---

## 12.9 Control Plane vs Worker — Quick Table ⚡ GOOD TO KNOW

| Area | Control Plane | Worker Node |
|------|--------------|-------------|
| Main job | Manage cluster state | Run app workloads |
| API server | ✅ | ❌ |
| etcd | ✅ (usually) | ❌ |
| Scheduler | ✅ | ❌ |
| Controller Manager | ✅ | ❌ |
| kubelet | May run | ✅ |
| Container runtime | If components run as containers | ✅ |
| App Pods | Usually restricted (taint) | ✅ |
| kube-proxy | Depends | ✅ (commonly) |
| CNI agent | Depends | ✅ (commonly) |

---

## 12.10 Common Misconceptions ✅ MUST HAVE

| ❌ Wrong | ✅ Correct |
|----------|-----------|
| Every cluster has one master | Can have 1 or many control-plane nodes |
| Every worker runs etcd | etcd only on control-plane |
| K8s requires Docker | K8s requires any CRI-compatible runtime |
| A Pod moves to another node | A NEW Pod (new UID) is created elsewhere |
| kube-proxy creates Pod network | CNI creates Pod network; kube-proxy handles Service forwarding |
| Every Pod has multiple containers | Most Pods have ONE container |
| Pod = Container | Pod = wrapper around 1+ containers (shared network/storage) |
| Scheduler starts containers | Scheduler picks node; kubelet + runtime start containers |
| Controller Manager runs apps | Controllers create/update API objects only |
| Control plane never runs workloads | It can, if scheduling is allowed (no taint or toleration present) |

---

## 12.11 Inspect Architecture in a Live Cluster ⚡ GOOD TO KNOW

```bash
# Nodes with details:
kubectl get nodes -o wide

# System components:
kubectl get pods -n kube-system -o wide

# API server health:
kubectl get --raw='/readyz?verbose'

# Node details (labels, taints, capacity, pods):
kubectl describe node worker01

# Node heartbeats:
kubectl get lease -n kube-node-lease
```

### 🔴 OpenShift Difference

| Aspect | Kubernetes (kubeadm) | OpenShift |
|--------|---------------------|-----------|
| CP visibility | `kubectl get pods -n kube-system` shows all | Some CP components hidden from tenant API |
| Static Pods | Control-plane components as static Pods | Managed by **operators** (not raw static Pods) |
| Node types | control-plane + worker | **master + worker + infra** (infra = dedicated for monitoring/logging/registry) |
| Infra nodes | Not a concept | Dedicated nodes for platform workloads (saves subscription costs) |
| Machine management | Manual | **Machine API** — MachineSet, Machine, MachineHealthCheck objects |
| Node OS | Any Linux | RHCOS (immutable, auto-updated via Machine Config Operator) |
| etcd management | Manual backup/restore | **etcd Operator** auto-manages (backup, defrag, scaling) |
| HA setup | Manual (haproxy + keepalived + multiple CP) | Built-in — `openshift-install` creates HA by default (IPI) |
| Health checks | `kubectl get --raw='/readyz'` | Same + **ClusterOperator** status (`oc get co`) |
| Cluster version | Manual tracking | **ClusterVersion** object — `oc get clusterversion` |

> **Interview tip:** "In OpenShift, you don't manually manage control-plane components — Operators do it. You check health with `oc get co` (ClusterOperators) rather than inspecting individual static Pods. OpenShift also adds infra nodes to separate platform workloads from app workloads."

---

## 💡 Interview Power Answers — Architecture

**Q: "Explain Kubernetes architecture."**
> "A K8s cluster has a control plane and worker nodes. The control plane runs the API server (front door), etcd (state store), scheduler (picks nodes for Pods), and controller manager (reconciles desired vs actual state). Workers run kubelet (node agent), a CRI-compatible runtime (containerd), and kube-proxy (Service forwarding). Everything talks through the API server — it's the single communication hub."

**Q: "What happens when you run kubectl apply?"**
> "kubectl sends the YAML to the API server. After auth and admission, it's persisted in etcd. The Deployment controller sees it and creates a ReplicaSet. The ReplicaSet controller creates Pod objects. The scheduler assigns each Pod to a node. The kubelet on that node creates the container via CRI, the CNI assigns a Pod IP, images are pulled, containers start, probes run, and the EndpointSlice controller marks the Pod as a ready Service backend."

**Q: "How does self-healing work?"**
> "Controllers continuously compare desired state with current state. If a Pod dies under a Deployment, the ReplicaSet controller creates a replacement Pod — it's a NEW Pod with a new UID, not the old one moving. If a node dies, the node controller taints it, and after eviction delays, workload controllers recreate affected Pods on healthy nodes. Standalone Pods without a controller are NOT recreated."

**Q: "What's the difference between kube-proxy and CNI?"**
> "CNI handles Pod networking — assigns IPs, creates interfaces, enables Pod-to-Pod communication. kube-proxy handles Service forwarding — programs iptables/IPVS rules so ClusterIP and NodePort work. They solve different problems."

**Q: "Can control-plane nodes run application Pods?"**
> "Yes, if there's no taint preventing it. kubeadm adds a `NoSchedule` taint to control-plane nodes by default. Remove it or add a toleration, and Pods can schedule there. In production, you keep them separate for stability."


---

---

# 🔍 SECTION 13: API Resources, Versions & kubectl explain

---

## 13.1 The Three Discovery Commands ✅ MUST HAVE

| Command | Answers | Example |
|---------|---------|---------|
| `kubectl api-resources` | What resource **types** exist? | `kubectl api-resources -o wide` |
| `kubectl api-versions` | What API **group/version** pairs are served? | `kubectl api-versions` |
| `kubectl explain` | What **fields** does a resource have? | `kubectl explain pod.spec.containers` |
| `kubectl get` | What **objects** exist right now? | `kubectl get pods` |

```
┌───────────────────────────────────────────────────────┐
│       Discovery vs Listing — Don't Confuse            │
├───────────────────────────────────────────────────────┤
│                                                       │
│  api-resources → "What TYPES does this cluster have?" │
│  api-versions  → "What API versions are served?"      │
│  explain       → "What FIELDS can I put in YAML?"     │
│  get           → "What OBJECTS exist right now?"       │
│                                                       │
│  Example:                                             │
│    api-resources → tells you "pods" is a valid type   │
│    get pods      → shows you nginx-abc123 is running  │
└───────────────────────────────────────────────────────┘
```

> **Key point:** These commands query your **live cluster**, not a static list in kubectl. Install a CRD → new rows appear instantly.

---

## 13.2 Resource vs Object vs Kind ✅ MUST HAVE

| Term | What It Is | Example |
|------|-----------|---------|
| **Resource** | API type + REST endpoint (lowercase, plural) | `pods`, `deployments` |
| **Object** | One instance of a resource | Pod named `nginx` in `default` ns |
| **Kind** | Capitalized type name in YAML | `Pod`, `Deployment` |
| **Short name** | Abbreviation for CLI speed | `po`, `deploy`, `svc`, `ns` |

```yaml
# In YAML → use Kind (capitalized)
kind: Deployment

# In kubectl → use resource name (lowercase, plural) or short name
kubectl get deployments
kubectl get deploy
```

---

## 13.3 kubectl api-resources Output ⚡ GOOD TO KNOW

```bash
kubectl api-resources
```

| Column | Meaning | Example |
|--------|---------|---------|
| NAME | Plural name for kubectl | `pods`, `deployments` |
| SHORTNAMES | Abbreviations | `po`, `deploy`, `svc` |
| APIVERSION | Group/version for YAML | `v1`, `apps/v1` |
| NAMESPACED | Scoped to namespace? | Pods = true, Nodes = false |
| KIND | YAML `kind:` value | `Pod`, `Deployment` |

### Useful Filters

```bash
# All resources with wide info (verbs, categories):
kubectl api-resources -o wide

# Only namespaced resources:
kubectl api-resources --namespaced=true

# Only cluster-scoped:
kubectl api-resources --namespaced=false

# Filter by API group:
kubectl api-resources --api-group=apps

# Filter by verb (what can I list?):
kubectl api-resources --verbs=list

# Compact names for scripting:
kubectl api-resources -o name

# Sort alphabetically:
kubectl api-resources --sort-by=name
```

### "kubectl get all" Gotcha

`kubectl get all` uses the `all` **category**, NOT a wildcard. It misses many resource types (Secrets, ConfigMaps, Ingress, etc.). To list everything:

```bash
kubectl api-resources --verbs=list --namespaced=true -o name | \
  xargs -n 1 kubectl get --show-kind --ignore-not-found -n default
```

---

## 13.4 API Groups & Versions ✅ MUST HAVE

### Core vs Named Groups

| Type | apiVersion in YAML | Examples |
|------|-------------------|----------|
| **Core** (no group prefix) | `v1` | Pod, Service, ConfigMap, Secret, Namespace |
| **Named group** | `group/version` | `apps/v1`, `batch/v1`, `networking.k8s.io/v1` |

```bash
kubectl api-versions
# Output (trimmed):
# apps/v1
# batch/v1
# networking.k8s.io/v1
# rbac.authorization.k8s.io/v1
# v1                          ← core resources
```

### Common apiVersion Cheat Sheet

| Resource | apiVersion |
|----------|-----------|
| Pod, Service, ConfigMap, Secret | `v1` |
| Deployment, DaemonSet, StatefulSet, ReplicaSet | `apps/v1` |
| Job, CronJob | `batch/v1` |
| Ingress | `networking.k8s.io/v1` |
| NetworkPolicy | `networking.k8s.io/v1` |
| Role, ClusterRole, RoleBinding | `rbac.authorization.k8s.io/v1` |
| PersistentVolume, PVC | `v1` |
| StorageClass | `storage.k8s.io/v1` |

> **Interview tip:** "Don't memorize apiVersions — use `kubectl api-resources` to look them up from the live cluster. But know that core resources are just `v1` and Deployments are `apps/v1`."

---

## 13.5 kubectl explain — Your YAML Writing Tool ✅ MUST HAVE

### Basic Usage

```bash
# Top-level fields:
kubectl explain pod
# Output:
# FIELDS:
#   apiVersion  <string>
#   kind        <string>
#   metadata    <ObjectMeta>
#   spec        <PodSpec>
#   status      <PodStatus>

# Drill into nested fields with dots:
kubectl explain pod.spec
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.resources
kubectl explain pod.spec.containers.ports
```

### Field Type Markers

| Output | Meaning |
|--------|---------|
| `<string>` | Single text value |
| `<integer>` | Number |
| `<[]Container>` | List of Container objects |
| `<map[string]string>` | Key-value string map (like labels) |
| `-required-` | You must include this field |

### Recursive View (One Extra Level)

```bash
kubectl explain pod --recursive
# Shows all fields expanded one level — good for overview
# For details, drill into specific paths
```

### Pin to Specific API Version

```bash
kubectl explain deployment --api-version=apps/v1
```

### Workflow: Build YAML from explain

```
1. kubectl api-resources | grep deployment
   → NAME: deployments, APIVERSION: apps/v1, KIND: Deployment

2. kubectl explain deployment.spec
   → Shows: replicas, selector, template, strategy...

3. kubectl explain deployment.spec.template.spec.containers
   → Shows: name, image, ports, resources, env...

4. Write your YAML using discovered fields

5. Validate without creating:
   kubectl apply --dry-run=server -f my-deploy.yaml
```

### Example: Build a Pod YAML from explain

```bash
# Discover:
kubectl api-resources | grep -w pods
# → pods  po  v1  true  Pod

kubectl explain pod.spec.containers
# → name (required), image, ports, resources, command, args...
```

```yaml
# Result:
apiVersion: v1
kind: Pod
metadata:
  name: explain-demo
  namespace: default
  labels:
    app: explain-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    ports:
    - containerPort: 80
```

```bash
# Validate:
kubectl apply --dry-run=server -f explain-demo.yaml
# → pod/explain-demo created (server dry run)
```

---

## 13.6 Custom Resources & CRDs ⚡ GOOD TO KNOW

When you install an Operator or CRD (like Calico), new resources appear in `api-resources`:

```bash
kubectl api-resources --api-group=crd.projectcalico.org
# NAME                    APIVERSION                 KIND
# globalnetworkpolicies   crd.projectcalico.org/v1   GlobalNetworkPolicy
# ippools                 crd.projectcalico.org/v1   IPPool
# networkpolicies         crd.projectcalico.org/v1   NetworkPolicy
```

```bash
# Explain CRDs too:
kubectl explain networkpolicy --api-version=crd.projectcalico.org/v1
```

> **Note:** CRD descriptions may be empty if the author didn't publish rich OpenAPI schemas.

---

## 13.7 Troubleshooting ⚡ GOOD TO KNOW

| Symptom | Cause | Fix |
|---------|-------|-----|
| `server doesn't have a resource type` | Typo / CRD not installed / wrong context | `kubectl api-resources` → check spelling |
| `explain cannot find resource` | Wrong context or extension missing | Confirm `kubectl config current-context` |
| `explain cannot find field` | Typo in path / field removed in your K8s version | Explain one segment at a time |
| Discovery looks outdated | Wrong context / API server was down | Reconnect, check context |
| CRD shows empty descriptions | CRD lacks OpenAPI schema | Read operator docs |

---

## 13.8 Singular, Plural & Qualified Names ⚡ GOOD TO KNOW

```bash
# All these work:
kubectl get pod          # singular
kubectl get pods         # plural (canonical)
kubectl get po           # short name

# When names conflict across groups, qualify:
kubectl get deployments.apps
kubectl get networkpolicies.networking.k8s.io
```

> **Best practice:** Use plural names in scripts/docs for clarity. Use short names for interactive speed.

---

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| Discovery commands | Same (`oc api-resources`, `oc api-versions`, `oc explain`) | Identical — `oc` supports all these |
| Extra resources | Only what you install | Many extra CRDs out-of-box (Routes, DeploymentConfigs, BuildConfigs, ImageStreams, etc.) |
| Routes vs Ingress | Ingress (networking.k8s.io/v1) | **Route** (route.openshift.io/v1) — OpenShift's native ingress |
| DeploymentConfig | Doesn't exist | `deploymentconfigs.apps.openshift.io` (legacy — Deployments preferred now) |
| BuildConfig | Doesn't exist | `buildconfigs.build.openshift.io` — built-in CI |
| ImageStream | Doesn't exist | `imagestreams.image.openshift.io` — image management |
| Security | PodSecurityAdmission | **SecurityContextConstraints** (SCCs) — `oc explain scc` |

```bash
# OpenShift has MANY more api-resources out of the box:
oc api-resources | wc -l
# Vanilla K8s: ~60-70 resources
# OpenShift: ~200+ resources (all the operators add CRDs)
```

> **Interview tip:** "OpenShift clusters have 3x more API resources than vanilla K8s because every built-in feature (Routes, Builds, ImageStreams, Machine API, etc.) is implemented as CRDs managed by Operators. Same discovery commands work — `oc api-resources`, `oc explain`."

---

## 💡 Interview Power Answers — API Discovery

**Q: "How do you find the correct apiVersion for a resource?"**
> "I run `kubectl api-resources` — it shows the preferred apiVersion for every resource type. For Deployments it shows `apps/v1`, for Pods it shows `v1`. I don't memorize these — I query the live cluster."

**Q: "How do you write YAML without Googling?"**
> "I use `kubectl explain`. Start with `kubectl explain deployment.spec` to see top-level fields, then drill down: `kubectl explain deployment.spec.template.spec.containers`. It shows field types, descriptions, and what's required. Then I validate with `kubectl apply --dry-run=server`."

**Q: "What's the difference between api-resources and api-versions?"**
> "`api-resources` lists resource TYPES (pods, deployments, services) with their group/version/kind. `api-versions` lists all served group/version PAIRS (apps/v1, batch/v1, v1). Use api-resources to find what you can create; api-versions to see what versions are served."

**Q: "How do CRDs show up?"**
> "When you install a CRD or Operator, new resources appear immediately in `kubectl api-resources`. For example, installing Calico adds resources under `crd.projectcalico.org`. You can explain them the same way: `kubectl explain <resource> --api-version=<group/version>`."


---

---

# 📁 SECTION 14: Kubernetes Namespaces

---

## 14.1 What Is a Namespace? ✅ MUST HAVE

A namespace = a **name scope** inside a cluster. Same name can exist in different namespaces.

```
┌─────────────────────────────────────────────────┐
│              ONE CLUSTER                         │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌── development ──┐    ┌── testing ───────┐   │
│  │                  │    │                  │   │
│  │  deploy/web ✅   │    │  deploy/web ✅   │   │
│  │  svc/web    ✅   │    │  svc/web    ✅   │   │
│  │                  │    │                  │   │
│  └──────────────────┘    └──────────────────┘   │
│                                                 │
│  Same name "web" — different namespaces = OK    │
│  Two "web" Deployments in SAME ns = ❌ Error    │
└─────────────────────────────────────────────────┘
```

**Key rules:**
- Cannot be nested (no namespace inside namespace)
- Not a separate cluster
- **Does NOT provide isolation** by itself (no network/security boundary)
- Names must be unique per kind within the same namespace

---

## 14.2 Default Namespaces ⚡ GOOD TO KNOW

| Namespace | Purpose |
|-----------|---------|
| `default` | Where objects go if you don't specify a namespace |
| `kube-system` | System components (CoreDNS, kube-proxy, etc.) |
| `kube-public` | Publicly readable cluster info |
| `kube-node-lease` | Node heartbeat Lease objects |

```bash
kubectl get namespaces
# NAME              STATUS   AGE
# default           Active   163m
# kube-node-lease   Active   163m
# kube-public       Active   163m
# kube-system       Active   163m
```

> ⚠️ Don't create namespaces starting with `kube-` — reserved for system use.

---

## 14.3 Namespaced vs Cluster-Scoped Resources ✅ MUST HAVE

| Scope | Examples | `-n` flag? |
|-------|----------|-----------|
| **Namespaced** | Pods, Deployments, Services, ConfigMaps, Secrets | ✅ Works |
| **Cluster-scoped** | Nodes, Namespaces, PersistentVolumes, ClusterRoles | ❌ Ignored |

```bash
# Find namespaced resources:
kubectl api-resources --namespaced=true

# Find cluster-scoped resources:
kubectl api-resources --namespaced=false
```

> `kubectl get nodes -n development` doesn't filter nodes — Nodes are cluster-scoped. `-n` is silently ignored.

---

## 14.4 Creating Namespaces ✅ MUST HAVE

### Imperative

```bash
kubectl create namespace development
kubectl create namespace testing
```

### Declarative (YAML)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: staging
```

```bash
kubectl apply -f namespace-staging.yaml
```

### Naming Rules

- Lowercase letters, numbers, hyphens only
- Must start/end with alphanumeric
- Max 63 characters
- Must be valid DNS label

```bash
kubectl create namespace Team_Dev   # ❌ FAILS — uppercase + underscore
kubectl create namespace team-dev   # ✅ Works
```

---

## 14.5 Working with Resources in Namespaces ✅ MUST HAVE

### Create in a Specific Namespace

```bash
# With -n flag:
kubectl create deployment web --image=nginx -n development
kubectl create deployment web --image=nginx -n testing

# Or in YAML:
metadata:
  name: web
  namespace: development
```

### List Resources

```bash
# In one namespace:
kubectl get pods -n development
kubectl get all -n testing

# Across ALL namespaces:
kubectl get pods -A
kubectl get pods --all-namespaces   # same thing
```

> **Tip:** If `kubectl get pods` shows nothing but `-A` shows your workload → you're in the wrong namespace.

---

## 14.6 Set Default Namespace (Context) ✅ MUST HAVE

Tired of typing `-n development` on every command? Set a default:

```bash
# Check current context:
kubectl config current-context

# Set default namespace:
kubectl config set-context --current --namespace=development

# Verify:
kubectl config get-contexts
# CURRENT  NAME                          NAMESPACE
# *        kubernetes-admin@kubernetes   development

# Now this works without -n:
kubectl get pods   # → shows development pods

# Reset to default:
kubectl config set-context --current --namespace=default
```

---

## 14.7 Namespace-Specific Contexts ⚡ GOOD TO KNOW

Create separate contexts per namespace (same cluster + user, different ns):

```bash
# Capture current context info:
ORIGINAL_CONTEXT=$(kubectl config current-context)
CLUSTER=$(kubectl config view --minify -o jsonpath='{.contexts[0].context.cluster}')
USER=$(kubectl config view --minify -o jsonpath='{.contexts[0].context.user}')

# Create namespace-specific contexts:
kubectl config set-context "${ORIGINAL_CONTEXT}-dev" \
  --cluster="$CLUSTER" --user="$USER" --namespace=development

kubectl config set-context "${ORIGINAL_CONTEXT}-test" \
  --cluster="$CLUSTER" --user="$USER" --namespace=testing

# Switch between them:
kubectl config use-context "${ORIGINAL_CONTEXT}-dev"
kubectl get pods   # → development pods

kubectl config use-context "${ORIGINAL_CONTEXT}-test"
kubectl get pods   # → testing pods
```

---

## 14.8 Cross-Namespace Service DNS ✅ MUST HAVE

Pods can reach Services in OTHER namespaces using DNS:

```
┌──────────────────────────────────────────────────────────┐
│         Cross-Namespace DNS Resolution                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Short form:                                             │
│    <service>.<namespace>                                 │
│    → web.testing                                         │
│                                                          │
│  Full form (FQDN):                                       │
│    <service>.<namespace>.svc.<cluster-domain>            │
│    → web.testing.svc.cluster.local                       │
│                                                          │
│  Within SAME namespace:                                   │
│    Just the service name → web                           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Example: Pod in `development` calls Service in `testing`

```bash
# Expose web in testing:
kubectl expose deployment web --port=80 -n testing

# From a pod in development:
curl http://web.testing          # cross-namespace!
curl http://web.testing.svc.cluster.local  # FQDN

# Within same namespace:
curl http://web                  # just the name
```

> ⚠️ **Namespaces do NOT block cross-namespace traffic.** You need **NetworkPolicy** for that.

---

## 14.9 Deleting Namespaces & Resources ⚡ GOOD TO KNOW

```bash
# Delete one resource:
kubectl delete deployment web -n development

# Delete "all" category resources (NOT everything!):
kubectl delete all --all -n testing
# ⚠️ "all" misses: Secrets, ConfigMaps, PVCs, Ingress, CRDs

# Delete entire namespace (removes EVERYTHING inside):
kubectl delete namespace staging
```

**Namespace stuck in `Terminating`?** Usually caused by:
- Finalizers that can't complete
- Unavailable API resources (broken CRD)
- Don't blindly edit finalizers — investigate first

---

## 14.10 What Namespaces DON'T Do ✅ MUST HAVE

| People Think... | Reality |
|----------------|---------|
| "Namespaces isolate networks" | ❌ No — use **NetworkPolicy** |
| "Namespaces control access" | ❌ No — use **RBAC** (Roles + RoleBindings) |
| "Namespaces limit resources" | ❌ No — use **ResourceQuota** + **LimitRange** |
| "Namespaces are security boundaries" | ❌ No — they're just name scopes |

```
Namespace alone = organization only

Namespace + NetworkPolicy = network isolation
Namespace + RBAC = access control
Namespace + ResourceQuota = resource limits
Namespace + ALL THREE = proper multi-tenancy
```

---

## 14.11 Common Mistakes ⚡ GOOD TO KNOW

| Mistake | What Happens | Fix |
|---------|-------------|-----|
| Wrong namespace in context | "No resources found" / "NotFound" | `kubectl get pods -A` to find it, then `-n` or fix context |
| Using `-n` with cluster-scoped resources | Silently ignored | Drop `-n` for nodes, PVs, ClusterRoles |
| `kubectl get all` thinks it's everything | Misses Secrets, ConfigMaps, CRDs | Use `api-resources --verbs=list` + xargs |
| Expecting ns deletion to be instant | Gets stuck on finalizers | Investigate finalizers, don't force-remove |
| Trying to move an object between namespaces | `metadata.namespace` is immutable | Recreate in new ns, delete from old |
| `sudo kubectl` gives different results | Different `~/.kube/config` for root | Run as your user |

---

### 🔴 OpenShift Difference

| Aspect | Kubernetes (Namespace) | OpenShift (Project) |
|--------|----------------------|---------------------|
| Name | `Namespace` | **`Project`** (wrapper around Namespace) |
| Create command | `kubectl create namespace dev` | `oc new-project dev` |
| Extra features | Nothing built-in | Projects auto-create: default ServiceAccount, RoleBindings, NetworkPolicy (if configured) |
| Self-service | Manual RBAC setup needed | Users can create their own Projects (self-service by default) |
| Quotas | Manual ResourceQuota | **ClusterResourceQuota** — admin can set quotas per user across all projects |
| Template | None | **Project template** — customize what gets created with every new project |
| Network isolation | None by default | **Default deny** NetworkPolicy (if multi-tenant SDN plugin or OVN-K with policy configured) |
| Listing | `kubectl get ns` | `oc get projects` (shows only what you have access to) |

```bash
# Kubernetes:
kubectl create namespace dev
kubectl get namespaces       # shows ALL namespaces

# OpenShift:
oc new-project dev           # creates Namespace + defaults
oc get projects              # shows only YOUR projects (RBAC filtered)
oc project dev               # switch namespace (like use-context)
```

> **Interview tip:** "In OpenShift, Projects = Namespaces + opinions. `oc new-project` auto-creates RBAC, ServiceAccounts, and optionally NetworkPolicy via a Project template. `oc get projects` only shows what you can access — unlike `kubectl get ns` which needs cluster-level permissions. OpenShift can provide network isolation out-of-box between projects."

---

## 💡 Interview Power Answers — Namespaces

**Q: "What are Kubernetes namespaces?"**
> "Namespaces are name scopes within a cluster. They let you have the same resource name in different namespaces — like having a 'web' Deployment in both development and testing. They're flat (no nesting), and they only provide organization — not security, network isolation, or resource limits. Those need RBAC, NetworkPolicy, and ResourceQuota."

**Q: "How do you access a Service in another namespace?"**
> "Use DNS: `<service>.<namespace>` — for example `web.testing` from any pod in the cluster. Full form is `web.testing.svc.cluster.local`. Cross-namespace traffic is allowed by default unless NetworkPolicy blocks it."

**Q: "How do you set a default namespace?"**
> "I use `kubectl config set-context --current --namespace=development`. This updates my kubeconfig so all commands default to that namespace. I can also create separate contexts per namespace and switch with `use-context`."

**Q: "Do namespaces provide isolation?"**
> "No — by themselves they're just name scopes. For real isolation you need: RBAC for access control, NetworkPolicy for network isolation, ResourceQuota for resource limits. Namespaces are the boundary those tools attach to, but they don't enforce anything alone."

**Q: "What's the difference between OpenShift Projects and K8s Namespaces?"**
> "Projects are Namespaces with extras. `oc new-project` auto-creates RBAC bindings, default ServiceAccount, and can apply a project template. Users can self-serve projects. `oc get projects` is RBAC-filtered — you only see what you can access. OpenShift can also apply default-deny NetworkPolicy per project for multi-tenancy."


---

---

# 🏷️ SECTION 15: Labels and Selectors

---

## 15.1 What Are Labels? ✅ MUST HAVE

Labels = key/value metadata on any K8s object. Used to **identify, group, and select** objects.

```
┌──────────────────────────────────────────────────┐
│              Labels in Action                     │
├──────────────────────────────────────────────────┤
│                                                  │
│  Pod "web-abc123":                               │
│    labels:                                       │
│      app: web                                    │
│      tier: frontend                              │
│      environment: production                     │
│                                                  │
│  Used by:                                        │
│    • kubectl get -l app=web  (filtering)         │
│    • Deployment selector     (ownership)         │
│    • Service selector        (traffic routing)   │
│    • NetworkPolicy           (security rules)    │
│    • Scheduler               (node selection)    │
└──────────────────────────────────────────────────┘
```

**Labels are NOT:**
- Names (many objects can share the same label)
- Annotations (those are for metadata selectors ignore)

---

## 15.2 Label Syntax Rules ⚡ GOOD TO KNOW

```
Format: [prefix/]name=value

Examples:
  environment=production          ← simple
  tier=frontend                   ← simple
  app.kubernetes.io/name=web      ← with prefix
  example.com/owner=platform      ← custom prefix
```

| Part | Max Length | Rules |
|------|-----------|-------|
| Name | 63 chars | Start/end alphanumeric; `-`, `_`, `.` allowed inside |
| Prefix (optional) | 253 chars | DNS subdomain format |
| Value | 63 chars (or empty) | Same as name rules |

**Reserved prefixes:** `kubernetes.io/` and `k8s.io/` — don't use for custom labels.

---

## 15.3 Where Labels Go — The Three Label Areas ✅ MUST HAVE

This is the **#1 confusion point**. A Deployment has THREE places for labels:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:                    # ① OBJECT labels (on the Deployment itself)
    app: web
spec:
  selector:
    matchLabels:             # ② SELECTOR (which Pods this controller owns)
      app: web
  template:
    metadata:
      labels:                # ③ TEMPLATE labels (copied onto every Pod)
        app: web
        tier: frontend
        environment: dev
    spec:
      containers:
      - name: nginx
        image: nginx:1.28
```

```
┌──────────────────────────────────────────────────────────┐
│         The Three Label Areas — Deployment               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ① metadata.labels                                       │
│     → Tags the Deployment OBJECT itself                  │
│     → kubectl get deploy --show-labels reads this        │
│     → Does NOT reach Pods                                │
│                                                          │
│  ② spec.selector.matchLabels                             │
│     → Which Pods this controller OWNS                    │
│     → IMMUTABLE after creation!                          │
│     → Must match template labels                         │
│                                                          │
│  ③ spec.template.metadata.labels                         │
│     → Copied onto every Pod the controller creates       │
│     → Must satisfy selector requirements                 │
│     → Can have EXTRA labels beyond selector              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Controller Ownership Flow

```
Deployment (metadata.labels: app=web)
  └── ReplicaSet (adds pod-template-hash automatically)
        └── Pod (gets spec.template.metadata.labels)

StatefulSet (no ReplicaSet in between)
  └── Pod (gets spec.template.metadata.labels directly)
```

### Critical Rules

| Rule | Consequence |
|------|-------------|
| Selector must match template | API rejects the object if they disagree |
| Selector is immutable | Can't change after first `apply` — must recreate |
| `kubectl label deployment` only changes ① | Does NOT add labels to Pods |
| Template labels persist across Pod recreates | Manual `kubectl label pod` labels don't survive |

---

## 15.4 kubectl label Commands ✅ MUST HAVE

### Add Labels

```bash
kubectl label pod nginx environment=production
kubectl label deployment web tier=frontend
kubectl label node worker01 disktype=ssd
kubectl label namespace dev environment=dev
```

### Multiple Labels at Once

```bash
kubectl label pod nginx release=2026.07 cost-center=eng
```

### Label All Matching Pods

```bash
kubectl label pods -l app=web tier=frontend
```

### Overwrite Existing Label

```bash
kubectl label pod nginx environment=staging --overwrite
# Without --overwrite → ERROR if key already exists
```

### Remove a Label (trailing hyphen)

```bash
kubectl label pod nginx environment-
kubectl label node worker01 disktype-
```

### Dry Run (Preview)

```bash
kubectl label pod nginx testkey=val --dry-run=client -o yaml
```

---

## 15.5 View Labels ⚡ GOOD TO KNOW

```bash
# Show all labels:
kubectl get pods --show-labels

# Show specific label keys as columns:
kubectl get pods -L app,tier,environment
# NAME       APP   TIER       ENVIRONMENT
# web-xxx    web   frontend   development

# Full YAML:
kubectl get pod web-xxx -o yaml
# → metadata.labels shows everything
```

---

## 15.6 Label Selectors — Filtering with -l ✅ MUST HAVE

### Equality-Based

```bash
kubectl get pods -l app=web              # equals
kubectl get pods -l tier==frontend       # same as =
kubectl get pods -l environment!=prod    # not equal (also matches missing key)
```

### Set-Based

```bash
kubectl get pods -l 'environment in (dev,staging)'
kubectl get pods -l 'tier notin (backend,database)'
kubectl get pods -l tier                 # key EXISTS
kubectl get pods -l '!owner'             # key DOES NOT EXIST
```

### Combine (AND logic)

```bash
kubectl get pods -l 'app=web,tier=frontend'
kubectl get pods -l 'app=web,environment in (dev,staging)'
```

> **No OR between different keys.** Use `in (...)` for OR within one key.

### Selector Operators Summary

| Operator | CLI | YAML (matchExpressions) | Meaning |
|----------|-----|------------------------|---------|
| `=` / `==` | `-l app=web` | `In` with one value | Exact match |
| `!=` | `-l app!=web` | `NotIn` | Not this value (or key absent) |
| `in` | `-l 'env in (a,b)'` | `In` | Value in set |
| `notin` | `-l 'env notin (a,b)'` | `NotIn` | Value not in set (or key absent) |
| exists | `-l tier` | `Exists` | Key present (any value) |
| not exists | `-l '!tier'` | `DoesNotExist` | Key absent |

---

## 15.7 matchLabels & matchExpressions in YAML ✅ MUST HAVE

### matchLabels (Equality)

```yaml
selector:
  matchLabels:
    app: web
    tier: frontend
```

### matchExpressions (Set-Based)

```yaml
selector:
  matchExpressions:
  - key: environment
    operator: In
    values: [development, staging]
  - key: tier
    operator: Exists
  - key: deprecated
    operator: DoesNotExist
```

### Combined (AND logic)

```yaml
selector:
  matchLabels:
    app: web
  matchExpressions:
  - key: environment
    operator: In
    values: [development, staging]
```

| Operator | `values` required? |
|----------|-------------------|
| In | Yes |
| NotIn | Yes |
| Exists | No (omit `values`) |
| DoesNotExist | No (omit `values`) |

> **Service spec.selector** only supports simple key=value map (no matchExpressions).

---

## 15.8 How Services Use Labels ✅ MUST HAVE

```
┌────────────────────────────────────────────────────────┐
│          Service → Pod Label Matching                  │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Service (spec.selector: app=web)                      │
│    │                                                   │
│    ▼  Finds all Pods with label app=web                │
│                                                        │
│  Pod A (app=web) ─── ✅ gets traffic                   │
│  Pod B (app=web) ─── ✅ gets traffic                   │
│  Pod C (app=api) ─── ❌ not selected                   │
│                                                        │
│  EndpointSlice stores the selected Pod IPs             │
└────────────────────────────────────────────────────────┘
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web        # must match Pod labels exactly
  ports:
  - port: 80
    targetPort: 80
```

**No endpoints?** → Service selector doesn't match any Pod labels. Check:
```bash
kubectl get svc web -o jsonpath='{.spec.selector}'
kubectl get pods --show-labels
kubectl get endpointslices -l kubernetes.io/service-name=web
```

---

## 15.9 Recommended Application Labels ⚡ GOOD TO KNOW

```yaml
labels:
  app.kubernetes.io/name: web
  app.kubernetes.io/instance: web-prod
  app.kubernetes.io/version: "2.0"
  app.kubernetes.io/component: frontend
  app.kubernetes.io/part-of: storefront
  app.kubernetes.io/managed-by: Helm
```

Not required — but consistent labeling helps dashboards, monitoring, and `kubectl` filtering across teams.

---

## 15.10 Labels vs Annotations vs Field Selectors ⚡ GOOD TO KNOW

| Feature | Labels | Annotations | Field Selectors |
|---------|--------|-------------|-----------------|
| Purpose | Selection & grouping | Metadata (non-identifying) | Filter by resource fields |
| Selectable? | ✅ `-l` | ❌ | ✅ `--field-selector` |
| Size limit | 63 chars value | 256 KB total | N/A |
| Example | `app=web` | `description: "Main web app"` | `status.phase=Running` |

```bash
# Label selector:
kubectl get pods -l app=web

# Field selector:
kubectl get pods --field-selector status.phase=Running
```

---

## 15.11 Patching Template Labels (Persist to New Pods) ✅ MUST HAVE

`kubectl label deployment` changes the Deployment object only. To add a label that **survives Pod recreates**, patch the template:

```bash
kubectl patch deployment web -n dev --type=merge \
  -p '{"spec":{"template":{"metadata":{"labels":{"environment":"production"}}}}}'

# Wait for rollout:
kubectl rollout status deployment/web -n dev
```

> ⚠️ Changing template labels triggers a new ReplicaSet + rolling update (Pod replacement).

---

## 15.12 Troubleshooting ✅ MUST HAVE

| Symptom | Cause | Fix |
|---------|-------|-----|
| `already has a value, --overwrite is false` | Key exists | Add `--overwrite` |
| `-l` returns nothing | Wrong namespace/spelling | `--show-labels` to inspect |
| Labeled Deployment but Pods don't have it | Only `metadata.labels` changed | Patch `spec.template.metadata.labels` |
| `selector does not match template labels` | Selector & template disagree | Align them in YAML |
| `spec.selector: field is immutable` | Tried to change selector | Recreate the Deployment |
| Service has no endpoints | Selector doesn't match Pod labels | Compare selectors & Pod labels |
| Label disappears after Pod restart | Only set with `kubectl label pod` | Add to template labels |
| Shell error with `in (...)` | Unquoted parentheses | Quote: `-l 'env in (a,b)'` |

---

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| Label commands | `kubectl label` | `oc label` (identical) |
| Selectors | Same syntax | Same syntax |
| Extra auto-labels | `pod-template-hash` | Same + `deploymentconfig` label (legacy DCs) |
| Node labels | Manual | **Machine API** can set labels via MachineSet spec (declarative) |
| App labels | Convention only | OpenShift console uses `app` and `app.kubernetes.io/` labels for topology view |
| Label-based routing | Via Ingress + Service selector | **Routes** use Service selector (same mechanism) |

```bash
# OpenShift node labeling via MachineSet (declarative):
oc edit machineset worker -n openshift-machine-api
# spec.template.spec.metadata.labels:
#   node-role.kubernetes.io/infra: ""
#   disktype: ssd

# Nodes created from this MachineSet auto-get these labels
# vs Kubernetes: kubectl label node worker01 disktype=ssd (imperative, per-node)
```

> **Interview tip:** "In OpenShift, node labels can be set declaratively via MachineSets — new nodes get labels automatically. In vanilla K8s, you label nodes imperatively or use a node-labeler. The label/selector system itself is identical."

---

## 💡 Interview Power Answers — Labels & Selectors

**Q: "Explain how labels and selectors work in Kubernetes."**
> "Labels are key/value pairs on any K8s object. Selectors filter objects by those labels. Controllers use selectors to own Pods — a Deployment's `spec.selector.matchLabels` defines which Pods belong to it. Services use selectors to route traffic to matching Pods. In kubectl, `-l app=web` filters output. Labels support equality (`=`, `!=`) and set-based (`in`, `notin`, `exists`) operations."

**Q: "A Deployment has labels in three places. What's the difference?"**
> "`metadata.labels` tags the Deployment object itself. `spec.selector.matchLabels` defines which Pods the controller owns — it's immutable after creation. `spec.template.metadata.labels` gets copied onto every Pod the controller creates. The selector must match the template. Extra template labels can exist for Services to use."

**Q: "Why doesn't labeling a Deployment change its Pods?"**
> "`kubectl label deployment` only changes `metadata.labels` on the Deployment object. Pods get labels from `spec.template.metadata.labels`. To add a label to all managed Pods, patch the template — which triggers a rolling update."

**Q: "Service has no endpoints — how do you troubleshoot?"**
> "Check the Service selector with `kubectl get svc -o jsonpath='{.spec.selector}'`. Then check Pod labels with `--show-labels`. If they don't match, either fix the Service selector or fix the Pod template labels. Also check EndpointSlices to confirm the mapping."

**Q: "What's the difference between matchLabels and matchExpressions?"**
> "`matchLabels` is simple equality — key must equal value. `matchExpressions` supports set-based logic: In, NotIn, Exists, DoesNotExist. You can combine both — they're ANDed together. Service selectors only support the simple map form, not matchExpressions."


---

---

# 🫛 SECTION 16: Pods & Pod Lifecycle

---

## 16.1 What Is a Pod? ✅ MUST HAVE

A Pod = the **smallest deployable unit** in Kubernetes. It wraps one or more containers on the same node.

```
┌────────────────────────────────────────────────────┐
│                    POD                              │
├────────────────────────────────────────────────────┤
│                                                    │
│  Containers share:                                 │
│    • Network namespace (same IP)                   │
│    • Port space (containers can't use same port)   │
│    • Volumes (shared storage)                      │
│    • IPC                                           │
│                                                    │
│  ┌───────────┐  ┌───────────┐                     │
│  │Container 1│  │Container 2│  (multi-container    │
│  │  (nginx)  │  │ (sidecar) │   is optional)      │
│  └───────────┘  └───────────┘                     │
│         │              │                           │
│         └──── localhost ─────┘                     │
│                                                    │
│  Pod IP: 192.168.5.10                              │
│  Node: worker01                                    │
└────────────────────────────────────────────────────┘
```

**Most Pods have ONE container.** Multi-container = when processes must share localhost or volumes (sidecars, init containers).

---

## 16.2 Pod Manifest Structure ✅ MUST HAVE

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
  namespace: pod-lifecycle-demo
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.27-alpine
    ports:
    - name: http
      containerPort: 80
  restartPolicy: Always
```

| Section | What It Contains | Who Writes It |
|---------|-----------------|---------------|
| `metadata` | name, namespace, labels | You |
| `spec` | containers, restartPolicy, volumes, lifecycle | You |
| `status` | phase, conditions, containerStatuses, podIP | Kubernetes (read-only) |

---

## 16.3 Pod Phases ✅ MUST HAVE

```
┌─────────────────────────────────────────────────────┐
│              Pod Phase Flow                          │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────┐    ┌─────────┐    ┌───────────┐      │
│  │ Pending │───►│ Running │───►│ Succeeded │      │
│  └─────────┘    └─────────┘    └───────────┘      │
│       │              │                              │
│       │              └─────────►┌────────┐         │
│       │                         │ Failed │         │
│       └─────────────────────────┴────────┘         │
│                                                     │
│  Long-running Pods stay in Running until deleted    │
└─────────────────────────────────────────────────────┘
```

| Phase | Meaning |
|-------|---------|
| **Pending** | Accepted, but not running yet (scheduling, image pull, volume mount) |
| **Running** | Bound to node, at least one container is running/starting/restarting |
| **Succeeded** | All containers exited with code 0, won't restart |
| **Failed** | All containers terminated, at least one failed, won't restart |
| **Unknown** | Node communication lost |

---

## 16.4 Phase vs STATUS vs Container State ✅ MUST HAVE

**These are THREE different things — don't confuse them:**

| Layer | Where | Examples |
|-------|-------|----------|
| **Pod Phase** | `.status.phase` | Pending, Running, Succeeded, Failed |
| **kubectl STATUS** | `kubectl get pods` column | CrashLoopBackOff, ImagePullBackOff, ContainerCreating, Terminating |
| **Container State** | `.status.containerStatuses[].state` | Waiting, Running, Terminated |

```bash
# Read phase:
kubectl get pod web -o jsonpath='{.status.phase}'

# Read container state:
kubectl get pod web -o jsonpath='{.status.containerStatuses[0].state}'

# Read everything together:
kubectl describe pod web
```

> **Key insight:** A Pod can be phase=Running but STATUS=CrashLoopBackOff (container keeps crashing but Pod is still bound to node and restarting).

---

## 16.5 Container States ⚡ GOOD TO KNOW

| State | Meaning | Common Causes |
|-------|---------|---------------|
| **Waiting** | Not running yet | Image pull, sandbox setup, restart backoff |
| **Running** | Process executing | Normal operation |
| **Terminated** | Process stopped | Clean exit, error, killed by kubelet |

After restart, previous instance → `lastState`. View with:
```bash
kubectl logs <pod> --previous
```

---

## 16.6 Pod Conditions ✅ MUST HAVE

Conditions = independent checkpoints. A Pod can be Running but NOT Ready.

| Condition | Meaning |
|-----------|---------|
| `PodScheduled` | Node assigned |
| `PodReadyToStartContainers` | Sandbox + networking ready |
| `Initialized` | All init containers done |
| `ContainersReady` | All containers pass readiness |
| `Ready` | Pod can serve traffic (what Services check) |

```bash
kubectl get pod web -o jsonpath='{range .status.conditions[*]}{.type}{"\t"}{.status}{"\n"}{end}'
```

> **Services only route traffic to Pods where `Ready=True`.**

---

## 16.7 restartPolicy ✅ MUST HAVE

| Value | Behavior | Use Case |
|-------|----------|----------|
| **Always** (default) | Restart regardless of exit code | Long-running apps (web servers) |
| **OnFailure** | Restart only on non-zero exit | Batch jobs that should retry |
| **Never** | Never restart | One-shot tasks, debugging |

### Restart Backoff (CrashLoopBackOff)

```
Container exits → kubelet restarts with exponential backoff:
  10s → 20s → 40s → 80s → 160s → 300s (max 5 min)

After 10 minutes of successful running → backoff resets

During backoff:
  Container state = Waiting
  Reason = CrashLoopBackOff
  Pod phase = still Running (!)
```

### Example: Always (keeps restarting)

```yaml
spec:
  restartPolicy: Always
  containers:
  - name: demo
    image: busybox:1.36
    command: ["sh", "-c", "echo Starting; sleep 5; exit 1"]
# → Phase stays Running, RESTARTS increases, eventually CrashLoopBackOff
```

### Example: Never (fails once, stays Failed)

```yaml
spec:
  restartPolicy: Never
  containers:
  - name: demo
    image: busybox:1.36
    command: ["sh", "-c", "echo done; exit 1"]
# → Phase becomes Failed, no restarts
```

---

## 16.8 Lifecycle Hooks ⚡ GOOD TO KNOW

| Hook | When It Runs | Use Case |
|------|-------------|----------|
| **postStart** | After container created (no guarantee vs entrypoint order) | Write config files, register with service |
| **preStop** | Before stop signal during termination | Drain connections, deregister |

### Handler Types

| Handler | Description |
|---------|------------|
| `exec` | Run a command inside container |
| `httpGet` | HTTP GET against Pod IP + port |
| `sleep` | Wait fixed duration (K8s 1.29+) |

```yaml
spec:
  terminationGracePeriodSeconds: 30
  containers:
  - name: nginx
    image: nginx:1.27-alpine
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo started > /tmp/poststart"]
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 5"]
```

**Key facts:**
- postStart failure → container terminated
- preStop shares time budget with grace period (not additive!)
- Long postStart delays container becoming Ready

---

## 16.9 Graceful Termination Sequence ✅ MUST HAVE

When you delete a Pod (`kubectl delete pod web`):

```
┌──────────────────────────────────────────────────────────┐
│          Pod Termination Sequence                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. API server sets deletion timestamp + grace period    │
│                                                          │
│  2. Two things happen IN PARALLEL:                       │
│     ┌─────────────────────┐  ┌────────────────────────┐ │
│     │ kubelet shutdown    │  │ EndpointSlice update   │ │
│     │                     │  │ (removes from Service) │ │
│     │ 3. Run preStop hook │  │                        │ │
│     │ 4. Send SIGTERM     │  │ terminating=true       │ │
│     │ 5. Wait grace period│  │ ready=false            │ │
│     │ 6. SIGKILL (force)  │  │                        │ │
│     └─────────────────────┘  └────────────────────────┘ │
│                                                          │
│  Default grace period: 30 seconds                        │
│  preStop + SIGTERM share this budget (not additive)      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

```bash
# Custom grace period:
kubectl delete pod web --grace-period=10

# Force delete (dangerous — last resort):
kubectl delete pod web --force --grace-period=0
```

> **Interview tip:** "preStop and SIGTERM share the same grace period budget. If preStop takes 25 seconds and grace period is 30, the app only gets 5 seconds to handle SIGTERM before SIGKILL."

---

## 16.10 Bare Pod vs Controller-Managed Pod ✅ MUST HAVE

| | Bare Pod | Controller-Managed Pod |
|--|----------|----------------------|
| Created by | You (`kubectl apply`) | Controller (Deployment, StatefulSet, etc.) |
| Auto-replaced? | ❌ No — gone forever | ✅ Yes — controller recreates |
| Rollout/history? | ❌ No | ✅ Yes |
| Use case | Learning, debugging, smoke tests | **ALL production workloads** |

| Requirement | Controller |
|-------------|-----------|
| Stateless long-running app | Deployment |
| Stable identity + per-replica storage | StatefulSet |
| One Pod per node | DaemonSet |
| Run to completion | Job |
| Scheduled batch | CronJob |

> **Rule:** Never use bare Pods in production. Always use a controller.

---

## 16.11 Essential Pod Commands ⚡ GOOD TO KNOW

```bash
# Create:
kubectl apply -f web-pod.yaml

# Status:
kubectl get pod web -o wide
kubectl get pod web --watch

# Inspect (phase + conditions + events):
kubectl describe pod web

# Logs:
kubectl logs web
kubectl logs web --previous    # after restart
kubectl logs web -c sidecar    # specific container

# Exec into:
kubectl exec web -- nginx -v
kubectl exec -it web -- /bin/sh

# Delete:
kubectl delete pod web --grace-period=10

# Events (sorted):
kubectl get events --sort-by=.lastTimestamp
```

---

## 16.12 Troubleshooting Pod States ✅ MUST HAVE

| Symptom | First Step | Common Cause |
|---------|-----------|--------------|
| **Pending** | `kubectl describe` → Events | Scheduling failure, no matching node, PVC not bound |
| **CrashLoopBackOff** | `kubectl logs --previous` | App error, missing config, wrong command |
| **Running but 0/1 Ready** | Check conditions + readiness probe | Probe failing, app not listening on expected port |
| **ImagePullBackOff** | Check image name + registry auth | Typo in image, private registry, no pull secret |
| **Terminating (stuck)** | Check finalizers + preStop | Finalizer can't complete, volume won't detach |
| **Error (no restarts)** | `describe` → exit code | restartPolicy: Never + non-zero exit |

```bash
# The troubleshooting trio:
kubectl describe pod <name>
kubectl logs <name> --previous
kubectl get events --sort-by=.lastTimestamp
```

---

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| Pod creation | Any user can create Pods in their namespace | **SCCs (Security Context Constraints)** restrict what Pods can do |
| Default SCC | No restrictions (unless PSA configured) | `restricted-v2` SCC — no root, no privileged, read-only root FS |
| Root containers | Allowed by default | ❌ Blocked by default (must request privileged SCC) |
| UID | Container runs as image-defined UID | OpenShift assigns **random UID** from namespace range |
| Pod Security | PSA (Pod Security Admission) labels on namespace | SCCs (more granular than PSA) |
| Debugging | `kubectl exec` / `kubectl debug` | `oc debug node/worker01` — spawns privileged pod on node |
| Init containers | Same | Same |
| Lifecycle hooks | Same | Same |

```bash
# OpenShift: check which SCC a Pod uses:
oc get pod web -o yaml | grep scc
# openshift.io/scc: restricted-v2

# Debug a node directly:
oc debug node/worker01
# → spawns a privileged pod with host filesystem at /host
```

> **Interview tip:** "In OpenShift, Pods run as a random UID with restricted-v2 SCC by default. You can't run as root unless explicitly granted a privileged SCC. This is a major security difference from vanilla K8s where any container can run as root by default. OpenShift also adds `oc debug` for node-level troubleshooting."

---

## 💡 Interview Power Answers — Pods & Lifecycle

**Q: "What is a Pod?"**
> "A Pod is the smallest deployable unit in Kubernetes. It's one or more containers that share a network namespace (same IP), port space, and optional volumes on a single node. Most Pods have one container. The Pod is what gets scheduled — not the container directly."

**Q: "Explain Pod lifecycle phases."**
> "A Pod goes through: Pending (accepted but not running — scheduling/image pull), Running (at least one container is up), then Succeeded (all exited 0) or Failed (at least one exited non-zero). The kubectl STATUS column shows more detail like CrashLoopBackOff or ImagePullBackOff — those aren't phases, they're container reasons."

**Q: "What is CrashLoopBackOff?"**
> "It means a container keeps crashing and the kubelet is applying exponential backoff between restarts (10s, 20s, 40s... up to 5 min). The Pod phase is still Running because it's bound to a node. To debug: `kubectl logs --previous` to see the crash output, and `kubectl describe` for events and exit codes."

**Q: "What happens when you delete a Pod?"**
> "Kubernetes starts graceful shutdown. In parallel: the kubelet runs preStop hook then sends SIGTERM, and the EndpointSlice controller removes the Pod from Service endpoints. After the grace period (default 30s), if the process is still running, SIGKILL is sent. preStop and SIGTERM share the same grace period budget."

**Q: "Why not use bare Pods in production?"**
> "Bare Pods aren't replaced when they die or their node fails. Controllers (Deployment, StatefulSet, Job) automatically recreate missing Pods to maintain desired state. Bare Pods are only for learning and quick tests."

**Q: "How does OpenShift differ with Pods?"**
> "OpenShift applies Security Context Constraints (SCCs) by default. The `restricted-v2` SCC blocks root containers, assigns random UIDs, and enforces read-only root filesystems. In vanilla K8s, any container can run as root unless you configure Pod Security Admission. OpenShift also adds `oc debug node/` for privileged node access."


---

---

# 🚀 SECTION 17: Deployments, Rolling Updates & Rollbacks

---

## 17.1 What Is a Deployment? ✅ MUST HAVE

A Deployment manages **stateless, long-running Pods** via ReplicaSets. It handles scaling, rolling updates, and rollbacks.

```
┌──────────────────────────────────────────────────────┐
│            Deployment Ownership Chain                 │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Deployment (web)                                    │
│    │                                                 │
│    ├── ReplicaSet (web-86b6cb7b94) ← revision 1     │
│    │     ├── Pod (web-86b6cb7b94-abc)                │
│    │     ├── Pod (web-86b6cb7b94-def)                │
│    │     └── Pod (web-86b6cb7b94-ghi)                │
│    │                                                 │
│    └── ReplicaSet (web-5b6bd7f99b) ← revision 2     │
│          (scaled to 0 after rollout)                 │
│                                                      │
│  Each Pod-template change = new ReplicaSet = new     │
│  revision. Old RS kept for rollback.                 │
└──────────────────────────────────────────────────────┘
```

**Key facts:**
- Deployment → creates/manages ReplicaSets → which create Pods
- Pod-template changes create new revisions (new ReplicaSet)
- Scaling does NOT create a new revision
- Old ReplicaSets kept for rollback (controlled by `revisionHistoryLimit`)

---

## 17.2 Deployment YAML ✅ MUST HAVE

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: deploy-lab
  labels:
    app: web
spec:
  replicas: 3                      # desired Pod count
  revisionHistoryLimit: 5          # old RS to keep for rollback
  progressDeadlineSeconds: 60      # time before "stuck" rollout reported
  selector:
    matchLabels:
      app: web                     # IMMUTABLE after creation
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                  # extra Pods allowed above desired
      maxUnavailable: 0            # desired Pods allowed down
  template:
    metadata:
      labels:
        app: web                   # must match selector
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
```

### kubectl get deployments Columns

| Column | Meaning |
|--------|---------|
| READY | Ready replicas / desired |
| UP-TO-DATE | Pods running latest template |
| AVAILABLE | Replicas satisfying availability |

---

## 17.3 Scaling ✅ MUST HAVE

```bash
# Imperative scale:
kubectl scale deployment web --replicas=4

# Verify:
kubectl get deployment web
# READY: 4/4

# Declarative (reapply YAML with different replicas):
kubectl apply -f web-deploy.yaml   # restores YAML's value
```

**Key point:** Scaling changes `spec.replicas` only — no new revision, no new ReplicaSet.

> If you scale imperatively but later `kubectl apply` the original YAML, the YAML's replica count wins (declarative desired state).

---

## 17.4 Rolling Updates ✅ MUST HAVE

### Trigger a Rolling Update

Any change to `spec.template` triggers a new rollout:

```bash
# Change image:
kubectl set image deployment/web nginx=nginx:1.27.0

# Or change env:
kubectl set env deployment/web VERSION=2.0

# Or edit YAML and apply:
kubectl apply -f web-deploy.yaml
```

### What Happens During Rolling Update

```
┌──────────────────────────────────────────────────────────┐
│        Rolling Update (maxSurge:1, maxUnavailable:0)     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Desired: 3 replicas                                     │
│                                                          │
│  Step 1: Create 1 new Pod (surge) → total: 4             │
│  Step 2: New Pod Ready → terminate 1 old Pod → total: 3  │
│  Step 3: Create 1 new Pod → total: 4                     │
│  Step 4: New Pod Ready → terminate 1 old → total: 3      │
│  Step 5: Create 1 new Pod → total: 4                     │
│  Step 6: New Pod Ready → terminate last old → total: 3   │
│                                                          │
│  Result: 3 new Pods, 0 old Pods, zero downtime           │
└──────────────────────────────────────────────────────────┘
```

### Watch the Rollout

```bash
# Watch RS and Pods in real-time:
kubectl get rs,pod -l app=web --watch

# Check rollout status:
kubectl rollout status deployment/web --timeout=120s

# Confirm image:
kubectl get deployment web -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

## 17.5 Update Strategy ✅ MUST HAVE

### RollingUpdate (Default)

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1          # how many EXTRA Pods above desired
    maxUnavailable: 0    # how many desired Pods can be DOWN
```

| Preset | maxSurge | maxUnavailable | Behavior |
|--------|----------|----------------|----------|
| **Availability-focused** | 1 | 0 | Always full capacity, surge one extra |
| **Capacity-constrained** | 0 | 1 | No extra resources, replace in-place |
| **Fast rollout** | 25% | 25% | Default — balanced speed + availability |

**Rules:**
- Both accept absolute numbers or percentages
- Both cannot be 0 at the same time
- maxUnavailable % → rounded DOWN
- maxSurge % → rounded UP

### Recreate Strategy

```yaml
strategy:
  type: Recreate
```

- Kills ALL old Pods BEFORE starting new ones
- Causes downtime
- Use when: app can't run two versions simultaneously

---

## 17.6 Pause & Resume (Batch Changes) ⚡ GOOD TO KNOW

Pause a Deployment to batch multiple template changes into ONE rollout:

```bash
# Pause:
kubectl rollout pause deployment/web

# Make multiple changes (no rollout happens yet):
kubectl set image deployment/web nginx=nginx:1.27.0
kubectl set env deployment/web RELEASE_STAGE=canary

# Resume (triggers ONE rollout with all changes):
kubectl rollout resume deployment/web
kubectl rollout status deployment/web
```

**While paused:**
- UP-TO-DATE = 0 (new template not deployed yet)
- AVAILABLE stays at full count (old Pods still running)
- Cannot rollback until resumed
- Progress deadline timer doesn't count

---

## 17.7 Revision History & Rollback ✅ MUST HAVE

### View History

```bash
kubectl rollout history deployment/web
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         <none>
# 3         <none>

# Details of specific revision:
kubectl rollout history deployment/web --revision=2
```

### Rollback

```bash
# Rollback to previous revision:
kubectl rollout undo deployment/web

# Rollback to specific revision:
kubectl rollout undo deployment/web --to-revision=1

# Verify:
kubectl rollout status deployment/web
kubectl get deployment web -o jsonpath='{.spec.template.spec.containers[0].image}'
```

**How rollback works:**
- Restores the old Pod template as a NEW revision
- Revision numbers shift (rolled-back revision disappears, new number assigned)
- Always rerun `rollout history` before using `--to-revision`

### revisionHistoryLimit

| Value | Behavior |
|-------|----------|
| 5 (default in many setups) | Keeps 5 old ReplicaSets for rollback |
| 0 | No rollback possible (history deleted) |
| 10 | More rollback options but more objects in etcd |

> Old ReplicaSets are the rollback mechanism. Once cleaned up, that revision is gone forever.

---

## 17.8 Failed Rollout Example ✅ MUST HAVE

```bash
# Deploy bad image:
kubectl set image deployment/web nginx=nginx:not-a-real-tag

# Watch it fail:
kubectl rollout status deployment/web --timeout=75s
# → error: deployment "web" exceeded its progress deadline

# What happened:
kubectl get rs,pod -l app=web
# Old RS: 3/3 Ready (still serving — maxUnavailable:0 saved us!)
# New RS: 1/1 with 0 Ready (ErrImagePull/ImagePullBackOff)

# Fix: rollback
kubectl rollout undo deployment/web
kubectl rollout status deployment/web
```

**Key insight:** With `maxUnavailable: 0`, a failed rollout doesn't kill your running Pods. Only the surge Pod fails. Your app stays up!

---

## 17.9 Deployment Conditions ⚡ GOOD TO KNOW

```bash
kubectl describe deployment web
# Conditions:
#   Type           Status  Reason
#   Available      True    MinimumReplicasAvailable
#   Progressing    True    NewReplicaSetAvailable
```

| Condition | Meaning |
|-----------|---------|
| `Available` | Min replicas are Ready |
| `Progressing` | Rollout active or completed |
| `ReplicaFailure` | Can't create replicas (quota, etc.) |

When `progressDeadlineSeconds` expires:
```
Progressing: False
Reason: ProgressDeadlineExceeded
```
> This does NOT auto-rollback. The controller keeps retrying. You must rollback manually.

---

## 17.10 Troubleshooting ✅ MUST HAVE

| Symptom | Cause | Fix |
|---------|-------|-----|
| Rollout stuck waiting | New Pods not Ready | `kubectl describe pod` + check probes/images |
| `ProgressDeadlineExceeded` | New Pods never healthy | Rollback: `kubectl rollout undo` |
| No Pods created | Quota, admission error, selector mismatch | Check events, describe deployment |
| Old RS not scaling down | New Pods not passing readiness | Fix readiness probe or image |
| `kubectl apply` reverted my image | YAML has old image | Update YAML to match desired state |
| Image update didn't trigger rollout | Changed outside `spec.template` | Only template changes trigger rollouts |
| UP-TO-DATE = 0 | Deployment is paused | `kubectl rollout resume` |

---

## 17.11 Key Commands Cheat Sheet ⚡ GOOD TO KNOW

```bash
# Create:
kubectl create deployment web --image=nginx:1.25.4 --replicas=3

# Scale:
kubectl scale deployment web --replicas=5

# Update image:
kubectl set image deployment/web nginx=nginx:1.27.0

# Set env:
kubectl set env deployment/web VERSION=2.0

# Rollout status:
kubectl rollout status deployment/web

# History:
kubectl rollout history deployment/web
kubectl rollout history deployment/web --revision=3

# Pause/Resume:
kubectl rollout pause deployment/web
kubectl rollout resume deployment/web

# Rollback:
kubectl rollout undo deployment/web
kubectl rollout undo deployment/web --to-revision=1

# Restart (recreate all Pods without changing template):
kubectl rollout restart deployment/web
```

---

### 🔴 OpenShift Difference

| Aspect | Kubernetes (Deployment) | OpenShift |
|--------|------------------------|-----------|
| Preferred workload | `Deployment` (apps/v1) | Same — `Deployment` (apps/v1) |
| Legacy alternative | N/A | **DeploymentConfig** (deprecated — don't use for new apps) |
| Triggers | Manual (`kubectl set image`, `kubectl apply`) | DeploymentConfigs had **auto-triggers** (ImageStream change → auto-deploy). Deployments don't. |
| Image management | You specify full image:tag | **ImageStreams** can track tags + trigger updates (with DC, not Deployment) |
| Rollout commands | `kubectl rollout` | `oc rollout` (identical) + `oc rollback` for DCs |
| Strategy types | RollingUpdate, Recreate | Same + DCs had "Custom" strategy (run your own deploy script) |
| Lifecycle hooks | Pod-level postStart/preStop | DCs had **deployment hooks** (pre, mid, post) — separate concept |
| Canary/Blue-Green | Manual (multiple Deployments + Service selector) | Same — or use **OpenShift Routes** with traffic splitting |

```bash
# OpenShift: check if using Deployment or DeploymentConfig:
oc get deployment,deploymentconfig -n myapp

# DC rollback (legacy):
oc rollback dc/web --to-version=2

# Deployment rollback (same as K8s):
oc rollout undo deployment/web
```

> **Interview tip:** "OpenShift used to have DeploymentConfigs with auto-triggers from ImageStreams, but they're deprecated now. Modern OpenShift uses standard Kubernetes Deployments. The rollout commands are identical. For canary/blue-green, OpenShift Routes can split traffic by weight — something you'd need an Ingress controller or service mesh for in vanilla K8s."

---

## 💡 Interview Power Answers — Deployments

**Q: "What is a Deployment and how does it relate to ReplicaSets?"**
> "A Deployment is a controller for stateless long-running workloads. It creates ReplicaSets, which create Pods. Each Pod-template change creates a new ReplicaSet (new revision). The old ReplicaSet is kept (scaled to 0) for rollback. The chain is: Deployment → ReplicaSet → Pods."

**Q: "Explain a rolling update."**
> "When you change the Pod template (like updating an image), the Deployment creates a new ReplicaSet and gradually scales it up while scaling the old one down. `maxSurge` controls how many extra Pods can exist above desired count, `maxUnavailable` controls how many can be down. With maxSurge:1 and maxUnavailable:0, you always have full capacity — one new Pod comes up before an old one goes down."

**Q: "How do you rollback a failed deployment?"**
> "`kubectl rollout undo deployment/web` restores the previous revision's Pod template. For a specific revision: `--to-revision=1`. Rollback creates a new revision number — it doesn't literally go back in time. The key protection: with `maxUnavailable: 0`, a bad image only affects the surge Pod — existing Pods keep running."

**Q: "What's the difference between Recreate and RollingUpdate strategy?"**
> "RollingUpdate replaces Pods incrementally — zero downtime. Recreate kills all old Pods before starting new ones — causes downtime but guarantees only one version runs at a time. Use Recreate when your app can't handle two versions simultaneously (like database schema changes)."

**Q: "What triggers a new rollout?"**
> "Only changes to `spec.template` trigger a new rollout and create a new ReplicaSet. Scaling (`spec.replicas`), changing metadata labels on the Deployment, or pausing — none of these trigger rollouts. Common triggers: image change, env var change, resource limit change, any Pod-template modification."

**Q: "What does progressDeadlineSeconds do?"**
> "It's a timeout. If the rollout doesn't make progress within that time, Kubernetes sets `Progressing: False` with reason `ProgressDeadlineExceeded`. But it does NOT auto-rollback — the controller keeps trying. You must manually rollback with `kubectl rollout undo`."


---

---

# 🗄️ SECTION 18: StatefulSets

---

## 18.1 What Is a StatefulSet? ✅ MUST HAVE

A StatefulSet manages Pods that need **stable identity** — unlike Deployments where Pods are interchangeable.

```
┌──────────────────────────────────────────────────────────────┐
│         StatefulSet vs Deployment                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Deployment:                                                 │
│    web-86b6cb7b94-abc  ← random suffix, interchangeable     │
│    web-86b6cb7b94-def  ← any Pod can replace any other      │
│    web-86b6cb7b94-ghi                                        │
│                                                              │
│  StatefulSet:                                                │
│    database-0  ← stable name, ordinal 0, own PVC, own DNS   │
│    database-1  ← stable name, ordinal 1, own PVC, own DNS   │
│    database-2  ← stable name, ordinal 2, own PVC, own DNS   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Three Pillars of StatefulSet Identity

| Property | What It Means |
|----------|--------------|
| **Stable Pod name** | `database-0` keeps name after recreation (new UID, same name) |
| **Stable DNS hostname** | `database-0.database.sts-lab.svc.cluster.local` |
| **Stable persistent storage** | Ordinal 0 always reattaches to PVC `data-database-0` |

---

## 18.2 When to Use StatefulSet vs Deployment ✅ MUST HAVE

| Use StatefulSet When | Use Deployment When |
|---------------------|---------------------|
| Stable ordinal names needed (db-0, db-1) | Pods are interchangeable |
| Per-replica persistent storage | Shared or no storage |
| Ordered startup/shutdown matters | Order doesn't matter |
| Predictable per-Pod DNS | Single Service IP is fine |
| Databases, message brokers, cluster members | Web apps, APIs, stateless workers |

**Examples:** PostgreSQL, MongoDB, Kafka, Zookeeper, Elasticsearch, Redis Cluster

---

## 18.3 StatefulSet Architecture ✅ MUST HAVE

```
┌────────────────────────────────────────────────────────────┐
│          StatefulSet Components                             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  StatefulSet (database)                                    │
│    │                                                       │
│    ├── Headless Service (database, clusterIP: None)        │
│    │     → Provides per-Pod DNS records                    │
│    │     → NO load-balanced VIP                            │
│    │                                                       │
│    ├── Pod: database-0                                     │
│    │     └── PVC: data-database-0 → PV (1Gi)             │
│    │                                                       │
│    ├── Pod: database-1                                     │
│    │     └── PVC: data-database-1 → PV (1Gi)             │
│    │                                                       │
│    └── Pod: database-2                                     │
│          └── PVC: data-database-2 → PV (1Gi)             │
│                                                            │
│  DNS:                                                      │
│    database-0.database.sts-lab.svc.cluster.local           │
│    database-1.database.sts-lab.svc.cluster.local           │
│    database-2.database.sts-lab.svc.cluster.local           │
└────────────────────────────────────────────────────────────┘
```

**No ReplicaSet in between!** StatefulSet → Pods (directly).

---

## 18.4 StatefulSet YAML ✅ MUST HAVE

### Headless Service (required)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
  namespace: sts-lab
spec:
  clusterIP: None          # ← THIS makes it headless
  selector:
    app: database
  ports:
  - port: 80
    name: web
```

### StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
  namespace: sts-lab
spec:
  serviceName: database        # must match headless Service name
  replicas: 3
  selector:
    matchLabels:
      app: database            # immutable after creation
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: nginx
        image: nginx:1.27-alpine
        volumeMounts:
        - name: data
          mountPath: /var/lib/data
  volumeClaimTemplates:        # creates 1 PVC per Pod
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: local-path
      resources:
        requests:
          storage: 1Gi
```

### Key Fields

| Field | Purpose |
|-------|---------|
| `serviceName` | Links to headless Service for DNS identity |
| `replicas` | Number of ordered Pods |
| `selector` | Must match template labels (immutable) |
| `volumeClaimTemplates` | Creates per-Pod PVCs (optional but common) |
| `podManagementPolicy` | `OrderedReady` (default) or `Parallel` |
| `updateStrategy.type` | `RollingUpdate` (default) or `OnDelete` |

---

## 18.5 Ordered Lifecycle (OrderedReady) ✅ MUST HAVE

Default `podManagementPolicy: OrderedReady`:

```
Scale UP:    0 → Ready → 1 → Ready → 2 → Ready
Scale DOWN:  2 deleted → 1 deleted → 0 deleted
Updates:     2 updated → 1 updated → 0 updated (reverse)
```

| Operation | Order |
|-----------|-------|
| Scale up | 0, 1, 2... (each must be Ready before next starts) |
| Scale down | Highest ordinal first (...2, 1, 0) |
| Rolling update | Highest ordinal first (reverse order) |

**Parallel policy** (`podManagementPolicy: Parallel`):
- Scale up/down all at once (no waiting)
- Still maintains stable identity + per-Pod storage
- Updates still follow `updateStrategy`

---

## 18.6 Per-Replica Storage (volumeClaimTemplates) ✅ MUST HAVE

```
PVC naming: <claimTemplate-name>-<statefulset-name>-<ordinal>

  data-database-0  → binds to PV → /var/lib/data on Pod 0
  data-database-1  → binds to PV → /var/lib/data on Pod 1
  data-database-2  → binds to PV → /var/lib/data on Pod 2
```

**Key behaviors:**
- Pod recreated → reattaches to same PVC → data survives
- Scale down → PVCs are **NOT deleted** (by default)
- Scale back up → same ordinal rebinds to existing PVC
- `volumeClaimTemplates` cannot be updated on existing StatefulSet

### PVC Retention Policy

```yaml
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain     # when StatefulSet deleted
    whenScaled: Retain      # when scaled down
```

| Policy | Retain (default) | Delete |
|--------|-----------------|--------|
| `whenDeleted` | PVCs survive StatefulSet deletion | PVCs deleted with StatefulSet |
| `whenScaled` | PVCs kept for removed ordinals | PVCs deleted for scaled-down Pods |

---

## 18.7 DNS & Network Identity ✅ MUST HAVE

**FQDN format:**
```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

**Examples:**
```
database-0.database.sts-lab.svc.cluster.local
database-1.database.sts-lab.svc.cluster.local

# Short (within same namespace):
database-0.database
database-1.database
```

**Headless Service:**
- Returns individual Pod IPs (no load-balanced VIP)
- Each Pod gets its own A record
- Pods can discover peers via DNS

```bash
# Resolve a specific Pod:
nslookup database-0.database
# → 10.244.1.25

# Resolve the Service name (returns ALL Pod IPs):
nslookup database
# → 10.244.1.25, 10.244.2.30, 10.244.1.26
```

> **Key:** Recreated Pod gets same name + same DNS record — but may get new IP. Use DNS names, not IPs!

---

## 18.8 Scaling ⚡ GOOD TO KNOW

```bash
# Scale up:
kubectl scale statefulset database --replicas=4
# → Creates database-3 (after 0,1,2 are Ready)

# Scale down:
kubectl scale statefulset database --replicas=2
# → Deletes database-3, then database-2
# → PVCs data-database-2 and data-database-3 REMAIN

# Scale back to 4 → ordinals 2,3 rebind to existing PVCs
```

---

## 18.9 Updates (RollingUpdate & OnDelete) ⚡ GOOD TO KNOW

### RollingUpdate (Default)

```bash
kubectl set image statefulset/database nginx=nginx:1.28-alpine
kubectl rollout status statefulset/database
```

Updates in **reverse ordinal order**: 2 → 1 → 0

### Partition (Canary Updates)

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2    # only ordinals >= 2 get updated
```

- Ordinals below partition keep old template (even if deleted and recreated!)
- Lower partition gradually to roll forward

### OnDelete

- Changes template in API but doesn't update Pods
- YOU manually delete each Pod to trigger update
- Useful for manual control over stateful app upgrades

---

## 18.10 Deletion — Order Matters! ⚡ GOOD TO KNOW

```bash
# Ordered shutdown (recommended for stateful apps):
kubectl scale statefulset database --replicas=0
kubectl wait --for=delete pod -l app=database --timeout=180s
kubectl delete statefulset database

# Direct delete (NOT guaranteed ordered):
kubectl delete statefulset database
# Pods terminate but NOT necessarily in reverse order!
```

> **Rule:** Scale to 0 first if your app needs ordered shutdown (databases!).

---

## 18.11 Troubleshooting ✅ MUST HAVE

| Symptom | Cause | Fix |
|---------|-------|-----|
| Only Pod 0 appears | OrderedReady — waiting for Pod 0 to be Ready | `kubectl describe pod database-0` |
| Next ordinal never starts | Previous Pod stuck (image pull, probe fail) | Fix the lower-ordinal Pod first |
| PVCs remain after deletion | Default Retain policy | Delete manually or set `whenDeleted: Delete` |
| DNS name doesn't resolve | Missing headless Service / serviceName mismatch / Pod not Ready | Check Service selector + serviceName |
| Pod Pending | PVC unbound (no StorageClass / no PV available) | `kubectl describe pod` Events + `kubectl get pvc` |
| Template change ignored | `OnDelete` strategy | Manually delete Pod to trigger update |
| Selector rejected | Labels mismatch | Fix before first apply (immutable) |

---

### 🔴 OpenShift Difference

| Aspect | Kubernetes | OpenShift |
|--------|-----------|-----------|
| StatefulSet | Same (`apps/v1`) | Identical |
| Storage | You provision StorageClass | **OCS/ODF (OpenShift Data Foundation)** often pre-configured |
| Dynamic provisioning | Depends on cluster setup | ODF provides RWO, RWX, RBD, CephFS out-of-box |
| Monitoring | Manual Prometheus setup | Built-in monitoring for StatefulSet metrics |
| Storage console | CLI only | Web Console shows PVC bindings visually |
| Operator-managed DBs | You write StatefulSets manually | **Operators for databases** (CrunchyData PostgreSQL, Strimzi Kafka) — they create StatefulSets for you |
| Route to specific Pod | Not built-in (need headless + client logic) | Can create **Route** pointing to specific Pod via Service subset |

```bash
# OpenShift: check storage classes (usually pre-configured):
oc get storageclass
# NAME                          PROVISIONER
# ocs-storagecluster-ceph-rbd   openshift-storage.rbd.csi.ceph.com
# ocs-storagecluster-cephfs     openshift-storage.cephfs.csi.ceph.com

# Operator-managed database (example: CrunchyData):
oc get postgresclusters
# The operator handles StatefulSet creation, scaling, backups
```

> **Interview tip:** "StatefulSets work identically in OpenShift and vanilla K8s. The difference is operational: OpenShift usually has ODF providing storage out-of-box, and you'd use database Operators (CrunchyData, Strimzi) that create and manage StatefulSets for you rather than writing them manually. The Operator handles backups, failover, scaling — you just declare the desired state in the Operator's CRD."

---

## 💡 Interview Power Answers — StatefulSets

**Q: "What is a StatefulSet and when do you use it?"**
> "A StatefulSet manages Pods that need stable identity: predictable names (db-0, db-1), stable DNS hostnames via a headless Service, and per-replica persistent storage via volumeClaimTemplates. Use it for databases, message brokers, and clustered apps where Pods aren't interchangeable. Use a Deployment when replicas are fungible."

**Q: "How does StatefulSet identity survive Pod recreation?"**
> "When database-0 is deleted, the controller recreates a Pod with the same name and ordinal. It gets a new UID and possibly new IP, but the same DNS name resolves to it, and it reattaches to the same PVC (data-database-0). So data and network identity persist — only the Pod object is new."

**Q: "Explain ordered lifecycle in StatefulSets."**
> "With the default OrderedReady policy: scale-up creates Pods 0, 1, 2 in order — each must be Ready before the next starts. Scale-down removes from highest ordinal first. Rolling updates also go in reverse (2, 1, 0). This matters for databases where the primary must be up before replicas join."

**Q: "What happens to PVCs when you scale down or delete a StatefulSet?"**
> "By default, PVCs are retained — never auto-deleted. Scale down from 3 to 2: Pod database-2 is gone but PVC data-database-2 remains. Scale back to 3: ordinal 2 rebinds to the same PVC. Delete the StatefulSet: Pods go away but all PVCs stay. This protects data but means manual cleanup."

**Q: "What's a headless Service and why does StatefulSet need one?"**
> "A headless Service has `clusterIP: None`. Instead of providing one VIP that load-balances, it creates individual DNS records for each Pod. StatefulSet uses it (via `serviceName`) to give each Pod a stable FQDN like `database-0.database.ns.svc.cluster.local`. Without it, Pods can't be addressed individually."

**Q: "How do you do a canary update with StatefulSets?"**
> "Use `partition` in the rolling update strategy. Set `partition: 2` and only ordinals >= 2 get the new template. Test the canary, then lower partition to 1, then 0 to roll out fully. Ordinals below partition keep the old template even if deleted."
