# Week 4 · Day 1 — Mon Jul 20 — DP 1D + Rate-Limiting Algorithms

> The day dynamic programming stops being memorisation and becomes a recipe. Every 1D DP is the same three sentences: *what is the state, how does it transition, what's the base case* — derive that and the code writes itself. On the systems side, rate limiting is the most reachable "protect the platform" topic in a senior interview: the answer that scores names **which algorithm**, **why** (burst vs smooth), and **the exact Redis structure** behind it — token bucket on a HASH with a Lua script, sliding window log on a ZSET. Both halves share a theme: pick the cheapest structure that satisfies the constraint, no more.

📌 **Study today:** 1D DP — state + transition framework (LC 70, 198, 213, 139, 300, 322) · rate-limiting algorithms (token bucket / leaky bucket / fixed + sliding window, with Redis structures) · DP re-derivation drills · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: 1D DP Foundations (~2.5 hr)

### Theory deep-dive — the universal framework

Dynamic programming is **recursion with the repeated work cached**. You apply it when a problem has (1) *optimal substructure* — the answer is built from answers to smaller subproblems — and (2) *overlapping subproblems* — the same subproblem is solved many times by naive recursion. If subproblems don't overlap, it's divide-and-conquer, not DP.

The recipe, every time:

1. **State** — write it as an English sentence: "`dp[i]` = the answer for the first `i` elements / for target `i`." If you can't say the sentence, you can't write the code.
2. **Transition** — how does `dp[i]` depend on `dp[i-1]`, `dp[i-2]`, …? This is the hard step. *Draw a small example and watch what choice you make at index `i`.*
3. **Base case** — the smallest subproblem you can answer directly (`dp[0]`, `dp[1]`, empty string).
4. **Direction** — **top-down** (memoization: recursion + a cache array/`HashMap`) or **bottom-up** (tabulation: fill an array with loops).
5. **Space optimisation** — if `dp[i]` only reads `dp[i-1]` and `dp[i-2]`, drop the whole array and keep two variables → O(1) space.

**Top-down vs bottom-up:** memoization is easier to write (mirror the recurrence, add a cache) and only computes subproblems you actually need, but it carries recursion-stack overhead and can stack-overflow on deep inputs. Tabulation has no stack risk, is cache-friendly (sequential array access), and is easiest to space-optimise — **prefer it for production code**, reach for memoization when the recurrence is awkward to flatten.

### Worked example — LC 70 Climbing Stairs (all three forms)

State: `dp[i]` = number of distinct ways to reach step `i`. To land on step `i` you came from `i-1` (one step) or `i-2` (two steps), so `dp[i] = dp[i-1] + dp[i-2]` — it's Fibonacci. Base: `dp[0] = 1` (one way to stand at the bottom — do nothing), `dp[1] = 1`.

```java
// 1) Top-down memoization. O(n) time, O(n) space (cache + call stack).
public int climbStairs(int n) {
    return climb(n, new int[n + 1]);
}
private int climb(int n, int[] memo) {
    if (n <= 1) return 1;
    if (memo[n] != 0) return memo[n];          // cache hit — no recompute
    return memo[n] = climb(n - 1, memo) + climb(n - 2, memo);
}

// 2) Bottom-up tabulation. O(n) time, O(n) space.
public int climbStairsTab(int n) {
    if (n <= 1) return 1;
    int[] dp = new int[n + 1];
    dp[0] = dp[1] = 1;
    for (int i = 2; i <= n; i++) dp[i] = dp[i - 1] + dp[i - 2];
    return dp[n];
}

// 3) Space-optimised. O(n) time, O(1) space — the version to write in an interview.
public int climbStairsOpt(int n) {
    int prev2 = 1, prev1 = 1;                  // dp[i-2], dp[i-1]
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
//  n=5 -> dp: [1,1,2,3,5,8] -> 8 ways
```

**The move:** write the O(1) version, then say "the array form makes the recurrence obvious; I collapsed it because `dp[i]` only reads the last two states."

### Worked example — LC 198 House Robber (justify, don't memorise)

State: `dp[i]` = max money robbable from houses `0..i`. At house `i` you face a *binary choice*: **rob it** (then you skip `i-1`, so you add `nums[i]` to `dp[i-2]`) or **skip it** (carry `dp[i-1]`). You take the better:

```java
public int rob(int[] nums) {
    int prev2 = 0, prev1 = 0;                  // dp[i-2], dp[i-1]
    for (int num : nums) {
        int curr = Math.max(prev1, prev2 + num);   // skip vs rob
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// [2,7,9,3,1] -> rob 2,9,1 = 12 (not 7+3=10)
```

The "can't rob adjacent houses" rule *is* the `prev2 + num` term — robbing `i` forces you back to the best solution that ended at or before `i-2`. You derived the constraint as a recurrence, not recited it.

### Complexity summary

| Problem | State | Transition | Time | Space (optimised) |
|---|---|---|---|---|
| Climbing Stairs (70) | ways to reach step `i` | `dp[i-1]+dp[i-2]` | O(n) | O(1) |
| House Robber (198) | max loot through `i` | `max(dp[i-1], dp[i-2]+nums[i])` | O(n) | O(1) |
| House Robber II (213) | circle | two linear passes | O(n) | O(1) |
| Word Break (139) | first `i` segmentable? | OR over splits `j<i` | O(n²·L) | O(n) |
| LIS (300) | longest ending at `i` | `max(dp[j]+1)` | O(n²) / O(n log n) | O(n) |
| Coin Change (322) | min coins for amount `i` | `min(dp[i-coin]+1)` | O(amount·coins) | O(amount) |

### Practice set

**1. LC 70 — Climbing Stairs** (Easy)
- **Approach:** Fibonacci; code all three forms above, land on the two-variable O(1) version.
- **Key insight:** `dp[0]=1` not 0 — "one way to do nothing." Off-by-one on the base case is the only trap.
- **Complexity:** O(n) / O(1). **Target: < 6 min.**

**2. LC 198 — House Robber** (Medium)
- **Approach:** rob-or-skip recurrence, two rolling variables.
- **Key insight:** `prev2 + num` encodes "skip the neighbour." Initialise both to 0 so the first house just becomes `max(0, nums[0])`.
- **Complexity:** O(n) / O(1). **Target: AC in 8 min, no notes.**

**3. LC 213 — House Robber II** (Medium)
- **Approach:** houses in a circle → house 0 and house n-1 are now adjacent and can never *both* be robbed. Run House Robber I on `nums[0..n-2]` and on `nums[1..n-1]`, return the max. Special-case `n==1`.
- **Key insight:** *why splitting works* — every valid selection either excludes the first house or excludes the last; the two passes cover both cases, and any optimal selection falls in at least one.
- **Complexity:** O(n) / O(1).

**4. LC 139 — Word Break** (Medium)
- **Approach:** `dp[i]` = can `s[0..i)` be segmented? For each `i`, scan `j < i`: if `dp[j]` is true and `s[j..i)` is in the dictionary, set `dp[i]=true` and break.
```java
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    int n = s.length();
    boolean[] dp = new boolean[n + 1];
    dp[0] = true;                              // empty prefix is segmentable
    for (int i = 1; i <= n; i++)
        for (int j = 0; j < i; j++)
            if (dp[j] && dict.contains(s.substring(j, i))) { dp[i] = true; break; }
    return dp[n];
}
```
- **Key insight:** this is identical to DFS-with-memoization — `dp[j]` is exactly "can I reach index `j`?" Putting the dict in a `HashSet` makes each lookup O(L). Complexity O(n²·L) with L the average word length.
- **Complexity:** O(n²·L) / O(n). **Follow-up:** LC 140 Word Break II returns all segmentations — backtracking + memo of `index → list of suffixes`.

**5. LC 300 — Longest Increasing Subsequence** (Medium)
- **Approach:** O(n²) first — `dp[i]` = length of the longest increasing subsequence *ending at* `i`; for each `j < i` with `nums[j] < nums[i]`, `dp[i] = max(dp[i], dp[j]+1)`. Then the O(n log n) patience-sort: keep a `tails` array where `tails[k]` is the smallest possible tail of an increasing subsequence of length `k+1`; binary-search the insertion point of each number.
```java
// O(n log n) — interviewers push for this after the O(n^2) version.
public int lengthOfLIS(int[] nums) {
    List<Integer> tails = new ArrayList<>();
    for (int x : nums) {
        int lo = 0, hi = tails.size();
        while (lo < hi) {                       // first index with tails[mid] >= x
            int mid = (lo + hi) >>> 1;
            if (tails.get(mid) < x) lo = mid + 1; else hi = mid;
        }
        if (lo == tails.size()) tails.add(x);   // extend
        else tails.set(lo, x);                  // replace to keep tails minimal
    }
    return tails.size();
}
```
- **Key insight:** `tails` is **not** the LIS itself — its *length* is the answer. Replacing a larger tail with `x` keeps future extension options maximal. (`Arrays.binarySearch` on an `int[]` also works; the manual lower-bound is clearer for the strict-increasing case.)
- **Complexity:** O(n log n) / O(n). **Real-world:** LIS ≈ the longest chain of mutually-compatible microservice/API versions you can upgrade through without a breaking jump.

**6. LC 322 — Coin Change** (Medium)
- **Approach:** unbounded knapsack. `dp[a]` = min coins to make amount `a`; for each coin, `dp[a] = min(dp[a], dp[a-coin]+1)`. Init `dp[0]=0`, rest to a sentinel (`amount+1`, larger than any real answer).
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);                 // "infinity" sentinel
    dp[0] = 0;
    for (int a = 1; a <= amount; a++)
        for (int c : coins)
            if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
}
```
- **Key insight:** **greedy fails.** coins=[1,3,4], amount=6 → greedy picks 4+1+1 = 3 coins; DP finds 3+3 = 2. DP explores every sub-amount, so it can't be fooled by a locally-large coin.
- **Complexity:** O(amount × coins) / O(amount). **Real-world:** resembles Smart360's LLM-provider cost routing — DP would be *optimal* but too slow for the real-time path, which is why production used heuristic scoring. Knowing when DP is overkill is as valuable as knowing when to use it.

---

## Block B — Rate-Limiting Algorithms (~2 hr)

> Cross-ref: `../../07-collections.md` — the in-memory analogues are a `Map` (token bucket = counter + timestamp), a `Queue` (leaky bucket), and a sorted structure (sliding window log = `TreeMap`/`ZSET`). Rate limiting is just choosing the cheapest structure that enforces the constraint you actually have.

### The mental model

A rate limiter answers one question per request: **"has this client exceeded its allowance — allow or reject (HTTP 429)?"** The algorithms differ on two axes: do they **allow bursts**, and how much **memory/accuracy** they cost. Pick by the traffic shape, not by what sounds fanciest.

| Algorithm | How it works | Burst? | Memory | Redis structure |
|---|---|---|---|---|
| **Token bucket** | tokens refill at a fixed rate, each request spends one | Yes (≤ bucket size) | O(1) | HASH `{tokens, last_refill}` |
| **Leaky bucket** | requests queue, drain at a fixed output rate | No (smoothed) | O(queue) | LIST as a queue |
| **Fixed window counter** | a counter resets every window | Yes (boundary burst) | O(1) | STRING with TTL |
| **Sliding window log** | store the timestamp of *every* request | No | O(requests in window) | ZSET (score = timestamp) |
| **Sliding window counter** | interpolate between two adjacent fixed windows | ~No | O(1) | two STRINGs |

### Token bucket — the default for bursty traffic

A bucket holds up to `burstCapacity` tokens; tokens are added at `replenishRate` per second (capped at the bucket size); each request consumes one token; an empty bucket → 429. The elegance: a client that's been idle has a full bucket and can **burst**, but sustained throughput is still capped at `replenishRate`. This is **Spring Cloud Gateway's `RequestRateLimiter`**.

```yaml
# Spring Cloud Gateway (Spring Boot 3) — token bucket via Redis
spring:
  cloud:
    gateway:
      routes:
        - id: analytics
          uri: lb://analytics-service
          predicates:
            - Path=/api/analytics/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 5      # 5 req/s sustained
                redis-rate-limiter.burstCapacity: 20     # allow a 20-req burst
                redis-rate-limiter.requestedTokens: 1
                key-resolver: "#{@userKeyResolver}"      # bean: rate-limit per user
```

```java
// The key resolver decides WHO is being limited (per user, per IP, per API key).
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(
        exchange.getRequest().getHeaders().getFirst("X-User-Id"));
}
```

**Why a Lua script in Redis?** The refill is a read-modify-write: read `{tokens, last_refill}`, compute tokens to add from elapsed time, clamp to bucket size, decrement by the cost, write back. Done as separate commands, two pods racing could both see "1 token left" and both succeed. Redis executes a Lua script **atomically** (single-threaded command execution), so the whole read-compute-write is one indivisible op — no distributed lock needed. Spring Cloud Gateway ships exactly this script.

### Leaky bucket — when you need *smooth* output

Requests enter a fixed-size queue and drain at a constant rate. Bursts are absorbed (queued) or dropped when the queue is full, but downstream sees a perfectly steady stream — ideal when the protected resource hates spikes (a legacy SOAP backend, a payment processor with a strict TPS cap). Cost: O(queue) memory and added latency (requests wait in line).

### Fixed window vs sliding window — the accuracy story

- **Fixed window counter:** one counter per window (`key = user:minute`), `INCR` with a TTL. O(1) and trivial, but it has the **boundary-burst flaw**: a client can send the full limit in the last second of one window and again in the first second of the next — 2× the limit across a window boundary.
- **Sliding window log:** a ZSET of request timestamps. On each request, `ZREMRANGEBYSCORE` to drop entries older than the window, `ZCARD` to count what remains, allow if under the limit, `ZADD` the new timestamp. Perfectly accurate, but memory grows with request volume — **not for high-throughput endpoints**.
- **Sliding window counter:** the practical middle ground. Keep two fixed-window counters (current and previous) and estimate:
  `estimate = prev_count × (1 − elapsed_fraction_of_current_window) + curr_count`.
  O(1) memory, and in production it lands within ~0.003% of the true sliding window. This is what most real limiters (e.g. Cloudflare's) actually use.

The boundary-burst example: a client sends 10 requests in the last 0.5s of window 1 and 10 in the first 0.5s of window 2 with a limit of 10/window. Fixed window allows all 20 (each window saw exactly 10). Sliding window counter, evaluated at the start of window 2, computes `prev_count × (1 − 0.5) + curr_count = 10 × 0.5 + 10 = 15 > 10` → it rejects the overflow. The weighted carry-over of the previous window is what catches the straddling burst.

### Distributed concern — fail-open vs fail-closed

When the rate-limiter's Redis is **down**, Spring Cloud Gateway's default is **fail-open**: requests are allowed and rate limiting is temporarily lost. That's the right call for an internal platform (availability > limiting). For billing or abuse-sensitive endpoints you want **fail-closed** (reject when you can't verify) — a deliberate, stated trade-off, not a default you forgot to change.

### Project tie-in

- **Smart360 API Gateway:** analytics users fire ~10 queries then idle for minutes → **token bucket** (`replenishRate=5`, `burstCapacity=20`) lets the burst through while capping sustained load. A sliding window log would *reject* their burst even though the system was idle — wrong tool for spiky human traffic.
- **WebX:** a strict downstream payment TPS cap → **leaky bucket** to smooth output and never exceed the processor's contractual rate.
- **Deep Fathom:** per-API-key fixed-window counters for coarse abuse protection where the boundary burst is acceptable and O(1) memory at high volume matters.

---

## Block C — DSA: DP Re-Derivation Drills (~1.5 hr)

### Theory

The goal of this block is **fluency**, not new problems. In an interview you won't recall code — you'll re-derive the recurrence in 60 seconds from the state sentence. Drill that motion until it's reflexive.

### Drill set

**Re-derive each recurrence on paper in ≤ 2 min, no peeking:**

| Problem | Say the state sentence | Then the transition |
|---|---|---|
| Climbing Stairs | "ways to reach step `i`" | `dp[i] = dp[i-1] + dp[i-2]` |
| House Robber | "max loot from houses `0..i`" | `dp[i] = max(dp[i-1], dp[i-2]+nums[i])` |
| House Robber II | "circle = two linear runs" | `max(rob(0..n-2), rob(1..n-1))` |
| Word Break | "is `s[0..i)` segmentable?" | `dp[i] = ∃ j<i: dp[j] ∧ s[j..i)∈dict` |
| LIS | "longest IS ending at `i`" | `dp[i] = max(dp[j]+1)` for `j<i, nums[j]<nums[i]` |
| Coin Change | "min coins for amount `i`" | `dp[a] = min(dp[a-c]+1)` per coin |

If you stall on any one, that's tonight's re-solve.

**Teach-back (rubber duck):** explain `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` out loud — *why* skipping the neighbour forces you back to `dp[i-2]`, not what the symbols are. If you can teach it, you own it.

**Memoization vs tabulation, stated cleanly:** memoization = top-down recursion + cache, only touches needed states, recursion-stack overhead; tabulation = bottom-up array fill, no stack risk, cache-friendly, easiest to space-optimise. Default to tabulation in production.

**Practice extensions (optional, if time):**
- **LC 746 Min Cost Climbing Stairs** — same shape as 70 but minimising a cost.
- **LC 91 Decode Ways** — Climbing Stairs with validity guards (a digit can't be 0; a two-digit chunk must be 10–26).
- **LC 152 Maximum Product Subarray** — track *both* max and min running products (a negative flips them).

---

## 💻 Practice coding questions

1. **LC 70 Climbing Stairs** — three forms; finish on the two-variable O(1) version.
2. **LC 198 House Robber** — rob-or-skip; two rolling variables; AC in 8 min.
3. **LC 213 House Robber II** — circle → two linear passes; handle `n==1`.
4. **LC 139 Word Break** — `dp[j] ∧ s[j..i)∈dict`; dict in a `HashSet`.
5. **LC 300 LIS** — O(n²) then the O(n log n) patience-sort `tails` array.
6. **LC 322 Coin Change** — unbounded knapsack; sentinel init; explain why greedy fails.
7. **LC 746 Min Cost Climbing Stairs** — minimise instead of count.
8. **LC 91 Decode Ways** — Climbing Stairs with 0/10–26 validity guards.
9. **LC 152 Maximum Product Subarray** — carry both running max and min.
10. **LC 53 Maximum Subarray** — Kadane's: `dp[i] = max(nums[i], dp[i-1]+nums[i])`.
11. **LC 121 Best Time to Buy/Sell Stock** — track min-so-far + best profit (1D scan).
12. **LC 140 Word Break II** — backtracking + memo of `index → suffix lists`.
13. **LC 354 Russian Doll Envelopes** — sort + LIS on the second dimension.

> For each: say the state sentence aloud *before* coding. Re-time LC 198 and LC 322 — target ≤ 8 and ≤ 12 min.

---

## 🎤 Interview questions

1. **What two properties must a problem have for DP to apply?** Optimal substructure (the answer is built from subproblem answers) and overlapping subproblems (the same subproblem recurs). Without overlap it's divide-and-conquer.
2. **Walk through your DP framework.** State (a sentence), transition (dependence on smaller states), base case (smallest answerable), direction (top-down memo vs bottom-up tabulation), then space-optimise if only the last few states are read.
3. **Memoization vs tabulation — when each?** Memo is easier to write and computes only needed states but risks stack overflow; tabulation is cache-friendly, stack-safe, and easiest to space-optimise — preferred for production. Use memo when the recurrence is hard to flatten into loops.
4. **Climbing Stairs is Fibonacci — is Fibonacci always DP?** Fibonacci is *one instance* of DP. DP is the general state+transition method; many DP problems are 2D or min/max (Edit Distance, Burst Balloons). Don't let the Fibonacci intuition make you expect everything to be additive.
5. **House Robber — justify the recurrence, don't recite it.** At house `i` you rob it (add `nums[i]` to the best ending at `i-2`, since `i-1` is now forbidden) or skip it (carry `dp[i-1]`); take the max. The adjacency rule *is* the `dp[i-2]` term.
6. **House Robber II — why does splitting into two passes work?** In a circle, house 0 and house n-1 are adjacent, so they can't both be robbed. Every valid selection excludes the first or the last; running House Robber I on `[0..n-2]` and `[1..n-1]` covers both cases.
7. **Coin Change — why doesn't greedy work?** coins=[1,3,4], amount=6: greedy 4+1+1=3 coins, DP 3+3=2. A locally-large coin can force more small coins later; DP evaluates every sub-amount so it isn't fooled.
8. **Coin Change I vs II — what changes?** I minimises coin *count* (`min`, value DP); II *counts ways* (sum DP). In II the loop order matters: outer=coins, inner=amount → combinations; swapping counts permutations.
9. **LIS in O(n log n) — explain the `tails` array.** `tails[k]` is the smallest tail of any increasing subsequence of length `k+1`. Binary-search each number's lower bound; extend if it's largest, else replace to keep tails minimal. The length of `tails` is the answer, not the array's contents.
10. **Word Break — why is the DP equivalent to DFS with memo?** `dp[j]` answers "can I segment up to index `j`?" — exactly the memoized "is this start position reachable?" The bottom-up table just fills those same states iteratively.
11. **When is DP the *wrong* tool in production?** When it's correct but too slow for the latency budget (Smart360's real-time LLM cost routing) — a heuristic that's 95% as good in 1ms beats an optimal DP in 50ms. Knowing when to abandon optimality is senior judgment.
12. **Token bucket vs leaky bucket — when each?** Token bucket allows controlled bursts (idle clients accumulate tokens) — best for spiky human traffic. Leaky bucket smooths output to a constant rate — best when the downstream can't tolerate spikes.
13. **Fixed window's boundary-burst flaw and the fix?** A client can send the full limit at the end of one window and again at the start of the next → 2× the limit briefly. Sliding window counter fixes it by weighting the previous window's count into the current estimate.
14. **Why a Lua script for Redis token bucket?** The refill is a read-compute-write; separate commands let two pods race and both succeed. Redis runs a Lua script atomically (single-threaded), making it one indivisible op — no distributed lock.
15. **Sliding window log vs counter — the trade-off?** Log (ZSET of timestamps) is perfectly accurate but O(requests-in-window) memory — bad at high throughput. Counter is O(1) and within ~0.003% accuracy via interpolation — the practical default.
16. **Your rate limiter's Redis goes down — what happens?** Spring Cloud Gateway defaults to fail-open (allow, lose limiting). Right for internal platforms; for billing/abuse endpoints switch to fail-closed (reject when unverifiable) — a deliberate trade-off.
17. **How do you decide *who* gets limited?** A `KeyResolver` keys the limit by user ID, API key, or IP. Per-user is fairest; per-IP risks punishing NAT'd corporate clients; per-API-key suits B2B quotas.
18. **Rate-limit a multi-instance gateway correctly.** Keep the counter in shared Redis (not in-process) so all gateway pods share one view; use the atomic Lua refill so concurrent pods don't double-spend tokens.

---

## ✅ Self-check

1. Write House Robber from a blank editor in 8 minutes, no notes — time yourself.
2. State the DP framework's five steps in order, then derive the Coin Change recurrence from its state sentence.
3. A client sends 10 requests in the last 0.5s of window 1 and 10 in the first 0.5s of window 2 (limit 10/window). Explain how the sliding window counter rejects the overflow that fixed window misses. (Weighted carry-over: `10×0.5 + 10 = 15 > 10`.)
4. Why does Coin Change need DP while Climbing Stairs is "just" Fibonacci? (Coin Change minimises over choices; greedy fails. Climbing Stairs is a pure additive count.)
5. Name the Redis structure behind each of the five rate-limiting algorithms.

---

<div align="center">

← [Week 3](../week-03/) · [Week 4](README.md) · [02-tue-jul-21.md](02-tue-jul-21.md) →

</div>
