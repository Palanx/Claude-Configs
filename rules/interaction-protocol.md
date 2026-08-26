# Interaction protocol

## Implement only on explicit order

Do not create, edit, or delete any file unless I gave an explicit instruction
to implement.

Explicit: "implement X", "fix it", "apply that", "do it", "go ahead".
Not explicit: "what do you think of X", "X is broken", "we could do Y",
"how would you solve Z", a pasted error, a pasted ticket, a pasted diff.

When the request is not an explicit order, stop and ask which I want:

1. Implement it now.
2. Answer as a question — explanation only, zero file changes.
3. Write a plan — use the planning skill.

Ask once, in one short message. Never guess. Never start with a "small" edit
while waiting. Ambiguity resolves to asking, not to acting.

Reading files, searching the codebase, and running non-mutating commands never
need approval.

An order covers the whole unit of work it names, including files other rules
require as part of it — tests, checks, migrations. It does not cover files or
call sites the order did not name. If doing the work correctly means touching
code outside its stated scope, say so and wait before touching it. Writing a
plan file is the exception to the no-writes rule: it is what option 3 asks for.

## Tech debt

When work starts on a ticket or task, read the tech debt log —
`.cursor/rules/tech-debt.mdc` or `.claude/rules/tech-debt.md`, whichever exists.
It is my log of already-triaged debt, not a to-do list. If neither exists, skip
this section and never create it.

Check it for entries touching the same files, modules, or behavior as the
current task. If any match, ask before doing anything else:

    Related tech debt: <section title> — <one-line summary>. Fix it in this
    session, or leave it?

- Ask once per session, listing every match together. Do not drip-feed.
- If I decline, do not raise it again in that session.
- If I accept, treat it as an explicit order for that entry only.
- Never fix logged debt that was not approved, even when the file is already
  open and the fix is trivial. The log records that the debt was seen and left
  alone on purpose; do not re-report an entry as a discovery.
- Never edit the log silently. After a fix is confirmed, propose the update and
  wait.
- If an approved implementation introduces new debt, say so and propose an
  entry. Do not write it unprompted.
