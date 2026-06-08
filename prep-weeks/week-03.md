# Week 3 — Foundations (Jun 22–28, 2026)

> Master the bedrock that every Java interview tests first: recursion, binary search, Spring internals, and Java concurrency — then connect every concept back to your resume.

---

## 🎯 Week Goal

Walk into any interview and answer cold questions on Spring IoC/DI, `@Transactional`, bean lifecycle, binary search variants, and Java concurrency without hesitation. Implement binary-search-on-answer and recursive backtracking under time pressure. Articulate concurrency decisions you made in WebX (async LLM jobs) and Smart360 (parallel queries) at senior-engineer depth.

---

## ✅ By Sunday you can...

- [ ] Implement iterative and recursive binary search from scratch, and explain the off-by-one logic
- [ ] Solve Search in Rotated Sorted Array, Koko Eating Bananas, and Search a 2D Matrix correctly on first attempt
- [ ] Explain binary-search-on-answer pattern and identify when a problem needs it
- [ ] Generate all Subsets and Permutations using recursive backtracking with state-restore
- [ ] Whiteboard the full Spring request lifecycle: Servlet container → DispatcherServlet → HandlerMapping → Interceptors → Controller → Service (proxy) → Repository → back
- [ ] Explain `@Transactional` proxy mechanics AND the self-invocation pitfall with a fix
- [ ] Distinguish ApplicationContext vs BeanFactory and all 8 steps of bean lifecycle (incl. where AOP proxies appear)
- [ ] Explain auto-configuration: `AutoConfiguration.imports`, `@ConditionalOnClass`, `@ConditionalOnMissingBean`, and how to debug what fired
- [ ] Compare `synchronized` vs `ReentrantLock`, explain `volatile`, write a deadlock scenario and prevent it
- [ ] Explain `CompletableFuture` chaining and connect it to WebX's async LLM job design
- [ ] Answer "ThreadLocal in Tomcat" without notes

---

## 📅 Daily Checklist

---

### Monday Jun 22 — Binary Search Fundamentals

**DSA Block (45 min)**

- [ ] Implement binary search iteratively. Write it from scratch — no looking up. Test with even/odd length arrays and target not found.
  - Invariant to memorise: `while (left <= right)`, `mid = left + (right - left) / 2`, shrink with `left = mid + 1` / `right = mid - 1`.
  - Ask yourself: why `left + (right - left) / 2` instead of `(left + right) / 2`? (Integer overflow when left+right > Integer.MAX_VALUE — mention this in interviews.)
- [ ] LC 704 — Binary Search (Easy). Target: <5 min.
- [ ] LC 33 — Search in Rotated Sorted Array (Medium). Work through both cases: pivot left of mid, pivot right of mid. The key insight: one of the two halves is always sorted — use that to decide which half contains the target.
- [ ] LC 153 — Find Minimum in Rotated Sorted Array (Medium). No target, just find the inflection point. Invariant: `nums[mid] > nums[right]` means pivot is in the right half.

**Self-check:** Can you identify which half is sorted in a rotated array in O(1) decision? If not, re-read and re-code before Tuesday.

---

### Tuesday Jun 23 — Binary Search on Answer + 2D Matrix

**DSA Block (45 min)**

- [ ] Binary-search-on-answer pattern: when the answer space is a monotonic range (integers or real numbers) and you can write a feasibility function `canDo(mid)`, binary-search the answer itself, not an array index. Template:
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
- [ ] LC 875 — Koko Eating Bananas (Medium). `canEat(speed)` = can Koko finish all piles in `h` hours at this speed? Answer space: [1, max(piles)]. Target: < 15 min.
- [ ] LC 74 — Search a 2D Matrix (Medium). The matrix is row-sorted and each row starts > last row's end → treat it as a flat sorted array. `mid` maps to `(mid/cols, mid%cols)`. Target: < 10 min.

**Core topic (45 min): ApplicationContext vs BeanFactory + IoC/DI deep-dive**

- [ ] Read your 01-java-language-basics.md + interview-qa.md Spring sections.
- [ ] Know cold:
  - `BeanFactory` = lazy initialisation by default, minimal; `ApplicationContext` extends it and adds eager singleton init, event propagation (`ApplicationEventPublisher`), i18n (`MessageSource`), AOP proxy post-processing, `@Async`/`@Scheduled` support. Always use `ApplicationContext` in practice.
  - DI modes: constructor (recommended — immutable, easy to test), setter (optional deps), field (`@Autowired` on field — avoid: hides dependencies, breaks in non-Spring tests).
  - `@Primary` vs `@Qualifier`: `@Primary` sets a default bean for autowiring ambiguity; `@Qualifier("beanName")` is explicit. Use `@Qualifier` when you intentionally have multiple implementations (e.g., two `DataSource` beans for primary/replica routing in Smart360).
- [ ] Write: under what circumstances would you inject `ApplicationContext` directly? (Answer: to call `getBean()` for prototype beans, to publish events, to programmatically check bean definitions. Rarely needed — flag it as a design smell if overused.)

**Self-check:** Explain the difference to a junior dev in 90 seconds.

---

### Wednesday Jun 24 — Spring Bean Lifecycle + @Transactional Internals

**DSA Block (45 min)**

- [ ] LC 78 — Subsets (Medium). Two approaches:
  - Cascading: start with `[[]]`, iterate array, for each existing subset add a new subset with current element appended.
  - Backtracking: at each index choose include/exclude, recurse, pop. Prefer backtracking — interviewers will ask you to extend it to Subsets II (with duplicates) immediately.
- [ ] Trace the backtracking tree for input `[1,2,3]` on paper. Count leaves = 8 = 2^3.

**Core topic (45 min): Bean Lifecycle + @Transactional**

- [ ] Whiteboard all 8 lifecycle phases from your interview-qa.md, then extend:
  - After step 6 (BeanPostProcessor.postProcessAfterInitialization), ask: what specifically happens here for `@Transactional`? The `AnnotationAwareAspectJAutoProxyCreator` (itself a `BeanPostProcessor`) checks if the bean has any advice (transactional, AOP, `@Async`, `@Cacheable`). If yes, it returns a CGLIB subclass proxy instead of the original bean — the original bean is wrapped, not replaced.
  - Why CGLIB for classes vs JDK proxy for interfaces? CGLIB subclasses the target class at byte-code level (requires class to be non-final, method to be non-private). JDK dynamic proxy only works on interface types. Spring Boot 2.x+ defaults to CGLIB even for interfaces.
- [ ] `@Transactional` propagation — know these cold:
  - `REQUIRED` (default): join existing tx; if none, start new. 95% of use cases.
  - `REQUIRES_NEW`: suspend any existing tx, start fresh, commit independently. Critical for: audit log that must persist even if outer tx rolls back, idempotency tables, outbox pattern in your WebX.
  - `NESTED`: savepoint inside the existing tx — rollback to savepoint on failure, outer tx continues. Rarely used but interviewers love asking.
  - `NOT_SUPPORTED`: suspend existing tx, run without one. For read-only ops where you explicitly don't want tx overhead.
  - `NEVER`: throw if a tx exists. Useful to assert no-tx context.
- [ ] Self-invocation pitfall — must be able to demo:
  ```java
  @Service
  public class OrderService {
      // THIS CALL BYPASSES THE PROXY — no transaction!
      public void placeOrder() {
          this.processPayment();  // 'this' is the raw bean, not the proxy
      }
  
      @Transactional
      public void processPayment() { ... }
  }
  ```
  Fix 1: inject self — `@Autowired private OrderService self; self.processPayment();`
  Fix 2: fetch from context — `applicationContext.getBean(OrderService.class).processPayment();`
  Fix 3: restructure — move `processPayment` to a separate `@Service` class (cleanest).
- [ ] Three other `@Transactional` pitfalls to mention unprompted:
  - Private methods: proxy cannot override them → annotation silently ignored.
  - Default `rollbackFor` = `RuntimeException` only — checked exceptions commit. Use `@Transactional(rollbackFor = Exception.class)` for services that throw checked exceptions.
  - Long transactions holding DB connections: avoid calling external HTTP services or LLM APIs inside a `@Transactional` method. You held a Postgres connection for the duration of a 20-second LLM call — this starves the connection pool. Fix: do the LLM call outside the transaction boundary (you did this in WebX — cite it).

**Self-check:** Draw the proxy call stack on paper: caller → CGLIB proxy → `TransactionInterceptor` → actual bean method.

---

### Thursday Jun 25 — Spring Boot Auto-Configuration + Full Request Lifecycle

**DSA Block (45 min)**

- [ ] LC 46 — Permutations (Medium). Backtracking with a `used[]` boolean array or by swapping. Practice both.
  - State restore pattern (critical for all backtracking problems):
    ```
    backtrack(path, remaining):
        if remaining is empty: add path to results; return
        for each choice in remaining:
            path.add(choice)        // choose
            remaining.remove(choice)
            backtrack(path, remaining)
            remaining.add(choice)   // RESTORE (un-choose)
            path.remove(last)
    ```
  - Time complexity: O(n! × n) — explain it: n! permutations, each of length n.

**Core topic (45 min): Auto-configuration mechanics + DispatcherServlet request lifecycle**

- [ ] Auto-configuration deep-dive (go beyond interview-qa.md):
  - Boot 3.x: `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` replaces `spring.factories`. Each line is a class name. Spring loads them all but `@Conditional*` annotations gate each one.
  - Key conditionals: `@ConditionalOnClass(DataSource.class)` — only if the class is on the classpath. `@ConditionalOnMissingBean(DataSource.class)` — only if you haven't defined your own. `@ConditionalOnProperty("spring.datasource.url")` — only if the property is set.
  - How to debug: set `logging.level.org.springframework.boot.autoconfigure=DEBUG` in `application.properties`. You'll see a "CONDITIONS EVALUATION REPORT" — which auto-configs matched and which were excluded and why. Interviewers love hearing this.
  - `spring-boot-autoconfigure-processor`: generates `META-INF/spring-autoconfigure-metadata.properties` at compile time to speed up conditional evaluation — conditions can be pre-filtered before classes are loaded.
  - How to disable one: `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)` — you do this in test slices and WebX unit tests to avoid needing a real DB.
- [ ] Full request lifecycle — whiteboard without notes:
  ```
  Browser/Client
      ↓ HTTP Request
  Servlet Container (Tomcat) — assigns thread from ThreadPool
      ↓
  DispatcherServlet.doDispatch()
      ↓
  HandlerMapping — matches URL to @RequestMapping method, returns HandlerExecutionChain
      ↓
  HandlerInterceptor.preHandle() — your JWT validation filter runs here (or in SecurityFilterChain before this)
      ↓
  HandlerAdapter — resolves @RequestBody (via HttpMessageConverter e.g. Jackson), @PathVariable, @RequestParam
      ↓
  @Controller / @RestController method
      ↓
  @Service — CGLIB proxy checks @Transactional, starts tx if needed
      ↓
  @Repository / Spring Data JPA — Hibernate Session, SQL execution
      ↓
  Response builds back up through same chain
      ↓
  HandlerInterceptor.postHandle()
      ↓
  HttpMessageConverter serialises return value to JSON
      ↓
  HandlerInterceptor.afterCompletion()
      ↓
  HTTP Response to client
  ```
  Key interviewer follow-up: "Where does Spring Security fit?" → `SecurityFilterChain` is a servlet `Filter` — it runs BEFORE DispatcherServlet. `@PreAuthorize` is AOP on the service method — inside the service call.

**Self-check:** Where exactly does `@ControllerAdvice` / `@ExceptionHandler` intercept? (Answer: `DispatcherServlet` catches exceptions from the handler and delegates to `HandlerExceptionResolver` → `ExceptionHandlerExceptionResolver` → `@ExceptionHandler`. NOT a filter, not a servlet. It runs after the controller throws.)

---

### Friday Jun 26 — Combination Sum + Week Review Sprint

**DSA Block (45 min)**

- [ ] LC 39 — Combination Sum (Medium). Backtracking where the same element can be reused. Key difference from Permutations: pass `start` index to avoid revisiting earlier elements. This prevents duplicates in the result.
  ```
  backtrack(candidates, target, start, path, result):
      if target == 0: result.add(copy of path); return
      if target < 0: return
      for i from start to end:
          path.add(candidates[i])
          backtrack(candidates, target - candidates[i], i, path, result)  // i not i+1 (reuse allowed)
          path.remove(last)
  ```
- [ ] Quick re-attempt LC 33 and LC 153 from Monday without notes. Target: < 10 min each. If you're slower, note why and fix the gap before the weekend.

**Core topic (30 min): Spring internals consolidation**

- [ ] Write, without notes, the 8-step bean lifecycle. Check against interview-qa.md.
- [ ] Write the three ways to hook into bean initialization and their order: `@PostConstruct` vs `InitializingBean.afterPropertiesSet()` vs `init-method`. Execution order: `@PostConstruct` → `afterPropertiesSet()` → `init-method`. The `BeanPostProcessor` wraps around all of them.
- [ ] Explain `@Lazy`: delays bean instantiation until first use. Useful for circular dependency resolution (last resort — better to restructure) and expensive beans not always needed. Trade-off: first-call latency; NPEs if lazy bean fails on init (you won't see it at startup).

**Self-check (15 min):** Answer these aloud, timer running, no notes:
- What fires `@PostConstruct`?
- What happens if your `@Transactional` service calls another method on `this`?
- How does `@ConditionalOnMissingBean` prevent your DataSource from being overridden by auto-config?

---

### Saturday Jun 27 — Concurrency Deep Dive (Weekend Session, ~4 hr)

**DSA Block (60 min)**

- [ ] Re-solve LC 875 (Koko), LC 74 (2D Matrix), LC 78 (Subsets) back-to-back. Goal: clean code, no bugs, first try. Time yourself.
- [ ] LC 54 — Spiral Matrix (Medium) — bonus if time allows; useful for Amazon/Google rounds.

**Concurrency Block (~3 hr)**

Work through your 06-concurrency-and-collections.md, but push deeper on each topic:

**`synchronized` vs `ReentrantLock` — go deeper than your notes:**
- [ ] `synchronized` is a JVM keyword — lock acquire/release is automatic (even on exception). `ReentrantLock` is explicit — you MUST call `unlock()` in a `finally` block or you leak the lock forever.
- [ ] `ReentrantLock` advantages: `tryLock(timeout)` to avoid blocking forever (crucial for deadlock avoidance), `lockInterruptibly()` so a blocked thread can be interrupted, fairness policy (`new ReentrantLock(true)` — FIFO queue, prevents starvation but lower throughput), multiple condition variables (`lock.newCondition()`).
- [ ] When to use `ReentrantLock` in practice: timeout-based lock acquisition in your LLM job queue (you don't want a thread waiting forever for a slot), any scenario where you have two+ condition variables (producer-consumer with full/empty conditions).
- [ ] `ReadWriteLock`: `ReentrantReadWriteLock` — multiple concurrent readers OR one exclusive writer. Perfect for: in-memory config map read by all request threads, rarely written by refresh thread. In WebX, the LLM provider routing table could use this.

**`volatile` — be precise:**
- [ ] `volatile` guarantees: visibility (write is immediately visible to all threads, no CPU cache inconsistency) and ordering (prevents instruction reordering around the volatile access). It does NOT guarantee atomicity.
- [ ] Classic valid use: a `volatile boolean running` flag read by a worker thread, written by the shutdown thread. Because it's a single write/read with no compound operation, volatile is sufficient.
- [ ] Classic invalid use: `volatile int counter; counter++` — this is still a read-modify-write race. Use `AtomicInteger` instead.
- [ ] Java Memory Model (JMM) connection: `volatile` establishes a happens-before relationship — all writes before a volatile write are visible to all reads after the volatile read of the same variable.

**`CompletableFuture` — connect to WebX:**
- [ ] Chain methods:
  - `thenApply(fn)` — transform result, same thread
  - `thenCompose(fn)` — flat-map, fn returns another `CompletableFuture` (avoid nested futures)
  - `thenCombine(other, fn)` — combine two independent futures
  - `allOf(cf1, cf2, ...)` — wait for all, no return value
  - `anyOf(cf1, cf2, ...)` — first completed wins
  - `exceptionally(fn)` — handle exception, provide fallback
  - `handle(fn)` — handles both success and exception in one callback
- [ ] Thread pool: by default `supplyAsync` uses `ForkJoinPool.commonPool()`. In production, always pass an explicit `Executor` — `CompletableFuture.supplyAsync(() -> ..., customExecutor)`. The common pool is shared by the whole JVM; a slow LLM call can starve other tasks.
- [ ] WebX connection: the async LLM job architecture from interview-qa.md used `202 Accepted` + polling. Internally, you can represent each job as a `CompletableFuture` stored in a `ConcurrentHashMap<jobId, CompletableFuture<LLMResult>>`. When a client polls, check `future.isDone()`. When LLM responds, `future.complete(result)`. This is cleaner than a separate status table for in-memory tracking.

**ThreadLocal — the Tomcat/Spring gotcha:**
- [ ] `ThreadLocal<T>` gives each thread its own isolated copy of a value. Tomcat assigns one thread per HTTP request → Spring Security's `SecurityContextHolder` stores the `Authentication` in a `ThreadLocal` — that's why it's available anywhere in the call stack without passing it around.
- [ ] CRITICAL pitfall with thread pools: Tomcat reuses threads. If you store something in a `ThreadLocal` during request A and forget to call `remove()`, the NEXT request on the same thread inherits the stale value. This is a security vulnerability (leaking user context between tenants) and a memory leak. Spring Security calls `SecurityContextHolder.clearContext()` in its filter chain after the request — always do the same with your custom `ThreadLocal`s.
- [ ] In WebX, if you propagate `ThreadLocal` values across `@Async` boundaries (e.g., passing tenant ID to an async LLM worker), `ThreadLocal` does NOT propagate — the new thread starts fresh. Fix: use `InheritableThreadLocal` (copies at thread creation — still has pool reuse problems) or explicitly pass the value as a method argument or task parameter (cleanest).

**Deadlock — live example:**
- [ ] Write this scenario from memory and explain it:
  ```java
  // Thread 1                    // Thread 2
  synchronized(lockA) {         synchronized(lockB) {
      synchronized(lockB) {         synchronized(lockA) {
          // work                    // work
      }                         }
  }                             }
  // Deadlock: T1 holds A, waits B; T2 holds B, waits A
  ```
- [ ] Four conditions (Coffman): mutual exclusion, hold-and-wait, no preemption, circular wait. Eliminating any one breaks deadlock. Most practical: eliminate circular wait (always acquire locks in the same global order). In code: sort resources by ID before locking.

**Self-check (30 min):** Answer these without notes:
- Tomcat gets 1000 concurrent requests. Describe exactly what happens at the thread pool level.
- In WebX, why did you use `@Async` with a custom `ThreadPoolTaskExecutor` instead of letting each request spawn its own thread?
- What is the difference between `future.get()` blocking and `future.join()`? (`get()` throws checked `ExecutionException`/`InterruptedException`; `join()` throws unchecked `CompletionException` — prefer `join()` in lambda chains.)

---

### Sunday Jun 28 — End-of-Week Integration + Mock Interview

**DSA Block (60 min)**

- [ ] Full timed mock: pick 2 problems you haven't re-attempted this week. Timer: 25 min each. No hints.
- [ ] If stuck at 15 min, write down exactly where you got stuck — this becomes a specific study item for Week 4.

**Integration Session (~2 hr): Connect everything to resume**

- [ ] For each project, prepare a 90-second technical story that weaves this week's topics:
  - **Smart360**: `@Transactional(propagation = REQUIRES_NEW)` for audit log writes. Custom `ThreadPoolExecutor` for parallel S3 URL caching. N+1 fix via `@EntityGraph` inside a `REQUIRED` transaction.
  - **WebX**: `@Async` with bounded `ThreadPoolTaskExecutor` for LLM jobs. `CompletableFuture` for multi-provider fan-out. `volatile boolean` shutdown flag. `ThreadLocal` for tenant ID propagation — and why you cleared it after each job.
  - **API Gateway**: `@ConditionalOnProperty` to toggle circuit breaker in non-prod environments. CGLIB proxy on gateway filter beans.
- [ ] Auto-configuration mini-project: open any Spring Boot app, add `debug=true` to `application.properties`, run it, read the CONDITIONS EVALUATION REPORT. Note 3 auto-configs that matched and 3 that didn't — be able to explain why for an interview.
- [ ] Read the Spring `TransactionSynchronizationManager` source (just the Javadoc) — it stores the current transaction in a `ThreadLocal`. This is the implementation detail that explains why `@Transactional` works per-thread in Tomcat.

**End-of-week mock Q&A (45 min):** Have a friend or record yourself answering 5 random questions from the Sample Interview Questions below. No notes. Time the answers.

---

## 🧠 Concepts to Master This Week (Depth, Gotchas, Follow-ups)

### Binary Search

**Core invariant:** `left <= right` for standard search; `left < right` for finding a boundary. Mixing them causes off-by-one bugs or infinite loops. Know which to use when.

**Rotated array insight:** You can always determine which half is sorted by comparing `nums[mid]` to `nums[left]` (or `nums[right]`). The unsorted half contains the inflection point. Once you know which half is sorted, you know where the target can and cannot be.

**Binary-search-on-answer:** Any problem whose answer is a monotonic integer or real and where you can test feasibility in O(n) is a candidate. Typical phrasing: "minimum maximum", "maximum minimum", "largest X such that Y", "fewest/most X to achieve Y". Koko Eating Bananas, Ship Packages Within D Days (LC 1011), Capacity To Ship (same pattern), Split Array Largest Sum (LC 410) — all follow this template.

**Off-by-one gotcha with `right = mid` vs `right = mid - 1`:** When `left < right` loop (boundary finding), use `right = mid` (because `mid` is still a candidate for the answer). When `left <= right` loop (exact search), use `right = mid - 1` (because if `nums[mid] != target`, `mid` is eliminated). Getting this wrong causes TLE (infinite loop) or missed answers.

### Backtracking

**Template contract:** Choose → recurse → un-choose. The un-choose step (state restore) is what distinguishes backtracking from naive recursion. If you forget it, you corrupt the path for sibling branches.

**Complexity:** Subsets = O(2^n × n). Permutations = O(n! × n). Combination Sum = O(2^(target/min_candidate) × target/min_candidate). Always state this in an interview — it shows you understand why these don't scale.

**Pruning:** In Combination Sum, sort the array first. If `candidates[i] > target`, break the inner loop — no point trying larger values. This reduces constant factor significantly on test cases with large candidates.

**Duplicates:** Subsets II (LC 90), Combination Sum II (LC 40), Permutations II (LC 47) all add a `if (i > start && candidates[i] == candidates[i-1]) continue;` deduplication guard. This is a common extension question.

### Spring IoC/DI

**ApplicationContext is a superset of BeanFactory.** Production code always uses `ApplicationContext`. `BeanFactory` lazy-initializes; `ApplicationContext` eager-initializes all singletons at startup (fail fast on misconfiguration). The full class hierarchy: `BeanFactory` → `ListableBeanFactory` → `ApplicationContext` → `ConfigurableApplicationContext` → `AnnotationConfigApplicationContext` / `GenericWebApplicationContext`.

**Circular dependency:** Constructor injection fails fast at startup. Field/setter injection resolves it lazily (Spring temporarily injects a partial bean). This is why constructor injection is actually safer — it surfaces cycles at boot time rather than at runtime. Best fix: redesign to remove the cycle (usually a missing abstraction or misplaced responsibility).

**`@Autowired` vs `@Inject` vs `@Resource`:** `@Autowired` = Spring; `@Inject` = JSR-330 (type-match only, no Spring-specific features); `@Resource` = JSR-250 (name-match by default). `@Autowired(required = false)` allows a null injection if no bean found — use cautiously.

### Spring Boot Auto-Configuration

**Debug technique every interviewer loves:** `--debug` flag or `debug=true` property → Conditions Evaluation Report in logs. Shows exactly which auto-configurations matched, which were skipped and why. Use this when debugging "why is my DataSource not being auto-configured?" or "why did my custom bean get ignored?".

**Custom auto-configuration:** You can write your own. Create a class annotated `@AutoConfiguration`, add `@ConditionalOnClass`, add it to `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`. This is how every Spring Boot starter works (e.g., `spring-boot-starter-data-redis` ships an auto-config that creates a `RedisConnectionFactory` if `Lettuce` is on the classpath and no bean is defined).

**`@SpringBootApplication` is a composed annotation:** `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`. Knowing this lets you explain why removing `@EnableAutoConfiguration` disables auto-config while keeping component scanning.

### @Transactional Internals

**Proxy creation timing:** The proxy is created at step 6 of bean lifecycle (BeanPostProcessor.postProcessAfterInitialization), not at method call time. This means the proxy IS the bean — when Spring injects `OrderService` into your controller, it injects the CGLIB proxy, not the raw `OrderService` class. Calling `getClass()` on an injected `@Transactional` service returns the CGLIB subclass name.

**Transaction synchronization:** `TransactionSynchronizationManager` uses a `ThreadLocal<Map<Object, Object>>` to store the current connection per thread. This is why `@Transactional` is inherently thread-local — each HTTP request thread has its own transaction. It also explains why you cannot share a transaction between an HTTP thread and an `@Async` thread — the `@Async` method runs on a different thread with a different (or no) transaction.

**`@Transactional` on `@Repository` interfaces:** Spring Data JPA repositories have their own transaction management — each query method is wrapped in `REQUIRED` by default. When your service calls `repository.save()` inside a service-level `@Transactional`, the repository joins the service's transaction (REQUIRED propagation). No separate transaction is created.

### Concurrency

**`volatile` vs `synchronized` vs `Atomic`:**
- `volatile`: visibility + ordering, no atomicity. Zero contention overhead.
- `synchronized`/`ReentrantLock`: mutual exclusion + visibility + ordering. Thread blocking overhead.
- `AtomicInteger` etc: CAS (Compare-And-Swap) based — lock-free but not wait-free. Best for single-variable counters.
- `ConcurrentHashMap`: segment-level locking (Java 7) / CAS + synchronized on bucket head (Java 8+). Reads are lock-free. Use instead of `Collections.synchronizedMap(new HashMap<>())` (which locks the entire map).

**`ForkJoinPool` vs `ThreadPoolExecutor`:** `ForkJoinPool` is work-stealing — idle threads steal tasks from busy threads' queues. Good for recursive divide-and-conquer tasks (parallel sorting, tree traversal). `CompletableFuture.supplyAsync()` uses `ForkJoinPool.commonPool()` by default. Bad for blocking I/O tasks (LLM calls) — use a dedicated `ThreadPoolExecutor` with enough threads to cover blocking time.

---

## 🎤 Sample Interview Questions (incl. Curveballs) — 15 Qs

**Q1: Implement binary search. Now modify it for a rotated sorted array. What if there are duplicates?**

> Implement LC 704 (standard), then LC 33 (rotated). For duplicates (LC 81), the key problem: if `nums[left] == nums[mid] == nums[right]`, you cannot determine which half is sorted. Shrink both pointers: `left++; right--`. Worst case degrades to O(n). Mention this trade-off explicitly.

---

**Q2: What is binary-search-on-answer? Give me a real example from your experience.**

> When you can define a monotonic feasibility function on the answer space, you binary-search the answer itself rather than an index. In WebX, when configuring LLM job concurrency limits, you could model "what is the minimum thread count such that p99 latency < 2s?" — feasibility = run a load test at that concurrency, check latency. The answer space [1, maxThreads] is monotonic (more threads = lower latency up to a point). Cite Koko Eating Bananas as the textbook form.

---

**Q3: What is the Spring bean lifecycle? Where do AOP proxies get created?**

> Walk through all 8 steps from your interview-qa.md. Key precision: AOP proxies are created at step 6 by `AnnotationAwareAspectJAutoProxyCreator`, which is itself a `BeanPostProcessor`. After step 6, the bean in the context IS the proxy. The original object is accessible via `AopUtils.getTargetObject(proxy)` but never directly injected.

---

**Q4: Your @Transactional method is not rolling back. Walk me through your debugging process.**

> 1. Is the exception a `RuntimeException` or checked? Default rollback only for `RuntimeException`. If checked: add `rollbackFor = Exception.class`.
> 2. Is the method being called via `this`? Self-invocation bypasses the proxy — no transaction is active.
> 3. Is the method `private`? CGLIB cannot override private methods — annotation silently ignored.
> 4. Is there a `try-catch` swallowing the exception before it reaches the proxy's interceptor?
> 5. Is the correct `PlatformTransactionManager` configured (JPA vs JDBC)?
> Debugging tool: add `logging.level.org.springframework.transaction=TRACE` — logs every tx begin/commit/rollback.

---

**Q5: Why can't you start a new @Transactional operation from inside an @Async method when the caller had a transaction?**

> `@Async` runs on a different thread. `TransactionSynchronizationManager` uses `ThreadLocal` to track the current connection. The new thread has no `ThreadLocal` value → no active transaction. The `@Async` method starts its own new transaction (if annotated `@Transactional`) or runs without one. This is by design — distributed transactions across threads require explicit coordination (e.g., passing a transaction context manually, which Spring does not do automatically). In WebX, this is why LLM job submission saves the `jobId` to DB (commits immediately) before dispatching to the async executor — the async worker fetches by ID from a fresh transaction.

---

**Q6 (Curveball): A senior dev says "just use @Transactional on everything to be safe." How do you respond?**

> Respectfully push back. Overusing `@Transactional` causes: (1) long-running transactions holding DB connections, starving the pool — especially dangerous if the method does HTTP calls or LLM inference mid-transaction; (2) unintended transaction join via REQUIRED propagation — a bug in a helper method can roll back a parent transaction you didn't expect; (3) read-only queries wrapped in write transactions waste lock overhead. Best practice: annotate at service boundary, not repository, with the minimal propagation and isolation needed. Mark read-only queries `@Transactional(readOnly = true)` — Hibernate skips dirty checking, DB can route to replica.

---

**Q7: Explain the difference between volatile and synchronized. When is volatile enough?**

> `volatile` is enough when: (a) only one thread writes, others only read, AND (b) no compound read-modify-write operation is needed. Example: a `volatile boolean shutdownFlag`. NOT enough for: incrementing a counter, check-then-act patterns. Use `synchronized` or `Atomic*` for those. Key JMM point: `volatile` write happens-before all subsequent `volatile` reads of the same variable — this is the formal guarantee.

---

**Q8: You're seeing intermittent data corruption in a production service with multiple async workers. How do you investigate?**

> 1. Identify shared mutable state: look for instance variables in `@Service` beans accessed by multiple threads (Spring singletons are NOT thread-safe by default unless you make them so).
> 2. Check for missing synchronization on collections: `ArrayList`, `HashMap` are not thread-safe. Should be `CopyOnWriteArrayList` or `ConcurrentHashMap`.
> 3. Check `ThreadLocal` leaks: if Tomcat thread pool is reusing threads, stale `ThreadLocal` values from previous requests could corrupt data.
> 4. Enable thread-safety analysis: run with `-Djdk.virtualThreadScheduler.maxPoolSize=1` in tests to serialize execution and reproduce races deterministically.
> 5. Use `jstack` or async-profiler in production to capture thread states and detect contention.
> In Smart360: found an instance variable `private List<String> cachedUrls` on a singleton service, written and read by concurrent requests without synchronization. Fixed with `CopyOnWriteArrayList`.

---

**Q9: How does Tomcat handle concurrent HTTP requests at the thread level? What happens when all threads are busy?**

> Tomcat (default connector: NIO) maintains a thread pool via `ThreadPoolExecutor` — configured via `server.tomcat.threads.max` (default 200) and `server.tomcat.threads.min-spare` (default 10). Each accepted connection is handed to a worker thread. When all worker threads are busy, new connections queue in the acceptor queue (bounded by `server.tomcat.accept-count`, default 100). When the queue is also full, new connections are rejected with connection refused. For Spring Boot in Azure Container Apps, you scale horizontally before hitting this — set `server.tomcat.threads.max` slightly above your expected concurrency per pod, right-size the pod, then let ACA auto-scale replicas.

---

**Q10: What is the difference between ReentrantLock and synchronized? When have you used ReentrantLock specifically?**

> `synchronized` is simpler — JVM-managed, automatically released on exit or exception, no import needed. `ReentrantLock` is more flexible: `tryLock(timeout)` avoids indefinite blocking, `lockInterruptibly()` allows thread interruption while waiting, fairness option, multiple condition variables. In WebX, could have used `ReentrantLock.tryLock(5, SECONDS)` on the LLM provider slots to avoid a request thread blocking indefinitely if all 5 providers were busy — instead timeout and return a 503. `synchronized` can't do this.

---

**Q11 (Curveball): If ApplicationContext is so powerful, why does BeanFactory still exist?**

> Legacy and embedded use cases. `BeanFactory` is the minimal contract — useful in memory-constrained environments (IoT, Android historically) or framework internals where you need a container without the full event/i18n overhead. Also, Spring Cloud Function and some serverless contexts use lightweight factories. As a developer building services, you will never directly use `BeanFactory` — but knowing the hierarchy shows you understand the design. `ApplicationContext` IS a `BeanFactory` (via interface inheritance), so everywhere `BeanFactory` is accepted, `ApplicationContext` works.

---

**Q12: Walk me through what happens when Spring Boot cannot find a required bean.**

> At context refresh time (step 5 of startup), Spring resolves all `@Autowired` dependencies. If a required bean is missing: `NoSuchBeanDefinitionException` is thrown, context fails to start, and the application exits. You can make an injection optional with `@Autowired(required = false)` or `Optional<T>`. For auto-configured beans, `@ConditionalOnMissingBean` means Spring only creates the bean if YOU haven't defined one — so if you define your own `DataSource`, Boot's auto-configured one is skipped. If you think you defined a bean but it's not being found: check component scan range (is the class in a package under the `@SpringBootApplication` class?), check it's annotated with a stereotype, check for `@Profile` gating.

---

**Q13: How does CompletableFuture fit into WebX's async LLM job processing? What are the risks?**

> In WebX, when a user submits a long-running LLM analysis (2–20 min): the HTTP thread returns `202 Accepted` immediately. A `CompletableFuture` is created via `CompletableFuture.supplyAsync(() -> llmClient.call(request), llmExecutor)` and stored in `ConcurrentHashMap<jobId, CompletableFuture<Result>>`. The polling endpoint checks `future.isDone()`. On completion, `future.thenAccept(result -> persist(result))` writes to DB. Risks: (1) if the JVM restarts, all in-flight `CompletableFuture`s are lost — need a persistent job store (DB-backed queue) for production durability; (2) the custom `llmExecutor` must be sized correctly — too few threads and jobs queue up, too many and you hit provider rate limits; (3) `exceptionally()` must be handled or errors are silently swallowed.

---

**Q14 (Curveball): Two engineers disagree on whether to use synchronized or ConcurrentHashMap for a shared cache in a Spring singleton. How do you settle it?**

> Depends on the operations. If the cache is read-heavy and writes are rare: `ConcurrentHashMap` wins — reads are lock-free, writes use CAS/fine-grained locks, far better throughput than `synchronized` on the whole method. If you need atomic read-modify-write (e.g., "check if entry exists, if not compute and insert atomically"): use `ConcurrentHashMap.computeIfAbsent()` — it's atomic and lock-free for most cases. Use `synchronized` only if you need to atomically update multiple keys or entries as one unit. In practice: for a URL cache like in Smart360, `ConcurrentHashMap` with `computeIfAbsent(() -> generatePresignedUrl())` is the right answer — no contention on reads, atomic on writes.

---

**Q15 (Curveball): Explain why @Transactional(readOnly = true) can improve performance, and give a specific example from your work.**

> Three gains: (1) Hibernate disables dirty checking on the session — at flush time (end of transaction), Hibernate normally iterates all loaded entities and compares to snapshots to detect changes. `readOnly = true` tells Hibernate to skip this. For a query loading 500 entities for a report, this is significant CPU savings. (2) Spring's JDBC `Connection.setReadOnly(true)` hint — some JDBC drivers and connection pools (HikariCP) route read-only connections to a read replica. (3) The underlying DB (PostgreSQL) can skip overhead for read-only transactions. In Smart360, the dashboard query that was originally 60s fetched hundreds of joined entities — after annotating the service method `@Transactional(readOnly = true)`, Hibernate dirty checking overhead was eliminated, contributing to the latency reduction alongside the index and JOIN FETCH fixes.

---

## 🌟 Extraordinary-Candidate Edge

**Go beyond "I used it" to "I designed it this way because..."**

1. **Name the proxy type unprompted.** When discussing `@Transactional`, say: "Spring Boot 2.x+ defaults to CGLIB proxying even for interface-based beans — it was changed from JDK proxy default in Boot 2.0. This means your `@Transactional` service class cannot be `final` and transactional methods cannot be `private`." Most candidates don't know this changed.

2. **Quote the CONDITIONS EVALUATION REPORT.** When asked about auto-configuration, say: "I actually set `debug=true` on my projects to read the Conditions Evaluation Report at startup — it shows every auto-config class, whether it matched or not, and the exact condition that gated it. This is how I found that our custom `DataSource` bean wasn't being picked up in one environment."

3. **Connect `ThreadLocal` to security.** Say: "Spring Security's `SecurityContextHolder` is backed by a `ThreadLocal` — that's why `Authentication` is accessible anywhere in the call stack without passing it as a parameter. In our multi-tenant system, we used the same pattern for `TenantContext.ThreadLocal` but always called `remove()` in a `finally` block in our filter, because Tomcat reuses threads from its pool. Missing that `remove()` is a cross-tenant data leak — it manifested in one early bug where a super-admin request's authentication bled into the next non-admin request on the same thread."

4. **Size your thread pools with math.** When asked about `ThreadPoolTaskExecutor` for `@Async`: "We sized the pool using Little's Law: average concurrency = arrival rate × average service time. For our LLM jobs arriving at ~5 req/min with average 8-minute LLM latency: 5/60 req/s × 480s = 40 concurrent jobs. We set `corePoolSize=40, maxPoolSize=60` with a bounded queue of 100 — and added a `/actuator/metrics` endpoint exposing `executor.active` to alert when we approach saturation."

5. **Know one non-obvious `@Transactional` production story.** "We had a bug where an LLM enrichment step inside a `@Transactional` service method held a Postgres connection open for the full 8-minute LLM call. With a 20-connection HikariCP pool and 3 concurrent users, the 4th user's request would hang waiting for a connection. Fix: moved the LLM call outside the transaction — fetch data in transaction, close it, call LLM, open new transaction to save result. This is the idiomatic pattern: short transactions, LLM calls outside."

6. **On binary-search-on-answer:** "I actually applied this pattern outside of LeetCode — when tuning our Azure Container Apps auto-scale rules. We needed the minimum `minReplicas` that kept p99 latency under 500ms. That's a monotonic feasibility function on an integer answer space. I binary-searched replica counts in our staging load tests rather than trying every value linearly."

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 on each. Anything below 3 is a mandatory revisit before Week 4.

| Skill | Target | Your Score | Notes |
|---|---|---|---|
| Binary search — standard implementation from scratch | 5 | | |
| Search in Rotated Sorted Array (LC 33) — first attempt, no bugs | 4 | | |
| Koko Eating Bananas — identify binary-search-on-answer independently | 4 | | |
| Subsets via backtracking — with state restore | 4 | | |
| Permutations via backtracking — with used[] array | 4 | | |
| Combination Sum — correct start-index to avoid duplicates | 4 | | |
| Spring bean lifecycle — all 8 steps without notes | 5 | | |
| @Transactional: proxy type, self-invocation pitfall, propagations | 5 | | |
| ApplicationContext vs BeanFactory — 3+ real differences | 4 | | |
| Auto-configuration: how conditionals work, how to debug | 4 | | |
| Full request lifecycle whiteboard (DispatcherServlet → repo) | 4 | | |
| synchronized vs ReentrantLock — when to use each | 4 | | |
| volatile — guarantees and limits, correct use case | 4 | | |
| CompletableFuture — chain methods, thread pool choice, WebX connection | 4 | | |
| ThreadLocal — Tomcat pool reuse gotcha, Spring Security usage | 4 | | |
| Deadlock — write scenario, 4 Coffman conditions, prevention | 4 | | |

**Score interpretation:**
- 4–5 on all: you're ready for any Spring/concurrency question
- 3 on 1–2 items: revisit those topics in Week 4's daily review slot
- Below 3 on any item: schedule a focused 45-min re-drill before the next topic compounds on it

**Bonus check:** Can you connect every concept above to at least one specific line of code or decision in Smart360, WebX, or your API Gateway? If yes on 80%+ of rows: you are an extraordinary candidate, not just a prepared one.

---

*Built on interview-qa.md — no duplication, only depth. Week 4 picks up with: databases (query plans, indexes, transactions), system design foundations, and advanced DSA (trees + graphs).*
