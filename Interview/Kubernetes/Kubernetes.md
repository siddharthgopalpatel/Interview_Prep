# Kubernetes — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 12, 25, 37, 43, 53

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 12: Kubernetes & EKS

---

### Q: What are some common Kubernetes troubleshooting scenarios you have faced in production?

**Project Reference:** P3 (Kubernetes Platform — EKS, self-managed, OpenShift)

**Answer:**

> "Top scenarios I've dealt with:
>
> 1. **Pod stuck in CrashLoopBackOff** — App failing on startup. Check: `kubectl logs <pod> --previous`. Common causes: missing config/secret, wrong image tag, DB not reachable, OOMKilled (check `kubectl describe pod` → Last State → Reason).
>
> 2. **Pod stuck in Pending** — No node can schedule it. Check: `kubectl describe pod` → Events. Causes: insufficient CPU/memory on nodes (Karpenter not scaling?), PVC not bound (storage class issue), node affinity/taint mismatch.
>
> 3. **OOMKilled** — Container hit memory limit. Fix: increase limits OR fix the memory leak. Check: `kubectl describe pod` → `OOMKilled`, then `dmesg | grep -i oom` on the node.
>
> 4. **Service unreachable** — Pod is running but can't be reached. Check: selector mismatch between Service and Pod labels, endpoints (`kubectl get endpoints`), NetworkPolicy blocking traffic, kube-proxy iptables rules not propagated.
>
> 5. **Node NotReady** — kubelet stopped or disk pressure. Check: `kubectl describe node` → Conditions. SSH to node: `systemctl status kubelet`, `journalctl -u kubelet`, `df -h`.
>
> 6. **ImagePullBackOff** — Can't pull image. Causes: wrong image name, ECR auth expired (`aws ecr get-login-token`), registry unreachable, image doesn't exist.
>
> **My approach:** Always start with `kubectl describe` → Events section tells you 80% of the story."

---

### Q: How can you provide EKS pods access to DynamoDB and S3 without using static access/secret keys?

**Project Reference:** P4 (Landing Zone — OIDC), P3 (EKS — IRSA)

**Answer:**

> "**IRSA — IAM Roles for Service Accounts.**
>
> How it works:
> 1. Create an IAM role with the permissions needed (DynamoDB read/write, S3 access)
> 2. Create a trust policy that trusts the EKS OIDC provider — scoped to a specific Kubernetes ServiceAccount in a specific namespace
> 3. Annotate the K8s ServiceAccount: `eks.amazonaws.com/role-arn: arn:aws:iam::123:role/my-role`
> 4. Pod uses that ServiceAccount → EKS injects a web identity token → AWS SDK exchanges it for temporary IAM credentials automatically
>
> **Result:** Pod gets short-lived credentials (15 min–12 hr). No static keys. No secrets to rotate. If pod is compromised, credentials expire automatically.
>
> **Key security points:**
> - Scoped to namespace + ServiceAccount (not cluster-wide)
> - Least privilege — each pod gets ONLY what it needs
> - Auditable via CloudTrail — you see which pod assumed which role
>
> We use this for everything: ECR pull, S3 access, DynamoDB, Secrets Manager. Zero static credentials in our cluster."

---

### Q: What version of EKS are you currently on?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "We're on **EKS 1.31** currently, with an upgrade to 1.32 planned next quarter. We stay one version behind latest to let the community shake out bugs. EKS supports N-3 versions, so we have runway.
>
> We upgrade every 3-4 months to stay within support. Each upgrade follows our tested runbook — control plane first, then managed node groups, then self-managed components (CoreDNS, kube-proxy, VPC CNI add-ons)."

**Key fact:** Don't say "latest" — shows you don't track versions. Give a specific version and show you have an upgrade cadence.

---

### Q: How would you plan and carry out a migration of an EKS cluster from version 1.33 to 1.34?

**Project Reference:** P3 (Kubernetes Platform — upgrades)

**Answer:**

> "It's a sequential, tested process:
>
> **Pre-upgrade:**
> 1. Read release notes — check deprecated APIs (`kubectl convert` or `kubent` to find deprecated resources in our manifests)
> 2. Check add-on compatibility — VPC CNI, CoreDNS, kube-proxy, Karpenter, ArgoCD, Istio versions against K8s 1.34 support matrix
> 3. Test in non-prod first — upgrade Dev cluster, run full integration test suite
> 4. Backup — etcd snapshot (self-managed) or just ensure Terraform state is current (EKS managed)
>
> **Upgrade sequence:**
> 1. **Control plane** — `aws eks update-cluster-version --name cluster --kubernetes-version 1.34`. Takes ~20 min. Zero downtime (EKS manages it).
> 2. **Add-ons** — Update CoreDNS, kube-proxy, VPC CNI to compatible versions.
> 3. **Node groups** — Rolling update. Launch new 1.34 nodes → drain old nodes → terminate. Pods migrate gracefully via PodDisruptionBudgets.
> 4. **Validate** — All pods running, services reachable, monitoring healthy, no deprecated API warnings.
>
> **Key risks:** Deprecated APIs (moved to GA in new version), webhook compatibility, CNI plugin version mismatch. That's why we test in Dev first."

---

### Q: Which tool would you use to deploy your Docker image onto an EKS cluster?

**Project Reference:** P1 (DevSecOps Pipeline — ArgoCD)

**Answer:**

> "**ArgoCD** — GitOps-based. CI pipeline (Jenkins) pushes the image to ECR, updates the Helm values file in the GitOps repo, ArgoCD auto-syncs it to EKS.
>
> For the Helm chart itself: **Helm** packages the Kubernetes manifests with templating and version control."

**Note:** Short answer as asked. If they probe deeper, reference the existing "How do you deploy a service into Kubernetes" answer.

---

### Q: Have you worked with cloud-native tools like Cert-Manager or Nginx Ingress?

**Project Reference:** P3 (Kubernetes Platform), P10 (Ingress Controllers)

**Answer:**

> "Yes, both:
>
> **Cert-Manager** — Automates TLS certificate lifecycle in Kubernetes. We use it with Let's Encrypt (for non-prod) and AWS ACM PCA (for prod). It creates Certificate resources, requests from the CA, stores in K8s Secrets, and auto-renews before expiry. No manual cert rotation.
>
> **Nginx Ingress Controller** — Our L7 ingress in self-managed clusters. Handles path-based routing, TLS termination, rate limiting, custom headers. On EKS, we use AWS ALB Ingress Controller instead (native ALB integration), but Nginx is used in our self-managed kubeadm clusters.
>
> Both are deployed via Helm with ArgoCD managing their lifecycle."

---

### Q: If release notes state that your Nginx ingress controller is not compatible with the new Kubernetes version, what would you do?

**Project Reference:** P3 (Kubernetes — upgrade planning)

**Answer:**

> "Three options in priority order:
>
> 1. **Upgrade Nginx Ingress first** — Check if a newer version of Nginx Ingress IS compatible with K8s 1.34. Usually, the ingress controller releases a compatible version within weeks. Upgrade Nginx Ingress → then upgrade K8s. This is the normal path.
>
> 2. **If no compatible version exists yet** — Wait. Don't upgrade K8s until ingress controller supports it. Ingress is critical path — all traffic flows through it. Breaking it = full outage.
>
> 3. **If wait isn't an option (security patch urgency)** — Evaluate switching to AWS ALB Ingress Controller (aws-load-balancer-controller) which is maintained by AWS and always compatible with latest EKS. Migrate ingress resources, test, then upgrade K8s.
>
> **Key principle:** Never upgrade K8s if a critical cluster component (ingress, CNI, CSI driver) isn't compatible. Test the full stack in Dev first. The upgrade is only safe when ALL components work together."

---

### Q: What happens when a user clicks on a public DNS (abc.com) until the request reaches the pod in your EKS cluster?

**Project Reference:** P2 (3-Tier), P3 (Kubernetes networking)

**Answer:**

> "Full flow:
>
> 1. **DNS resolution** — Browser queries DNS. Route53 returns the ALB's IP address (or CloudFront if CDN is in front).
>
> 2. **TLS handshake** — Browser establishes HTTPS with ALB. SSL terminates at ALB (ACM certificate).
>
> 3. **ALB routing** — ALB checks Ingress rules (path/host-based). Forwards to the correct Target Group.
>
> 4. **Target Group → Node** — ALB sends traffic to a NodePort on one of the EKS worker nodes (or directly to pod IP if using IP-mode target groups with VPC CNI).
>
> 5. **kube-proxy / iptables** — If NodePort mode: kube-proxy's iptables rules DNAT the request to the actual pod IP (could be on same node or different node).
>
> 6. **Pod receives request** — Traffic enters the pod's network namespace, hits the container port, application processes it.
>
> **With Istio (P6):** Between step 5 and 6, the Envoy sidecar intercepts traffic (iptables redirect in pod's network namespace), applies mTLS, AuthorizationPolicy, then forwards to the app container on localhost.
>
> **IP-mode (what we use):** ALB sends directly to pod IP (VPC CNI assigns routable IPs to pods). Skips NodePort/kube-proxy. Lower latency, better load distribution."

---

### Q: What parameters, besides CPU, memory, and disk, would you alert on to monitor a Kubernetes cluster?

**Project Reference:** P3 (Observability stack)

**Answer:**

> "Beyond the obvious CPU/memory/disk:
>
> 1. **Pod restart count** — Pods restarting = CrashLoopBackOff, OOMKill, or liveness probe failure. Alert if restarts > 3 in 5 minutes.
>
> 2. **Pending pods** — Pods stuck in Pending = scheduling failure (resource exhaustion, node issues). Should be 0 in production.
>
> 3. **Node conditions** — DiskPressure, MemoryPressure, PIDPressure, NetworkUnavailable. Any True = alert.
>
> 4. **API server latency** — If etcd or API server is slow (>500ms), the whole cluster suffers. Alert on apiserver_request_duration_seconds.
>
> 5. **Certificate expiry** — Kubelet, API server, etcd certs. Alert 30 days before expiry (cert-manager handles this for app certs).
>
> 6. **Endpoint readiness** — `kube_endpoint_address_not_ready` — services with no ready endpoints = user-facing outage.
>
> 7. **HPA at max** — If HPA is at max replicas for >10 minutes, it can't scale further. Need bigger nodes or higher max.
>
> 8. **PVC usage** — Persistent volumes approaching capacity (>85%). Especially critical for stateful workloads.
>
> 9. **Network errors** — Pod network drops, DNS resolution failures (CoreDNS errors).
>
> 10. **Image pull failures** — ECR token expired, rate limiting."

---

### Q: What steps would you take to secure a default Kubernetes cluster?

**Project Reference:** P3 (Kubernetes security), P1 (DevSecOps)

**Answer:**

> "Default K8s is INSECURE out of the box. Here's what I harden:
>
> 1. **RBAC** — Disable anonymous access. Create specific Roles/ClusterRoles per team. No one gets cluster-admin except break-glass.
>
> 2. **Pod Security Standards** — Enforce `restricted` profile. No root containers, no privilege escalation, no hostNetwork, no hostPID.
>
> 3. **Network Policies** — Default-deny all traffic. Explicitly allow only required paths. Ingress → frontend → backend → database. Nothing else.
>
> 4. **Secrets encryption** — Enable encryption at rest for etcd (KMS provider on EKS). Secrets are encrypted, not just base64.
>
> 5. **Image policies** — Kyverno: only images from trusted ECR registry, only signed images (Cosign). Block `latest` tag, block Docker Hub in prod.
>
> 6. **API server access** — Private endpoint (no public access). Access only via VPN/bastion. Audit logging enabled → CloudTrail.
>
> 7. **Node security** — Minimal AMI (Bottlerocket), no SSH to nodes in prod, IMDSv2 enforced, regular patching.
>
> 8. **Service accounts** — `automountServiceAccountToken: false` by default. Only mount when needed.
>
> 9. **Resource limits** — LimitRange per namespace. Prevent one pod from starving others.
>
> 10. **Runtime security** — Falco for anomaly detection, read-only root filesystem on all containers."

---

### Q: What are the security constraints called in the Kubernetes world?

**Project Reference:** P3 (Kubernetes security)

**Answer:**

> "They've evolved over time:
>
> - **Pod Security Policies (PSP)** — Deprecated since K8s 1.21, removed in 1.25. Was the original way to restrict what pods can do.
>
> - **Pod Security Standards (PSS) + Pod Security Admission (PSA)** — The replacement. Three levels:
>   - `privileged` — no restrictions (only for system components)
>   - `baseline` — prevents known privilege escalations (no hostNetwork, no hostPID)
>   - `restricted` — full hardening (non-root, drop all capabilities, read-only rootfs, no privilege escalation)
>
> - **External policy engines (what we use):**
>   - **Kyverno** — Kubernetes-native policy engine. We write ClusterPolicies that enforce: no root, require resource limits, require labels, only signed images, block NodePort services.
>   - **OPA/Gatekeeper** — Alternative to Kyverno. Rego-based policies.
>
> We use PSA (baseline at namespace level) + Kyverno (for granular enforcement beyond what PSA offers). Belt and suspenders."

---

---
---

# ~~SECTION 13: Terraform Advanced~~ → Moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)

---
---

# SECTION 25: Kubernetes Fundamentals

---

### Q: Why would you use Kubernetes, what are its advantages, and what are its different components?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "**Why Kubernetes:**
> You have 15+ microservices in containers. You need them to self-heal, scale, deploy without downtime, and communicate securely. Doing this manually on VMs is impossible at scale. Kubernetes automates container orchestration.
>
> **Key advantages:**
> 1. **Self-healing** — Pod crashes → K8s restarts it. Node dies → pods reschedule to healthy nodes.
> 2. **Auto-scaling** — HPA scales pods on CPU/memory/custom metrics. Karpenter scales nodes.
> 3. **Rolling updates + rollback** — Zero-downtime deployments. One command rollback.
> 4. **Service discovery** — Pods find each other via DNS (CoreDNS). No hardcoded IPs.
> 5. **Declarative** — Describe desired state in YAML. K8s converges to it.
>
> **Components:**
>
> | Control Plane | Worker Node |
> |---|---|
> | API Server (entry point for all commands) | kubelet (manages pods on node) |
> | etcd (key-value store — cluster state) | kube-proxy (network routing/iptables) |
> | Scheduler (assigns pods to nodes) | Container runtime (containerd) |
> | Controller Manager (ensures desired state) | Pods (your application) |
>
> Plus: CoreDNS (service discovery), CNI plugin (Calico/VPC CNI — pod networking), Ingress Controller (external traffic entry)."

---

### Q: What was the size of the infrastructure you managed using Kubernetes?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "Across our three Kubernetes environments:
>
> - **EKS (primary production):** 3 clusters (dev/staging/prod), 15+ microservices, ~50-80 pods in production, 10-20 worker nodes (Karpenter scales dynamically), multi-AZ across 3 availability zones.
>
> - **Self-managed (kubeadm):** 1 cluster, 5 masters (HA etcd), 15 workers. Used for specific workloads requiring more control.
>
> - **OpenShift:** 1 cluster, managed by client. We deploy workloads to it.
>
> **Total:** ~500+ pods across environments, serving the contact center platform with 99.95% uptime SLA. Supporting 15+ microservices with full observability stack (Prometheus, Grafana, Jaeger, EFK).
>
> The Kubernetes platform supports the same Verizon contact center infrastructure that previously ran on 500+ VMs — we're progressively containerizing workloads."

---

### Q: What are the three types of services in Kubernetes?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "Four actually (3 main + 1 headless):
>
> | Type | What It Does | Use Case |
> |---|---|---|
> | **ClusterIP** (default) | Internal-only IP. Reachable only within cluster | Service-to-service communication (backend → database) |
> | **NodePort** | Exposes on each node's IP at a static port (30000-32767) | Development/testing, legacy access |
> | **LoadBalancer** | Provisions external cloud load balancer (ALB/NLB on AWS) | Exposing service to internet |
> | **Headless** (`clusterIP: None`) | No cluster IP. Returns pod IPs directly via DNS | StatefulSets (each pod individually addressable — Kafka, databases) |
>
> **What we use:**
> - **ClusterIP** — 90% of services. Internal communication.
> - **LoadBalancer** — NOT used directly. Instead, we use Ingress Controller (ALB) which is more flexible (path/host routing, SSL, WAF).
> - **NodePort** — Avoided in production (security risk — exposes ports on all nodes).
>
> **Key point:** In production, we expose services via **Ingress** (ALB controller) not raw LoadBalancer type. Ingress gives you path-based routing, SSL termination, WAF — one ALB for multiple services instead of one NLB per service."

---
---

# SECTION 37: Kubernetes Manifests

---

### Q: Can you explain the contents of deployment.yaml and service.yaml, and why they are different?

**Project Reference:** P3 (Kubernetes Platform), P1 (Helm deployments)

**Answer:**

> "They serve completely different purposes:
>
> **deployment.yaml — WHAT to run:**
> ```yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   name: payment-service
> spec:
>   replicas: 3                    # How many pods
>   selector:
>     matchLabels:
>       app: payment               # Which pods this deployment manages
>   template:
>     spec:
>       containers:
>       - name: payment
>         image: ecr.aws/payment:v2.1   # Container image
>         ports:
>         - containerPort: 8080     # Port app listens on
>         resources:
>           requests:
>             cpu: 100m
>             memory: 256Mi
>         livenessProbe:            # Is it alive?
>           httpGet:
>             path: /health
>             port: 8080
> ```
> **Purpose:** Defines the application — what image, how many replicas, resource limits, health checks, update strategy.
>
> **service.yaml — HOW to reach it:**
> ```yaml
> apiVersion: v1
> kind: Service
> metadata:
>   name: payment-service
> spec:
>   selector:
>     app: payment                  # Routes to pods with this label
>   ports:
>   - port: 80                     # Service port (what callers use)
>     targetPort: 8080             # Pod port (where traffic goes)
>   type: ClusterIP                # Internal only
> ```
> **Purpose:** Provides stable networking — a fixed DNS name and IP that routes traffic to healthy pods regardless of which pod IPs change.
>
> **Why separate:**
> - Deployment = compute concern (what runs, how it scales)
> - Service = networking concern (how traffic reaches it)
> - Pods are ephemeral (IPs change). Service provides a stable endpoint.
> - You can update a Deployment (new image) without touching the Service. Networking stays stable."

---
---

# SECTION 43: Kubernetes Core Concepts

---

### Q: What is the difference between a Pod and a Container?

**Project Reference:** P3 (Kubernetes Platform)

**Answer:**

> "**Container** = a single running process in an isolated environment (namespaces + cgroups + rootfs). It's a Docker/containerd unit.
>
> **Pod** = Kubernetes' smallest deployable unit. A wrapper around one or MORE containers that share:
> - Same network namespace (same IP, same localhost, same ports)
> - Same storage volumes
> - Same lifecycle (start together, die together)
>
> **Why the distinction matters:**
>
> | Aspect | Container | Pod |
> |---|---|---|
> | **Scope** | Single process | Group of related containers |
> | **Networking** | Own network stack | SHARED network (containers in same pod talk via localhost) |
> | **Scheduling** | N/A | Scheduled together on same node |
> | **Use case** | One app process | Main app + sidecar (Envoy, log collector) |
>
> **Real example (P6 — Istio):**
> One Pod contains:
> - Container 1: `payment-service` (our app — port 8080)
> - Container 2: `envoy-proxy` (Istio sidecar — intercepts traffic)
>
> They share the same network namespace — Envoy can intercept traffic on `localhost` because they're in the same Pod. If they were separate Pods, this wouldn't work.
>
> **Rule:** One container per Pod (90% of cases). Multiple containers only for sidecar patterns (logging, proxying, init containers)."

---

### Q: What is the difference between a ConfigMap and a Secret?

**Project Reference:** P3 (Kubernetes), P1 (DevSecOps)

**Answer:**

> | Aspect | ConfigMap | Secret |
> |---|---|---|
> | **Purpose** | Non-sensitive configuration (app settings, feature flags) | Sensitive data (passwords, tokens, certificates) |
> | **Storage** | Plain text in etcd | Base64-encoded in etcd (+ encrypted at rest with KMS on EKS) |
> | **Access** | No special restrictions by default | Can restrict via RBAC (limit who can `get secrets`) |
> | **Size limit** | 1 MB | 1 MB |
> | **Mounted as** | Env vars or volume files | Env vars or volume files |
>
> **Example:**
> ```yaml
> # ConfigMap — non-sensitive
> apiVersion: v1
> kind: ConfigMap
> data:
>   LOG_LEVEL: \"info\"
>   MAX_RETRIES: \"3\"
>   APP_MODE: \"production\"
>
> # Secret — sensitive
> apiVersion: v1
> kind: Secret
> type: Opaque
> data:
>   DB_PASSWORD: cGFzc3dvcmQxMjM=    # base64 encoded
>   API_KEY: c2VjcmV0a2V5MTIz
> ```
>
> **Key misconception:** base64 is NOT encryption. Anyone who can read the Secret can decode it. Real security comes from:
> - RBAC (restrict who can `kubectl get secrets`)
> - Encryption at rest (KMS envelope encryption in EKS)
> - External Secrets Operator (secrets fetched from Secrets Manager, not stored in Git)"

---

### Q: What are Persistent Volumes (PV) and Persistent Volume Claims (PVC)?

**Project Reference:** P3 (Kubernetes — stateful workloads)

**Answer:**

> "**PV** = the actual storage resource (the disk). Provisioned by admin or dynamically by StorageClass.
>
> **PVC** = a request for storage by a pod. 'I need 10Gi of SSD storage.'
>
> **Analogy:** PV is a house. PVC is a rental application. Pod is the tenant.
>
> ```yaml
> # PVC — what the pod asks for
> apiVersion: v1
> kind: PersistentVolumeClaim
> metadata:
>   name: data-volume
> spec:
>   accessModes: [ReadWriteOnce]
>   resources:
>     requests:
>       storage: 10Gi
>   storageClassName: gp3
>
> # Pod uses the PVC
> spec:
>   volumes:
>   - name: data
>     persistentVolumeClaim:
>       claimName: data-volume
>   containers:
>   - volumeMounts:
>     - mountPath: /data
>       name: data
> ```
>
> **How it works on EKS:**
> 1. Pod requests PVC (10Gi, gp3)
> 2. StorageClass (EBS CSI driver) dynamically provisions an EBS gp3 volume
> 3. EBS volume attached to the node where pod is scheduled
> 4. Mounted into the container at `/data`
> 5. Pod deleted → PVC retained (data persists). New pod can claim same PVC.
>
> **Key facts:**
> - `ReadWriteOnce` = only one node can mount it (EBS limitation)
> - `ReadWriteMany` = multiple nodes (need EFS, not EBS)
> - `persistentVolumeReclaimPolicy: Retain` = data survives PVC deletion (safety for databases)"

---

### Q: How do you perform scaling over node groups in Kubernetes/EKS?

**Project Reference:** P3 (Kubernetes — Karpenter), P7 (Cost Optimization)

**Answer:**

> "Two approaches — we use Karpenter:
>
> **1. Managed Node Groups + Cluster Autoscaler (traditional):**
> - Define node group with min/max/desired
> - Cluster Autoscaler watches for Pending pods (pods that can't schedule due to no capacity)
> - Pending pods detected → Autoscaler adds nodes from the node group
> - Scale down: nodes with low utilization → drain pods → terminate node
>
> **2. Karpenter (what we use — better):**
> - No pre-defined node groups. Karpenter provisions RIGHT-SIZED instances on demand.
> - Pod goes Pending → Karpenter evaluates pod requirements (CPU, memory, GPU, architecture) → provisions the optimal instance type (could be m5.large, c5.xlarge, or even Spot)
> - Consolidation: detects underutilized nodes → moves pods → terminates extra nodes
>
> **Why Karpenter over Cluster Autoscaler:**
>
> | Aspect | Cluster Autoscaler | Karpenter |
> |---|---|---|
> | Instance types | Pre-defined in node group | Picks from 60+ types dynamically |
> | Speed | ~2 min to scale | ~30 seconds |
> | Efficiency | May over-provision (fixed instance type) | Right-sizes per workload |
> | Spot handling | Basic | Diversifies across instance families, handles interruptions |
> | Consolidation | No (leaves underutilized nodes) | Yes (bin-packs and removes waste) |
>
> **Result (P7):** Karpenter + Spot = 60-70% savings on worker node compute costs."

---

### Q: Scenario — You have an Nginx container running, but you cannot get into it. What are the probable scenarios?

**Project Reference:** P3 (Kubernetes troubleshooting)

**Answer:**

> "Assuming 'can't get into it' means `kubectl exec` fails or you can't access Nginx via its service:
>
> **Can't `kubectl exec` into the container:**
>
> 1. **No shell in image** — Minimal/distroless Nginx image has no `bash` or `sh`. Fix: `kubectl exec -it pod -- /bin/sh` (try sh, not bash). If neither exists → use ephemeral debug container: `kubectl debug -it pod --image=busybox`
>
> 2. **Container is crashing** — It's in CrashLoopBackOff. Can't exec into a crashed container. Check: `kubectl logs pod` or `kubectl logs pod --previous` for last crash output.
>
> 3. **Security policy blocking exec** — Pod Security Standard or Kyverno policy blocking `exec` to production pods (we do this). Check RBAC: do you have `pods/exec` permission?
>
> 4. **Read-only filesystem** — Container runs with `readOnlyRootFilesystem: true`. Exec works but you can't write/install anything inside.
>
> **Can't reach Nginx via network:**
>
> 5. **Pod is running but not Ready** — Readiness probe failing. Pod exists but removed from Service endpoints. Check: `kubectl get endpoints`
>
> 6. **Wrong port** — Nginx listening on 80, but Service targets port 8080. Check `containerPort` vs `targetPort`.
>
> 7. **NetworkPolicy blocking** — Default-deny ingress policy, no rule allowing traffic to Nginx pod.
>
> 8. **Nginx config error** — Nginx started but misconfigured (wrong `server_name`, `listen` directive). Check: `kubectl logs pod` for Nginx error logs.
>
> **My debugging order:** `kubectl describe pod` → `kubectl logs` → `kubectl get endpoints` → `kubectl exec` (if possible) → check NetworkPolicies."

---

---
---

# SECTION 53: Kubernetes Scheduling & Observability (Deep Dive)

---

### Q: Explain node affinity, pod affinity/anti-affinity, taints & tolerations, topology spread constraints with production examples.

**Project Reference:** P3 (Kubernetes Platform — scheduling)

**Answer:**

> **Node Affinity:** Schedule on specific nodes (required=hard, preferred=soft). Example: 'only SSD nodes' or 'prefer us-east-1a'.
>
> **Pod Anti-Affinity:** Spread replicas. topologyKey=hostname (across nodes) or zone (across AZs). Example: 'don't put 2 payment pods on same node'.
>
> **Pod Affinity:** Co-locate. Example: 'put cache on same node as app for low latency'.
>
> **Taints & Tolerations:** Repel pods from nodes. Taint node (gpu=true:NoSchedule) → only pods with matching toleration schedule there. Use: GPU nodes, spot instances, dedicated team nodes.
>
> **Effects:** NoSchedule (block new), PreferNoSchedule (soft), NoExecute (evict existing).
>
> **Topology Spread:** Even distribution. maxSkew:1 + topologyKey:zone = perfectly spread across AZs.
>
> **In our setup:** We use pod anti-affinity to spread replicas across AZs (survive AZ failure), Karpenter uses taints for Spot nodes (only fault-tolerant workloads tolerate it), and topology spread ensures no AZ gets more than 1 extra pod vs others.

---

### Q: How to monitor K8s cluster and apps? Prometheus ServiceMonitor, kube-state-metrics, alerting strategy?

**Project Reference:** P3 (Observability stack — Prometheus/Grafana)

**Answer:**

> **Layers:**
> - Infrastructure → node_exporter (CPU, memory, disk per node)
> - Cluster → kube-state-metrics (pod phase, deployment replicas, restarts, node conditions)
> - Application → custom app metrics via ServiceMonitor
> - Autoscaling → metrics-server (for HPA + `kubectl top`)
>
> **metrics-server:** Lightweight, CPU/memory only, for HPA + kubectl top. NOT for alerting.
> **Prometheus:** Full monitoring, custom metrics, alerting, long-term storage.
> **kube-state-metrics:** Cluster-level objects (pod phase, deployment replicas, restarts, node conditions).
>
> **ServiceMonitor:** CRD that auto-configures Prometheus scraping by matching Service labels. No manual prometheus.yml editing. New service with correct labels → automatically scraped.
>
> **Alerting strategy:**
> - P1 (page): service down, node NotReady, error>5%
> - P2 (hours): high latency, disk>80%, HPA at max
> - P3 (ticket): pod restarts, memory trending up, cert expiring
>
> **Dashboards:** Cluster overview, namespace view, application RED (Rate/Errors/Duration), node resources, HPA status.

---

### Q: Explain pod states: ImagePullBackOff, Pending, Evicted, OOMKilled, CreateContainerConfigError. How to debug each?

**Project Reference:** P3 (Kubernetes troubleshooting)

**Answer:**

> | State | Cause | Debug | Fix |
> |---|---|---|---|
> | **ImagePullBackOff** | Can't pull image (typo, no imagePullSecret, DockerHub rate limit, network) | `kubectl describe pod` → Events | Check image name, add secret, use private registry/mirror |
> | **Pending** | Can't schedule (no resources, selector mismatch, taint, PVC unbound) | `kubectl describe pod` → Events (FailedScheduling) | Scale nodes, fix selectors, add toleration, check StorageClass |
> | **Evicted** | Kubelet kicked pod (disk/memory/ephemeral storage pressure) | `kubectl describe node` → Conditions | Clean images, fix leaks, increase disk, set resource requests |
> | **OOMKilled** | Exceeded memory limit (cgroup killed it) | `kubectl describe pod` → Last State: OOMKilled | Increase limit, fix memory leak, set -Xmx for JVM |
> | **CreateContainerConfigError** | Missing ConfigMap or Secret referenced by pod | `kubectl describe pod` → Events | Create the missing resource in same namespace |
>
> **Debug tree:**
> - Pending → check Events section for scheduling reason
> - ImagePull → describe pod, verify image exists and credentials
> - CrashLoop → `kubectl logs --previous` for crash output
> - OOM → check `kubectl top pods`, increase memory limits
> - Evicted → check node conditions, clean up disk

---

### Q: What metrics are you monitoring with Prometheus and Grafana, and did you configure them yourself?

**Project Reference:** P3 (Observability — full stack ownership)

**Answer:**

> "Yes, configured the full stack myself. Deployed via `kube-prometheus-stack` Helm chart (Prometheus Operator, AlertManager, Grafana, node-exporter, kube-state-metrics). Custom ServiceMonitors for application metrics. Grafana dashboards provisioned as code (JSON in Git). AlertManager routing: PagerDuty (P1), Slack (P2), Jira (P3).
>
> **What I monitor:**
>
> **Infrastructure (USE method):**
> - CPU utilization, memory usage, disk I/O, network throughput per node
> - Node conditions (pressure, NotReady), kubelet health, API server latency
>
> **Kubernetes:**
> - Pod restarts, pending pods, OOMKilled count, CrashLoopBackOff
> - HPA current vs desired replicas
> - PVC usage %, node allocatable vs requested
> - Deployment rollout status
>
> **Application (RED method):**
> - Rate: `http_requests_total` (requests per second)
> - Errors: `http_requests_total{status=~'5..'}` (error rate)
> - Duration: `http_request_duration_seconds` (P50, P95, P99)
> - Custom business metrics (orders/min, queue depth)
>
> **Key principle:** RED for services, USE for infrastructure. Alert on symptoms (user impact), not causes. Dashboard per team/namespace."

---
---

