# Testing

Tests are production code: same naming, same clarity. They test behavior, not
implementation — a test that knows about private methods breaks on every refactor and
protects nothing.

## What needs a test

| Work | Test |
|---|---|
| New behavior at Evolving or Core tier | A new test. No new behavior without one |
| Bug fix | A regression test that **fails before the fix and passes after**. Passing before the fix means it does not reproduce the bug |
| Refactor | The existing suite stays green; no new test required |
| Throwaway tier | One runnable check: the smallest thing that fails if the logic breaks — an assert-based self-check or one small test file. Trivial one-liners need none |
| Engine-bound presentation (rendering, animation, VFX, UI layout in Unity/Unreal) | Ask before skipping. Test the logic underneath it; keep that logic engine-agnostic so it can be tested |

Skipping a test where one is required is the user's decision, not yours. Ask.

## Writing one

- **Watch it fail first** when the test comes before the code: it must fail, not error, and
  fail with the expected message. A test that already passes tests existing behavior.
- **Real code over mocks.** Mock only what cannot run in the test: network, clock, payment
  gateway, the engine. Asserting a mock was called tests the mock.
- **Arrange–Act–Assert**, one assertion concept per test, named after the behavior
  (`retries a failed operation 3 times`), not the function.
- **Minimal code to green.** No options, retries, or configurability the test does not demand.

## The pyramid

Unit — fast, isolated, no I/O — the majority. Integration — boundaries: DB, external APIs,
adapters, engine bridges. End-to-end — few, critical paths only.

For deterministic simulations, replay tests (recorded inputs re-run, state asserted) are the
highest-leverage regression net.

## Running

Before calling a change done, run the project's test command, not only the file you touched.
Every failure in that run goes in the report by name, including ones you did not cause.
