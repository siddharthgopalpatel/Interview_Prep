# Technical Architecture Concepts — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 7, 21

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 7: Technical Architecture & Concepts

---

### Q: If you are to design a highly available and fault tolerant architecture on AWS, what are the key things you'll look at?

**Project Reference:** P2 (3-Tier AWS Architecture), P8 (Multi-Region HA/DR)

**Answer:**

> "Five key pillars:
>
> 1. **Multi-AZ everything** — VPC across 3 AZs. ALB distributes traffic across AZs. EC2 ASG spans all 3. Aurora Multi-AZ with automatic failover (<30 seconds). No single AZ dependency.
>
> 2. **Auto-scaling** — ASG with target tracking (CPU-based or request-based). Scale 3→20 instances. Karpenter for Kubernetes node scaling.
>
> 3. **Stateless compute** — Application instances are disposable. Session data in ElastiCache/DynamoDB, not local disk. Any instance can serve any request.
>
> 4. **Database resilience** — Aurora Multi-AZ (same region HA) + Aurora Global Database (cross-region DR). Read replicas for read-heavy workloads.
>
> 5. **Health checks + circuit breakers** — ALB health checks remove unhealthy targets in seconds. Route53 health checks trigger DNS failover for regional outages.
>
> For fault tolerance specifically: assume anything can fail. Design so that when it does, traffic automatically routes to healthy components without human intervention."

---

### Q: How would you manage the database if it's a multi-region setup?

**Project Reference:** P8 (Multi-Region HA/DR)

**Answer:**

> "We use **Aurora Global Database**:
>
> - **Primary region** (us-east-1): handles all writes. Replication to secondary region in <1 second (typically ~100ms).
> - **Secondary region** (us-west-2): read-only replica. Promotes to read-write in ~1 minute during failover.
> - **RPO: <1 second** — minimal data loss during regional failure.
>
> **Failover process:**
> 1. Route53 health check detects primary region down
> 2. DNS failover routes traffic to secondary region
> 3. Aurora secondary promotes to primary (detach + promote, ~60 seconds)
> 4. Application in DR region starts receiving write traffic
>
> **Key consideration:** After failover, the old primary must be re-created as a secondary. It's NOT automatic re-join. We handle this in our DR runbook.
>
> **Alternative for non-Aurora:** DynamoDB Global Tables (active-active, multi-region writes, no manual failover needed). We'd use this for session stores or metadata — not transactional data."

---

### Q: Explain when a user hits a website from the internet, what are the different steps that happen in the background?

**Project Reference:** P2 (3-Tier AWS Architecture)

**Answer:**

> "Full request flow in our architecture:
>
> 1. **DNS resolution** — User types `app.example.com`. Browser queries DNS → Route53 returns CloudFront distribution IP (or ALB IP if no CDN).
>
> 2. **TLS handshake** — Browser establishes HTTPS connection. SSL terminates at CloudFront or ALB (ACM certificate).
>
> 3. **CDN cache check** — CloudFront checks if static content is cached at edge. If yes → serve directly (fast). If no → forward to origin.
>
> 4. **WAF inspection** — Request passes through AWS WAF rules (SQL injection, XSS, rate limiting, geo-blocking). Bad request → blocked.
>
> 5. **Load balancer** — ALB receives request, routes to healthy target (EC2/pod) based on path rules and health checks.
>
> 6. **Application processing** — App server handles business logic, queries database (Aurora), returns response.
>
> 7. **Database query** — App connects to Aurora reader/writer endpoint. Connection pooling, query execution, response.
>
> 8. **Response back** — Reverse path: App → ALB → CloudFront (caches if cacheable) → User.
>
> Total time for a cached request: ~20-50ms. Uncached with DB: ~100-300ms."

---

### Q: Explain the concept of stateful and stateless infrastructure?

**Project Reference:** P2 (3-Tier AWS), P3 (Kubernetes — StatefulSets vs Deployments)

**Answer:**

> "**Stateless:** Instance doesn't store any data locally. Kill it, replace it — no data loss. All shared state lives externally (database, cache, object store). Example: our EC2 web servers in ASG, Kubernetes Deployments. Any instance serves any request.
>
> **Stateful:** Instance holds data that must persist. You can't just kill and replace it without handling the data. Example: databases, Kafka brokers, etcd nodes. In Kubernetes, these run as StatefulSets with Persistent Volumes.
>
> **Why it matters for DevOps:**
> - Stateless = easy to scale, easy to patch (just terminate and launch new). Our ASG rolling update works because instances are stateless.
> - Stateful = hard to scale, hard to patch (must drain data, failover, then patch). Our database patching needs Aurora failover, not just 'terminate instance.'
>
> **Our rule:** Keep compute stateless wherever possible. Push state to managed services (Aurora, ElastiCache, S3). This is why we use EBS-backed PVCs in K8s only for databases, not for application pods."

---

### Q: How would you debug a slow performing application where the web page takes a lot of time to render?

**Project Reference:** P3 (Observability stack), P6 (Istio — Jaeger tracing)

**Answer:**

> "I follow a **top-down approach** — start broad, narrow down:
>
> 1. **Where is the time spent?** — Check browser dev tools (Network tab). Is it DNS? TLS? TTFB (time to first byte)? Large payload? If TTFB is high → backend problem. If content download is slow → payload/network.
>
> 2. **Backend tracing** — Use Jaeger distributed tracing. It shows exactly which microservice call took how long. 'Order service: 50ms, Payment service: 2.5 seconds' — found the bottleneck.
>
> 3. **Database slow queries** — Check Aurora Performance Insights or slow query log. A missing index on a 10M row table = 3-second query.
>
> 4. **Resource saturation** — Check Prometheus/Grafana: Is the pod CPU-throttled? Memory swapping? Is the node overloaded? Check `kubectl top pods`.
>
> 5. **Network latency** — Check if cross-AZ calls are adding latency. Istio metrics show inter-service latency per hop.
>
> 6. **External dependencies** — Third-party API slow? Check timeout settings, add circuit breakers.
>
> Key: Don't guess. Use traces to pinpoint the exact bottleneck, then fix surgically."

---

### Q: In terms of monitoring, tell me the difference between metrics and traces?

**Project Reference:** P3 (Observability stack — Prometheus + Jaeger)

**Answer:**

> "**Metrics** = WHAT is happening (aggregated numbers).
> - 'HTTP 500 errors increased by 40% in the last 5 minutes'
> - 'Average response time is 350ms'
> - 'CPU usage is at 85%'
> - Good for: alerting, dashboards, trends. Tools: Prometheus, CloudWatch.
>
> **Traces** = WHY is it happening (per-request journey).
> - 'This specific request took 3.2 seconds because the payment-service DB query took 2.8 seconds'
> - Shows the full call chain across microservices with timing per hop
> - Good for: debugging specific slow requests, finding bottlenecks. Tools: Jaeger, X-Ray.
>
> **How they work together:** Metrics tell you THERE IS a problem (alert: latency spike). Traces tell you WHERE the problem is (this specific DB call in this specific service). You need both — metrics for detection, traces for diagnosis."

---

### Q: Tell me the difference between a monolithic and microservices architecture?

**Project Reference:** P1 (15+ microservices), P3 (Kubernetes platform)

**Answer:**

> | Aspect | Monolith | Microservices |
> |---|---|---|
> | **Codebase** | Single deployable unit | Many independent services |
> | **Deployment** | Deploy everything together | Deploy services independently |
> | **Scaling** | Scale the whole app | Scale only the hot service |
> | **Failure** | One bug can crash everything | Failure isolated to one service |
> | **Complexity** | Simple to start, hard to maintain at scale | Complex infra, but each service is simple |
>
> **In our environment:** We run 15+ microservices on Kubernetes. Each team owns their service, deploys independently via ArgoCD, scales independently via HPA. A bug in the notification service doesn't take down the payment service.
>
> **The trade-off:** Microservices need more infrastructure — service mesh (Istio), distributed tracing (Jaeger), centralized logging (EFK). A monolith is simpler if your team is small and app is straightforward."

---

### Q: How would you enable the data of one microservice to reach another microservice if they are in different clusters?

**Project Reference:** P6 (Istio Service Mesh), P8 (Multi-Region)

**Answer:**

> "Three common approaches depending on the use case:
>
> 1. **API Gateway / External service** — Service A in Cluster 1 calls Service B in Cluster 2 via an external API Gateway (Kong, AWS API Gateway) or Ingress endpoint. Simple, works across any cluster. Downside: goes through the internet/ALB, adds latency.
>
> 2. **Istio Multi-Cluster Mesh** — Both clusters are part of the same Istio mesh. Service discovery works transparently across clusters. Service A calls Service B by Kubernetes service name — Istio routes it cross-cluster over mTLS. Seamless, secure, no code changes.
>
> 3. **Event-driven (async)** — Services communicate via a shared message broker (Kafka, SQS, EventBridge). Service A publishes an event, Service B in another cluster consumes it. No direct coupling, eventual consistency.
>
> **What we'd use:** For synchronous real-time calls → Istio multi-cluster or API Gateway. For async data sharing → Kafka/EventBridge. Never direct pod-to-pod across clusters without encryption and service identity."

---

### Q: What do you know about API gateways and APIs?

**Project Reference:** P2 (ALB + WAF), P5 (Serverless — API Gateway + Lambda)

**Answer:**

> "An API Gateway is a single entry point that sits in front of your backend services. It handles:
>
> - **Routing** — `/users` → user service, `/orders` → order service
> - **Authentication/Authorization** — Validate JWT tokens, API keys before traffic hits backend
> - **Rate limiting** — Prevent abuse (100 requests/min per client)
> - **SSL termination** — Handle TLS at the gateway, backends get plain HTTP
> - **Request transformation** — Add headers, modify payloads
> - **Caching** — Cache GET responses to reduce backend load
>
> **In our work:**
> - AWS API Gateway + Lambda (P5) — Serverless APIs, pay per request, built-in throttling
> - ALB as a basic API gateway (P2) — Path-based routing to different target groups
> - Istio Ingress Gateway (P6) — K8s-native, mTLS, AuthorizationPolicies, traffic management
>
> For microservices, the API gateway decouples clients from internal service topology. Clients call one URL, gateway routes to the right service."

---
---

# SECTION 21: Microservices Concepts

---

### Q: What do you mean by 'loosely coupled'? Can you give an example?

**Project Reference:** P1 (15+ microservices), P5 (Event-driven architecture)

**Answer:**

> "**Loosely coupled** means services can change, deploy, scale, and fail independently without affecting each other.
>
> **Example from our system:**
> - **Order Service** places an order → publishes an event to SQS/EventBridge: `OrderPlaced`
> - **Notification Service** listens for `OrderPlaced` → sends email confirmation
> - **Inventory Service** listens for `OrderPlaced` → reduces stock count
>
> Order Service doesn't know or care about Notification Service. It just fires the event and moves on. If Notification Service crashes — orders still work. If we replace the Notification Service with a completely new implementation — Order Service doesn't change.
>
> **Tightly coupled (bad):** Order Service directly calls Notification Service via HTTP synchronously. Notification down = order fails. Can't change Notification without coordinating with Order team.
>
> **How we achieve loose coupling:**
> - Async communication (SQS, EventBridge) — not direct HTTP calls
> - API contracts (OpenAPI spec) — agreed interface, implementation is independent
> - Separate databases per service — no shared DB schema
> - Independent deployment via ArgoCD — one service deploys without touching others"

---
---

