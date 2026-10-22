---
created: '2026-08-22'
difficulty: intermediate
mastery: growing
source: System Design Trainer, ch. 1.5 (Su vol.1 ch.1 pp.27-28; simulator saga HamsterChat)
status: analyzing
tags:
- system-design
- autoscaling
- stateless
- elasticity
- scale-in
- drain
updated: '2026-08-22'
---

# 🧠 Topic: Autoscaling as a Consequence of Stateless

> [!abstract] Executive Summary (ELI5)
> Autoscaling is adding and removing servers automatically, by load. It's not a separate technology you bolt on — it's a **privilege the stateless web tier earns**: only interchangeable servers make add/remove routine, and only routine operations can be trusted to an automaton. In stateful, autoscaling fails in **both directions**: a new server can't help pinned clients, and a removed server takes live sessions with it.

---

## 🎯 The "Why"

- **Problem Solved:** static headroom (×2) can't cover unpredictable spikes (×100) — you can't pre-pay for a curve you can't predict. The chain from ch. 1.4: static margin is bounded → need autoscaling → autoscaling needs stateless.
- The interview trap: naming autoscaling as a feature while the design still has in-RAM sessions.

## 💡 Mental Model & Analogy

- A stateless pool is a stack of identical rental bikes: taking one in or out changes nothing for riders already rolling. A stateful pool is a cloakroom where each attendant holds specific coats — adding an attendant doesn't move coats, and firing one **burns the coats**.

## 🛠️ Core Principles

### Both directions must be safe
- **Scale-out is useless for pinned clients:** in stateful, a new server serves only *new* clients; the overloaded "own" server of a hot user keeps dying while the new node idles at 0/N (simulator: noisy 30 rps client pinned to a 25 rps server; adding a server changed nothing).
- **Scale-in evicts state:** removing a stateful node = killing its sessions. An automated scale-in that logs users out is worse than manual operation — "the night of the failed scale-in". Stateless removal is safe **after drain**: stop sending new traffic, let in-flight finish, then kill (naive on/off simulators skip drain and honestly show a small failure burst).
- Therefore: automatable = routine in **both** directions. That's why autoscaling is a *consequence* of stateless, not a feature of a load balancer.

### Why vertical scaling is not an autoscaling strategy
- It "works" (more slots on the hot box) but is **not elastic in time**: you can't rent extra power for three peak hours. It steps in whole machines, usually with replacement downtime (the swap itself loses in-flight requests), the price grows superlinearly, and there's a hardware ceiling.
- Elasticity means **capacity matches the bill in time** — that's horizontal + per-hour instances.

### Manual vs automatic (the distinction interviewers listen for)
- Manual: a human watches dashboards and clicks — reaction in minutes, doesn't survive 03:00 spikes or ×100 jumps.
- Automatic: thresholds on load (queue depth, CPU, RPS) with **hysteresis/cooldowns** (e.g. overload 3 s → add one; idle 5 s at utilization < 25% and live > 1 → remove least-loaded) so the pool doesn't flap around the threshold.
- Unpredictable spikes are closed **only** by the automatic kind.

## ⚠️ Pitfalls

- Saying "autoscaling" while sessions live in server RAM — it's blocked in both directions.
- Forgetting scale-in: autoscaling is two-sided, and the shrink side is where stateless is truly mandatory (drain).
- Treating vertical "autoscaling" (resize instance) as equivalent — not elastic, replacement downtime.
- No cooldowns → thrashing nodes in/out at the threshold.

## 🔗 Deep Connections

- [[Stateless vs Stateful - the SAM Lens]] — S axis: why pinned clients make scale-out useless.
- [[Scaling Evolution - from Single Server to Stateless Web Tier]] — stage 4 and the ×100 chain.
- [[Scaling Strategies]] (udemy) — vertical/horizontal/diagonal vocabulary.
- [[Headroom - Practical Capacity Ceilings]] — thresholds are utilization-based, so headroom sets them.