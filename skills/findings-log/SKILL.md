---
name: findings-log
description: >
  Write and maintain a project's findings log — expensive, verified facts about how the code
  actually behaves, where the obvious reading of the source is wrong. Use when a fact like
  that has just been established and should outlive the session ("anotá esto como finding",
  "log this finding", "guardá lo que aprendimos de X", "save what we learned about X", "add a
  findings entry"), and when asked what is already known about a part of the codebase. Not
  for decisions, defects deliberately left in place, or architecture overviews — those have
  other owners, named in the refusal table below.
---

# findings-log

A record of how the code **actually behaves** where the obvious reading of the source says
otherwise, established at real cost. Not a decision, not a defect, not a map of the repo.

That is what makes it worth its cost. A finding is the most expensive kind of knowledge a
session produces and the least durable: nothing in the repository records it, no test asserts
it, and the next session reads the same misleading source and reaches the same wrong
conclusion. One entry replaces an afternoon.

## What belongs here

Non-obvious facts about how the code behaves, where the obvious reading is wrong, that cost
real work to establish. Nothing else. Two tests, both required:

- **Would a competent reader get it wrong from the source alone?** If the code says what it
  does, it is not a finding — it is the code.
- **Did establishing it cost more than reading one file?** Cross-repo tracing, a debugger
  session, reading a query plan, asking someone who was there. A fact confirmed by a
  five-second grep is not worth an entry; the entry costs more to maintain than to rederive.

A finding is **not binding**. It describes, it does not constrain. Say so in the file, or a
reviewer reads it as a rule and starts enforcing an observation.

## What does not, and who owns it instead

A refusal with no destination sends the fact back to the limbo it came from. Name the owner:

| Not a finding | Owner |
|---|---|
| Architecture overview, directory layout, module inventory | The repo's generated index, if it has one — it is derived from the code and regenerated, so a hand-written copy is a copy that rots |
| A convention the project follows now | The project's standing-conventions doc |
| Build, test, lint, or run commands | The project's toolchain config, or its README |
| A decision, with its rationale | The project's decision records (ADRs) |
| A defect understood and deliberately left | The project's tech debt log |
| Something that will be false next week | Nowhere. Say it in the session and let it go |

In a project that has none of those, the refusals still hold in substance: a fact that any of
them *would* own does not become a finding because the owner is missing. Say which artifact
the project lacks and offer to write it, rather than smuggling its content in here.

## Where it lives

One file per **domain**, at whichever path the tooling reads:

- `.claude/rules/<domain>.md` — Claude Code, with `paths:` frontmatter
- `.cursor/rules/<domain>.mdc` — Cursor, with `globs:` and `alwaysApply: false`

Never create one unprompted. A project without a findings log has decided not to keep one.

The file is **tracked**, in any repository where tracking is possible. The cost of rederiving
a finding is paid per person, not per machine; an untracked log is the same loss one machine
wider. Where the repository cannot receive new files — someone else's repo, a workspace under
a no-touch policy — it is untracked by force, and that is a constraint of the situation, not
a property of findings. Do not carry it over to a repo that has no such constraint.

**Scoping is not optional, and getting it wrong is expensive.** A rule file with no `paths:`
(or no `globs:`, with `alwaysApply: false`) loads into **every session**, at the same priority
as the project's main instruction file. A forgotten `paths:` does not make the log
unavailable — it makes it permanent, another always-loaded document competing for adherence
with the ones that earned their place. Every file this skill writes has a scope, always.

The frontmatter is a list of quoted globs, and it covers every path named in every entry's
`Files:` line. In `.claude/rules/<domain>.md`:

    ---
    paths:
      - "src/sync/**/*.ts"
      - "src/db/schema.sql"
    ---

In `.cursor/rules/<domain>.mdc` the key is `globs:`, same list shape, plus `alwaysApply: false`
alongside it.

**If the project already keeps its findings somewhere else** — a standing-conventions doc with
a section for observed behaviour is the common case — that is the home, and it stays the home.
Append there and write no rule file. A finding in two places is a finding that will disagree
with itself.

## Entry schema

    ## <Short title> (verified YYYY-MM-DD)

    Files: `PathOrType`, `PathOrType`, ...

    <The fact. What the obvious reading would be, and why it is wrong.>

    <What it buys: the design it collapses, the investigation nobody has to repeat.>

**The date is when a human or a session last checked the entry against the code** — not when
the fact was discovered. Together with `Files:` it makes staleness a query rather than a
feature: `git log --since="<date> 00:00" -- <files>` answers "did the ground move" without
rereading anything. Keep the `00:00`: git fills a missing time with the current time of day,
so a bare `--since=<date>` silently hides every change committed on that date.

**`Files:` is exhaustive, and it is the index.** It is how a session working on one of those
files discovers the entry exists, and it is what the `paths:` / `globs:` frontmatter has to
cover. Mark files that resolve outside the repository — generated caches, vendored sources —
because a reader cannot follow them where they are.

**Why the obvious reading is wrong** is the entry. Without it you have written a comment that
belongs next to the code. "The queue retries on network errors" is a code comment. "`Flush()`
looks like it retries on network errors — the `catch` on line 40 says so — but the client it
calls has already swallowed and translated those, so the branch only ever sees validation
errors and the retry is dead code" is a finding.

**What it buys** is what justifies the entry's existence. A fact that changes nothing is
trivia. Name the design it collapses or the search it ends.

Worked example:

    ## Archived tasks never reach the board query (verified 2026-03-14)

    Files: `src/board/BoardQuery.ts`, `src/db/views.sql`

    `BoardQuery.forColumn()` appears to filter archived tasks in application code, and the
    `if (!task.archived)` guard says so. That guard has never run: the SQL view it selects
    from excludes archived rows, and has since the view was introduced.

    Anything designed around "archived tasks arrive and get filtered late" is designing
    against a path that does not exist. The archive state is gone before the client sees a
    row, so archive-aware rendering, archive counts, and an unarchive-without-refetch flow
    are all unreachable from this query — they need a different one.

## Maintenance

- **Keep the index in sync.** Adding an entry means adding its files to `paths:` / `globs:`.
  An entry the tooling never surfaces is an entry nobody reads.
- **Re-verify, don't rewrite.** When an entry is confirmed still true, bump its date. A
  corrected entry left with its old date reads as fresh and lies twice.
- **Delete when it stops being true.** The code changed, the finding is history, and history
  lives in git. A findings log that accumulates is a log nobody finishes reading.

### When to offer revalidation

**Only when an entry's date is older than the last change to a file it names.** That is one
`git log` away and it is the whole trigger.

Do not offer revalidation on attach. The rule fires on every read of every matching file, and
a prompt that fires unconditionally gets dismissed unconditionally — after a week it is noise
the reader has trained themselves past, which is worse than no prompt at all.

Do not instruct anyone to run the build or the tests to confirm what an entry says. In plenty
of projects that is not something a session can do, and an instruction that cannot be
satisfied is an invitation to claim it was.

## Voice

Name the symbol, name the file, name the condition. "The caching is confusing" is not an
entry. "`Resolve()` caches by display name, not by id, so two records that render identically
share one cache slot and the second read returns the first record's payload" is.
