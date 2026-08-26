---
name: global-engineering-guidelines
description: >
  Apply these non-negotiable engineering standards to ANY coding task, regardless of size,
  language, or context. Covers clean code (naming, function/class design, comments, magic
  values), SOLID, Clean Architecture layering and the dependency rule, the design pattern
  consultation protocol, Git commit discipline (Conventional Commits, branch strategy), error
  handling categories, testing philosophy (AAA, test pyramid), security baseline, and the
  communication protocol for proposing approaches before acting. ALWAYS use this skill before
  writing code, reviewing code, designing a class or module, choosing a design pattern, drafting
  a commit message, or starting any implementation task — "quick fixes" don't exempt a task from
  these rules. For system-level architecture scoping and tradeoff scoring between architecture
  styles, defer to the architecture-evaluation skill — this skill supplies the layering rules
  and pattern protocol that feed into that evaluation, not a substitute for it.
---

# Global Engineering Guidelines

> These rules are non-negotiable. Every task — regardless of size, language, or context —
> follows these principles. "Quick fixes" and "just this once" exceptions do not exist.

---

## 1. Core Mentality

- **Build it right the first time.** Shortcuts compound into debt. Production-quality from day one.
- **Architecture is non-negotiable.** Even scripts and small utilities follow structural discipline.
- **Clarity over cleverness.** Code is written once, read many times.
- **Ask, don't assume.** When a decision has multiple valid paths, surface the options. Never pick silently.

---

## 2. Clean Code Principles

### Naming
- Names must reveal intent. If a name requires a comment to explain it, rename it.
- Use domain language (ubiquitous language). Names should match the business concept, not the technical implementation.
- Avoid abbreviations unless universally known in the domain (e.g., `id`, `url`, `api`).
- Boolean names are assertions: `isActive`, `hasPermission`, `canRetry` — never `flag`, `status`, `check`.

### Functions & Methods
- **Single Responsibility:** One function does one thing. If you need "and" to describe it, split it.
- **Small is correct.** Functions should fit on one screen. If they don't, decompose.
- **No side effects** unless the function name explicitly communicates mutation (e.g., `saveUser`, `clearCache`).
- **Max arguments: 3.** Beyond that, introduce a parameter object / value object.
- **Command-Query Separation:** A function either returns a value OR changes state — never both.

### Classes & Modules
- A class is responsible for one concept. One reason to change.
- Constructors only assign. No logic, no I/O, no side effects in constructors.
- Prefer composition over inheritance. Inheritance is a last resort.
- Keep public surface minimal. Expose only what consumers need.

### Comments
- Comments explain **why**, never **what**. The code explains what.
- Outdated comments are worse than no comments. Delete or update, never leave stale.
- TODOs must include a ticket/issue reference: `// TODO(#123): refactor after migration`.

### Magic Values
- Zero tolerance for magic numbers or magic strings in logic.
- All constants are named, typed, and live in a dedicated constants/config layer.

---

## 3. SOLID Principles

| Principle | Rule |
|---|---|
| **S** — Single Responsibility | Every module/class has one reason to change |
| **O** — Open/Closed | Open for extension, closed for modification. Use abstractions, not conditionals |
| **L** — Liskov Substitution | Subtypes must be substitutable for their base types without breaking behavior |
| **I** — Interface Segregation | Many specific interfaces over one general-purpose interface |
| **D** — Dependency Inversion | Depend on abstractions, not concretions |

Before writing any class or module, explicitly identify which SOLID principle each design
decision supports. If a decision violates one, flag it and propose an alternative.

---

## 4. Clean Architecture

### Layer Structure (innermost → outermost)

```
┌─────────────────────────────────────────┐
│           Frameworks & Drivers          │  ← Web, DB, UI, external libs
├─────────────────────────────────────────┤
│          Interface Adapters             │  ← Controllers, Presenters, Gateways
├─────────────────────────────────────────┤
│           Application Layer            │  ← Use Cases / Application Services
├─────────────────────────────────────────┤
│             Domain Layer                │  ← Entities, Value Objects, Domain Services
└─────────────────────────────────────────┘
```

### Dependency Rule (absolute)
- Dependencies point **inward only**. Domain knows nothing about the outside world.
- The domain layer has **zero imports** from frameworks, ORMs, HTTP libs, or external services.
- Cross-layer communication happens through **interfaces/ports defined in the inner layer**.

### Key Patterns
- **Repository Pattern:** Data access is always behind an interface.
- **Use Case / Interactor:** Each business operation is a dedicated use case class.
- **DTOs at boundaries:** No entity leakage across layers.
- **Dependency Injection:** Dependencies are injected, never instantiated inside the consumer.

### Default Structure
- Default: **Modular Monolith** — one deployable unit, hard internal boundaries by domain context.
- Each module owns its own models, use cases, and data access. No cross-module direct imports.
- Cross-module communication goes through a defined interface or event — never a direct model import.

> **Handoff:** This section defines the layering rules and the default structure assumption.
> When a decision requires full scope assessment and a tradeoff table across architecture
> styles (e.g., monolith vs. microservices vs. event-driven), run the **architecture-evaluation**
> skill — it consumes these layering rules as a fixed constraint, not as a competing alternative.

---

## 5. Design Pattern Protocol

> Never pick a pattern unilaterally. Surface options with tradeoffs and wait for confirmation.

### Required consultation scenarios
- Creational pattern selection (Factory, Builder, Prototype, Singleton)
- Structural patterns (Adapter vs Facade vs Decorator)
- Behavioral patterns (Strategy vs State vs Command)
- Event-driven vs request-driven communication
- Sync vs async processing model
- Caching strategy selection
- Error handling approach (exceptions vs result types vs error codes)

### How to surface options

```
Pattern decision required: [brief description of the problem]

Option A — [Pattern Name]
  + [advantage]
  - [tradeoff]
  Best for: [context where this wins]

Option B — [Pattern Name]
  + [advantage]
  - [tradeoff]
  Best for: [context where this wins]

→ Which approach fits your context?
```

Never proceed past a pattern decision without an explicit answer.

---

## 6. Git & Version Control Discipline

### Absolute rules
- **Never commit automatically.** Every commit requires explicit user approval.
- **Never commit with a generic message.** Commit messages are authored, not generated.
- **Never push without explicit instruction.**
- **Never force push** unless explicitly requested and consequences are acknowledged.

### Commit message standard (Conventional Commits)

```
<type>(<scope>): <imperative summary, max 72 chars>

[body: explain WHY, not WHAT. Wrap at 72 chars.]

[footer: references, breaking changes]
```

**Types:** `feat` | `fix` | `refactor` | `test` | `docs` | `chore` | `perf` | `ci` | `build`

**Examples:**
```
feat(auth): add refresh token rotation on session renewal
fix(payments): prevent double-charge on network timeout retry
refactor(users): extract email validation into domain value object
```

### Branch strategy
- Confirm branching strategy before creating branches.
- Default: `feature/`, `fix/`, `refactor/`, `chore/` prefixes from main/develop.
- Never work directly on `main` or `master` without explicit consent.

---

## 7. Error Handling

- Errors are domain citizens. Define error types in the domain layer.
- Fail fast, fail loud in development. Silence only in production, with proper logging.
- Never swallow exceptions with empty catch blocks.
- **Error categories:**
  - Domain errors → handled, user-facing
  - Infrastructure errors → logged, retried or surfaced
  - Programming errors → crash fast, fix at root
- Propose error handling strategy before implementing complex flows.

---

## 8. Testing Philosophy

- Tests are first-class citizens. Same quality standards as production code.
- Test behavior, not implementation.
- **Pyramid model:**
  - Unit: fast, isolated, no I/O — the majority
  - Integration: verify boundaries (DB, APIs, adapters)
  - E2E: minimal, high-value critical paths only
- Arrange-Act-Assert (AAA) structure. One assertion concept per test.
- No tests, no merge. New behavior requires new tests.

---

## 9. Security Baseline

- **Zero secrets in code.** All credentials live in environment variables or secret managers.
- **Zero trust by default.** Validate all inputs at system boundaries.
- **Principle of least privilege.** Services and modules have only the permissions they need.
- Flag any operation touching auth, permissions, PII, or external data as security-sensitive.

---

## 10. Communication Protocol

### Before starting a task
1. Confirm understanding of the requirement.
2. Identify architectural impact (which layers are affected?).
3. Surface any pattern decisions needed.
4. Propose the approach before writing code.

### During implementation
- If a decision point arises mid-task, stop and surface it. Don't pick silently.
- Flag any deviation from these guidelines as a deliberate tradeoff, not a shortcut.

### Agentic handoffs (TOON)
- **T (Task):** Specific objective.
- **O (Outcome):** Expected result / success criteria.
- **O (Objects):** State, data, and context passed forward.
- **N (Next):** Immediate next action.

### Tone
- Technical precision over politeness. No filler phrases.
- Assume senior-level expertise unless demonstrated otherwise.
- Disagreements are technical, not personal. Argue with evidence.

---

## 11. What Requires Explicit Approval

| Action | Why |
|---|---|
| `git commit` | Message and scope must be intentional |
| `git push` | Irreversible without force-push consequences |
| Deleting files or directories | Data loss risk |
| Modifying `.env` or config files | Environment impact |
| Installing new dependencies | Supply chain and bundle size impact |
| Changing public API contracts | Breaking change risk |
| Running database migrations | Schema changes are permanent |
| Deploying to any environment | Production impact |
