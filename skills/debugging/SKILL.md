---
name: debugging
description: >
  Find the root cause of a bug before fixing it: read the error completely, reproduce, check
  recent changes, instrument boundaries, trace the bad value back to its source, then test one
  hypothesis at a time — and stop to question the design after three failed fixes. Use for any
  bug, test failure, crash, build failure, regression, or unexpected behavior, before proposing
  a fix: "no funciona", "se rompió", "falla el test", "tira este error", "why is this
  happening", "fix this bug", a pasted stack trace. For bugs that are intermittent or only
  reproduce on device, use repro-loop for the instrumentation loop.
---

# debugging

No fix without root cause investigation first. A symptom fix is a failure even when the
symptom disappears. Under time pressure this matters more, not less: systematic is faster
than thrashing.

Complete each phase before the next. This skill points at `repro-loop` and at
`engineering-guidelines` references; where those are absent, the phases below still stand on
their own.

## Phase 1 — Root cause investigation

1. **Read the error completely.** Full stack trace, line numbers, file paths, error codes,
   warnings. The answer is often already in it.
2. **Reproduce consistently.** Exact steps, every time. Not reproducible → gather data, don't
   guess. Intermittent or device-only → `repro-loop`.
3. **Check recent changes.** `git diff`, recent commits, new dependencies, config changes, and
   environment differences between where it works and where it doesn't.
4. **Instrument component boundaries** in multi-component systems: log what enters and leaves
   each boundary (input → system → engine, or CI → build → package → device), run once, and
   investigate only the component where it breaks.
5. **Trace the bad value backwards** to where it originates. Fix at the source.

## Phase 2 — Pattern analysis

1. Find a working example of the same thing in this codebase.
2. Read the reference implementation completely if applying a known pattern.
3. List every difference between working and broken, however small. "That can't matter" is how
   the cause gets skipped.
4. Check dependencies: required components, settings, environment, implicit assumptions.

## Phase 3 — Hypothesis and test

1. **One hypothesis, stated:** "X is the root cause because Y."
2. **Smallest change that tests it.** One variable at a time.
3. Didn't hold → a new hypothesis. Never stack a second fix on a failed one.
4. **Not knowing is a valid answer.** "I don't understand X" — then research or ask. Never
   invent a mechanism.

## Phase 4 — Fix

1. **Regression test first**, failing before the fix
   (`engineering-guidelines/references/testing.md`). If the bug cannot be tested — Editor-only
   rendering, device-only timing — say so and state how the fix will be checked instead.
2. **One fix, at the root cause.** No bundled refactor, no "while I'm here".
3. **Fix where all callers route through.** Find every caller of the function first; one guard
   in the shared function fixes the sibling callers the report did not name.
4. **Verify:** the new test passes, no other test broke, the original symptom is gone
   (`engineering-guidelines/references/verification.md`).

## The 3-fix rule

Three failed fixes means the model of the problem is wrong, not that the fourth patch is the
right one. Stop and surface it: "Three attempts failed. The likely issue is [structural
cause]. Options: [A] / [B]."

## Red flags

A fix "to see what happens" · several changes at once · "it's probably X" without evidence ·
reading only the last line of the trace · a retry, timeout, or sleep added to make a race go
away · disabling the failing test.
