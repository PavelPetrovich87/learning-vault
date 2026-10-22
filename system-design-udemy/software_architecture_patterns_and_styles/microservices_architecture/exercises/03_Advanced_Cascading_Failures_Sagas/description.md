---
status: archived
exercise_type: interview_simulation
difficulty: advanced
pattern: microservices
source_concepts:
  - "[[Microservices Challenges]]"
  - "[[Microservices Communication]]"
  - "[[Structuring Microservices]]"
  - "[[Microservices Scaling Strategies]]"
  - "[[Event-Driven Architecture]]"
  - "[[Microservices Overview]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: Cascading Failures and the Missing Saga

## Type
**Interview simulation** — staff-level system-design interview. The interviewer is a staff engineer who probes deeply into data-consistency theory, resilience patterns, and the engineering judgment of *where* to apply each. Expect to be pushed past the textbook answer into trade-offs.

## Difficulty
**Advanced.** Tests two of the hardest production realities in microservices at once: **(1)** a slow downstream service cascading through synchronous chains to take down entire request paths, and **(2)** multi-service distributed transactions leaving the system inconsistent on partial failure — because there is no saga. A strong answer diagnoses both, proposes a resilience architecture, and designs a saga with explicit compensations.

## The Situation

**Company:** Marketcrate — a large e-commerce platform. Database-per-service microservices, mature infrastructure (Kubernetes, service mesh, distributed tracing, feature flags). The DevOps tax is fully paid. This is not a maturity problem.

**Two production crises are happening simultaneously:**

### Crisis 1 — Cascading Failures

The order-placement path is a long synchronous chain:

```
Cart → Pricing → Inventory → Payment → Promo → (back to Cart for confirmation)
```

Every hop is a blocking HTTP call, each with its own connection pool and thread budget. Last month, the **Inventory** service slowed to a crawl during a flash sale (a hot item, a DB query that wasn't indexed for the new load profile). What happened next:

- Cart's calls to Inventory started timing out — but only after **consuming threads** for the full timeout window (default 10s, with retries).
- Cart's thread pool exhausted. Incoming requests queued and timed out at the load balancer.
- Pricing and Payment, which sit *upstream* of Inventory in other call paths, also stalled waiting on shared downstream dependencies. Their pools exhausted too.
- The blast radius grew until **the entire checkout path was down** — not just the broken Inventory service. A single slow service took down a path it didn't even directly serve in some requests.

The team's reflex fixes (longer timeouts, more retries) made it *worse*: longer timeouts held threads longer; retries multiplied load onto an already-failing service.

### Crisis 2 — Data Inconsistency (No Saga)

The order-placement *workflow* spans four services with database-per-service, so there is no distributed transaction:

1. **Order** — create the order record (status: `PENDING`).
2. **Inventory** — reserve stock for the items.
3. **Payment** — charge the customer.
4. **Order** — mark `CONFIRMED` (or cancel on failure).

Each step writes to its own database. There is **no saga** — no coordination, no compensating actions. So when step 3 fails, the system is left in states like:

- **Orphaned reservations:** Inventory reserved stock, Payment failed, but Inventory is never told to release. Stock "leaks" — items stuck in reserved limbo, eventually never sold.
- **Charged-but-unconfirmed orders:** Payment succeeded, but a later step threw, and Order stays `PENDING` forever. The customer was charged for an order the system thinks didn't complete.
- **Double charges on retries:** naive retry logic re-charges on transient failures because Payment operations aren't idempotent.

The on-call runbook for these is manual SQL surgery at 3 AM. It is not scaling.

**The mandate:** design an architecture that (a) stops cascading failures from one slow service taking down entire request paths, and (b) guarantees the order-placement workflow reaches a consistent end state — even when individual steps fail.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **Checkout team** | Fast, reliable order placement; immediate success/failure response for the customer | Wants synchronous validation; resists async because it complicates the UX ("did my order go through?") |
| **Payments team** | Never double-charge, never lose a charge, full audit trail | Cares most about idempotency and exactly-once semantics; pushes back on anything that risks duplicates |
| **Inventory team** | Accurate stock, no leaked reservations | Wants reservations released promptly on failure; resents being blamed for "leaks" caused by upstream not telling them |
| **On-call / SRE** | Blast radius contained, fast recovery, sleep | Wants bulkheads, circuit breakers, and self-healing — not bigger instance sizes |
| **Business / merchandising** | Maximize completed orders, never refuse a good customer, no wrong money | Tolerates eventual consistency for *fulfillment* but not for *money* |
| **Engineering leadership** | A clean, durable pattern, not a patch per incident | Wants the saga and the resilience patterns institutionalized, not bolted on |

## Constraints

- **Database-per-service is staying.** There is no shared DB to fall back to, and no 2PC / XA across services — it doesn't perform at this scale and the team has ruled it out.
- **The synchronous chain is the root cause of Crisis 1.** Resilience patterns (circuit breakers, bulkheads, timeouts) are necessary but not sufficient if the chain itself remains for cross-service *writes*.
- **Distributed transactions are not transactional.** Any multi-service write workflow needs a **saga** with explicit compensating actions to remain consistent on partial failure.
- **The customer experience matters.** Order placement must give the customer a clear, fast answer in the common case. Eventual consistency is acceptable for fulfillment, not for "did you take my money."
- **Idempotency is mandatory** for any retried operation, especially Payment.
- **Infrastructure is mature** — message broker, distributed tracing, feature flags, service mesh all available. The patterns are implementable today.

## The Ask

The interviewer expects the candidate to address **both** crises:

1. **Resilience architecture (Crisis 1):** how do you stop a slow downstream from cascading? Name the patterns (circuit breaker, bulkhead, timeout, retry with backoff) and — critically — say *where* each applies and what it does and does not protect against.
2. **The structural fix for the chain:** the resilience patterns buy you time, but the real fix is to **stop chaining synchronous calls for cross-service writes.** Propose moving the order-placement write path to async. Justify the sync-vs-async split.
3. **The saga (Crisis 2):** design a saga for order placement (create order → reserve inventory → charge payment → confirm). Choose **choreography vs. orchestration** and defend it. Define the **compensating actions** for each step (release inventory, refund payment). Handle the failure paths.
4. **The dual-write problem:** order creation + "publish OrderPlaced event" is a dual-write across DB and broker. How do you avoid the half-committed state? (**Outbox pattern.**)
5. **Idempotency & ordering:** consumers must be idempotent (re-deliveries are real) and handle out-of-order messages. How?
6. **The read path:** the customer's "where's my order?" view aggregates state across services. Does the candidate consider **CQRS / a read model** rather than live fan-out?

## Hidden Traps

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading and answer first.
>
> 1. **Resilience patterns alone are not the answer.** Circuit breakers + bulkheads + timeouts on every sync call are *necessary* — they contain the blast radius — but they don't fix the root cause: a synchronous chain for cross-service writes. Strong candidates say "these buy us time; the real fix is async for writes" and propose both layers. Weak candidates stop at circuit breakers.
> 2. **Longer timeouts and aggressive retries make it worse.** The team already learned this the hard way. Strong candidates name *retry storms* / *retry amplification* and prescribe bounded retries with exponential backoff + jitter, and timeouts *shorter* than the thread-pool saturation point.
> 3. **Bulkheads are the unsung hero.** Circuit breakers get the press, but bulkheads (separate thread/connection pools per downstream) are what actually prevent one slow dependency from exhausting the pool for *all* dependencies. Strong candidates lead with bulkheads.
> 4. **Choreography vs. orchestration — the choice must be defended, not defaulted.** For a workflow with money and explicit compensation (refund), **orchestration** is usually the better fit: a central orchestrator makes the saga legible, the compensations explicit, and the state machine debuggable. Choreography is attractive for simple fan-out but becomes a debugging nightmare when a refund goes wrong six services deep. Strong candidates justify the trade-off.
> 5. **Compensations must be explicit and idempotent.** "Release inventory," "refund payment" are not afterthoughts — they're first-class operations. And they must be idempotent (a compensation may be retried). Weak candidates hand-wave "and we undo it."
> 6. **The dual-write / Outbox problem.** Creating the order in the DB and publishing `OrderPlaced` to the broker is a dual-write — if either fails you're inconsistent. Strong candidates raise the **Outbox pattern** (write the event in the same DB transaction, a separate process ships it to the broker) or transactional outbox + CDC. This is the single most-missed point.
> 7. **Naive "2PC" is a trap.** Suggesting two-phase commit / XA across services is rejected: it doesn't perform, it blocks on failure, and the team has ruled it out. Sagas are the answer, not distributed transactions.
> 8. **Idempotency and ordering.** Consumers *will* see duplicates and out-of-order messages. Strong candidates require idempotency keys on Payment (and every retryable op) and either event-sourced/optimistic ordering or a saga state machine that rejects stale transitions.
> 9. **Ignoring the read path.** A customer's "order status" view that fans out live across Order, Inventory, Payment, Fulfillment is itself a cascade risk and a latency sink. Strong candidates propose a **CQRS read model** (or at least a denormalized order-status projection) fed by events.
> 10. **Forgetting the user experience of async.** If order placement goes async, how does the customer know it worked? Strong candidates address the response contract: immediate "order received, we're processing" + a notification / status poll / push when confirmed, with clear failure messaging. They don't leave the customer staring at a spinner.

## Reference (what a strong answer covers)

**Crisis 1 — Resilience architecture:**
- Apply on **every synchronous call**: bounded **timeouts** (shorter than thread-pool saturation), **retries with exponential backoff + jitter** (capped, to avoid retry storms), **circuit breakers** (open on sustained failure to fail fast instead of queuing), and **bulkheads** (separate pools per downstream so one slow service can't starve the others).
- These **contain the blast radius** but don't fix the architecture. Be explicit: "circuit breakers prevent the cascade; bulkheads prevent pool exhaustion; neither removes the coupling."
- **The structural fix:** move **cross-service writes** off the synchronous chain. The order-placement write path becomes a saga (below). Synchronous calls remain only where an *immediate* answer is required (e.g., pre-flight validation: "is the item in stock *right now*?"), and even those get the resilience patterns above.

**Crisis 2 — The saga:**
- **Orchestration** is the recommended model for this workflow. A **OrderOrchestrator** (its own service or a process within Order) owns the state machine: `create order → reserve inventory → charge payment → confirm` (or compensate on failure).
- Orchestrator issues commands and listens for replies/events; it persists saga state so it can resume after a crash.
- **Steps and compensations:**
  1. `createOrder` (Order) → compensation: `cancelOrder` (mark order CANCELLED).
  2. `reserveInventory` (Inventory) → compensation: `releaseInventory` (free the reservation).
  3. `chargePayment` (Payment) → compensation: `refundPayment` (refund the charge).
  4. `confirmOrder` (Order) → terminal success; no compensation (terminal).
  - If step 3 fails, the orchestrator runs compensations for 2 then 1 in reverse: release inventory, cancel order. End state is consistent (no leaked reservation, no orphan order).
  - Every compensation and every step is **idempotent** (keyed by `sagaId` + `stepId`) so retries are safe.
- **Why orchestration over choreography:** money + explicit refunds + legible debugging. Choreography (each service reacts to events) is simpler to start but makes the end-to-end flow and its compensations implicit and hard to reason about when a refund misfires six hops deep. The orchestrator makes the state machine a first-class, observable artifact.

**Dual-write — the Outbox pattern:**
- The orchestrator writes its state transition **and** the outbound command/event in the **same DB transaction** to an *outbox* table. A separate process (or CDC) ships outbox rows to the broker. This avoids the "DB committed but event never published" (and inverse) inconsistency. No dual-write across DB + broker.
- Symmetrically, consumers apply messages **transactionally** with their side-effect where possible, or rely on idempotency + an applied-message log.

**Idempotency & ordering:**
- Every Payment (and every retryable step) carries an **idempotency key** (`sagaId`); duplicates collapse to the same result. No double-charges on retry.
- The saga state machine rejects stale/out-of-order transitions (e.g., a late `PaymentSucceeded` arriving after the saga already compensated is detected and ignored or reconciled).
- An **applied-message / event-log table** per consumer deduplicates redelivered messages.

**The read path (CQRS):**
- The customer's "order status" view is served from a **read model / projection** updated by events (`OrderCreated`, `InventoryReserved`, `PaymentCharged`, `OrderConfirmed`), not by live fan-out across services. This keeps the read path fast, decoupled, and cascade-proof.

**The customer experience:**
- Order placement returns immediately: "Order received, confirming…" Then a push/notification/pollable status delivers the confirmed-or-failed result. Failure messaging is explicit ("payment declined — your cart is saved, no charge was made"). The customer is never left staring at a spinner with no contract.

**Honest framing:** The resilience patterns stop the bleeding today; the saga + async write path is the durable cure. Both ship — in that order.

## Related Concept Notes
- [[Microservices Challenges]] — cascading failures, distributed transactions, sagas, the dual-write problem, Outbox pattern
- [[Microservices Communication]] — sync vs. async, circuit breakers, bulkheads, timeouts, retries, idempotency
- [[Structuring Microservices]] — database-per-service, single-writer, why cross-service writes can't be transactional
- [[Microservices Scaling Strategies]] — resilience patterns, CQRS, event-driven scaling
- [[Event-Driven Architecture]] — events as the async backbone for the saga and the read model
- [[Microservices Overview]] — the consistency/resilience trade-offs inherent to the style

---

*Last updated: 2026-07-15*
