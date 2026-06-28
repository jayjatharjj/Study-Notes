# Week 5 · Day 4 — Thu Jul 30 — DSA Maintenance + LLD Mock + Full-Stack LLM Design + CORS/Token

> Consolidation week shifts the centre of gravity: DSA drops to **maintenance** — two timed problems to keep the patterns warm, not learn anything new — and the real work moves to the two things that decide a senior full-stack loop. **Block B is a low-level-design (LLD) mock** — designing classes (Design Twitter / parking lot) with defensible SOLID, where the interviewer is watching *how you carve responsibilities*, not whether the code compiles — chased by a **full-stack LLM async-job feature** drawn end-to-end (202 Accepted → durable job state → SSE). **Block C is the full-stack integration story** that ties your Vue front-end to your Spring back-end as one coherent narrative: **CORS** (preflight + credentials), the **JWT client flow** (401 → refresh → retry, with the recursion guard), and the **API error contract**. Today is where "I've built features" becomes "I can design and integrate them under questioning."

📌 **Study today:** DSA maintenance — Timed Mixed Set #4 (favour weakest patterns) · LLD mock: Design Twitter classes + SOLID · full-stack LLM async-job feature · CORS + JWT client flow + API error contract · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA Maintenance: Timed Mixed Set #4

By Week 5 you are not learning patterns — you are keeping them at the surface so they fire cold in an OA. The maintenance set is deliberately small (two problems, 45 min total) but **run under real OA conditions**: timer on, no hints, fresh editor, no pre-announcing the pattern. The goal is fluency under a clock, not coverage.

### How to run it

1. **Pick 2 problems across patterns**, weighted toward whatever rated **weakest** in your Week 4 self-assessment. If DP/heap scored 3, pull one DP and one heap; if graphs were shaky, pull a BFS/DFS. Don't pick two of your strongest — maintenance means shoring up the soft spots.
2. **45-minute hard cap for both.** Clarify the inputs/edge cases aloud (even alone), then code.
3. **State complexity unprompted** — best and worst if they differ — and **name one alternative approach per problem** before you "submit." That second-approach reflex is what separates a senior signal from a junior one.

### Suggested rotation (don't reuse Mon–Wed picks)

| Pattern | Candidate (LC) | Key insight to re-verify | Complexity |
|---|---|---|---|
| DP (1D) | Coin Change (322) | unbounded knapsack; `dp[a] = min(dp[a], dp[a-coin]+1)` | O(amount·coins) |
| DP (2D) | Edit Distance (72) | three transitions: insert/delete/replace | O(mn) |
| Heap | Kth Largest (215) | size-k *min*-heap; root = k-th largest | O(n log k) |
| Graph | Course Schedule (207) | cycle detection = topological sort feasibility | O(V+E) |
| Sliding window | Longest Substring w/o Repeating (3) | shrink-from-left on a duplicate; last-index map | O(n) |
| Intervals | Merge Intervals (56) | sort by start, extend or push | O(n log n) |

**The maintenance discipline:** after each problem write the time taken, the one-line key insight, and the alternative. Anything that ran **over time** gets one extra rep tomorrow. That's the whole protocol — light, but non-negotiable, because the patterns decay if untouched for a week.

> **Worked re-verification — Coin Change (LC 322).** The maintenance value is re-deriving, not recalling:
> ```java
> public int coinChange(int[] coins, int amount) {
>     int[] dp = new int[amount + 1];
>     Arrays.fill(dp, amount + 1);          // sentinel "infinity" (can't need more than `amount` coins)
>     dp[0] = 0;
>     for (int a = 1; a <= amount; a++)
>         for (int coin : coins)
>             if (coin <= a) dp[a] = Math.min(dp[a], dp[a - coin] + 1);
>     return dp[amount] > amount ? -1 : dp[amount]; // sentinel survived → unreachable
> }
> ```
> Say aloud: "Unbounded knapsack — each coin reusable, so the amount loop is outer and I don't restrict reuse; the sentinel `amount+1` is the cleanest 'infinity'. O(amount·coins) time, O(amount) space. Alternative: BFS over amounts for the *fewest-coins* framing — same complexity, sometimes clearer."

---

## Block B — LLD Mock + Full-Stack LLM Feature

This block has two halves: a **45-minute LLD mock** (classes + SOLID) and a **15-minute full-stack LLM async-job design**. LLD is the round most mid-level candidates under-prepare — it's not "draw boxes at scale," it's "carve responsibilities into classes so a new requirement drops in without editing existing code." The interviewer is reading your *judgment about change*.

### LLD mock (45 min) — Design Twitter classes (or parking lot)

Pick one and drive it. **Design Twitter** is the higher-value pick because it bridges cleanly into the News Feed HLD you already drilled. Parking lot is the fallback (enums for spot/vehicle types, a `ParkingLot` that allocates the nearest fit, a `Ticket` value object, pricing Strategy) — same SOLID muscles, less HLD bridge.

**Design Twitter — the class breakdown:**

| Class / interface | Responsibility | What it must NOT do |
|---|---|---|
| `User` | id, profile (handle, name) | hold feed logic or know about tweets |
| `Tweet` | immutable value object: id, authorId, text, timestamp | be mutable or carry follow logic |
| `FollowGraph` | `follow` / `unfollow` / `followeesOf(userId)` — encapsulates the social graph | expose its internal map; merge timelines |
| `FeedService` | `postTweet(userId, text)` / `getTimeline(userId)` | construct its own dependencies or hard-code merge order |
| `TimelineMerger` (interface) | merge followees' tweets into one ordered timeline | be a single concrete class — it's the extension seam |

**The SOLID defence — say each out loud:**

- **SRP (Single Responsibility):** each class has exactly one reason to change. `FollowGraph` changes only if the social-graph storage changes; `FeedService` changes only if the *orchestration* changes. If a "muted accounts" requirement forced you to edit three classes, the responsibilities are wrong.
- **OCP + DIP (Open/Closed + Dependency Inversion):** `FeedService` depends on the **`TimelineMerger` interface**, not a concrete merger. Swapping chronological → ranked is a new implementation injected in — `FeedService` is never touched. That is the Open/Closed principle made concrete, and naming the **Strategy pattern** here is the senior signal.
- **The litmus test:** *"Could a new joiner add a ranked feed (or a muted-accounts filter) without modifying `FeedService`?"* If yes, your seams are right. If they'd have to crack open `FeedService`, you've coupled policy to orchestration.

```java
interface TimelineMerger {                       // the extension seam (Strategy)
    List<Tweet> merge(List<List<Tweet>> perFolloweeTweets, int limit);
}

class ChronologicalMerger implements TimelineMerger { // one impl — heap-merge of k sorted lists
    public List<Tweet> merge(List<List<Tweet>> lists, int limit) {
        PriorityQueue<Tweet> pq = new PriorityQueue<>((a, b) -> b.timestamp - a.timestamp);
        for (List<Tweet> l : lists) if (!l.isEmpty()) pq.offer(l.get(0)); // newest per followee
        // ... pop `limit`, advance that followee's pointer — the merge-K-sorted pattern
        return /* top `limit` by recency */ List.of();
    }
}

class FeedService {
    private final FollowGraph graph;
    private final TimelineMerger merger;          // ← depends on the abstraction (DIP)
    FeedService(FollowGraph graph, TimelineMerger merger) { // injected, not constructed here
        this.graph = graph; this.merger = merger;
    }
    List<Tweet> getTimeline(int userId, int limit) {
        var followees = graph.followeesOf(userId);
        // gather each followee's recent tweets, then delegate ordering to the strategy
        return merger.merge(/* per-followee tweet lists */ List.of(), limit);
    }
}
```

**Bridge to scale (the move that lands the round):** "The in-memory `FollowGraph` *is* the LLD. At scale this becomes the **fan-out-on-write vs fan-out-on-read** decision from the News Feed HLD — but the interface boundaries stay identical: `FeedService` still depends on `TimelineMerger`, the merger just reads from a precomputed Redis feed instead of in-memory lists. LLD and HLD are the same design at two zoom levels." This is the connective-tissue answer that proves you don't see LLD and HLD as separate skills.

**Two LLD anti-patterns to avoid out loud:**
- **God class** — a `Twitter` class that holds users, tweets, the graph, *and* the merge. One reason to change becomes five. Decompose.
- **Leaky encapsulation** — returning `FollowGraph`'s internal `Map<Integer, Set<Integer>>` so callers mutate it directly. Return copies or expose intent-named methods (`followeesOf`), never the field.

### Full-stack LLM async-job feature (15 min)

Draw it end-to-end, justifying each box — this is your WebX project generalised into an interview answer.

```text
UI ──POST /jobs──▶ API Gateway ──▶ LLM Orchestration Service
                                     │ 1. validate request
                                     │ 2. write PENDING row → PostgreSQL (durable)
                                     │ 3. publish job → Redis Streams / RabbitMQ
                                     └─ 4. return 202 Accepted + { jobId }
                                                   │
        Worker Pool ◀──consume── queue            │
          │ route to provider by cost/capability/rate-limit headroom
          │ (Redis atomic INCR/EXPIRE per-provider window)
          │ call provider → write result → status COMPLETED → publish completion
          ▼
Status/Result Service:  GET /jobs/{id}/status  (poll every 3s)
                        GET /jobs/{id}/stream  (SSE — partial tokens)
          ▼
UI: progress bar; on COMPLETED render result; on FAILED → error + retry button
```

**Justify every decision (the answers the interviewer is fishing for):**

| Decision | Why |
|---|---|
| **202 Accepted + jobId** (not sync response) | an LLM job can run 20 min; a synchronous HTTP request times out at the gateway/LB. Return immediately, let the client track. |
| **PostgreSQL for job state** | durable — a worker crash leaves a `PENDING`/`PROCESSING` row a watchdog can recover. In-memory state dies with the pod. |
| **Redis Streams for the queue + rate-limit** | atomic `INCR`/`EXPIRE` gives a distributed per-provider rate window; Streams give consumer groups + ack for at-least-once delivery. |
| **SSE over WebSocket** for the token stream | the stream is **unidirectional** (server→client), runs over HTTP/1.1, and proxies cleanly through the gateway. WebSocket's bidirectionality is overkill — reserve it for chat. |
| **Idempotency** | a duplicate "submit" click must not start two jobs. Client-supplied idempotency key (or a content hash) → return the *existing* jobId. Also prevents double-billing on provider retries. |

**The failure-mode close (always volunteer this):** "Worker pod killed mid-`PROCESSING` → heartbeat every 30s + a `@Scheduled` watchdog marks the job `STALE` after 2 min and re-queues it; because provider calls carry a request ID they're idempotent, so the re-run can't double-bill." That's the WebX curveball answered before it's asked.

---

## Block C — Full-Stack Integration: CORS / Token Handling / Error Contract

The signal here is being able to tell the **whole front-to-back story as one narrative** — most candidates know CORS *or* JWT in isolation; few connect Vue → gateway → Spring → error shape into a single coherent flow. Drill all three pieces, then say the 90-second integration story.

### CORS — preflight + credentials

The browser's **same-origin policy** is the root cause: when your front-end (`app.example.com`) calls your API (`api.example.com`), the browser **sends the request but blocks the *response* from JavaScript** unless the server returns the right CORS headers. CORS is a *browser* enforcement — `curl` and server-to-server calls ignore it entirely (a frequent point of confusion to call out).

- **Simple vs preflighted:** a "simple" request (GET/POST with standard headers) goes straight through. Anything non-simple — `PUT`/`DELETE`, a custom `Authorization` header, `Content-Type: application/json` — triggers a **preflight `OPTIONS`** first: the browser asks "may I send this?" and the server must answer **200** with `Access-Control-Allow-Methods` / `-Allow-Headers`. `Access-Control-Max-Age` caches that preflight so it isn't re-sent on every call.
- **Credentials (cookies/auth):** to send credentials you need **both** sides — server `Access-Control-Allow-Credentials: true` **and** client `withCredentials: true` (Axios) / `credentials: 'include'` (fetch). **The trap:** with credentials, `Access-Control-Allow-Origin` **cannot be `*`** — it must echo the exact origin. The `*` + credentials combo is the single most common CORS bug.
- **Where to configure it:** once, at the **Spring Cloud Gateway**, so downstream services inherit one consistent policy instead of each team re-implementing (and mis-implementing) it. "Security policy lives at the edge, not scattered across services."

### JWT client flow — the 401 → refresh → retry dance

- **Token storage:** access token **in memory** (a JS variable / Vuex-Pinia store), **never `localStorage`** — `localStorage` is readable by any injected script, so an **XSS** vulnerability becomes total token theft. The refresh token lives in an **HttpOnly, Secure, SameSite cookie** — unreadable by JS, sent automatically, immune to that XSS exfiltration.
- **Axios interceptors — the mechanism:**
  - *Request interceptor* attaches `Authorization: Bearer <accessToken>` to every outgoing call.
  - *Response interceptor* on **401** → call `/auth/refresh` (the HttpOnly cookie rides along automatically) → store the new access token → **retry the original request** transparently. The user never sees the blip.
- **The recursion guard (the curveball everyone misses):** if the request that 401'd **is itself `/auth/refresh`**, do **not** retry — the refresh token is expired too. Detect that URL in the interceptor and **redirect to login**. Without this guard you get an **infinite refresh loop** when both tokens are dead (the "after 15 min idle, actions fail silently" bug).
- **Other status codes:** **403** = authenticated but not authorized → show "access denied," **do not** retry. **5xx / network** → a friendly toast, never a raw exception/stack string leaked to the user.

```js
// response interceptor — the core of the flow
api.interceptors.response.use(r => r, async err => {
  const req = err.config;
  if (err.response?.status === 401 && !req._retried) {
    if (req.url.includes('/auth/refresh')) {        // ← recursion guard
      forceLogout(); return Promise.reject(err);     // both tokens dead → login
    }
    req._retried = true;
    await refreshAccessToken();                      // HttpOnly cookie sent automatically
    return api(req);                                 // retry the original request
  }
  return Promise.reject(err);
});
```

### API error contract — consistent shape, consumed uniformly

The back-end must give **every** error the same JSON shape so the front-end handles them with one code path:

- **`@RestControllerAdvice` + `@ExceptionHandler`** maps domain exceptions to consistent JSON: `{ timestamp, status, error, message, path }`.
- **Mapping:** `@Valid` failure → **400** with a field-level error list; a violated business rule → **409**; a downstream/provider failure → a clean **502/503** (never leak the raw downstream stack).
- **Front-end consumption:** **field/validation errors render inline** on the form; **system errors go to a toast**; the loading spinner resets in a **`finally`** so the UI never hangs on error.

### Say the integration story aloud (under 90s)

> "Vue talks to Spring Boot through the **gateway**, which does JWT validation, CORS, and routing in one place. The front-end uses a single global **Axios instance** with a **request interceptor** that attaches the Bearer token and a **response interceptor** that, on a 401, silently refreshes via the HttpOnly cookie and retries — with a recursion guard so a dead refresh token redirects to login instead of looping. On the back-end, **`@RestControllerAdvice`** gives every error one consistent JSON shape, so the front-end consumes validation errors inline and system errors as a toast — uniformly. One token flow, one CORS policy at the edge, one error contract."

---

## 💻 Practice coding questions

1. Coin Change (LC 322) — re-derive the unbounded-knapsack DP cold; state the BFS alternative.
2. Edit Distance (LC 72) — the three transitions; space-optimise to two rows.
3. Kth Largest (LC 215) — size-k min-heap; name QuickSelect as the in-memory alternative.
4. Course Schedule (LC 207) — topological sort / cycle detection; Kahn's vs DFS-colour.
5. Longest Substring Without Repeating (LC 3) — sliding window with a last-index map.
6. Sketch the `TimelineMerger` interface + a `ChronologicalMerger` (heap-merge of k sorted lists) and a `RankedMerger` stub — prove `FeedService` needs no edit to swap them.
7. Write the parking-lot core: enums for spot/vehicle type, `ParkingLot.park()` allocating the nearest fit, a pricing `Strategy` — defend SRP.
8. Code the Axios response interceptor with the 401 → refresh → retry flow **and** the `/auth/refresh` recursion guard.
9. Write a `@RestControllerAdvice` that maps `MethodArgumentNotValidException` → 400 (field list) and a `BusinessRuleException` → 409, both as the standard JSON shape.
10. (Stretch) Add a "muted accounts" filter to the Twitter LLD without touching `FeedService` — prove the seam holds.

---

## 🎤 Interview questions

1. **In your Twitter LLD, why does `FeedService` depend on a `TimelineMerger` interface, not a concrete class?** So a ranked or chronological merge is a swappable Strategy injected in — `FeedService` never changes when the ordering policy does (Open/Closed + Dependency Inversion). The litmus test: a new joiner adds a ranked feed without editing `FeedService`.
2. **What's the SRP justification for splitting `FollowGraph` out of `FeedService`?** Each has one reason to change: `FollowGraph` if graph storage changes, `FeedService` if orchestration changes. Merging them means a graph change risks the feed logic and vice versa.
3. **How does the LLD relate to the News Feed HLD?** Same design, two zoom levels: the in-memory `FollowGraph` becomes fan-out-on-write vs fan-out-on-read at scale, but `FeedService → TimelineMerger` boundaries are identical — the merger just reads a precomputed Redis feed instead of in-memory lists.
4. **Why return 202 Accepted for the LLM job instead of the result?** The job can run minutes; a synchronous request times out at the gateway/LB. Return immediately with a jobId; the client polls or subscribes to SSE.
5. **Why PostgreSQL for job state and not just the queue?** Durability — a worker crash leaves a `PROCESSING` row a watchdog can detect and re-queue. The queue alone gives at-least-once delivery but no recoverable record of in-flight work.
6. **SSE vs WebSocket for the token stream?** SSE: unidirectional server→client, HTTP/1.1, proxies cleanly through the gateway — right for a read-only token stream. WebSocket's bidirectionality is overkill here; reserve it for chat where the client also pushes.
7. **How do you make the job submission idempotent?** A client-supplied idempotency key (or content hash) maps to an existing jobId, so a duplicate click returns the in-flight job rather than starting a second — also prevents double-billing on provider retries.
8. **Explain CORS preflight.** For non-simple requests (PUT/DELETE, `Authorization` header, JSON body), the browser first sends an `OPTIONS` asking permission; the server answers 200 with allowed methods/headers, cached by `Access-Control-Max-Age`. Only then is the real request sent.
9. **Why can't `Access-Control-Allow-Origin` be `*` with credentials?** The spec forbids wildcard origin when `Allow-Credentials: true` — you must echo the exact origin. The `*` + credentials combo silently fails; it's the #1 CORS bug.
10. **Why store the access token in memory, not localStorage?** `localStorage` is readable by any injected script, so an XSS hole becomes full token theft. In-memory dies on refresh (acceptable), and the refresh token sits in an HttpOnly cookie JS can't read.
11. **(Curveball) After 15 min idle, actions fail silently with no redirect — debug.** Access *and* refresh tokens both expired; the 401 from `/auth/refresh` is being caught and retried recursively into an infinite loop. Fix: detect the refresh URL in the interceptor and redirect to login instead of retrying.
12. **(Curveball) Front-end on app.example.com, API on api.example.com — browser blocks the call. Debug.** DevTools → Network → look for a failed `OPTIONS` or a missing `Access-Control-Allow-Origin`; verify exact-origin match (no `*` with credentials), `withCredentials` + `Allow-Credentials` on both sides, the gateway CORS path pattern, and no trailing-slash mismatch.
13. **Why configure CORS at the gateway instead of each service?** One consistent policy at the edge — security isn't scattered across teams who each mis-configure it, and downstream services stay focused on business logic.
14. **What does `@RestControllerAdvice` buy you?** Every exception maps to one consistent JSON shape (`timestamp/status/error/message/path`), so the front-end consumes all errors through one path — inline for validation (400), toast for system (5xx).
15. **(Curveball) Two users submit the same LLM prompt at the same instant — one job or two?** Idempotency is per-key/per-user, not per-content-globally — two different users get two jobs (different billing/ownership). The same user double-clicking gets one. If dedup across users is desired, a content-hash cache returns the shared result, but ownership/audit must still record both requests.

---

## ✅ Self-check

1. Could a new joiner add a ranked feed (or muted-accounts filter) to your Twitter LLD **without modifying `FeedService`**? Name the principle and the pattern.
2. Draw the full-stack LLM feature end-to-end and justify 202, PostgreSQL job state, Redis rate-limit, SSE-over-WebSocket, and idempotency — one sentence each.
3. Explain step-by-step what happens on a 401 mid-session, including the recursion guard, and why `localStorage` is an XSS risk.
4. State the `*` + credentials CORS trap and where you'd configure CORS — in one breath.
5. Did both timed DSA problems finish inside 45 min, with complexity *and* an alternative stated for each?

---

*Nav: ← [Day 3 (Wed Jul 29)](03-wed-jul-29.md) · [Week 5](README.md) · [Day 5 (Fri Jul 31)](05-fri-jul-31.md) →*
