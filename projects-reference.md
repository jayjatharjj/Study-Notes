# Projects Reference — Smart360 · Deep Fathom · WebX

> The grounding doc for every system-design drive, project deep-dive, and STAR story. Know these cold — architecture, *your* decisions, the trade-offs you didn't take, and the numbers. Interviewers probe depth here; this is where 2.5 YOE either reads as senior or shallow.

---

## Smart360 — Multi-tenant performance-review platform (AWS)

**What it is:** A multi-tenant 360-degree performance-review SaaS — tenants (companies) run review cycles; users give/receive feedback; dashboards aggregate it.

**Architecture:** 5 microservices on **Spring Cloud** — API Gateway (Spring Cloud Gateway) → Auth → Data → Visualization → Notification. **Eureka** for service discovery. **PostgreSQL** (multi-tenant), **Redis** (two uses: JWT-revocation blacklist + S3 pre-signed-URL cache), **AWS S3** for artifacts. JWT validated **at the gateway** (single policy point), not per service.

**Your key decisions + trade-offs:**
- **Stateless JWT at the edge** over per-service session lookup → −60% sign-in latency (no per-request DB roundtrip). Trade-off: revocation needs a Redis blacklist (stateless tokens can't be "deleted").
- **Eureka** for discovery — chosen for team familiarity; *I'd use Kubernetes-native discovery today* (Eureka became redundant infra once we had K8s in mind).
- **Redis cache for S3 pre-signed URLs** (TTL 55 min vs 60-min URL expiry) → −80% S3 API calls. Trade-off: a small staleness window, acceptable for read URLs.
- **RBAC** at repository/table/permission granularity via Spring Security + JWT claims, enforced by a shared gateway filter.

**Hardest problem:** an **N+1 query** on the `User → Roles → Permissions` graph fired ~600 queries per authenticated request (3 queries × 200 users). **Fix:** `@EntityGraph(attributePaths = {"roles","roles.permissions"})` + `JOIN FETCH` collapsed it to 1 query; added a composite index on `(tenant_id, user_id)`; cached hot reads in Redis.

**Numbers:** **60 s → 2–3 s (96% latency cut)** · **−80% S3 calls** · **−60% sign-in latency**.

**What I'd do differently:** K8s-native discovery from day one; Postgres advisory locks instead of Redis for the token blacklist (one fewer moving part).

**Drives it grounds:** caching, API gateway, RBAC/JWT, N+1/JPA, multi-tenant data, "design a dashboard/feed."

---

## Deep Fathom — Multi-tenant B2B SaaS on Azure (FedRAMP-grade)

**What it is:** A B2B multi-tenant SaaS on Azure with a compliance (FedRAMP/GovCloud) requirement — strong tenant isolation was non-negotiable.

**Architecture:** **Azure Container Apps** (most services, scale-to-zero) + **AKS** (services needing custom operators); **PostgreSQL Flexible Server**, **Redis**, **Azure Front Door** (CDN/edge), **Key Vault** (secrets). **Bicep IaC** provisions 30+ resources reproducibly.

**Your key decisions + trade-offs:**
- **PostgreSQL Row-Level Security** (`tenant_id = current_setting('app.tenant_id')`) over app-layer `WHERE tenant_id = ?` filtering → isolation enforced at the **DB layer**, safe even if an app query forgets the filter. Trade-off: every connection must set the tenant context (a `DataSource` interceptor); harder cross-tenant admin queries (superuser path).
- **Bicep** over Terraform → native Azure, no state-file management, first-class VS Code support. Trade-off: Azure-only (Terraform is multi-cloud).
- **Container Apps** (scale-to-zero) for most services; **AKS** only where K8s operators were needed → cost vs control balance.

**Hardest problem:** FedRAMP required **identical staging + GovCloud environments** — solved by **parameterizing the Bicep modules per region**, reproducing the whole stack in GovCloud from the same codebase.

**Numbers:** **−57% CI/CD pipeline (23 → 10 min)** via BuildKit parallel builds + `--cache-from` registry caching + GitLab `needs:` DAG restructuring · **RLS across 50+ tables**.

**What I'd do differently:** introduce the **outbox pattern** (Debezium CDC) earlier for event publishing instead of polling.

**Drives it grounds:** multi-tenancy/isolation, IaC + reproducible environments, CI/CD, CDN/Front Door, secrets management, SQL-vs-NoSQL, "design a multi-tenant platform."

---

## WebX — LLM-integrated product (async at scale)

**What it is:** A product with LLM-powered features (document analysis / generation) where operations run **2–20 minutes** — far past HTTP timeouts — across multiple AI providers.

**Architecture:** Spring Boot backend; **async job architecture** — `POST /jobs → 202 Accepted + jobId`, job persisted in **PostgreSQL**, worker processes it, **SSE** pushes completion (client also can poll `GET /jobs/{id}`). A **unified provider proxy** abstracts 5+ LLM providers; routes by **cost/capability** (cheap model for classification, premium for hard generation). **Circuit breaker** (Resilience4j) for graceful degradation. Vue.js / React frontend.

**Your key decisions + trade-offs:**
- **202 + jobId + SSE** over synchronous calls → LLM ops exceed ~30 s load-balancer timeouts; async is the only correct shape. Trade-off: more moving parts (job store, worker, push channel).
- **Job state in PostgreSQL**, not in-memory → pod restarts don't lose jobs (durability). Trade-off: DB write per state transition.
- **Unified provider interface** → swap/route providers without touching callers; cost optimization by routing. Trade-off: lowest-common-denominator API; provider-specific features need escape hatches.
- **Circuit breaker + fallback** → a provider outage degrades to a cheaper/alt provider instead of failing the feature.

**Hardest problem:** providers stream in **different formats** (OpenAI SSE vs Anthropic streaming) — abstracted into a **common reactive stream** so the client sees one contract.

**Numbers:** async jobs handling 2–20 min ops reliably; 5+ providers behind one proxy; provider-cost routing.

**What I'd do differently:** a real queue (SQS/RabbitMQ) instead of DB polling for better backpressure + retry semantics.

**Drives it grounds:** async/job systems, SSE/streaming, LLM integration, circuit breakers/resilience, rate limiting per provider, "design an async processing / notification / LLM feature system."

---

## Quick metric recall (say these cold)

| Metric | Project | One-line story |
|--------|---------|----------------|
| 60 s → 2–3 s (96%) | Smart360 | N+1 → EntityGraph + index + Redis |
| −80% S3 calls | Smart360 | Redis cache of pre-signed URLs (55-min TTL) |
| −60% sign-in latency | Smart360 | stateless JWT at the gateway |
| −57% CI/CD (23→10 min) | Deep Fathom | BuildKit + registry cache + GitLab DAG |
| RLS on 50+ tables | Deep Fathom | DB-layer tenant isolation |
| 2–20 min async jobs, 5+ providers | WebX | 202 + jobId + SSE, unified proxy |

*Use **"I"** for what you personally did; name the **trade-off you rejected** in every story — that's the senior signal.*
