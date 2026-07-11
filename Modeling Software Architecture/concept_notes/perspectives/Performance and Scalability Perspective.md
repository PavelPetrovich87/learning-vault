---
status: researching
tags: [c4-model, architecture, perspective, performance, scalability, udemy, fundamentals]
created: 2026-07-10
updated: 2026-07-10
difficulty: intermediate
---

# 🧠 Topic: Performance and Scalability Perspective

> [!abstract] Executive Summary (ELI5)
> This perspective asks two questions: **"is it fast enough *now*?" (performance)** and **"will it stay fast when load grows?" (scalability).** It's a lens you apply *to* existing views — no diagram of its own. Hold it up to the Concurrency view ("what runs in parallel, where's the bottleneck?"), the Deployment view ("are the fast-path components close together?"), and the Operational view ("does this journey hit the DB five times?") and demand refinements: caching here, async there, a replica over there.

> [!warning] The #1 confusion — Performance ≠ Scalability
> They're related but different. **Performance** is about a single request under current load — latency, response time, throughput. **Scalability** is about behavior *as load grows* — does doubling the users double the latency, or can you absorb it by adding capacity? A fast system that doesn't scale breaks at 10×; a scalable system that's slow per-request is still unpleasant. You need both.

---

## 🎯 The "Why"

- **Problem Solved:** Performance and scalability problems discovered in production are the most expensive class of bugs — they require architectural changes (not just tuning) under live pressure. Applying this perspective *during design* — "what's the bottleneck under 10× load?" — forces the load-bearing decisions (caching, async, partitioning) to be made when they're cheap.
- **Alternatives:** "We'll load-test before launch." Load testing validates an already-built architecture; it rarely surfaces the need to *restructure* (e.g., extract a hot path, denormalize data). By then restructuring is costly.
- **Trade-offs:** Designing for scale you'll never reach is over-engineering. The discipline is to model the *realistic* growth path and build for the next 1–2 orders of magnitude, not for hypothetical billions.

## 💡 Mental Model & Analogy

- **Analogy:** A supermarket checkout. **Performance** = how fast one customer is served (the cashier's speed, the scanner's speed). **Scalability** = what happens when 50 customers show up — do you open more registers (horizontal scale), or does the single line grow to the back of the store (collapse)? A store with one ultra-fast cashier is performant but not scalable; a store that can open 10 registers is scalable. You need a fast cashier *and* the ability to add more.

```mermaid
graph TD
    Load[Growing load] --> Q{Can it absorb growth?}
    Q --|yes: scalable| OK[System stays responsive]
    Q --|no: bottleneck| Bad[Latency spikes, collapse]
    OK --> How["How?<br/>scale up (bigger node) or<br/>scale out (more nodes)"]
    Bad --> Where["Where's the bottleneck?<br/>CPU, DB, network, lock contention"]
```

## 🛠️ Core Principles

1. **Performance and scalability are distinct concerns.** Performance = behavior under current load (latency, throughput). Scalability = behavior *as load grows*. Optimize them separately — a change that improves one may hurt the other.
2. **Find the bottleneck before optimizing.** Every system has one dominant constraint at a time (the Theory of Constraints). Scaling the non-bottleneck does nothing. Identify the bottleneck first (CPU? DB? network? lock?), then address *that*.
3. **Scale up vs. scale out.** *Scale up* (vertical) = bigger machine (more CPU/RAM). *Scale out* (horizontal) = more machines. Scale-out is usually required for real growth, but demands the architecture be stateless/partitionable.
4. **Statelessness enables horizontal scale.** A stateless service can be replicated freely; a stateful one (sticky sessions, in-memory cache) cannot. Where you place state determines how far you can scale.
5. **Asynchrony decouples load from response.** Moving slow work off the request path (queues, background jobs) keeps the fast path fast and absorbs load spikes by buffering.

## 🏗️ Key Concerns (applied to views)

### Applied to the Concurrency View
*Where's the parallelism, and where's the contention?*
- This is the most important view for this perspective — concurrency is where performance and scalability live or die.
- Identify the bottleneck: is it CPU-bound, I/O-bound, lock-contention-bound, or DB-bound?
- Are runtime units genuinely parallel, or interleaved on a single event loop (see [[Concurrency Viewpoint]] → "async ≠ parallel")?

### Applied to the Deployment View
*Are the fast-path components close together?*
- **Latency is physical:** a DB in a different region than its API adds tens of milliseconds per call — multiplied per query in a journey.
- **Scaling topology:** can you add replicas? Is the load balancer in the right place? Is the single primary DB the bottleneck?
- **Proximity of hot-path components** — co-locate the API and its cache/DB where latency matters.

### Applied to the Operational View
*Does the journey hit the DB five times?*
- **N+1 query patterns**, chatty inter-service calls, synchronous external API calls on the request path.
- **Caching opportunities** — read-heavy data that changes rarely should be cached, not re-fetched.
- **Moving work off the request path** — anything that doesn't need to be synchronous (email, analytics, reporting) goes to a queue.

### Capacity planning (the quantification)
*How much can it handle, and how do you know?*
- Define the **load profile**: requests/sec, data volume, concurrency, peak vs. average.
- Identify the **ceiling** for each resource: at what load does CPU hit 100%, DB connections exhaust, queue depth overflow?
- Decide the **scaling trigger**: when do you add capacity, and is it automatic (autoscaling) or manual?

## ↔️ How the Perspective Connects

| Applied to | The question it asks |
|---|---|
| [[Concurrency Viewpoint]] | What's the bottleneck — CPU, DB, lock, network? Is parallelism real or interleaved? |
| [[Deployment Viewpoint]] | Are hot-path components co-located? Can replicas be added? Where's the single point of failure? |
| [[Operational Viewpoint]] | Does the journey make redundant calls? What can be cached or moved async? |
| [[Information Viewpoint]] | Are there N+1 data access patterns? Is data partitionable for scale-out? |
| [[Perspective]] siblings | Tension with [[Cost Perspective]] (scaling costs money); with [[Availability and Resilience Perspective]] (load spikes can cause failures). |

## ⚠️ Pitfalls & Anti-patterns

- **Optimizing before identifying the bottleneck:** tuning the fast path while the DB is the real constraint. Use profiling/measurement, not guesses.
- **Confusing performance with scalability:** making one request faster while the system still collapses at 10× load. Address both.
- **Designing for hypothetical scale:** building for millions of users when the real growth is 10×. Over-engineering adds cost and complexity for scale you'll never reach.
- **Stateful services blocking scale-out:** sticky sessions or in-memory state that prevents free replication. Move state to a shared store or eliminate it.
- **Synchronous external calls on the request path:** calling a slow third-party API synchronously ties up a request thread per call. Move to async/queue where possible.
- **Single primary DB as the scaling ceiling:** a write-heavy system with one primary DB will hit a wall that no number of read replicas solves. Plan for partitioning/sharding early if writes are the bottleneck.
- **Ignoring the network:** "microservices" with chatty synchronous calls across regions. Network latency, not code speed, becomes the bottleneck.

## 🔗 Deep Connections

- **Parent Topic:** [[Perspective]], [[Viewpoint]]
- **Most relevant viewpoints:** [[Concurrency Viewpoint]] (the primary lens for this perspective), [[Deployment Viewpoint]] (topology/latency), [[Operational Viewpoint]] (journey call patterns)
- **Sibling perspectives:** [[Cost Perspective]] (scaling costs money — tension), [[Availability and Resilience Perspective]] (load spikes cause failures — overlap)
- **Related:** [[System Architecture]] (Performance and Scalability are Quality Attributes), [[Key Elements of Architecture View]] (connectors/protocols drive latency)
- **Prerequisites:** [[Perspective]], [[Concurrency Viewpoint]].

## ❓ Active Recall Questions

> [!faq]- What's the difference between performance and scalability?
> **Performance** is behavior under *current* load — latency, response time, throughput per request. **Scalability** is behavior *as load grows* — does the system absorb more users by adding capacity, or does it collapse? A fast system that doesn't scale breaks at 10×; a scalable system that's slow per-request is still unpleasant. You need both.

> [!faq]- Why must you identify the bottleneck before optimizing?
> Because of the Theory of Constraints — every system has one dominant constraint at a time, and scaling the non-bottleneck does nothing. If the DB is the bottleneck, optimizing the API's CPU usage won't help. Profile and measure first, then address *that* constraint.

> [!faq]- What's the difference between scaling up and scaling out, and which enables real growth?
> **Scale up** (vertical) = a bigger machine (more CPU/RAM). **Scale out** (horizontal) = more machines. Real growth usually requires scale-out, but it demands the architecture be **stateless and partitionable** — a stateful service (sticky sessions, in-memory cache) can't be freely replicated.

> [!faq]- Why does "where you place state" determine how far you can scale?
> Because statelessness enables horizontal scale — a stateless service can be replicated without coordination. A stateful service (sticky sessions, in-memory session data) is pinned to a node, so you can't add replicas freely. Moving state to a shared store or eliminating it is often the key enabler of scale-out.

> [!faq]- How does this perspective interact with the Cost Perspective?
> With tension — scaling costs money (more nodes, bigger DBs, more network egress). The discipline is to scale for the *realistic* growth path (next 1–2 orders of magnitude), not to over-provision for hypothetical load. Capacity planning quantifies the trade-off.

---

## 📝 Related Notes

- [[Perspective]]
- [[Concurrency Viewpoint]]
- [[Deployment Viewpoint]]
- [[Operational Viewpoint]]
- [[Cost Perspective]]
- [[Availability and Resilience Perspective]]
