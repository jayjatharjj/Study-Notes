# Week 7 · Day 3 — Wed Aug 12 — System-Design-Under-Time Mock + Referral Asks

📌 **Today:** Run one 45-min system-design mock cold so the first real SD round doesn't catch you flat-footed. Then push 5 referral asks toward the weekly target.

**DSA maintenance (~45 min):**
- [ ] **LC 1046 – Last Stone Weight** (Easy) — max-heap warm-up. Under 10 min.
- [ ] **LC 215 – Kth Largest Element** (Medium) — min-heap of size k (O(n log k)); mention QuickSelect as the "can you do better?" follow-up. Under 20 min.

**Primary activity — system design under time (45-min mock):**

Pick a prompt cold, set a 45-min timer, whiteboard on paper. Suggested: **"Design a scalable file storage service"** or **"Design a notification system for 10M notifications/day."** Drive it end-to-end — do not wait to be prompted. The skeleton never changes:
- [ ] **Requirements + capacity (8 min):** clarify file sizes/volume, read:write ratio, consistency needs, global vs regional. Rough estimate (e.g. 10M users × 5 GB = 50 TB; 100K uploads/day × 10 MB = 1 TB/day).
- [ ] **High-level design (10 min):** for file storage — chunked upload via **pre-signed URLs** (data never flows through your servers), metadata service on PostgreSQL, S3/Blob for content, CDN on the read path. Name the pre-signed-URL caching pattern from Smart360.
- [ ] **Two deep dives (12 min):** metadata sharding (hash on `owner_id`), leader-follower replication + lag handling, *or* notification channel abstraction + dedup/idempotency + retry/DLQ.
- [ ] **Failure modes (10 min):** orphaned chunks (S3 lifecycle cleanup), DB leader failover (CP trade-off), CDN stale-after-delete (invalidation), provider 5xx → exponential-backoff retry → DLQ.
- [ ] **Trade-offs + observability (5 min):** name CAP/PACELC where relevant; one sentence on how you'd *operate* it (golden signals, trace propagation). Tie at least one decision to your real work.

**Referral outreach (30 min):**
- [ ] Send 5 referral asks (toward the ~10 weekly target), prioritising companies where you've applied. Same template, personalised, logged.

**Pipeline:** Log the SD mock self-grade and 5 referral asks in the [tracker](../applications-and-referrals.md).

**EOD check:**
- [ ] Did the SD mock cover failure modes *and* a capacity estimate, not just the happy path?
- [ ] Are 5 referral asks logged?
- [ ] Both DSA problems done under time?

---
*Nav: ← [Day 2](02-tue-aug-11.md) · [Week 7](README.md) · [Day 4 →](04-thu-aug-13.md)*
