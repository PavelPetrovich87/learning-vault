---
status: researching
tags: [c4-model, architecture, perspective, availability, resilience, udemy, fundamentals]
created: 2026-07-10
updated: 2026-07-10
difficulty: intermediate
---

# 🧠 Topic: Availability and Resilience Perspective

> [!abstract] Executive Summary (ELI5)
> This perspective asks: **"will the system keep running when things fail — and recover gracefully when it can't?"** *Availability* is about staying up (no downtime). *Resilience* is about surviving and recovering from failures (a crashed service, a lost node, a network partition). It's a lens applied *to* existing views — hold it up to the Deployment view ("what if this node dies?"), the Concurrency view ("what if this thread hangs?"), and the Operational view ("what's the recovery sequence?") and demand redundancies, fallbacks, and recovery paths.

> [!warning] The #1 confusion — Availability ≠ Resilience
> **Availability** is measured uptime — "the system is reachable." **Resilience** is failure-handling ability — "when a dependency dies, the system degrades gracefully instead of cascading to a full outage." A system can be highly available on average (99.99%) but not resilient (when it fails, it fails completely and takes hours to recover). High availability without resilience gives you rare but catastrophic outages.

---

## 🎯 The "Why"

- **Problem Solved:** Distributed systems are *defined* by partial failure — something is always degraded, restarting, or unreachable. Applying this perspective during design forces you to answer "what happens when X fails?" for every dependency, and to build in redundancies and recovery *before* the first 3 AM page.
- **Alternatives:** "We'll add retries when we see problems." Reactive resilience produces cascading failures and long outages — because the failure modes (cascading collapse, thundering herd, stuck retries) only appear under real failure, when fixing them means a fire drill.
- **Trade-offs:** Resilience adds complexity (redundancy, fallback logic, recovery code) and cost (extra instances, multi-AZ). The discipline is to match the resilience investment to the *business cost of downtime* — not every service needs 99.99%.

## 💡 Mental Model & Analogy

- **Analogy:** A hospital. **Availability** = the ER is open and admitting patients (uptime). **Resilience** = when the power fails, the generators kick in, critical systems stay on battery, and non-critical surgeries are postponed rather than catastrophically aborted. The hospital doesn't just "go down" — it degrades gracefully and recovers. A hospital that's open 24/7 but collapses entirely during a blackout is available but not resilient.

```mermaid
graph TD
    F[Failure occurs:<br/>node, network, dependency] --> R{Resilient?}
    R --|yes| Grace["Degrades gracefully<br/>(fallback, retry, circuit break)"]
    R --|no| Cascade[Cascading failure<br/>→ full outage]
    Grace --> Recov[Recovery:<br/>restart, failover, replay]
    Recov --> Up[Back to normal]
    Cascade --> LongDR[Long, painful recovery]
```

## 🛠️ Core Principles

1. **Assume failure is normal.** In distributed systems, every dependency will fail, restart, stall, or partition at some point. Designing *as if* things always work guarantees cascading collapses. Design for the steady state of partial failure.
2. **Availability is about redundancy; resilience is about graceful degradation.** Availability multiplies capacity (replicas, failover). Resilience contains failures (circuit breakers, bulkheads, fallbacks) so one failure doesn't take down everything.
3. **Eliminate single points of failure (SPOFs).** A component with no redundancy is a SPOF — its death takes down everything depending on it. Every critical-path component needs a fallback.
4. **Define the failure model explicitly.** What failures do you tolerate? (a node crash, a network partition, a full region loss, a third-party outage.) Each tolerated failure needs a designed response.
5. **Recovery is part of the design, not an afterthought.** How does the system get back to normal after a failure? (restart, failover, replay from queue, restore from backup.) An unrecoverable system has an outage even if it failed gracefully.

## 🏗️ Key Concerns (applied to views)

### Applied to the Deployment View
*What if this node/zone/region dies?*
- **Redundancy** — multiple instances across availability zones; no SPOF on the critical path.
- **Failover** — when the primary DB dies, does a replica promote automatically? How long does it take (RTO)?
- **Data durability** — if a node is lost, is the data safe (replicated, backed up)? How much data can you lose (RPO)?
- **Multi-AZ vs. multi-region** — match the topology to the failure model. Tolerating a zone loss needs multi-AZ; tolerating a region loss needs multi-region.

### Applied to the Operational View
*What's the failure and recovery sequence?*
- **Retries with backoff** — transient failures get retried, but not infinitely (which causes load spikes).
- **Circuit breakers** — stop calling a failing dependency; let it recover instead of hammering it.
- **Bulkheads** — isolate failures so one degraded service doesn't exhaust threads/connections for everything else.
- **Fallbacks** — when the recommendation service is down, show a default; don't fail the whole page.
- **Recovery procedures** — replay from a queue, restore from snapshot, fail back after fix.

### Applied to the Concurrency View
*What if this thread/handler hangs?*
- **Timeouts on every external call** — a hung dependency shouldn't hold a thread forever.
- **Thread/connection pool limits** — a slow downstream shouldn't exhaust the pool and starve everything.
- **Queue depth and dead-letter handling** — what happens when the work queue overflows?

### Quantifying availability (the SLA layer)
- **Uptime targets** — 99.9% (≈ 8.7h downtime/yr), 99.99% (≈ 52min/yr), 99.999% ("five nines," ≈ 5min/yr). Each "9" roughly doubles cost.
- **RTO** (Recovery Time Objective) — how long until service is restored after a failure.
- **RPO** (Recovery Point Objective) — how much data you can afford to lose.
- These targets drive the redundancy/recovery investment — they're the bridge between business and architecture.

## ↔️ How the Perspective Connects

| Applied to | The question it asks |
|---|---|
| [[Deployment Viewpoint]] | What if this node/zone/region dies? Where are the SPOFs? |
| [[Operational Viewpoint]] | What's the failure and recovery sequence? Are there circuit breakers and fallbacks? |
| [[Concurrency Viewpoint]] | What if this thread hangs? Are there timeouts and bulkheads? |
| [[Information Viewpoint]] | Is data durable (replicated/backed up)? How much can be lost (RPO)? |
| [[Perspective]] siblings | Overlaps with [[Security Perspective]] (DoS is both); tension with [[Cost Perspective]] (redundancy costs money). |

## ⚠️ Pitfalls & Anti-patterns

- **Assuming things won't fail:** designing for the happy path. The first real failure becomes a cascading outage because no one built a response.
- **Retries without backoff or limit:** a failing dependency + aggressive retries = amplified load that takes down the dependency permanently (and often the caller too). Retries need backoff and a cap.
- **No bulkheads:** one slow service exhausts a shared thread/connection pool, starving unrelated services. Isolate resources per dependency.
- **SPOFs on the critical path:** a single load balancer, a single primary DB, a single cache — each is one failure away from a full outage.
- **Confusing availability with resilience:** hitting an SLA on average while having catastrophic, slow-to-recover failures. Measure both uptime *and* recovery behavior.
- **Untested recovery:** "we have failover" — that's never been rehearsed. Failover that doesn't work when needed is worse than no failover (it gives false confidence). Game-day / chaos testing validates recovery.
- **Ignoring cascading failure:** one component dies, and because everything depends on it synchronously, everything dies. Asynchrony and decoupling contain the blast radius.

## 🔗 Deep Connections

- **Parent Topic:** [[Perspective]], [[Viewpoint]]
- **Most relevant viewpoints:** [[Deployment Viewpoint]] (redundancy/failover topology), [[Operational Viewpoint]] (failure/recovery sequences), [[Concurrency Viewpoint]] (timeouts/bulkheads)
- **Sibling perspectives:** [[Security Perspective]] (DoS resistance is shared), [[Cost Perspective]] (redundancy/recovery cost money), [[Performance and Scalability Perspective]] (load spikes cause failures)
- **Related:** [[System Architecture]] (Availability and Reliability are Quality Attributes)
- **Prerequisites:** [[Perspective]], [[Deployment Viewpoint]].

## ❓ Active Recall Questions

> [!faq]- What's the difference between availability and resilience?
> **Availability** is measured uptime — the system is reachable. **Resilience** is failure-handling ability — when a dependency fails, the system degrades gracefully and recovers instead of cascading to a full outage. A system can be highly available on average (99.99%) but not resilient (when it fails, it fails completely and recovers slowly). High availability without resilience gives rare but catastrophic outages.

> [!faq]- Why must distributed systems be designed assuming failure is normal?
> Because distributed systems are *defined* by partial failure — something is always degraded, restarting, or partitioned. Designing as if things always work guarantees cascading collapses when the first failure hits. Designing for the steady state of partial failure (retries, circuit breakers, fallbacks) contains failures instead of amplifying them.

> [!faq]- What's a single point of failure (SPOF), and why is it the central concern of this perspective?
> A SPOF is a component with no redundancy — its death takes down everything depending on it (a single load balancer, primary DB, or cache). Eliminating SPOFs on the critical path is the core of availability: every critical component needs a replica, failover, or fallback.

> [!faq]- Why are retries without backoff or a cap dangerous?
> Because a failing dependency + aggressive retries = amplified load that takes the dependency down permanently (and often the caller too). Retries need exponential backoff and a cap, plus a circuit breaker to stop calling a dependency that's clearly broken. Otherwise the cure becomes worse than the disease.

> [!faq]- What are RTO and RPO, and how do they drive architectural decisions?
> **RTO** (Recovery Time Objective) = how long until service is restored after a failure. **RPO** (Recovery Point Objective) = how much data you can afford to lose. These targets, set by the business cost of downtime/data loss, drive the redundancy and recovery investment — e.g., a low RPO demands synchronous replication; a low RTO demands automated failover.

---

## 📝 Related Notes

- [[Perspective]]
- [[Deployment Viewpoint]]
- [[Operational Viewpoint]]
- [[Concurrency Viewpoint]]
- [[Security Perspective]]
- [[Cost Perspective]]
