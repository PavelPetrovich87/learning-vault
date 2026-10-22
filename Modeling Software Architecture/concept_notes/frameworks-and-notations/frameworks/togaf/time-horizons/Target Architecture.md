---
status: archived
tags: [c4-model, architecture, togaf, time-horizons, target-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Target Architecture

> [!abstract] Executive Summary (ELI5)
> The **Target Architecture** is the **desired future state** of the architecture — the to-be vision. It describes where the enterprise *wants to be*: the capabilities, applications, data, and technology it aims for. Its job is to provide **direction**: it is the destination the [[Baseline Architecture|Baseline]] evolves toward via [[Transition Architecture|Transitions]]. Without a Target, the architecture drifts; with one, every change can be checked against "does this move us closer to where we want to be?"

> [!note] Part of the TOGAF deep-dive — one of three time horizons
> One of three horizons in [[Architecture Time Horizons]]. It is the "where we want to be" destination; the [[Transition Architecture|Transitions]] lead to it from the [[Baseline Architecture|Baseline]].

---

## 🎯 The "Why"

- **Problem Solved:** Without a Target, every change is local and reactive — there is no "closer to the goal" because there is no goal. The Target provides a **shared destination** so initiatives can be prioritized by whether they advance toward it, and so the Baseline's gap has a meaningful "to" end.
- **Alternatives:** Drifting reactively (no target), or a Target with no Baseline/Transitions (a vision with no path). Both fail — the first has no direction; the second has no execution.

## 🛠️ Core Characteristics

- **It is prescriptive, not just descriptive.** Where the Baseline records what *is*, the Target states what *should be* — the intended future.
- **It spans the BDAT domains.** Future capabilities, future data, future applications, future technology — across [[TOGAF Architecture Domains|all four domains]].
- **It is established early and refined.** The vision is set in Phase B (Architecture Vision), then detailed in Phases C/D alongside the Baseline. See [[Architecture Development Method]].
- **It is reachable via Transitions.** A good Target is ambitious but achievable through a sequence of [[Transition Architecture|Transitions]] from the Baseline — not a fantasy.

## ↔️ Where it sits

- **Horizon:** the destination (to-be).
- **Paired with:** [[Baseline Architecture]] (as-is) — together they define the gap.
- **Connected via:** [[Transition Architecture]] (the path).
- **Analyzed in:** Phase E (gap analysis: what must change to reach it).

> A common failure is a Target with no path — an aspirational vision disconnected from the Baseline and Transitions. Another is a Target frozen for years while the business changes; it must be revisited each ADM cycle (Phase H → B).

## ⚠️ Pitfalls & Anti-patterns

- **No Target (drift):** no shared destination; changes are reactive and local.
- **Target with no path:** a vision with no Baseline gap analysis and no Transitions — aspiration, not architecture.
- **Frozen Target:** a to-be defined once and never revisited while the business evolves. Re-enter via Phase H.
- **Target too vague to guide decisions:** "modern and cloud-native" without concrete capabilities/apps/data. Make it specific enough to prioritize against.
- **Target ignoring the Baseline constraints:** a desired future that is unreachable from the current state in any realistic sequence.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Time Horizons]], [[TOGAF]]
- **Sibling horizons:** [[Baseline Architecture]] (from), [[Transition Architecture]] (path)
- **Where it's established:** [[Architecture Development Method]] (Phase B vision; C/D detail)
- **Prerequisites:** [[TOGAF]], [[Architecture Time Horizons]].

## ❓ Active Recall Questions

> [!faq]- What is the Target Architecture, and why is it needed?
> The **desired future state** (to-be) — where the enterprise wants to be across the BDAT domains. It is needed to provide **direction**: without it, changes are reactive and there's no "closer to the goal." It is the destination the Baseline evolves toward via Transitions.

> [!faq]- What are the common failures with the Target?
> (1) **No Target** — drift, reactive changes. (2) **Target with no path** — a vision with no Baseline/Transitions, i.e. aspiration not architecture. (3) **Frozen Target** — defined once and never revisited as the business changes. A good Target is specific, reachable via Transitions, and re-visited each ADM cycle.

> [!faq]- How do Target, Baseline, and Transitions relate?
> The **Baseline** is the as-is, the **Target** is the to-be, and the **Transitions** are the intermediate milestones between them. Gap analysis (Phase E) measures Baseline→Target; the Transitions (Phase F) are the sequenced path. All three are one architecture at different moments in time. See [[Architecture Time Horizons]].

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Time Horizons]]
- [[Baseline Architecture]]
- [[Transition Architecture]]
- [[Architecture Development Method]]
