# Week 5 · Day 1 — Mon Jul 27 — DSA Maintenance + URL Shortener / File Storage HLD + STAR #1–#5

> Week 5 flips the mode. The blade is sharp; the work now is *keeping* it sharp while you build fluency for the things an interviewer actually scores — system designs you can drive cold, and STAR stories that land in under two minutes with a number in every answer. DSA shrinks to **maintenance**: one timed mixed set under OA conditions, just enough to keep the muscle warm. The weight shifts to **Block B** — today, walking the URL shortener and scalable file storage end-to-end (requirements → capacity → API → data model → scale → trade-offs) — and **Block C**, where you write and rehearse the first five STAR stories until each carries your real metrics (96% latency, 57% CI/CD, 80% S3) without hesitation. The discipline this week: *lead with the answer, tie every decision to your own work, end every behavioral answer on a number.*

📌 **Study today:** DSA maintenance — one timed mixed set (OA simulation) · system-design mock: URL shortener + scalable file storage (driven end-to-end) · STAR stories #1–#5 — write + speak aloud, timed · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA Maintenance: Timed Mixed Set #1 (OA simulation)

Maintenance, not learning. The goal is *repeatable* fluency: prove you can sit down cold and finish two mixed-pattern mediums in 45 minutes with correct complexity stated. Don't pre-announce the pattern to yourself — part of the skill is recognising it.

### Protocol (treat it as a real OA)

1. **Set a 45-minute timer.** Fresh editor, no hints, no autocomplete crutches, no peeking at notes.
2. **Pick 2 across patterns** — one DP/greedy from Week 4, one heap/graph. Suggested pairing today:
   - **LC 322 Coin Change** (DP — unbounded knapsack) *or* **LC 56 Merge Intervals** (greedy/sorting).
   - **LC 215 Kth Largest** (heap / QuickSelect) *or* **LC 207 Course Schedule** (graph — cycle detection / topo sort).
3. **Clarify aloud before coding** — input ranges, duplicates, empty input, return shape. Two sentences, even alone.
4. **Narrate while coding**, state complexity *before* you "submit."
5. **After the timer**, for each problem write: time/space (best and worst if they differ), the one-line key insight, and one follow-up the interviewer might ask.

### Quick re-derivation cheats (glance only if stuck >5 min)

- **Coin Change (322):** `dp[a] = min over coins c of dp[a-c] + 1`; base `dp[0]=0`; unreachable = ∞. O(amount × coins) time, O(amount) space. Follow-up: *count the ways* (LC 518) flips the loop order (coins outer, amounts inner) to avoid permutation double-counting.
- **Merge Intervals (56):** sort by start; if `cur.start ≤ last.end`, merge (`last.end = max(last.end, cur.end)`), else append. O(n log n). Follow-up: *insert interval* into an already-sorted list in O(n).
- **Kth Largest (215):** size-k **min**-heap → root is the k-th largest, O(n log k), streaming-friendly. Follow-up: QuickSelect for expected O(n) in-memory.
- **Course Schedule (207):** build adjacency + indegree; Kahn's BFS — if you can't dequeue all nodes, there's a cycle → impossible. O(V+E). Follow-up: *return the order* (LC 210) is the same algorithm, collecting the dequeue sequence.

### Self-grade rubric

| Signal | Green | Yellow | Red |
|---|---|---|---|
| Both finished | ≤ 45 min | 45–60 min | unfinished |
| Complexity stated | unprompted, correct | correct after prompt | wrong/missing |
| Bugs | compiled + passed mentally | one off-by-one | logic broken |

Anything red → mark it for an extra rep this week; this is maintenance, so a single weak rep is the signal, not a crisis.

---

## Block B — System-Design Mock: URL Shortener + Scalable File Storage

This is the heart of the day. Drive each design **end-to-end** — requirements → capacity → API → data model → scale → trade-offs — out loud, as if a panel is watching. Lead with the answer, then justify.

### URL Shortener (driven, ~20 min)

**1) Requirements.** Functional: shorten a long URL → short code; redirect short → long; optional custom alias; optional expiry; click analytics. Non-functional: **read-heavy (~100:1 read/write)**, redirect latency **< 100 ms p99**, high availability (a dead shortener breaks every embedded link), short codes never collide.

**2) Capacity estimation.** Say the numbers aloud:
- Writes: 100M new URLs/month ≈ **~40 writes/s** avg, ~400/s peak.
- Reads: 100× → **~4 000 reads/s** avg, tens of thousands at peak.
- Storage: 100M/month × 5 yr ≈ **6B URLs**; ~500 B/row → **~3 TB**. Trivially shardable.
- **Key length via base-62** (`[A-Za-z0-9]`): 62⁶ ≈ 56B, 62⁷ ≈ 3.5T. **7 chars** covers the 5-year horizon with headroom.

**3) API.**
```
POST /api/v1/shorten     { longUrl, customAlias?, expiresAt? } → 201 { shortUrl }
GET  /{shortCode}        → 301/302 redirect to longUrl
GET  /api/v1/{code}/stats → { clicks, createdAt, ... }
```

**4) Data model.** Single table, sharded by `short_code`:
```
url_mapping( short_code PK, long_url, owner_id, created_at, expires_at, click_count )
```

**5) Short-code generation — two strategies, state the trade-off:**
- **Counter + base-62:** a distributed counter (e.g. a ticket server handing out ranges, or Redis `INCR` blocks) → encode to base-62. Guarantees uniqueness, no collision check, sequential-but-opaque. Best default.
- **Hash + collision check:** `base62(md5(longUrl))[:7]`; on the rare collision, re-salt and retry. Lets the same URL map deterministically, but needs a uniqueness check on write.

**6) Redirect path (the hot path).** Cache-aside: `GET /{code}` → look up Redis (`code → longUrl`); hit → redirect; miss → DB → populate Redis (TTL) → redirect. With 100:1 read skew and a hot-key cache, almost every redirect is served from memory in single-digit ms.

**7) Analytics — keep it off the hot path.** Don't write to the DB synchronously on every redirect. Fire an async event (Kafka / queue) → a consumer aggregates click counts. The redirect returns immediately; analytics is eventually consistent.

**8) The signature trade-off — 301 vs 302:**
- **301 (Permanent):** browsers and proxies **cache** it → subsequent clicks never reach your server → fastest, cheapest, but you **lose click tracking** and can't change the target.
- **302 (Found/Temporary):** every click hits your server → full analytics and editable targets → more load.
- "I'd default to **302** when analytics matter (the usual product requirement); **301** only when the link is immutable and analytics aren't needed."

### Scalable File Storage (driven, ~25 min)

The interviewer's favourite because it touches uploads, metadata, CDN, and replication lag in one design. Tie it explicitly to **Smart360** (pre-signed URLs) and **Deep Fathom** (Front Door / CDN).

**1) Requirements + capacity.**
- Files up to **5 GB** → **chunked (multipart) upload**. Read-heavy (~1:10 write/read). **Eventual consistency for content, strong consistency for metadata.** Global users → **CDN**.
- 10M users × 5 GB = **50 TB** baseline; 100K uploads/day × 10 MB = **~1 TB/day** growth. State these out loud — the numbers justify S3 + CDN over a single fileserver.

**2) API.**
```
POST /files            { name, size, mimeType } → { fileId, [ {partNo, presignedUrl} ... ] }
PUT  <presignedUrl>    (client → S3 directly, per chunk)
POST /files/{id}/complete  { parts:[{partNo, etag}] } → 202 (async assembly)
GET  /files/{id}       → 302 to a pre-signed download URL
GET  /files?owner=me   → list (cursor-paginated)
```

**3) Upload pipeline (drive it).**
- Client → **Upload Service**: gets a `fileId` + a **per-chunk pre-signed S3 URL**.
- Client uploads chunks **directly to S3** — **no file bytes ever flow through your servers** (this is the whole point; your app tier doesn't become the bandwidth bottleneck).
- Client notifies completion → async **assembly** (`CompleteMultipartUpload`) → **Metadata Service** sets `status=READY` → publishes a `FileUploaded` event.
- *"Same pre-signed-URL pattern I used in Smart360, extended to per-chunk for large files."*

**4) Metadata Service + data model.** **PostgreSQL** (strong consistency for metadata), **sharded on `owner_id`** so "list my files" is a single-shard query (no scatter-gather), with **read replicas** for the read-heavy listing path.
```
file_metadata( file_id PK, owner_id (shard key), name, size, mime, s3_key,
               status, created_at, folder_id )
  -- unique (owner_id, folder_id, name) if names must be unique per folder
```

**5) Read path + CDN.** Download Service **validates permission** → issues a **pre-signed URL (TTL 15 min)** → caches it in **Redis (TTL 10 min, safely inside the 15-min expiry)**. Put **CloudFront / Azure Front Door** in front of S3 for shared/public files. *"Front Door via Bicep in Deep Fathom did exactly this CDN offload."*

**6) Replication + failure modes.**
- **S3** is cross-AZ by default; add **cross-region replication** for DR.
- **PostgreSQL** leader-follower → **replication lag** bites "list my files" right after upload. Mitigation: route **post-upload reads to the leader**, *or* render the file optimistically from the POST response (optimistic UI) while the replica catches up.
- **Orphaned chunks** (incomplete uploads) → an **S3 lifecycle rule** deletes incomplete multipart uploads after 24 h.
- **Metadata leader dies** → promote a replica (CP choice — correctness over availability for metadata).
- **CDN stale after a delete** → fire a CDN **invalidation on the delete event** (batch the calls to stay within rate limits).
- **Common gaps to name proactively:** virus scanning (async scan before `READY`), per-user **quota enforcement**, and **audit logging** of access.

**7) Trade-offs to state.** Direct-to-S3 upload (scales bandwidth, but the client needs pre-signed URLs and your assembly is async) · strong metadata + eventual content (correctness where it matters, cheap where it doesn't) · `owner_id` sharding (fast "my files," but cross-user admin queries become scatter-gather).

---

## Block C — STAR Stories #1–#5 (write + rehearse)

Write each story out, then **speak it aloud, timed**. Structure every one as **S → T → A → R** (Situation, Task, Action, Result), and make sure the **R is a number**. Every sentence should carry a specific detail — vague answers read as "watched, didn't own."

### #1 — Performance win (the flagship, 96%)

- **S:** Smart360 dashboards were loading in **~60 seconds** — users were abandoning the page.
- **T:** I owned bringing load time to something usable, with **zero schema changes** allowed (live multi-tenant DB).
- **A:** Turned on **Hibernate statistics** → found **47 queries per page load** — a classic **N+1**. Fixed with `JOIN FETCH` + `@EntityGraph`; added **composite indexes** `(tenant_id, created_at)` and `(user_id, status)`; layered a **Redis read cache (5-min TTL)**; and cached the **S3 pre-signed URLs** (TTL = expiry − 30s buffer), which cut **S3 calls by ~80%**.
- **R:** **60 s → 2–3 s — a 96% reduction**, zero schema changes, and S3 calls down ~80%.
- *Target: 90 seconds spoken, "96%" said naturally.*

### #2 — Conflict / disagreement

- **S:** A real design disagreement — e.g. **stateless JWT vs server sessions** (or async event-driven vs sync REST, or Bicep vs the existing Terraform).
- **T:** Reach a decision without damaging the relationship or stalling the project.
- **A:** Documented **both options with explicit trade-offs**, proposed a **time-boxed PoC / decision matrix**, and acknowledged the **kernel of validity** in their concern (e.g. session revocation is genuinely simpler). Reached consensus on the data; where we couldn't, escalated cleanly with the matrix.
- **R:** Chosen approach shipped, **relationship intact**, and a quantified outcome (e.g. the JWT path cut sign-in latency ~60% — see Smart360). *Disagree on the idea, never the person.*

### #3 — Ownership / leadership

- **S:** CI/CD pipeline took **~23 minutes**; everyone complained, nobody owned it.
- **T:** Self-assigned the fix — **no one asked me to.**
- **A:** Profiled with the **GitLab pipeline trace** → found **3 bottlenecks** (serial stages, no layer caching, redundant builds). Restructured into a **DAG** with parallel stages and enabled **BuildKit** layer caching.
- **R:** **23 → 10 minutes — a 57% cut**, multiplied across every developer, every day. *Proactive identification, measured impact.*

### #4 — Biggest failure

- **S:** Shipped the **LLM-proxy routing without retry + fallback**, assuming provider uptime.
- **T:** It was my call and my code.
- **A:** A provider had a partial outage → **jobs failed in production**. I owned it immediately, added **retry with exponential backoff + a fallback provider + a circuit breaker**, and made **"design for failure first"** a checklist item before sign-off.
- **R:** Incident resolved, recurrence eliminated; the lesson — *assume every dependency will fail* — now shapes how I design. *Honest consequence, specific lesson, follow-through. Under 90 s.*

### #5 — Why leaving

- **Frame on ambition, never negativity:** "I've had strong ownership and impact in my current role, and I'm looking for **larger scale, a stronger engineering culture, and compensation that reflects the level** I'm operating at. I want to own a domain end-to-end at a company building at scale."
- Never criticise the current employer. **Speak it 3× until it sounds natural, not rehearsed.**

> **Rehearsal rule:** record yourself. If you say "basically," "kind of," or trail off without a number, do it again. The number is the proof of ownership.

---

## 💻 Practice coding questions

A **lighter maintenance set** — pick the two timed ones for Block A, keep the rest as warm-ups / re-derivation reps.

1. **Coin Change (LC 322)** — unbounded-knapsack DP; state `dp[a]=min(dp[a-c]+1)`; then the LC 518 *count-ways* follow-up.
2. **Merge Intervals (LC 56)** — sort by start, merge overlaps; then *insert interval* (LC 57).
3. **Kth Largest Element (LC 215)** — size-k min-heap; state the QuickSelect follow-up.
4. **Course Schedule (LC 207)** — Kahn's topo sort for cycle detection; then LC 210 *return the order*.
5. **Two Sum (LC 1)** — hashmap one-pass; a 5-minute warm-up to start the timer relaxed.
6. **Valid Parentheses (LC 20)** — stack; another fast confidence rep.
7. **(Stretch) Number of Islands (LC 200)** — grid BFS/DFS flood fill; restate complexity O(rows×cols).
8. **(Stretch) Climbing Stairs (LC 70)** — the 1D-DP "hello world"; re-derive state + transition aloud.

---

## 🎤 Interview questions

1. **Design a URL shortener — start with capacity.** → ~40 writes/s, ~4 000 reads/s, 6B rows over 5 yr → **7-char base-62** keys; counter+base-62 (default) vs hash+collision-check.
2. **301 vs 302 for the redirect — which and why?** → 301 is browser-cached (fastest, but loses analytics and can't change target); 302 hits your server every time (full analytics, editable). Default 302 when analytics matter.
3. **How do you keep the redirect under 100 ms?** → Cache-aside in Redis (`code→longUrl`); with 100:1 read skew almost every redirect is a memory hit; analytics fired as an async event, never inline.
4. **File storage — walk the upload flow.** → Client gets `fileId` + per-chunk pre-signed S3 URLs → uploads chunks **directly to S3** (no bytes through your servers) → complete → async assembly → metadata `READY`. Smart360 single-file pattern extended to chunks.
5. **Why shard file metadata on `owner_id`?** → Co-locates a user's files on one shard so "list my files" is single-shard, not scatter-gather. Trade-off: cross-user admin queries fan out.
6. **(Curveball) Two users upload a file with the same name simultaneously.** → Name isn't the PK; `file_id` UUID is. For "unique per folder," a `(owner_id, folder_id, name)` constraint → 409 on the loser, enforced within a shard.
7. **You uploaded a file but it's missing from "my files." Why, and fix?** → Read replica lag — the list read hit a follower behind the leader. Fix: route post-upload reads to the leader, or optimistic UI from the POST response.
8. **How do you prevent orphaned chunks from incomplete uploads?** → An S3 lifecycle rule deletes incomplete multipart uploads after 24 h; metadata stays `PENDING` until assembly succeeds.
9. **Why pre-signed URLs at all — why not proxy the bytes?** → Offloads bandwidth to S3, so your app tier never becomes the throughput bottleneck; the client talks straight to object storage with a short-lived, scoped credential.
10. **Tell me about a performance problem you solved.** → Smart360 60 s → 2–3 s (96%): Hibernate stats → 47-query N+1 → JOIN FETCH/EntityGraph + composite indexes + Redis + S3 URL cache (80% fewer S3 calls), zero schema changes.
11. **Tell me about a disagreement with a colleague.** → Documented both options with trade-offs, proposed a time-boxed PoC/decision matrix, found the kernel of validity in their view, reached consensus or escalated cleanly — relationship intact, quantified outcome.
12. **Tell me about something you owned end-to-end that no one asked you to.** → The slow CI/CD: GitLab trace → 3 bottlenecks → DAG + BuildKit → 23 → 10 min (57%), compounding across the whole team daily.
13. **What's your biggest failure?** → Shipped LLM-proxy routing without retry/fallback, assumed provider uptime → production incident; added retry+backoff+fallback+circuit breaker; "design for failure first" is now a sign-off gate.
14. **Why are you leaving?** → Ambition framing: scale, engineering culture, comp reflecting the level — never negativity about the current employer.
15. **(Behavioral curveball) What would your current manager say is your weakness?** → Pick a real, improving one (e.g. "I used to over-engineer for hypothetical scale; I now size the solution to the actual load and document why") — and show the concrete change.
16. **Why a size-k *min*-heap for the k *largest*?** → It keeps the k best seen; its root (smallest of those) is the k-th largest; O(n log k) and streaming-friendly, vs a max-heap of all n.
17. **(DSA) Coin Change — why does loop order matter in the count-ways variant?** → Coins-outer/amounts-inner counts *combinations*; the reverse counts *permutations* (double-counts orderings). For min-coins, order doesn't change correctness.

---

## ✅ Self-check

1. Did **both** timed Block-A problems finish within 45 minutes, with correct complexity stated unprompted?
2. Can you drive the **URL shortener** end-to-end (capacity → base-62 key length → cache-aside redirect → 301/302 trade-off) in under 8 minutes from memory?
3. Can you walk the **file-storage upload pipeline** (pre-signed chunks → direct-to-S3 → async assembly → metadata READY) without notes, and name three failure modes?
4. Can you tell the **performance win** story under 2 minutes with "**96%**" and "**80% fewer S3 calls**" spoken naturally?
5. Are **STAR #1–#5** all written *and* spoken aloud at least once, each ending on a number or concrete outcome?

---

<div align="center">

← [Week 4](../week-04/) · [Week 5](README.md) · [02-tue-jul-28.md](02-tue-jul-28.md) →

</div>
