# Week 9 · Day 2 — Tue Aug 25 — Execution Drills: Coding Interview Mechanics

📌 **Today:** Drill the *mechanics* of executing under live pressure — clarifying before coding, narrating audibly, a structured "I don't know," and graceful recovery from being stuck. A structured "I don't know" beats a correct answer delivered with hesitation.

**DSA maintenance (~30 min):**
- [ ] One problem from your weak-area list, solved **with narration** — talk through the approach aloud as if the interviewer is listening.
- [ ] After: state time complexity, space complexity, and one alternative (even if worse). This habit separates strong candidates from mediocre ones.

**Primary activity:**

**Interview execution drill (60 min) — structured solo mock, four phases:**

*Phase 1 — Clarifying questions (10 min):*
- [ ] Take any problem ("Design a rate limiter" / "Find all paths in a grid"). Before writing a line, ask **5 clarifying questions aloud**:
  - "What scale — thousands of req/s or millions?" · "Do paths avoid obstacles? Is the grid rectangular?" · "One-time query or does it support updates?" · "Latency constraints? Read- or write-heavy?" · "Optimise for space or time?"
- [ ] Make pause-and-clarify a reflex. Diving in without it looks impulsive.

*Phase 2 — Thinking out loud (20 min):*
- [ ] Fresh problem — **LC 207 (Course Schedule)** or **LC 994 (Rotting Oranges)**. Narrate every step:
  - "My first instinct is BFS because..." · "I see a cycle issue — let me handle that..." · "I'll start with brute force to validate, then optimise..." · "One edge case: empty input..."
- [ ] Record it. Play it back. Confident structured thinker, or muttering?

*Phase 3 — Handling "I don't know" (15 min):*
- [ ] Pick 3 questions from interview-qa.md you're least confident on. Practice the recovery script aloud:
  - "I don't have the exact answer top-of-head, but let me reason through it..." · "What I know for certain is [related concept], so I'd expect..." · "I've seen a similar pattern in [project] — I'd approach it..." · "In production my first step would be to look at [tool/doc]..."
- [ ] Never freeze.

*Phase 4 — Recovery from stuck (15 min):*
- [ ] Take a hard unsolved problem — **LC 124 (Binary Tree Max Path Sum)** or **LC 297 (Serialize/Deserialize Binary Tree)**. Get stuck on purpose, then recover:
  - "Let me re-read the constraints — maybe I'm overcomplicating." · "Let me trace a tiny example." · "What data structure naturally models this?" · "Could I solve a simpler version first?" · "Could you hint at the data structure you have in mind?"
- [ ] A graceful hint-ask earns more respect than 5 minutes of silence.

**Pipeline:** Respond to recruiters within hours. Log any round outcome the same day. Keep applications tapered — focus is live loops, not new top-of-funnel.

**EOD check:**
- [ ] Did you narrate a full solve out loud without going silent to "think"?
- [ ] Can you give three phrasings of "I don't know the exact answer, but..."?
- [ ] DSA narrated solve done?

---
*Nav: ← [Day 1 (Mon Aug 24)](01-mon-aug-24.md) · [Week 9](README.md) · [Day 3 (Wed Aug 26)](03-wed-aug-26.md) →*
