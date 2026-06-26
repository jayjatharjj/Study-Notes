# Week 2 · Day 2 — Tue Jul 07 — Recursion / Backtracking + @Transactional Internals

> Backtracking is the pattern people fear and interviewers love, because it exposes whether you actually understand recursion or just memorized two templates. The whole game is *choose → recurse → un-choose*; forget the un-choose and you corrupt sibling branches. On the Spring side, `@Transactional` is the single most-asked "explain the internals" topic — and almost every candidate gets the self-invocation pitfall wrong. Today you'll be able to draw the proxy call stack on paper.

📌 **Study today:** Backtracking — choose/recurse/un-choose (LC 39, 40, 46, 78) · `@Transactional` internals (propagation, isolation, self-invocation, proxy) · more backtracking (LC 79, 51, 90) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Backtracking Core (~2.5 hr)

### Theory deep-dive

Backtracking is **DFS over a decision tree**. At each node you make a *choice*, recurse into the subtree that choice opens, then **undo the choice** so the next sibling branch starts from a clean state. That undo — the *un-choose* — is the defining act; without it, naive recursion leaks state across branches.

```
backtrack(path, choices):
    if base case:
        results.add(new copy of path)   // COPY — path is mutated after this
        return
    for each choice in choices:
        path.add(choice)                // CHOOSE
        backtrack(path, remaining)      // RECURSE
        path.removeLast()               // UN-CHOOSE (restore)
```

**Three things candidates botch:**
1. **Adding the path by reference, not a copy** — `results.add(path)` stores the *same mutable list*; later mutations corrupt every stored result. Always `new ArrayList<>(path)`.
2. **Forgetting the un-choose** — the path grows monotonically and every result is wrong.
3. **Confusing `start` vs `used[]`** — `start` index for order-*insensitive* combinations (forward-only choice), `used[]` boolean for order-*sensitive* permutations (every position is a fresh choice over all elements).

**Complexity — state it out loud:**
- Subsets: **O(2ⁿ × n)** — 2ⁿ subsets, each costs O(n) to copy.
- Permutations: **O(n! × n)** — n! permutations, each length n.
- Combination Sum: **O(2^(target/min) × …)** — bounded by tree depth target/min element.

**Pruning** is the difference between passing and TLE: sort the input, then `break` the loop when `candidates[i] > remaining` (everything after is bigger too).

### Worked example — LC 78 Subsets

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), res);
    return res;
}
private void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> res) {
    res.add(new ArrayList<>(path));          // every node is a valid subset -> add a COPY
    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);                   // choose
        backtrack(nums, i + 1, path, res);   // recurse with i+1 (no reuse)
        path.remove(path.size() - 1);        // un-choose
    }
}
// nums=[1,2,3] -> [],[1],[1,2],[1,2,3],[1,3],[2],[2,3],[3]  (2^3 = 8 leaves)
```
**Complexity:** O(2ⁿ × n) time, O(n) recursion depth. Trace the tree for `[1,2,3]` on paper.

### Practice set

**1. LC 78 — Subsets** (Medium)
- **Approach:** include/exclude each index; every recursion node is itself a subset.
- **Key insight:** pass `i + 1` (not `i`) — each element used at most once; the loop's natural advance produces all subsets.
- **Complexity:** O(2ⁿ × n). **Follow-up:** the bitmask form — iterate `mask` 0..2ⁿ−1, include `nums[j]` when bit `j` is set.

**2. LC 46 — Permutations** (Medium)
- **Approach:** `used[]` boolean array; at each position try every unused element.
```java
private void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> res) {
    if (path.size() == nums.length) { res.add(new ArrayList<>(path)); return; }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;  path.add(nums[i]);
        backtrack(nums, used, path, res);
        path.remove(path.size() - 1);  used[i] = false;   // restore BOTH
    }
}
```
- **Key insight:** restore *both* `used[i]` and `path`. The swap-based variant avoids `used[]` but mutates `nums` — practice both.
- **Complexity:** O(n! × n). **Follow-up:** LC 47 Permutations II — duplicates → sort + `if (i>0 && nums[i]==nums[i-1] && !used[i-1]) continue;`.

**3. LC 39 — Combination Sum** (Medium)
- **Approach:** elements reusable; pass `start = i` (not `i+1`) to allow reuse while preventing duplicate combinations.
```java
private void backtrack(int[] c, int target, int start, List<Integer> path, List<List<Integer>> res) {
    if (target == 0) { res.add(new ArrayList<>(path)); return; }
    for (int i = start; i < c.length; i++) {
        if (c[i] > target) break;            // pruning (c sorted ascending)
        path.add(c[i]);
        backtrack(c, target - c[i], i, path, res);  // i, NOT i+1 -> reuse allowed
        path.remove(path.size() - 1);
    }
}
```
- **Key insight:** sort first, then `break` on `c[i] > target` — major pruning. Recursing with `i` permits reuse; never going below `start` prevents duplicate combos.
- **Complexity:** O(2^(target/min) × depth). **Follow-up:** count combinations only → DP coin-change variant (no backtracking needed).

**4. LC 40 — Combination Sum II** (Medium)
- **Approach:** each candidate used once → recurse with `i + 1`; sort and add the dedup guard.
- **Key insight:** `if (i > start && c[i] == c[i-1]) continue;` skips duplicate *siblings* at the same tree level while still allowing the same value deeper (parent→child).
- **Complexity:** O(2ⁿ × n). **Follow-up:** explain why `i > start` (not `i > 0`) — we only skip duplicates *at this level*, not across the whole array.

---

## Block B — @Transactional Internals (~2 hr)

> Cross-ref: `../../04-exception-handling-and-memory.md` (checked vs runtime exceptions drive rollback behavior), `../../06-concurrency-and-collections.md` (`ThreadLocal` — the mechanism behind per-thread transactions), `../../05-enums-and-annotations.md` (`@Transactional` is an annotation read by reflection at proxy-creation time).

### Proxy mechanics — the foundation

`@Transactional` is **not** magic in the method body — it's an **AOP proxy** wrapping the bean (created at bean-lifecycle step 6, from yesterday). When you call the method, you actually call the proxy, which runs a `TransactionInterceptor` *around* the real method: begin tx → invoke target → commit (or rollback on exception).

- **CGLIB subclass proxy** for classes — generates a runtime subclass that overrides your methods. Requires a **non-final class** and **non-private, non-final methods** (it can't override those).
- **JDK dynamic proxy** for interfaces — implements the interface with a reflective handler.
- **Spring Boot 2.x+ defaults to CGLIB even for interface-based beans** (`spring.aop.proxy-target-class=true`). So `injectedService.getClass()` prints something like `OrderService$$EnhancerBySpringCGLIB$$...`. Say this unprompted — most candidates don't know it changed.

```
caller --> CGLIB proxy --> TransactionInterceptor --> actual OrderService.placeOrder()
                                  |                              |
                          begin tx (getConnection,         business logic + repo calls
                          set ThreadLocal)                       |
                                  +-------- commit / rollback ---+
```

### Propagation — know cold

| Propagation | Behavior | Use case |
|---|---|---|
| `REQUIRED` (default) | Join existing tx, or start one | 95% of methods |
| `REQUIRES_NEW` | Suspend outer, start a fresh independent tx, commit separately | Audit logs, idempotency tables, outbox — must persist even if the outer tx rolls back |
| `NESTED` | Savepoint inside the outer tx; inner rollback rolls back to the savepoint only | Partial rollback within one tx (JDBC savepoints) |
| `SUPPORTS` | Join if one exists, else run non-transactionally | Read methods callable from both contexts |
| `NOT_SUPPORTED` | Suspend any tx, run without one | Long non-tx work |
| `NEVER` | Throw if a tx exists | Assert no-tx context |
| `MANDATORY` | Throw if no tx exists | Must be called within a tx |

### Isolation — the anomalies each prevents

| Isolation | Prevents | Allows |
|---|---|---|
| `READ_UNCOMMITTED` | nothing | dirty reads |
| `READ_COMMITTED` (Postgres default) | dirty reads | non-repeatable, phantom |
| `REPEATABLE_READ` | dirty, non-repeatable | phantom (in standard SQL; Postgres MVCC also blocks phantoms here) |
| `SERIALIZABLE` | all three | — (highest cost) |

- **Dirty read:** read another tx's uncommitted change. **Non-repeatable read:** same row read twice yields different values (another tx updated+committed between). **Phantom read:** same range query returns new rows (another tx inserted+committed).

### The self-invocation pitfall — must be able to demo

```java
@Service
public class OrderService {
    public void placeOrder() {
        this.processPayment();   // 'this' = RAW bean, bypasses the proxy -> NO transaction!
    }
    @Transactional
    public void processPayment() { /* ... */ }
}
```
Because `this` references the *target*, not the proxy, the `TransactionInterceptor` never runs. **Fixes:**
1. Inject self: `@Autowired private OrderService self;` then `self.processPayment();` (self is the proxy).
2. `applicationContext.getBean(OrderService.class).processPayment();`
3. **Move `processPayment` to a separate `@Service`** — cleanest; the call crosses a bean boundary so it hits the proxy.

### Three more pitfalls to mention unprompted

1. **Private methods:** `@Transactional` on a `private` method is silently ignored — CGLIB can't override private methods.
2. **`rollbackFor` default:** rolls back only on `RuntimeException`/`Error`, **not checked exceptions**. For checked exceptions use `@Transactional(rollbackFor = Exception.class)`. Also: a `try/catch` that swallows the exception → no rollback (no exception propagates).
3. **Never call external HTTP/LLM APIs inside a `@Transactional` method** — you hold a DB connection for the whole call. **WebX bug:** a 20-second LLM call inside a tx held a Postgres/HikariCP connection for 20s, exhausting the pool under load. **Fix:** commit the local state first, do the LLM call *outside* the tx boundary.

### `@Transactional(readOnly = true)`

Hibernate skips dirty checking at flush (no snapshot diffing → faster), Spring hints `Connection.setReadOnly(true)` (HikariCP can route to a read replica), and Postgres skips read-only bookkeeping. A real **Smart360** latency win on a 500-entity report query.

### Why a tx can't span an HTTP thread and an @Async thread

`TransactionSynchronizationManager` stores the active connection in a **`ThreadLocal<Map<Object,Object>>`** — the tx is inherently per-thread. An `@Async` method runs on a *different* thread with an *empty* ThreadLocal, so it has no view of the caller's tx. **WebX pattern:** commit the `jobId` row, then dispatch async; the worker opens its own fresh tx and fetches by ID.

### Project tie-ins

- **Smart360:** `@Transactional(propagation = REQUIRES_NEW)` for audit-log writes (persist even if the business tx rolls back); `readOnly = true` on report queries.
- **WebX:** moved the LLM call outside the tx after the connection-pool incident; commits `jobId` before `@Async` dispatch.
- **Deep Fathom:** a batch import uses `NESTED` savepoints so one bad record rolls back without losing the whole batch.

---

## Block C — DSA: Backtracking Variations (~1.5 hr)

**1. LC 79 — Word Search** (Medium)
- **Approach:** DFS from each cell; mark the cell visited (mutate to a sentinel like `'#'`), recurse in 4 directions, then **restore** the original char.
```java
private boolean dfs(char[][] b, String w, int i, int j, int k) {
    if (k == w.length()) return true;
    if (i<0||i>=b.length||j<0||j>=b[0].length||b[i][j]!=w.charAt(k)) return false;
    char tmp = b[i][j]; b[i][j] = '#';                      // choose (mark visited)
    boolean found = dfs(b,w,i+1,j,k+1) || dfs(b,w,i-1,j,k+1)
                 || dfs(b,w,i,j+1,k+1) || dfs(b,w,i,j-1,k+1);
    b[i][j] = tmp;                                          // un-choose (restore)
    return found;
}
```
- **Key insight:** the common bug is forgetting to restore the cell — the un-choose applies to grids too.
- **Complexity:** O(m·n·4^L), L = word length. **Follow-up:** LC 212 (many words) → use a Trie to prune.

**2. LC 51 — N-Queens** (Hard)
- **Approach:** place one queen per row; track occupied **columns**, **diagonals (`r+c`)**, and **anti-diagonals (`r-c`)** in sets for O(1) conflict checks.
- **Key insight:** `r + c` is constant along a `\` diagonal, `r - c` along a `/` diagonal — that's the O(1) conflict test. Offset `r-c` by `n` if using an array.
- **Complexity:** O(n!). **Follow-up:** LC 52 — count solutions only, no board build.

**3. LC 90 — Subsets II** (Medium)
- **Approach:** duplicates in input; sort, then skip duplicate siblings.
- **Key insight:** `if (i > start && nums[i] == nums[i-1]) continue;` — same level-dedup pattern as Combination Sum II.
- **Complexity:** O(2ⁿ × n). **Follow-up:** contrast with LC 78 — the only delta is the sort + skip guard.

---

## 💻 Practice coding questions

1. **LC 78 Subsets** — choose/skip per index; add a copy at every node.
2. **LC 90 Subsets II** — sort + `i > start` skip for duplicates.
3. **LC 46 Permutations** — `used[]` boolean; restore both on return.
4. **LC 47 Permutations II** — sort + `!used[i-1]` duplicate skip.
5. **LC 39 Combination Sum** — reuse via `start = i`; `break` pruning.
6. **LC 40 Combination Sum II** — single use via `i+1`; level-dedup guard.
7. **LC 77 Combinations** — choose k of n; pure `start` template.
8. **LC 79 Word Search** — grid DFS + restore the cell.
9. **LC 51 N-Queens** — column + two diagonal sets.
10. **LC 22 Generate Parentheses** — track open/close counts as the constraint.
11. **LC 131 Palindrome Partitioning** — partition + palindrome check per cut.
12. **LC 17 Letter Combinations of a Phone Number** — map digits → letters, backtrack.
13. **LC 216 Combination Sum III** — k numbers from 1–9 summing to n; double pruning.

> For each: state the dedup/reuse rule (`start` vs `used[]`, `i` vs `i+1`) before coding, and the complexity after.

---

## 🎤 Interview questions

1. **What makes backtracking different from plain recursion?** The *un-choose* (state restore) step after recursing — it lets sibling branches start clean.
2. **Why `new ArrayList<>(path)` when adding a result?** `path` is mutated after the add; storing it by reference would corrupt every stored result.
3. **When do you use a `start` index vs a `used[]` array?** `start` for order-insensitive combinations (forward-only choice prevents duplicate combos); `used[]` for order-sensitive permutations (every position reconsiders all elements).
4. **Why does Combination Sum recurse with `i`, but Combination Sum II with `i+1`?** `i` allows reusing the same element; `i+1` consumes each candidate once.
5. **Why `if (i > start && a[i] == a[i-1]) continue;` and not `i > 0`?** We skip duplicate *siblings at the same tree level* only; the same value may still appear deeper (parent→child), so the guard is relative to `start`, not the array origin.
6. **State the complexity of Subsets, Permutations, Combination Sum.** O(2ⁿ·n), O(n!·n), O(2^(target/min)·depth).
7. **How is `@Transactional` implemented under the hood?** An AOP proxy (CGLIB or JDK) created at bean-lifecycle step 6; a `TransactionInterceptor` runs around the method: begin → invoke → commit/rollback.
8. **CGLIB vs JDK dynamic proxy — when each, and what are CGLIB's constraints?** CGLIB subclasses (default in Boot 2.x+, even for interfaces); needs a non-final class and non-private/non-final methods. JDK proxy implements interfaces. `getClass()` reveals the CGLIB subclass name.
9. **Your `@Transactional` method isn't rolling back — debug it.** Checked vs runtime exception (`rollbackFor`); self-invocation via `this`; private/final method; a try/catch swallowing the exception; wrong `PlatformTransactionManager`. Tool: `logging.level.org.springframework.transaction=TRACE`.
10. **Explain the self-invocation pitfall and three fixes.** `this.method()` calls the raw target, bypassing the proxy → no tx. Fixes: inject self, `getBean(...)`, or move the method to a separate bean (cleanest).
11. **Why can't a `@Transactional` op start inside an `@Async` method whose caller had a tx?** `TransactionSynchronizationManager` is ThreadLocal-based; the async thread has no inherited tx. Commit the `jobId` before dispatch; the worker opens a fresh tx.
12. **Difference between `REQUIRED`, `REQUIRES_NEW`, and `NESTED`?** `REQUIRED` joins or starts one tx; `REQUIRES_NEW` suspends the outer and runs an independent tx (separate commit); `NESTED` uses a savepoint inside the outer tx.
13. **Where would you use `REQUIRES_NEW`?** Audit logs / idempotency rows / outbox that must persist even if the business tx rolls back.
14. **What does `readOnly = true` actually do?** Hibernate skips dirty checking, Spring hints `Connection.setReadOnly(true)` (HikariCP may route to a replica), Postgres skips read-only overhead — a real latency win on read-heavy queries.
15. **Default `rollbackFor` behavior?** Rolls back on `RuntimeException`/`Error` only. Checked exceptions need `rollbackFor = Exception.class`.
16. **What anomaly does each isolation level prevent?** READ_COMMITTED → dirty; REPEATABLE_READ → +non-repeatable; SERIALIZABLE → +phantom.
17. **(Curveball) "Just put `@Transactional` on everything to be safe."** Push back: long txs starve the connection pool (worse with HTTP/LLM mid-tx), unintended `REQUIRED` joins, wasted locks on reads. Annotate at the service boundary; `readOnly = true` for queries.
18. **Why is calling an LLM/HTTP API inside a transaction dangerous?** The DB connection is held for the entire external call — under load this exhausts the pool. Do the external call outside the tx boundary (WebX learned this the hard way).
19. **Can `@Transactional` go on a private method?** No — silently ignored; CGLIB can't override private methods. Same for `final`.
20. **Draw the proxy call stack.** caller → CGLIB proxy → `TransactionInterceptor` (begin tx, set ThreadLocal connection) → actual bean method → commit/rollback.

---

## ✅ Self-check

1. Why does a `start` index prevent duplicate *combinations* but `used[]` is needed for *permutations*? (Combinations are order-insensitive — `start` enforces a forward-only choice; permutations are order-sensitive — every position is a fresh choice.)
2. Draw the proxy call stack on paper: caller → CGLIB proxy → `TransactionInterceptor` → actual bean method.
3. Write the self-invocation pitfall from memory and give all three fixes.
4. State the complexity of Subsets, Permutations, and Combination Sum without looking.
5. Which two propagations would you reach for to (a) persist an audit log regardless of outer rollback, (b) join the caller's tx? (`REQUIRES_NEW`, `REQUIRED`.)

---

*Nav: ← [01-mon-jul-06.md](01-mon-jul-06.md) · [Week 2](README.md) · [03-wed-jul-08.md](03-wed-jul-08.md) →*
