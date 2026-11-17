# Delegating to subagents

A subagent starts cold: it inherits none of the session's reasoning, so the brief decides the
output. Delegate to keep the main thread's context on the problem — a broad search where only
the conclusion matters, bulk mechanical work, a fresh-eyes review — not to avoid thinking.
Keep inline anything that needs the conversation's accumulated context, small bounded edits,
and design decisions.

## Cascade, never converge

Subagents run in sequence. Each one starts from the previous one's verified result.

- **Never dispatch parallel agents whose outputs must be merged**, or whose work depends on a
  contract still open. Expanding one spec can change what another agent is assuming; the
  resync that follows costs more than running them in order, and the orchestrator pays every
  agent's context to do it.
- Parallel is acceptable only for read-only work with no shared conclusion — independent
  searches whose results are read, not merged. When in doubt, run in sequence.
- The user decides when delegation is worth it at all for a project; a veto holds until lifted.

## The brief

- **Task** — the objective, and what it must not touch.
- **Outcome** — success criteria concrete enough to verify; acceptance criteria as tests where
  possible.
- **Objects** — the exact files it may create or edit, the files it must read, the paths, SHAs,
  constraints, and prior decisions it needs. It cannot see this session.
- **Next** — the exact shape of the report wanted back.

Context the agent does not need is cost, not help.

## What the subagent may do to verify

The compiler, the linter, and the tests are its only judges: build the affected project, lint
the touched files, run the affected tests. Not inside a subagent: starting databases,
containers, or the app; calls against live services; scratch programs "to try it"; web
searches unless the task is research; installing packages; commit, push, merge, or deploy.
Manual verification is the orchestrator's, once, after the cascade.

Same failure three times for the same cause → stop and report the exact error, not a fourth
attempt.

Report back in about 15 lines: files changed · build, lint, and test commands with their
numbers · assumptions and open items. No restating the brief, no explaining the code.

## After it returns

- **A success report is not evidence.** Read the diff and run the tests
  (`verification.md`). A subagent that could not find a file often reports a plausible summary
  of work it did not do.
- **Never relaunch or resume an agent without first reading what it left behind.** Half-done
  work is finished where it is, not redone from scratch.
