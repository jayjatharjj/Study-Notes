# Cheat-Sheet — Java & Spring Rapid Reference

> Offline rapid-revision. One-liners + tables. Cross-refs: modules 06/07/08 + interview-qa.md.

## Core Java

- `==` compares references (identity); `.equals()` compares logical value. For primitives `==` compares value.
- **String immutability**: once created, never changes — enables safe sharing, caching, hashing (cached `hashCode`), thread-safety, use as `HashMap` keys / JWT tokens.
- **String pool**: string literals interned in a pool → identical literals share one object. `new String("x")` forces a new heap object (`==` fails); `.intern()` puts/returns the pooled copy.
- **`equals`/`hashCode` contract**: equal objects MUST have equal hashCodes; unequal may collide. Override both together. `equals` reflexive/symmetric/transitive/consistent. Break it → broken `HashMap`/`HashSet` lookups.
- **`final` / `finally` / `finalize`**:

  | keyword | meaning |
  |---|---|
  | `final` | can't reassign var / override method / extend class |
  | `finally` | block always runs after try/catch (cleanup) |
  | `finalize` | GC pre-collection hook — deprecated, never rely on it (use try-with-resources / `Cleaner`) |

- **Checked vs unchecked**: checked = `Exception` subclasses (not `RuntimeException`) → compiler forces catch/declare (recoverable, e.g. `IOException`). Unchecked = `RuntimeException`/`Error` → programming bugs (`NPE`, `IllegalArgument`), not enforced.
- **Pass-by-value**: Java is always pass-by-value. For objects the *reference value* is copied — you can mutate the object, but reassigning the param doesn't affect the caller.
- **Autoboxing + Integer cache**: `Integer` caches `-128..127`. `Integer a=100,b=100; a==b` → `true` (cached); `127` → true, `128` → false. Always compare wrappers with `.equals()`.
- **Generics / type erasure**: generics are compile-time only; type params erased to `Object`/bound at runtime. No `new T[]`, no `instanceof List<String>`, no primitive type args.
- **PECS** — *Producer Extends, Consumer Super*: `? extends T` to read out (producer), `? super T` to write in (consumer). `List<? extends Number>` read-only-ish; `List<? super Integer>` write Integers.

## Collections

| List | Backing | Access | Insert/remove | Use when |
|---|---|---|---|---|
| `ArrayList` | array | O(1) random | O(n) middle, amortized O(1) tail | default; index-heavy reads |
| `LinkedList` | doubly-linked | O(n) | O(1) at known node | rare; queue/deque ends |
| `ArrayDeque` | circular array | — | O(1) both ends | stack/queue (beats `Stack`/`LinkedList`) |

- **HashMap internals**: array of buckets; `put` → `hashCode(k)` → spread `(h ^ (h >>> 16))` → index. Walk bucket via `equals`. Load factor **0.75** → resize (double) at 75% full. Bucket → **Red-Black tree** at **≥8 entries AND table capacity ≥64** (`MIN_TREEIFY_CAPACITY`); below 64 it resizes instead. Resize splits each bucket into low/high halves (no full rehash needed).
- **ConcurrentHashMap**: bucket-level concurrency — **CAS** on empty-bin insert, **`synchronized` on the bin head** for collisions. No whole-map lock (unlike `Hashtable`/`synchronizedMap`). Null keys/values not allowed.
- **fail-fast vs fail-safe**:

  | | fail-fast | fail-safe |
  |---|---|---|
  | Examples | `ArrayList`, `HashMap` iterators | `CopyOnWriteArrayList`, `ConcurrentHashMap` |
  | On concurrent mod | throws `ConcurrentModificationException` (modCount check) | iterates a snapshot, no throw |

- **Comparable vs Comparator**: `Comparable.compareTo` = natural order, one ordering, in the class. `Comparator` = external, multiple orderings (`comparing().thenComparing().reversed()`).
- **Pick a collection**: ordered + indexed → `ArrayList`; unique → `HashSet`; unique + sorted → `TreeSet`; unique + insertion-order → `LinkedHashSet`; key-value → `HashMap`; sorted map → `TreeMap`; LRU → `LinkedHashMap` (access-order + `removeEldestEntry`); concurrent → `ConcurrentHashMap`/`CopyOnWriteArrayList`.

## Concurrency

- **Thread vs Runnable**: prefer `Runnable`/`Callable` (decouples task from execution, allows pooling, leaves `extends` free). `Callable` returns a value + throws checked; `Runnable` doesn't.
- **`synchronized` vs `ReentrantLock`**:

  | | `synchronized` | `ReentrantLock` |
  |---|---|---|
  | Acquire | implicit (block/method) | explicit `lock()`/`unlock()` (in `finally`) |
  | Features | simple, auto-release | `tryLock(timeout)`, fairness, interruptible, multiple `Condition`s |

- **`volatile`**: guarantees **visibility** + ordering, NOT atomicity. Good for flags (`volatile boolean running`); fails for `count++` (read-modify-write).
- **`Atomic*`** (`AtomicInteger`/`AtomicLong`/`AtomicReference`): lock-free CAS, atomic compound ops — use for counters instead of `synchronized`.
- **ThreadLocal**: per-thread variable. **Pool leak trap**: pooled threads are reused → always `remove()` in `finally`, else value leaks to the next task / memory.
- **ExecutorService + pool sizing**: CPU-bound → ~`N_cores` (`+1`); I/O-bound → higher (`cores * (1 + wait/compute)`). Prefer `ThreadPoolExecutor` with **bounded queue** + `CallerRunsPolicy` for backpressure; always `shutdown()` + `awaitTermination()`.
- **CompletableFuture**: `supplyAsync` start; `thenApply` (sync map) vs `thenCompose` (flatMap, chain another future); `thenCombine` (join two); `allOf(...).join()` (wait all); `exceptionally` / `handle` for errors.
- **Deadlock — 4 Coffman conditions** (all needed): mutual exclusion, hold-and-wait, no-preemption, circular-wait. Avoid: global lock ordering, `tryLock(timeout)`, single lock.
- **happens-before**: if A happens-before B, A's writes are visible to B. Sources: `synchronized` unlock→lock, `volatile` write→read, thread `start()`→run, run→`join()`.

## JVM & GC

- **Memory areas**: **Heap** (objects, GC'd, shared) · **Stack** (per-thread frames, locals, refs) · **Metaspace** (class metadata, native mem, replaced PermGen in 8) · PC register · native method stack.
- **Generational GC**: Young (Eden + 2 Survivors) → minor GC; survivors promoted to **Old gen** → major/full GC. Weak generational hypothesis: most objects die young.
- **G1 vs ZGC**:

  | | G1 | ZGC |
  |---|---|---|
  | Default since | JDK 9 | opt-in (prod-ready 15+) |
  | Heap | small–large (≤ ~tens GB) | very large (TB) |
  | Pause | ~tens of ms, tunable `MaxGCPauseMillis` | sub-millisecond, concurrent |
  | Pick when | general default | huge heap + strict latency |

- **Common flags**: `-Xms`/`-Xmx` (heap min/max), `-XX:+UseG1GC` / `-XX:+UseZGC`, `-XX:MaxGCPauseMillis`, `-XX:MaxMetaspaceSize`, `-XX:+HeapDumpOnOutOfMemoryError`.
- **OOM types**: `Java heap space` (too many live objects / leak), `GC overhead limit exceeded`, `Metaspace` (classloader leak), `unable to create native thread` (too many threads), `Direct buffer memory`.

## Spring / Boot 3

- **Bean lifecycle (8 steps)**: 1 instantiate → 2 DI → 3 `*Aware` callbacks → 4 `BeanPostProcessor.before` → 5 `@PostConstruct`/`afterPropertiesSet` → **6 `BeanPostProcessor.after` (AOP proxies created here)** → 7 ready → 8 `@PreDestroy`/`destroy` on shutdown. AOP/`@Transactional`/`@Async`/`@Cacheable` all wrap beans at step 6.
- **IoC/DI**: container owns object creation + wiring. **Constructor injection** (preferred) → immutable, `final` fields, mandatory deps, testable, no circular-dep hiding. Field injection → concise but hides deps, no `final`, hard to unit-test.
- **`@Transactional`**: proxy (JDK dynamic / CGLIB) intercepts → opens tx via `PlatformTransactionManager` → commits, or rolls back on **unchecked** exceptions only (checked → `rollbackFor`).
  - **Propagation**: `REQUIRED` (default, join existing) · `REQUIRES_NEW` (suspend + new tx, e.g. audit logging) · `NESTED` · `SUPPORTS` · `MANDATORY`.
  - **Isolation**: `READ_COMMITTED` (typical) → `REPEATABLE_READ` → `SERIALIZABLE` (more isolation, less concurrency).
  - **Self-invocation pitfall**: calling a `@Transactional` method from the same class bypasses the proxy → no tx. Also ignored on `private`/`final` methods. **Fix**: self-inject the bean, split into another bean, or `AopContext.currentProxy()`.
  - **Pool starvation**: a long `@Transactional` holds its DB connection the whole method; never do remote/REST/LLM I/O inside a tx — keep them short.
- **Bean scopes**: `singleton` (default, one per container) · `prototype` (new each inject) · web: `request` · `session` · `application` · `websocket`.
- **Auto-config**: Boot 3 reads `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` → `@AutoConfiguration` classes applied via `@ConditionalOnClass` / `@ConditionalOnMissingBean` / `@ConditionalOnProperty`. Exclude with `@SpringBootApplication(exclude = ...)`.
- **Request lifecycle**: client → `DispatcherServlet` (front controller) → `HandlerMapping` → `@Controller` → `@Service` → `@Repository`/JPA → response via `HttpMessageConverter` (Jackson) → client.
- **`@ControllerAdvice`** + `@ExceptionHandler`: centralized REST error handling → consistent JSON error bodies + status codes (pairs with `ResponseEntity` / `ProblemDetail`).
- **JWT / OAuth2 flow**: login → server issues signed **JWT** (stateless, header.payload.signature) → client sends `Authorization: Bearer <token>` → filter validates signature/expiry → sets `SecurityContext`. OAuth2: client → authorization server → auth code → access token → resource server.
- **JPA N+1**: 1 query for parents + N for each lazy association. **Fixes**: `JOIN FETCH` in JPQL, `@EntityGraph` on the repo method, batch fetching (`@BatchSize`), or DTO projections. Diagnose via Hibernate statistics / SQL logs.
- **HikariCP sizing**: small pool wins — `~ (2 * cores) + effective_spindles`; tune `maximumPoolSize`, `connectionTimeout`, `leakDetectionThreshold`. Slow `@Transactional` checks out a connection for its whole duration → starves the pool under load.
- **Profiles**: `@Profile("dev")`, `application-{profile}.yml`, activate via `spring.profiles.active`.
- **Testing slices**:

  | Annotation | Loads | For |
  |---|---|---|
  | `@WebMvcTest` | MVC layer + `MockMvc` (mock services) | controllers |
  | `@DataJpaTest` | JPA + in-memory/embedded DB, rolls back | repositories |
  | `@SpringBootTest` | full context | integration / e2e |

## Modern Java (17 / 21)

- **Records**: immutable transparent data carrier — auto `equals`/`hashCode`/`toString`/accessors. Use for DTOs, projections, value objects, multi-return. **Not for JPA `@Entity`** (need no-arg ctor + mutable fields); implicitly `final` (no inheritance). Add a **compact constructor** for validation / defensive copies (`List.copyOf`).

  ```java
  public record Money(long cents, String currency) {
      public Money { if (cents < 0) throw new IllegalArgumentException(); }
  }
  ```

- **Sealed types**: `sealed interface Shape permits Circle, Rectangle {}` — closed hierarchy; each subtype must be `final` / `sealed` / `non-sealed`. Records satisfy `final` automatically → `sealed interface + record` = algebraic data type, enables exhaustive `switch` (drop `default`).
- **Switch + pattern matching**: switch *expressions* (`->`, `yield`), exhaustiveness checked over enum/sealed. **Type patterns** + **record deconstruction** (21):

  ```java
  String r = switch (shape) {
      case Circle(Point c, double rad) -> "circle r=" + rad;
      case Rectangle r2               -> "rect";
  };
  ```

- **Text blocks**: `"""` multi-line strings for embedded JSON/SQL/HTML; `.formatted(...)` to interpolate.
- **Virtual threads (21)**: lightweight JVM-managed threads; on blocking I/O the vthread **unmounts** from its carrier (platform) thread → thread-per-request scales to millions in plain blocking style. **Do NOT pool** them — `Executors.newVirtualThreadPerTaskExecutor()`, one per task. **Pinning**: vthread stuck to its carrier (can't unmount) inside `synchronized` blocks or native/JNI calls → use `ReentrantLock` instead. Pools/`Semaphore` still for CPU-bound or rate-limiting a scarce resource.
- **Jakarta / Boot 3 migration**: JDK 17 baseline; `javax.*` → `jakarta.*` namespace (Spring 6 / Boot 3 require it) — update imports, persistence/validation/servlet APIs, and dependency versions.
