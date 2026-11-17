# Architecture: SOLID, Clean Architecture, design patterns

Applies in full at the **Core** tier. At **Evolving**, keep the dependency rule and skip the
ceremony. At **Throwaway**, none of this applies — one file that works is correct.

## SOLID

| Principle | Rule |
|---|---|
| **S** — Single Responsibility | One reason to change per module or class |
| **O** — Open/Closed | Open for extension, closed for modification. Abstractions, not conditionals |
| **L** — Liskov Substitution | Subtypes substitutable for their base types without breaking behavior |
| **I** — Interface Segregation | Many specific interfaces over one general-purpose one |
| **D** — Dependency Inversion | Depend on abstractions, not concretions |

Before writing a Core-tier class or module, name which principle each design decision
serves. A decision that violates one is flagged and an alternative proposed. "Why is this an
interface?" needs an answer other than "that's the pattern".

## Clean Architecture

```
┌─────────────────────────────────────────┐
│           Frameworks & Drivers          │  ← Web, DB, UI, engine, external libs
├─────────────────────────────────────────┤
│          Interface Adapters             │  ← Controllers, Presenters, Gateways
├─────────────────────────────────────────┤
│           Application Layer             │  ← Use Cases / Application Services
├─────────────────────────────────────────┤
│             Domain Layer                │  ← Entities, Value Objects, Domain Services
└─────────────────────────────────────────┘
```

### Dependency rule (absolute at Core tier)

- Dependencies point **inward only**. The domain knows nothing about the outside world.
- The domain layer has **zero imports** from frameworks, ORMs, HTTP libraries, engines, or
  external services. An ORM annotation on an entity is a violation, not a convenience.
- Cross-layer communication goes through **interfaces/ports defined in the inner layer**; the
  outer layer supplies the adapter.

### Key patterns

- **Repository** — data access behind an interface.
- **Use case / interactor** — one class per business operation. No fat controllers, no god
  services.
- **DTOs at boundaries** — no entity leakage across layers.
- **Dependency injection** — dependencies are injected, never constructed by their consumer.

### Default structure

- **Modular monolith**: one deployable unit, hard internal boundaries by domain context.
  Distributed services only when a named constraint (independent scaling, team autonomy,
  isolation) forces it.
- Each module owns its models, use cases, and data access. Cross-module communication goes
  through a published interface or an event — never a direct model import.

Choosing between architecture styles (monolith vs services vs event-driven) is a scoping and
tradeoff decision: run the `architecture-evaluation` skill. It takes these layering rules as a
fixed constraint, not as a competing alternative. For game clients, the engine boundary rules
in `gamedev-client-architecture` come first.

## Design pattern protocol

Never pick a pattern unilaterally when several are valid and the choice lasts. Consult on:

- Creational (Factory, Builder, Prototype, Singleton)
- Structural (Adapter vs Facade vs Decorator)
- Behavioral (Strategy vs State vs Command)
- Event-driven vs request-driven · sync vs async · caching strategy
- Error model (exceptions vs result types vs error codes) · storage or schema shape

```
Decision required: [problem in one line]

Option A — [name]
  + [advantage]
  - [tradeoff]
  Best for: [context where it wins]

Option B — [name]
  + [advantage]
  - [tradeoff]
  Best for: [context where it wins]

Recommendation: [A or B] because [reason]. Which fits your context?
```

Always give a recommendation — a survey without a position is not help. Do not proceed past
the decision without an answer.
