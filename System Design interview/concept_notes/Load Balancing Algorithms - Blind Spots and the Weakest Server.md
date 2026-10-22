---
created: '2026-08-22'
difficulty: intermediate
mastery: growing
source: System Design Trainer, ch. 1.5 (Su vol.1 ch.1 pp.14, 26-27; simulator saga
  HamsterChat)
status: analyzing
tags:
- system-design
- load-balancer
- round-robin
- least-connections
- tail-latency
- health-check
updated: '2026-08-22'
---

# 🧠 Topic: Load Balancing Algorithms — Blind Spots and the Weakest Server

> [!abstract] Executive Summary (ELI5)
> A load balancer's algorithm is a rule for dividing incoming requests. The two classics — round-robin (take turns) and least-connections (send to the busiest-free) — both divide **load counters**, not **server capability**. Neither knows how much capacity a server has, and neither can fix a capacity deficit: an algorithm divides the pie, it doesn't bake a new one.

---

## 🎯 The "Why"

- **Problem Solved:** "We have a load balancer, but one server keeps drowning." The algorithm is only a heuristic; knowing its blind spots tells you when the fix is a config change and when it's hardware (or pulling a node from rotation).
- Simulator evidence (trainer ch. 1.5, saga): round-robin kept sending ~15 rps to a server that could only do 10 — the queue grew exactly at (share − capacity) per second.

## 💡 Mental Model & Analogy

- Round-robin is a waiter dealing cards one-by-one to identical-looking players — even if one player has tiny hands (capacity) and drops cards (queues).
- Least-connections is the same waiter counting plates in front of each player and dealing to the one with fewest — it equalizes **plates**, not **hand size**.

## 🛠️ Core Principles

### Round-robin — count-blind
- Splits traffic by **request count**, ~equal shares for identical servers (fair and cheap).
- Blind spot: equal counts ≠ equal load. A degraded server (fewer slots) receiving an equal share accumulates a queue growing at (share − capacity) per second; the strong sibling idles with headroom.
- Also jitter-blind in time: by count it's fair, by instant it's bursty — momentary 100% utilization on one node is normal noise.

### Least-connections — counter-blind, but must count in-flight
- Score = **busy slots + queue depth + in-flight requests** on the edge to that server. Picks the minimum.
- The in-flight term is mandatory (nginx `least_conn` semantics: a connection counts **from the moment of forwarding**, not arrival). Counters only move when a request *lands*; while it travels, the server looks free — without in-flight the LB bursts dozens of requests at the "least loaded" node and you get 0/N ↔ N/N+queue oscillations (real bug caught in the trainer simulator: 600 ms blindness at 40 rps ≈ 24 invisible requests).
- Blind spot: balances **load counters**, not **capability**. It keeps feeding a weak server until its counter rises to the others' level — the degraded node ends up with a small permanent queue as its "equalizer" (honest physics, not a bug).

### Neither algorithm adds capacity
- If total input > total capacity, queues grow somewhere no matter the algorithm — LC just chooses *where* more wisely.
- If a pool has ample total capacity but one weak node, the LB will still route it a share.

### The weakest server sets the aggregate tail
- Pool percentiles are dominated by the **worst** backend, not the average: one degraded server carrying 25% of traffic with a ~1 s queue pushed pool p95 = p99 ≈ 1100 ms while p50 stayed 200 ms — SLO (p95 ≤ 1 s) broken with 60 rps capacity against 40 rps load.
- Remedy is **not** a smarter algorithm: health checks + pull the degraded node from rotation (real topic: active/passive health checking). The LB won't heal a sick server; it will faithfully keep killing your percentiles.

## ⚠️ Pitfalls

- Answering "add least-connections" to a *capacity* problem (deficit is fixed by a server, not an algorithm).
- Forgetting in-flight in the LC score → burst oscillations.
- Expecting the LB to notice degradation — it sees counters, not health.
- Treating a small permanent queue on the weakest node as a bug — it's LC equilibrium.

## 🔗 Deep Connections

- [[Scaling Evolution - from Single Server to Stateless Web Tier]] — the LB is the door to horizontal scaling; sticky mode turns it into a router-by-ownership.
- [[Stateless vs Stateful - the SAM Lens]] — why stateless is what makes "any server can serve anyone" true at all.
- [[Percentiles and Tail Latency]] — why the tail, not the mean, exposes the weakest server.
- [[Headroom - Practical Capacity Ceilings]] — why queues appear before the hard limit.