# Week 2 · Day 1 — Mon Jul 06 — Binary Search + Search-on-Answer + Spring Bean Lifecycle

> The bedrock day. Binary search is the single most-tested pattern in coding interviews, and its "search-on-answer" variant is the one that separates people who memorized a template from people who understand the invariant. On the Spring side, the bean lifecycle and IoC/DI are what every "explain how Spring works under the hood" question reduces to — and the answer that scores is the one that names *where the AOP proxy appears*.

📌 **Study today:** Binary search — standard + rotated + 2D (LC 704, 33, 153, 74) · Spring bean lifecycle + IoC/DI · binary-search-on-answer (LC 875, 1011, 410) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Binary Search Fundamentals (~2.5 hr)

### Theory deep-dive

Binary search works on any **monotonic** decision space — not just sorted arrays. The mental model: you maintain a window `[left, right]` that is *guaranteed* to contain the answer (this is the **loop invariant**), and each iteration halves it. Get the invariant wrong and you get an off-by-one or an infinite loop (TLE), the two classic failure modes.

**The two templates — know which you're writing before you type:**

| | Exact-match search | Boundary / first-true search |
|---|---|---|
| Condition | `while (left <= right)` | `while (left < right)` |
| `mid` | `left + (right - left) / 2` | same |
| Shrink right | `right = mid - 1` (mid eliminated) | `right = mid` (mid still a candidate) |
| Shrink left | `left = mid + 1` | `left = mid + 1` |
| Returns | index or `-1` | `left` (== `right` at exit) |

**The overflow rule (say this in interviews):** use `mid = left + (right - left) / 2`, **not** `(left + right) / 2`. When `left + right > Integer.MAX_VALUE` the naive form overflows to a negative index → `ArrayIndexOutOfBoundsException`. This is the bug that lived in `java.util.Arrays.binarySearch` for nine years until 2006.

**Why `left + (right-left)/2` rounds down** — it biases `mid` toward `left`. With `left < right` template + `left = mid + 1` you must always make progress; if you wrote `right = mid` with a down-rounded mid and `left = mid` (no `+1`) you'd loop forever on a 2-element window.

### Worked example — LC 704 Binary Search (iterative, from scratch)

```java
// Classic exact-match template. O(log n) time, O(1) space.
public int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;   // invariant: answer in [left, right]
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;   // target in right half
        else right = mid - 1;                           // target in left half
    }
    return -1;   // not found
}
// nums=[-1,0,3,5,9,12], target=9 -> 4 ; target=2 -> -1
```
**Complexity:** O(log n) time, O(1) space. Test even length, odd length, empty array, target smaller than all / larger than all, target absent in the middle.

### Practice set

**1. LC 704 — Binary Search** (Easy)
- **Approach:** exact-match template above.
- **Key insight:** the `<=` with `right = mid - 1` cleanly eliminates `mid` each step; the window can become empty (`left > right`) which is the "not found" exit.
- **Complexity:** O(log n) / O(1). **Target: < 5 min.**
- **Follow-up:** return the *insertion point* instead of `-1` → return `left` (LC 35 Search Insert Position).

**2. LC 33 — Search in Rotated Sorted Array** (Medium)
- **Approach:** at each `mid`, **one half is always sorted**. Detect which: if `nums[left] <= nums[mid]` the left half is sorted, else the right half is. Then check if `target` lies within the sorted half's range; if yes recurse there, else the other half.
```java
public int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        if (nums[left] <= nums[mid]) {              // left half sorted
            if (nums[left] <= target && target < nums[mid]) right = mid - 1;
            else left = mid + 1;
        } else {                                    // right half sorted
            if (nums[mid] < target && target <= nums[right]) left = mid + 1;
            else right = mid - 1;
        }
    }
    return -1;
}
```
- **Key insight:** the sorted-half decision is **O(1)** — that is the whole trick. Comparing `target` against the *sorted* half's bounds is unambiguous; the unsorted half is where the pivot hides.
- **Complexity:** O(log n) / O(1).
- **Follow-up:** LC 81 with duplicates — when `nums[left] == nums[mid] == nums[right]` you can't tell which half is sorted, so `left++; right--`, degrading worst-case to **O(n)**. State the trade-off.

**3. LC 153 — Find Minimum in Rotated Sorted Array** (Medium)
- **Approach:** no target; find the inflection point. Use the boundary template (`left < right`, `right = mid`).
```java
public int findMin(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[right]) left = mid + 1;  // min is strictly right of mid
        else right = mid;                             // min is mid or left of it
    }
    return nums[left];   // left == right -> the minimum
}
```
- **Key insight:** compare to `nums[right]`, not `nums[left]`. If `nums[mid] > nums[right]`, the rotation point (minimum) must be in `(mid, right]`. This is the cleanest invariant for "find the pivot".
- **Complexity:** O(log n) / O(1).
- **Follow-up:** return the *index* of the rotation, or handle duplicates (LC 154 → again `right--` on a tie).

**4. LC 74 — Search a 2D Matrix** (Medium)
- **Approach:** rows sorted, each row starts after the previous row ends → treat the `m × n` grid as one flat sorted array of length `m*n`. Map a 1D index `mid` to `(mid / cols, mid % cols)`.
```java
public boolean searchMatrix(int[][] matrix, int target) {
    int rows = matrix.length, cols = matrix[0].length;
    int left = 0, right = rows * cols - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int val = matrix[mid / cols][mid % cols];   // index decode
        if (val == target) return true;
        else if (val < target) left = mid + 1;
        else right = mid - 1;
    }
    return false;
}
```
- **Key insight:** the `mid / cols`, `mid % cols` decode lets one binary search cover the whole grid in **O(log(m·n))**.
- **Complexity:** O(log(m·n)) / O(1). **Target: < 10 min.**
- **Follow-up:** LC 240 (each row & column sorted but rows don't continue) — that's *not* one flat array; use the staircase O(m+n) walk from top-right instead.

---

## Block B — Spring Bean Lifecycle + IoC/DI (~2 hr)

> Cross-ref: `../../02-oop-fundamentals.md` (constructors, static context), `../../03-oop-advanced.md` (Singleton/Factory patterns — Spring *is* an industrial Factory + Singleton registry), `../../05-enums-and-annotations.md` (`@Component`, `@Autowired` are annotations processed by reflection).

### Inversion of Control & Dependency Injection — the core idea

**IoC**: instead of your object calling `new` to build its collaborators, the *container* builds them and hands them to you. You declare *what* you need; Spring decides *when* and *how* to build it. **DI** is the mechanism: the container injects collaborators. The win is decoupling — a `OrderService` depends on the `PaymentGateway` *interface*, and Spring wires the concrete impl, so you swap Stripe→Adyen with one config change and test with a mock with zero framework.

### ApplicationContext vs BeanFactory

- **`BeanFactory`** — the bare container: lazy init by default, no auto post-processing. Minimal footprint.
- **`ApplicationContext`** — extends `BeanFactory` and adds: **eager singleton instantiation** at startup (fail-fast on a broken bean), **event publishing** (`ApplicationEventPublisher`), **i18n** (`MessageSource`), automatic **`BeanPostProcessor`/`BeanFactoryPostProcessor`** registration (this is what makes AOP/`@Transactional` work), and `@Async`/`@Scheduled` support.
- **Always use `ApplicationContext` in production.** The hierarchy: `BeanFactory` → `ListableBeanFactory` → `ApplicationContext` → `ConfigurableApplicationContext` → `AnnotationConfigApplicationContext` / `GenericWebApplicationContext` (the one Spring Boot's `SpringApplication.run` creates for web apps).

### DI modes — and why constructor injection wins

```java
@Service
public class ReportService {
    private final ReportRepository repo;     // final -> guaranteed set, immutable, thread-safe
    private final PdfRenderer renderer;
    // No @Autowired needed since Spring 4.3 when there's a single constructor.
    public ReportService(ReportRepository repo, PdfRenderer renderer) {
        this.repo = repo; this.renderer = renderer;
    }
}
```
- **Constructor injection (recommended):** dependencies are `final` and immutable, the object is never in a half-built state, mandatory deps are explicit, and it surfaces circular dependencies **at startup**. You can construct it in a plain unit test with `new` — no Spring needed.
- **Setter injection:** for genuinely optional dependencies.
- **Field injection (`@Autowired` on a field) — avoid:** hides dependencies, can't make fields `final`, and breaks plain `new`-based unit tests (you'd need reflection or a running context).

### `@Primary` vs `@Qualifier`

When two beans satisfy one type, Spring needs a tie-breaker. `@Primary` marks one as the default; `@Qualifier("name")` selects explicitly at the injection point. Use `@Qualifier` when both impls are intentional — e.g., a **Smart360** primary/replica `DataSource` split: `@Qualifier("primaryDataSource")` for writes, `@Qualifier("replicaDataSource")` for read-heavy report queries.

### The 8-step bean lifecycle — whiteboard all of it

```
1. Instantiate            constructor called (or factory method)
2. Populate properties    dependencies injected (@Autowired fields/setters)
3. Aware callbacks        BeanNameAware, BeanFactoryAware, ApplicationContextAware
4. BeanPostProcessor.postProcessBeforeInitialization()
5. Initialization         @PostConstruct -> InitializingBean.afterPropertiesSet() -> init-method
6. BeanPostProcessor.postProcessAfterInitialization()   <-- AOP PROXY CREATED HERE
7. Bean is ready / in use
8. Destruction            @PreDestroy -> DisposableBean.destroy() -> destroy-method
```

**The precision that scores:** AOP proxies (`@Transactional`, `@Async`, `@Cacheable`, `@PreAuthorize`) are created at **step 6** by `AnnotationAwareAspectJAutoProxyCreator` — itself a `BeanPostProcessor`. After step 6, **the object stored in the context IS the proxy**; the original target bean is wrapped, not replaced. This single fact explains the self-invocation pitfall you'll study tomorrow.

**Init-hook order (memorize):** `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → custom `init-method`. The `BeanPostProcessor` before/after callbacks bracket all three.

### Circular dependencies

```java
@Service class A { A(B b){} }   // A needs B
@Service class B { B(A a){} }   // B needs A  -> BeanCurrentlyInCreationException at startup
```
- **Constructor injection** fails fast at boot — *good*, the cycle is visible immediately.
- **Field/setter injection** resolves it lazily via Spring's three-level singleton cache (early reference exposure) — it "works" but hides a design smell.
- **Best fix:** redesign to break the cycle (extract a third bean, or invert one dependency to an event).
- **Last resort:** `@Lazy` on one injection point delays that bean's instantiation until first use — trade-off: first-call latency and init failures surface late.

### Project tie-ins

- **Smart360:** two `DataSource` beans selected by `@Qualifier` for primary/replica routing; `@PostConstruct` warms a reference-data cache after properties are injected.
- **Deep Fathom:** an analytics pipeline bean uses constructor injection so the (heavy) model loader is `final` and the bean fails fast at startup if the model path is misconfigured — you never serve a half-initialized analyzer.
- **WebX:** a provider-routing config bean is `ApplicationContextAware` to look up provider beans by name at runtime; constructor injection everywhere else surfaced a real A↔B cycle at boot that field injection had been silently hiding.

---

## Block C — DSA: Binary-Search-on-Answer (~1.5 hr)

### Theory

When the answer is a number in a **monotonic** range and you can write a boolean `feasible(candidate)` that is **false…false, true…true** (or the reverse) across the range, binary-search the *answer itself*, not an array index. You're searching the smallest (or largest) value that satisfies feasibility.

**Recognize it by phrasing:** "**minimum** speed/capacity/size such that…", "**maximum** … such that …", "**smallest largest** sum", "**fewest** days/splits". Whenever the brute force is "try every value and check", and *check* is monotonic, it's binary-search-on-answer.

**Template (find minimum feasible):**
```java
int left = MIN_ANSWER, right = MAX_ANSWER;
while (left < right) {
    int mid = left + (right - left) / 2;
    if (feasible(mid)) right = mid;   // mid works -> try smaller (mid still candidate)
    else               left = mid + 1;// mid fails -> need bigger
}
return left;                          // smallest feasible value
```

### Practice set

**1. LC 875 — Koko Eating Bananas** (Medium)
```java
public int minEatingSpeed(int[] piles, int h) {
    int left = 1, right = 0;
    for (int p : piles) right = Math.max(right, p);     // answer space [1, max pile]
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canFinish(piles, h, mid)) right = mid;
        else left = mid + 1;
    }
    return left;
}
private boolean canFinish(int[] piles, int h, int speed) {
    long hours = 0;
    for (int p : piles) hours += (p + speed - 1) / speed;  // ceil division
    return hours <= h;
}
```
- **Key insight:** answer space `[1, max(piles)]`; `feasible(speed)` = can finish in ≤ h hours. Ceil division `(p + speed - 1) / speed` avoids floating point. **Target: < 15 min.**
- **Complexity:** O(n · log(max pile)).

**2. LC 1011 — Capacity to Ship Packages Within D Days** (Medium)
- **Approach:** same template; `feasible(cap)` = can ship all packages in ≤ D days carrying ≤ `cap` per day. Answer space `[max(weights), sum(weights)]` — the low bound must hold the single heaviest package; the high bound ships everything in one day.
- **Key insight:** the low bound is `max`, not 1 — a capacity below the heaviest package can never work.
- **Complexity:** O(n · log(sum)). **Follow-up:** return the day-by-day split, not just the capacity.

**3. LC 410 — Split Array Largest Sum** (Hard)
- **Approach:** minimize the largest subarray sum across `k` contiguous splits. `feasible(maxSum)` = greedily walk the array starting a new subarray whenever adding the next element would exceed `maxSum`; count subarrays; feasible iff count ≤ k. Answer space `[max(nums), sum(nums)]`.
- **Key insight:** identical structure to Ship Packages — "minimum largest sum with ≤ k pieces" *is* "minimum capacity with ≤ k days". Recognizing the isomorphism is the whole point.
- **Complexity:** O(n · log(sum)). **Follow-up:** the DP solution is O(n²·k) — mention that binary-search-on-answer is strictly better here.

---

## 💻 Practice coding questions

1. **LC 704 Binary Search** — exact-match template. Return `-1` when absent.
2. **LC 35 Search Insert Position** — same loop, return `left` (the insertion point).
3. **LC 33 Search in Rotated Sorted Array** — identify the sorted half in O(1).
4. **LC 81 Search in Rotated Sorted Array II** — duplicates → `left++; right--` on the ambiguous tie; worst case O(n).
5. **LC 153 Find Minimum in Rotated Sorted Array** — compare `nums[mid]` to `nums[right]`.
6. **LC 74 Search a 2D Matrix** — flatten with `mid/cols`, `mid%cols`.
7. **LC 34 Find First and Last Position** — two boundary searches (lower-bound + upper-bound).
8. **LC 875 Koko Eating Bananas** — feasibility = finish in ≤ h hours.
9. **LC 1011 Ship Packages in D Days** — answer space `[max, sum]`.
10. **LC 410 Split Array Largest Sum** — minimize the largest piece across k splits.
11. **LC 540 Single Element in a Sorted Array** — binary search on parity of indices; O(log n).
12. **LC 4 Median of Two Sorted Arrays** (Hard) — binary search the partition of the smaller array; O(log(min(m,n))). Stretch goal.

> For each, state approach aloud before coding, write complexity, then code without hints. Re-time LC 33 and LC 875 — target ≤ 10 min each.

---

## 🎤 Interview questions

1. **Why `left + (right - left) / 2` instead of `(left + right) / 2`?** Avoids integer overflow when `left + right > Integer.MAX_VALUE`, which would produce a negative index. Real bug in JDK's `Arrays.binarySearch` until 2006.
2. **When do you use `while (left <= right)` vs `while (left < right)`?** `<=` for exact match (you eliminate `mid` with `right = mid - 1`); `<` for boundary/first-true searches where `mid` is still a candidate (`right = mid`). Mixing them causes off-by-one or infinite loops.
3. **How do you binary search a rotated array — what's the O(1) decision?** One half is always sorted; compare `nums[left]` to `nums[mid]` to find which, then check whether `target` lies in the sorted half's range. The unsorted half holds the pivot.
4. **What changes with duplicates in a rotated array?** When `nums[left] == nums[mid] == nums[right]` you can't decide the sorted half, so `left++; right--`, degrading worst case to O(n).
5. **What is binary-search-on-answer? Give a real example from your work.** Searching a monotonic answer space with an O(n) feasibility test. WebX: "minimum thread-pool size such that p99 < 2s" — load-test feasibility over `[1, maxThreads]`. Textbook form: Koko.
6. **What phrasings signal binary-search-on-answer?** "minimum maximum", "maximum minimum", "smallest largest sum", "fewest/most X to achieve Y", "largest X such that Y".
7. **Why is the lower bound `max(weights)` and not 1 in Ship Packages?** A daily capacity below the single heaviest package can never ship it — that value is infeasible, so it can't be the answer.
8. **Binary search isn't only for sorted arrays — explain.** It works on any monotonic predicate. The array is just one monotonic space; `feasible()` is the general form.
9. **What is IoC and how does DI implement it?** IoC inverts who constructs collaborators — the container, not your code. DI is the delivery mechanism: the container injects dependencies you declared.
10. **ApplicationContext vs BeanFactory — name three real differences.** Eager singleton init (fail-fast), automatic `BeanPostProcessor` registration (enables AOP/`@Transactional`), event publishing + i18n. Production always uses `ApplicationContext`.
11. **Walk the 8-step bean lifecycle. Where is the AOP proxy created?** Instantiate → populate → aware → BPP-before → init (`@PostConstruct` → `afterPropertiesSet` → init-method) → **BPP-after (proxy created here by `AnnotationAwareAspectJAutoProxyCreator`)** → ready → destroy.
12. **What's the init-hook order?** `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → custom `init-method`.
13. **Why prefer constructor injection over field injection?** Immutable `final` fields, no half-built object, mandatory deps explicit, cycles caught at startup, and unit-testable with plain `new`.
14. **`@Primary` vs `@Qualifier`?** `@Primary` is a default tie-breaker; `@Qualifier("name")` is an explicit per-injection selection. Use `@Qualifier` for multiple intentional impls (primary/replica DataSource).
15. **How does Spring resolve a circular dependency?** Field/setter via the three-level singleton cache (early reference exposure); constructor injection cannot and throws `BeanCurrentlyInCreationException` at startup — which is the *correct* signal to redesign.
16. **What does `@Lazy` do to a bean, and when would you use it?** Delays instantiation until first access; a last-resort cycle-breaker, at the cost of first-call latency and late-surfacing init failures.
17. **Is a Spring singleton thread-safe?** The container guarantees one instance, not thread safety. Stateless singletons are safe; mutable shared state needs synchronization.
18. **After step 6, what object does another bean get when it `@Autowired`s a `@Transactional` service?** The CGLIB proxy, not the raw target — which is why `this.method()` self-calls bypass the proxy (tomorrow's topic).

---

## ✅ Self-check

1. Can you identify which half of a rotated array is sorted as an O(1) decision? If not, re-code LC 33 before Tuesday.
2. What phrasings signal binary-search-on-answer? (minimum maximum / maximum minimum / largest X such that Y / fewest-most X to achieve Y.)
3. Write the 8-step bean lifecycle from memory and circle the step where the AOP proxy is created (step 6).
4. State the difference between the `<=` and `<` binary-search templates and which shrink (`right = mid - 1` vs `right = mid`) pairs with each.
5. Why is constructor injection safer than field injection for catching circular dependencies?

---

*Nav: ← [Week 1](../week-01/) · [Week 2](README.md) · [02-tue-jul-07.md](02-tue-jul-07.md) →*
