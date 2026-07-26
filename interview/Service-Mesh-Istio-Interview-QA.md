# Interview Q&A Bank: Service Mesh with Istio

## Project: Zero-Trust Networking & Observability for Microservices on EKS

**Technologies:** Istio, Envoy, Kubernetes (EKS), mTLS, AuthorizationPolicy, VirtualService, DestinationRule, Jaeger, Kiali, Prometheus

---

## Section 1: Project Story

### Q1: Walk me through the Service Mesh project in 2 minutes.

**Answer:**
Implemented Istio service mesh on our EKS platform for 15 microservices. Strict mTLS encrypts all internal traffic automatically — extending our external TLS (ALB) to cover east-west traffic between services. AuthorizationPolicies implement zero-trust at service identity level — more granular than NetworkPolicies which only work at IP/label level. VirtualService enables internal canary deployments (complementing Argo Rollouts from Project 1 which handles north-south canary). Jaeger distributed tracing found a 3-second OpenAI bottleneck in minutes — would have taken hours without it. Circuit breakers prevent cascade failures when external APIs go slow. 10-15% resource overhead but saves weeks of engineering time not implementing retry/TLS/tracing in 5 different languages.

---

### Q2: Why did you add a service mesh? What problem existed before?

**Answer:**
Before mesh: all internal traffic between microservices was plain HTTP (unencrypted). Any pod could call any other pod. No visibility into which service was causing latency. Each team had to implement their own retry logic, timeouts, and error handling in different languages (Python, Java, Node.js).

After mesh: mTLS everywhere (zero code changes), identity-based access control (only auth-service can call user-db), distributed tracing (find bottleneck in 30 seconds), uniform retry/timeout/circuit breaker for ALL services regardless of language.

---

## Section 2: Technical Deep-Dive

### Q3: How does Istio's sidecar injection work? What happens when a pod starts?

**Answer:**
1. Namespace labeled `istio-injection=enabled`
2. When pod is created → Istio's mutating admission webhook intercepts the API call
3. Webhook modifies pod spec: injects Envoy proxy container as sidecar + init container
4. Init container sets up iptables rules — ALL traffic in/out of app container is redirected through Envoy
5. App still calls `http://auth-service:80` — but iptables redirects to local Envoy (localhost:15001)
6. Envoy encrypts (mTLS), applies policies, collects metrics, then forwards to destination's Envoy
7. Destination Envoy decrypts, verifies identity, forwards to destination app container

**Key:** App has NO idea mesh exists. Zero code changes. Transparent interception via iptables.

---

### Q4: How does mTLS work in Istio? How are certificates managed?

**Answer:**
1. `istiod` runs a Certificate Authority (CA)
2. When pod starts, Envoy sidecar requests a certificate from istiod via SDS (Secret Discovery Service)
3. istiod issues a SPIFFE identity certificate: `spiffe://cluster.local/ns/app-prod/sa/learneasyai-sa`
4. Certificate is short-lived (24 hours by default) — auto-rotated
5. When Service A calls Service B: both Envoys verify each other's certificates (MUTUAL TLS)
6. Both sides prove identity cryptographically — not just "I'm from the right IP"

**PeerAuthentication (STRICT mode):**
```yaml
spec:
  mtls:
    mode: STRICT  # All traffic MUST be mTLS. Plain HTTP rejected.
```

Zero manual cert management. Zero cert rotation scripts. Zero code changes.

---

### Q5: How does AuthorizationPolicy differ from Kubernetes NetworkPolicy?

**Answer:**
| | NetworkPolicy (K8s) | AuthorizationPolicy (Istio) |
|---|---|---|
| Layer | L3/L4 (IP, port, label) | L7 (HTTP method, path, headers, identity) |
| Identity | Pod labels (can be spoofed) | mTLS certificate (cryptographically verified) |
| Example | "Allow port 8080 from namespace X" | "Allow only POST /charge from order-service ServiceAccount" |
| Encryption | ❌ No | ✅ All traffic encrypted |
| Granularity | Port-level | Method + path + header level |

**We use BOTH:** NetworkPolicy for broad subnet-level controls, AuthorizationPolicy for fine-grained identity-based access. Defence in depth.

---

### Q6: How does Istio VirtualService enable internal canary deployments?

**Answer:**
```yaml
spec:
  hosts: [auth-service]
  http:
    - route:
        - destination: {host: auth-service, subset: stable}
          weight: 90
        - destination: {host: auth-service, subset: canary}
          weight: 10
```

When LearnEasyAI calls auth-service → Envoy routes 90% to stable, 10% to canary. Zero code changes in caller.

**Difference from Argo Rollouts (Project 1):**
- Argo Rollouts: Canary for EXTERNAL traffic (north-south, users → app)
- Istio VirtualService: Canary for INTERNAL traffic (east-west, service → service)

Both complement each other. Different layers.

---

### Q7: How does a circuit breaker work in Istio?

**Answer:**
**Problem:** OpenAI service goes slow (5s response). Without circuit breaker → LearnEasyAI keeps calling → all threads waiting → LearnEasyAI becomes slow → cascade failure.

**DestinationRule with outlierDetection:**
```yaml
outlierDetection:
  consecutive5xxErrors: 3       # After 3 errors in...
  interval: 30s                 # ...30 seconds...
  baseEjectionTime: 60s         # ...stop calling for 60s
  maxEjectionPercent: 50        # Never eject more than 50% of endpoints
```

**What happens:** After 3 consecutive failures, Envoy removes that endpoint from the pool for 60 seconds. Traffic goes to healthy endpoints. After 60s, endpoint is re-added and tested again.

**Analogy:** If a restaurant keeps giving food poisoning (3 errors), stop sending customers there for 60 seconds. Check again later.

---

### Q8: How does Jaeger distributed tracing work without code changes?

**Answer:**
1. Envoy sidecar automatically generates a trace span for EVERY request it proxies
2. Span includes: timestamp, duration, source, destination, HTTP status, latency
3. Spans are linked by trace headers (B3 or W3C traceparent) propagated between services
4. All spans sent to Jaeger collector → stored → queryable via Jaeger UI

**Result:**
```
User request → LearnEasyAI (12ms)
                  → auth-service (8ms)
                  → openai-service (3200ms) ← BOTTLENECK!
                      → OpenAI API (3150ms)
```

Found 3-second bottleneck in 30 seconds. Without tracing: check 5 services' logs manually → hours.

**One caveat:** App must propagate trace headers (forward `x-b3-*` or `traceparent` headers). Not code-instrumentation, but header forwarding needed.

---

## Section 3: Troubleshooting

### Q9: mTLS is enabled but one service can't communicate with another. How do you debug?

**Answer:**
1. Check PeerAuthentication mode: Is it STRICT? If destination expects mTLS but source isn't in mesh → connection refused.
2. Check if sidecar is injected: `kubectl get pod -o yaml | grep istio-proxy`. No sidecar = no mTLS.
3. Check AuthorizationPolicy: Is there a deny-all? Is the source service's identity in the allow list?
4. `istioctl proxy-status` — shows if Envoy config is synced. "STALE" = istiod can't reach the proxy.
5. `istioctl proxy-config listeners <pod>` — verify traffic is being intercepted.
6. Check if destination has correct labels matching DestinationRule subsets.

**Most common cause:** Sidecar not injected (namespace not labeled) or AuthorizationPolicy too restrictive (forgot to allow the new service).

---

### Q10: Latency increased after enabling Istio. How do you investigate?

**Answer:**
1. **Expected overhead:** ~2-5ms per hop (Envoy processing). If increase is 2-5ms → normal.
2. **If increase is 50ms+:** Something else is wrong.
3. **Check:** `istioctl proxy-config cluster <pod>` — are connection pools configured correctly?
4. **Common causes:**
   - Envoy doing TLS handshake on every request (connection not reused). Fix: connection pooling in DestinationRule.
   - Too many retries configured (3 retries × 3 services = 9 total attempts on failure).
   - Sidecar requesting too many resources → CPU throttled. Fix: increase sidecar resource limits.
5. **Jaeger trace:** Compare latency WITH mesh vs without. Pinpoints exactly where time is spent.

---

### Q11: After deploying new AuthorizationPolicy, some legitimate traffic is blocked. How do you fix?

**Answer:**
1. **Check policy:** `kubectl get authorizationpolicy -n <ns>` — is there a deny-all catching traffic?
2. **Order matters:** If deny-all exists, EVERY allowed path must be explicitly listed.
3. **Debug tool:** `istioctl x authz check <pod>` — shows which policy is blocking.
4. **Common mistakes:**
   - Forgot to allow health check path (`GET /health`) — readiness probe fails → pod removed from service.
   - ServiceAccount name mismatch (typo in `principals` field).
   - Missing namespace in source principal: `cluster.local/ns/WRONG-NS/sa/service-sa`.
5. **Quick fix:** Temporarily set `action: AUDIT` (logs but doesn't block) → check logs → fix → switch back to ALLOW.

---

## Section 4: System Design

### Q12: When should you NOT use a service mesh?

**Answer:**
| Scenario | Use Mesh? | Why |
|---|---|---|
| <10 services, one team | ❌ | Overkill — NetworkPolicies + Argo Rollouts sufficient |
| Resource-constrained cluster | ❌ | 10-15% overhead per pod matters |
| Team doesn't understand proxies | ❌ | Debugging becomes harder, not easier |
| Only need encryption (no traffic mgmt) | ⚠️ Maybe | Consider Cilium (eBPF, no sidecar, less overhead) |
| 15+ services, multiple teams, compliance | ✅ | Communication complexity explodes without mesh |
| Zero-trust compliance requirement | ✅ | Must encrypt + verify identity everywhere |
| Need per-request observability without code changes | ✅ | Tracing without instrumenting 15 services |

---

### Q13: How would you implement zero-trust networking for 50 microservices?

**Answer:**
1. **Default deny everywhere:** AuthorizationPolicy with empty spec in every namespace
2. **Explicit allow per call path:** Map out which service calls which (service dependency graph from Kiali)
3. **Identity-based, not IP-based:** Use ServiceAccount principals in policies (cryptographically verified via mTLS)
4. **L7 granularity:** Not just "A can reach B" — "A can POST /orders on B, but not DELETE /orders"
5. **Automated policy generation:** Use Kiali's traffic graph to auto-generate initial policies from observed traffic
6. **Gradual rollout:** Start with AUDIT mode (log violations, don't block) → review → switch to ENFORCE
7. **Exception handling:** Some services need broad access (monitoring, service mesh control plane) → explicit wide policies with documentation

---

## Section 5: Comparison & Decisions

### Q14: Istio vs Linkerd vs Cilium — when would you choose each?

**Answer:**
| | Istio | Linkerd | Cilium |
|---|---|---|---|
| Overhead | High (~128Mi/pod) | Low (~20Mi/pod) | Lowest (no sidecar, eBPF) |
| Features | Full (traffic mgmt, security, observability) | Basic (mTLS, retries, metrics) | Growing (mTLS, observability, NetworkPolicy) |
| Circuit breaker | ✅ | ❌ | ❌ |
| L7 AuthorizationPolicy | ✅ Advanced | ✅ Basic | ✅ Growing |
| Learning curve | Steep | Moderate | Moderate |
| Best for | Enterprise, full features, compliance | Simpler needs, less ops overhead | Performance-critical, kernel-level, no sidecar wanted |

**Our choice (Istio):** Needed advanced AuthorizationPolicies (L7, per-path), circuit breakers (external APIs), and Jaeger integration. Worth the overhead for 15 services.

---

### Q15: mTLS (Istio) vs NetworkPolicy — can't NetworkPolicy be enough?

**Answer:**
| | NetworkPolicy | mTLS (Istio) |
|---|---|---|
| Encrypts traffic | ❌ No — just allows/blocks | ✅ Yes — all traffic encrypted |
| Verifies identity | ❌ Label-based (can be spoofed by anyone who can set labels) | ✅ Certificate-based (cryptographic proof) |
| Stops man-in-the-middle | ❌ No | ✅ Yes |
| Compliance (SOC2, PCI) | ⚠️ Partial (segmentation only) | ✅ Full (encryption + identity + audit) |

**When NetworkPolicy alone is enough:** Internal tools, non-sensitive data, no compliance requirement.
**When you need mTLS:** Payment data, PII, compliance requires encrypted internal traffic, zero-trust mandate.

---

### Q16: Service mesh vs implementing retries/TLS in application code?

**Answer:**
| | App-level (each team implements) | Service mesh (infrastructure-level) |
|---|---|---|
| Consistency | ❌ Each team does differently | ✅ Uniform across all services |
| Languages | Must implement in Python, Java, Node.js separately | ✅ Language-agnostic (Envoy handles all) |
| Code changes | Every service modifies code | ✅ Zero code changes |
| Updates | Change retry logic → redeploy 15 services | Change one DestinationRule → all services get it |
| Observability | Each team adds tracing SDK | ✅ Automatic (Envoy collects for all) |

**Mesh wins** when you have 10+ services in multiple languages. Not worth it for 3 services in one language.

---

## Section 6: Behavioral

### Q17: How did you roll out Istio without disrupting production?

**Answer (STAR):**
- **Situation:** 15 microservices running on EKS. Can't afford downtime during mesh adoption.
- **Task:** Add Istio without breaking existing traffic patterns.
- **Action:** Phased rollout:
  1. Install Istio in `PERMISSIVE` mode (accepts both mTLS and plain HTTP) — zero breakage
  2. Enable sidecar injection on ONE non-critical namespace first (monitoring). Verify for 1 week.
  3. Inject into staging namespace. Run full test suite. Verify metrics flow to Jaeger.
  4. Inject into production namespace. Still PERMISSIVE (both encrypted and plain work).
  5. After 2 weeks of stable operation → switch to STRICT (only mTLS allowed).
  6. Add AuthorizationPolicies gradually — AUDIT mode first, then ENFORCE.
- **Result:** Zero downtime. Zero incidents during rollout. Full mesh operational in 4 weeks.

---

### Q18: Tell me about a time distributed tracing saved you during an incident.

**Answer (STAR):**
- **Situation:** Users reported "app is slow" (page load 5+ seconds). 5 services in the call chain — which one?
- **Task:** Find the bottleneck quickly. SLA breach in progress.
- **Action:** Opened Jaeger → filtered by slow requests (>3s) → clicked a trace:
  - LearnEasyAI: 12ms ✅
  - auth-service: 8ms ✅
  - openai-service: 3200ms 🔴 ← FOUND IT
  - OpenAI external API: 3150ms ← root cause: external API degraded
- **Result:** Root cause identified in 2 minutes. Added caching for repeated prompts → response time dropped to 50ms for cache hits. Without tracing: would've taken 2+ hours checking each service's logs.

---

## Section 7: Future & Improvements

### Q19: What's next for the service mesh on your platform?

**Answer:**
| Priority | Improvement | Why |
|---|---|---|
| 1 | Ambient mesh (Istio without sidecars) | Reduce 10-15% overhead — ztunnel at node level |
| 2 | Rate limiting per service (Envoy filter) | Protect services from noisy neighbors |
| 3 | Fault injection (chaos testing) | Test: "What if auth-service has 5s latency?" — verify circuit breaker works |
| 4 | Multi-cluster mesh | Same policies across dev/staging/prod clusters |
| 5 | WASM plugins | Custom Envoy filters without recompiling (auth, logging, transformation) |

---

### Q20: If you started over, would you still choose Istio?

**Answer:**
**For our use case (15 services, compliance, need tracing + circuit breaker):** Yes, still Istio.

**But if starting fresh in 2026:** I'd seriously evaluate **Cilium with eBPF** — no sidecar overhead, kernel-level networking, growing feature set (mTLS, observability, L7 policy). The industry is moving toward sidecar-less meshes.

**When I'd switch:**
- If Cilium adds equivalent L7 AuthorizationPolicy + Jaeger integration → switch (less overhead)
- If we only needed mTLS without traffic management → Linkerd (simpler, lighter)
- If overhead became unacceptable for latency-sensitive paths → Cilium (eBPF = fastest)

**Key principle:** Mesh is a tool, not a religion. Choose based on requirements, not hype.

---

*End of Q&A Bank — 20 questions covering all 7 dimensions*
