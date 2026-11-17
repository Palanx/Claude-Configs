---
name: engineering-guidelines
description: >
  Situational engineering detail behind the always-on baseline in rules/engineering.md: SOLID,
  Clean Architecture layering and the dependency rule, the design pattern consultation
  protocol, testing (regression tests, proportional testing, AAA), the evidence each
  completion claim requires, and how to receive a code review. Use when designing or
  restructuring a class or module, choosing a design pattern, writing or planning tests, about
  to claim work is done, fixed, or passing, or acting on review feedback. For system-level
  architecture scoping and tradeoff scoring, use architecture-evaluation; for bugs, debugging.
---

# engineering-guidelines

The baseline every task follows — right-sizing, reuse, clean code, errors, security,
commits — is `rules/engineering.md`, loaded every session. This skill holds what only some
tasks need. Read the reference that matches the situation; do not load all of them.

| Reference | Read when |
|---|---|
| `references/architecture.md` | Designing or restructuring a Core-tier module, or choosing a design pattern |
| `references/testing.md` | Writing tests, or implementing a feature or fix that needs one |
| `references/verification.md` | About to claim anything is done, fixed, building, or passing |
| `references/code-review.md` | Receiving review feedback — from a human or an agent |

Without the rule loaded — claude.ai, or a surface that reads skills but not rules — apply the
right-sizing gate before any of these: full architecture applies only to Core-tier code.
