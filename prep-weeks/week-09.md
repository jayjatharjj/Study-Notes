# Week 9 — Peak + Active Applications (Aug 10–16, 2026)

> The blade is sharp. Now swing it with precision. This week you shift from prep mode to execution mode: timed mixed DSA sets simulate real OA pressure, system design goes to true scale, and 10–15 quality applications go out the door. Referrals are the highest-leverage move you can make — one warm intro beats 50 cold applications. By Sunday you have run a full mock interview and have a live pipeline of companies actively in play.

---

## 🎯 Week Goal

1. Complete 6 timed DSA sets (2 problems / 45 min, 3 sessions this week) mixing all patterns learned — simulate real OA conditions with no hints, fresh editor, time pressure.
2. Design "a scalable file storage / data platform" at whiteboard depth: sharding, replication, CAP theorem, consistency models, partitioning — no hand-waving.
3. Submit 10–15 quality applications (curated, not spray-and-pray) with emphasis on warm referrals — minimum 5 LinkedIn outreach messages sent to engineers at target GCCs.
4. Complete one full mock interview (DSA + system design) on Pramp, interviewing.io, or with a peer by Saturday.

---

## ✅ By Sunday you can...

- Solve 2 LeetCode problems in 45 minutes under OA conditions, back-to-back, with correct time/space complexity stated.
- Implement from scratch: **LC 215 Kth Largest Element** (QuickSelect + heap), **LC 347 Top K Frequent Elements**, **LC 295 Find Median from Data Stream**, **LC 23 Merge K Sorted Lists** — explaining the heap invariant aloud as you type.
- Whiteboard the full "scalable file storage / data platform" design covering: chunked upload pipeline, metadata service, storage layer with sharding strategy, read path with CDN offload, replication (leader-follower), consistency trade-offs under CAP, and failure modes.
- Explain the difference between strong consistency and eventual consistency with a concrete example from your own work (PostgreSQL vs Redis cache invalidation in Smart360).
- Articulate the difference between horizontal sharding strategies (range, hash, directory/consistent hashing) and their respective failure modes.
- Explain leader-follower replication lag and how it affects reads — and how your application code should handle it.
- Recite your referral outreach message from memory and have sent at least 5 personalised versions.
- Explain your resume's top 3 metrics under real interview pressure — no hedging, no approximate numbers.

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday, Aug 10 — Heap Foundations + System Design: Sharding & Replication

**DSA — Timed Set #1 (45 min, OA simulation)**

Set the timer. No hints. Fresh LeetCode editor. Treat this as a real OA.

- [ ] **LC 215 – Kth Largest Element in an Array** (Medium). Two approaches:
  - *Min-heap of size k*: iterate array, push each element, pop if heap size > k. Final answer = heap top. O(n log k) time, O(k) space. This is the interview-preferred approach for streaming/unknown-size input.
  - *QuickSelect*: average O(n), worst O(n²) — know it as a follow-up. Partition array around pivot; recurse on the side containing the k-th position. The interviewer WILL ask "can you do better than O(n log k)?"
  - Know both. Implement heap first (more robust), mention QuickSelect when they push.
- [ ] **LC 347 – Top K Frequent Elements** (Medium).
  - Build a frequency map (HashMap). Push (frequency, element) into a min-heap of size k. O(n log k). Return heap elements.
  - Bucket sort O(n) follow-up: create buckets[0..n] where bucket[i] = list of elements with frequency i. Iterate buckets from high to low, collect until k elements. Know this but implement heap version first under pressure.
- [ ] After timer: review time/space for both. Write them down. Grade yourself: Did you get both working within 45 min?

**Core Block (30 min) — Sharding and Replication Deep Dive**

- [ ] Understand and be able to explain these three sharding strategies cold:
  - *Range sharding*: partition by key range (e.g., user IDs 0–1M on shard 1, 1M–2M on shard 2). Simple; bad: hotspots if data isn't uniformly distributed (e.g., most active users in range 0–100K).
  - *Hash sharding*: `shard = hash(key) % N`. Distributes evenly; bad: resharding when you add nodes requires rehashing all keys. Tie to your PostgreSQL knowledge.
  - *Consistent hashing*: place both nodes and keys on a ring; key maps to nearest node clockwise. Adding/removing a node only redistributes 1/N of keys. Used by Cassandra, DynamoDB, and Redis Cluster. This is the GCC-level answer.
- [ ] Understand leader-follower replication:
  - Leader accepts all writes. Followers replicate asynchronously (most common) or synchronously (rare, expensive).
  - Replication lag: between write to leader and propagation to follower. During lag, a read from follower returns stale data.
  - *Read-your-writes consistency*: after a user writes, their next read must see their own write. Solution: route the user's reads to the leader for a brief window after a write, OR use synchronous replication for critical reads.
  - *Monotonic reads*: once a user reads value V, they should never read an older value. Solution: hash user to a consistent replica for reads.
- [ ] Connect to your work: "In Smart360, caching S3 pre-signed URLs in Redis with a TTL is an eventual consistency choice — the cache may serve a slightly stale URL, which is acceptable because URLs remain valid for their full S3 expiry window. Strong consistency here (always fetch fresh) would negate the 80% S3 reduction."

**Self-Check**
- [ ] Can you implement the min-heap solution for Kth Largest without looking at any notes?
- [ ] Can you name the failure modes of range vs hash vs consistent hashing in one sentence each?

---

### Tuesday, Aug 11 — Heap Deep Cut + CAP Theorem & Consistency Models

**DSA — Timed Set #2 (45 min, OA simulation)**

- [ ] **LC 295 – Find Median from Data Stream** (Hard).
  - Two-heap approach: max-heap for the lower half, min-heap for the upper half. Invariant: lower half size == upper half size (or off by 1). Maintain invariant on each `addNum`. Median = top of max-heap (odd total) or average of both tops (even total).
  - Complexity: O(log n) per insert, O(1) for median. Space O(n).
  - Common mistake: forgetting to rebalance after every insertion. Talk through the rebalancing steps aloud even in a silent OA — write it as a comment.
- [ ] **LC 23 – Merge K Sorted Lists** (Hard).
  - Min-heap of (value, listNode). Initialize with the head of each list. Pop minimum, add to result, push that node's next if it exists. O(n log k) where n = total nodes, k = number of lists.
  - Know the naive O(nk) approach (merge two at a time sequentially) and why the heap is better.
  - Alternative: divide and conquer — merge lists pairwise, repeat. O(n log k) same complexity but different constant. Know this as a follow-up.
- [ ] Grade yourself again. Both mediums/hards in 45 min is the bar. If LC 295 took > 30 min alone, mark it for extra reps.

**Core Block (30 min) — CAP Theorem + Consistency Models**

- [ ] CAP Theorem — the precise statement (not the oversimplification):
  - In the presence of a **network partition** (P), a distributed system must choose between **Consistency** (C — every read returns the most recent write) and **Availability** (A — every request receives a non-error response, though possibly stale).
  - Without partition, you can have both. Since network partitions are a reality, the real choice is C vs A *during a partition*.
  - PostgreSQL (single-node or synchronous replication): CP — prioritises consistency; during a partition, the isolated replica stops accepting writes rather than serve stale data.
  - Redis (default config): AP-leaning — serves reads from potentially stale data rather than fail; prioritises availability.
  - Cassandra: AP by default (tunable via `QUORUM` / `ALL` read/write consistency levels to move toward C).
  - Tie to your work: "In Smart360, we used PostgreSQL as the source of truth (CP) and Redis as a performance cache (AP-leaning). The system explicitly tolerated eventual consistency in the cache layer — pre-signed URL TTLs were tuned to be safe in the eventual consistency window."
- [ ] Consistency models spectrum — know these four cold:
  - *Strong consistency*: every read returns the value of the most recent write. Expensive; requires coordination (leader reads or quorum). PostgreSQL with synchronous_commit=on.
  - *Read-your-writes consistency*: you always see your own writes. Weaker than strong but sufficient for most user-facing features.
  - *Eventual consistency*: given no new writes, all replicas converge to the same value eventually. DNS is the canonical example. Redis replication default. Your S3 URL cache.
  - *Causal consistency*: reads that follow a write (causally related) must reflect that write. Weaker than strong; stronger than eventual. Used in MongoDB sessions.
- [ ] PACELC model — the extension interviewers at GCCs love to test:
  - PACELC: even when there is no partition (E), there is a trade-off between Latency (L) and Consistency (C).
  - DynamoDB: PA/EL — available during partition, low latency (eventual) normally.
  - Spanner/CockroachDB: PC/EC — consistent during partition, consistent (but higher latency) normally.
  - Mentioning PACELC will immediately distinguish you as someone who has thought beyond CAP.

**Self-Check**
- [ ] Can you explain the two-heap invariant for Find Median out loud without touching your notes?
- [ ] Can you place PostgreSQL, Redis, Cassandra, and DynamoDB correctly on the CAP spectrum and justify each?

---

### Wednesday, Aug 12 — Timed Set #3 + System Design Mock: File Storage Platform

**DSA — Timed Set #3 (45 min, OA simulation)**

Mixed patterns — no pre-announcing which type. Pick two from the list below that you have NOT done this week. Timer starts now.

- [ ] Choose 2 from: **LC 767 Reorganize String** (Medium, greedy + heap), **LC 451 Sort Characters by Frequency** (Medium, heap), **LC 373 Find K Pairs with Smallest Sums** (Medium, heap), **LC 378 Kth Smallest Element in a Sorted Matrix** (Medium, binary search or heap), **LC 253 Meeting Rooms II** (Medium, heap / sweep line — prerequisite for many scheduling problems).
- [ ] After timer: for each problem, write:
  - Time complexity (best case and worst case if they differ)
  - Space complexity
  - The key insight in one sentence ("this is a min-heap of size k because...")
  - One follow-up question the interviewer might ask and your answer

**Core Block (60 min) — Full System Design: Scalable File Storage / Data Platform**

This is the peak session this week. Do it on paper or a whiteboard. No notes initially — design it from first principles, then check gaps.

**The prompt**: "Design a scalable file storage service — users can upload files of any size, share them with others, retrieve them quickly, and the system must handle 10 million users and 50 TB of data."

Work through it in this order:

- [ ] **Requirements clarification** (5 min — always do this):
  - What file sizes? Up to 5 GB (chunked upload needed).
  - Read-heavy or write-heavy? Read-heavy (1:10 write/read ratio typical).
  - Consistency requirement? Eventual is acceptable for file content; strong for metadata (did the file upload succeed?).
  - Global users or regional? Global → CDN offload.
  - Are files immutable after upload or editable? Immutable simplifies everything.

- [ ] **Capacity estimation** (3 min — show the interviewer you think quantitatively):
  - 10M users × avg 5 GB storage = 50 TB total.
  - Assume 100K daily uploads × avg 10 MB = 1 TB/day new data.
  - Reads: 10:1 ratio = 1M reads/day = ~12 reads/sec average (peak 100–200 RPS).
  - These numbers shape every architectural decision.

- [ ] **Upload pipeline** (draw it):
  1. Client requests an upload session from the **Upload Service** → gets a `fileId` and a list of pre-signed S3 URLs (one per chunk, 10 MB each).
  2. Client uploads each chunk directly to S3 using the pre-signed URLs (no traffic through your servers for data — this is critical at scale).
  3. Client notifies Upload Service that all chunks are uploaded.
  4. Upload Service triggers a **File Assembly Job** (async) that calls S3 `CompleteMultipartUpload`.
  5. Metadata Service records: `fileId`, `ownerId`, `fileName`, `size`, `S3 key`, `status=READY`.
  6. Upload Service publishes `FileUploaded` event to the message broker.

  **Connect to your work**: "This is the same pre-signed URL pattern I used in Smart360 — except there I generated per-file URLs; here I extend it to per-chunk URLs for large-file support."

- [ ] **Metadata Service design**:
  - PostgreSQL for metadata (strong consistency needed — did the upload succeed or not?).
  - Schema: `files(file_id, owner_id, name, s3_key, size_bytes, mime_type, status, created_at, updated_at)`, `file_permissions(file_id, user_id, role)`.
  - Sharding strategy: hash shard on `owner_id` — keeps all files of one user on the same shard, enabling efficient "list my files" queries without cross-shard joins.
  - Read replicas for read-heavy query load (list files, file info).

- [ ] **Storage layer**:
  - S3 (or Azure Blob Storage) for binary content. Redundancy built-in (11 nines durability).
  - S3 storage classes: hot data in S3 Standard; after 90 days → S3 Infrequent Access; after 1 year → S3 Glacier. Lifecycle policies automate this.
  - At 50 TB, you are NOT building your own distributed block storage. Interviewers respect this decision.

- [ ] **Read path + CDN**:
  - Download request hits the **Download Service** → validates permission (hit Metadata Service) → generates pre-signed S3 URL (TTL = 15 min) → returns URL to client.
  - For frequently accessed files: cache the pre-signed URL in Redis (TTL = 10 min, slightly under S3 expiry). This is literally the pattern from Smart360 — name it explicitly.
  - CDN (CloudFront/Azure Front Door) in front of S3 for public or shared files. Cache at edge, 95%+ cache hit rate for popular files.
  - Connect to your work: "In Deep Fathom I provisioned Azure Front Door via Bicep for exactly this CDN offload pattern. At scale it eliminates origin hits for repeated reads."

- [ ] **Replication and consistency**:
  - S3 is already replicated across AZs by default. For cross-region: S3 Cross-Region Replication.
  - PostgreSQL: leader-follower. Writes go to leader. Read replicas handle list/info queries.
  - Replication lag implication: after file upload success, if client immediately queries "list my files" from a replica that hasn't caught up yet, the new file won't appear. Solutions: (a) route the user's reads to the leader for 5 seconds post-upload, (b) use the event-sourced approach (Upload Service publishes the file directly to the response via the event).

- [ ] **Failure modes and mitigations** (draw the diagram of what breaks):
  - Upload Service crashes mid-upload → chunks are orphaned in S3. Mitigation: S3 lifecycle policy to delete incomplete multipart uploads after 24 hours; idempotent retry via the same `fileId`.
  - Metadata DB leader dies → promote replica; brief write unavailability is acceptable (CP trade-off). Time to detect + promote: 10–30 seconds with automated failover (RDS Multi-AZ, Azure PostgreSQL Flexible HA).
  - S3 bucket becomes regional single point → Cross-Region Replication as described.
  - CDN serves stale content after file deletion → CDN invalidation on delete event. Cost: CDN invalidation is a paid API call, so batch them.

- [ ] **APIs** (sketch them):
  ```
  POST /files/upload/initiate            → { fileId, uploadUrls: [ {chunkIndex, presignedUrl} ] }
  POST /files/upload/{fileId}/complete   → { fileId, status }
  GET  /files/{fileId}                   → { metadata }
  GET  /files/{fileId}/download          → { presignedUrl, expiresAt }
  DELETE /files/{fileId}                 → 204
  POST /files/{fileId}/share             → { shareLink, expiresAt }
  ```

- [ ] After designing: review your diagram against the requirements. What did you miss? Common gaps: virus scanning on upload (add a scan step before status=READY), quota enforcement (check owner's used storage before accepting upload), audit logging.

**Self-Check**
- [ ] Did you mention CAP trade-offs explicitly in the metadata vs storage layers?
- [ ] Did you connect at least two patterns to your actual resume work?
- [ ] Did you cover failure modes, not just the happy path?

---

### Thursday, Aug 13 — Applications Day #1 + Referral Outreach

**DSA (30 min — lighter day, keep the streak)**
- [ ] Solve **LC 692 – Top K Frequent Words** (Medium). Extension of LC 347 — but now with string comparison for tie-breaking. Use a min-heap with custom comparator: (frequency ASC, word DESC) so the heap root is always the "worst" of the top-k. When heap size > k, pop. O(n log k). This is a clean exercise in writing a custom comparator under pressure.
- [ ] Review the recurrence / invariant for all four heap problems this week (LC 215, 347, 295, 23) on paper in 10 minutes. If any is fuzzy, solve one example by hand.

**DSA — Design-a-Data-Structure mini-block (30 min)**

Design-DS rounds are high-frequency at product-tier companies and pair naturally with the heap/hashmap composition you've drilled all week — they test whether you can compose primitives (array + hashmap + heap) under an O(1)/O(log n) operation constraint, not just call a library. Do these two; narrate the operation complexity as you go.

- [ ] **LC 380 – Insert Delete GetRandom O(1)** (Medium). The trick is two structures working together: an `ArrayList<Integer>` for the values + a `HashMap<value, index>` mapping each value to its position in the list. `insert`: append to list, record index in map. `getRandom`: index the list at `random.nextInt(size)`. `remove` is the key insight — you cannot remove from the middle of an array in O(1), so **swap the target with the last element**, update the moved element's index in the map, then pop the last slot. All three operations O(1). Common mistake: forgetting to update the map index of the element you swapped into the hole.
- [ ] **LC 460 – LFU Cache** (Hard). This is the natural step-up from the LRU cache taught in **week 04** — LRU evicts by recency; LFU evicts by *frequency*, breaking ties by recency. Structure: (1) `HashMap<key, Node>` for O(1) lookup; (2) `HashMap<freq, LinkedHashSet<key>>` — the **frequency buckets**, where each bucket holds the keys at that exact frequency in insertion/recency order; (3) a **min-freq pointer** tracking the lowest frequency currently present. On `get`/`put`-hit: remove the key from its current freq bucket, bump it to the `freq+1` bucket; if the emptied bucket was at `minFreq`, increment `minFreq`. On `put` eviction: drop the least-recently-used key from the `minFreq` bucket (the `LinkedHashSet` preserves recency within the tie), and reset `minFreq = 1` on the new insert. All operations O(1). The min-freq pointer is what avoids scanning buckets to find the eviction candidate — state that aloud, it's the signal the interviewer wants.
- [ ] After both: write the operation complexity table (insert / delete / get / getRandom or evict) and the one-line "why this structure combination" for each. If LFU's bucket bookkeeping felt shaky, re-trace one eviction by hand on paper — it's the part candidates fumble live.

**Core Block (15 min) — Partitioning Strategies Review**
- [ ] Know the difference between *partitioning* and *sharding* (often used interchangeably, but precisely: partitioning = splitting data within one DB instance across tables/tablespaces; sharding = splitting across separate DB instances/servers).
- [ ] PostgreSQL table partitioning: `PARTITION BY RANGE (created_at)` for time-series data (logs, events). Each month = one partition. Queries with `WHERE created_at > X` only scan the relevant partition (partition pruning). Connect to your work: "For file metadata, partitioning by `created_at` month would allow dropping old partitions as data ages out."
- [ ] Consistent hashing deep cut: when a node is added, only the keys between the new node and its predecessor on the ring move — no global rehash. This is why Redis Cluster and Cassandra can rebalance incrementally. The number of virtual nodes per physical node (vnodes) controls load balance — more vnodes = better distribution but higher coordination overhead.

**Applications Block (75 min) — Quality Applications + Referrals**

*Target company research (20 min)*
- [ ] Open your tracking sheet. Identify 5 GCC targets and 5 product mid-tier targets (total 10). Criteria:
  - GCCs to consider: Citi Technology (Pune/Bengaluru), Morgan Stanley Technology (Mumbai/Bengaluru), Wells Fargo Technology, JPMC Software, Goldman Sachs Engineering, Deutsche Bank Technology, Barclays Technology, Visa Technology, Mastercard Technology, Cisco GCC, Walmart Global Tech, Target Tech, Salesforce India, SAP Labs, Adobe India, Oracle India, DE Shaw, Optum Technology (UHG), Fidelity India, HSBC Technology.
  - Product mid-tier: Razorpay, Zepto, Juspay, Groww, Setu, Navi, Slice (tech-first fintechs where Java Spring Boot is core), Postman, BrowserStack, Druva, Darwinbox.
  - Filter: Java/Spring Boot in JD, 2–5 YOE eligible, ₹18–30 LPA range, Pune/Bengaluru/remote.
- [ ] For each of the 10 companies: 5 minutes research. What does their engineering blog say? What tech stack is in the JD? What recent news (fundraise, product launch, layoffs)? Log notes in tracking sheet.

*LinkedIn Referral Outreach (30 min)*
- [ ] For each of your top 5 GCC/product targets: find 1–2 engineers (SDE2, Senior SDE, or Tech Lead) on LinkedIn who work there using: LinkedIn search → Company filter → "Java" OR "Spring Boot" in profile → 2nd-degree connections prioritized.
- [ ] Send 5 outreach messages using this template (personalise the [BRACKETS]):

  ```
  Hi [Name], I noticed you're a [role] at [Company] — I've been following 
  [Company]'s engineering work, particularly [specific thing: recent blog post / 
  product / tech talk]. I'm a Full Stack Java engineer (Java 17, Spring Boot, 
  Kubernetes, Azure) with 2.5 YOE looking to make a move into a product/GCC role. 
  I came across a [specific role] opening at [Company] and would love to get your 
  perspective on the team culture or, if you're open to it, a referral. Happy to 
  share my resume. No pressure either way — appreciate you reading this!
  ```

  Rules: one message per person, under 150 words, specific detail about their company (NOT "I've always wanted to work at..."), reference a specific open role. No resume in first message unless they ask.

- [ ] Log all 5 outreach attempts in tracking sheet: Company | Name | LinkedIn URL | Date Sent | Response.

*Apply directly (25 min)*
- [ ] Apply to 3–4 of the 10 researched companies where you don't have a referral contact yet, via LinkedIn Easy Apply or company careers portal. Tailor the cover note in 3 sentences: (1) specific relevant project, (2) tech match to JD, (3) what you want. Never send the exact same note twice.
- [ ] Total today: 5 referral messages + 3–4 direct applications = 8–9 rows in tracking sheet.

**Self-Check**
- [ ] Are all applications and outreach messages logged in the tracking sheet?
- [ ] Did every message include a specific personalised detail about that company?

---

### Friday, Aug 14 — Applications Day #2 + System Design: Partitioning Deep Cut

**DSA (30 min)**
- [ ] Solve **LC 1046 – Last Stone Weight** (Easy). Max-heap. Quick 10-min warm-up — it's an OA favourite for heap validation.
- [ ] Solve **LC 621 – Task Scheduler** (Medium). Greedy + heap pattern. Key insight: the idle time is determined by the most frequent task. Math formula approach: `result = max(n_tasks, (max_freq - 1) * (n + 1) + count_of_max_freq)`. Know both the greedy simulation (use max-heap, simulate cooldown with queue) and the math formula. Interviewers at mid-tier product companies love this one — it's a real scheduling problem.

**Core Block (45 min) — Partitioning + Full Architecture Story**

- [ ] Partitioning strategies — tie them to interview narratives:
  - *Horizontal partitioning (sharding by row)*: different rows on different DB instances. What you designed Wednesday. Scale-out write capacity.
  - *Vertical partitioning (by column)*: separate hot columns (frequently accessed) from cold columns (rarely accessed) into different tables or even DBs. Example: split `users.profile_json` (large, infrequently read) from `users.auth` (small, hit on every request). Reduces row width, improves cache efficiency for auth queries.
  - *Functional partitioning*: separate entire domains into separate data stores. Your microservices architecture already does this — Authorization service has its own DB, Data service has its own. This is the DDD-aligned approach.
- [ ] Review the full file storage / data platform design from Wednesday. Without looking at notes, re-draw the architecture from memory on paper. Time yourself: should take under 10 min if Wednesday's session landed.
- [ ] Gap analysis: what did you leave out on the first draw? Add it to the mental checklist.

**Applications Block (30 min)**
- [ ] Check for any LinkedIn message replies from Thursday. Respond within 2 hours of receipt — this is the highest-leverage activity.
- [ ] Apply to 4–5 more companies (target: 12–15 total applications across Thurs–Sat). Focus on any GCC targets you haven't hit yet.
- [ ] Naukri: update "Last Active" timestamp (just save the profile again) to bump yourself in recruiter searches. Check inbound recruiter messages and respond to any that match your criteria.
- [ ] Review tracking sheet. Any applications from 3+ days ago with no response? Draft a 1-sentence follow-up: "Hi [Name], I applied for [Role] on [Date] and wanted to confirm receipt and express my continued interest. Happy to answer any initial questions."

**Self-Check**
- [ ] Can you re-draw the file storage architecture in under 10 minutes from memory?
- [ ] Is your total application count at 10+ entering the weekend?

---

### Saturday, Aug 15 — Full Mock Interview Day

This is the most important day of the week. Treat it like a real interview. Get up, get dressed, sit at your desk.

**Preparation (30 min)**
- [ ] Find a mock partner on Pramp (pramp.com → schedule a "Data Structures" session) or interviewing.io. Book for mid-morning.
- [ ] If no partner available: use the self-mock protocol — phone camera recording, timer, no pausing.
- [ ] Review your talking points for the 90-second differentiator pitch and the Smart360/Deep Fathom stories. This is your last review before the mock — do NOT cram.

**DSA — OA Timed Set (45 min, pure simulation)**
- [ ] Pick any 2 problems you haven't solved this week. Recommended: **LC 355 Design Twitter** (Medium — heap + OOP design) or **LC 264 Ugly Number II** (Medium — heap/DP) + **LC 703 Kth Largest Element in a Stream** (Easy — heap, streaming variant of LC 215).
- [ ] Timer. No notes. Record complexity and explanation.

**LLD / OOD reframe of LC 355 — "now design the *classes*" (20 min)**

LC 355 as a DSA problem is just a heap merge over follow lists. But a product/GCC interviewer will pivot it into a **low-level design** round: "forget the heap trick — design the *classes* for this." This is your second LLD rep (the first is the parking-lot / OOD exercise in **week 03**). Do it on paper; the goal is clean responsibilities and defensible SOLID choices, not the algorithm.

- [ ] Sketch the class model:
  - `User` — id, profile; owns no feed logic (avoid the God-object trap).
  - `Tweet` — id, authorId, content, timestamp; immutable value object.
  - `FollowGraph` — `follow(followerId, followeeId)`, `unfollow(...)`, `followeesOf(userId)`. Encapsulates the social graph so feed code never touches the raw adjacency structure.
  - `FeedService` — `postTweet(userId, tweet)`, `getTimeline(userId, limit)`. Orchestrates; depends on `FollowGraph` and a `TimelineMerger` *via interfaces*, not concretes.
  - `TimelineMerger` (interface) — `merge(List<TweetStream> sources, int limit): List<Tweet>`. The heap-merge from the DSA version is just ONE implementation (`HeapTimelineMerger`); a `ChronologicalMerger` or a future `RankedMerger` can drop in without touching `FeedService`.
- [ ] **Defend the SOLID / extensibility choices aloud:**
  - *SRP*: `FollowGraph` owns relationships, `FeedService` owns orchestration, `TimelineMerger` owns ordering — each has one reason to change.
  - *Open/Closed + DIP*: `FeedService` depends on the `TimelineMerger` *interface*, so swapping chronological → ranked feed (the same candidate-generation-vs-ranking split from the Week 10 News Feed HLD) is a new class, not an edit to existing code.
  - *Strategy pattern*: `TimelineMerger` is a Strategy — name it; it's the high-signal move.
  - Mention the bridge to scale: "the in-memory `FollowGraph` is the LLD; at scale this becomes the fan-out-on-write vs fan-out-on-read decision — the interface boundaries stay identical, only the implementation moves behind them."
- [ ] Self-check: could a new joiner add a "muted accounts" filter or a ranked feed *without modifying* `FeedService`? If not, your boundaries are wrong — fix them.

**Full Mock Interview (90 min)**
- [ ] Round 1 — DSA (45 min): Partner gives you 2 problems. Narrate your thinking aloud. Ask clarifying questions before coding. State complexity before submitting.
- [ ] Round 2 — System Design (45 min): One of these prompts (have your partner choose):
  - "Design a scalable file storage service." (You've prepped this — it should feel familiar.)
  - "Design a real-time notification system for 10 million users."
  - "Design a URL shortener at scale."
  - Demonstrate: requirements clarification → capacity estimation → high-level design → deep dives on 2–3 components → failure modes → trade-offs.
- [ ] After mock: write down 3 things you did well and 3 specific areas to improve. Log them in the tracking sheet under a "Mock Feedback" tab.
- [ ] **Mock debrief — observability as a design pillar:** review your system-design round with one lens: *did you design the system's observability, or only its happy path?* At GCC level, observability is an SD **deliverable**, not a Spring Actuator ops afterthought. When you design a system, also design how you'd *operate* it:
  - **The four golden signals** — latency, traffic, errors, saturation. Name them in the design: "I'd expose P50/P95/P99 latency, request rate, error rate, and resource saturation per service." This is the SRE vocabulary interviewers listen for.
  - **Trace propagation across microservices** — a correlation/trace ID generated at the gateway and propagated on every downstream hop (W3C `traceparent` header / Micrometer Tracing), so one request is followable across all 5 services. Without it, debugging a distributed failure is guesswork.
  - **SLOs + alerting** — define the SLO ("99.9% of feed loads under 200ms") and alert on *symptoms* (SLO burn rate), not raw CPU. "If error-budget burn exceeds X, page; saturation trends, dashboard only."
  - Add to your design-round checklist: after failure modes, say one sentence on observability — "here's how I'd know this failure mode is happening in production." That single sentence separates a candidate who *designs* systems from one who only *draws* them.

**Applications Block (90 min)**
- [ ] Final push: get to 15 applications total if not there yet. Prioritise referral-assisted applications over cold applies.
- [ ] Follow up on Thursday's outreach messages if no reply yet. Keep it short.
- [ ] Research 2–3 companies where you want to apply next week (Week 10): read their engineering blogs, understand their stack, note open roles. Preparation for a better-quality application beats speed.

**Self-Check**
- [ ] Did you complete the full mock (DSA + system design)?
- [ ] Did you write down 3 improvements from the mock feedback?
- [ ] Is the application count at 13–15?

---

### Sunday, Aug 16 — Review, Consolidate, Week 10 Prep

**DSA (60 min) — Mixed Review + One Stretch Hard**
- [ ] Write on paper — recurrence or invariant (not code) for every heap problem this week: LC 215, 347, 295, 23, 692, 621. Time: 15 minutes. If any is fuzzy, re-solve it now.
- [ ] Solve **LC 632 – Smallest Range Covering Elements from K Lists** (Hard, stretch). Uses a min-heap of (value, listIndex, elementIndex) + tracking current max. This is Merge K Sorted Lists with a twist — the heap invariant is the same. If this takes > 30 min without cracking it, read the approach, understand it, don't grind. Understanding the pattern is the goal.
- [ ] Revisit the two timed-set problems from Saturday. Can you solve them again in half the time? Speed comes from internalising the pattern, not re-reading the solution.

**Core Block (60 min) — Mock Feedback + Architecture Consolidation**
- [ ] Take the 3 improvement areas from Saturday's mock feedback. For each one:
  - If DSA: solve 1 related problem targeting that exact gap.
  - If system design: write 3 bullet points explaining that component more precisely.
  - If behavioural: rewrite the STAR story for that question with more specific numbers.
- [ ] Re-narrate the file storage design out loud, from scratch, for 8–10 minutes. Target: cover requirements, capacity estimation, upload pipeline, metadata service (with sharding decision and justification), read path with CDN, replication strategy, CAP trade-off, and 2 failure modes. If you can do this fluently, you're interview-ready on this design.
- [ ] Add PACELC to your mental model: practice saying "The real question beyond CAP is PACELC — even without a partition, there's a latency vs. consistency trade-off in normal operation. PostgreSQL optimises for consistency even at latency cost; DynamoDB optimises for latency with eventual consistency by default."

**Planning (30 min)**
- [ ] Review tracking sheet: any recruiter responses? Schedule replies / calls immediately — response speed signals seriousness.
- [ ] Tally: total applications submitted, total referral messages sent, total responses received. Set Week 10 targets.
- [ ] Plan Week 10 focus:
  - DSA: Graphs (BFS/DFS, topological sort, shortest path) — LC 207 Course Schedule, LC 210, LC 417 Pacific Atlantic Water Flow, LC 200 Number of Islands (if not done), LC 127 Word Ladder.
  - System design: Design a real-time collaborative document editor (operational transforms / CRDT concepts) OR Design a distributed job scheduler.
  - Applications: Increase response rate — if < 10% response rate, review resume / outreach message quality. Consider Instahyre or Cutshort for more targeted outreach.
- [ ] Write 3 things you know today that you didn't know last Sunday.

---

## 🧠 Concepts to Master This Week

### Heap / Priority Queue — Four Essential Patterns

**Pattern 1: Top-K with Min-Heap**
- Use a min-heap of size k. For each new element: push it. If size > k, pop the minimum.
- At the end, the heap contains the k largest elements; the top is the k-th largest.
- Time: O(n log k). Space: O(k).
- Problems: LC 215, LC 347, LC 692.
- Intuition: the heap acts as a "bouncer" — only the k best get to stay.

**Pattern 2: Two-Heap for Streaming Median**
- Max-heap (lower half) + min-heap (upper half). Size invariant: `|lower| - |upper| <= 1`.
- Add number: push to max-heap; balance by popping max of lower half to upper half if needed; rebalance sizes.
- Median: if odd total → top of max-heap; if even → average of both tops.
- Time: O(log n) per insert, O(1) per query. Space: O(n).
- Problem: LC 295.

**Pattern 3: Merge K Sorted with Min-Heap**
- Initialize heap with the first element of each list (value, listRef).
- Each pop gives the global minimum. Push that node's next if exists.
- Time: O(n log k). Space: O(k).
- Problems: LC 23, LC 632 (stretch).

**Pattern 4: Lazy Deletion / Custom Comparator**
- When heap elements become invalid (e.g., stale cache entries), mark them invalid and skip on pop rather than removing — O(1) mark vs O(log n) remove.
- Custom comparator: in Java, `PriorityQueue<int[]>((a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1].compareTo(b[1]))`.
- Problems: LC 692 (tie-breaking by string), LC 373 (k smallest pairs).

### System Design — At-Scale Concepts

**CAP Theorem — The Precise Version**
- In the presence of a network partition, choose Consistency OR Availability.
- C = all nodes return the same data (most recent write). A = every node responds (possibly stale). P = partition tolerance (must have this in a distributed system).
- The real question: "How does your system behave DURING a partition?"
  - CP example: PostgreSQL synchronous_commit=on — the replica stops accepting reads if it falls behind during a partition (returns error), to avoid serving stale data.
  - AP example: Cassandra with ANY consistency level — serves stale data rather than return an error.

**PACELC — The Extension**
- PA/EL systems: AWS DynamoDB, Cassandra default — available during partition; optimise for low latency (eventual) normally.
- PC/EC systems: Google Spanner, CockroachDB — consistent during partition; accept higher latency normally for consistency.
- Most real-world systems are tunable (Cassandra's consistency levels, DynamoDB read modes). The interview answer: "It depends on the SLA for this specific read. For payment confirmation, I'd use a quorum read; for a 'recently active users' dashboard, eventual is fine."

**Sharding Decision Framework**
- Ask: (1) What is the most common query pattern? → This should determine the shard key (keep data needed for a query on the same shard). (2) Will the shard key cause hotspots? → Uniform distribution is a must. (3) How often do nodes join/leave? → If frequently, use consistent hashing; if rarely, hash mod N is fine.
- For file storage metadata: shard on `owner_id` → all files for a user on one shard → efficient "list my files."
- For time-series data (logs, events): range shard on `timestamp` → allows efficient range scans; risk of hot current-time shard → mitigate with write-ahead buffering or randomised time-bucket suffix.

**Leader-Follower Replication — Handling Lag**
- Replication lag = time between write commits on leader and propagation to follower. In PostgreSQL streaming replication: typically 10ms–2s depending on load and network.
- Three strategies for applications:
  1. *Read from leader for critical reads*: route all reads that require freshness to the leader. Cost: defeats the purpose of replicas for those queries.
  2. *Read-your-writes*: after a write, note the write timestamp; on subsequent reads, if replica WAL LSN < write LSN, route to leader. PostgreSQL clients expose LSN tracking.
  3. *Accept and handle staleness*: for truly non-critical reads (file list after upload), show optimistic UI (add the file to the list from the POST response) while the DB catches up. This is the most pragmatic pattern.

**Consistent Hashing — The Interview Depth Signal**
- Hash ring: both nodes and keys are hashed to [0, 2^32). Each key belongs to the first node clockwise on the ring.
- Adding a node: only the keys between the new node and its predecessor move → O(n/k) key moves, not O(n).
- Virtual nodes (vnodes): each physical node maps to V points on the ring (V typically 100–200). This ensures even distribution even with heterogeneous nodes.
- Real-world: Redis Cluster uses 16384 hash slots (not a pure ring, but same principle); Cassandra uses vnodes; Memcached uses consistent hashing directly.

---

## 🎤 Sample Interview Questions (incl. Curveballs) — 14 Qs + Pointers

### Heap / DSA

**1. "Implement Kth Largest Element. Now, what if the array is being streamed — you receive elements one at a time and must answer the query at any point?"**
- Pointer: The batch version (min-heap of size k, O(n log k)) naturally extends to the streaming version. LC 703 is exactly this. On each new element, push to the heap; if size > k, pop. Current answer = heap top. O(log k) per element, O(k) space. This is an LLM integration / real-time analytics use-case — tie it to monitoring dashboards.

**2. "Why is a min-heap used to find the k LARGEST elements — shouldn't it be a max-heap?"**
- Pointer: This trips up many candidates. A min-heap of size k keeps the k largest elements seen so far. The minimum of those k elements sits at the top. When a new element arrives: if it's larger than the heap top (the current k-th largest), it deserves to be in the top k, so pop the top and push the new element. A max-heap of size k would give you the k SMALLEST elements with the same logic. Make this reasoning explicit — it demonstrates deep understanding.

**3. "Curveball: In Find Median from Data Stream, what if 90% of the numbers are in the range [0, 100]? Can you do better?"**
- Pointer: Yes — bucket sort / counting sort. Maintain a count array of size 101 + "negative bucket" + "large bucket". Track total elements. Median query: iterate buckets until you find the middle element(s). O(1) amortized for integer data in a known range vs O(log n) for the heap approach. This is the extraordinary-candidate follow-up — you know when to abandon the general solution for a specialised one.

**4. "Walk me through Merge K Sorted Lists. What is the exact time complexity and where does the log k come from?"**
- Pointer: n = total number of nodes across all lists. k = number of lists. Each element is pushed and popped from the heap exactly once. Each heap operation is O(log k) because the heap contains at most k elements (one from each list). Total: O(n log k). The log k factor is the key insight — not log n, because the heap is bounded by k. If asked about space: O(k) for the heap (plus O(n) for the output).

### System Design

**5. "Design a scalable file storage service. Start with the upload flow."**
- Pointer: Lead with the chunked upload via pre-signed URLs. The key insight: "Data never flows through my application servers — clients upload directly to S3/Blob Storage using time-bounded pre-signed URLs. This eliminates a massive bandwidth bottleneck at my application tier." Draw the sequence: client → Upload Service (gets URLs) → S3 direct upload → Upload Service (completes). Mention that you did this in Smart360 for single files; here you extend it to chunks for files > 100 MB.

**6. "How do you handle consistency in your metadata service? What if two users upload a file with the same name simultaneously?"**
- Pointer: Name is not the primary key — `file_id` (UUID) is. Simultaneous uploads of same-named files are allowed by default; they get different `file_id`s. If the product requirement is "unique name per folder per user," add a unique constraint on `(owner_id, folder_id, name)` — the second insert will fail with a constraint violation, which the Upload Service catches and returns a 409 Conflict. PostgreSQL enforces this transactionally; no distributed lock needed within a shard.

**7. "Curveball: Your file storage service is working fine at 1 million users. At 10 million users, the metadata DB becomes the bottleneck. Walk me through how you'd shard it without downtime."**
- Pointer: This is a live migration question. Steps: (1) Choose the new shard key (`owner_id` hash). (2) Dual-write: new writes go to both the old DB and the new sharded setup. (3) Background migration: copy existing rows to the new sharded DB in batches (pg_dump / custom migration job). (4) Verify checksums / row counts per shard match. (5) Switch reads to the new sharded setup, then stop dual-writes to old DB. (6) Decommission old DB. The critical principle: dual-write first, verify second, cut over third. Never cut over without verification. Zero-downtime is achieved because no step requires taking the service down.

**8. "Walk me through CAP theorem. Give me a concrete example where you made a C vs A decision in your own work."**
- Pointer: "In Smart360, I explicitly chose availability over consistency for the pre-signed URL cache. Redis cached S3 URLs with a TTL slightly below S3's expiry. During a Redis node failure, reads fell back to regenerating the URL from S3 — so availability was maintained at the cost of occasional freshness. This was the right call because a slightly stale (but still valid) URL is indistinguishable from a fresh one to the user. If I needed strong consistency — say, immediately invalidating a URL after file deletion — I'd have used a cache-aside pattern with explicit invalidation events."

**9. "Curveball: I've heard CAP theorem is outdated — PACELC is better. What's your take?"**
- Pointer: "PACELC is a refinement of CAP, not a replacement. CAP focuses only on partition scenarios. PACELC makes the additional observation that even during normal operation (no partition), there's a latency vs consistency trade-off. A system like Google Spanner (PC/EC) accepts higher latency on every operation to provide global consistency — even without a partition. A system like DynamoDB (PA/EL) gives you low latency and eventual consistency during normal ops. The actionable implication is that you can't just say 'we're CP' — you also need to answer 'what's our latency budget for consistency in normal operation?' That's often the more relevant design question."

**10. "Curveball: How would you add virus scanning to your file storage service without adding latency to the upload path?"**
- Pointer: Async scanning. After the file upload completes, set file `status=PENDING_SCAN`. Publish a `FileUploaded` event. A Virus Scanner Worker consumes the event, pulls the file from S3, scans it (ClamAV or a cloud AV API). If clean, updates `status=READY`. If infected, updates `status=QUARANTINED` and triggers deletion + notification. The download endpoint checks status: if not READY, return 202 (still processing) or 403 (quarantined). Users see the file immediately after upload but can't download it until scanning completes. Typical scan time: < 5 seconds for most files. This is the real-world production pattern at Dropbox/Box.

### Java / Spring Boot

**11. "Curveball: You're using @Cacheable and @Transactional on the same service method. Explain the proxy wrapping order and what goes wrong if caching happens before the transaction commits."**
- Pointer: (From interview-qa.md, extend it.) Spring applies `@Cacheable` via a `CacheInterceptor` and `@Transactional` via a `TransactionInterceptor`. By default, both are applied as AOP proxies. The outer proxy runs first; if `@Cacheable` is outer, the cache is checked/populated BEFORE the transaction commits — meaning you could cache a value that gets rolled back. Fix: use `@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)` to populate cache only on commit, OR use `@CacheEvict` on write methods so the next read re-fetches after the transaction completes. You can also control proxy order with `@Order` on the aspect.

**12. "In your LLM proxy that routes across 5 providers, how did you implement provider-specific rate limiting without a distributed lock?"**
- Pointer: Redis sliding window counter. For each provider: `INCR provider:openai:window:{minute_bucket}; EXPIRE ...; GET ...`. If count exceeds the rate limit, skip to next provider. No distributed lock needed because Redis's single-threaded command execution makes INCR atomic. The routing logic: try providers in priority order (cost-ascending or capability-based); skip any that are over their rate limit; fallback to a queue if all are throttled. This avoids the thundering herd problem of multiple instances competing on locks.

**13. "Curveball: Your microservices are fine at 100 RPS. At 10,000 RPS, which component breaks first and why?"**
- Pointer: Think through each layer. API Gateway → horizontal scaling is straightforward (stateless, add instances). Authorization Service → JWT validation is CPU-bound (signature verification) but stateless; scales horizontally. Database → likely the first real bottleneck at 10K RPS if queries aren't indexed perfectly and connections aren't pooled correctly. Connection pool size: `HikariCP` default is 10 connections per instance; at 10K RPS you need many instances each with 10 connections. 20 instances × 10 connections = 200 DB connections — PostgreSQL default max_connections is 100, so you'd need to tune this or add PgBouncer as a connection pooler. Redis → single-threaded but handles ~100K ops/sec easily; not the bottleneck. The extraordinary answer names the connection pool as the first failure mode and the solution.

**14. "Curveball: Tell me something non-obvious about Kubernetes networking that most developers don't know."**
- Pointer: Pick one of these genuine depth signals:
  - "Kubernetes Service is not a load balancer — it's iptables rules (kube-proxy). Traffic goes directly from the sending pod to a selected backend pod via NAT rules, not through the Service IP as a proxy. The Service IP is a virtual IP that exists only in iptables." (iptables mode)
  - "When you delete a pod, there's a race condition: the pod is removed from the Endpoints object, but it takes a few seconds for kube-proxy to propagate the iptables update to all nodes. During that window, other pods can still route traffic to the terminating pod. This is why `preStop: sleep 5` is often added to pods — it gives kube-proxy time to drain routing before the pod stops accepting connections."
  - "Network Policies are not enforced by Kubernetes itself — enforcement requires a CNI plugin that supports them (Calico, Cilium, Weave). Without a compatible CNI, `NetworkPolicy` objects are silently ignored."

---

## 🌟 Extraordinary-Candidate Edge

### The Referral-First Mindset

Most candidates apply cold through job portals and wait. Extraordinary candidates use the referral channel first. A referred candidate is 4–5× more likely to be interviewed and has a higher offer rate. The mechanism: recruiter queues are prioritised by source; referred candidates skip the ATS filter.

**The specific week-9 move**: For each top-5 GCC target, spend 15 minutes finding the right person on LinkedIn (not HR — engineers, ideally SDE2+ or tech leads who know the team), send the template message, and follow up once after 5 days if no reply. Your hit rate will be 20–30% if your message is specific and non-generic. Three positive referrals in a week is better than 50 cold applications.

### Depth Signals to Drop Naturally This Week

- When discussing sharding: "I'd use consistent hashing rather than hash-mod-N so that when we add a node, only 1/k of the keys move rather than a full rehash. Redis Cluster implements this with 16,384 hash slots."
- When discussing CAP: "Beyond CAP, PACELC captures the latency vs consistency trade-off in normal operation — which is actually more relevant for day-to-day query design than the partition scenario."
- When discussing heap problems: "The min-heap-of-size-k pattern for top-K is O(n log k), not O(n log n) — the log k factor comes from the heap being bounded by k, not n. For very large n with small k, this is significantly better."
- When asked about PostgreSQL read replicas: "We'd need to handle replication lag carefully for reads-after-writes — specifically, tracking the WAL LSN of the write and routing the subsequent read to the leader if the replica hasn't caught up to that LSN yet."
- After a mock: proactively say "A follow-up question you might ask is [X] — the answer would be [Y]." This shows you understand the problem family and the interviewer's intent.

### What "Extraordinary" Looks Like in System Design Interviews

1. You ask 3–4 clarifying questions before drawing anything. You explicitly state your assumptions.
2. You give a capacity estimate before the high-level design. Even a rough "100K DAU, 10 MB avg file = 1 TB/day" shows quantitative thinking.
3. You design for failure first, then for the happy path — you say "the failure mode here is X, so I'm designing Y to mitigate it."
4. You make explicit trade-offs: "I'm choosing eventual consistency here because the user experience impact is negligible compared to the cost of synchronous cross-shard writes."
5. You connect at least one design decision to a real system you've built. This is your edge — most candidates can only talk about textbook designs.

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 after Sunday's review. Feed any 1–2 ratings directly into Week 10's Day 1–2 sessions.

| Area | Target | Your Rating | Gap / Next Action |
|---|---|---|---|
| LC 215 Kth Largest — both heap and QuickSelect | 5 | | |
| LC 347 Top K Frequent — heap + bucket sort follow-up | 5 | | |
| LC 295 Find Median — two-heap invariant, implement cold | 5 | | |
| LC 23 Merge K Sorted — heap, state complexity precisely | 5 | | |
| LC 621 Task Scheduler — greedy insight | 4 | | |
| Timed OA sets (3×) — both problems solved in 45 min | 5 | | |
| CAP theorem — precise definition + personal example | 5 | | |
| PACELC — can explain it and place DynamoDB/Spanner | 4 | | |
| Sharding strategies — range/hash/consistent hashing | 5 | | |
| Leader-follower replication lag + 3 mitigation strategies | 4 | | |
| File storage design — full whiteboard from memory (8 min) | 5 | | |
| Upload pipeline — pre-signed URL chunked flow | 5 | | |
| Metadata sharding decision justified | 5 | | |
| Failure modes — 3 named and mitigated | 5 | | |
| Full mock interview completed (DSA + system design) | 5 | | |
| Mock feedback written: 3 strengths + 3 improvements | 5 | | |
| 10–15 applications submitted and logged | 5 | | |
| 5 referral outreach messages sent | 5 | | |
| Referral template memorised | 4 | | |
| 90-second differentiator pitch — fluent | 5 | | |

**Scoring guide:**
- 5: Can do it cold, under pressure, on a whiteboard.
- 4: Solid but one edge case might cause a hesitation.
- 3: Concept understood, can't execute under pressure — needs more reps.
- 2: Shaky — carry into Week 10 Day 1.
- 1: Not comfortable — block 2 hours in Week 10 specifically for this.

**Week 9 exit criteria (minimum bar before Week 10):**
- All 4 core heap problems solved at least twice (once this week + once in Sunday review).
- Full mock interview completed with written feedback.
- At least 10 applications submitted and tracked.
- At least 5 referral messages sent.
- File storage design narrated fluently from memory in under 10 minutes.

**If you missed the bar:** Carry the gap forward as a named task in Week 10. Track it in the sheet under "Carry-Forward" — visibility kills avoidance.

---

*Week 9 complete → Week 10: Graphs (BFS/DFS topological sort, shortest path) + System Design (real-time collaborative editor or distributed job scheduler) + interview scheduling acceleration (phone screens should be starting from referrals sent this week).*
