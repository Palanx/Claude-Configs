---
name: architecture-evaluation
description: >
  Evaluate and optimize software architecture through structured scope analysis and a
  multi-dimensional tradeoff table. ALWAYS use this skill when the user asks to design,
  review, propose, validate, or discuss a system architecture, even at a high level.
  Trigger also when the user mentions: "how should I structure this", "what pattern fits here",
  "monolith vs microservices", "should I split this", "is this scalable", "architecture for X",
  or any phrase implying a structural or system-level decision. Covers scope assessment,
  candidate architecture selection, and tradeoff scoring across maintainability, portability,
  complexity, reusability, performance, scalability, and coupling.
---

# Architecture Evaluation Skill

> Before proposing any architecture, run this evaluation. Never recommend a structural approach
> without completing the scope assessment and tradeoff analysis. Architecture decisions are
> permanent in practice — get them right upfront.

---

## Phase 1 — Scope Assessment

Ask or infer the following. If the conversation already contains answers, extract them.
Do not ask for what can be reasonably inferred.

### 1.1 Project Dimensions

| Dimension | Questions to resolve |
|---|---|
| **Domain** | What is the core business problem? What are the bounded contexts? |
| **Team** | How many developers? Seniority distribution? Single team or multiple? |
| **Timeline** | MVP deadline? Long-term runway? Is this exploratory or production? |
| **Scale** | Expected users/requests at launch? In 12 months? In 3 years? |
| **Data** | Read-heavy or write-heavy? Relational, document, time-series? Volume? |
| **Integrations** | External APIs, third-party services, legacy systems? |
| **Compliance** | Regulatory constraints (GDPR, HIPAA, SOC2)? Audit requirements? |
| **Deployment** | Cloud provider? On-prem? Serverless? Containerized? |
| **Budget** | Infrastructure cost sensitivity? Operational overhead tolerance? |

### 1.2 Scope Classification

After gathering dimensions, classify the project:

```
Scope: [Small | Medium | Large | Enterprise]

Small    → 1–3 devs, single domain, < 10k users, low integration surface
Medium   → 3–10 devs, 2–4 bounded contexts, 10k–500k users, moderate integrations
Large    → 10+ devs, multiple teams, 500k+ users, significant integrations
Enterprise → Multiple org units, compliance-critical, global scale, legacy constraints
```

This classification gates which candidate architectures are viable (see Phase 2).

---

## Phase 2 — Candidate Architecture Selection

Based on scope classification, surface only viable candidates. Do not propose architectures
outside the viable range without explicit justification.

| Architecture Style | Viable Scope | Core Assumption |
|---|---|---|
| Monolith (Layered) | Small | Single team, fast iteration, low ops overhead |
| Modular Monolith | Small–Medium | Growth anticipated; want boundaries without ops cost |
| Vertical Slice | Medium | Feature teams, independent deployment not yet needed |
| Microservices | Large–Enterprise | Independent scaling, independent deployment, polyglot teams |
| Event-Driven | Medium–Enterprise | Loose coupling, async workflows, high throughput |
| Serverless | Small–Large | Spiky load, low ops, stateless operations |
| Hexagonal / Ports & Adapters | Any | Testability, infrastructure independence, DDD alignment |
| CQRS + Event Sourcing | Large–Enterprise | Audit trail, read/write asymmetry, complex domain |
| Cell-Based | Enterprise | Blast radius isolation, multi-region, massive scale |

**Default recommendation:** When scope is Small or Medium, default to **Modular Monolith**.
It provides clean internal boundaries without the operational overhead of distributed systems.
Justify any deviation from this default explicitly.

---

## Phase 3 — Tradeoff Table

For each candidate architecture, score every dimension on a scale of 1–5.
Score from the perspective of **this specific project's constraints**, not in the abstract.

### Scoring Key

```
5 = Excellent fit / no friction
4 = Good fit / minor friction
3 = Neutral / manageable friction
2 = Poor fit / significant friction
1 = Severe misalignment / high risk
```

### Tradeoff Dimensions

| Dimension | Definition | What drives the score |
|---|---|---|
| **Maintainability** | How easy is it to change, fix, and extend over time? | Module boundaries, cognitive load, documentation needs, onboarding time |
| **Portability** | How easily can the system move across environments, clouds, or runtimes? | Infrastructure coupling, vendor lock-in, containerization, abstraction layers |
| **Complexity** | How much cognitive and operational overhead does this introduce? | Distributed systems tax, deployment pipelines, debugging surface, learning curve |
| **Reusability** | How much of the code/logic can be shared across contexts or products? | Service boundaries, shared libraries, API design, domain model generality |
| **Performance** | How well does it meet latency, throughput, and resource efficiency needs? | Network hops, serialization cost, caching opportunities, data locality |
| **Scalability** | How well does it handle growth in users, data, or load? | Horizontal scaling, statelessness, partitioning strategy, bottleneck surface |
| **Coupling** | How isolated are components from each other's internals? | Direct dependencies, shared databases, synchronous call chains, data contracts |

### Tradeoff Table Template

```
┌─────────────────────┬──────────────────┬──────────────────┬──────────────────┐
│ Dimension           │ [Architecture A] │ [Architecture B] │ [Architecture C] │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Maintainability     │       X/5        │       X/5        │       X/5        │
│ Portability         │       X/5        │       X/5        │       X/5        │
│ Complexity          │       X/5        │       X/5        │       X/5        │
│ Reusability         │       X/5        │       X/5        │       X/5        │
│ Performance         │       X/5        │       X/5        │       X/5        │
│ Scalability         │       X/5        │       X/5        │       X/5        │
│ Coupling            │       X/5        │       X/5        │       X/5        │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ TOTAL               │      XX/35       │      XX/35       │      XX/35       │
├─────────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Key Risk            │ [main risk]      │ [main risk]      │ [main risk]      │
│ Key Strength        │ [main strength]  │ [main strength]  │ [main strength]  │
└─────────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

After scoring, annotate each score with a one-line rationale.
Do not present raw scores without explanation.

### Weighted Scoring (Optional — for high-stakes decisions)

If the project has clearly prioritized dimensions, apply weights:

```
Example: E-commerce platform (performance and scalability are critical)
  Maintainability  × 1.0
  Portability      × 0.8
  Complexity       × 1.0
  Reusability      × 0.8
  Performance      × 1.5   ← critical
  Scalability      × 1.5   ← critical
  Coupling         × 1.0

Weighted Total = Σ(score × weight)
```

Ask the user if they want to apply weights before computing.

---

## Phase 4 — Recommendation

After completing the tradeoff table, produce a structured recommendation:

```
## Architecture Recommendation

**Recommended:** [Architecture Name]
**Runner-up:** [Architecture Name] (viable if [condition])
**Eliminated:** [Architecture Name] — reason: [one line]

**Decision rationale:**
[2–4 sentences connecting scope constraints to the winning architecture's strengths
and explaining why the tradeoffs are acceptable given this project's context.]

**Key risks to monitor:**
- [Risk 1] → Mitigation: [approach]
- [Risk 2] → Mitigation: [approach]

**Evolution path:**
[What would trigger a migration to a more complex architecture, and what would that look like?]
```

---

## Phase 5 — Architecture Decision Record (ADR)

For every architecture decision, offer to produce a minimal ADR:

```markdown
# ADR-[NNN]: [Decision Title]

**Date:** YYYY-MM-DD
**Status:** Proposed | Accepted | Deprecated

## Context
[What situation forced this decision?]

## Decision
[What was decided and why?]

## Tradeoff Summary
[Reference the tradeoff table scores and key rationale.]

## Consequences
**Positive:** [what gets better]
**Negative:** [what gets harder]
**Risks:** [what to watch]

## Review Trigger
[Under what conditions should this decision be revisited?]
```

Always offer an ADR after a final recommendation is accepted. Architecture decisions that
aren't recorded are decisions that will be relitigated in every future discussion.

---

## Execution Rules

1. **Never skip Phase 1.** A scope-less architecture recommendation is noise.
2. **Never recommend a single option without a tradeoff table.** One option isn't a decision.
3. **Never score without rationale.** Every cell needs a one-liner justification.
4. **Surface assumptions explicitly.** "I'm assuming X — correct me if wrong."
5. **Flag irreversible decisions.** Some choices (DB engine, event bus, API contract) are
   extremely expensive to change. Mark them as high-stakes before the user commits.
6. **Integrate with global-engineering-guidelines.** Pattern selection (§5) and architecture
   layers (§4 Clean Architecture) from that skill are inputs here, not alternatives.
