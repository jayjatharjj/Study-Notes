# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a documentation-only Java study + interview-prep repository — no build system, tests, linters, or dependencies. It has two parts: **(1)** 8 sequential Java modules (`01`–`08`) — reference theory; **(2)** a **full-time interview-prep sprint** under `prep-weeks/` — a day-by-day plan with a DSA solutions bank, question banks, cheat-sheets, and trackers. It is designed to be **fully self-contained / offline-usable** (no LeetCode or external lookups needed). Start at [`STUDY-OFFLINE.md`](STUDY-OFFLINE.md).

```
Study-Notes/
├── 01..08-*.md                 — Java theory modules (basics → modern Java + Spring Boot 3)
├── STUDY-OFFLINE.md            — how to study the whole repo with no internet (the map)
├── README.md                   — master index
├── projects-reference.md       — Smart360 / Deep Fathom / WebX (grounds system design + STAR)
├── interview-qa.md             — why-focused Q&A bank
├── career-plan-2026.md         — market analysis + full sprint timetable + strategy
├── PROGRESS.md                 — daily completion tracker
├── cheat-sheets/               — dsa-patterns · system-design · java-spring (rapid revision)
└── prep-weeks/
    ├── README.md               — week index (Week 0 + W1–W12)
    ├── week-00-basics/         — 1-day refresh + 130-Q interview bank + 76 coding problems
    ├── week-01/ .. week-05/    — STUDY weeks: deep per-day files (theory + practice + interview Qs)
    │                             + interview-answers.md per week
    ├── week-06/ .. week-12/    — EXECUTION weeks: per-day apply/interview/negotiate checklists
    ├── dsa-solutions/          — ~137 problems, full statements + Java 17 solutions (12 pattern files)
    └── applications-and-referrals.md — quotas, referral playbook, Floor/Safety-Net ladder, trackers
```

The prep calendar runs Sun Jun 28 (1-day refresh) → full-time study W1–5 (Mon Jun 29 – Sat Aug 1) → execution W6–12 (Aug 3 – Sep 20). Each `prep-weeks/week-NN/` is a folder of numbered per-day files (`01-mon-*.md`) + a `README.md` index; dates are real 2026 calendar dates, so edits that shift the schedule must keep weekday/date pairs consistent.

## Content Patterns

- Each module begins with a **Quick Reference** blockquote summarizing the module.
- **Interview Q&A sections** appear at the end of each module — why-focused, not just what.
- Code examples mark anti-patterns as `// BAD` and preferred patterns as `// GOOD`, with expected output in comments.
- Real-world context is woven throughout: String immutability → JWT/caching, HashMap → DI containers, thread pools → Tomcat, exceptions → REST error handling.

## Working with the Notes

Search across modules:
```bash
grep -rn "your-term" Study-Notes/
```

To cross-reference a concept, check the relevant module by number — the sequence is the dependency order (01 before 03, etc.).
