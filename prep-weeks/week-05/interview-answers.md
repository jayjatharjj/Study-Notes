# Week 5 — Interview Question Answers

Crisp model answers to every interview question across Week 5 — system-design mocks (URL shortener, file storage, news feed, notification, chat, typeahead, LLD), 10 STAR stories, and the three project deep-dives. For STAR/project prompts, see [projects-reference.md](../../projects-reference.md); for design patterns see [cheat-sheets/system-design.md](../../cheat-sheets/system-design.md).

---

## Day 1 — URL Shortener / File Storage HLD + STAR #1–#5

1. **Design a URL shortener — start with capacity.**
   ~100M new URLs/month → ~40 writes/s avg (~400 peak); reads are 100× → ~4 000 reads/s avg; ~6B rows over 5 yr at ~500 B = ~3 TB (trivially shardable). Base-62 over `[A-Za-z0-9]` gives 62⁷ ≈ 3.5T, so **7-char keys** cover the horizon. Why capacity first: the read/write skew (100:1) is what justifies the cache-aside redirect path. Gotcha: state the numbers aloud — the design decisions must trace back to them.

2. **301 vs 302 for the redirect — which and why?**
   301 (Permanent) is browser/proxy-cached so subsequent clicks never reach you — fastest and cheapest but you **lose click analytics and can't change the target**. 302 (Found) hits your server every time — full analytics, editable target, more load. Default **302 when analytics matter** (the usual product need); 301 only for immutable links where tracking is irrelevant. Gotcha: 301's caching is sticky — once a browser caches it you can't reclaim those clicks.

3. **How do you keep the redirect under 100 ms?**
   Cache-aside in Redis (`code → longUrl`): hit → redirect; miss → DB → populate with TTL → redirect. With 100:1 read skew and hot-key caching, almost every redirect is a single-digit-ms memory hit. Why it works: the working set of popular links is tiny relative to total. Gotcha: never write analytics synchronously on the redirect — fire an async event (Kafka/queue) so the hot path stays clean.

4. **File storage — walk the upload flow.**
   Client → Upload Service returns a `fileId` + per-chunk pre-signed S3 URLs → client uploads chunks **directly to S3** (no bytes ever flow through your servers) → client signals complete → async assembly (`CompleteMultipartUpload`) → Metadata Service sets `status=READY` → publishes `FileUploaded`. Why: offloading bandwidth to S3 keeps the app tier from becoming the throughput bottleneck. Gotcha: this is Smart360's single-file pre-signed pattern extended to per-chunk for files up to 5 GB.

5. **Why shard file metadata on `owner_id`?**
   It co-locates a user's files on one shard so "list my files" is a single-shard query, not scatter-gather across the cluster. Why it matters: the listing path is read-heavy and latency-sensitive. Gotcha: the trade-off is that cross-user admin queries now fan out across shards — acceptable because they're rare.

6. **(Curveball) Two users upload a file with the same name simultaneously.**
   No conflict — the name isn't the primary key; a `file_id` UUID is, so both succeed independently. If names must be unique *per folder*, a `(owner_id, folder_id, name)` unique constraint makes the second writer lose with a 409, enforced inside a single shard. Gotcha: the constraint only holds because both rows live on the same `owner_id` shard.

7. **You uploaded a file but it's missing from "my files." Why, and fix?**
   Read-replica lag — the list query hit a follower that hadn't yet received the new metadata row from the leader. Fix: route post-upload reads to the leader for a short window, or render the file optimistically from the POST response while the replica catches up. Gotcha: this is the classic read-your-own-writes problem with leader-follower replication.

8. **How do you prevent orphaned chunks from incomplete uploads?**
   An S3 lifecycle rule deletes incomplete multipart uploads after 24 h; the metadata row stays `PENDING` until assembly succeeds, so it never appears as `READY`. Why: clients abandon uploads, and orphaned chunks otherwise accumulate cost silently. Gotcha: don't mark metadata `READY` before assembly completes, or you'll surface broken files.

9. **Why pre-signed URLs at all — why not proxy the bytes?**
   Pre-signed URLs offload bandwidth to S3, so your app tier never becomes the throughput bottleneck; the client talks straight to object storage with a short-lived, scoped credential. Why: proxying GBs through your servers means scaling the app tier for raw I/O. Gotcha: the URL must be short-TTL and scoped to one object/operation, or it becomes a leak vector.

10. **Tell me about a performance problem you solved. (STAR #1)**
    Smart360 dashboards loaded in ~60 s; Hibernate statistics revealed 47 queries per load — a classic N+1. I fixed it with `JOIN FETCH` + `@EntityGraph`, added composite indexes `(tenant_id, created_at)` and `(user_id, status)`, a Redis read cache (5-min TTL), and cached S3 pre-signed URLs (80% fewer S3 calls) — **60 s → 2–3 s (96%)**, zero schema changes on a live multi-tenant DB. Gotcha: the mechanism (N+1 → JOIN FETCH) is the proof the number is real. See [projects-reference.md](../../projects-reference.md).

11. **Tell me about a disagreement with a colleague. (STAR #2)**
    On a real design split (e.g. stateless JWT vs server sessions), I documented both options with explicit trade-offs, proposed a time-boxed PoC / decision matrix, and acknowledged the kernel of validity in their concern (session revocation is genuinely simpler). We reached consensus on the data; where we couldn't, I escalated cleanly with the matrix — relationship intact, chosen path shipped. Gotcha: disagree on the idea, never the person. See [projects-reference.md](../../projects-reference.md).

12. **Tell me about something you owned end-to-end that no one asked you to. (STAR #3)**
    CI/CD took ~23 min and everyone complained but no one owned it, so I self-assigned the fix. The GitLab pipeline trace exposed 3 bottlenecks (serial stages, no layer caching, redundant builds); I restructured into a DAG with parallel stages and enabled BuildKit caching — **23 → 10 min (57%)**, compounding across every developer every day. Gotcha: proactive identification plus a measured, team-wide impact is the ownership signal. See [projects-reference.md](../../projects-reference.md).

13. **What's your biggest failure? (STAR #4)**
    I shipped the LLM-proxy routing without retry or fallback, assuming provider uptime; a partial outage then failed jobs in production. I owned it immediately, added retry with exponential backoff + a fallback provider + a circuit breaker, and made "design for failure first" a sign-off checklist item. Gotcha: be honest about the consequence and concrete about the lesson and the follow-through — keep it under 90 s. See [projects-reference.md](../../projects-reference.md).

14. **Why are you leaving? (STAR #5)**
    Frame on ambition, never negativity: "I've had strong ownership and impact, and I'm looking for larger scale, a stronger engineering culture, and compensation that reflects the level I'm operating at — I want to own a domain end-to-end at a company building at scale." Gotcha: never criticise the current employer; rehearse until it sounds natural, not scripted. See [projects-reference.md](../../projects-reference.md).

15. **(Behavioral curveball) What would your current manager say is your weakness?**
    Pick a real, improving one: "I used to over-engineer for hypothetical scale; I now size the solution to the actual load and document why." Why it lands: it's specific and shows a concrete change, not a humble-brag. Gotcha: avoid disguised strengths ("I work too hard") — interviewers see through them instantly.

16. **Why a size-k *min*-heap for the k *largest*?**
    The size-k min-heap keeps the k best elements seen; its root (the smallest of those) is exactly the k-th largest. It's O(n log k) and streaming-friendly, versus a max-heap of all n at O(n log n). Gotcha: it's counter-intuitive — you use a *min*-heap to find the *largest* because you evict whatever falls below the current k-th best.

17. **(DSA) Coin Change — why does loop order matter in the count-ways variant?**
    Coins-outer / amounts-inner counts *combinations* (each coin considered once per pass); the reverse counts *permutations*, double-counting orderings like {1,2} and {2,1}. Why: the outer loop fixes which coins are "available" so far. Gotcha: for the *min-coins* problem order doesn't affect correctness — only the count-ways variant (LC 518) is sensitive to it.

---

## Day 2 — News Feed / Notification HLD + STAR #6–#10

1. **Design a News Feed — fan-out on write or read?**
   The hybrid: push (fan-out-on-write) for normal users, pull (fan-out-on-read) for celebrity/high-follower accounts above a tunable threshold. Why: it caps write amplification (you never fan out a 1M-follower post) while keeping reads O(1) for the 99%. Gotcha: name the follower threshold as an explicit, tunable knob — it's the lever the whole design hinges on.

2. **Open with the core tension of a News Feed.**
   Write amplification vs read amplification. Push makes reads O(1) but a 1M-follower post means 1M feed writes; pull makes writes cheap (store once) but every read becomes a fan-in merge across all followees. Why lead with this: the asymmetry between rare posts and frequent reads is the entire design. Gotcha: stating the tension up front signals you understand the problem before drawing boxes.

3. **Why not `OFFSET/LIMIT` for the feed?**
   `OFFSET` skips a fixed count and is unstable under concurrent writes — a new post shifts the window, causing duplicates or skipped items. Use an opaque cursor `(score, post_id)` and query "items with score < cursor," which anchors to a value rather than a position. Gotcha: this is true for any high-write paginated list, not just feeds.

4. **How do you keep the feed cache bounded?**
   Per-user Redis sorted set (`post_id → score`) capped at ~800 entries; inactive users' feeds are generated lazily on login rather than maintained continuously. Why: most users read only the top of their feed, and you shouldn't pay to maintain feeds for users who never read. Gotcha: lazy generation trades a slightly slower first-login read for huge savings on inactive accounts.

5. **Candidate generation vs ranking — why separate them?**
   Candidate generation is the fan-out (who *could* appear); ranking is a swappable scorer over those candidates (recency, affinity, engagement velocity, content type). Separating them lets you swap chronological → ranked without touching fan-out. Gotcha: collapsing the two means any ranking change forces a rewrite of the candidate path.

6. **Notification — how do you guarantee no duplicates?**
   An idempotency key = `event_id + user_id + channel`, enforced with Redis `SETNX` or a DB unique constraint — the second attempt is a no-op. Why it matters: duplicate notifications are the #1 user complaint. Gotcha: dedup must be per-channel — the same event legitimately fans out to push *and* email, so the channel is part of the key.

7. **A user muted marketing — should they still get a security alert?**
   Yes. Preferences are consulted *before* dispatch at category granularity, so security/transactional categories bypass a marketing mute. Why before dispatch: checking after you've sent is too late, and category-level control prevents a blanket mute from killing critical alerts. Gotcha: never treat "muted" as global — categorise.

8. **What happens when a notification provider is down?**
   Per-channel workers retry with exponential backoff; on retry exhaustion the message goes to a dead-letter queue (DLQ) for inspection — never silently dropped. Other channels are unaffected because workers are isolated per channel. Gotcha: an SMS gateway outage must not block email/push — isolation is the reliability property.

9. **Why "outbox + at-least-once + idempotency" and not just "send it"?**
   The outbox records the event atomically with the business change so no event is ever lost; at-least-once delivery plus consumer-side idempotency gives *effectively* exactly-once despite retries. Why not just send: a naive send loses events on crash and duplicates on retry. Gotcha: true exactly-once delivery is impossible — you get exactly-once *effect* via idempotency.

10. **How does the News Feed reuse a pattern from your work?**
    Fan-out-on-write is the outbox/Kafka pattern from Deep Fathom — `PostCreated` → Kafka → feed-builder workers update follower caches asynchronously, the same decoupling as the notification service. Why it lands: grounding the design in shipped work outranks a textbook answer. Gotcha: name the concrete event and worker, not just "I used Kafka." See [projects-reference.md](../../projects-reference.md).

11. **Tell me about yourself. (STAR #6)**
    90-second pitch, never chronological: current role (Java 17 / Spring Boot, Azure, Kubernetes) → impact numbers first (60 s → 2–3 s / 96%, CI/CD 23 → 10 min / 57%, 30+ Bicep resources, 5-provider LLM proxy) → what I want (own a domain end-to-end at scale) → bridge to this company. Gotcha: lead with numbers in the first sentence, not the third. See [projects-reference.md](../../projects-reference.md).

12. **Tell me about a time you mentored someone. (STAR #7)**
    A junior was stuck on the multi-tenant flow; I taught the *why* (how tenant isolation propagates from the JWT through Hibernate filters), left a runbook so the next person wouldn't need me, and followed up a week later. Result: their ramp shortened (they shipped the next multi-tenant feature solo) and the runbook became team reference. Gotcha: teach the why, don't just hand over the fix. See [projects-reference.md](../../projects-reference.md).

13. **Tell me about tough feedback you received. (STAR #8)**
    A senior told me my PRs were too large to review well. I listened without defending, asked clarifying questions, then made a visible change — split work into small PRs each with a one-line intent — and closed the loop by asking the reviewer if it was better. Result: review turnaround improved and they noticed unprompted. Gotcha: receive feedback as a gift; the closed loop is what proves you acted. See [projects-reference.md](../../projects-reference.md).

14. **How do you prioritise under a tight deadline? (STAR #9)**
    Make an explicit must-have vs nice-to-have split, cut by reversibility and user impact (defer anything reversible and low-impact), and communicate the cut to stakeholders *before* it becomes a surprise. Result: hit the date with must-haves solid; deferred items shipped next cycle. Gotcha: prioritisation is a decision you communicate, not a corner cut quietly. See [projects-reference.md](../../projects-reference.md).

15. **Tell me about customer impact you drove. (STAR #10)**
    Users reported dashboards timing out; I started from the user pain, traced it to the N+1, fixed it, and validated against the user outcome — not just the latency graph. Result: users stopped abandoning the page, satisfaction rose, and the 96% engineering metric followed *because* the user goal drove it. Gotcha: lead with user value, let the metric be the consequence. See [projects-reference.md](../../projects-reference.md).

16. **(DSA) Rotting Oranges — why multi-source BFS?**
    Seeding the queue with all rotten cells at once models simultaneous spread; each BFS level equals one minute. Single-source BFS would compute a wrong, sequential spread. Gotcha: after the BFS, if any fresh orange remains, return -1 — don't forget the unreachable check.

17. **(DSA) Sliding window — the classic bug in Longest Substring?**
    Moving `left` to `lastIndex + 1` without a `max` clamp: a stale last-index from outside the current window can drag `left` *backwards*, corrupting the length. Fix: `left = max(left, lastIndex + 1)`. Gotcha: this only bites when a repeated char's earlier occurrence is already behind the window's left edge.

18. **(Curveball) Celebrity posts and a normal user share one feed — ordering?**
    At read time, merge the pushed (precomputed) feed with the pulled celebrity posts using the same score (timestamp or rank), then the cursor operates on the merged, ordered result. Why: both sources share a scoring scheme, so the merge is a standard k-way merge by score. Gotcha: the cursor must anchor to the merged stream, not either source individually.

---

## Day 3 — Chat / Typeahead HLD + Project Deep-Dives

1. **Chat — WebSocket or SSE, and why?**
   WebSocket: chat needs bidirectional traffic (client → server too), whereas SSE is server → client only. SSE suits the read-only LLM token stream, not chat. Gotcha: don't reach for WebSocket reflexively — for a one-way stream SSE is simpler to proxy; reserve WebSocket for genuine bidirectionality.

2. **How do you route a message to a user across many backend servers?**
   A gateway layer maps `userId → (server, connectionId)` in Redis, so any backend can locate a recipient's live socket regardless of which node it landed on. Heartbeats (ping/pong) clean up dead mappings. Gotcha: with millions of long-lived connections the mapping is the crux — without it you can't deliver to a user whose socket is on a different node.

3. **Why order messages by sequence number, not timestamp?**
   A per-conversation monotonic sequence number, assigned server-side, is immune to sender clock skew; two phones with skewed clocks would otherwise reorder messages. Clients sort by sequence. Gotcha: the sequence generator is a per-conversation coordination point — cheap, because conversations partition naturally.

4. **How do you track presence at scale?**
   Heartbeat TTL keys: `SETEX presence:userId 30s` on each ping; key expiry means offline. Fan out presence changes to contacts; eventual consistency is fine (a few seconds' stale "last seen" hurts no one). Gotcha: don't try to make presence strongly consistent — the cost isn't worth it for a soft signal.

5. **Group chat — how do you fan out?**
   Small groups → fan-out-on-write (write the message into each member's inbox); large channels → fan-out-on-read (store once, members pull). It's the same hybrid as the News Feed, for the same reason: cap write amplification on huge groups. Gotcha: the threshold between "small" and "large" is a tunable knob.

6. **Why a wide-column store for messages, and what keys?**
   The access pattern is append + range-scan by conversation (no joins) at write-heavy scale, which fits Cassandra/HBase. Partition by `conversation_id`, cluster by `message_seq` → efficient "fetch last N / paginate older." Gotcha: a relational store with joins is the wrong shape here; TTL/archival tiering moves cold messages out.

7. **Typeahead — how do you hit <100 ms?**
   A sharded trie/FST with precomputed top-k completions at each node → a query is walk-to-the-prefix-then-read, never a request-time DFS over the subtree. Debounce on the client (~150–300 ms); cache hot prefixes in Redis. Gotcha: precomputing top-k per node is the trick — a request-time subtree search would blow the latency budget.

8. **How does the typeahead index stay fresh without racey reads?**
   An offline pipeline aggregates query logs → rebuilds the trie → ships it as a versioned blob with an atomic pointer flip, so reads never see a half-built index. Gotcha: the atomic swap is what prevents readers from observing a partially built trie; accept minor staleness for race-free updates.

9. **Why rank typeahead by frequency, not alphabetically?**
   Users want popular completions — "fa" should suggest "facebook," not "fabaceae." Frequency comes from aggregated query logs in the offline pipeline. Gotcha: alphabetical ranking is trivial to build but useless for relevance; the value is entirely in the popularity signal.

10. **Walk me through Smart360.**
    Multi-tenant SaaS (Java 17 / Spring Boot / Vue). Performance: N+1 (47 queries/load) → `JOIN FETCH`/`@EntityGraph` + composite indexes + Redis + S3 URL cache → 60 s → 2–3 s (96%). Security: server sessions → stateless JWT at the gateway (~60% sign-in cut) + 3-level RBAC. Architecture: strangler-fig migration, notification boundary first (~30% gain). Gotcha: each contribution carries its own number. See [projects-reference.md](../../projects-reference.md).

11. **(Curveball) Smart360's hardest bug?**
    `LazyInitializationException` — accessing a lazy association outside the Hibernate session/transaction. Fixed with `@Transactional(readOnly = true)` on the read path and `@EntityGraph` to fetch eagerly where needed. Gotcha: the root cause is the session closing before the lazy proxy is touched — fix it at the fetch strategy, not by forcing EAGER everywhere.

12. **(Curveball) How does Smart360 enforce multi-tenancy?**
    Tenant id rides in the JWT → a Hibernate `@Filter` auto-applies the tenant predicate to every query → Postgres Row-Level Security (RLS) as a database-level backstop so a code bug can't leak cross-tenant. Gotcha: defence in depth — the RLS backstop matters precisely because application filters can be forgotten.

13. **Walk me through Deep Fathom's infra.**
    FedRAMP-aligned Azure: 30+ resources in modular Bicep (Container Apps, PostgreSQL Flexible, Redis, Key Vault, Front Door, private endpoints), one parameter file per environment. CI/CD restructured into a DAG with BuildKit → 23 → 10 min (57%). Reliability: liveness/readiness probes + graceful shutdown, managed-identity Key Vault, RLS across 50+ tables. Gotcha: same modules, different param file per env — that's the IaC discipline. See [projects-reference.md](../../projects-reference.md).

14. **(Curveball) Key Vault is down when Deep Fathom starts.**
    Deploy fails safe — a pod that can't fetch its secrets at boot won't serve traffic, which beats running misconfigured. But runtime secrets are cached in memory, so a post-startup Key Vault outage doesn't take running pods down. Gotcha: name *both* cases (startup fails safe, runtime is resilient) — that's the senior answer.

15. **Walk me through WebX.**
    LLM orchestration for 2–20 min jobs across 5 providers. `POST /jobs` → 202 + jobId; PostgreSQL for durable job state + Redis Streams as the work queue; a worker pool routes by cost/capability/rate-limit headroom (Redis atomic INCR/EXPIRE per provider); results via SSE/polling; idempotency keys dedupe submissions. Gotcha: a synchronous response would time out on a 20-min job — async is forced by the runtime. See [projects-reference.md](../../projects-reference.md).

16. **(Curveball) WebX worker pod killed mid-job.**
    Neither lost nor double-billed: workers heartbeat every 30 s; a `@Scheduled` watchdog marks a job STALE after ~2 min of no heartbeat and re-queues it. Because provider calls carry a request id they're idempotent, so the re-run doesn't double-bill. Gotcha: durability (the PostgreSQL row) is what lets the watchdog recover the orphaned job.

17. **Tie your three projects into one sentence.**
    "Smart360 taught me performance engineering, Deep Fathom infra ownership, WebX async + LLM integration — now I want to own a domain end-to-end at scale." Gotcha: the arc should sound like growth, not a list. See [projects-reference.md](../../projects-reference.md).

18. **(DSA) Why sentinel head/tail nodes in the LRU list?**
    They eliminate every null-check at the list edges; insert and remove become unconditional pointer swaps, keeping both O(1) and bug-free. Gotcha: without sentinels you special-case empty list, single element, and head/tail removal — the sentinels collapse all those branches.

19. **(DSA) LRU vs LFU — what changes?**
    LRU evicts least-recently-used via a doubly-linked list (move-to-front on touch). LFU evicts least-*frequently*-used with recency as the tie-break, needing frequency buckets + a min-freq pointer — a step harder. Gotcha: LFU's min-freq pointer is what keeps eviction O(1) despite tracking counts.

20. **(Curveball) SSE vs WebSocket for the LLM stream — defend it.**
    SSE is unidirectional server → client, runs over plain HTTP/1.1, and proxies trivially through the gateway — right for a read-only token stream. WebSocket's bidirectionality is overkill and adds proxying/connection complexity; reserve it for chat where the client also pushes. Gotcha: "use the least powerful tool that fits" is the principle to name.

---

## Day 4 — LLD Mock + Full-Stack LLM Design + CORS/Token

1. **In your Twitter LLD, why does `FeedService` depend on a `TimelineMerger` interface, not a concrete class?**
   So a ranked or chronological merge is a swappable Strategy injected in — `FeedService` never changes when the ordering policy does (Open/Closed + Dependency Inversion). The litmus test: a new joiner adds a ranked feed without editing `FeedService`. Gotcha: depending on the abstraction is what makes the seam an *extension point* rather than a rewrite.

2. **What's the SRP justification for splitting `FollowGraph` out of `FeedService`?**
   Each has exactly one reason to change: `FollowGraph` if the social-graph storage changes, `FeedService` if the orchestration changes. Merging them means a graph change risks the feed logic and vice versa. Gotcha: if a "muted accounts" requirement forced edits to three classes, the responsibilities are drawn wrong.

3. **How does the LLD relate to the News Feed HLD?**
   Same design, two zoom levels: the in-memory `FollowGraph` becomes the fan-out-on-write vs fan-out-on-read decision at scale, but the `FeedService → TimelineMerger` boundaries are identical — the merger just reads a precomputed Redis feed instead of in-memory lists. Gotcha: showing LLD and HLD are one design proves you don't treat them as separate skills.

4. **Why return 202 Accepted for the LLM job instead of the result?**
   The job can run minutes; a synchronous request times out at the gateway/load balancer. Return immediately with a jobId and let the client poll or subscribe to SSE. Gotcha: 202 (Accepted, not yet complete) is the correct semantic — 200 would imply the work is done.

5. **Why PostgreSQL for job state and not just the queue?**
   Durability — a worker crash leaves a `PROCESSING` row a watchdog can detect and re-queue. The queue alone gives at-least-once delivery but no recoverable record of in-flight work. Gotcha: in-memory state dies with the pod; the DB row is what makes recovery possible.

6. **SSE vs WebSocket for the token stream?**
   SSE: unidirectional server → client, HTTP/1.1, proxies cleanly through the gateway — right for a read-only token stream. WebSocket's bidirectionality is overkill here; reserve it for chat where the client also pushes. Gotcha: same principle as Day 3 — match the tool to the directionality.

7. **How do you make the job submission idempotent?**
   A client-supplied idempotency key (or content hash) maps to an existing jobId, so a duplicate "submit" click returns the in-flight job rather than starting a second — also prevents double-billing on provider retries. Gotcha: the key must be client-supplied and stable across retries, or the dedup window is useless.

8. **Explain CORS preflight.**
   For non-simple requests (PUT/DELETE, an `Authorization` header, a JSON `Content-Type`), the browser first sends an `OPTIONS` asking permission; the server must answer 200 with `Access-Control-Allow-Methods`/`-Allow-Headers`, cached by `Access-Control-Max-Age`. Only then is the real request sent. Gotcha: CORS is *browser-enforced* — curl and server-to-server calls ignore it entirely.

9. **Why can't `Access-Control-Allow-Origin` be `*` with credentials?**
   The spec forbids a wildcard origin when `Access-Control-Allow-Credentials: true` — you must echo the exact origin. The `*` + credentials combo silently fails. Gotcha: this is the single most common CORS bug; you also need client-side `withCredentials`/`credentials: 'include'`.

10. **Why store the access token in memory, not localStorage?**
    `localStorage` is readable by any injected script, so an XSS hole becomes total token theft. In-memory storage dies on refresh (acceptable), and the refresh token sits in an HttpOnly, Secure, SameSite cookie that JS can't read. Gotcha: the HttpOnly cookie is immune to XSS exfiltration precisely because JS can't access it.

11. **(Curveball) After 15 min idle, actions fail silently with no redirect — debug.**
    Both access *and* refresh tokens have expired; the 401 from `/auth/refresh` itself is being caught and retried recursively into an infinite loop. Fix: detect the `/auth/refresh` URL in the response interceptor and redirect to login instead of retrying. Gotcha: without the recursion guard, a dead refresh token loops forever instead of logging out.

12. **(Curveball) Front-end on app.example.com, API on api.example.com — browser blocks the call. Debug.**
    DevTools → Network → look for a failed `OPTIONS` preflight or a missing `Access-Control-Allow-Origin`. Verify exact-origin match (no `*` with credentials), `withCredentials` + `Allow-Credentials` on both sides, the gateway's CORS path pattern, and no trailing-slash mismatch. Gotcha: the failing request is often the preflight, not the actual call — check OPTIONS first.

13. **Why configure CORS at the gateway instead of each service?**
    One consistent policy at the edge — security isn't scattered across teams who each mis-configure it, and downstream services stay focused on business logic. Gotcha: "security policy lives at the edge, not in every service" is the principle; per-service CORS guarantees drift.

14. **What does `@RestControllerAdvice` buy you?**
    Every exception maps to one consistent JSON shape (`timestamp/status/error/message/path`), so the front-end consumes all errors through one path — inline for validation (400), toast for system (5xx). Gotcha: it centralises error mapping so you never leak a raw stack trace to the client (downstream failures become a clean 502/503).

15. **(Curveball) Two users submit the same LLM prompt at the same instant — one job or two?**
    Two — idempotency is per-key/per-user, not per-content-globally, so different users get different jobs (separate billing/ownership). The same user double-clicking gets one. If cross-user dedup is wanted, a content-hash cache can return a shared result, but ownership/audit must still record both requests. Gotcha: don't conflate "same prompt" with "same job" — ownership matters.

---

## Day 5 — Full Mock Interview + Profiles + Target List

1. **Walk me through your approach before you code.**
   Restate the problem; confirm constraints (input size, duplicates, null/empty, no-solution return); name the pattern and the expected complexity; *then* code. Clarifying first is the senior signal — juniors start typing immediately. Gotcha: it's only ~30 seconds but it changes how the whole round is scored.

2. **Design a URL shortener.**
   Capacity → base-62 key length (7 chars); counter+base-62 (default, no collision check) vs hash+collision-check (deterministic per URL); redirect via Redis cache-aside; analytics as an async event off the hot path; 301 (cacheable, loses tracking) vs 302 (every hit reaches you). Gotcha: lead with capacity — it sizes every later choice. See [cheat-sheets/system-design.md](../../cheat-sheets/system-design.md).

3. **File storage — start with the upload flow.**
   Chunked pre-signed S3 URLs; bytes never flow through your servers; client → Upload Service → direct-to-S3 → complete → metadata `READY`. "Smart360's single-file pre-signed pattern, extended to per-chunk." Gotcha: the "no bytes through your servers" point is the whole reason for pre-signed URLs. See [projects-reference.md](../../projects-reference.md).

4. **Notification — how do you guarantee no duplicates?**
   Idempotency key (`event_id + user_id + channel`) via `SETNX` or a unique constraint; the second attempt is a no-op. Duplicate notifications are the #1 user complaint. Gotcha: the channel is part of the key — the same event legitimately reaches push and email.

5. **What's your one observability sentence for any design?**
   Golden signals (latency / traffic / errors / saturation), trace context propagated across services, and SLO burn-rate alerting so you page on user-facing degradation, not raw CPU. Gotcha: most candidates forget observability entirely — volunteering it as a closing sentence lands the round.

6. **Why should we believe your impact numbers?**
   Each ties to a mechanism: 60 s → 2–3 s came from eliminating 47 queries/load (N+1) via `JOIN FETCH` + composite indexes + Redis; 57% CI/CD from a GitLab DAG + BuildKit. The mechanism is the proof. Gotcha: a number without a mechanism reads as invented — always pair them. See [projects-reference.md](../../projects-reference.md).

7. **(Recruiter) What's your current CTC / expectation?**
   Deflect to role-and-fit first: "I'd rather focus on the role and total opportunity, then talk numbers once we both see a fit." If pressed, a researched band (₹18–24 LPA), never the current number; if a form forces a figure, full CTC (base + variable + PF + benefits). Gotcha: never anchor on your current salary — anchor on the researched band.

8. **(Recruiter) When can you join?**
   Immediately — no notice period. Lead with it; it's a concrete reason to move you up the queue. Gotcha: state it without hedging — it's a genuine differentiator in every recruiter pipeline.

9. **Why these target companies?**
   Java/Spring stack match (verified per JD), 2–5 YOE band, comp aligned to the level, and a tiered strategy — dream GCC/product (Tier-1), strong mid-tier (Tier-2), and calibration loops (Tier-3) run in parallel from day one. Gotcha: tiers encode strategy, not snobbery — run all three at once, never Tier-1-then-fallback.

10. **(Self-debrief framing) What would you do differently in that round?**
    Name a specific, fixable thing — "I buried the fan-out trade-off; next time I lead with it" — not a vague "be better." Self-awareness with a concrete fix is the signal. Gotcha: a real, narrow miss is more credible than claiming the round was flawless.

---

## Day 6 — Final Mock / Closeout + Referral Connections

1. **What's the single thing you'd most want another rep on before a loop?**
   Name it honestly and say how you closed it — self-aware gap-closing is itself a strong signal. Gotcha: don't claim "nothing" — that reads as low self-awareness; show the gap *and* the fix.

2. **Re-narrate your weakest HLD end-to-end.**
   Drive the full arc: requirements → capacity → architecture → 2–3 deep-dives → failure modes → trade-offs → one observability sentence — under 10 minutes, no notes. Gotcha: naming the rejected option in the trade-offs section is the senior move; don't skip it. See [cheat-sheets/system-design.md](../../cheat-sheets/system-design.md).

3. **Tell me about yourself.**
   90-second pitch: current role → quantified achievements (60 s → 2–3 s, 57% CI/CD, 30+ Bicep, 5-provider LLM proxy) → what you want → bridge to the company. Never a chronological CV recital. Gotcha: numbers in the first sentence, not the third. See [projects-reference.md](../../projects-reference.md).

4. **When can you join?**
   Immediately, no notice period — lead with it; it speeds you up in every queue. Gotcha: same as Day 5 — say it first and without qualification.

5. **(Recruiter) Walk me through your current CTC expectations.**
   Deflect to role/fit first; a researched band (₹18–24 LPA) if pressed; full CTC (base + variable + PF + benefits) only if a form forces a figure. Gotcha: never volunteer your current number — it caps the offer.

6. **Why a referral instead of applying cold?**
   A referred application is ~4–5× more likely to get a callback; a referred candidate is a known quantity with a vouched-for signal that moves them past the cold-resume pile. Gotcha: the referral is about *signal*, not favouritism — it's why it converts so much better.

7. **(Networking) Why are you reaching out to me specifically?**
   Reference a specific detail about their work — a talk, a repo, a shared technical decision — never a generic "I admire the company." Specificity earns the accept and, later, the referral. Gotcha: a copy-paste connect note gets ignored; the one concrete detail is the whole point.

8. **How do you make sure your application isn't filtered before a human sees it?**
   Mirror the JD's top ~5 keywords in the JD's exact phrasing on a per-application tailoring pass — ATS matches strings, not synonyms, and most sub-20-LPA roles filter by ATS first ("Spring Cloud Gateway," not "API gateway"). Gotcha: mirror only true matches — don't keyword-stuff or fabricate, the goal is passing the filter on real skills.

---

*[← Week 5](README.md)*
