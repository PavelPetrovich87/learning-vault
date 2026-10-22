---
status: archived
exercise_type: interview_simulation
difficulty: advanced
pattern: event_driven
source_concepts:
  - "[[Event Driven Overview]]"
  - "[[Sync vs Async Systems]]"
  - "[[Pub Sub and Event Streaming]]"
  - "[[Event Driven Key Components]]"
  - "[[Event Driven Challenges]]"
  - "[[Event Driven Best Practices]]"
  - "[[Event Driven Use Cases]]"
  - "[[Microservices Architecture]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: Never Double-Charge, Never Miss a Charge — A Payments Event Backbone

## Type
**Interview simulation** — staff-level system-design interview. The interviewer is a rigorous staff engineer who probes idempotency, the Outbox pattern, dead-letter handling, schema evolution across years, and the "exactly-once myth" with patience and precision. Senior-level reasoning about durability, replay safety, and the tension between synchronous client contracts and asynchronous processing is expected.

## Difficulty
**Advanced.** Tests the hardest event-driven scenario: a payments platform where correctness is absolute (never double-charge, never miss a charge), event lifetimes span years, schemas must evolve without breaking consumers, and the system must remain correct under replay, redelivery, partial failure, and network-level nondeterminism.

## The Situation

**Company:** Ledgerline — a payments processor handling card charges, refunds, settlements, and payouts for thousands of merchants. Real money, real regulators, real audit trails. A double-charge is a customer incident and a compliance report; a missed charge is lost revenue and a reconciliation nightmare.

**The mandate:** The platform is moving from a request-response model to an **event-driven core**. The CTO's requirement, delivered flatly: *"Never double-charge. Never miss a charge. Be able to tell any auditor exactly what happened to any transaction, ever."*

**The design problem:** Charge requests arrive via API (synchronous client expectation — the merchant's server wants a fast answer), but the actual processing that follows — fraud checks, card-network authorization, capture, settlement, ledger writes, merchant notification, reconciliation — is inherently a multi-step, long-lived, failure-prone workflow. The candidate must design an event-driven system that:

- Accepts a charge request and gives the merchant a synchronous, correct response.
- Drives the subsequent steps asynchronously via events, durably.
- Achieves **effectively-once** processing end-to-end — no duplicate charges, no lost charges — *despite* the broker delivering at-least-once.
- Survives replay (re-processing old events must never double-charge).
- Evolves its event schemas over years without breaking consumers or the audit trail.
- Never silently drops a transaction: poison messages and unrecoverable failures have a home.

**Scale and context (revealed by the interviewer if you ask):**
- **Volume:** ~5k charges/sec at peak; settlements batch periodically.
- **Durability:** events must be retained for **years** (regulatory/compliance), queryable for audit, and replayable for investigation and disaster recovery.
- **Downstream:** the card networks (Visa/MC) and processors (Stripe-style) are synchronous, eventually-consistent, idempotency-key-aware, and occasionally wrong (a capture can succeed at the network but timeout to us).
- **Today's pain:** dual-write bugs (the DB commits but the event is lost, or vice-versa); duplicate charges on retries; no clean audit trail; schema drift across consumer services.
- **Team:** mature platform engineering; runs Kafka; has CI/CD, tracing, feature flags — the DevOps tax is paid.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **Merchants (clients)** | A fast, synchronous answer to "did it work?" | The truth (captured funds) settles asynchronously; a fast answer can be a lie |
| **Compliance / audit** | A complete, immutable, replayable record for years | Audit retention fights storage cost and schema churn |
| **Risk / fraud** | A chance to block bad charges before capture | Adds a step before money moves; must not stall good charges |
| **Finance / reconciliation** | Every charge, refund, and settlement ties out to the penny | Partial failures and network timeouts are the enemy of clean books |
| **Card networks / processors** | Idempotency keys, their own retries and timeouts | A retried charge without an idempotency key = a double charge |
| **On-call / SRE** | No silent money loss, no 3 AM poison-message fires | Needs DLQs, replay safety, clear ownership of stuck transactions |
| **Engineering / future you** | Schemas that evolve for years without rewrites | Event contracts outlive the code that first wrote them |

## Constraints

- **Correctness is absolute.** A double-charge or a missed charge is a financial and compliance incident. The system must achieve **effectively-once** end-to-end — not rely on broker-level guarantees.
- **The dual-write problem is central.** Writing to the DB and emitting an event are two operations; one can fail after the other succeeds. Naive "commit then publish" loses events on crash; "publish then commit" can publish effects of a rolled-back transaction. This must be solved, not hand-waved.
- **Delivery is at-least-once.** Any broker will redeliver under failure. Consumers and downstream processors must tolerate it. Claiming "the broker does exactly-once" is insufficient.
- **Replay must be safe.** Re-processing an event (disaster recovery, new consumer backfill, fraud re-run) must never duplicate a charge or a ledger entry. Idempotency must hold across replays, not just within a session.
- **Schemas live for years.** Events written today will be consumed and audited for years; the schema must evolve backward-compatibly without breaking old consumers, old stored data, or the audit trail.
- **Poison messages happen.** Some events will be unprocessable (corrupt, reference a deleted entity, hit a network bug). They cannot be dropped, and they cannot block the stream forever — they need a DLQ and ownership.
- **Synchronous client contract.** The merchant expects a low-latency API response, but the full processing is async. The design must reconcile the two without lying to the client.
- **External idempotency keys.** The card network/processor accepts an idempotency key; replaying the same key returns the original result instead of a duplicate charge. This is a load-bearing correctness mechanism.

## The Ask

The interviewer expects a design that addresses:

1. **The end-to-end effectively-once story** — how the system never double-charges and never misses a charge, across broker redelivery, consumer crashes, network timeouts, and replay. Name the mechanism at each step.
2. **The dual-write problem and the Outbox pattern** — how the DB write and the event emission happen atomically, so neither can be lost in isolation.
3. **Idempotency throughout the pipeline** — consumer-level dedupe (event id), and idempotency keys propagated to the card network/processor so a retried charge is a no-op, not a duplicate.
4. **Dead-letter handling** — what happens to unprocessable events; how they're parked, alerted, and resolved without silent loss.
5. **Schema evolution over years** — schema registry, compatibility rules, how you evolve without breaking old consumers or the audit trail, and how you handle the years of already-stored events.
6. **Replay safety** — what makes re-processing old events safe; the relationship between idempotency and replay.
7. **The sync-vs-async tension** — how the API gives the merchant a correct synchronous answer while the heavy processing is async, without lying and without blocking.
8. **The audit trail** — how the system can answer "what happened to transaction X?" for any regulator, years later.

## Hidden Traps

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **The exactly-once myth.** Candidates often claim "Kafka transactions give exactly-once." The rigorous truth: Kafka's transactions provide effectively-once across its own *consume-transform-produce* loop — they do **not** extend end-to-end to your database writes or the card network. True exactly-once *delivery* is rare and usually irrelevant; what payments actually needs is **at-least-once delivery + idempotent side effects** ("effectively-once processing"). A staff candidate states this clearly and designs the idempotency, not the guarantee.
> 2. **The dual-write problem, hand-waved.** "We commit to the DB, then publish the event" loses the event on a crash between the two. "Publish then commit" can emit an event for a transaction that rolls back. The staff answer is the **Outbox pattern**: write the business state *and* the outbox row in one DB transaction; a separate relay publishes the outbox row to the broker and marks it sent. Neither half can be lost in isolation. Any candidate who doesn't raise the dual-write problem unprompted hasn't operated this.
> 3. **Idempotency keys to the network — the real backstop.** Broker-level dedupe isn't enough because the *side effect* (a card charge) lives outside the broker. A staff answer propagates an **idempotency key** (e.g., the charge id or a client-supplied key, à la Stripe) to the card network/processor, so a retried authorization/capture returns the original result instead of charging twice. This is what makes *replay* and *redelivery* safe at the money layer.
> 4. **Replay safety is a separate property, not implied.** "Idempotent consumers" must be idempotent *across replays*, including months later — which means the dedupe store must outlive any individual processing run, and side-effecting calls must carry stable idempotency keys. A candidate who says "we're idempotent" but can't explain what happens when a two-year-old event is replayed hasn't closed the loop.
> 5. **Poison messages and the DLQ.** Some events are unprocessable and retrying forever blocks the partition (or, worse, silently retried indefinitely). A staff answer specifies: bounded retries, then a **dead-letter queue** with alerts and **clear ownership** (a human/ops process resolves them), and crucially — a poison message in payments must never be silently dropped or auto-acknowledged; it must be visible and reconcilable.
> 6. **Schema evolution over years.** With events retained for years and consumers evolving independently, an unmanaged schema is a time bomb. The staff answer: a **schema registry**, a self-describing or registered format (Avro/Protobuf), strict **backward-compatible** evolution rules (additive fields, no breaking renames), a producer-rollout-gated-on-consumer-readiness process, and a plan for the *already-stored* historical events (they must remain readable under new schemas — which is exactly what backward compatibility buys).
> 7. **The sync-vs-async tension, unresolved.** Merchants want a fast synchronous answer; the truth settles async. A weak answer blocks the API until everything finishes (defeats the purpose) or returns a blind "ok" that may be wrong (lies). A staff answer draws the line clearly: synchronously validate, durably record the *intent* (via Outbox), and return a charge id + a status the merchant can poll or receive via webhook; the authoritative state evolves async, and the client contract reflects that ("authorized" vs "captured" vs "settled").
> 8. **Event sourcing as the audit trail — but not a toy.** A staff candidate may frame the event log as an append-only **event sourcing** backbone: each `ChargeRequested`, `Authorized`, `Captured`, `Settled`, `Refunded` is a fact; the current state is a projection. This gives the auditor the exact history of any transaction. But they must also address: rebuilding projections from the log (replay safety again), and the cost/complexity of operating it. Naming it without hand-waving the trade-offs is seniority; claiming it solves everything is naivety.
> 9. **Network timeouts with successful side effects.** The card network can *succeed* at capturing a charge while timing out to us. Retrying naively without an idempotency key double-charges. The staff answer treats "timeout" as "unknown state" and resolves via a network *inquiry* keyed by the idempotency key before retrying — never a blind retry.
> 10. **Ignoring reconciliation.** Even with perfect idempotency, a daily reconciliation job that compares the event log, the ledger, and the network's settlement file is the final safety net. A staff answer names it as the backstop that catches the one-in-a-million drift no in-band mechanism catches.

## Reference (what a strong answer covers)

- **Effectively-once, end to end — the honest framing:** The system is **at-least-once delivery + idempotent side effects**. No reliance on broker-level exactly-once for correctness. Each layer names its mechanism:
  - Broker: at-least-once (redelivery tolerated).
  - Consumer: dedupe every event by a stable **event id** against a processed-events record, in the same transaction as the side effect.
  - Card network/processor: send a stable **idempotency key** (charge id or client-supplied) so a retried authorization/capture returns the original result.
  - Reconciliation: a daily job diffs the event log, the ledger, and the network settlement file; drift is a paging incident.
- **The Outbox pattern (dual-write solution):** The charge service writes its business state **and** an outbox row (the event payload) in a **single DB transaction**. A separate **transactional outbox relay** (or CDC, e.g., Debezium) reads unsent outbox rows, publishes them to the broker, and marks them sent — with at-least-once semantics tolerated by consumer idempotency. The DB commit and the event emission can never diverge.
- **Idempotency & replay safety:** Consumers store processed `(eventId)` (or ` eventId + consumer`) in a table co-committed with their side effect, so a redelivered or *replayed* event is a no-op. Downstream, the idempotency key to the card network makes replay safe at the money layer: re-processing a two-year-old `CaptureRequested` returns the original capture, not a new charge. This is what makes disaster-recovery and backfill safe.
- **Dead-letter handling:** Each consumer has bounded retries (exponential backoff); after N failures the event lands in a **DLQ** with full context and an alert. In payments, nothing is auto-acknowledged-or-dropped — DLQ entries are reconciled by an on-call + finance process, and a stuck transaction is always visible and resolvable.
- **Schema evolution over years:** A **schema registry** with Avro/Protobuf; evolution is strictly **backward-compatible** (additive optional fields, no breaking changes without a new event type or a versioned topic). Producers roll out new schemas only after consumers are ready; the registry enforces compatibility at deploy time. Historical events stay readable because backward compatibility is maintained — the audit trail spanning years is queryable under the current schema.
- **Sync vs async tension — resolved:** The API synchronously validates the request, writes the `ChargeRequested` intent + outbox row in one transaction, and returns a **charge id** with a status the merchant polls or receives via **webhook**. The client contract distinguishes `pending → authorized → captured → settled` (or `declined`/`failed`) — it never lies, and it never blocks on the full async pipeline. The merchant's fast answer is "we have your request, here's its id and its current state," not "the money moved."
- **Audit trail via event sourcing:** The event log is an append-only sequence of facts (`ChargeRequested`, `Authorized`, `Captured`, `Settled`, `Refunded`, `Disputed`). Current charge state is a **projection** rebuilt from the log. For any regulator query, the full history of any transaction is replayable from the log — the system can answer "what happened to this charge?" exactly, years later. Projections can be rebuilt from the log safely *because* replay is idempotent.
- **Unknown-state handling at the network:** A timeout from the card network is treated as **unknown**, not failure — the system issues an **inquiry** keyed by the idempotency key before deciding to retry or mark failed. No blind retries on the money path.

## Related Concept Notes
- [[Event Driven Overview]] — event-driven topology, event as immutable fact, the backbone concept
- [[Event Driven Challenges]] — the exactly-once myth, dual-write problem, ordering, idempotency, poison messages
- [[Event Driven Best Practices]] — Outbox pattern, schema registry, idempotency keys, DLQs, reconciliation
- [[Pub Sub and Event Streaming]] — Kafka durability/retention, replay, at-least-once semantics
- [[Event Driven Key Components]] — brokers, schema registry, outbox relay, DLQs, projections
- [[Sync vs Async Systems]] — the synchronous-client-vs-async-processing tension and how to resolve it
- [[Event Driven Use Cases]] — payments/financial as the canonical correctness-critical domain
- [[Microservices Architecture]] — service boundaries, independent consumers, contracts in an event-driven system

---

*Last updated: 2026-07-15*
