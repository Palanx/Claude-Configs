# Verification before completion

No completion claim without fresh evidence: if the command that proves it has not run in this
session, after the last change, the claim is an assertion.

## The gate

1. **Identify** the command that proves the claim.
2. **Run** it, fresh and complete — not a subset, not an earlier result.
3. **Read** the full output and the exit code. Count the failures.
4. **Compare** it to the claim. It confirms → state the claim with the evidence. It doesn't →
   state the actual status with the evidence.

## What each claim requires

| Claim | Requires | Not sufficient |
|---|---|---|
| Tests pass | The project's full test command, 0 failures | A previous run, only the touched file, "should pass" |
| Linter clean | Linter output, 0 errors | A partial check |
| Builds | The build command, exit 0 | Linter passing, the IDE showing no errors |
| Unity: compiles | Editor log or batchmode compile with 0 errors | The IDE's analyzer, a script that opened fine |
| Unity: player build works | A player build for the target platform, run | Play mode in the Editor — Editor-only code paths, stripping and IL2CPP differ |
| Android: works on device | Installed build run on a device or emulator, logcat read | A green Gradle build |
| Bug fixed | The original symptom retested and gone | Code changed, assumed fixed |
| Regression test works | Failed before the fix, passes after | Passes once, after the fix |
| Subagent completed | The diff shows the changes | The agent reported success |
| Requirements met | A line-by-line pass over the request | The suite is green |

When a claim cannot be verified from here — it needs a device, the Editor, a build only the
user can run — say which claim is unverified and what would verify it.

## Red flags — stop and run the command

"should", "probably", "seems to" · declaring success before running anything · about to
commit, push, or open a PR unverified · trusting a subagent's report · extrapolating a partial
check to the whole.
