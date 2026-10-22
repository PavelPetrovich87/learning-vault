---
created: '2026-08-19'
difficulty: intermediate
mastery: growing
source: System Design Trainer, ch. 1.4 (Kleppmann, DDIA ch.1.3 "Describing Performance",
  pp.36-41)
status: analyzing
tags:
- system-design
- latency
- percentiles
- tail-latency
- slo
- performance
- interview
updated: '2026-08-19'
---

# 🧠 Topic: Percentiles and Tail Latency

> [!abstract] Executive Summary (ELI5)
> Average latency is like "average depth of a river" — it tells you nothing about the spot where someone drowns. Response time is a **distribution**, not a number: sort all response times ascending, and a percentile is simply a **position** in that list. p50 = the middle (typical user), p95/p99 = the tail (your most valuable users). The mean averages a 10ms request with a 10s hang and reports a comfortable lie.

---

## 🎯 The "Why"

- **Problem Solved:** "Mean latency 80ms" hides the tail: 99 requests at 10ms + 1 at 10s → mean 110ms while one user in a hundred sits on a hang. Reporting the mean means the product manager asks "why does every 100th user wait 10 seconds?" — and you have no answer.
- **Business impact (Amazon):** users with the slowest requests are the ones with the most data in their accounts — the most valuable customers. +100ms ≈ −1% sales; +1s ≈ −16% satisfaction.
- **Alternatives:** mean (hides tail), median alone (p50 still hides the tail — it IS the middle), single p99 (misses the typical experience). Use the **trio p50/p95/p99**.

## 💡 Mental Model & Analogy

Percentile = a **position in the sorted-from-fastest list**, not "a representative of a group". For 100 requests where 95 took 50ms, 4 took 1s, 1 took 10s:

- p50 = 50ms (position 50 — inside the fast group)
- p95 = 50ms (position 95 — still the fast group, boundary)
- p99 = 1s (position 99)
- max = 10s

**Classic mistake:** mapping groups to percentiles directly ("p95 = 1s because that's the second group") — a whole magnitude off. Sort first, then index.

**Mechanics (nearest-rank):** for N samples, p-th percentile = element at index `ceil(p·N) − 1` of the sorted array. No interpolation — keeps discrete groups honest. (Production: t-digest / HdrHistogram, Kleppmann §1.3.)

## 🛠️ Core Principles

### Response time ≠ latency
- **Response time** = what the client sees: service time + network + **queue wait**. 
- **Latency (wait time)** = how long the request waits to be served, excluding the work itself.
- Response time ≥ latency, **always**. Diagnostic value: slow-from-queue ≠ slow-from-processing.

### Head-of-line blocking
A server processes only a bounded number of jobs in parallel (slots/cores). Slow requests occupy all slots → subsequent fast requests **queue behind them** → tail inflates. Consequences (Kleppmann):
1. **Measure response time on the client side** — the server doesn't see its own queue from the client's vantage point.
2. **Don't shorten the queue in load tests** — a test client that waits for each response before sending the next never builds a queue; you're testing a system that doesn't exist.

### SLO / SLA — percentiles as contracts
- **SLO** (objective): "we aim for p50 < 200ms, p99 < 1s".
- **SLA** (legal agreement): "if p99 > 1s more than 0.1% of the time → compensation".
- Availability nines attach here: `p99 < 1s at 99.9% uptime` is a contract, not a wish. See [[Latency Budget - System Design]].

### Median's quiet trap
p50 is about **one** request. A session issues many requests — the probability that at least one of N requests lands in the tail grows fast with N. The tail bites more often than single-request intuition suggests.

## ⚠️ Pitfalls

- Reporting the mean (or the median alone) — always the p50/p95/p99 trio.
- Confusing response time with latency on an interview = "doesn't distinguish work from waiting for work".
- Chasing p99.9+ without a business reason — price usually dwarfs gain; Amazon stopped at high percentiles deliberately.
- Group→percentile direct mapping instead of sort-then-index.

## 🔗 Related

- [[Latency Budget - System Design]] — turning percentile SLOs into per-component ms budgets.
- [[Little's Law and Capacity Planning]] — why the queue exists in the first place.
- [[Back-of-the-Envelope Size Calculations]] — the load funnel that produces the RPS you measure percentiles at.

## ✅ Self-check

1. Mean 80ms — is the service fast? → Unknown; need p50/p95/p99.
2. 95×50ms + 4×1s + 1×10s: p95? → 50ms (position 95 in sorted list).
3. Why client-side measurement? → Server doesn't see its own queue.
4. Why stop at p99? → Diminishing returns, external random factors; tail = valuable users, but perfection doesn't pay back.