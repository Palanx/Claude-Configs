---
name: gamedev-client-architecture
description: Architecture guidance for game client code and simulation systems, centered on deciding what should depend on the game engine and what must stay engine-agnostic. Use this skill whenever the user is designing, reviewing, or refactoring gameplay systems, simulation logic, or client-side game code — including questions like "should this depend on the engine?", "how do I decouple this from Unreal/Unity?", "engine-agnostic design", "ECS or components?", "fixed or variable timestep?", or any request to structure game logic, game state, or gameplay features. Trigger even if the user doesn't say "architecture" — proposing a class design for a gameplay system counts.
---

# Game Client & Simulation Architecture

Guidance for structuring client-side game code, with the engine boundary as the first and most consequential decision. Generic engineering rules (SOLID, naming, commits, clean code) live in `global-engineering-guidelines`; system-level tradeoff scoring lives in `architecture-evaluation`. This skill covers only what is specific to games. When those skills are unavailable, apply standard engineering judgment; do not restate their content here.

## 1. Engine Boundary — evaluate BEFORE proposing any design

Every module in a proposal must be classified. Do this explicitly, not implicitly — state the classification in your answer when it isn't obvious.

**Engine-agnostic (the default for logic):**
- Simulation logic, game rules, domain state
- Algorithms, state machines, progression/economy systems
- Anything that needs unit tests running without booting the engine

**Engine-side (justified, not lazy):**
- Presentation: rendering, VFX, animation, audio, UI
- Input handling, asset loading/management
- Engine physics integration
- Any code where wrapping the engine costs more than it buys

**The boundary itself:** interfaces and adapters are defined by the domain; the engine implements them. Never the reverse — domain code must not know engine types, lifecycles, or threading assumptions.

### Abstraction test (apply per module, never globally)

Abstract only if at least one answer is YES:

1. Does it need unit tests that run without the engine?
2. Is a second real implementation plausible — another engine, a headless simulation, offline replay? "Plausible" means it exists on the roadmap, not "maybe someday."
3. Does the logic have value independent of the engine (game rules, domain knowledge)?

If all three are NO → use the engine directly. Indirection without a use case is debt, not investment. The goal is abstraction that pays for itself, not speculative extensibility layered onto something simple.

Note that question 2 passes more often than it looks for simulation code: headless simulation for testing and deterministic replay are legitimate, common second implementations in game development. It almost never passes for presentation code. This is why simulation logic converges on engine-agnostic while rendering does not — the test produces the right answer without dogma.

### Documented exceptions — do not wrap these

- **Math types** (`FVector`, `Vector3`, quaternions, transforms): wrapping them creates conversion friction in hot paths and buys nothing. Use the engine's types throughout, or a single well-known math library on the agnostic side with conversion only at the boundary — never a bespoke wrapper. In Unity DOTS code, use `Unity.Mathematics` types directly for Burst compatibility.
- **Unreal reflection/GC**: anything that must participate in the `UObject` system (reflection, garbage collection, serialization, Blueprint exposure) cannot live outside the engine without costly duplication. Keep the `UObject` shell engine-side and delegate logic to plain C++ classes it owns.
- **Unity lifecycle/serialization**: `MonoBehaviour` and `ScriptableObject` shells stay engine-side; delegate logic to plain C# classes they own. Two Unity-specific traps for domain code: `UnityEngine.Object` overrides `==` so destroyed objects compare equal to `null` — never let agnostic code hold engine object references; and Unity's serializer pressures data classes toward engine idioms (`[SerializeField]` fields) — keep serialized data in the shell, not the domain.
- **Burst/Jobs (Unity)**: Burst-compiled code cannot touch managed objects; either treat it as engine-side despite being plain-looking C#, or design the agnostic module Burst-compatible from the start (structs, no allocations) — pick one deliberately.
- **Engine containers in engine-side code**: inside engine-side modules, use the engine's containers and idioms (`TArray`/`TMap` in Unreal, `NativeArray` in Unity Jobs). Fighting the engine inside its own territory is friction without benefit.
- **Frame-coupled utilities**: debug drawing, profiling markers, logging macros. Wrap only if logs must be captured in headless runs.

## 2. Simulation Core

**Timestep:** fixed timestep for simulation, variable for presentation, interpolation between the two. Recommend variable-only simulation exclusively for games where determinism and physics stability genuinely don't matter (most pure-UI or casual games) — and say so explicitly when making that call.

**Determinism requirements** (only when replay, lockstep, or reproducible tests are goals — check before imposing them):
- Explicit, stable update order across systems; never iterate unordered containers to drive simulation
- Seeded RNG owned by the simulation, one stream per concern; never engine or global RNG inside sim code
- Floating point: same-platform determinism is achievable with care; cross-platform bit-exactness is a major commitment (fixed-point or strict FP control). Ask which one the user actually needs before designing for it.
- All simulation inputs (player commands, time, RNG seeds) must flow through a capturable path — this is what makes replay testing possible.
- Engine physics (PhysX in Unity, Chaos in Unreal) is not deterministic by default. A deterministic sim either avoids engine physics for authoritative state or adopts a deterministic physics library on the agnostic side — flag this early, it is a common late-discovery failure.

**State ownership:** simulation owns authoritative state; presentation holds derived, disposable copies. One-way data flow sim → presentation. Events/commands flow presentation → sim, never direct mutation.

## 3. Structural Patterns — when each pays its cost

- **Plain OOP / composition:** the default for small-to-mid systems. Choose it unless a concrete pressure below exists. It's the cheapest to write, read, and test.
- **Engine component model** (Unreal ActorComponents, Unity MonoBehaviours): for engine-side glue and presentation. Do not put core game rules directly in engine components — that welds logic to the engine and fails the boundary test.
- **ECS:** justified by (a) thousands of homogeneous entities with cache-sensitive iteration, or (b) determinism/replay architectures that benefit from flat, serializable state. Not justified by fashion or by "it might scale later." Partial adoption is valid — an ECS island for the hot system inside an otherwise OOP codebase.
- **State machines / behavior trees:** prefer explicit FSMs for gameplay states with few transitions; behavior trees when designers need to author behavior. Both belong on the agnostic side.

When comparing these for a specific system, hand the tradeoff scoring to `architecture-evaluation`; this section supplies the game-specific selection pressures.

## 4. Simulation / Presentation Separation

- Presentation reads sim state through a narrow query surface (snapshot, read-only view, or event stream) — not by reaching into sim internals.
- Visual smoothing (interpolation, animation blending, tweens) lives entirely in presentation. If a designer asks to "make it feel smoother," the sim should not change.
- Common failure to flag in reviews: gameplay logic keyed to animation events or frame rate. Both invert the dependency and break determinism.
- Sim time and wall-clock time are different clocks. Pause, slow-motion, and fast-forward manipulate sim time; UI and audio usually follow wall clock. Mixing them is a recurring bug source — name which clock each system uses.

## 5. Client-Server (conditional — only when a server exists)

Do not introduce networking abstractions into a single-player architecture "just in case"; that fails the abstraction test. When multiplayer is on the roadmap:

- The sim/presentation boundary from §4 is the same boundary prediction needs: client prediction re-runs the agnostic simulation with local inputs, which is only possible if the sim is engine-free and deterministic. This is the strongest practical payoff of the boundary discipline.
- Standard toolkit: client-side prediction + server reconciliation for the local player, snapshot interpolation for remote entities, input buffering with sequence numbers. Recommend against rolling custom transport; the architecture concern is where prediction hooks into the sim, not sockets.
- Authoritative server changes state ownership: client sim state becomes a prediction, not truth. Design the reconciliation path (rollback + re-simulate, or correction blending) before implementing prediction, not after.

## 6. Testing Consequences of the Boundary

Only what follows from this architecture; general test strategy belongs to the generic testing skill.

- Engine-agnostic modules get plain unit tests in the native test framework — no engine boot, no play mode. If a test needs the engine, the module is misclassified or the boundary leaks; treat that as a design smell, not a testing problem.
- Deterministic sims unlock **replay-based testing**: record input streams, re-run the sim, assert on final or checkpointed state. This is the highest-leverage regression net for simulation-heavy games and should be proposed whenever §2's determinism requirements are already being met.
- Float assertions use tolerances (ULP or epsilon appropriate to magnitude) — exact equality only in bit-exact deterministic builds, where it is not just allowed but the point.
- Engine-side code gets thin smoke/integration tests only; don't chase coverage there — the boundary exists precisely so the valuable logic is testable elsewhere.