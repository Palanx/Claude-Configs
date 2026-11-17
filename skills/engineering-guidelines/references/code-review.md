# Receiving a code review

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
- **Act by severity.** Critical (data loss, security, wrong behavior shipped) immediately ·
  Important (bug under a reachable condition, missing boundary validation) before
  proceeding · Minor noted, batched, or declined with a reason.
- **A marked shortcut is a decision, not a finding** — unless the expected load already
  crosses the ceiling its comment names.
- **Out-of-scope items** are listed and left for the user to rule on, not absorbed silently
  into the diff.
