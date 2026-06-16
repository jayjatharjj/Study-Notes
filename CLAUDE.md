# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a documentation-only Java study notes repository — no build system, tests, linters, or dependencies. The content is 8 sequential markdown modules (~5500 lines) covering Java fundamentals through advanced and modern patterns, with a production/interview focus.

```
Study-Notes/
├── 01-java-language-basics.md          — JVM, primitives, control flow, Arrays, Strings, I/O
├── 02-oop-fundamentals.md              — Classes, constructors, encapsulation, static context
├── 03-oop-advanced.md                  — Inheritance, polymorphism, abstractions, lambdas, Singleton/Factory
├── 04-exception-handling-and-memory.md — Exception hierarchy, try-with-resources, GC, final/finally/finalize
├── 05-enums-and-annotations.md         — Enums with methods/fields, built-in and custom annotations
├── 06-concurrency-and-collections.md   — Thread creation, synchronization, Executor Framework, Future/Callable
├── 07-collections.md                   — Full Collections API: List/Set/Queue/Map implementations and algorithms
└── 08-modern-java.md                   — Records, sealed types, switch/pattern matching, text blocks, virtual threads, Jakarta/Spring Boot 3
```

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
