# Interview Q&A — Jay Jathar

Profile: 2+ years Full Stack (Java 17, Spring Boot, Spring Cloud, Vue.js, PostgreSQL, Redis, AWS, Azure, Docker, Kubernetes, JWT/OAuth2, LLM integration)

---

## Java & Spring Boot Core

**Q: What is the difference between `@Component`, `@Service`, and `@Repository`?**

> All three are specializations of `@Component` and register beans in the Spring context. `@Repository` adds persistence exception translation (converts SQL exceptions to Spring's `DataAccessException`). `@Service` is a semantic marker for business logic. In your microservices, you'd use `@Repository` on JPA interfaces, `@Service` for authorization logic, and `@Component` for generic utilities.

---

**Q: How does Spring Boot auto-configuration work?**

> Spring Boot reads `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Boot 3.x), finds `@AutoConfiguration` classes, and conditionally applies them using `@ConditionalOnClass`, `@ConditionalOnMissingBean`, etc. For example, if `DataSource` is on the classpath and no custom one is defined, it auto-configures one from `application.properties`. You can exclude specific auto-configs with `@SpringBootApplication(exclude = ...)`.

---

**Q: Explain Bean scopes in Spring.**

> The default scope is **singleton** (one instance per ApplicationContext). Other scopes: **prototype** (new instance per injection), **request/session/application** (web-aware scopes). A common pitfall: injecting a prototype bean into a singleton — you always get the same prototype. Fix: inject `ApplicationContext` and call `getBean()`, or use `@Lookup`.

---

**Q: What happens internally when a Spring Boot application starts?**

> 1. `main()` calls `SpringApplication.run()`
> 2. Creates a `ConfigurableApplicationContext` (Servlet or Reactive)
> 3. Loads `ApplicationContextInitializer` and `ApplicationListener` from `spring.factories`
> 4. Runs auto-configuration — scans classpath, evaluates `@Conditional` annotations
> 5. Refreshes context: instantiates beans, resolves dependencies, runs `@PostConstruct`
> 6. Publishes `ApplicationReadyEvent` — your `CommandLineRunner` / `ApplicationRunner` beans fire here
>
> Key hook points: `ApplicationListener<ApplicationReadyEvent>` for post-startup tasks, `@PostConstruct` for bean initialization, `SmartLifecycle` for controlled startup/shutdown ordering.

---

**Q: How does Spring Boot externalized configuration work and what is the property resolution order?**

> Spring resolves properties in this priority (higher overrides lower):
> 1. Command-line arguments (`--server.port=9090`)
> 2. `SPRING_APPLICATION_JSON` env variable
> 3. OS environment variables
> 4. `application-{profile}.properties` outside the jar
> 5. `application.properties` outside the jar
> 6. `application-{profile}.properties` inside the jar
> 7. `application.properties` inside the jar
> 8. `@PropertySource` annotations
> 9. Default properties
>
> In Azure Container Apps, environment variables override the packaged `application.properties` — this is exactly this resolution order in action.

---

**Q: What is the difference between `@ConfigurationProperties` and `@Value`?**

> `@Value("${property.key}")` injects a single value — brittle (typos fail silently at runtime), not type-safe, no IDE autocompletion for custom properties.
>
> `@ConfigurationProperties(prefix = "app.jwt")` binds an entire namespace to a typed POJO with validation support via `@Validated`. It's refactor-safe, supports complex types (lists, maps), and works with `spring-boot-configuration-processor` to generate IDE metadata. Always prefer `@ConfigurationProperties` for any group of related properties.

---

**Q: How does `@Transactional` work under the hood? What are its common pitfalls?**

> Spring wraps your bean in a proxy (JDK dynamic proxy or CGLIB). When a transactional method is called, the proxy intercepts it, starts a transaction via `PlatformTransactionManager`, invokes the real method, then commits or rolls back.
>
> **Pitfalls:**
> - **Self-invocation**: calling a `@Transactional` method from within the same class bypasses the proxy — no transaction. Fix: inject self or use `ApplicationContext`.
> - **Default rolls back only on `RuntimeException`** — checked exceptions commit. Specify `rollbackFor = Exception.class` if needed.
> - **Wrong propagation**: `REQUIRED` (default) joins an existing tx. `REQUIRES_NEW` suspends it and starts fresh — use this for audit logging that must persist even if the outer tx rolls back.
> - **`@Transactional` on `private` methods** — silently ignored by proxy.

---

**Q: Explain Spring's bean lifecycle.**

> 1. Instantiation (constructor called)
> 2. Dependency injection (setter/field injection)
> 3. `BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware` callbacks
> 4. `BeanPostProcessor.postProcessBeforeInitialization()`
> 5. `@PostConstruct` / `InitializingBean.afterPropertiesSet()`
> 6. `BeanPostProcessor.postProcessAfterInitialization()` — this is where AOP proxies are created
> 7. Bean is ready for use
> 8. On shutdown: `@PreDestroy` / `DisposableBean.destroy()`
>
> `BeanPostProcessor` is how Spring AOP, `@Transactional`, `@Async`, and `@Cacheable` all work — they wrap beans in proxies at step 6.

---

**Q: What changes when migrating to Spring Boot 3 / Spring 6, and what breaks on upgrade? (the Jakarta namespace question)**

> This is a near-guaranteed interview question for any Boot 3 resume. Spring Boot 3 / Spring Framework 6 ship three breaking baselines:
>
> 1. **`javax.*` → `jakarta.*` namespace.** The biggest one. When Java EE moved to the Eclipse Foundation as Jakarta EE, Oracle's trademark forced a package rename. Every EE-derived import changes: `javax.persistence.*` → `jakarta.persistence.*` (JPA: `@Entity`, `@Id`, `@Column`), `javax.validation.*` → `jakarta.validation.*` (`@NotNull`, `@Valid`), `javax.servlet.*` → `jakarta.servlet.*` (filters, `HttpServletRequest`), `javax.annotation.*` → `jakarta.annotation.*` (`@PostConstruct`, `@PreDestroy`). What breaks: code won't compile until imports are updated, and **any third-party library still on `javax.*` is incompatible** — you must bump every dependency to a Jakarta-aware version. IDE "organize imports" doesn't fully handle it; use the OpenRewrite recipe or Spring's migrator.
>
> 2. **JDK 17 baseline.** Spring Boot 3 requires Java 17 minimum (Boot 2.x ran on Java 8). You must upgrade the JDK, which can surface issues with reflection-heavy libraries and the strong module encapsulation introduced after Java 8.
>
> 3. **Observability overhaul.** Spring Cloud Sleuth is gone — tracing moves to **Micrometer Tracing** (with Micrometer Observation as the unified metrics+tracing API). `spring-cloud-sleuth` dependencies must be replaced with `micrometer-tracing-bridge-otel` (or `-brave`).
>
> Other notable changes: `application.properties` deprecations are enforced, Hibernate 6 (with stricter SQL/dialect behaviour), and `@ConfigurationProperties` constructor binding tightened. Practical upgrade order: bump JDK to 17 → migrate `javax`→`jakarta` (OpenRewrite) → upgrade all third-party deps to Jakarta versions → swap Sleuth for Micrometer Tracing → run the test suite.

---

## Spring Cloud & Service Communication

**Q: What is Spring Cloud Gateway and how is it different from Zuul?**

> Spring Cloud Gateway is built on Spring WebFlux (reactive, non-blocking, Netty). Zuul 1.x is blocking (Servlet-based). Gateway supports predicates (route matching rules) and filters (pre/post processing) in a functional style.
>
> In an API Gateway microservice, Gateway handles: JWT validation filter (pre-filter), routing to downstream services, rate limiting via `RequestRateLimiter` filter backed by Redis, and response header stripping.

---

**Q: How does service discovery work with Spring Cloud and Eureka?**

> Each service registers itself with Eureka Server on startup (heartbeat every 30s). Clients use `DiscoveryClient` or load-balanced `RestTemplate`/`WebClient` annotated with `@LoadBalanced`. Spring Cloud LoadBalancer resolves service names to actual instances from the Eureka registry and applies a load-balancing strategy (round-robin by default).
>
> In Kubernetes, Eureka is often replaced by **Kubernetes Service discovery** — Spring Cloud Kubernetes reads from the K8s API directly. No separate Eureka server needed.

---

**Q: What is Feign and why use it over `RestTemplate`?**

> Feign is a declarative HTTP client — you define an interface with `@FeignClient("service-name")` and annotate methods with `@GetMapping`, etc. Spring generates the implementation at runtime.
>
> Advantages over `RestTemplate`: no boilerplate URL construction, integrates with Eureka/LoadBalancer automatically, built-in support for fallbacks via Resilience4j, and Feign interceptors for adding auth headers globally. `WebClient` is the reactive alternative for non-blocking calls.

---

**Q: How does Circuit Breaker pattern work with Resilience4j in Spring Boot?**

> A circuit breaker has three states:
> - **Closed**: requests flow normally, failures are counted
> - **Open**: after failure threshold (e.g., 50% of last 10 calls failed), all requests fail fast
> - **Half-Open**: after a wait duration, lets a probe request through; if it succeeds, closes the circuit
>
> ```yaml
> resilience4j.circuitbreaker:
>   instances:
>     dataService:
>       slidingWindowSize: 10
>       failureRateThreshold: 50
>       waitDurationInOpenState: 10s
> ```
> Use with `@CircuitBreaker(name = "dataService", fallbackMethod = "fallback")`. Combine with `@Retry` and `@Bulkhead` for full resilience.

---

## Microservices Patterns

**Q: How did you handle inter-service communication in your 5-microservice architecture?**

> REST for synchronous calls between API Gateway → Data/Visualization services. For the user management migration to event-driven, async messaging so the Authorization service could publish events consumed by the Notification service — this gave a 30% efficiency boost. For resilience, Spring Cloud's circuit breaker pattern prevents cascade failures.

---

**Q: What is the difference between API Gateway pattern and BFF (Backend for Frontend)?**

> An API Gateway is a single entry point handling cross-cutting concerns (auth, rate limiting, routing). A BFF is a specialized gateway per client type (mobile BFF vs web BFF), tailored to that client's data needs. A BFF is appropriate when different clients (Vue.js web vs mobile) have very different data requirements.

---

**Q: How do you handle distributed transactions across microservices?**

> Avoid 2PC (too slow, couples services). Instead use the **SAGA pattern**: either choreography (services emit events and react) or orchestration (a saga orchestrator drives the flow). For data consistency without transactions, use **eventual consistency** + idempotent operations + outbox pattern (write events to DB in the same transaction as the domain change, then publish from the outbox).

---

**Q: What is the Saga pattern and when would you use choreography vs. orchestration?**

> Sagas replace distributed transactions by breaking them into local transactions, each publishing an event. If a step fails, compensating transactions undo previous steps.
>
> **Choreography**: each service listens for events and reacts. No central coordinator — loose coupling, but hard to visualize the overall flow and debug failures.
>
> **Orchestration**: a saga orchestrator explicitly tells each service what to do and handles failures. Easier to reason about but introduces a central dependency.
>
> For terminal steps with no compensation needed (e.g., notifications), choreography works well. For multi-step flows (order → payment → inventory), orchestration gives better control.

---

**Q: What is the Outbox Pattern and why does it solve a real problem?**

> **Problem**: you save an entity to DB and publish an event to a message broker. If the app crashes between the two, the event is lost.
>
> **Solution**: write both the entity change AND the event to the **same DB transaction** (to an `outbox` table). A separate process (poller or CDC via Debezium) reads the outbox table and publishes to the broker, then marks the row as processed. The event is guaranteed to be published because it's atomic with the domain change.

---

**Q: Explain CQRS and when it makes sense.**

> Command Query Responsibility Segregation separates write operations (Commands) from read operations (Queries) into different models — often different services or databases. The write side updates the source of truth; an event propagates the change to a denormalized, query-optimized read projection.
>
> Makes sense when: read/write loads are asymmetric, read queries are complex aggregations, or you need multiple read representations of the same data. Adds complexity — don't use for simple CRUD.

---

**Q: How do you handle configuration management across multiple microservices?**

> - **Spring Cloud Config Server**: centralized config in Git, services fetch on startup. Supports `@RefreshScope` for dynamic refresh without restart.
> - **Kubernetes ConfigMaps/Secrets**: mounted as env vars or volumes, Spring Cloud Kubernetes reads them natively.
> - **Azure Key Vault / AWS Parameter Store**: for secrets specifically.
>
> In Azure, Key Vault backed by Azure App Configuration is the production pattern — secrets never in env vars, rotated without redeployment.

---

## JWT / OAuth2 Security

**Q: How does JWT-based authentication work, and what are its security risks?**

> Client authenticates → server issues a signed JWT (header.payload.signature). On subsequent requests, the client sends the token; the server validates the signature without hitting the DB.
>
> **Risks:**
> - **No revocation**: JWTs are valid until expiry — mitigate with short expiry + refresh tokens + Redis token blacklist
> - **Algorithm confusion attacks**: always explicitly specify `HS256`/`RS256`, never `alg: none`
> - **Sensitive data in payload**: it's base64-encoded, not encrypted — use JWE if needed

---

**Q: You reduced sign-in latency by 60% with OAuth2/JWT. How?**

> Previously, every request hit the DB to validate session tokens. By switching to stateless JWT validation — verifying the signature locally using the public key — the DB roundtrip on each request was eliminated. Redis is kept for the token blacklist for logout/revocation scenarios.

---

**Q: What is RBAC and how did you implement it at repository/table/permission levels?**

> RBAC assigns permissions to roles, not users directly. Implementation: store role-permission mappings in DB, encode roles in JWT claims, then use Spring Security's `@PreAuthorize("hasRole('...')")` or a custom `AccessDecisionManager` for fine-grained checks at repository/table/permission levels.

---

## Database & Performance

**Q: You improved query performance from 60s to 2–3s. What techniques did you use?**

> Root cause was N+1 queries — Hibernate issuing a separate query per entity relationship. Fixes: used `JOIN FETCH` in JPQL and `@EntityGraph` to eager-load in a single query. Added composite indexes on frequently filtered columns. Cached S3 pre-signed URLs in Redis with a TTL slightly under the URL expiry — repeated requests served from cache instead of re-fetching from S3 (80% S3 retrieval reduction).

---

**Q: What is Row-Level Security in PostgreSQL and when would you use it?**

> RLS lets you define policies that filter rows automatically based on the current DB user/role — the DB itself enforces multi-tenancy, not the application layer.
>
> ```sql
> CREATE POLICY tenant_isolation ON orders
>   USING (tenant_id = current_setting('app.tenant_id')::int);
> ```
> The app sets `SET app.tenant_id = 123` per connection. Advantage over app-layer filtering: security can't be bypassed by a code bug. Used in Deep Fathom for multi-tenant isolation across 50+ tables.

---

**Q: How do you size a HikariCP connection pool, and how can a slow `@Transactional` method starve it?**

> **Pool size is not "bigger is better".** The pool holds DB connections, and the database has a hard cap (`max_connections` in PostgreSQL, default ~100). Total connections across all app instances/services must stay under that cap, with headroom for migrations, admin tools, and replicas. A common starting formula is `connections ≈ (core_count * 2) + effective_spindle_count` — most OLTP services run well with a small pool (e.g. 10), because connections are held only for the brief duration of a query, not the whole request.
>
> Key HikariCP settings:
> - `maximumPoolSize` — the cap; keep `sum(maximumPoolSize across instances) < db max_connections`.
> - `connectionTimeout` (default 30s) — how long a thread waits for a free connection before failing with `SQLTransientConnectionException`.
> - `leakDetectionThreshold` (e.g. `60000`) — logs a stack trace when a connection is held longer than N ms without being returned; invaluable for finding connections that aren't closed.
> - `maxLifetime` / `idleTimeout` — recycle connections before the DB or a firewall kills them.
>
> **How a long `@Transactional` starves the pool:** a `@Transactional` method **holds its DB connection for the entire method duration**, not just the query. If that method makes a slow REST/LLM call or sleeps inside the transaction, the connection sits idle-but-checked-out. Under load, all `maximumPoolSize` connections get held by waiting transactions, new requests block on `connectionTimeout` and then fail, and throughput collapses even though the DB is idle. Fixes: keep transactions short, never do remote/network I/O inside `@Transactional`, move long work outside the tx boundary, and set `leakDetectionThreshold` to catch the offenders.

---

## Azure / Cloud

**Q: What is Azure Container Apps vs. AKS?**

> ACA is a serverless container platform (built on Kubernetes internally) — you don't manage nodes, just define scaling rules and container specs. AKS is managed Kubernetes where you control the cluster. ACA is faster to set up and cheaper for variable workloads (scales to zero). Choose AKS when you need full Kubernetes control — custom operators, specific node configurations, or complex networking.

---

**Q: Explain what Bicep IaC does and why it's preferred over ARM templates.**

> Bicep is a DSL that compiles to ARM JSON. Benefits: cleaner syntax, type safety, first-class VS Code tooling, and modular decomposition. Provisioning 30+ Azure resources (Container Apps, PostgreSQL Flexible Server, Key Vault, Redis, Front Door) using modular Bicep enables reproducible staging and FedRAMP GovCloud environments from the same codebase.

---

## LLM Integration

**Q: How did you handle long-running LLM operations (2–20 min)?**

> Synchronous HTTP isn't viable for 20-minute jobs. Pattern: **async job queue** — client submits request → server returns `202 Accepted` with a `jobId` → worker processes the LLM call asynchronously → client polls `/jobs/{id}/status` or receives WebSocket/SSE push on completion. For routing across 5+ LLM providers, a unified proxy abstracted provider differences behind a common interface, enabling intelligent routing based on cost/capability/load.

---

## Observability

**Q: How do you implement distributed tracing in microservices?**

> Use **Micrometer Tracing** (Spring Boot 3.x, replaces Sleuth). It propagates a `traceId` across service calls via HTTP headers (W3C `traceparent`). Each service adds its own `spanId`. Export to Zipkin, Jaeger, or Azure Monitor.
>
> ```xml
> <dependency>
>   <groupId>io.micrometer</groupId>
>   <artifactId>micrometer-tracing-bridge-otel</artifactId>
> </dependency>
> ```
> The `traceId` automatically appears in logs via SLF4J MDC — correlates a slow request across API Gateway → Authorization → Data service chain.

---

**Q: What Spring Boot Actuator endpoints are most useful in production?**

> - `/actuator/health` — liveness/readiness probes for Kubernetes
> - `/actuator/metrics` — Micrometer metrics, scraped by Prometheus
> - `/actuator/info` — build info (git commit, version)
> - `/actuator/loggers` — change log levels at runtime without restart
> - `/actuator/env` — inspect resolved property values (restrict in prod)
> - `/actuator/circuitbreakers` — Resilience4j state per circuit
>
> Always expose only health and info publicly; gate the rest behind Spring Security or a management port.

---

## Testing Microservices

**Q: What is the difference between `@SpringBootTest`, `@WebMvcTest`, and `@DataJpaTest`?**

> - `@SpringBootTest`: loads full application context. Slow. Use for integration tests that need real wiring.
> - `@WebMvcTest(MyController.class)`: loads only web layer. Mocks service layer. Fast. Use for controller logic, request mapping, validation.
> - `@DataJpaTest`: loads only JPA layer with embedded H2. Rolls back after each test. Use for repository query testing.
>
> Use the narrowest slice that tests what you care about. `@SpringBootTest` for everything is ~10× slower.

---

**Q: How do you test interactions with external services (REST calls)?**

> - **WireMock**: spins up a mock HTTP server — best for integration tests of Feign clients
> - **MockWebServer** (OkHttp): lighter weight for `WebClient` tests
> - **`@MockBean`**: replaces a Spring bean with a Mockito mock — use to test service class in isolation without HTTP
>
> For contract testing across microservices, **Spring Cloud Contract** or **Pact** generates stubs from contracts, ensuring consumer and provider stay in sync.

---

## Deployment & Operational

**Q: How does zero-downtime deployment work in Kubernetes with Spring Boot?**

> 1. **Readiness probe**: `/actuator/health/readiness` — K8s only routes traffic once this returns 200
> 2. **Liveness probe**: `/actuator/health/liveness` — K8s restarts the pod if this fails
> 3. **Rolling update strategy**: K8s starts new pods, waits for readiness, then terminates old ones
> 4. **Graceful shutdown**: `server.shutdown=graceful` + `spring.lifecycle.timeout-per-shutdown-phase=30s` — drains in-flight requests before shutdown
>
> Without graceful shutdown configured, K8s sends SIGTERM and the app drops live requests.

---

**Q: Your CI/CD pipeline runtime dropped 57% (23→10 min). What techniques achieved that?**

> Main gains from **parallel Docker builds** — previously stages ran sequentially. Used Docker BuildKit with `--cache-from` pointing to the registry image, so unchanged layers were cache hits (60–80% hit rate with Turbo monorepo caching). Split the pipeline so unit tests, integration tests, and build ran concurrently. Used GitLab's DAG pipeline (`needs:` keyword) to express the dependency graph instead of sequential stages.

---

## Behavioral (STAR Format)

**Q: Tell me about a time you significantly improved system performance.**

> **Situation**: Data retrieval in Smart360 took 60 seconds, causing timeout errors and poor UX.
> **Task**: Optimize without changing the data model.
> **Action**: Profiled queries with Hibernate statistics, found N+1 patterns, rewrote with JOIN FETCH and EntityGraph. Added indexes on join columns. Integrated Redis caching for repeated queries. Cached AWS S3 pre-signed URLs in Redis.
> **Result**: Latency dropped to 2–3 seconds (96% improvement), S3 retrieval cut by 80%.

---

**Q: Describe a situation where you had to make a technical decision with trade-offs.**

> **Situation**: Needed to migrate user management from monolithic service to event-driven microservices.
> **Task**: Choose migration strategy with acceptable risk.
> **Action**: Chose incremental strangler-fig approach — extracted the notification boundary first (most independent), validated the event-driven model, then expanded. This avoided a big-bang rewrite.
> **Result**: 30% efficiency boost delivered incrementally with controlled risk.

---

*Last updated: 2026-06-04*
