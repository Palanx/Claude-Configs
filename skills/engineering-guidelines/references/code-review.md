# Code review — requesting, performing, receiving

## Requesting a review

Give the reviewer crafted context, never the session history:

```
Range:      <base-sha>..<head-sha>
Built:      <what this change does, 2-3 lines>
Should do:  <the requirement or plan it implements>
Load:       <one person running a tool | many users or processes at once>
Ask for:    correctness bugs, missed callers, security at trust boundaries,
            reuse or simplification. Severity-tag each finding.
```

## Performing a review

- **Read past the diff.** Callers of every changed function, what it calls, its tests. A
  changed signature, return value, or behavior means finding every caller.
- **State the load assumed** and judge scale against it: check-then-write races, per-item
  queries, unbounded growth, O(n²) on real input, per-process state that must be shared.
- **Every finding carries a concrete case:** "this input or situation produces this wrong
  result". No case, no finding. No "consider", no taste, no vague worries.
- **A marked shortcut is a decision, not a finding** — unless the expected load already
  crosses the ceiling its comment names.
- **Spec silence is not permission.** Behavior the spec does not mention is judged by what a
  reasonable user would expect.
- **List what was declined.** One line per behavior considered and set aside as out of scope,
  with the reason. The author rules on each; nothing is dropped silently.

Output, one line per finding, most severe first:

```
path:line: <severity>: <problem>. <fix>.
```

Severities: **Critical** (data loss, security, wrong behavior shipped) · **Important** (bug
under a reachable condition, missing boundary validation) · **Minor** (clarity, naming,
duplication). Skip pure formatting unless it changes meaning.

End with a verdict — `ship` or `fix N first` — and `not checked:` for anything that mattered
but could not be verified.

## Receiving a review

Review feedback is a technical claim about the code, not a social exchange. A reviewer —
human or agent — can be wrong. Verify before implementing; disagree with evidence, not with
deference.

```
1. READ        the complete feedback before acting on any of it
2. UNDERSTAND  restate each item in your own words — or ask
3. VERIFY      check the claim against the actual codebase
4. EVALUATE    is it right for THIS codebase?
5. RESPOND     technical acknowledgment, or reasoned pushback
6. IMPLEMENT   one item at a time, verifying each
```

- **No performative agreement.** No "you're absolutely right" and no "implementing that now"
  before the claim is verified. That is how a wrong review comment becomes a bug.
- **Unclear feedback stops everything.** If some items are unclear, ask about all of them
  before implementing any — items are often related, and partial understanding produces the
  wrong change.
- **Act by severity.** Critical immediately · Important before proceeding · Minor noted,
  batched, or declined with a reason.
- **Out-of-scope items** are listed and left for the user to rule on, not absorbed silently
  into the diff.
