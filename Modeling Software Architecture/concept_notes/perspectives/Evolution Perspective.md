---
status: researching
tags: [c4-model, architecture, perspective, evolution, changeability, udemy, fundamentals]
created: 2026-07-10
updated: 2026-07-10
difficulty: intermediate
---

# 🧠 Topic: Evolution Perspective

> [!abstract] Executive Summary (ELI5)
> This perspective asks: **"can we change the system without breaking it — and without rewriting it?"** It's about changeability, extensibility, and adaptability over the system's lifetime. Software that can't evolve dies — either it rots into an unmaintainable "big ball of mud," or every change becomes so risky that the team freezes. Applying this perspective means designing seams, loose coupling, and clear extension points so future changes are local, not system-wide.

> [!warning] The #1 myth — "We'll design for flexibility later"
> Flexibility retrofitted later is called a rewrite. The systems that evolve well are the ones designed *from the start* with clear module boundaries, stable interfaces, and decoupled change axes. Evolution isn't a phase; it's a property of the architecture. If coupling is high on day one, every future change is a surgery.

---

## 🎯 The "Why"

- **Problem Solved:** The cost of change grows over time in a poorly-evolvable system — each new feature touches more code, breaks more tests, and takes longer. Eventually the system becomes "too risky to change," and the only option is a rewrite. The Evolution perspective keeps the cost of change flat by designing for it.
- **Alternatives:** "YAGNI — don't build flexibility until you need it." YAGNI is about not building *features* prematurely, not about ignoring *changeability*. You don't pre-build features, but you *do* keep seams clean so features can be added cheaply later.
- **Trade-offs:** Excessive abstraction ("design for every possible future") creates accidental complexity and makes the current behavior harder to understand. The discipline is to design for *likely* change axes, not hypothetical ones.

## 💡 Mental Model & Analogy

- **Analogy:** A building designed for evolution has **modular partitions** instead of load-bearing walls between rooms — you can reconfigure the layout without demolishing the structure. A building where every wall is load-bearing can't be changed without risking collapse. In software, the "load-bearing walls" are tight coupling; the "modular partitions" are clean interfaces and loose coupling.

```mermaid
graph TD
    Change[New requirement] --> Q{Coupling?}
    Q --|low: clean seams| Local["Local change<br/>(one module)"]
    Q --|high: tangled| Ripple["Ripple effect<br/>(many modules)"]
    Local --> Safe[Low risk, fast]
    Ripple --> Risky[High risk, slow, buggy]
```

## 🛠️ Core Principles

1. **Design for likely change, not every change.** Identify the axes along which the system is likely to evolve (new payment providers, new regions, new UIs) and decouple *those*. Don't abstract everything — abstract the change.
2. **Coupling is the enemy of evolution.** The more one component knows about another's internals, the more a change ripples. Loose coupling (stable interfaces, async contracts) localizes change.
3. **Stable interfaces, swappable implementations.** Behind a stable interface, the implementation can change freely. The interface is the contract; everything behind it is free to evolve.
4. **Bounded contexts isolate change.** In DDD terms, changes within one bounded context don't bleed into others. Clear ownership boundaries (see [[Key Elements of Architecture View]]) keep change local.
5. **Reversibility over prediction.** When unsure, prefer decisions that are *easy to reverse* over decisions that lock you in. Defer irreversible choices (a database engine, a cloud vendor) until you have to commit.

## 🏗️ Key Concerns (applied to views)

### Applied to the Container/Component Views
*Are the boundaries clean enough to change one thing without touching others?*
- **Cohesion and coupling** — do components have single, clear responsibilities? Are dependencies minimized and explicit?
- **Interface stability** — do components expose stable contracts, or do callers reach into internals?
- **Dependency direction** — do dependencies point toward stability (domain core), not away from it (toward volatile details)?

### Applied to the Information View
*Can the data model evolve without breaking consumers?*
- **Backward-compatible schema changes** — adding fields is safe; removing/renaming is not. Plan for additive evolution.
- **Event schema versioning** — if events are the integration mechanism, their schemas must evolve without breaking consumers.
- **Bounded contexts** — does one service's data model bleed into another's? (The shared-database connector from [[Key Elements of Architecture View]] is the highest-coupling, least-evolvable integration.)

### Applied to the Deployment View
*Can you deploy part of the system independently?*
- **Independent deployability** — can you ship a change to one service without redeploying everything? This is a primary driver of the microservices pattern.
- **Feature flags** — decouple *release* from *deployment*; turn features on/off without code changes.

### Extensibility patterns (the toolkit)
- **Plugin/strategy patterns** — new behavior added by implementing an interface, not modifying existing code (Open-Closed Principle).
- **Event-driven extension** — new capabilities subscribe to events without the publisher knowing they exist.
- **Configuration-driven behavior** — rules, thresholds, routing moved to config so they change without code.

## ↔️ How the Perspective Connects

| Applied to | The question it asks |
|---|---|
| [[Key Elements of Architecture View]] | Are components cohesive and loosely coupled? Are connectors stable? |
| [[Information Viewpoint]] | Can schemas evolve backward-compatibly? Is data ownership bounded? |
| [[Deployment Viewpoint]] | Can parts be deployed independently? Are releases decoupled from deploys? |
| [[Perspective]] siblings | Overlaps with [[Development Resource Perspective]] (evolvable code is maintainable code); tension with [[Cost Perspective]] (clean seams cost effort upfront). |

## ⚠️ Pitfalls & Anti-patterns

- **Big ball of mud:** no clear structure; every change ripples unpredictably. The endpoint of ignored evolution.
- **Over-engineering for hypothetical change:** abstracting "in case we need it" for changes that never come. Adds accidental complexity; makes current behavior harder to read.
- **Shared databases as integration:** the least-evolvable coupling — every schema change risks breaking every consumer. (See [[Key Elements of Architecture View]] pitfall.)
- **Reaching into internals:** callers depending on another component's implementation details, not its interface. Every internal change breaks them.
- **Irreversible early decisions:** locking in a database, framework, or vendor before understanding the real constraints. Prefer reversible choices until you must commit.
- **No extension points:** every new feature requires modifying existing, stable code (violating Open-Closed). New behavior should be *added*, not *inserted*.
- **Confusing YAGNI with "don't design for change":** YAGNI says don't build unneeded *features*; it doesn't say ignore *changeability*. Keep seams clean regardless.

## 🔗 Deep Connections

- **Parent Topic:** [[Perspective]], [[Viewpoint]]
- **Most relevant viewpoints:** [[Key Elements of Architecture View]] (coupling/cohesion), [[Information Viewpoint]] (schema evolution), [[Deployment Viewpoint]] (independent deployability)
- **Sibling perspectives:** [[Development Resource Perspective]] (maintainability), [[Cost Perspective]] (upfront design effort)
- **Related:** [[System Architecture]] (Maintainability/Evolvability are Quality Attributes)
- **Prerequisites:** [[Perspective]], [[Key Elements of Architecture View]].

## ❓ Active Recall Questions

> [!faq]- What does the Evolution Perspective ensure, and why is "we'll add flexibility later" a myth?
> It ensures the system can change without breaking or requiring a rewrite — by keeping coupling low, interfaces stable, and extension points clear. "We'll add flexibility later" is a myth because flexibility retrofitted into a tangled, coupled system *is* a rewrite. Evolution is a property designed in from day one, not a phase.

> [!faq]- Why is coupling the enemy of evolution?
> Because coupling makes change ripple — the more one component knows about another's internals, the more a change to one breaks others. Loose coupling (stable interfaces, async contracts, bounded contexts) localizes change so a modification touches one module, not many.

> [!faq]- How does YAGNI relate to designing for change?
> YAGNI says don't build unneeded *features* prematurely — it does *not* say ignore *changeability*. You don't pre-build features, but you *do* keep seams clean (stable interfaces, low coupling) so features can be added cheaply later. Confusing the two leads to either over-engineering or an unmaintainable mess.

> [!faq]- Why is a shared database a problem for evolution?
> It's the highest-coupling integration: every consumer depends on the same schema, so any schema change risks breaking all of them. It makes the data model nearly impossible to evolve safely. (See [[Key Elements of Architecture View]] — shared-data connector.) Preferred alternatives: APIs, events, or service-owned data.

> [!faq]- What does "reversibility over prediction" mean as an evolution strategy?
> When unsure about a decision, prefer the one that's *easy to reverse* over the one that locks you in. Defer irreversible choices (database engine, cloud vendor, framework) until you understand the real constraints. A reversible wrong choice costs little; an irreversible wrong choice is expensive.

---

## 📝 Related Notes

- [[Perspective]]
- [[Key Elements of Architecture View]]
- [[Information Viewpoint]]
- [[Deployment Viewpoint]]
- [[Development Resource Perspective]]
- [[Cost Perspective]]
