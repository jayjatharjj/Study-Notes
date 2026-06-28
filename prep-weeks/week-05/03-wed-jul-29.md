# Week 5 · Day 3 — Wed Jul 29 — DSA Maintenance + Chat / Typeahead HLD + Project Deep-Dives

> Real-time systems are where "I've built a CRUD app" and "I understand distributed state" diverge. **Chat** forces you to reason about millions of long-lived connections, message ordering immune to clock skew, the delivery/read-receipt state machine, and write-heavy storage — and the celebrity-vs-normal fan-out split you already know from the News Feed reappears for small vs large groups. **Typeahead** is the opposite shape: brutally read-heavy, sub-100 ms, served from a precomputed sharded trie, never a request-time search. Then Block C closes the loop you've been building all week — the **three project deep-dives** (Smart360, Deep Fathom, WebX) narrated as architecture *stories* with trade-offs, each landing in time, each curveball handled. By the end of today, your five HLDs and three talk-tracks should feel like recitation, not recall.

📌 **Study today:** DSA maintenance — timed mixed set #3 + one design-DS (LRU/LFU) · system-design mock: chat/messaging + typeahead (driven end-to-end) · project deep-dive rehearsals — Smart360 / Deep Fathom / WebX as narrated architecture stories · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA Maintenance: Timed Mixed Set #3 + LRU/LFU

Same OA protocol (45-min timer, fresh editor, complexity stated). Today **include one design-DS** — the single most-asked "design a data structure" question. **Know LC 146 (LRU) cold.**

### Today's pairing

- **LRU Cache (LC 146)** — doubly-linked list + HashMap, all ops O(1). The one to know cold.
- One more mixed problem rotating a still-shaky pattern (a DP, graph, or interval question from earlier weeks).

### LRU Cache — the structure to recite

A `HashMap<key, Node>` for O(1) lookup + a **doubly-linked list** ordered most-recent → least-recent. `get` moves the node to the front; `put` inserts/updates at the front and **evicts the tail** when over capacity. The doubly-linked list is what makes both "move to front" and "remove from tail" O(1).

```java
class LRUCache {
    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0), tail = new Node(0, 0); // sentinels

    LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail; tail.prev = head; // empty list between sentinels
    }
    public int get(int key) {
        Node n = map.get(key);
        if (n == null) return -1;
        remove(n); addFront(n);            // touch → most recent
        return n.val;
    }
    public void put(int key, int val) {
        if (map.containsKey(key)) { Node n = map.get(key); n.val = val; remove(n); addFront(n); return; }
        if (map.size() == capacity) {      // evict LRU (the node before tail)
            Node lru = tail.prev; remove(lru); map.remove(lru.key);
        }
        Node n = new Node(key, val); addFront(n); map.put(key, n);
    }
    private void remove(Node n) { n.prev.next = n.next; n.next.prev = n.prev; }
    private void addFront(Node n) { n.next = head.next; n.next.prev = n; head.next = n; n.prev = head; }
    static class Node { int key, val; Node prev, next; Node(int k, int v){ key=k; val=v; } }
}
```

**Talking point:** "Sentinel head/tail nodes remove every null-check on the edges — insert and remove are unconditional pointer swaps." Contrast with **LFU (LC 460)**: LFU evicts by *frequency* (recency as tie-break) and needs frequency buckets + a min-freq pointer — a step harder, but the same composition discipline.

---

## Block B — System-Design Mock: Chat + Typeahead

### Chat / Messaging (driven, ~25 min)

**1) Requirements.** 1:1 and group messaging; **real-time delivery**; **presence** (online/last-seen); **delivery + read receipts**; message **ordering**; durable history. Non-functional: low latency, millions of concurrent connections, write-heavy.

**2) WebSocket connection management.** A **persistent bidirectional socket per client** — contrast **SSE**, which is server→client only; chat needs client→server too, so WebSocket is the right tool (SSE is for the read-only LLM token stream — see WebX). Millions of long-lived connections need a **gateway layer** that maps **`userId → (server, connectionId)` in Redis** so *any* backend can locate a recipient's socket. **Reconnects + heartbeats (ping/pong)** detect dead sockets and clean up stale mappings.

**3) Presence.** Online/offline/last-seen via **heartbeat TTL keys** — `SETEX presence:userId 30s` on each ping; key expiry = offline. **Fan out presence changes to contacts**; **eventual consistency is fine** (a few seconds' stale "last seen" hurts no one).

**4) Message ordering.** A **per-conversation monotonic sequence number** — *not* wall-clock. Clients order by sequence, which is **immune to sender clock skew** (two phones with skewed clocks would otherwise reorder messages). The sequence is assigned server-side per conversation.

**5) Delivery / read receipts.** A small state machine: **SENT → DELIVERED → READ**, flowing back to the sender as **lightweight status messages** over the same socket.

**6) Group fan-out — the hybrid returns.** **Small groups → fan-out-on-write** (write the message into each member's inbox). **Large channels → fan-out-on-read** (store once, members pull) — the **same hybrid as the News Feed**, for the same reason: cap write amplification on huge groups.

**7) Storage.** **Wide-column store (Cassandra / HBase)**, **partitioned by `conversation_id`**, **clustered by `message_seq`** → efficient "fetch last N / paginate older" at write-heavy scale. **TTL / archival tiering** moves old messages to cold storage. Wide-column over a relational store because the access pattern is "append + range-scan by conversation," not joins.

**8) Failure modes + trade-offs.** A backend node dies → its sockets drop → clients reconnect through the gateway and re-register in Redis. A message is sent but the recipient is offline → it's persisted; delivered on reconnect (receipts advance then). Trade-off: per-conversation sequencing gives clean ordering but means the sequence generator is a per-conversation coordination point (cheap, since conversations partition naturally).

### Typeahead / Autocomplete (driven, ~20 min)

**1) Requirements.** Brutally **read-heavy**, **< 100 ms p99**, suggestions ranked by **popularity** (not lexicographic), eventually-fresh (new trends can lag minutes).

**2) Serving path.** Client (**debounced**, ~150–300 ms) → edge → **sharded suggestion index** (a **trie / FST sharded by prefix**). Each node stores its **precomputed top-k** completions, so a query is **walk-to-the-prefix-then-read** — *never* a request-time DFS over the subtree (that would blow the latency budget).

**3) Ranking.** By **historical query frequency**, not alphabetical — "fa" should suggest "facebook," not "fabaceae." Frequency comes from the offline pipeline.

**4) Offline pipeline.** Query logs → **stream/batch aggregation** → per-prefix counts → **build/refresh the trie** → ship to the serving shards via a **versioned blob swap** (an **atomic pointer flip** to the new index, so reads never see a half-built trie). **Cache hot prefixes in Redis** in front of the shards.

**5) Trade-offs.** Precomputed top-k per node trades extra build-time + storage for O(prefix-length) reads — exactly right when reads ≫ writes and latency is the constraint. Versioned blob swap trades a little staleness (the index refreshes periodically, not per query) for atomic, race-free updates.

---

## Block C — Project Deep-Dives: Smart360 / Deep Fathom / WebX

Rehearse each talk-track **aloud, recorded, until tight.** Each is a *story* — context → your contributions with numbers → trade-offs — and you must field the standard curveballs. No "basically," no filler, a number in every contribution.

### Smart360 (~5 min) — multi-tenant SaaS, Java 17 / Spring Boot / Vue

Three contributions, each a mini-arc:

- **Performance:** N+1 (47 queries/load, found via Hibernate statistics) → `JOIN FETCH` / `@EntityGraph` + composite indexes `(tenant_id, created_at)`, `(user_id, status)` + **Redis read cache** (5-min TTL) + **S3 pre-signed-URL cache** (TTL = expiry − buffer, **80% fewer S3 calls**) → **60 s → 2–3 s (96%)**, zero schema changes.
- **Security:** migrated **server sessions → stateless JWT** validated **at the gateway** → **~60% sign-in latency cut**; layered **3-level RBAC**.
- **Architecture:** **strangler-fig** migration of user-management — carved the **notification boundary first** → **~30% efficiency gain** — extracting a service without a big-bang rewrite.

**Curveballs (pre-load the answers):**
- *JWT filter testing?* → **WireMock** to stub the auth provider + `@WebMvcTest` / `MockMvc` to assert the filter rejects bad tokens and passes good ones.
- *Hardest bug?* → **`LazyInitializationException`** — accessing a lazy association outside the transaction/session; fixed with `@Transactional(readOnly=true)` on the read path and `@EntityGraph` to fetch eagerly where needed.
- *How does multi-tenancy actually isolate data?* → **tenant id in the JWT** → **Hibernate `@Filter`** auto-applies the tenant predicate → **Postgres Row-Level Security (RLS)** as a database-level backstop so a code bug can't leak cross-tenant.

### Deep Fathom (~5 min) — FedRAMP-aligned Azure platform

- **Infra as code:** **30+ Azure resources in modular Bicep** — Container Apps, **PostgreSQL Flexible**, Redis, **Key Vault**, **Front Door**, **private endpoints** — with **one parameter file per environment** (dev/stage/prod from the same modules).
- **CI/CD:** restructured into a **DAG** with **BuildKit** caching → **23 → 10 min (57%)**.
- **Reliability:** **Kubernetes liveness/readiness probes + graceful shutdown** (drain in-flight requests on SIGTERM); **Key Vault accessed via managed identity** (no secrets in config); **RLS across 50+ tables**.

**Curveball:**
- *Key Vault is unreachable at startup — what happens?* → **Deploy fails safe** (a pod that can't fetch its secrets at boot won't serve traffic — better than running misconfigured). But **runtime secrets are cached in memory**, so an outage *after* startup doesn't take running pods down. Name both cases — that's the senior answer.

### WebX (~4 min) — LLM orchestration, 5 providers, 2–20 min jobs

- **Async contract:** `POST /jobs` → **202 Accepted + jobId** (an LLM job can run 20 minutes; a synchronous HTTP request would time out).
- **State + queue:** **PostgreSQL** for **durable job state** (a crash leaves a `PENDING`/`PROCESSING` row for recovery) + **Redis Streams** as the work queue.
- **Worker pool:** routes each job to a provider by **cost / capability / rate-limit headroom**, tracked with **Redis atomic INCR/EXPIRE** per provider; results streamed back via **SSE / polling**; **idempotency keys** dedupe duplicate submissions.

**Curveball:**
- *A worker pod is killed mid-PROCESSING — does the job vanish or double-bill?* → Neither. Workers **heartbeat every 30 s**; a **`@Scheduled` watchdog marks a job STALE after ~2 min** of no heartbeat and **re-queues** it; because provider calls are **idempotent (request id)**, the re-run **doesn't double-bill**.

> **The arc to tie them together (say this if asked "walk me through your projects"):** *"Smart360 taught me performance engineering, Deep Fathom gave me infra ownership, WebX pushed me into async + LLM integration — now I want to own a domain end-to-end at scale."*

---

## 💻 Practice coding questions

Lighter maintenance set — two timed for Block A (one being the design-DS), the rest as reps.

1. **LRU Cache (LC 146)** — HashMap + doubly-linked list with sentinels; all ops O(1). **Know cold.**
2. **LFU Cache (LC 460)** — kv map + frequency buckets + min-freq pointer; contrast with LRU's recency-only eviction.
3. **Insert Delete GetRandom O(1) (LC 380)** — array + index map, swap-with-last delete.
4. **Implement Trie (LC 208)** — directly relevant to typeahead; insert/search/startsWith.
5. **Design HashMap (LC 706)** — buckets + chaining; a quick composition warm-up.
6. **Kth Largest in a Stream (LC 703)** — size-k min-heap maintained across `add` calls.
7. **(Stretch) Design Add and Search Words (LC 211)** — trie with `.` wildcard DFS; trie reps for typeahead.
8. **(Stretch) Min Stack (LC 155)** — auxiliary min stack; O(1) min — another clean design-DS rep.

---

## 🎤 Interview questions

1. **Chat — WebSocket or SSE, and why?** → WebSocket: chat needs bidirectional (client→server too); SSE is server→client only and suits the read-only LLM token stream, not chat.
2. **How do you route a message to a user across many backend servers?** → A gateway maps `userId → (server, connectionId)` in Redis so any backend can find the recipient's live socket; heartbeats clean up dead mappings.
3. **Why order messages by sequence number, not timestamp?** → A per-conversation monotonic sequence is immune to sender clock skew; wall-clock from skewed devices can reorder messages.
4. **How do you track presence at scale?** → Heartbeat TTL keys (`SETEX presence:userId 30s`); expiry = offline; fan out changes to contacts; eventual consistency is fine.
5. **Group chat — how do you fan out?** → Small groups fan-out-on-write to each inbox; large channels fan-out-on-read — the same hybrid as the News Feed, capping write amplification.
6. **Why a wide-column store for messages, and what keys?** → Append + range-scan by conversation (no joins) at write-heavy scale; partition by `conversation_id`, cluster by `message_seq` → efficient "last N / paginate older."
7. **Typeahead — how do you hit <100 ms?** → Sharded trie/FST with precomputed top-k per node → walk-to-prefix-then-read (no request-time DFS); debounce on the client; hot prefixes cached in Redis.
8. **How does the typeahead index stay fresh without racey reads?** → Offline pipeline aggregates query logs → rebuilds the trie → ships via a versioned blob with an atomic pointer flip, so reads never see a half-built index.
9. **Why rank typeahead by frequency, not alphabetically?** → Users want popular completions ("fa"→"facebook"); frequency comes from aggregated query logs in the offline pipeline.
10. **Walk me through Smart360.** → Multi-tenant SaaS; performance (N+1→JOIN FETCH/EntityGraph + indexes + Redis + S3 cache → 60s→2–3s, 96%), security (session→JWT at gateway, ~60% sign-in cut, 3-level RBAC), architecture (strangler-fig, notification boundary first, ~30% gain).
11. **(Curveball) Smart360's hardest bug?** → `LazyInitializationException` accessing a lazy association outside the session; fixed with `@Transactional(readOnly=true)` + `@EntityGraph`.
12. **(Curveball) How does Smart360 enforce multi-tenancy?** → Tenant id in the JWT → Hibernate `@Filter` applies the tenant predicate → Postgres RLS as a DB-level backstop against code bugs.
13. **Walk me through Deep Fathom's infra.** → 30+ Azure resources in modular Bicep (Container Apps, PostgreSQL Flexible, Redis, Key Vault, Front Door, private endpoints), one param file per env; DAG + BuildKit CI/CD 23→10 min (57%); probes + graceful shutdown; managed-identity Key Vault; RLS across 50+ tables.
14. **(Curveball) Key Vault is down when Deep Fathom starts.** → Deploy fails safe (won't serve misconfigured); but runtime secrets are cached in memory, so a post-startup outage doesn't take running pods down.
15. **Walk me through WebX.** → LLM orchestration for 2–20 min jobs across 5 providers; POST→202+jobId; PostgreSQL durable job state + Redis Streams queue; worker pool routes by cost/capability/rate-limit (Redis INCR/EXPIRE); SSE/polling for results; idempotency keys.
16. **(Curveball) WebX worker pod killed mid-job.** → Heartbeat every 30s; `@Scheduled` watchdog marks STALE after ~2 min and re-queues; idempotent provider calls prevent double-billing.
17. **Tie your three projects into one sentence.** → "Smart360 taught me performance engineering, Deep Fathom infra ownership, WebX async + LLM — now I want to own a domain end-to-end at scale."
18. **(DSA) Why sentinel head/tail nodes in the LRU list?** → They eliminate edge null-checks; insert and remove become unconditional pointer swaps, keeping both O(1) and bug-free.
19. **(DSA) LRU vs LFU — what changes?** → LRU evicts least-recently-used (a doubly-linked list); LFU evicts least-*frequently*-used with recency as the tie-break (frequency buckets + a min-freq pointer).
20. **(Curveball) SSE vs WebSocket for the LLM stream — defend it.** → SSE is unidirectional server→client, plain HTTP/1.1, trivial to proxy through the gateway; WebSocket's bidirectionality is overkill for a read-only token stream — reserve it for chat.

---

## ✅ Self-check

1. Did the **chat design** cover connection routing, sequence ordering, the receipt state machine, the small-vs-large group fan-out split, and the wide-column partition/cluster key — **unprompted**?
2. Can you explain the **typeahead** serving path (sharded trie → precomputed top-k → walk-then-read) and the offline refresh (versioned blob swap) in under 5 minutes?
3. Do **all three talk-tracks** land in time (Smart360 ~5, Deep Fathom ~5, WebX ~4) with **no "basically"** and a number in every contribution?
4. Can you handle all four curveballs cold — **`LazyInitializationException`**, **Key Vault outage**, **crashed worker**, **multi-tenancy isolation**?
5. Can you recite **LC 146 (LRU)** from scratch in ≤10 minutes and explain what the sentinels and the doubly-linked list each buy you?

---

<div align="center">

← [02-tue-jul-28.md](02-tue-jul-28.md) · [Week 5](README.md) · [04-thu-jul-30.md](04-thu-jul-30.md) →

</div>
