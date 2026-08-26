---
name: tech-debt-log
description: >
  Write and maintain a project's triaged tech debt log — the file that records debt
  found and deliberately left alone, so it is never re-reported as a discovery and
  never fixed by accident. Use when adding or updating an entry ("anotá esto como
  deuda", "log this as tech debt", "add a debt entry", "update the debt log"), when
  asked what debt exists in a project, or when a piece of debt is found during other
  work and the decision is to leave it. Consulting the log before starting a task is
  governed by the interaction protocol rule, not by this skill.
---

# tech-debt-log

A record of debt that was **found, understood, and deliberately left alone**. Not a
to-do list, not a backlog, not a place for "we should probably". Every entry is a
decision someone already made.

That is what makes the log worth its cost: without it, the same defect is rediscovered
every few months, re-investigated from zero, and either re-reported as news or fixed in
passing by someone who never saw the reason it was left.

## Where it lives

One file per project, at whichever path the tooling uses:

- `.cursor/rules/tech-debt.mdc` — Cursor, with `alwaysApply: false` and a `globs:` list
- `.claude/rules/tech-debt.md` — Claude Code, with `paths:` frontmatter

Never create it unprompted. A project without one has decided not to keep one.

## Entry schema

    ## <Short title> (reviewed YYYY-MM-DD)

    Files: `TypeOrPath`, `TypeOrPath`, ...

    <What the code does, why it is wrong, and why nothing breaks today.>

    <The fix, with its cost.>

    <Links: ticket, wiki, server sources.>

**Title and review date.** The date is when a human last confirmed the entry still
describes reality — not when the debt was introduced. Stale entries are worse than no
entry, so the date is what tells a reader whether to trust it.

**Files.** Exhaustive. This is the index: it is how a future session working on
`FooPanel` discovers there is already an entry about it. Mark files that resolve outside
the repository — generated package caches, vendored sources — because they cannot be
fixed in place.

**Why nothing breaks today.** The most valuable line in most entries, and the one people
skip. "Nothing crashes because every caller goes through X, which always passes the
argument" is what turns a scary-looking finding into a known, bounded risk — and it names
the exact condition under which it stops being true.

**The fix, with its cost.** Not just what to do: what it costs. When there is more than
one option, list them cheapest first, and say what each one buys. An entry whose fix is
"stop sending the cursor" is a deletion, but if it costs a full re-fetch on every resync,
that bandwidth *is* the reason it is a deliberate decision rather than a drive-by fix.
Without the cost, the next reader just applies the cheap one.

### Blocks that appear when they apply

**`Not debt, do not convert:`** — code that matches the pattern of the entry but is
correct as written, with why. Without it, the next person applying the fix sweeps up the
call sites that were meant to stay.

**`Pending merge as of YYYY-MM-DD:`** — when the fix depends on work that has not landed.
Name the PR and the branch. Otherwise a reader on `main` finds the symbol missing and
concludes the entry is stale, when it is only early.

**What already works** — anything discovered while investigating that a future session
would otherwise redo: a grep that resolves the offending prefabs, a debugger that already
aggregates the failures, a flag that already exists. Record the method, not just the
result. This is the section that pays for the log.

**Where it was found** — often a different ticket than the one the entry is about. Say so;
it explains why the entry exists and gives the reader the original context.

## Maintenance

- **Keep the index in sync.** Adding an entry means adding its files to `globs:` /
  `paths:` in the frontmatter. An entry the tooling never surfaces is an entry nobody
  reads.
- **Re-review, don't rewrite.** When an entry is confirmed still true, bump its date.
  When it is no longer true, delete it — a corrected entry with an old date reads as
  fresh and lies twice.
- **Delete on fix.** Once the debt is gone the entry is history, and history lives in
  git. Do not leave "fixed in #1234" behind; a log of resolved entries is a log nobody
  finishes reading.

## Voice

Same as any technical prose worth reading: name the symbol, name the file, name the
condition. "Fix the retry logic" is not an entry. "`UploadQueue.Flush()` retries on any
exception, including the validation error that can never succeed, so one malformed item
blocks the queue until the process restarts" is.
