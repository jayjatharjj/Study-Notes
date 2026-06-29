# Week 2 — Interview Question Answers

> Attempt each one aloud first, then check yourself here. Week 2 covers binary search, backtracking, stacks/linked lists, Spring core + `@Transactional`, concurrency, and REST/JWT. (Spring Boot 3 / Java 17 baseline.)

## Day 1 — Binary Search + Search-on-Answer + Spring Bean Lifecycle

1. **Why `left + (right - left) / 2` instead of `(left + right) / 2`?** `left + right` can exceed `Integer.MAX_VALUE` and wrap negative, yielding a bad index. The subtraction form never overflows. Gotcha: this was a real JDK `Arrays.binarySearch` bug fixed only in 2006.

2. **`while (left <= right)` vs `while (left < right)`?** Use `<=` for exact-match search where you discard `mid` (`right = mid - 1`); use `<` for first-true/boundary search where `mid` stays a candidate (`right = mid`). Mixing the two causes off-by-one or infinite loops.

3. **Binary search a rotated array — the O(1) decision?** One half is always sorted. Compare `nums[left]` to `nums[mid]` to identify the sorted half, then test whether `target` falls in that half's range; otherwise recurse into the other. The pivot lives in the unsorted half.

4. **What changes with duplicates in a rotated array?** When `nums[left] == nums[mid] == nums[right]` you can't tell which half is sorted, so shrink with `left++; right--`. Worst case degrades to O(n). Gotcha: only this exact triple-equality case needs the fallback.

5. **What is binary-search-on-answer?** Searching a *monotonic answer space* with an O(n) feasibility test (Koko, Ship Packages). Real example: "minimum thread-pool size such that p99 < 2s" — load-test `feasible(size)` over `[1, maxThreads]`.

6. **What phrasings signal it?** "minimum maximum", "maximum minimum", "smallest largest sum", "fewest/most X to achieve Y", "largest X such that Y" — anything optimizing a value over a monotonic predicate.

7. **Why is the lower bound `max(weights)` in Ship Packages, not 1?** A daily capacity below the heaviest single package can never ship that package, so every value under `max(weights)` is infeasible and can't be the answer.

8. **"Binary search isn't only for sorted arrays" — explain.** It works on any monotonic predicate. A sorted array is just one monotonic space; the general form searches the smallest input where `feasible()` flips false→true.

9. **What is IoC and how does DI implement it?** IoC inverts *who builds collaborators* — the container, not your code. DI is the delivery mechanism: the container injects the dependencies you declared (constructor/setter/field).

10. **ApplicationContext vs BeanFactory — three real differences.** `ApplicationContext` adds eager singleton init (fail-fast at startup), automatic `BeanPostProcessor` registration (enables AOP/`@Transactional`), and event publishing + i18n. Production always uses `ApplicationContext`.

11. **Walk the 8-step bean lifecycle — where is the AOP proxy created?** Instantiate → populate → aware callbacks → BPP-before → init (`@PostConstruct` → `afterPropertiesSet` → init-method) → **BPP-after (proxy created here by `AnnotationAwareAspectJAutoProxyCreator`)** → ready → destroy.

12. **Init-hook order?** `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → custom `init-method`. Gotcha: `@PostConstruct` is Jakarta-annotation-driven and runs first.

13. **Why constructor injection over field injection?** Gives immutable `final` fields, no half-constructed objects, mandatory deps made explicit, circular deps caught at startup, and unit-testable with plain `new` (no reflection). See [interview-qa.md](../../interview-qa.md) and module [03-oop-advanced.md](../../03-oop-advanced.md).

14. **`@Primary` vs `@Qualifier`?** `@Primary` is a default tie-breaker when multiple candidates match; `@Qualifier("name")` is an explicit per-injection selection. Use `@Qualifier` for intentional multi-impl (primary/replica `DataSource`).

15. **How does Spring resolve a circular dependency?** Field/setter cycles resolve via the three-level singleton cache (early reference exposure). Constructor cycles cannot — Spring throws `BeanCurrentlyInCreationException` at startup, which is the correct signal to redesign.

16. **What does `@Lazy` do, and when use it?** Delays instantiation until first access; usable as a last-resort cycle-breaker. Cost: first-call latency and init failures surfacing late instead of at startup.

17. **Is a Spring singleton thread-safe?** No — the container only guarantees one instance, not thread safety. Stateless singletons are safe; mutable shared state needs synchronization. See [06-concurrency-and-collections.md](../../06-concurrency-and-collections.md).

18. **After step 6, what does another bean get when it `@Autowired`s a `@Transactional` service?** The CGLIB proxy, not the raw target. Gotcha: that's why `this.method()` self-calls bypass the proxy and skip the transaction (Day 2's topic).

## Day 2 — Recursion / Backtracking + @Transactional Internals

1. **What makes backtracking different from plain recursion?** The *un-choose* (state-restore) step after the recursive call — it resets shared state so sibling branches start clean.

2. **Why `new ArrayList<>(path)` when storing a result?** `path` keeps mutating after the add; storing the reference would let later mutations corrupt every saved result. Copy defensively.

3. **`start` index vs `used[]` array?** `start` for order-insensitive combinations (forward-only choice avoids duplicate combos); `used[]` for order-sensitive permutations (every position reconsiders all elements).

4. **Why does Combination Sum recurse with `i` but Combination Sum II with `i+1`?** `i` allows reusing the same element (unlimited supply); `i+1` consumes each candidate exactly once.

5. **Why `if (i > start && a[i] == a[i-1]) continue;` and not `i > 0`?** You only skip duplicate *siblings at the same tree level*. The same value may legitimately appear deeper (parent→child), so the guard is relative to `start`, not the array origin.

6. **Complexity of Subsets, Permutations, Combination Sum?** O(2ⁿ·n), O(n!·n), and O(2^(target/min)·depth) respectively — the `·n`/`·depth` is the cost of copying each result.

7. **How is `@Transactional` implemented under the hood?** An AOP proxy (CGLIB or JDK) created at bean-lifecycle step 6; a `TransactionInterceptor` runs around the method: begin → invoke → commit on success / rollback on error.

8. **CGLIB vs JDK dynamic proxy — when each, and CGLIB's constraints?** CGLIB subclasses the target (default in Boot 2.x+, even for interfaces) and needs a non-final class with non-private/non-final methods. JDK proxies implement interfaces only. `getClass()` reveals the CGLIB `$$` subclass name.

9. **`@Transactional` isn't rolling back — debug it.** Check: checked vs runtime exception (`rollbackFor`); self-invocation via `this`; private/final method; a try/catch swallowing the exception; wrong `PlatformTransactionManager`. Tool: `logging.level.org.springframework.transaction=TRACE`.

10. **Self-invocation pitfall and three fixes.** `this.method()` hits the raw target, bypassing the proxy → no tx. Fixes: inject self, call via `getBean(...)`, or move the method to a separate bean (cleanest).

11. **Why can't a `@Transactional` op start inside an `@Async` method whose caller had a tx?** `TransactionSynchronizationManager` is `ThreadLocal`-based; the async thread inherits no tx. Commit the `jobId` before dispatch; let the worker open a fresh tx.

12. **`REQUIRED` vs `REQUIRES_NEW` vs `NESTED`?** `REQUIRED` joins or starts one tx; `REQUIRES_NEW` suspends the outer and runs an independent tx with its own commit; `NESTED` uses a savepoint inside the outer tx (rollback to savepoint only).

13. **Where would you use `REQUIRES_NEW`?** Audit logs, idempotency rows, or outbox entries that must persist even when the business tx rolls back.

14. **What does `readOnly = true` actually do?** Hibernate skips dirty checking, Spring hints `Connection.setReadOnly(true)` (HikariCP may route to a replica), and Postgres skips read-only overhead — a real latency win on read-heavy queries.

15. **Default `rollbackFor` behavior?** Rolls back only on `RuntimeException`/`Error`. Checked exceptions commit by default — add `rollbackFor = Exception.class` to roll back on them. See [04-exception-handling-and-memory.md](../../04-exception-handling-and-memory.md).

16. **What anomaly does each isolation level prevent?** READ_COMMITTED → dirty reads; REPEATABLE_READ → + non-repeatable reads; SERIALIZABLE → + phantom reads. Higher isolation = more locking/cost.

17. **(Curveball) "Just put `@Transactional` on everything."** Push back: long txs starve the connection pool (worse with HTTP/LLM mid-tx), cause unintended `REQUIRED` joins, and waste locks on reads. Annotate at the service boundary; `readOnly = true` for queries.

18. **Why is calling an LLM/HTTP API inside a transaction dangerous?** The DB connection is held for the whole external call; under load the pool exhausts. Do external calls *outside* the tx boundary.

19. **Can `@Transactional` go on a private method?** No — silently ignored, because CGLIB can't override private (or final) methods. No error, just no transaction.

20. **Draw the proxy call stack.** caller → CGLIB proxy → `TransactionInterceptor` (begin tx, bind ThreadLocal connection) → actual bean method → commit/rollback on return/throw.

## Day 3 — Stacks + Monotonic Stack + Spring Boot Autoconfig & Request Lifecycle

1. **Monotonic stack in one sentence.** A stack kept in increasing or decreasing order; when a new element breaks the order you pop and resolve the popped elements, giving O(n) amortized solutions for next-greater / span / histogram problems.

2. **Why `ArrayDeque` instead of `java.util.Stack`?** `Stack` extends `Vector` (synchronized → slower) and iterates bottom-to-top oddly; `ArrayDeque` is unsynchronized, array-backed, and the official recommendation. See [07-collections.md](../../07-collections.md).

3. **Why is a monotonic-stack pass O(n) despite the nested while loop?** Each element is pushed once and popped once over the whole pass → amortized O(1) per element, O(n) total.

4. **Classic bug in Largest Rectangle in Histogram?** Forgetting to drain the stack after the main loop. Fix with a sentinel 0-height bar appended at the end so everything gets resolved.

5. **How handle the circular array in Next Greater Element II?** Iterate `2n` indices using `i % n`, pushing only during the first lap, so wrap-around elements resolve without double-counting.

6. **Min Stack with O(1) getMin?** Store the running min alongside each pushed value (a pair, or a parallel min-stack); pop both together so the min is always current.

7. **Queue with two stacks — complexity?** Push onto `in`; on poll, if `out` is empty, drain `in` into `out`, then pop `out`. Amortized O(1) per op (each element moves at most once).

8. **How does Spring Boot decide which beans to auto-configure?** It loads every class listed in `AutoConfiguration.imports`, then each `@Conditional*` annotation gates registration (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`).

9. **What replaced `spring.factories` in Boot 3?** `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` — a plain newline-delimited list of config classes.

10. **How do you debug what auto-configured (and what didn't)?** Set `debug=true` and read the CONDITIONS EVALUATION REPORT, which lists positive/negative matches with the exact gating condition.

11. **How does `@ConditionalOnMissingBean` let you override defaults?** Boot's default bean backs off when you define your own, so your bean wins — you customize simply by declaring one of the same type.

12. **What is `@SpringBootApplication` composed of?** `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` — a meta-annotation bundling the three.

13. **Walk the full request lifecycle.** Tomcat thread → SecurityFilterChain → DispatcherServlet → HandlerMapping → `preHandle` → HandlerAdapter (arg binding, `@Valid`) → controller → service proxy (`@Transactional`) → repository → `postHandle` → JSON serialization → `afterCompletion` → response.

14. **Where does `@ControllerAdvice`/`@ExceptionHandler` intercept?** Inside the DispatcherServlet *after* the handler throws, via `HandlerExceptionResolver` → `ExceptionHandlerExceptionResolver`. Gotcha: it is not a filter or servlet, so it can't catch errors thrown before the dispatcher.

15. **Filter vs HandlerInterceptor?** A `Filter` is Servlet-spec, sees the raw request/response, and runs before/after the DispatcherServlet (security, CORS); a `HandlerInterceptor` is Spring MVC and runs inside the dispatcher around the matched handler.

16. **Where does JWT validation run, and where does `@PreAuthorize`?** JWT validation runs in a servlet `Filter` in the SecurityFilterChain (before the dispatcher, URL-level); `@PreAuthorize` is AOP inside the service method (fine-grained, per-method).

17. **What does `@SpringBootApplication(exclude = ...)` do?** Disables a specific auto-configuration class — common in test slices to avoid wiring a real DB.

18. **How does a third-party starter (e.g., starter-data-redis) work?** An `@AutoConfiguration` class with `@ConditionalOn*` guards, listed in `AutoConfiguration.imports` — the same recipe you'd use for a custom starter.

19. **401 vs 403?** 401 Unauthorized = not authenticated (missing/invalid credentials); 403 Forbidden = authenticated but lacking permission.

20. **409 vs 422?** 409 Conflict = state conflict (duplicate, version mismatch); 422 Unprocessable Entity = syntactically valid but semantically invalid (validation failure).

## Day 4 — Linked Lists + LRU Cache + REST Versioning / Idempotency

1. **Why a dummy head?** It removes head-is-null and head-mutation special cases; the real head becomes `dummy.next`, and you `return dummy.next`. Cleaner add/remove with no edge branches.

2. **Reverse a list iteratively — walk the pointers.** `prev/curr/next`: save `next = curr.next`, point `curr.next = prev`, advance `prev = curr; curr = next`. O(n) time, O(1) space; return `prev`.

3. **Recursive reversal — base case and rewiring line?** Base: `head == null || head.next == null`. Rewire on the way back up: `head.next.next = head; head.next = null;`. Gotcha: O(n) stack space.

4. **How does Floyd's cycle detection work, and why must they meet?** Slow advances +1, fast +2; inside a cycle the gap shrinks by 1 each step, so they must coincide. No cycle → fast reaches null.

5. **After detecting a cycle, how do you find its start?** Reset one pointer to head; advance both +1; they meet at the cycle entry. Proof: distance relation `F = nC − a`.

6. **Remove nth from end in one pass?** Fixed-gap two-pointer: advance `fast` n steps, then move both until `fast.next == null`. A dummy head cleanly handles removing the actual head.

7. **Why a doubly-linked list for LRU, not singly?** O(1) removal needs the predecessor pointer; a singly-linked list would need an O(n) scan to find it, breaking the O(1) guarantee.

8. **Why both a HashMap and a DLL?** HashMap gives O(1) lookup by key; the DLL gives O(1) reorder and tail eviction. Neither alone gives O(1) for both lookup and recency ordering.

9. **What do sentinel nodes buy you?** They eliminate all null/empty-list checks in add/remove — the list always has a head and a tail node, so every real node has neighbors.

10. **`LinkedHashMap` access-order — what does the constructor flag do?** `accessOrder=true` moves an entry to the end on `get`; override `removeEldestEntry` to auto-evict the LRU entry. A built-in LRU in ~5 lines. See [07-collections.md](../../07-collections.md).

11. **Make LRU thread-safe — what changes?** Wrap operations in a lock: the get-then-move is a compound op, so `ConcurrentHashMap` alone isn't enough. Alternatives: striped locks or `Caffeine`.

12. **401 vs 403?** 401 = not authenticated (missing/bad token); 403 = authenticated but not authorized.

13. **409 vs 422 — when each?** 409 = state conflict (stale version, duplicate key); 422 = well-formed but semantically invalid payload.

14. **Offset vs cursor pagination on a 10M-row table?** Offset is simple but O(offset) and inconsistent under concurrent inserts; cursor (keyset) is O(log n) and stable — use cursor for feeds.

15. **DELETE /orders/123 called twice — 404 or 204?** Both defensible since idempotency means the same final state; 204 on the second call gives a cleaner client experience (no spurious error).

16. **Make POST /transfers safe to retry.** Require an `Idempotency-Key` header; store key→result in Redis with TTL; replays return the cached result without re-charging. Claim the key atomically with `SET NX` to avoid double-execution on concurrent retries.

17. **Which HTTP methods are idempotent, and why does it matter?** GET, PUT, DELETE. It matters because clients, proxies, and load balancers may safely auto-retry them without side effects.

18. **Version an API without breaking clients?** Prefer additive changes; if breaking, run versions side-by-side and signal end-of-life with `Deprecation`/`Sunset` headers.

19. **Why is PATCH's idempotency debated?** It depends on semantics: `{set: x}` is idempotent, `{increment: x}` is not — PATCH carries no inherent guarantee.

20. **How does Redis implement LRU, and how does it relate to your hand-rolled cache?** `maxmemory-policy allkeys-lru` *approximates* LRU via random sampling for performance; the DLL+HashMap model is the exact form Redis trades precision against.

## Day 5 — Mixed DSA Review + JWT / OAuth2 + Global Exception Handling

1. **Three parts of a JWT — which is secure?** `header.payload.signature`; only the signature provides integrity. The payload is base64-encoded, not encrypted — readable by anyone. Never put secrets in it.

2. **HS256 vs RS256 — when each?** HS256 (shared secret) for a monolith; RS256 (private sign / public verify) for multi-service so verifiers hold only the public key, never the signing key.

3. **Walk through token validation on a protected request.** Extract the Bearer token → verify signature + `exp` locally (no DB) → optional O(1) blacklist check → set `SecurityContextHolder`.

4. **Why is local validation a latency win?** No per-request DB/session lookup; signature verification is CPU-only. Gotcha: the trade-off is you can't instantly revoke without a blacklist.

5. **How do you handle logout with a stateless JWT?** Add the `jti` to a Redis blacklist with TTL = remaining token lifetime; delete the refresh token; check the blacklist per request.

6. **What is refresh-token rotation and why?** Issue a new refresh token on each use and invalidate the old one; a replayed old token (within a token family) signals theft → revoke the whole family.

7. **Where should access vs refresh tokens live in a browser?** Access token in JS memory (more XSS-resistant than `localStorage`); refresh token in an `HttpOnly; Secure; SameSite=Strict` cookie.

8. **Explain the algorithm-confusion attack and the fix.** Attacker forces `alg: none` or downgrades RS256→HS256 (using the public key as the HMAC secret). Fix: pin the expected algorithm in the parser; never trust the token's own `alg`.

9. **OAuth2 grant types — mobile vs service-to-service?** Authorization Code + PKCE for mobile/public clients; Client Credentials for service-to-service. Avoid Implicit (deprecated) and Resource Owner Password (legacy).

10. **What does PKCE protect against?** Authorization-code interception on public clients that have no client secret — it binds the code exchange to a per-request verifier.

11. **RBAC at API/service/DB — what breaks with only one layer?** API-only is bypassed by a bug or a new caller; the service layer adds defense-in-depth; DB row-level security protects against a compromised service leaking cross-tenant data.

12. **Where does `@ControllerAdvice` intercept in the request flow?** After the controller throws, inside `DispatcherServlet` via `HandlerExceptionResolver` → `ExceptionHandlerExceptionResolver`. Not a filter.

13. **A bad JWT throws before the controller — does `@ControllerAdvice` catch it?** No — it's thrown in the `SecurityFilterChain`; `ExceptionTranslationFilter` handles it and returns 401/403. `@ControllerAdvice` only sees exceptions inside the dispatcher.

14. **Why must the generic exception handler never return `ex.getMessage()`?** It leaks internals (SQL, class names, file paths) useful to an attacker. Log server-side; return a generic message + error code. See [04-exception-handling-and-memory.md](../../04-exception-handling-and-memory.md).

15. **400 or 422 for a validation failure?** Defensible either way: 400 if you treat the body as a malformed request, 422 if syntactically valid but semantically wrong. Pick one and stay consistent across the API.

16. **What triggers `MethodArgumentNotValidException`?** `@Valid` on a `@RequestBody` parameter when a Bean Validation constraint fails — Spring throws it before the controller body runs.

17. **`@Valid` vs `@Validated`?** `@Valid` is the JSR-380 trigger (incl. cascading into nested objects); `@Validated` is Spring's variant adding validation groups and method-level validation.

18. **How do you write a custom validation rule?** A `@Constraint(validatedBy = ...)` annotation plus a `ConstraintValidator<A, T>` implementation. See [05-enums-and-annotations.md](../../05-enums-and-annotations.md).

19. **Why does Spring Security break in WebFlux, and how is it fixed?** `SecurityContextHolder` is `ThreadLocal`-backed but reactive chains hop threads. Use `ReactiveSecurityContextHolder`, which stores context in the Reactor `Context`.

20. **What's RFC 7807, and why use it?** A standard `ProblemDetail` error format (`type`, `title`, `status`, `detail`, `instance`) giving consistent, machine-readable errors across services. Spring Boot 3 supports it natively via `ProblemDetail`.

## Day 6 — Timed DSA + Concurrency Deep-Dive + LLD Primer

1. **`synchronized` vs `ReentrantLock` — when ReentrantLock?** When you need `tryLock(timeout)`, interruptible locking, fairness, or multiple `Condition`s. Example: `tryLock(5, SECONDS)` to return 503 instead of blocking under load. See [06-concurrency-and-collections.md](../../06-concurrency-and-collections.md).

2. **What does `volatile` guarantee, and what doesn't it?** Visibility and ordering (happens-before), not atomicity. Fine for a shutdown flag; wrong for `counter++`.

3. **Why is `counter++` unsafe even when `counter` is `volatile`?** It's a read-modify-write; two threads can read the same value and both write the same incremented result, losing an update. Use `AtomicInteger`.

4. **`volatile` vs `synchronized` vs `Atomic*` vs `ConcurrentHashMap`?** Visibility-only / mutual exclusion / lock-free CAS / lock-free reads with per-bucket-CAS writes. Pick the cheapest tool that meets the guarantee you need.

5. **Default executor of `CompletableFuture.supplyAsync` — why a trap?** It's `ForkJoinPool.commonPool()`, shared with parallel streams; a slow blocking call starves it for the whole JVM. Pass an explicit `Executor`.

6. **`thenApply` vs `thenCompose`?** `thenApply` maps a value (function returns a plain value); `thenCompose` flat-maps a function that itself returns a `CompletableFuture`, avoiding nested `CompletableFuture<CompletableFuture<T>>`.

7. **`future.get()` vs `future.join()`?** `get()` throws checked `ExecutionException`/`InterruptedException`; `join()` throws unchecked `CompletionException` — cleaner inside lambda/stream chains.

8. **How does `ThreadLocal` interact with Tomcat's thread pool?** Threads are reused, so an un-removed value leaks to the next request → cross-tenant data leak plus memory leak. Always `remove()` in a `finally`.

9. **Does `ThreadLocal` propagate to `@Async` threads?** No — it's a different thread. Pass the value explicitly. Gotcha: `InheritableThreadLocal` copies only at thread *creation*, so it doesn't help with pooled threads.

10. **Write a deadlock and prevent it.** Two threads acquiring two locks in opposite order. Fix: global lock ordering (e.g. sort by resource id) or `tryLock` with backoff.

11. **Name the four Coffman conditions.** Mutual exclusion, hold-and-wait, no preemption, circular wait — breaking any one prevents deadlock.

12. **Tomcat gets 1000 concurrent requests — what happens at the pool level?** 200 worker threads (default `server.tomcat.threads.max`), then the `accept-count` queue (default 100), then connections are refused. Scale replicas before that point.

13. **How would you size a thread pool for LLM jobs?** Little's Law: concurrency = throughput × latency. ~5 req/min × 8 min ≈ 40 → `corePoolSize=40, maxPoolSize=60`, a bounded queue, plus a saturation metric.

14. **`ForkJoinPool` vs a dedicated `ThreadPoolExecutor`?** ForkJoin (work-stealing) suits CPU-bound divide-and-conquer; a dedicated pool for blocking I/O keeps it from starving the common pool.

15. **Parking Lot — what classes, and where does `canFit` live?** `ParkingLot` / `Level` / `Spot` / `Vehicle` / `Ticket`; `canFit(Vehicle)` belongs on `Spot` (it owns the size constraint).

16. **Which SOLID principles does injecting `ParkingStrategy`/`PricingStrategy` serve?** DIP (depend on interfaces, not concretes) and OCP (a new strategy is a new class, no edits to existing code).

17. **Add EV charging without breaking existing code — how?** A new `SpotType` plus a `canFit` rule and/or a new strategy implementation — OCP, with no edits to existing classes.

18. **Concurrent gates parking at once — avoid double-booking without serializing everything?** Use a per-`Level` lock or a `ConcurrentHashMap` of availability with CAS, rather than one lot-wide `synchronized` block.

19. **Where do AOP/`@Transactional` proxies appear in the bean lifecycle?** Step 6, `postProcessAfterInitialization`, created by `AnnotationAwareAspectJAutoProxyCreator`; the injected bean *is* the proxy.

20. **Connect `ThreadLocal` to a security feature you've used.** `SecurityContextHolder` is `ThreadLocal`-backed; the same pattern backs a `TenantContext` — and both must be `remove()`d in a `finally` to avoid leaking across pooled requests.

---

*[← Week 2](README.md)*
