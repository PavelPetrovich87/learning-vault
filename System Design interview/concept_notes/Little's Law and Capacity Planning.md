---
created: '2026-08-19'
difficulty: intermediate
mastery: growing
source: "System Design Trainer, ch. 1.4 (\U0001F534 synthesis beyond Kleppmann; classical
  queueing theory, Little 1961)"
status: analyzing
tags:
- system-design
- capacity
- queueing
- little-s-law
- scalability
- interview
updated: '2026-08-19'
---

# 🧠 Topic: Little's Law and Capacity Planning

> [!abstract] Executive Summary (ELI5)
> A shop with cash registers: customers arrive at λ per second, each takes W seconds at the register — so at any moment there are L = λ·W people inside. If L exceeds the number of registers, a queue grows. Same law for servers: **average requests in flight = arrival rate × response time**. It's the simplest capacity estimate in system design.

---

## 🎯 The "Why"

- **Problem Solved:** "How many workers/slots/connections do we need?" — Kleppmann describes head-of-line blocking but gives no formula to size it. Little's Law (John D. C. Little, 1961) connects the three numbers you already have from earlier steps: peak QPS (from the load funnel), target response time (from the NFR/latency budget) → average occupancy.
- **Note:** this is a **course synthesis** beyond the book anchor (marked 🔴 in the trainer) — DDIA doesn't cover it; it's classical queueing theory stripped to its architecturally useful core. Full theory (M/M/1, M/M/c, Erlang) is out of scope.

## 🛠️ Core Principles

### The law

```
L = λ · W
```

- **L** — average number of requests **inside the system** (being served + waiting in queue); a dimensionless counter.
- **λ** (lambda) — arrival rate, requests/sec; your **peak QPS** from the estimation funnel.
- **W** — average time a request spends in the system (wait + service); your **response time**.

**Dimensions matter (interview trap):** λ = req/s, W = s/req, L = requests in flight. λ is NOT L; mixing them is an instant red flag.

### The capacity threshold

```
λ · W ≤ N   (N = number of slots/workers)
```

Below the threshold the server "breathes" (steady state: what arrives, leaves). Near it the queue grows **non-linearly** and p99 explodes (see [[Percentiles and Tail Latency]] — head-of-line blocking). It's a **threshold, not a target**: running flush to it = queue on the edge of explosion.

### Worked example (messenger, 10M DAU)

- 10M DAU × 2 msgs/day = 20M/day → avg ≈ 230 rps → peak ×2 ≈ **460 rps**.
- Target W = 200ms → L = 460 × 0.2 = **≈92 requests in flight** at peak.
- Provision ≥ ~92 slots; in practice ~×2 margin — but a ×2 margin dies at an overlapping peak (×3–4), because L doubles when λ doubles.
- Punchline: you can't pre-buy ×100 capacity → **autoscaling**, which requires a **stateless** tier (see [[Stateless vs Stateful - the SAM Lens]]).

### Choosing the system boundary

λ, W, N are measured relative to **what you draw around**:
- **Narrow (service only):** W = service time, no network — good for sizing workers on one box.
- **Wide (to the client):** W = full response time including network — good for the NFR contract the user feels.

Same formula, different numbers, different questions. State the boundary.

## ⚠️ Pitfalls

- Treating λ·W ≤ N as a target instead of a threshold.
- Confusing λ (req/s) with L (requests in flight).
- Forgetting the peak multiplier — systems fail at peak, not average.
- Sizing by mean latency while the tail (p99) is what explodes near the threshold.

## 🔗 Related

- [[Percentiles and Tail Latency]] — why the queue inflates the tail.
- [[Back-of-the-Envelope Size Calculations]] — where λ comes from.
- [[Scaling Strategies]] — what to do when one box's N isn't enough.

## ✅ Self-check

1. Formula and units? → L = λ·W; req/s × s = requests in flight.
2. 1000 rps, W=50ms — how many in flight? → 50.
3. Why isn't a ×2 margin enough? → Overlapping peaks double λ → double L → cross the threshold; static margin is bounded → autoscaling.
4. Mnemonic? → "capacity, latency, queue" = the three letters of the formula (N, W, λ/L).