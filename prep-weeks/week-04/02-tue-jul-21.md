# Week 4 · Day 2 — Tue Jul 21 — DP 2D + Resilience4j

> Yesterday's 1D DP becomes a grid. Once you see that a 2D `dp[i][j]` over two sequences is *the same recipe with two indices*, the "scary" problems — Edit Distance, LCS, Maximal Square — all collapse into "match → extend, mismatch → take the best neighbour." On the systems side, Resilience4j is the resilience layer every senior backend interview probes: draw the **circuit-breaker state machine with the exact config on each edge**, distinguish a **bulkhead** (limits concurrency) from a **circuit breaker** (cuts off a dead dependency), and state the **wrap order**. Both halves reward precision over hand-waving.

📌 **Study today:** 2D DP — Unique Paths, Edit Distance, LCS, Maximal Square, palindromes (LC 62, 72, 1143, 221, 5) · Resilience4j (circuit-breaker state machine + bulkhead + Retry/TimeLimiter, wrap order) · knapsack / subset DP (LC 416, 518) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: 2D Dynamic Programming (~2.5 hr)

### Theory deep-dive

A 2D DP is the 1D recipe with a second index. The two common shapes:

- **Grid DP** — `dp[i][j]` = the answer for the subgrid ending at cell `(i,j)`; transitions come from the cells above/left/diagonal (Unique Paths, Maximal Square, Minimum Path Sum).
- **Two-sequence DP** — `dp[i][j]` = the answer for `a[0..i)` against `b[0..j)`; transitions compare `a[i-1]` to `b[j-1]` (LCS, Edit Distance, Distinct Subsequences). The classic indexing convention: `dp` has dimensions `(m+1) × (n+1)` so row/col 0 is the "empty prefix" base case, and `a[i-1]`/`b[j-1]` are the characters being compared at `dp[i][j]`.

**Space optimisation:** if `dp[i][j]` only reads the previous row (`dp[i-1][*]`) and the current row to its left, you can roll the table down to **two 1D arrays**, then often **one** array with a single `prev` scalar holding the `dp[i-1][j-1]` diagonal. Always mention this — interviewers ask "can you do it in O(n) space?" after the O(mn) version.

### Worked example — LC 1143 Longest Common Subsequence (the parent problem)

State: `dp[i][j]` = length of the LCS of `text1[0..i)` and `text2[0..j)`. If the current characters match, they extend the LCS of the smaller prefixes (`dp[i-1][j-1] + 1`); if not, drop one character from either string and take the better (`max(dp[i-1][j], dp[i][j-1])`).

```java
public int longestCommonSubsequence(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];          // row/col 0 = empty prefix = 0
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            dp[i][j] = (a.charAt(i - 1) == b.charAt(j - 1))
                ? dp[i - 1][j - 1] + 1
                : Math.max(dp[i - 1][j], dp[i][j - 1]);
    return dp[m][n];
}
//  a="abcde", b="ace" -> LCS "ace" = 3
```

**Reconstruct the actual subsequence** (sometimes asked): walk back from `dp[m][n]` — if characters match, prepend it and go diagonally; else step toward the larger of up/left.

### Worked example — LC 72 Edit Distance (LCS in disguise)

State: `dp[i][j]` = min operations to turn `word1[0..i)` into `word2[0..j)`. Three edits map to three neighbours:

- **insert** a char into word1 → `dp[i][j-1] + 1`
- **delete** a char from word1 → `dp[i-1][j] + 1`
- **replace** (or free match) → `dp[i-1][j-1] + (chars differ ? 1 : 0)`

Base: `dp[i][0] = i` (delete all of word1), `dp[0][j] = j` (insert all of word2).

```java
public int minDistance(String w1, String w2) {
    int m = w1.length(), n = w2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;   // delete-all base
    for (int j = 0; j <= n; j++) dp[0][j] = j;   // insert-all base
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (w1.charAt(i - 1) == w2.charAt(j - 1))
                dp[i][j] = dp[i - 1][j - 1];                       // free match
            else
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],         // replace
                              Math.min(dp[i - 1][j], dp[i][j - 1])); // delete, insert
        }
    return dp[m][n];
}
//  "horse" -> "ros" = 3 (replace h->r, delete r, delete e)
```

**The expert move:** say out loud "Edit Distance is LCS with three operations instead of one — the diagonal is the match/replace axis, the verticals are insert/delete." Seeing that connection is the signal.

### Complexity summary

| Problem | State | Time | Space (optimised) |
|---|---|---|---|
| Unique Paths (62) | paths to `(i,j)` | O(mn) | O(n) — one row |
| Edit Distance (72) | min ops `w1[0..i)→w2[0..j)` | O(mn) | O(n) — rolling row |
| LCS (1143) | LCS of two prefixes | O(mn) | O(n) |
| Maximal Square (221) | square side at `(i,j)` | O(mn) | O(n) |
| Longest Palindromic Substring (5) | expand-around-centre | O(n²) | O(1) |
| Partition Equal Subset (416) | reachable subset sums | O(n·sum) | O(sum) |
| Coin Change II (518) | ways to make amount | O(n·amount) | O(amount) |

### Practice set

**1. LC 62 — Unique Paths** (Medium)
- **Approach:** `dp[i][j] = dp[i-1][j] + dp[i][j-1]`; first row/col all 1 (only one way along an edge). Reduce to a single 1D row updated in place.
```java
public int uniquePaths(int m, int n) {
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[j] += dp[j - 1];                  // dp[j]=above (old), dp[j-1]=left (new)
    return dp[n - 1];
}
```
- **Key insight:** also pure combinatorics — `C(m+n-2, m-1)` (choose which of the `m+n-2` moves are "down"). Know both; the combinatorial form is O(min(m,n)).
- **Complexity:** O(mn) / O(n).

**2. LC 72 — Edit Distance** (Hard)
- **Approach:** the three-neighbour recurrence above; free move on a match.
- **Key insight:** it's LCS with three ops. Get the two base rows right — `dp[i][0]=i`, `dp[0][j]=j` — they encode "convert to/from the empty string."
- **Complexity:** O(mn) / O(n) with a rolling row. **Follow-up:** reconstruct the edit script by backtracking.

**3. LC 1143 — Longest Common Subsequence** (Medium)
- **Approach:** match → `dp[i-1][j-1]+1`, mismatch → `max(dp[i-1][j], dp[i][j-1])`.
- **Key insight:** subsequence (order preserved, gaps allowed) ≠ substring (contiguous). This DP is the parent of Edit Distance and Distinct Subsequences.
- **Complexity:** O(mn) / O(n).

**4. LC 221 — Maximal Square** (Medium)
- **Approach:** `dp[i][j]` = side of the largest all-`1` square whose **bottom-right corner** is `(i,j)`. When `matrix[i][j]=='1'`, `dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1`. Track the global max side; answer = side².
```java
public int maximalSquare(char[][] m) {
    int rows = m.length, cols = m[0].length, best = 0;
    int[][] dp = new int[rows + 1][cols + 1];     // padded with a 0 border
    for (int i = 1; i <= rows; i++)
        for (int j = 1; j <= cols; j++)
            if (m[i - 1][j - 1] == '1') {
                dp[i][j] = 1 + Math.min(dp[i - 1][j],
                              Math.min(dp[i][j - 1], dp[i - 1][j - 1]));
                best = Math.max(best, dp[i][j]);
            }
    return best * best;
}
```
- **Key insight:** the side is **bottlenecked by its three neighbours** — a square of side `k+1` at `(i,j)` requires squares of side ≥ `k` to its top, left, and top-left. Prove it on a 2×2: any missing `1` caps the min at 0 or 1.
- **Complexity:** O(mn) / O(n) with a rolling row + diagonal scalar.

**5. LC 5 — Longest Palindromic Substring** (Medium)
- **Approach:** **expand around centre** (interview-preferred). Each of the `2n−1` centres (n single chars + n−1 gaps) expands outward while characters mirror. O(n²) time, O(1) space.
```java
public String longestPalindrome(String s) {
    int start = 0, maxLen = 0;
    for (int c = 0; c < s.length(); c++) {
        int len = Math.max(expand(s, c, c), expand(s, c, c + 1)); // odd, even
        if (len > maxLen) { maxLen = len; start = c - (len - 1) / 2; }
    }
    return s.substring(start, start + maxLen);
}
private int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return r - l - 1;                              // length of the palindrome
}
```
- **Key insight:** two centre types — odd-length (`c,c`) and even-length (`c,c+1`). The DP table form (`dp[i][j]` = "is `s[i..j]` a palindrome?") is also O(n²) but uses O(n²) space; expand-around-centre is strictly leaner. Mention **Manacher's O(n)** exists (transform with separators, exploit mirror symmetry) — saying so signals depth; **don't** implement it under pressure.
- **Complexity:** O(n²) / O(1).

---

## Block B — Resilience4j: Circuit Breaker + Bulkhead + Retry/TimeLimiter (~2 hr)

> Cross-ref: `../../06-concurrency-and-collections.md` — `ThreadPoolBulkhead` runs calls on a bounded `ExecutorService` and returns a `CompletableFuture`; the semaphore bulkhead is a `Semaphore` cap on concurrent calls. The whole topic is applied concurrency control around an unreliable dependency.

### The mental model

Resilience4j is a lightweight fault-tolerance library (the modern replacement for the now-defunct Hystrix). It provides **five composable decorators** you wrap around a remote call: **CircuitBreaker, Retry, RateLimiter, TimeLimiter, Bulkhead**. Each protects against a different failure mode. The skill is knowing *which* protects against *what*, and *in what order* to stack them.

### Circuit breaker — the state machine (draw this from memory)

```
            failureRate >= failureRateThreshold
            OR slowCallRate >= slowCallRateThreshold
            (only after minimumNumberOfCalls in the window)
   CLOSED ───────────────────────────────────────────────► OPEN
     ▲                                                        │
     │ permittedNumberOfCallsInHalfOpenState                  │ after
     │ calls ALL succeed                                      │ waitDurationInOpenState
     │                                                        ▼
   CLOSED ◄───────────────  HALF_OPEN  ───── any failure ──► OPEN
```

- **CLOSED** — calls pass through; outcomes are recorded in the sliding window. Trips to **OPEN** when the failure rate *or* slow-call rate crosses its threshold — but **only after `minimumNumberOfCalls`** have been recorded (this prevents a single early failure on startup from opening the breaker).
- **OPEN** — calls are **short-circuited** instantly (fail fast → fallback), giving the dependency time to recover. After `waitDurationInOpenState`, it moves to **HALF_OPEN**.
- **HALF_OPEN** — lets `permittedNumberOfCallsInHalfOpenState` probe calls through. All succeed → back to **CLOSED**; any one fails → straight back to **OPEN**.
- `slidingWindowType`: **COUNT_BASED** (last N calls) or **TIME_BASED** (calls in the last N seconds).

```yaml
# Spring Boot 3 — application.yml
resilience4j:
  circuitbreaker:
    instances:
      llmProvider:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 20
        minimumNumberOfCalls: 10           # don't judge until 10 calls seen
        failureRateThreshold: 50           # open at >=50% failures
        slowCallRateThreshold: 80          # open at >=80% slow calls
        slowCallDurationThreshold: 2s      # a call >2s counts as "slow"
        waitDurationInOpenState: 30s       # stay OPEN 30s before probing
        permittedNumberOfCallsInHalfOpenState: 3
```

```java
@Service
public class LlmClient {
    @CircuitBreaker(name = "llmProvider", fallbackMethod = "fallback")
    public String complete(String prompt) {
        return webClient.post()...block();         // remote call
    }
    // Fallback MUST share the signature + a trailing Throwable parameter.
    public String fallback(String prompt, Throwable t) {
        return "Service temporarily unavailable. Please retry shortly.";
    }
}
```

**Multi-instance gotcha:** Resilience4j state is **in-process** — 20 pods each keep their own window, so they can be in different states. Under round-robin load this usually converges fine (failures spread across pods, all trip eventually). Truly *global* state needs a Redis-backed adapter, which adds latency and a new failure point — know the trade-off, don't reach for it by default.

### Bulkhead vs circuit breaker (never bluff this)

A **circuit breaker** *cuts off* calls to a dependency that has already failed (trips on a failure threshold, short-circuits). A **bulkhead** *limits* how much of your resources a single dependency can consume, so a **slow-but-not-yet-failing** dependency can't exhaust your threads and starve everything else. Named after a ship's watertight compartments — a breach floods one compartment, not the whole hull. Two flavours:

- **`Bulkhead` (semaphore)** — a counting semaphore caps *concurrent* calls; the call runs on the **caller's own thread**. Cheap, low overhead, but no timeout isolation. Good default for fast synchronous dependencies.
- **`ThreadPoolBulkhead` (thread-pool)** — the call runs in a **dedicated bounded pool** with its own queue; the caller's thread is freed immediately (you get a `CompletableFuture`). Costlier, but fully isolates a slow dependency — e.g. run slow LLM-provider calls in their own pool so they can never starve the threads serving fast data queries.

They're **complementary**: the bulkhead *contains the blast radius* while a dependency degrades; the circuit breaker *cuts off* one that's already dead.

### Retry and TimeLimiter, and the wrap order

- **Retry** — `maxAttempts`, `waitDuration`, exponential backoff, `retryExceptions` / `ignoreExceptions`. **Danger:** retrying non-idempotent operations (a POST that charges a card) can double-execute — guard with idempotency keys, or only retry idempotent calls.
- **TimeLimiter** — wraps a `CompletableFuture` and cancels it after a timeout; a timeout is recorded as a **failure** that feeds the circuit breaker. Without it, a hung call ties up a thread indefinitely.

**Wrap order matters: `TimeLimiter( Retry( CircuitBreaker( call ) ) )`.** Read it outside-in: the TimeLimiter bounds the *total* time including retries; Retry re-invokes the breaker-guarded call; the CircuitBreaker sits closest to the actual call so each attempt is recorded in its window. (Bulkhead goes innermost-or-outermost depending on whether you want to bound concurrency of attempts or of whole operations — commonly innermost so the bulkhead bounds each actual call.)

### Project tie-in

- **Smart360 — LLM provider:** circuit breaker around the model API (`failureRateThreshold=50`, `slowCallDurationThreshold=2s`) with a graceful "try again shortly" fallback; a `ThreadPoolBulkhead` isolated the slow LLM calls so a provider slowdown couldn't starve the threads serving fast dashboard queries.
- **Half-open thundering herd (war story):** two pods probed a half-recovered downstream simultaneously, both failed, both re-opened — flapping. Fix: `permittedNumberOfCallsInHalfOpenState=1` plus a health-check poller so only a controlled probe re-tests recovery.
- **WebX — idempotency:** retries were enabled only on idempotent reads; the payment POST used an idempotency key so a retried request was deduplicated server-side rather than charging twice.

---

## Block C — DSA: Knapsack / Subset DP (~1.5 hr)

### Theory — the loop-direction rule that trips everyone

The **0/1 knapsack** is the template: `dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])` if `wt[i] <= w` — each item used **at most once**. Space-optimised to a 1D `dp[w]`, the inner weight loop runs **right-to-left**:

- **0/1 (each item once) → iterate weights right-to-left.** Going high→low, `dp[w-wt]` still holds the *previous* item's value (the `i-1` row) when you read it, so the item can't be reused within the same pass.
- **Unbounded (item reusable) → iterate weights left-to-right.** Going low→high, `dp[w-wt]` has *already* been updated for the current item, so the item gets reused — which is exactly unbounded behaviour.

Memorise the one-liner: **0/1 = right-to-left; unbounded = left-to-right.** It's the single most-asked DP "gotcha."

### Practice set

**1. LC 416 — Partition Equal Subset Sum** (Medium, 0/1 knapsack)
- **Approach:** can a subset sum to `total/2`? If `total` is odd, return false immediately. Boolean knapsack: `dp[j] |= dp[j - num]`, inner loop **right-to-left**.
```java
public boolean canPartition(int[] nums) {
    int total = Arrays.stream(nums).sum();
    if ((total & 1) == 1) return false;            // odd total can't split evenly
    int target = total / 2;
    boolean[] dp = new boolean[target + 1];
    dp[0] = true;                                  // sum 0 always reachable
    for (int num : nums)
        for (int j = target; j >= num; j--)        // RIGHT-TO-LEFT (0/1)
            dp[j] |= dp[j - num];
    return dp[target];
}
```
- **Key insight:** right-to-left is what makes each number usable once. Left-to-right here would let a number be counted multiple times → wrong answer.
- **Complexity:** O(n × target) / O(target).

**2. LC 518 — Coin Change II** (Medium, unbounded knapsack)
- **Approach:** *count the number of ways* to make `amount` with unlimited coins. `dp[a] += dp[a - coin]`, inner loop **left-to-right** (coin reusable). **Outer loop = coins, inner = amounts** → counts **combinations** (each unordered coin set once).
```java
public int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;                                     // one way to make 0: empty set
    for (int coin : coins)                         // outer = coins -> combinations
        for (int a = coin; a <= amount; a++)       // LEFT-TO-RIGHT (unbounded)
            dp[a] += dp[a - coin];
    return dp[amount];
}
```
- **Key insight:** **loop order changes the meaning.** Outer=coins → combinations ({1,2} and {2,1} counted once). Swapping to outer=amounts → counts *permutations* (ordered sequences). Coin Change II wants combinations, so coins go outside.
- **Complexity:** O(n × amount) / O(amount).

**Contrast the two on the board:** LC 416 uses `|=` (reachability), LC 518 uses `+=` (counting); LC 416 is right-to-left (0/1), LC 518 is left-to-right (unbounded). Same skeleton, four switched details.

---

## 💻 Practice coding questions

1. **LC 62 Unique Paths** — grid sum; 1D row reduction; know `C(m+n-2, m-1)`.
2. **LC 63 Unique Paths II** — add obstacles → zero out blocked cells.
3. **LC 64 Minimum Path Sum** — grid DP with `min` instead of sum.
4. **LC 72 Edit Distance** — three-neighbour recurrence; get the base rows right.
5. **LC 1143 LCS** — match→diagonal+1, mismatch→max neighbour; reconstruct on request.
6. **LC 221 Maximal Square** — `min(3 neighbours)+1`; answer = side².
7. **LC 5 Longest Palindromic Substring** — expand-around-centre, odd + even.
8. **LC 516 Longest Palindromic Subsequence** — LCS of `s` with `reverse(s)`.
9. **LC 416 Partition Equal Subset Sum** — 0/1 boolean knapsack, right-to-left.
10. **LC 518 Coin Change II** — unbounded counting, outer=coins for combinations.
11. **LC 494 Target Sum** — transform to subset-sum knapsack.
12. **LC 583 Delete Operation for Two Strings** — `m+n − 2·LCS`.
13. **LC 1092 Shortest Common Supersequence** — LCS + reconstruct the merge.

> Drill the loop-direction rule until reflexive: state out loud "0/1 → right-to-left, unbounded → left-to-right" before writing the inner loop. Re-time LC 416 — target ≤ 12 min.

---

## 🎤 Interview questions

1. **How do you set up the indices for a two-sequence DP?** Use `(m+1)×(n+1)` so row/col 0 is the empty-prefix base case, and compare `a[i-1]` to `b[j-1]` at `dp[i][j]`. This makes the base cases natural and avoids off-by-ones.
2. **Edit Distance — explain the three transitions.** Insert → `dp[i][j-1]+1`; delete → `dp[i-1][j]+1`; replace → `dp[i-1][j-1]+1` (or `dp[i-1][j-1]` free on a match). Base rows: convert to/from the empty string.
3. **How is Edit Distance related to LCS?** It's LCS with three operations: the diagonal is the match/replace axis, the verticals/horizontals are insert/delete. Recognising the connection is the expert signal.
4. **Optimise a 2D string DP to O(n) space.** `dp[i][j]` only reads the previous row and the current row to the left → keep two 1D arrays, or one array plus a `prev` scalar holding the `dp[i-1][j-1]` diagonal.
5. **Maximal Square — why `min` of three neighbours plus one?** A square of side `k+1` ending at `(i,j)` needs side-`k` squares above, left, and top-left; the smallest of those three caps the new side. A single missing `1` limits the min, so the side can't grow past it.
6. **Longest Palindromic Substring — why expand-around-centre over the DP table?** Same O(n²) time but O(1) space vs O(n²). There are `2n−1` centres (odd and even). Manacher's gets O(n) via mirror symmetry — mention it, don't code it under pressure.
7. **0/1 knapsack 1D — why iterate weights right-to-left?** Right-to-left keeps `dp[w-wt]` as the previous item's value (the `i-1` row) when read, so each item is used at most once. Left-to-right would read an already-updated value → the item gets reused (unbounded).
8. **Coin Change II — why does loop order change the answer?** Outer=coins, inner=amounts counts each unordered coin set once (combinations); swapping to outer=amounts counts ordered sequences (permutations). The problem wants combinations, so coins go outside.
9. **Draw the Resilience4j circuit-breaker state machine.** CLOSED → OPEN when failure-rate or slow-call-rate crosses threshold (after `minimumNumberOfCalls`); OPEN → HALF_OPEN after `waitDurationInOpenState`; HALF_OPEN → CLOSED if all probe calls succeed, → OPEN on any failure.
10. **Bulkhead vs circuit breaker — the difference in one sentence each.** Circuit breaker *cuts off* a dependency that has already failed; bulkhead *limits* the resources one dependency can consume so a slow one can't starve the rest. Complementary.
11. **Semaphore bulkhead vs thread-pool bulkhead?** Semaphore caps concurrent calls on the caller's own thread (cheap, no timeout isolation); thread-pool runs calls in a dedicated bounded pool and frees the caller's thread (isolates slow dependencies fully, returns a `CompletableFuture`, costs more).
12. **What's the wrap order for the resilience decorators and why?** `TimeLimiter(Retry(CircuitBreaker(call)))`: TimeLimiter bounds total time including retries, Retry re-invokes the breaker-guarded call, CircuitBreaker sits closest so each attempt is recorded in its window.
13. **A timeout fires — does it count as a circuit-breaker failure?** Yes — the TimeLimiter cancels the future and the timeout is recorded as a failure, feeding the breaker's failure-rate window. That's why TimeLimiter and CircuitBreaker compose.
14. **20 pods, each with its own circuit state — is that a problem?** Usually not: under round-robin, failures spread and all pods converge to OPEN. Global state needs a Redis-backed adapter, adding latency and a failure point — only worth it if you truly need a single shared decision.
15. **You enabled Retry — what could go wrong?** Retrying non-idempotent operations double-executes (a retried payment POST charges twice). Restrict retries to idempotent calls, or use idempotency keys so the server deduplicates.
16. **Half-open thundering herd — what is it and how do you fix it?** Multiple pods probe a half-recovered downstream at once, all fail, all re-open (flapping). Fix: `permittedNumberOfCallsInHalfOpenState=1` + a controlled health-check poller.
17. **COUNT_BASED vs TIME_BASED sliding window — when each?** COUNT_BASED (last N calls) suits steady traffic; TIME_BASED (last N seconds) suits bursty/variable traffic where call counts swing — it reflects the recent *rate* rather than a fixed sample size.
18. **Why did Resilience4j replace Hystrix?** Hystrix is in maintenance mode (no active development); Resilience4j is lightweight, Java-8-functional, modular (only pull the decorators you need), and integrates cleanly with Spring Boot 3 and Micrometer metrics.

---

## ✅ Self-check

1. Code 0/1 knapsack bottom-up in 12 min, convert it to a 1D array, and explain why the inner loop runs right-to-left for 0/1 but left-to-right for unbounded.
2. Draw the Resilience4j circuit-breaker state machine from memory with the exact config parameter on each edge.
3. State the difference between a bulkhead and a circuit breaker in one sentence each, then say which bulkhead flavour you'd use for a slow LLM call and why.
4. Derive the Edit Distance recurrence from its state sentence, and say how it relates to LCS.
5. Why does Coin Change II put coins on the outer loop? (Combinations, not permutations.)

---

<div align="center">

← [01-mon-jul-20.md](01-mon-jul-20.md) · [Week 4](README.md) · [03-wed-jul-22.md](03-wed-jul-22.md) →

</div>
