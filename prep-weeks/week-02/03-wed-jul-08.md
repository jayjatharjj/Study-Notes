# Week 2 · Day 3 — Wed Jul 08 — Stacks + Monotonic Stack + Spring Boot Autoconfig & Request Lifecycle

> The monotonic stack is one pattern that unlocks an entire family of "next greater / span / histogram" problems — recognize it once and five LeetCode mediums collapse into one. On the Spring side, today is the whiteboard day: the full request lifecycle (Tomcat → DispatcherServlet → controller → service proxy → repo → back) and how auto-configuration actually decides what beans to create. Being able to draw both cold is what "senior" looks like in a system-knowledge round.

📌 **Study today:** Stacks + monotonic stack (LC 20, 155, 739, 496, 503, 901, 84) · Spring Boot auto-config + full request lifecycle · queues / stack drills · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Stacks & Monotonic Stack (~2.5 hr)

### Theory deep-dive

A **stack** is LIFO. In Java, back it with `Deque<Integer> stack = new ArrayDeque<>()` — **never the legacy `java.util.Stack`** (it extends `Vector`, so every op is `synchronized` → slower, and its iteration order is bottom-to-top which surprises people). `ArrayDeque` is array-backed, unsynchronized, and the official recommendation (`push`/`pop`/`peek`).

The **monotonic stack** keeps its elements (usually **indices**, so you can compute distances) in increasing or decreasing order. When a new element *breaks* the monotonic order, you **pop and resolve** the popped elements — at the moment you pop element X, the current element is X's answer (its next-greater, or the right boundary of its span/rectangle). Each element is pushed and popped **at most once** → **amortized O(n)** over the whole pass, even though there's a nested while loop.

**Decision guide:**
- "**next greater / next smaller** element" → monotonic stack.
- "**span**" (how far back the trend holds) → monotonic stack of (value, span) or indices.
- "**largest rectangle / max area**" under a histogram or in a matrix → monotonic increasing stack of indices.
- "**circular** next greater" → iterate `2n` indices with `i % n`.

### Worked example — LC 739 Daily Temperatures

```java
// Monotonic DECREASING stack of indices. For each day, how many days until a warmer one?
public int[] dailyTemperatures(int[] t) {
    int n = t.length;
    int[] res = new int[n];
    Deque<Integer> stack = new ArrayDeque<>();    // indices, temps decreasing bottom->top
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && t[i] > t[stack.peek()]) {
            int prev = stack.pop();               // t[i] is the next-warmer day for prev
            res[prev] = i - prev;                 // distance in days
        }
        stack.push(i);
    }
    return res;                                   // unresolved indices stay 0 (no warmer day)
}
// t=[73,74,75,71,69,72,76,73] -> [1,1,4,2,1,1,0,0]
```
**Complexity:** O(n) time (each index pushed/popped once), O(n) space. Trace `[73,74,75,71,69,72,76,73]` by hand first.

### Practice set

**1. LC 20 — Valid Parentheses** (Easy)
- **Approach:** push openers; on a closer, the stack top must be its matching opener (use a map `)→( ]→[ }→{`).
- **Key insight:** a closer on an *empty* stack → invalid; leftover openers at the end → invalid. Both edge cases trip people.
- **Complexity:** O(n) / O(n). **Target: ≤ 10 min. Follow-up:** LC 1249 — remove the minimum invalid parentheses.

**2. LC 155 — Min Stack** (Medium)
- **Approach:** push the running min alongside each value (a second stack, or a `long[]{val, curMin}` pair).
```java
class MinStack {
    private final Deque<int[]> st = new ArrayDeque<>();   // {value, minSoFar}
    public void push(int x) {
        int min = st.isEmpty() ? x : Math.min(x, st.peek()[1]);
        st.push(new int[]{x, min});
    }
    public void pop()      { st.pop(); }
    public int  top()      { return st.peek()[0]; }
    public int  getMin()   { return st.peek()[1]; }       // O(1)
}
```
- **Key insight:** O(1) `getMin` by storing the min *with* each element — no recomputation on pop.
- **Complexity:** all ops O(1). **Follow-up:** save space with a single value stack + a min stack that only pushes when a new ≤ min appears.

**3. LC 739 — Daily Temperatures** (Medium) — worked above. Decreasing index stack.

**4. LC 496 — Next Greater Element I** (Easy)
- **Approach:** one monotonic pass over `nums2` building `Map<value, nextGreater>`; then O(1) lookups for each `nums1` value.
- **Key insight:** the base case of the family — decouple the pre-compute (over `nums2`) from the query (over `nums1`).
- **Complexity:** O(n + m). **Follow-up:** what if values repeat? (Problem guarantees distinct; otherwise key by index.)

**5. LC 503 — Next Greater Element II** (Medium)
- **Approach:** **circular** array → iterate `2n` indices using `i % n`; only **push** during the first pass (`i < n`), resolve on both passes.
```java
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] res = new int[n];
    Arrays.fill(res, -1);
    Deque<Integer> stack = new ArrayDeque<>();        // indices, values decreasing
    for (int i = 0; i < 2 * n; i++) {
        int cur = nums[i % n];
        while (!stack.isEmpty() && nums[stack.peek()] < cur) res[stack.pop()] = cur;
        if (i < n) stack.push(i);                     // push only in the first lap
    }
    return res;
}
```
- **Key insight:** the second lap resolves elements that wrap around; pushing only in lap 1 avoids double-counting.
- **Complexity:** O(n) / O(n).

**6. LC 901 — Online Stock Span** (Medium)
- **Approach:** the canonical **span** problem; stack of `(price, span)` pairs. On each new price, pop while top price ≤ current, accumulating popped spans into the current span.
```java
class StockSpanner {
    private final Deque<int[]> st = new ArrayDeque<>();   // {price, span}
    public int next(int price) {
        int span = 1;
        while (!st.isEmpty() && st.peek()[0] <= price) span += st.pop()[1];
        st.push(new int[]{price, span});
        return span;
    }
}
```
- **Key insight:** by *absorbing* popped spans, each call is amortized O(1); you never re-scan resolved prices.
- **Complexity:** amortized O(1) per call. **Follow-up:** why amortized? Each price is pushed once and popped once across all calls.

**7. LC 84 — Largest Rectangle in Histogram** (Hard) — the flagship
- **Approach:** monotonic **increasing** index stack. For each bar, while the current height is *less* than the stacked bar's height, pop: the popped bar's rectangle extends from the new stack top (left boundary) to the current index (right boundary).
```java
public int largestRectangleArea(int[] h) {
    int n = h.length, best = 0;
    Deque<Integer> st = new ArrayDeque<>();
    for (int i = 0; i <= n; i++) {
        int curH = (i == n) ? 0 : h[i];               // sentinel 0 drains the stack
        while (!st.isEmpty() && h[st.peek()] >= curH) {
            int height = h[st.pop()];
            int leftBound = st.isEmpty() ? -1 : st.peek();
            int width = i - leftBound - 1;            // exclusive bounds on both sides
            best = Math.max(best, height * width);
        }
        st.push(i);
    }
    return best;
}
// h=[2,1,5,6,2,3] -> 10  (bars 5 and 6: height 5 x width 2 = 10)
```
- **Key insight:** the **sentinel 0** appended at the end (`i == n`) forces the stack to drain — the most common bug is leaving bars un-resolved. Width uses *exclusive* boundaries: `i - leftBound - 1`.
- **Complexity:** O(n) / O(n). **Trace `[2,1,5,6,2,3]` → 10. Follow-up:** LC 85 Maximal Rectangle — run this per row over a built-up histogram of consecutive 1s.

---

## Block B — Auto-Configuration + Full Request Lifecycle (~2 hr)

> Cross-ref: `../../05-enums-and-annotations.md` (`@Conditional*`, `@SpringBootApplication` are composed meta-annotations), `../../03-oop-advanced.md` (the proxy/decorator pattern behind interceptors and `@Transactional`), `../../06-concurrency-and-collections.md` (Tomcat's thread-per-request model feeds the lifecycle).

### Auto-configuration mechanics

Spring Boot's "convention over configuration" comes from **auto-configuration classes** that are conditionally applied.

- **Discovery:** Boot 3.x reads `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (a plain newline-delimited list of class names). This **replaced** the old `spring.factories` `EnableAutoConfiguration` key from Boot 2.x. Spring loads *all* listed classes, but each is gated by `@Conditional*` annotations.
- **Key conditionals:**
  - `@ConditionalOnClass(DataSource.class)` — apply only if that class is on the classpath (i.e., the JDBC starter is present).
  - `@ConditionalOnMissingBean(DataSource.class)` — back off if *you* already defined that bean (your bean wins — this is how Boot lets you override defaults).
  - `@ConditionalOnProperty("spring.datasource.url")` — apply only if a property is set.
  - Also: `@ConditionalOnBean`, `@ConditionalOnWebApplication`, `@ConditionalOnExpression`.

```java
@AutoConfiguration
@ConditionalOnClass(RedisTemplate.class)
@ConditionalOnProperty(prefix = "spring.data.redis", name = "host")
public class MyRedisAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean          // user-defined RedisTemplate wins
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory cf) {
        RedisTemplate<String,Object> t = new RedisTemplate<>();
        t.setConnectionFactory(cf);
        return t;
    }
}
```

- **Debugging — the interviewer favorite:** set `debug=true` (or `logging.level.org.springframework.boot.autoconfigure=DEBUG`) → Boot prints a **CONDITIONS EVALUATION REPORT** listing *Positive matches* (which auto-configs fired and why), *Negative matches* (which were skipped and the exact failing condition), and *Exclusions*. Quote this in interviews: "I used the conditions report to find a custom `DataSource` wasn't picked up because `@ConditionalOnMissingBean` had already matched a default."
- **Disabling one:** `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)` — used in test slices / unit tests to avoid a real DB.
- **`@SpringBootApplication` is composed of** `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.
- **Custom starter** = an `@AutoConfiguration` class with `@ConditionalOn*` guards, registered in `AutoConfiguration.imports`. Every starter (`spring-boot-starter-data-redis`, etc.) works exactly this way.

### Full request lifecycle — whiteboard without notes

```
Client --> HTTP request
  Tomcat (Servlet container) — picks a worker thread from its pool (default max 200)
   --> SecurityFilterChain (servlet Filters — JWT validation runs HERE, before the DispatcherServlet)
   --> DispatcherServlet.doDispatch()        (the front controller)
        --> HandlerMapping — URL -> @RequestMapping method (builds a HandlerExecutionChain)
        --> HandlerInterceptor.preHandle()
        --> HandlerAdapter — binds args: @RequestBody (HttpMessageConverter/Jackson),
                              @PathVariable, @RequestParam; runs @Valid
        --> @RestController method
              --> @Service — CGLIB proxy checks @Transactional, begins tx
                    --> @Repository / Spring Data JPA — Hibernate session, SQL
              <-- returns DTO
        <-- HandlerInterceptor.postHandle()
        --> HttpMessageConverter serialises the return value to JSON
        --> HandlerInterceptor.afterCompletion()
   <-- HTTP response
```

**Two follow-ups interviewers always ask:**
1. **Where does `@ControllerAdvice` / `@ExceptionHandler` intercept?** Not a filter, not a servlet. When the handler *throws*, `DispatcherServlet` catches it and delegates to a `HandlerExceptionResolver` → `ExceptionHandlerExceptionResolver` → your `@ExceptionHandler`. It runs **after** the controller throws, inside the DispatcherServlet, *before* the response is written.
2. **Where does `@PreAuthorize` run vs the security filter?** `@PreAuthorize` is **AOP on the service method** (inside the service call, via a method interceptor) — distinct from the `SecurityFilterChain`, which is a *servlet filter* that runs *before* the DispatcherServlet (coarse, URL-level). Filter = perimeter auth; `@PreAuthorize` = fine-grained, per-method.

**Filter vs Interceptor (a common confusion):** a `Filter` is a Servlet-spec component, sees raw `ServletRequest`/`Response`, runs before/after the DispatcherServlet (security, CORS, logging). A `HandlerInterceptor` is Spring MVC, runs *inside* the DispatcherServlet around the handler, and knows which handler matched.

### Project tie-ins

- **Smart360:** `@ConditionalOnProperty` toggles a feature-flag bean per environment; the conditions report once revealed a missing replica `DataSource` in staging.
- **WebX (API Gateway):** `@ConditionalOnProperty` toggles the circuit-breaker bean per environment; a custom `@AutoConfiguration` packages the provider-routing config as an internal starter; CGLIB proxies wrap the filter beans.
- **Deep Fathom:** a `@ConditionalOnClass` guard means the heavy analytics auto-config only loads when the optional ML module is on the classpath.

---

## Block C — Queues + Stack Consolidation (~1.5 hr)

### Queue / Deque model

- `ArrayDeque` as a **queue**: `addLast()` / `pollFirst()`. As a **stack**: `push()` (addFirst) / `pop()` (removeFirst). One class, both roles.
- `ArrayDeque` **beats `LinkedList`** for both queue and stack: array-backed, no per-node object allocation, better cache locality. Use `LinkedList` only when you need `List` indexing too.
- A **monotonic deque** underlies **Sliding Window Maximum (LC 239)** — keep indices of a decreasing window; the front is always the window max; evict indices that fall out of the window from the front, and smaller-than-current from the back. (Preview for later weeks.)

```java
// LC 239 preview — monotonic deque, O(n)
public int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> dq = new ArrayDeque<>();   // indices, values decreasing
    int n = nums.length, idx = 0;
    int[] res = new int[n - k + 1];
    for (int i = 0; i < n; i++) {
        while (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();   // out of window
        while (!dq.isEmpty() && nums[dq.peekLast()] < nums[i]) dq.pollLast(); // smaller -> useless
        dq.offerLast(i);
        if (i >= k - 1) res[idx++] = nums[dq.peekFirst()];
    }
    return res;
}
```

### Consolidation drills

- **Re-solve LC 739 Daily Temperatures cold** — no peeking at the morning's solution. Target ≤ 18 min.
- Write the **time/space complexity** for *every* stack problem from Block A in your notes (all O(n) time; O(n) space).
- One-line each: "Valid Parens = match on a stack", "Min Stack = store min alongside", "Daily Temps / NGE = monotonic decreasing", "Stock Span = absorb popped spans", "Histogram = increasing stack + sentinel".

---

## 💻 Practice coding questions

1. **LC 20 Valid Parentheses** — match closers against the stack top; check empty + leftover.
2. **LC 155 Min Stack** — store `{value, minSoFar}` for O(1) getMin.
3. **LC 739 Daily Temperatures** — decreasing index stack; result = distance.
4. **LC 496 Next Greater Element I** — pre-compute over nums2 into a map.
5. **LC 503 Next Greater Element II** — circular; iterate 2n with `i % n`, push only lap 1.
6. **LC 901 Online Stock Span** — `{price, span}` stack; absorb popped spans.
7. **LC 84 Largest Rectangle in Histogram** — increasing stack + sentinel 0; width = `i - left - 1`.
8. **LC 85 Maximal Rectangle** — per-row histogram + LC 84.
9. **LC 232 Implement Queue using Stacks** — two stacks (in/out), amortized O(1).
10. **LC 225 Implement Stack using Queues** — single queue, rotate on push.
11. **LC 1047 Remove All Adjacent Duplicates** — stack; pop on match.
12. **LC 394 Decode String** — two stacks (counts + strings) for nested `k[...]`.
13. **LC 239 Sliding Window Maximum** — monotonic deque, O(n) (stretch).
14. **LC 856 Score of Parentheses** — stack of running scores.

> State whether each is "plain stack" or "monotonic stack" before coding, and the complexity after.

---

## 🎤 Interview questions

1. **Explain a monotonic stack in one sentence.** A stack kept in increasing or decreasing order; when a new element breaks the order you pop and resolve the popped elements, which makes next-greater / span / histogram problems O(n) amortized.
2. **Why `ArrayDeque` instead of `java.util.Stack`?** `Stack` extends `Vector` (synchronized → slower) and iterates bottom-to-top; `ArrayDeque` is unsynchronized, array-backed, and the official recommendation.
3. **Why is a monotonic-stack pass O(n) despite the nested while loop?** Each element is pushed once and popped once across the whole pass → amortized O(1) per element.
4. **What's the classic bug in Largest Rectangle in Histogram?** Not draining the stack after the main loop — fix with a sentinel 0-height bar at the end.
5. **How do you handle the circular array in Next Greater Element II?** Iterate `2n` indices using `i % n`, pushing only during the first lap so wrap-around elements get resolved without double-counting.
6. **Min Stack with O(1) getMin — how?** Store the running min alongside each pushed value (pair or second stack); pop both together.
7. **Implement a queue with two stacks — complexity?** Push onto `in`; on poll, if `out` is empty, drain `in` into `out`; amortized O(1) per op.
8. **How does Spring Boot decide which beans to auto-configure?** It loads every class in `AutoConfiguration.imports`, then each `@Conditional*` annotation gates whether the bean is actually registered (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`).
9. **What replaced `spring.factories` in Boot 3?** `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.
10. **How do you debug what auto-configured (and what didn't)?** Set `debug=true` → read the CONDITIONS EVALUATION REPORT (positive/negative matches with the exact gating condition).
11. **How does `@ConditionalOnMissingBean` let you override defaults?** Boot's default bean backs off if you define your own — your bean wins, so you customize by simply declaring one.
12. **What is `@SpringBootApplication` composed of?** `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.
13. **Walk the full request lifecycle.** Tomcat thread → SecurityFilterChain → DispatcherServlet → HandlerMapping → preHandle → HandlerAdapter (arg binding, `@Valid`) → controller → service proxy (`@Transactional`) → repository → postHandle → JSON serialization → afterCompletion → response.
14. **Where does `@ControllerAdvice`/`@ExceptionHandler` intercept?** Inside DispatcherServlet, after the handler throws, via `HandlerExceptionResolver` → `ExceptionHandlerExceptionResolver` — not a filter or servlet.
15. **Filter vs HandlerInterceptor — difference?** Filter is Servlet-spec, sees raw request/response, runs before/after the DispatcherServlet (security, CORS); HandlerInterceptor is Spring MVC, runs inside the DispatcherServlet around the matched handler.
16. **Where does JWT validation run, and where does `@PreAuthorize`?** JWT in a servlet `Filter` in the SecurityFilterChain (before DispatcherServlet, URL-level); `@PreAuthorize` is AOP inside the service method (fine-grained, per-method).
17. **What does `@SpringBootApplication(exclude = ...)` do?** Disables a specific auto-configuration — common in test slices to avoid a real DB.
18. **How does a third-party starter (e.g., starter-data-redis) work?** An `@AutoConfiguration` class with `@ConditionalOn*` guards listed in `AutoConfiguration.imports` — exactly the custom-starter recipe.
19. **(Bridge to Thursday's REST block) 401 vs 403?** 401 Unauthorized = not authenticated (no/invalid credentials); 403 Forbidden = authenticated but lacking permission.
20. **(Bridge) 409 vs 422?** 409 Conflict = state conflict (duplicate, version mismatch); 422 Unprocessable Entity = syntactically valid but semantically invalid (validation failure).

---

## ✅ Self-check

1. Explain monotonic stack in one sentence.
2. Difference between 401 and 403? Between 409 and 422? (Bridges to Thursday's REST block.)
3. Whiteboard the full request lifecycle from Tomcat thread to HTTP response without notes, and mark where `@ControllerAdvice` and `@PreAuthorize` each intercept.
4. What replaced `spring.factories` in Boot 3, and what single flag prints the conditions evaluation report? (`AutoConfiguration.imports`; `debug=true`.)
5. Why is the sentinel 0 essential in Largest Rectangle in Histogram?

---

*Nav: ← [02-tue-jul-07.md](02-tue-jul-07.md) · [Week 2](README.md) · [04-thu-jul-09.md](04-thu-jul-09.md) →*
