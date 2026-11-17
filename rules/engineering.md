# Engineering baseline

Applies to every task that writes or changes code. Situational detail — architecture,
testing, verification, review — lives in the `engineering-guidelines` skill's references.

## Right-size before structuring

Classify the artifact first. The tier decides how much structure is correct; over-building a
throwaway and under-building a core module are the same mistake.

| Tier | What it is | Structure |
|---|---|---|
| Throwaway | Spike, one-shot script, run-once migration, disposable prototype | Minimum that works. One file is fine. No layers, ports, DTOs |
| Evolving | Shared utility, editor tool, internal tool, bounded feature likely to change | Clear functions and modules. Abstract on the second occurrence, not the first |
| Core | Domain or simulation core, public API, long-lived or multi-team code | Full SOLID and Clean Architecture — `engineering-guidelines/references/architecture.md` |

- The floor is tier-independent: clarity, validation at trust boundaries, error handling that
  prevents data loss, security, accessibility.
- Promote, don't pre-build. When an artifact starts changing often, move it up a tier then.
- A deliberate shortcut with a known limit carries a comment naming the ceiling and the upgrade
  path. Unmarked shortcuts are debt; marked ones are decisions.
- Tier ambiguous ("will this script become a tool?") → ask.

## Before writing code

1. **Already in this codebase?** An existing helper, type, or pattern → reuse it.
   Re-implementing what lives a few files over is the most common slop.
2. **Find every caller** of a function before changing it. One guard in the shared function is
   the smaller diff and the root-cause fix; patching only the reported path leaves sibling
   callers broken.
3. No abstraction without a second caller: no interface with one implementation, no factory for
   one product, no config for a value that never changes.

## Clean code

- Names reveal intent, in domain language. No abbreviations beyond universal ones (`id`, `url`).
  Booleans are assertions: `isActive`, `canRetry`.
- A function does one thing, fits on a screen, takes at most 3 arguments, and either returns a
  value or changes state — never both. Mutation is in the name (`saveUser`).
- Constructors only assign. Composition over inheritance. Minimal public surface.
- Comments explain why, never what. Stale comments get deleted. TODOs carry a reference:
  `// TODO(#123): ...`.
- No magic values in logic: named, typed constants.

## Errors

- Never swallow an exception. An empty catch block is forbidden; at minimum log, then decide.
- Fail fast and loud in development; silence only in production, and only with logging.
- Domain errors → handled and user-facing. Infrastructure errors → logged, retried or
  surfaced. Programming errors → crash fast, fix at the root.
- Propose the error strategy before implementing a complex flow.

## Security

- Zero secrets in code: environment variables or a secret manager.
- Validate every input at system boundaries, whatever its source. Least privilege.
- Say explicitly when a change touches auth, permissions, PII, or external data.

## Commits

Conventional Commits: `<type>(<scope>): <imperative summary, max 72 chars>`, body explains
why, not what. Types: `feat` `fix` `refactor` `test` `docs` `chore` `perf` `ci` `build`.
When to commit or push at all is governed by `interaction-protocol.md`.
