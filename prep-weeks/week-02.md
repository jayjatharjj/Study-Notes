# Week 2 — Foundations (Jul 6–11, 2026)

> Master the bedrock every Java interview tests first: binary search and its on-answer variant, recursion/backtracking, the linear structures (stacks, queues, linked lists), and the Spring internals + concurrency model behind your own shipped code.
>
> **Full-time, heads-down study week — Mon–Sat (6 study days), Sun rest.** Three blocks per day: **Block A** (DSA, ~2.5 hr), **Block B** (core/system knowledge, ~2 hr), **Block C** (a second DSA set or deep-dive, ~1.5 hr). No applications this week — pure skill build.

---

## 🎯 Week Goal

Walk into any interview and answer cold questions on Spring IoC/DI, `@Transactional`, bean lifecycle, auto-configuration, the full request lifecycle, and Java concurrency without hesitation. Solve any binary-search, backtracking, stack/monotonic-stack, or linked-list medium cold in ≤25 min. Implement an LRU cache and binary-search-on-answer under time pressure. Explain REST/JWT/OAuth2 production knowledge and articulate the concurrency and transaction decisions you made in WebX and Smart360 at senior-engineer depth.

---

## ✅ By Saturday you can...

- Implement iterative and recursive binary search from scratch and explain the off-by-one logic (`left <= right` vs `left < right`)
- Solve Search in Rotated Sorted Array, Koko Eating Bananas, and Search a 2D Matrix correctly on first attempt
- Explain binary-search-on-answer and identify when a problem needs it (Koko, Ship Packages, Split Array Largest Sum)
- Generate Subsets, Permutations, and Combination Sum via backtracking with correct state-restore and de-duplication
- Solve Valid Parentheses, Min Stack, Daily Temperatures, Next Greater Element (I/II), Online Stock Span, and Largest Rectangle in Histogram with the monotonic-stack model
- Solve Reverse/Merge Linked List, Floyd's cycle detection, Reorder List, Remove Nth from End; implement an LRU cache (HashMap + DLL) cold
- Whiteboard the full Spring request lifecycle: Servlet container → DispatcherServlet → HandlerMapping → Interceptors → Controller → Service (proxy) → Repository → back
- Explain `@Transactional` proxy mechanics, propagations, and the self-invocation pitfall with a fix
- Distinguish ApplicationContext vs BeanFactory and all 8 bean-lifecycle steps (incl. where AOP proxies appear)
- Explain auto-configuration: `AutoConfiguration.imports`, `@ConditionalOnClass`, `@ConditionalOnMissingBean`, and how to debug what fired
- Explain REST versioning, pagination (offset vs cursor), and idempotency keys; write a `@ControllerAdvice` from memory
- Describe JWT issuance → validation → refresh → RBAC and the OAuth2 grant types end-to-end
- Compare `synchronized` vs `ReentrantLock`, explain `volatile`, write and prevent a deadlock, and connect `CompletableFuture`/`ThreadLocal` to WebX

---

## 📅 Daily Checklist

---

### Monday Jul 6 — Binary Search + Search-on-Answer

📌 **Study today:** Binary search — standard + rotated + 2D (LC 704, 33, 153, 74) · Spring bean lifecycle + IoC/DI · binary-search-on-answer (LC 875, 1011, 410)

**DSA (Block A, ~2.5 hr) — Binary Search Fundamentals:**

Pattern: **Binary search**. Memorize the invariant before coding: `while (left <= right)` for exact search, `mid = left + (right - left) / 2`, shrink with `left = mid + 1` / `right = mid - 1`. Use `left + (right - left) / 2` (not `(left + right) / 2`) to avoid integer overflow when `left + right > Integer.MAX_VALUE` — mention this in interviews.

Problems (in order):
1. **Binary Search** (LC 704) — implement iteratively from scratch, no looking up. Test even/odd length arrays and target-not-found. Target: <5 min.
2. **Search in Rotated Sorted Array** (LC 33) — work both cases: pivot left of mid, pivot right of mid. Key insight: one of the two halves is always sorted — compare `nums[mid]` to `nums[left]`/`nums[right]` to decide which half holds the target in an O(1) decision.
3. **Find Minimum in Rotated Sorted Array** (LC 153) — no target, find the inflection point. Invariant: `nums[mid] > nums[right]` means the pivot is in the right half.
4. **Search a 2D Matrix** (LC 74) — row-sorted, each row starts > previous row's end → treat as one flat sorted array. `mid` maps to `(mid / cols, mid % cols)`. Target: <10 min.

**Core Topic (Block B, ~2 hr) — Spring Bean Lifecycle + IoC/DI:**

- **ApplicationContext vs BeanFactory**: `BeanFactory` = lazy init by default, minimal. `ApplicationContext` extends it and adds eager singleton init, event propagation (`ApplicationEventPublisher`), i18n (`MessageSource`), AOP proxy post-processing, `@Async`/`@Scheduled` support. Always use `ApplicationContext` in practice. Hierarchy: `BeanFactory` → `ListableBeanFactory` → `ApplicationContext` → `ConfigurableApplicationContext` → `AnnotationConfigApplicationContext`/`GenericWebApplicationContext`.
- **DI modes**: constructor (recommended — immutable, easy to test, surfaces cycles at startup), setter (optional deps), field (`@Autowired` on field — avoid: hides dependencies, breaks in non-Spring tests).
- **`@Primary` vs `@Qualifier`**: `@Primary` sets a default for ambiguity; `@Qualifier("beanName")` is explicit. Use `@Qualifier` for multiple intentional implementations (e.g., two `DataSource` beans for primary/replica routing in Smart360).
- **The 8-step bean lifecycle** — whiteboard all phases. Crucial precision: AOP proxies are created at step 6 (`BeanPostProcessor.postProcessAfterInitialization`) by `AnnotationAwareAspectJAutoProxyCreator` (itself a `BeanPostProcessor`). After step 6 the bean in the context IS the proxy — the original is wrapped, not replaced.
- **Init hooks and order**: `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → `init-method`. The `BeanPostProcessor` wraps around all of them.
- **Circular dependency**: constructor injection fails fast at startup; field/setter resolves lazily (partial bean). Constructor injection is safer — surfaces cycles at boot. Best fix: redesign to remove the cycle. `@Lazy` delays instantiation until first use (last-resort cycle break; trade-off: first-call latency, hidden init failures).

**DSA (Block C, ~1.5 hr) — Binary-Search-on-Answer:**

When the answer space is a monotonic range and you can write a feasibility function `feasible(mid)`, binary-search the answer itself, not an array index. Template:
```
left = min_possible_answer
right = max_possible_answer
while left < right:
    mid = left + (right - left) / 2
    if feasible(mid):
        right = mid        // or left = mid + 1 depending on direction
    else:
        left = mid + 1
return left
```
1. **Koko Eating Bananas** (LC 875) — `canEat(speed)` = can Koko finish all piles in `h` hours? Answer space `[1, max(piles)]`. Target: <15 min.
2. **Ship Packages Within D Days** (LC 1011) — same pattern; feasibility = can all packages ship within `D` days at this capacity? Answer space `[max(weights), sum(weights)]`.
3. **Split Array Largest Sum** (LC 410) — minimize the largest subarray sum across `k` splits; feasibility = can we split into ≤ k subarrays each ≤ mid?

**Self-check:**
1. Can you identify which half is sorted in a rotated array as an O(1) decision? If not, re-code before Tuesday.
2. Typical phrasings that signal binary-search-on-answer? (Answer: "minimum maximum", "maximum minimum", "largest X such that Y", "fewest/most X to achieve Y".)

---

### Tuesday Jul 7 — Recursion / Backtracking

📌 **Study today:** Backtracking — choose/recurse/un-choose (LC 39, 40, 46, 78) · `@Transactional` internals (propagation, isolation, self-invocation, proxy) · more backtracking (LC 79, 51, 90)

**DSA (Block A, ~2.5 hr) — Backtracking Core:**

Pattern: **Backtracking — choose → recurse → un-choose**. The un-choose (state restore) step distinguishes backtracking from naive recursion; forget it and you corrupt the path for sibling branches.
```
backtrack(path, choices):
    if base case: add copy of path to results; return
    for each choice:
        path.add(choice)        // choose
        backtrack(path, ...)     // recurse
        path.remove(last)        // un-choose (RESTORE)
```
Problems (in order):
1. **Subsets** (LC 78) — at each index choose include/exclude, recurse, pop. Trace the tree for `[1,2,3]` on paper: leaves = 2³ = 8. Complexity O(2ⁿ × n).
2. **Permutations** (LC 46) — backtracking with a `used[]` boolean array or by swapping; practice both. Complexity O(n! × n) — n! permutations, each length n.
3. **Combination Sum** (LC 39) — same element reusable; pass a `start` index (recurse with `i`, not `i+1`) to allow reuse while preventing duplicate combinations. Sort first; if `candidates[i] > target`, break the inner loop (pruning).
4. **Combination Sum II** (LC 40) — each candidate used once; add the dedup guard `if (i > start && candidates[i] == candidates[i-1]) continue;`.

**Core Topic (Block B, ~2 hr) — `@Transactional` Internals:**

- **Proxy mechanics**: CGLIB subclass proxy for classes (requires non-final class, non-private methods); JDK dynamic proxy for interfaces. Spring Boot 2.x+ defaults to CGLIB even for interface-based beans. Calling `getClass()` on an injected `@Transactional` service returns the CGLIB subclass name.
- **Propagation** — know cold: `REQUIRED` (default — join or start; 95% of cases); `REQUIRES_NEW` (suspend outer, start fresh, commit independently — audit logs, idempotency tables, outbox); `NESTED` (savepoint inside outer tx); `NOT_SUPPORTED` (suspend, run without tx); `NEVER` (throw if a tx exists).
- **Isolation**: `READ_COMMITTED` (default in Postgres), `REPEATABLE_READ`, `SERIALIZABLE` — know the anomalies each prevents (dirty / non-repeatable / phantom reads).
- **Self-invocation pitfall** — must be able to demo:
  ```java
  @Service
  public class OrderService {
      public void placeOrder() {
          this.processPayment();  // 'this' = raw bean, not the proxy → NO transaction
      }
      @Transactional
      public void processPayment() { ... }
  }
  ```
  Fixes: (1) inject self (`@Autowired private OrderService self;`); (2) `applicationContext.getBean(OrderService.class)`; (3) move `processPayment` to a separate `@Service` (cleanest).
- **Three more pitfalls to mention unprompted**: private methods → annotation silently ignored; default `rollbackFor` = `RuntimeException` only (use `rollbackFor = Exception.class` for checked exceptions); long transactions holding DB connections — never call external HTTP/LLM APIs inside a `@Transactional` method (you held a Postgres connection for a 20s LLM call in WebX → fix: do the LLM call outside the tx boundary).
- **`@Transactional(readOnly = true)`**: Hibernate skips dirty checking at flush; Spring hints `Connection.setReadOnly(true)` (HikariCP may route to a replica); Postgres skips read-only overhead — a real Smart360 latency win on a 500-entity report query.
- **TransactionSynchronizationManager** uses a `ThreadLocal<Map<Object,Object>>` to store the current connection per thread — this is why `@Transactional` is inherently thread-local and cannot span an HTTP thread and an `@Async` thread.

**DSA (Block C, ~1.5 hr) — Backtracking Variations:**

1. **Word Search** (LC 79) — grid DFS + backtracking; mark a cell visited, recurse 4 directions, then restore. Common bug: forgetting to restore the cell.
2. **N-Queens** (LC 51) — the canonical hard backtracking; track occupied columns and both diagonals (`r+c` and `r-c`) for O(1) conflict checks.
3. **Subsets II** (LC 90) — duplicates in input; sort, then skip `if (i > start && nums[i] == nums[i-1]) continue;`.

**Self-check:**
1. Why does a `start` index (not `used[]`) prevent duplicate *combinations* but `used[]` is needed for *permutations*? (Answer: combinations are order-insensitive — `start` enforces a forward-only choice; permutations are order-sensitive — every position is a fresh choice.)
2. Draw the proxy call stack on paper: caller → CGLIB proxy → `TransactionInterceptor` → actual bean method.

---

### Wednesday Jul 8 — Stacks + Monotonic Stack

📌 **Study today:** Stacks + monotonic stack (LC 20, 155, 739, 496, 503, 901, 84) · Spring Boot auto-config + full request lifecycle · queues / stack drills

**DSA (Block A, ~2.5 hr) — Stacks & Monotonic Stack:**

Pattern: **Monotonic stack** — maintain elements (usually indices) in increasing/decreasing order; pop and resolve when the current element breaks monotonicity. Used for next-greater-element, span, and histogram problems. Back the stack with `Deque<Integer> stack = new ArrayDeque<>()` — never the legacy synchronized `Stack` class.

Problems (in order):
1. **Valid Parentheses** (LC 20) — push openers, match on closers. Edge: closer on an empty stack → invalid. Target: ≤10 min.
2. **Min Stack** (LC 155) — two stacks (or value+min pairs); O(1) `getMin` by pushing the running min alongside.
3. **Daily Temperatures** (LC 739) — monotonic decreasing stack of indices; trace `[73,74,75,71,69,72,76,73]` by hand before coding.
4. **Next Greater Element I** (LC 496) — one monotonic-stack pass over `nums2` into a `Map<value, nextGreater>`, then O(1) lookups for `nums1`. The base case of the family.
5. **Next Greater Element II** (LC 503) — **circular** array; iterate `2n` indices using `i % n`; only push during the first pass.
6. **Online Stock Span** (LC 901) — the canonical **span** problem; stack of `(price, span)` pairs. On each new price, pop while top price ≤ current, accumulating spans — amortized O(1) per call.
7. **Largest Rectangle in Histogram** (LC 84, Hard) — the flagship. For each bar find the leftmost/rightmost bar ≥ its height via an index stack. Trace `[2,1,5,6,2,3]` → 10. Common mistake: not draining the stack after the loop — push a dummy 0-height sentinel at the end.

**Core Topic (Block B, ~2 hr) — Auto-Configuration + Full Request Lifecycle:**

- **Auto-configuration mechanics**: Boot 3.x uses `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (replaces `spring.factories`). Spring loads all listed classes but `@Conditional*` annotations gate each. Key conditionals: `@ConditionalOnClass(DataSource.class)`, `@ConditionalOnMissingBean(DataSource.class)`, `@ConditionalOnProperty("spring.datasource.url")`.
- **Debugging**: set `logging.level.org.springframework.boot.autoconfigure=DEBUG` (or `debug=true`) → "CONDITIONS EVALUATION REPORT" shows which auto-configs matched, which were skipped, and why. Interviewers love this.
- **Disabling one**: `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)` — used in test slices / WebX unit tests to avoid a real DB.
- **`@SpringBootApplication` is composed**: `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.
- **Custom auto-configuration**: a class annotated `@AutoConfiguration` + `@ConditionalOn*`, registered in `AutoConfiguration.imports` — this is how every starter (e.g., `spring-boot-starter-data-redis`) works.
- **Full request lifecycle** — whiteboard without notes:
  ```
  Client → HTTP request
  Tomcat (Servlet container) — assigns a thread from its pool
  → SecurityFilterChain (servlet Filter — JWT validation runs HERE, before DispatcherServlet)
  → DispatcherServlet.doDispatch()
  → HandlerMapping — URL → @RequestMapping method (HandlerExecutionChain)
  → HandlerInterceptor.preHandle()
  → HandlerAdapter — resolves @RequestBody (HttpMessageConverter/Jackson), @PathVariable, @RequestParam
  → @RestController method
  → @Service — CGLIB proxy checks @Transactional, starts tx
  → @Repository / Spring Data JPA — Hibernate session, SQL
  → response builds back up: postHandle() → HttpMessageConverter serialises JSON → afterCompletion()
  → HTTP response
  ```
  Follow-up: "Where does `@ControllerAdvice`/`@ExceptionHandler` intercept?" → `DispatcherServlet` catches the handler exception and delegates to `HandlerExceptionResolver` → `ExceptionHandlerExceptionResolver` → `@ExceptionHandler`. Not a filter, not a servlet — runs after the controller throws. `@PreAuthorize` is AOP on the service method (inside the service call).

**DSA (Block C, ~1.5 hr) — Queues + Stack Consolidation:**

- **Queue/Deque model**: `ArrayDeque` as queue (`addLast()`/`pollFirst()`); a monotonic deque underlies Sliding Window Maximum (LC 239 — preview for later weeks). `ArrayDeque` beats `LinkedList` (array-backed, no node allocation).
- Re-solve **Daily Temperatures** (LC 739) without looking at the morning's solution — target ≤18 min.
- Write time/space complexity for all the day's stack problems in your notes.

**Self-check:**
1. Explain monotonic stack in one sentence.
2. Difference between 401 and 403? Between 409 and 422? (Bridges to Thursday's REST block.)

---

### Thursday Jul 9 — Linked Lists + LRU Cache

📌 **Study today:** Linked lists — reversal, dummy head, Floyd's, two-pointer (LC 206, 21, 141, 142, 19, 143) · REST versioning + pagination + idempotency · LRU cache (LC 146)

**DSA (Block A, ~2.5 hr) — Linked Lists:**

Patterns: **dummy head** (eliminates head-is-null special cases), **two-pointer** (slow/fast for cycle and midpoint, fixed-gap for nth-from-end), and **reversal** (3 pointers: prev/curr/next). Draw boxes-and-arrows for a 3-node example on every problem.

Problems (in order):
1. **Reverse Linked List** (LC 206) — iterative (`prev=null, curr=head`, walk swapping pointers) AND recursive (base `head.next == null`); know both cold.
2. **Merge Two Sorted Lists** (LC 21) — dummy head to avoid head-special-case bugs.
3. **Linked List Cycle** (LC 141) — Floyd's tortoise/hare; slow +1, fast +2; meet iff a cycle exists.
4. **Linked List Cycle II** (LC 142) — find the cycle start: after meeting, reset one pointer to head, advance both at +1; they meet at the cycle entry.
5. **Remove Nth Node From End** (LC 19) — advance fast `n` steps, then move both until `fast.next == null`; dummy head handles removing the head.
6. **Reorder List** (LC 143) — three steps: find middle (slow/fast) → reverse second half → merge by interleaving. Most candidates skip step 2 — dry-run first.

**Core Topic (Block B, ~2 hr) — REST Versioning, Pagination, Idempotency:**

- **Resource naming**: nouns not verbs (`/users/{id}`, not `/getUser`), plurals for collections, sub-resources (`/users/{id}/orders`).
- **Status codes cold**: 200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500, 502, 503.
- **Versioning strategies**: URI path (`/v1/users` — most common/visible, breaks bookmarks); header (`Accept: application/vnd.app.v2+json` — clean URIs, less discoverable); query param (`?version=2` — easy to test, not purist). URI for public APIs (gateway routing); header for internal services. Breaking-change follow-up: `Deprecation: true` + `Sunset: <date>` headers; prefer additive (backward-compatible) changes.
- **Pagination**: offset (`?page=2&size=20` — simple, slow on large offsets since Postgres scans skipped rows); cursor/keyset (`?after=<base64 last-seen id>` — O(log n), stable under inserts/deletes; use for feeds/infinite scroll). Return total count, next cursor, `Link` headers (RFC 5988). Smart360 chose cursor for the data service because frequent inserts made offset pages inconsistent.
- **Idempotency**: an op is idempotent if N calls = same effect as one. GET ✓, PUT ✓, DELETE ✓ (404 or 204 on second call — both idempotent), POST ✗, PATCH ✗ (debated; `{increment: 5}` is not idempotent). Make POST safe with an `Idempotency-Key: <uuid>` header — server stores key → response in Redis with TTL; a duplicate request returns the cached result without re-executing (the bank-transfer retry pattern).

**DSA (Block C, ~1.5 hr) — LRU Cache:**

1. **LRU Cache** (LC 146) — a design problem disguised as DSA.
   - Interview-acceptable quick form: `LinkedHashMap` in access-order mode.
   - **Preferred full implementation**: `HashMap<Integer, Node>` + doubly-linked list. HashMap → O(1) lookup; DLL with MRU at head, LRU at tail; on `get`/`put` move the node to head; on capacity evict the tail. Sentinel head/tail nodes eliminate null checks.
   - Write the full version cold; target ≤30 min. Mention the production analogue: Redis with `EXPIRE` + `LFU`/`LRU` eviction.

**Self-check:**
1. Reverse `1→2→3→4→null` in your head → `4→3→2→1→null`. Draw the pointer diagram.
2. Trade-offs of cursor vs offset pagination on a 10M-row table?

---

### Friday Jul 10 — Mixed DSA Review + Auth/Errors

📌 **Study today:** Mixed DSA review (re-attempt week's hardest cold) · JWT lifecycle + OAuth2 + global exception handling (`@ControllerAdvice`) · weak-spot drills

**DSA (Block A, ~2.5 hr) — Mixed Review (timed, no hints):**

Re-attempt this week's hardest problems cold, 20–25 min each, talking aloud as if in a live interview:
- **Search in Rotated Sorted Array** (LC 33) and **Koko Eating Bananas** (LC 875) — target ≤10 min each.
- **Largest Rectangle in Histogram** (LC 84) — re-derive the stack invariant.
- **Reorder List** (LC 143) — the three-step pattern.
- For any you get wrong or slow on: write down exactly where you got stuck — it becomes a weak-spot drill in Block C.

**Core Topic (Block B, ~2 hr) — JWT + OAuth2 + Global Exception Handling:**

- **Token issuance**: POST credentials → `AuthService` validates (BCrypt) → build claims (`sub`, `roles`, `tenantId`, `iat`, `exp` ≈ 15 min) → sign with `HS256` (shared secret) or `RS256` (RSA private key, multi-service verify) → return `{accessToken, refreshToken, expiresIn}`; refresh token stored hashed in DB/Redis with a `tokenFamily` for rotation.
- **Validation** (every protected request): `JwtAuthenticationFilter extends OncePerRequestFilter` → extract Bearer token → validate signature locally (no DB call — the 60% latency win in Smart360) → check `exp` → check Redis blacklist (O(1)) → set `SecurityContextHolder` with roles.
- **Refresh**: POST refresh token → validate (DB lookup) → issue a new access token + rotate the refresh token (token-family rotation detects theft if the old one is replayed).
- **RBAC at three levels**: API (`@PreAuthorize("hasAuthority('READ_REPORTS')")`), service (defense-in-depth), database (Postgres RLS — `findByIdAndTenantId`, tenant isolation in queries). What breaks if only one level: a bug bypasses API-level; service-level is defense-in-depth; DB RLS means even a compromised service can't leak cross-tenant data.
- **OAuth2 grant types**: Authorization Code + PKCE (user-facing web/mobile — most secure, PKCE prevents code interception); Client Credentials (service-to-service, no user); Implicit (deprecated — never recommend); Resource Owner Password (legacy — avoid).
- **Token storage**: access token in-memory JS (not localStorage — XSS); refresh token in HttpOnly, Secure, SameSite=Strict cookie. **Algorithm confusion attack**: always pin the algorithm in the parser — never trust the token's `alg` header.
- **Logout on stateless JWT**: can't un-issue; add `jti` to a Redis blacklist with TTL = remaining lifetime; delete the refresh token. Filter-chain order: `SecurityContextPersistenceFilter` → `UsernamePasswordAuthenticationFilter` → custom `JwtAuthenticationFilter` → `ExceptionTranslationFilter` → `FilterSecurityInterceptor`.
- **Global exception handler** — write from memory:
  ```java
  @RestControllerAdvice
  public class GlobalExceptionHandler {
      @ExceptionHandler(ResourceNotFoundException.class)
      public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
          return ResponseEntity.status(HttpStatus.NOT_FOUND)
              .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
      }
      @ExceptionHandler(MethodArgumentNotValidException.class)
      public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
          List<String> errors = ex.getBindingResult().getFieldErrors().stream()
              .map(fe -> fe.getField() + ": " + fe.getDefaultMessage()).toList();
          return ResponseEntity.status(HttpStatus.BAD_REQUEST)
              .body(new ErrorResponse("VALIDATION_ERROR", errors.toString()));
      }
      @ExceptionHandler(AccessDeniedException.class)
      public ResponseEntity<ErrorResponse> handleForbidden(AccessDeniedException ex) {
          return ResponseEntity.status(HttpStatus.FORBIDDEN)
              .body(new ErrorResponse("FORBIDDEN", "Insufficient permissions"));
      }
      @ExceptionHandler(Exception.class)
      public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
          // log ex fully — never expose stack trace to the client
          return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
              .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
      }
  }
  ```
- **Bean Validation**: `@NotNull`, `@NotBlank`, `@NotEmpty`, `@Size`, `@Min/@Max`, `@Email`, `@Pattern`, `@Valid` (cascade trigger), `@Validated` (groups). `@Valid` on a parameter triggers MVC validation before the body runs; the `MethodArgumentNotValidException` is automatic. Custom validator: `@Constraint(validatedBy = MyValidator.class)` implementing `ConstraintValidator`.

**DSA (Block C, ~1.5 hr) — Weak-Spot Drills:**

- Drill the specific stuck points captured in Block A. Re-solve each from scratch.
- If clean, attempt **Spiral Matrix** (LC 54) as a bonus (useful for Amazon/Google rounds).

**Self-check:**
1. What status should a validation failure return — 422 or 400? Have an opinion and defend it.
2. Why must the generic `Exception` handler never return the exception message to the client?

---

### Saturday Jul 11 — Timed DSA + Concurrency Deep-Dive + LLD Primer

📌 **Study today:** Timed DSA set (2 cold problems) · concurrency deep-dive (synchronized/locks/volatile/CompletableFuture/ThreadLocal/deadlock) + LLD primer (parking lot + SOLID) · consolidation

**DSA (Block A, ~2.5 hr) — Timed Set:**

Treat this like a real interview — timer running, no hints, clean whiteboard-quality code.
- **Problem A** (20 min): re-solve **Koko Eating Bananas** (LC 875) or **Search a 2D Matrix** (LC 74) cold.
- **Problem B** (25 min): re-solve **Subsets** (LC 78) or **Daily Temperatures** (LC 739) cold.
- Score yourself: Did you state the approach before coding? Handle edge cases unprompted? Finish in time? Was the code compilable without fixes? Write what you'd do differently.

**Core Topic (Block B, ~2 hr) — Concurrency Deep-Dive + LLD Primer:**

**`synchronized` vs `ReentrantLock`:** `synchronized` is a JVM keyword — automatic acquire/release even on exception. `ReentrantLock` is explicit — you MUST `unlock()` in `finally` or leak the lock. `ReentrantLock` advantages: `tryLock(timeout)` (deadlock avoidance — e.g., timeout on the WebX LLM provider slots and return 503 instead of blocking forever), `lockInterruptibly()`, fairness (`new ReentrantLock(true)`), multiple condition variables. `ReentrantReadWriteLock` — many readers OR one writer; perfect for a read-heavy in-memory config map (the WebX provider routing table).

**`volatile`:** guarantees visibility + ordering, NOT atomicity. Valid: a `volatile boolean running` flag (single write/read). Invalid: `volatile int counter; counter++` (read-modify-write race — use `AtomicInteger`). JMM: a `volatile` write happens-before all subsequent `volatile` reads of the same variable.

**`CompletableFuture` (WebX async LLM jobs):** `thenApply` (transform), `thenCompose` (flat-map, avoids nested futures), `thenCombine` (combine two), `allOf`/`anyOf`, `exceptionally`/`handle`. Default `supplyAsync` uses `ForkJoinPool.commonPool()` — always pass an explicit `Executor` in production (a slow LLM call starves the shared pool). WebX pattern: each job as a `CompletableFuture` in `ConcurrentHashMap<jobId, CompletableFuture<LLMResult>>`; poll `isDone()`; `complete(result)` when the LLM responds. `join()` (unchecked `CompletionException`) is preferred over `get()` (checked) in lambda chains.

**`ThreadLocal` (Tomcat/Spring gotcha):** each thread gets an isolated copy. Tomcat = one thread per request → Spring Security's `SecurityContextHolder` stores `Authentication` in a `ThreadLocal`. CRITICAL: Tomcat reuses threads — if you store a value and forget `remove()`, the next request on that thread inherits stale state (cross-tenant data leak + memory leak). Spring Security calls `clearContext()` in its filter; always do the same for custom `ThreadLocal`s. `ThreadLocal` does NOT propagate across `@Async` boundaries — pass the value explicitly (cleanest) rather than relying on `InheritableThreadLocal`.

**Deadlock:** write from memory —
```
// Thread 1                 // Thread 2
synchronized(lockA) {       synchronized(lockB) {
    synchronized(lockB){}       synchronized(lockA){}
}                           }
// T1 holds A waits B; T2 holds B waits A
```
Four Coffman conditions: mutual exclusion, hold-and-wait, no preemption, circular wait. Most practical fix: eliminate circular wait — always acquire locks in a global order (sort resources by ID before locking).

**LLD Primer — "Design a Parking Lot" (the canonical OOD prompt).** Keep your 02/03 OOP notes open for SOLID. It's not about the right answer — it's clean responsibilities, interfaces over concretions, and naming which principle each choice serves.
- **Entities & responsibilities**: `ParkingLot` (orchestration only — `parkVehicle`/`unpark`), `Level` (owns its `Spot`s, knows availability), `Spot` (`{id, SpotType, occupied}`, `canFit(Vehicle)` lives here), `Vehicle` hierarchy (abstract → `Car`/`Truck`/`Motorcycle`/`ElectricCar`), `Ticket` (`{id, Spot, Vehicle, entryTime, exitTime}`).
- **Interfaces / seams**: `ParkingStrategy` (`Spot findSpot(...)` — `NearestFirst`/`BestFit`), `PricingStrategy` (`BigDecimal price(Ticket)` — `FlatHourly`/`TieredByVehicleType`/`WeekendSurge`); inject both into `ParkingLot`.
- **Call out the SOLID letter for each choice (this scores points)**: OCP — a new `SpotType`/`Vehicle`/strategy means a new subclass, no edits to existing classes; DIP — `ParkingLot` depends on the strategy *interfaces* (constructor-injected, same pattern as Spring DI); SRP — pricing isn't in `Ticket`, spot-finding isn't in `ParkingLot`; LSP — any `Vehicle` subtype is substitutable; ISP — separate small `ParkingStrategy`/`PricingStrategy` interfaces, not one fat manager.
- **Curveballs to rehearse**: add EV charging (new `SpotType` + `canFit` rule); weekend pricing (new `PricingStrategy`); concurrent gates (per-`Level` lock or `ConcurrentHashMap` of availability — ties to the concurrency block, NOT one lot-wide `synchronized`).
- **Other LLD sketches (name entities + the key strategy interface)**: Elevator (`SchedulingStrategy`), Vending machine (State pattern), Library (`PricingStrategy` for fines), Rate limiter (`TokenBucket`/`SlidingWindow` — the OO form of your Smart360 gateway Redis logic), Splitwise (`Split` hierarchy: equal/percent/exact).

**Block C (~1.5 hr) — Consolidation:**

- Write, without notes, the 8-step bean lifecycle and the three init hooks in order; check against your notes.
- Connect each concept to one line of code in Smart360/WebX/API Gateway:
  - **Smart360**: `@Transactional(propagation = REQUIRES_NEW)` for audit-log writes; custom `ThreadPoolExecutor` for parallel S3 URL caching; N+1 fix via `@EntityGraph` inside a `REQUIRED` tx; `@Transactional(readOnly = true)` to skip dirty checking.
  - **WebX**: `@Async` with a bounded `ThreadPoolTaskExecutor`; `CompletableFuture` multi-provider fan-out; `volatile boolean` shutdown flag; `ThreadLocal` tenant ID cleared after each job.
  - **API Gateway**: `@ConditionalOnProperty` to toggle the circuit breaker per environment; CGLIB proxy on filter beans.
- Open any Spring Boot app, add `debug=true`, run it, read the CONDITIONS EVALUATION REPORT; note 3 auto-configs that matched and 3 that didn't, and why.

**Self-check:**
1. Tomcat gets 1000 concurrent requests — describe exactly what happens at the thread-pool level. (Default `server.tomcat.threads.max=200`, `min-spare=10`; overflow queues in `accept-count` (default 100); beyond that, connections refused.)
2. Difference between `future.get()` and `future.join()`? (`get()` throws checked `ExecutionException`/`InterruptedException`; `join()` throws unchecked `CompletionException`.)

---

### Sunday Jul 12 — Rest

No study blocks. Let the week's material consolidate. Optionally skim the self-check answers you got wrong — but do not start new material.

---

## 🧠 Concepts to Master This Week (depth, gotchas, follow-ups)

### Binary Search
- **Invariant choice**: `left <= right` for exact search (use `right = mid - 1` — `mid` is eliminated when `nums[mid] != target`); `left < right` for boundary-finding (use `right = mid` — `mid` is still a candidate). Mixing them → off-by-one or infinite loop (TLE).
- **Rotated array**: compare `nums[mid]` to `nums[left]`/`nums[right]` to find the sorted half; the unsorted half holds the inflection point. Duplicates (LC 81): when `nums[left]==nums[mid]==nums[right]` you can't decide — `left++; right--`, degrading to O(n).
- **Binary-search-on-answer**: any monotonic answer with an O(n) feasibility test — Koko (875), Ship Packages (1011), Split Array Largest Sum (410).

### Backtracking
- **Contract**: choose → recurse → un-choose; the un-choose is what makes it backtracking.
- **Complexity**: Subsets O(2ⁿ × n); Permutations O(n! × n); Combination Sum O(2^(target/min) × …). State it in interviews.
- **Pruning**: sort, then break the inner loop when `candidates[i] > target`.
- **Duplicates**: Subsets II (90), Combination Sum II (40), Permutations II (47) all use `if (i > start && a[i]==a[i-1]) continue;`.

### Data Structures
- **Stack**: monotonic stack for next-greater/span/histogram; back with `ArrayDeque`, never the legacy `Stack`.
- **Queue/Deque**: monotonic deque → sliding-window max; `ArrayDeque` faster than `LinkedList`.
- **Linked List**: dummy head removes special cases; slow/fast for cycle and midpoint; iterative + recursive reversal; DLL + HashMap = LRU.

### Spring IoC/DI & Lifecycle
- ApplicationContext is a superset of BeanFactory (eager singletons, events, i18n, AOP). Production always uses ApplicationContext.
- AOP proxy created at step 6 of the lifecycle; the injected bean IS the proxy.
- `@Autowired` (Spring) vs `@Inject` (JSR-330) vs `@Resource` (JSR-250, name-match).

### @Transactional Internals
- Proxy created at lifecycle step 6, not at call time — `getClass()` on the injected service returns the CGLIB subclass.
- `TransactionSynchronizationManager` stores the connection in a `ThreadLocal` → tx is per-thread; cannot cross HTTP ↔ `@Async` threads.
- `@Transactional(readOnly = true)` disables Hibernate dirty checking, hints `Connection.setReadOnly(true)` (HikariCP can route to a replica) — a real Smart360 latency win.

### Auto-Configuration
- `debug=true` → CONDITIONS EVALUATION REPORT (the interviewer-favorite debug technique).
- Custom starter: `@AutoConfiguration` + `@ConditionalOn*` + an entry in `AutoConfiguration.imports`.

### REST / JWT / OAuth2
- Idempotency per method; idempotency keys make POST safe.
- JWT structure `header.payload.signature` — only the signature is secure; the payload is readable. `RS256` (private sign / public verify) for multi-service.
- Spring Security `SecurityContextHolder` is ThreadLocal-based — breaks in WebFlux; use `ReactiveSecurityContextHolder` (a Reactor `Context` on the `Mono`/`Flux` chain).

### Concurrency
- `volatile` (visibility+ordering, no atomicity, zero contention) vs `synchronized`/`ReentrantLock` (mutual exclusion, blocking) vs `Atomic*` (CAS, lock-free) vs `ConcurrentHashMap` (lock-free reads, CAS/bucket-lock writes — never `Collections.synchronizedMap`).
- `ForkJoinPool` (work-stealing, good for divide-and-conquer; default for `CompletableFuture`) vs a dedicated `ThreadPoolExecutor` for blocking I/O (LLM calls).

---

## 🎤 Sample Interview Questions (incl. curveballs)

1. **Implement binary search; now for a rotated array; now with duplicates.** → LC 704 → 33 → 81. With duplicates, `nums[left]==nums[mid]==nums[right]` forces `left++; right--`, worst case O(n) — state the trade-off.
2. **What is binary-search-on-answer? A real example from your work?** → Monotonic feasibility on the answer space. WebX: "minimum thread count such that p99 < 2s" — load-test feasibility, answer space `[1, maxThreads]`. Textbook form: Koko.
3. **Spring bean lifecycle — where are AOP proxies created?** → Step 6 by `AnnotationAwareAspectJAutoProxyCreator`; after it, the context holds the proxy.
4. **Your `@Transactional` method isn't rolling back — debug it.** → Checked vs runtime exception (`rollbackFor`); self-invocation via `this`; private method; try-catch swallowing the exception; wrong `PlatformTransactionManager`. Tool: `logging.level.org.springframework.transaction=TRACE`.
5. **Why can't you start a new `@Transactional` op inside an `@Async` method when the caller had a tx?** → `@Async` runs on another thread; the ThreadLocal-based `TransactionSynchronizationManager` has no value there. WebX commits the `jobId` before dispatch; the worker fetches by ID in a fresh tx.
6. **(Curveball) "Just put `@Transactional` on everything to be safe."** → Push back: long txs starve the pool (worse with HTTP/LLM calls mid-tx), unintended REQUIRED joins, wasted locks on reads. Annotate at the service boundary; `readOnly = true` for queries.
7. **`volatile` vs `synchronized` — when is `volatile` enough?** → One writer + no compound op (e.g., a shutdown flag). Not for counters / check-then-act.
8. **Implement Min Stack with O(1) getMin.** → Auxiliary min-stack pushed alongside; pop both together.
9. **(Curveball) Implement LRU without `LinkedHashMap`.** → Node(key,val,prev,next) + sentinels + `moveToHead`/`removeTail` + HashMap.
10. **DELETE /orders/123 called twice — 404 or 204?** → Both defensible (idempotency = same server state); return 204 for a cleaner client experience.
11. **Make POST /transfers safe to retry.** → `Idempotency-Key` header; store key→result in Redis with TTL; replay returns the stored result without re-charging.
12. **Logout invalidating a stateless JWT?** → Add `jti` to a Redis blacklist with TTL = remaining lifetime; delete the refresh token; check the blacklist (O(1)) per request.
13. **(Curveball) ReentrantLock vs synchronized — when did you use ReentrantLock?** → `tryLock(5, SECONDS)` on WebX LLM provider slots to return 503 instead of blocking forever.
14. **Tomcat handling 1000 concurrent requests — what when all threads are busy?** → 200 worker threads (default), then `accept-count` queue (100), then connection refused; scale replicas horizontally before this.
15. **(Curveball) RBAC at API/service/DB levels — what breaks with only one?** → API-level alone is bypassed by a bug; service-level adds defense-in-depth; DB RLS protects even a compromised service from cross-tenant leaks.

---

## 🌟 Extraordinary-Candidate Edge

1. **Name the proxy type unprompted.** "Spring Boot 2.x+ defaults to CGLIB even for interface-based beans — so a `@Transactional` class can't be `final` and transactional methods can't be `private`." Most candidates don't know this changed.
2. **Quote the CONDITIONS EVALUATION REPORT.** "I set `debug=true` to read which auto-config matched and the exact gating condition — it's how I found a custom `DataSource` wasn't picked up in one environment."
3. **Connect `ThreadLocal` to security.** `SecurityContextHolder` is ThreadLocal-backed; we used the same for `TenantContext` but always `remove()`d in a `finally` in our filter — missing it is a cross-tenant data leak (it bit us once when a super-admin auth bled into the next request on the same pooled thread).
4. **Size thread pools with math (Little's Law).** LLM jobs at ~5 req/min × 8-min latency → 40 concurrent; `corePoolSize=40, maxPoolSize=60`, bounded queue 100, plus an `executor.active` metric to alert on saturation.
5. **LRU beyond the trick.** Implement the DLL version, then: "In production I'd use Redis `EXPIRE` + `LFU` eviction — understanding the structure helps me reason about Redis's own eviction."

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5. Anything below 3 is a mandatory revisit before Week 3.

| Skill | Target | Your Score | Notes |
|---|---|---|---|
| Binary search — standard, from scratch | 5 | | |
| Search in Rotated Sorted Array (LC 33), first attempt | 4 | | |
| Koko / binary-search-on-answer — identify independently | 4 | | |
| Subsets / Permutations / Combination Sum — state-restore + dedup | 4 | | |
| Valid Parentheses / Min Stack | 4 | | |
| Daily Temperatures / Next Greater (monotonic stack) | 4 | | |
| Largest Rectangle in Histogram | 4 | | |
| Reverse / Merge / Cycle (Floyd's) linked lists | 4 | | |
| Reorder List / Remove Nth from End | 4 | | |
| LRU Cache (full HashMap + DLL) | 4 | | |
| Spring bean lifecycle — all 8 steps without notes | 5 | | |
| @Transactional: proxy type, self-invocation, propagations | 5 | | |
| ApplicationContext vs BeanFactory — 3+ real differences | 4 | | |
| Auto-configuration: conditionals + how to debug | 4 | | |
| Full request lifecycle whiteboard (DispatcherServlet → repo) | 4 | | |
| REST versioning + pagination + idempotency keys | 4 | | |
| JWT issuance → validation → refresh → RBAC | 4 | | |
| OAuth2 grant types (when to use which) | 4 | | |
| @ControllerAdvice + Bean Validation from memory | 4 | | |
| synchronized vs ReentrantLock — when to use each | 4 | | |
| volatile — guarantees, limits, correct use | 4 | | |
| CompletableFuture — chaining, executor choice, WebX link | 4 | | |
| ThreadLocal — Tomcat pool-reuse gotcha, Security usage | 4 | | |
| Deadlock — scenario, 4 Coffman conditions, prevention | 4 | | |
| LLD: Parking Lot classes + a SOLID letter per decision | 4 | | |

**Score interpretation:**
- 4–5 on all: ready for any Spring/concurrency/foundation-DSA question.
- 3 on 1–2 items: revisit in Week 3's daily review slot.
- Below 3 on any item: schedule a focused 45-min re-drill before the topic compounds.

**Bonus check:** Can you connect every concept above to at least one specific line of code or decision in Smart360, WebX, or your API Gateway? 80%+ → you are an extraordinary candidate, not just a prepared one.

---

*Week 2 of 12 — next: Week 3 (Mon Jul 13 – Sat Jul 18).*
