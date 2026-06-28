# Cheat-Sheet — System Design (HLD + LLD)

> Last-minute revision. Lead with the answer, justify with trade-offs, tie every decision to your own work (Smart360 / Deep Fathom / WebX). End on a number.

---

## 1. The HLD Framework (run this order, every time)

| # | Step | One line to say |
|---|---|---|
| 1 | **Clarify requirements** | Pin **functional** (features) + **non-functional** (scale, latency p99, availability, consistency, read/write ratio). "Let me confirm scope before designing." |
| 2 | **Back-of-envelope estimation** | QPS, storage, bandwidth — "let me size it so the design is justified by numbers, not vibes." |
| 3 | **API design** | The contract first — a few endpoints with request/response shapes; it forces the data flow. |
| 4 | **Data model** | Core entities + the **shard key** + SQL-vs-NoSQL choice driven by the access pattern. |
| 5 | **High-level diagram** | Client → LB → service(s) → cache → DB → queue/CDN. Draw the happy path end-to-end. |
| 6 | **Deep-dive one component** | Pick the interesting one (the hot path / the fan-out) and go deep — that's where seniority shows. |
| 7 | **Scale / bottlenecks** | Find the bottleneck (hot key, write amplification, single DB), then cache / shard / replicate / queue it. |
| 8 | **Trade-offs** | Name what you gave up (consistency for latency, write-amp for read speed). No design is free. |

**Discipline:** lead with the answer → justify → state the trade-off. Narrate aloud.

---

## 2. Estimation Numbers

**Latency ladder (orders of magnitude — know the gaps, not exact ns):**

| Operation | Time | Note |
|---|---|---|
| L1 cache ref | ~1 ns | |
| L2 cache ref | ~4 ns | |
| RAM ref | ~100 ns | 100× slower than L1 |
| SSD random read | ~16–100 µs | |
| Network round trip (same DC) | ~0.5 ms | |
| Disk (HDD) seek | ~10 ms | |
| Cross-region round trip | ~50–150 ms | the WAN tax — avoid on the hot path |

**Reflex:** RAM ≈ 100 ns, same-DC RTT ≈ 0.5 ms, cross-region ≈ 100 ms. Memory beats disk by ~1000×; that's why caching wins.

**Time / throughput:**
- **Seconds/day ≈ 86,400 ≈ 86.4K** (memorize). Month ≈ 2.5M s.
- **QPS = daily requests ÷ 86,400.** (1M/day ≈ 12/s; 100M/day ≈ 1160/s.)
- **Peak ≈ 2–10× average** — state a peak factor.

**Storage:** `items × size × retention × replication`.
- e.g. 100M URLs/mo × 5 yr × 500 B ≈ 6B rows × 500 B ≈ **3 TB** (× replication factor 3 ≈ 9 TB).

**Habit:** always ask the **read:write ratio** first. Read-heavy (100:1 → URL shortener; ~10:1 → file storage) ⇒ cache + read replicas. Write-heavy (chat) ⇒ wide-column + partitioning.

---

## 3. Building Blocks (one-liners)

**Load balancing**
- **L4 (transport):** routes by IP/port, no payload inspection — fast, protocol-agnostic.
- **L7 (application):** routes by URL/header/cookie — path-based routing, TLS termination, sticky sessions; smarter, slightly costlier.
- Algorithms: round-robin, least-connections, consistent-hash (sticky).

**Caching**
- **Cache-aside (lazy):** app checks cache → miss → load DB → populate. Default. (URL shortener redirect path.)
- **Write-through:** write cache + DB synchronously — consistent, slower writes.
- **Write-back (write-behind):** write cache, flush to DB async — fast, risk loss on crash.
- **Eviction:** LRU (recency), LFU (frequency + recency tie-break), TTL.
- **Stampede / thundering herd:** many misses hit DB at once on expiry → mitigate with request coalescing (single-flight), staggered/jittered TTL, or lock-on-recompute.

**CDN** — cache static/shared content at edge POPs near users; offloads origin + cuts latency. Invalidate on change (batch the calls). *(Deep Fathom: Azure Front Door fronting S3.)*

**DB indexing** — B-tree index turns O(n) scan into O(log n) lookup; index the columns your hot query filters/sorts by. Composite index order matters (leftmost-prefix). Cost: slower writes + storage. *(Smart360: composite `(tenant_id, created_at)`, `(user_id, status)`.)*

**SQL vs NoSQL**

| Family | Example | Use when |
|---|---|---|
| **Relational (SQL)** | PostgreSQL, MySQL | strong consistency, transactions, joins, structured schema (metadata, money) |
| **Key-value** | Redis, DynamoDB | O(1) lookups, cache, sessions, counters |
| **Wide-column** | Cassandra, HBase | write-heavy, append + range-scan by partition (chat messages, time-series) |
| **Document** | MongoDB | flexible/nested schema, per-document access |
| **Graph** | Neo4j | relationship-heavy traversals (social graph) |

Default: **SQL until a specific access pattern forces NoSQL.** *(Smart360/WebX kept Postgres as the CP source of truth.)*

**Replication (leader-follower / primary-replica)** — one leader takes writes; followers replicate the log + serve reads (scales reads, gives failover).
- **Async** (common): leader acks before followers apply → fast, but **stale reads** + possible loss on failover.
- **Sync** (rare): leader waits for follower ack → no loss, but every write pays slowest follower (real systems use **semi-sync** — one of N).
- **Replication lag** = window between leader commit and follower apply (Postgres streaming ~10 ms–2 s). Causes "I uploaded but it's not in my list."
- **Read-your-writes mitigations:** (1) read critical queries from leader; (2) **LSN compare** — route to leader if replica's applied LSN < write's LSN; (3) accept staleness + optimistic UI (most pragmatic). Add **monotonic reads** (pin user → one replica) so time never goes backwards.

**Sharding / partitioning** — split one logical dataset across N nodes; **shard key choice is the whole game** (shard on the field your hottest query filters by → single-shard query, no scatter-gather).

| Strategy | Rule | Failure mode |
|---|---|---|
| **Range** | key ranges → shards | hotspots if traffic skewed (newest-on-last-shard) |
| **Hash (mod N)** | `hash(key) % N` | adding a node rehashes ~all keys |
| **Consistent hashing** | nodes + keys on a ring; key → next node clockwise | uneven load without vnodes |

- **Consistent hashing:** add/remove a node moves only **~1/N of keys** (no global rehash). Give each node **~100–200 virtual nodes (vnodes)** to smooth load + spread a dead node's load across many neighbors. Used by Cassandra, DynamoDB, Redis Cluster (16,384 fixed slots).
- Pain: cross-shard joins (scatter-gather) + distributed transactions (Saga/2PC). *(Smart360 metadata sharded on `owner_id` → "list my files" is single-shard.)*

**CAP / PACELC**
- **CAP:** on a network **P**artition, choose **C** (reject/block to stay correct) or **A** (serve, risk stale). You can't drop P — partitions happen.
- **PACELC:** if **P**artition → A vs C; **E**lse (normal) → **L**atency vs **C**onsistency. The honest extension: even with no partition you trade latency for consistency.
- *(Postgres = CP source of truth; Redis cache = AP-leaning. The pre-signed-URL TTL in Redis is a deliberate eventual-consistency choice.)*

**Consistency models** — **Strong** (read sees latest write) → **Read-your-writes** → **Monotonic reads** → **Eventual** (replicas converge eventually). Pick the weakest that's still correct for the use case.

**Message queues / Kafka** — decouple producer/consumer; absorb bursts; enable async.
- **Partition** = ordered append-only log; ordering guaranteed **within** a partition only (key → partition).
- **Consumer group** = consumers splitting partitions for parallelism; one partition → one consumer in a group.
- **Offset** = consumer's read position (committed → resume after crash).
- **Delivery guarantees:** at-most-once / **at-least-once** (default, may dup) / exactly-once (costly). Pair at-least-once + idempotency for *effectively-once*.

**Idempotency** — same request applied twice = same result. Use an **idempotency key** (`event_id + user_id + channel`) enforced via SETNX / unique constraint → retries/dupes are no-ops. Essential under at-least-once. *(WebX: provider calls keyed by request id → no double-bill on re-run.)*

**Outbox** — write the domain change AND the event to one DB table **in the same transaction**; a poller / CDC (Debezium) publishes from the outbox → event guaranteed atomic with the change, no lost events. *(Deep Fathom: `PostCreated`/notification decoupling.)*

**Saga** — replace distributed transactions with a chain of local txns, each emitting an event; a failed step triggers **compensating transactions** to undo.
- **Choreography:** services emit/react to events — decentralized, can get tangled.
- **Orchestration:** a central orchestrator drives steps — easier to reason about, central dependency.
- Prefer over 2PC (2PC is slow + couples services).

**Rate limiting**
- **Token bucket:** tokens refill at rate R, cap B; request takes a token → allows bursts up to B. Most common.
- **Leaky bucket:** queue drains at fixed rate → smooths bursts into a steady stream.
- **Sliding window:** count requests in a rolling window (Redis) → accurate, no fixed-window edge spikes. *(Notification per-user limit; WebX per-provider INCR/EXPIRE.)*

**API gateway** — single entry point: auth, rate-limit, routing, TLS termination, request aggregation. *(Smart360: JWT validated at the gateway.)*

**Circuit breaker / bulkhead** (Resilience4j)
- **Circuit breaker:** CLOSED → (failures exceed threshold) → OPEN (fail fast, call fallback) → HALF-OPEN (trial) → CLOSED. Stops cascading failures.
- **Bulkhead:** isolate resources (thread pools / connection pools) per dependency so one slow dep can't drown the rest.
- *(WebX failure story: added retry + backoff + fallback provider + circuit breaker after a provider outage.)*

**Observability**
- **Golden signals (RED/USE):** **Latency, Traffic, Errors, Saturation.** Alert on these.
- **Three pillars:** metrics (aggregate), logs (events), **distributed tracing** (trace/span IDs propagated across services to follow one request end-to-end).

---

## 4. HLD One-Pagers

### URL Shortener
- **Requirements:** shorten/redirect, optional alias/expiry/analytics. Read-heavy **~100:1**, redirect **< 100 ms p99**, HA, no collisions.
- **Estimation:** ~40 writes/s (~400 peak), ~4000 reads/s; ~6B rows / 5 yr ≈ 3 TB; **base-62 7-char** key (62⁷ ≈ 3.5T).
- **Key decisions:** **counter + base-62** (no collision check, default) vs hash + collision-check. **Cache-aside Redis** (`code→longUrl`) on hot path. Analytics **off the hot path** via async event (Kafka). **301 vs 302:** 302 default (every click hits you → analytics + editable); 301 cached by browser (fastest, loses analytics).
- **Data model:** `url_mapping(short_code PK, long_url, owner_id, created_at, expires_at, click_count)`, sharded by `short_code`.
- **Scale points:** Redis hot-key cache absorbs 100:1 skew; async analytics consumer; trivially shardable storage.

### News Feed / Timeline
- **Requirements:** post, follow, ranked/chrono feed; read-heavy; low-latency feed.
- **Key decision — fan-out:**

| | Fan-out-on-write (push) | Fan-out-on-read (pull) |
|---|---|---|
| On post | write into every follower's feed | store once |
| On read | O(1) read precomputed feed | merge followees' posts at read |
| Cost | 1M-follower post = 1M writes | every read is a fan-in merge |
| Best for | normal users | celebrities |

- **The answer = hybrid:** push for normal users, **pull for celebrities** above a tunable follower threshold; merge celeb posts in at read time. Caps write amplification, keeps reads fast for the 99%.
- **Data model:** per-user **Redis sorted set** (`post_id → score`), capped ~800; inactive feeds built lazily on login. **Cursor pagination** `(score, post_id)`, never `OFFSET/LIMIT` (unstable under writes).
- **Scale points:** separate **candidate generation** (fan-out) from **ranking** (swappable scorer). *(Fan-out-on-write = outbox/Kafka pattern from Deep Fathom: `PostCreated` → Kafka → feed-builder workers.)*

### Chat / Messaging
- **Requirements:** 1:1 + group, real-time, presence, delivery/read receipts, ordering, durable history. Write-heavy, millions of concurrent connections.
- **Key decisions:** **WebSocket** (bidirectional; SSE is server→client only). Gateway maps **`userId → (server, connectionId)` in Redis** so any backend finds the recipient's socket; **heartbeats** clean dead mappings. **Presence** via TTL keys (`SETEX presence:userId 30s`; expiry = offline; eventual consistency fine).
- **Ordering:** per-conversation **monotonic sequence number** (immune to sender clock skew), not wall-clock.
- **Receipts:** SENT → DELIVERED → READ over the same socket.
- **Group fan-out:** small groups push, large channels pull (same hybrid as News Feed).
- **Data model:** **wide-column (Cassandra)** partitioned by `conversation_id`, clustered by `message_seq` → efficient "last N / paginate older."
- **Scale points:** offline recipient → persist, deliver on reconnect; node death → clients reconnect + re-register.

### Notification System
- **Requirements:** multi-channel (push/email/SMS/in-app), per-user preferences, **no duplicates** (#1 complaint), at-least-once + retries, delivery tracking.
- **Key decisions:** **`Channel` interface** per channel (adding WhatsApp = new impl, Open/Closed). **Template Service** (`templateId, locale, vars`) for i18n. **Preferences checked before dispatch at category granularity** (muted marketing still gets security alerts). Per-user **rate limit** (Redis sliding window).
- **Dedup:** idempotency key `event_id + user_id + channel` via SETNX.
- **Reliability:** queue-backed per-channel workers, **exponential-backoff retry**, **DLQ** on exhaustion (never silently dropped). State machine `QUEUED → SENT → DELIVERED → READ / FAILED` via provider webhooks.
- **Scale points:** isolated per-channel workers (SMS outage ≠ email outage); **outbox + at-least-once + idempotency = effectively exactly-once.**

### Typeahead / Autocomplete
- **Requirements:** brutally read-heavy, **< 100 ms p99**, ranked by **popularity** (not alphabetical), eventually-fresh.
- **Key decisions:** **sharded trie / FST partitioned by prefix**; each node stores **precomputed top-k** → query = walk-to-prefix-then-read (never request-time DFS). Client **debounce** (~150–300 ms). Hot prefixes in Redis.
- **Ranking:** historical query frequency ("fa" → "facebook", not "fabaceae").
- **Offline pipeline:** query logs → stream/batch aggregation → per-prefix counts → rebuild trie → ship via **versioned blob + atomic pointer flip** (reads never see a half-built index).
- **Scale points:** precompute trades build-time/storage for O(prefix-len) reads; versioned swap trades slight staleness for race-free updates.

### Rate Limiter
- **Requirements:** allow N req / window per key (user/IP/API key); distributed; low overhead; correct under concurrency.
- **Key decisions:** algorithm — **token bucket** (bursts), **leaky bucket** (smooth), **sliding window** (accurate). Store counters in **Redis** (atomic INCR/EXPIRE or Lua for atomicity); key = `userId:route:window`.
- **Scale points:** centralized Redis = single source of truth but a dependency (mitigate with local fallback / approximate); return `429` + `Retry-After`. *(WebX per-provider rate-limit headroom via Redis INCR/EXPIRE.)*

### File Storage / Dropbox
- **Requirements:** files up to 5 GB → **chunked (multipart) upload**; read-heavy ~10:1; **strong consistency for metadata, eventual for content**; global → CDN.
- **Estimation:** 10M users × 5 GB = 50 TB; 100K uploads/day × 10 MB ≈ 1 TB/day growth → justifies S3 + CDN over a fileserver.
- **Key decisions:** **pre-signed S3 URLs per chunk** → client uploads **directly to S3** (no bytes through your servers — app tier never the bandwidth bottleneck). Complete → async assembly (`CompleteMultipartUpload`) → metadata `READY`. Download = validate permission → pre-signed URL (TTL 15 min) cached in **Redis (TTL 10 min, inside the 15-min expiry)**; CDN for shared files.
- **Data model:** **PostgreSQL** metadata sharded on **`owner_id`** ("list my files" single-shard), read replicas; `file_metadata(file_id PK, owner_id, name, size, s3_key, status, ...)`, optional unique `(owner_id, folder_id, name)`.
- **Scale points:** replication lag on post-upload list → read leader / optimistic UI; orphaned chunks → S3 lifecycle rule (delete incomplete after 24 h); CDN stale on delete → invalidation event. *(Smart360 pre-signed pattern extended per-chunk; Deep Fathom Front Door CDN offload.)*

---

## 5. LLD / OOD

**Approach:** entities → responsibilities (one reason to change each) → **interfaces at the extension seams** → apply SOLID → inject dependencies (don't construct them inside).

**Litmus test:** "Could a new joiner add feature X without modifying existing classes?" If they must crack open the orchestrator, your seams are wrong.

**SOLID (one-liners):**
- **S — Single Responsibility:** one class, one reason to change.
- **O — Open/Closed:** open to extension, closed to modification (add a new impl, don't edit existing).
- **L — Liskov Substitution:** a subtype must be usable anywhere its base type is, without surprises.
- **I — Interface Segregation:** many small focused interfaces > one fat one; clients don't depend on methods they don't use.
- **D — Dependency Inversion:** depend on abstractions, not concretions (inject the interface).

**Anti-patterns to name:** **God class** (holds everything → five reasons to change); **leaky encapsulation** (returning internal `Map`/`Set` so callers mutate it — expose intent-named methods instead).

### Parking Lot (compact example)
- **Entities:** `Vehicle` (abstract; `Car`/`Bike`/`Truck` subtypes), `ParkingSpot` (with `SpotType` enum), `ParkingFloor`, `ParkingLot`, `Ticket` (immutable value object: id, spot, entry time), `Gate`.
- **Strategies (extension seams):**
  - `SpotAllocationStrategy` interface → `NearestFirst` / `BestFit` impls (Open/Closed — add a new policy without touching the lot).
  - `PricingStrategy` interface → `HourlyPricing` / `FlatRate` impls (Strategy pattern).
- **Flow:** `ParkingLot.park(vehicle)` → allocation strategy picks the nearest fitting free spot → mark occupied → issue `Ticket`. `unpark(ticket)` → free spot → pricing strategy computes fee from duration.
- **SOLID hits:** `ParkingSpot` knows only its own state (SRP); new vehicle/spot type = new enum + subtype, no edits (OCP); `ParkingLot` depends on the strategy interfaces, not concretes (DIP); pricing/allocation are swappable.

```java
interface PricingStrategy { double fee(Ticket t, Instant exit); }
interface SpotAllocationStrategy { Optional<ParkingSpot> allocate(List<ParkingFloor> floors, Vehicle v); }

class ParkingLot {                         // depends on abstractions (DIP)
    private final List<ParkingFloor> floors;
    private final SpotAllocationStrategy allocator;
    private final PricingStrategy pricing;
    ParkingLot(List<ParkingFloor> f, SpotAllocationStrategy a, PricingStrategy p) {
        this.floors = f; this.allocator = a; this.pricing = p;   // injected
    }
    Ticket park(Vehicle v) {
        ParkingSpot spot = allocator.allocate(floors, v)
            .orElseThrow(() -> new IllegalStateException("lot full"));
        spot.occupy(v);
        return new Ticket(spot, Instant.now());
    }
    double unpark(Ticket t) {
        t.spot().free();
        return pricing.fee(t, Instant.now());
    }
}
```

**Bridge to HLD:** LLD and HLD are the same design at two zoom levels — e.g. Design Twitter's in-memory `FollowGraph` + `TimelineMerger` interface becomes the fan-out-on-write/read decision at scale, but the interface boundaries stay identical.
