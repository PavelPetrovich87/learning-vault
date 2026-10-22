---
status: archived
tags: [c4-model, architecture, togaf, time-horizons, transition-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Transition Architecture

> [!abstract] Executive Summary (ELI5)
> A **Transition Architecture** is an **intermediate state** between the [[Baseline Architecture|Baseline (as-is)]] and the [[Target Architecture|Target (to-be)]] — a deliverable milestone along the migration path. Instead of one terrifying big-bang jump, you sequence several Transitions, each a coherent, *liveable* architecture the enterprise can actually run. Each Transition delivers part of the change, proves it works in production, and sets up the next step.

> [!note] Part of the TOGAF deep-dive — one of three time horizons
> One of three horizons in [[Architecture Time Horizons]]. There are typically *several* Transition Architectures forming the path from Baseline to Target.

---

## 🎯 The "Why"

- **Problem Solved:** The gap from Baseline to Target is often too large to deliver at once — too risky, too long, too disruptive. Transition Architectures **decompose the change** into staged, shippable milestones, each delivering value and reducing risk. They turn a big-bang gamble into an incremental, course-correctable journey.
- **Alternatives:** Big-bang migration — build the whole Target, then switch. High risk, long lead time, no checkpoints, and no early value. Or no Target at all (drift).
- **Trade-offs:** More Transitions = more milestones to coordinate and keep consistent. The payoff is de-risked, incremental delivery with early value.

## 🛠️ Core Characteristics

- **Each Transition is a coherent, runnable architecture.** It is not a partial build — it is a state the enterprise can actually operate. (Like a house you can inhabit mid-renovation.)
- **Each delivers part of the gap.** A Transition moves some capabilities/apps/data from Baseline toward Target, leaving the rest for later.
- **They are sequenced into a roadmap.** Phase F (Migration Planning) orders the Transitions and packages them into work packages/projects. See [[Architecture Development Method]].
- **They are consistent with both ends.** A Transition must fit the Baseline it evolves from and lead toward the Target — it is one architecture at a moment in time.

## ↔️ Where it sits

- **Horizon:** intermediate (between Baseline and Target).
- **Produced in:** ADM Phase F (Migration Planning), based on the gap analysis from Phase E.
- **Delivered by:** projects/work packages — each Transition corresponds to a deliverable chunk.
- **Governed by:** Phase G checks each delivered Transition conforms to its definition.

> Transition planning is where TOGAF meets project reality: the gap analysis (E) becomes a sequenced roadmap (F) of Transition Architectures, each a milestone the organization can commit to and operate.

## ⚠️ Pitfalls & Anti-patterns

- **No Transitions (big-bang):** one jump from Baseline to Target. High risk, no checkpoints.
- **Transitions that aren't liveable:** an intermediate state the enterprise can't actually run — it must be coherent and operational.
- **Too many tiny Transitions:** coordination overhead exceeds the value. Right-size the milestones.
- **Transitions disconnected from the gap:** Transition plans that don't trace to the Baseline→Target gap analysis are arbitrary.
- **No governance of delivered Transitions:** each delivered Transition must be checked (Phase G) against its definition — otherwise drift accumulates.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Time Horizons]], [[TOGAF]]
- **Sibling horizons:** [[Baseline Architecture]] (from), [[Target Architecture]] (to)
- **Where it's produced:** [[Architecture Development Method]] (Phase F, from Phase E gap analysis)
- **Prerequisites:** [[TOGAF]], [[Architecture Time Horizons]].

## ❓ Active Recall Questions

> [!faq]- What is a Transition Architecture, and what problem does it solve?
> An **intermediate state** between Baseline and Target — a deliverable milestone the enterprise can actually run. It solves the big-bang problem: instead of one risky jump, the change is decomposed into staged, shippable, low-risk milestones, each delivering part of the gap and early value.

> [!faq]- What makes a Transition "liveable," and why does that matter?
> A Transition must be a **coherent, operational architecture** the enterprise can actually run — not a partial build. It matters because the enterprise has to operate at each intermediate state; an un-liveable Transition would mean downtime or an unrunnable system mid-migration. (Like a house you can still inhabit mid-renovation.)

> [!faq]- Where are Transition Architectures produced in the ADM?
> In **Phase F (Migration Planning)**, based on the gap analysis from Phase E. Phase F sequences the Transitions into an implementation roadmap of work packages/projects, and Phase G later checks each delivered Transition conforms to its definition. See [[Architecture Development Method]].

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Time Horizons]]
- [[Baseline Architecture]]
- [[Target Architecture]]
- [[Architecture Development Method]]
