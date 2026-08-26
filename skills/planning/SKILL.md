---
name: planning
description: >
  Write an implementation plan that a human executes step by step: independently
  verifiable steps, exact verification commands, changes anchored to existing code,
  labelled snippets, no unresolved questions buried mid-step. Use whenever a plan is
  asked for before implementing — "hacé un plan", "plan de implementación", "escribí
  el plan", "necesito un plan para", "write a plan", "plan this out", "plan para este
  bug", "/planning" — for a bugfix, a refactor, a migration, a feature, or after
  belay's /expand-phase when a human will implement the phase by hand. This produces
  a document for a person to execute, not a script for the agent to run.
---

# planning

Plans exist so a human executes them step by step. A long plan read in one sitting hides
its own defects; executed one step at a time, defects surface while they are still cheap
to fix.

Write for one reader: a competent engineer who does not have the codebase in working
memory. They do not. You do, while writing the plan — so whatever you can re-derive by
reopening a file is exactly what you will leave out, and exactly what they cannot
reconstruct without redoing all of your reading. A step is written well when it can be
executed without opening any file except to edit it.

- Each step is independently executable and independently verifiable.
- Order steps so the repository is left working after each one.
- Prefer more, smaller steps over fewer, larger ones.
- Mark hard checkpoints explicitly: **STOP — verify before continuing.**

## Where the plan goes

Detect, in this order:

1. Inside a project (VCS root, or a tool directory exists):
   - `.cursor/` exists → `<project-root>/.cursor/plans/`
   - else `.claude/` exists → `<project-root>/.claude/plans/`
   - else → `<project-root>/.claude/plans/`
2. Outside a project: `~/.cursor/plans/` if it exists, otherwise `~/.claude/plans/`.

Create the directory if missing. Filename: `YYYY-MM-DD-<kebab-case-topic>.md`.

The date-prefixed name also keeps these distinguishable from Claude Code's own plan-mode
files, which share `~/.claude/plans/` and use a different naming shape.

Without filesystem access — claude.ai chat, or any surface with no file tools — the plan
**is** the response: same layout, same step schema, no Execution log, since that section
is written by whoever executes and it needs a file to be written into.

## File layout

1. **Header** — ticket or the fact that there is none, branch, assumptions, unknowns,
   open questions. Nothing else. No revision changelog and no notes on how the plan
   evolved: that is version control of your own drafting and it is noise to whoever
   executes.
2. **Context** — reference material: how the thing is modelled, constraints, findings and
   where they were verified. Read once, never executed. Steps link into it; it never
   contains an instruction.
3. **Steps.**
4. **Execution log** — the only section the executor writes into: values discovered while
   running (paths, ids, locks taken), and whatever could not be verified, with the reason.

## Step schema

Always present, in this order:

- **Goal** — one sentence: what is true after the step that was not true before.
- **Files** — full paths, exhaustive.
- **Action** — numbered imperatives, one action per item.
- **Verify** — the exact command, or the exact UI path, plus the expected observable
  result. "Check that it looks right" is not a verification.

Only when they apply, placed after Action:

- **Trap** — the plausible wrong implementation and the symptom it produces. Write it
  whenever the obvious reading of the action is wrong or incomplete. It is the only thing
  that stops the executor from "simplifying" the step and breaking it.
- **Rejected** — alternatives considered, each with the concrete rule, ADR, or mechanism
  that kills it. Without this the executor re-litigates the decision mid-execution.
- **Do not touch** — code inside the blast radius that must be left alone, and why.

## Voice

- One grammatical person for the whole file. Instructions are imperative and addressed to
  the executor.
- Never phrase an action impersonally or in the passive. "The colour is done with a value
  converter" reads as a description of code that already exists; "bind `IsActive` to the
  label's foreground through a value converter" is an instruction.
- Every block is exactly one of: action, verification, context, warning, decision. Never
  mix two of them in one paragraph. Anything that is not an instruction carries a label
  saying what it is.
- Reasoning for a non-obvious decision goes inside the step that depends on it, as a short
  marked block of three or four lines. Anything longer goes to Context and is referenced
  from the step.

## Anchors

- Anchor every change to existing code by file, then class or method, then the statement
  it goes after or before. Line numbers are a secondary aid, never the only anchor: they
  rot on the first commit by someone else.
- State ordering constraints explicitly wherever they exist — for example, apply the
  discount after the tax field is assigned, never before, because before it computes
  against the pre-tax total.

## Code in plans

No walls of generated code. Label every snippet:

- **Normative** — write it verbatim. Use this only where the exact text matters.
- **Illustrative** — it states the business rule or the shape; the real code is the
  executor's to write.

An unlabelled snippet is read as normative, so an illustrative one presented bare gets
typed in as-is.

## Decisions

A step never contains an unresolved question. If a decision blocks the work, either ask
before writing the plan, or write it as its own numbered step stating the default, what
changes under each branch, and what to do once chosen. "This one is up to you", buried
mid-step, halts execution with no way forward.

## Placeholders

No blanks to fill in inside a step. If a value can only be known at execution time, the
step says how to obtain it and to record it in the Execution log.

## After writing

Writing a plan file is not implementation. After writing it, stop. Do not begin executing
it, not even step 1, until told to.

## Belay mode

Applies when the request names a phase id, or `docs/phases/<id>/spec.md` exists
(`.belay/docs/phases/<id>/spec.md` in a corporate install). No spec → standalone; skip
this section.

The spec is authoritative, this plan is not. Derive in one direction only: spec → plan.
Never edit the plan to match reality — amend the spec and regenerate the plan **whole**. A
half-regenerated plan is exactly the drifting second document this mode exists to avoid.

Say so in the Header:

```
Derived from: docs/phases/<id>/spec.md
Status: disposable, regenerable, never authoritative — /validate-phase judges the
        spec, not this file.
```

### Mark what the spec did not give you

Writing the plan will need facts the spec does not state. Mark each one inline where you
used it:

    spec-gap: <what was missing> → <spec section it belongs to>

and collect them in a final section `## What the spec is missing`, one line each. That
list is the point of this mode: it turns "the plan was hard to write" into an actionable
edit, instead of an `undecidable` verdict discovered later by `/validate-phase`.

The filter, so the list stays readable — mark it **only** if a fresh session with
`CLAUDE.md` plus the phase directory would need it to complete the phase (belay's own
closure test, P5):

| Mark it | Leave it unmarked |
|---|---|
| A file you had to go find, a name, an ordering constraint, a dependency, a check the spec never stated | How a human physically performs a step: clicks, where to look in the Inspector, explanatory context |
| Belongs in the spec | Belongs in this plan, and nowhere else |

Mark both kinds and the list becomes noise; by the third phase nobody reads it. Execution
pedagogy must stay out of the spec — it would bloat the starved reviewer's input in
`/validate-phase` step 5 without telling it anything.

Each entry names the spec section it belongs to, because that decides the route:

- **Context pointers** or **Plan** → edit `spec.md` in place, regenerate this plan, keep
  going. No status change, no record owed: nothing has consumed the old spec yet.
- **Acceptance criteria** or **Out of scope** → the spec's contract changed. Set the phase
  status back to `pending` and re-run `/expand-phase <id>`; it gets regenerated, not
  patched.
- **Goal** → the cut itself is wrong: `/plan-feature`, which supersedes the row.

If the amendment lands after hand-implementation already started, it also goes into the
Deviations answer below — the spec has to end up describing the change that actually
happened.

### Execution log

In belay mode the Execution log is not the final record; `notes.md` is. When the work is
done, run `/implement-phase <id> --implemented`: its interview asks for Outcome,
Deviations, Debt and For later phases. Feed the Execution log into Deviations and Debt,
including every `spec-gap:` acted on. Then this plan can be deleted.

Belay knows nothing about this skill and must not: a repo that depends on a skill in
someone's home directory depends on a channel it cannot see. The dependency runs one way.
