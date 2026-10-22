---
status: archived
tags: [c4-model, architecture, togaf, time-horizons, baseline-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Baseline Architecture

> [!abstract] Executive Summary (ELI5)
> The **Baseline Architecture** is the **current state** of the architecture — the as-is. It describes what exists *today*: the current capabilities, applications, data, and technology. Its job is to be the **honest starting point** for change: you cannot plan how to get to the [[Target Architecture|Target]] if you don't know where you are now. Baseline is what gap analysis measures *from*.

> [!note] Part of the TOGAF deep-dive — one of three time horizons
> One of three horizons in [[Architecture Time Horizons]]. It is the "where we are now" anchor; the [[Transition Architecture|Transitions]] lead from it to the [[Target Architecture|Target]].

---

## 🎯 The "Why"

- **Problem Solved:** Without a Baseline, "the gap" is undefined — you don't know what to keep, replace, or build. A documented Baseline makes the current state explicit and shared, so the gap to the Target is measurable and the migration plannable.
- **Alternatives:** Jumping straight to a Target ("we'll figure out the current state as we go"). Result: surprises, missed dependencies, and a migration plan disconnected from reality.

## 🛠️ Core Characteristics

- **It is descriptive, not prescriptive.** The Baseline records what *is*, not what *should be*. Value judgments belong in the Target.
- **It spans the BDAT domains.** Current capabilities, current data, current applications, current technology — described across [[TOGAF Architecture Domains|all four domains]].
- **It is produced in ADM Phases C/D** (Business, then Data/Application/Technology), as the first half of describing each domain (Baseline, then Target). See [[Architecture Development Method]].
- **It must be kept current.** A stale Baseline makes gap analysis measure against a fiction. Re-validate it each ADM cycle.

## ↔️ Where it sits

- **Horizon:** the starting point (as-is).
- **Paired with:** [[Target Architecture]] (to-be) — together they define the gap.
- **Connected via:** [[Transition Architecture]] (the path between them).
- **Analyzed in:** Phase E (gap analysis).

> The most common failure with the Baseline is *not having one* — designing a Target in a vacuum. The second most common is having one that's *stale*. Both make the migration plan unreliable.

## ⚠️ Pitfalls & Anti-patterns

- **Skipping it:** designing only a Target. No measurable gap, no realistic plan.
- **Letting it go stale:** the as-is drifts from reality; gap analysis measures against fiction.
- **Mixing prescription into description:** "and we should replace this" belongs in the Target, not the Baseline.
- **Too detailed:** over-modeling the current state wastes time — capture architecturally significant as-is, not every detail.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Time Horizons]], [[TOGAF]]
- **Sibling horizons:** [[Target Architecture]], [[Transition Architecture]]
- **Where it's produced:** [[Architecture Development Method]] (Phases C/D)
- **Prerequisites:** [[TOGAF]], [[Architecture Time Horizons]].

## ❓ Active Recall Questions

> [!faq]- What is the Baseline Architecture, and why is it essential?
> It is the **current as-is state** of the architecture — what exists today across the BDAT domains. It is essential because it is the **honest starting point** for change: without it, the gap to the Target is undefined and the migration plan is disconnected from reality.

> [!faq]- What is the most common failure with the Baseline?
> **Not having one** — designing a Target in a vacuum, so there is no measurable gap. The second most common is a **stale** Baseline that no longer reflects reality, making gap analysis measure against a fiction.

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Time Horizons]]
- [[Target Architecture]]
- [[Transition Architecture]]
- [[Architecture Development Method]]
