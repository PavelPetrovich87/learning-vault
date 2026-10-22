---
status: archived
exercise_type: planning_meeting
difficulty: beginner
pattern: event_driven
source_concepts:
  - "[[Event Driven Overview]]"
  - "[[Sync vs Async Systems]]"
  - "[[Pub Sub and Event Streaming]]"
  - "[[Event Driven Key Components]]"
  - "[[Event Driven Challenges]]"
  - "[[Monolithic Architecture]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: The 4-Second Checkout — Redesigning for Events

## Type
**Planning meeting** — a working design session with a backend lead. Concrete, opinionated, no hand-waving. The persona wants a redesign you can put into a sprint, not a lecture on theory.

## Difficulty
**Easy.** Tests the single most common entry point into event-driven thinking: recognizing a synchronous chain as the root cause of brittleness, and proposing an event-driven fan-out. The trap is over-engineering it.

## The Situation

**Company:** Northbound Goods — a growing DTC e-commerce site (~80k orders/day, peaking at 5x during promotions). The catalog and storefront are solid; the pain is all in checkout.

**The pain:** Order placement currently runs **synchronously**, in sequence, calling five downstream services before it returns "Order Confirmed" to the customer:

1. **Inventory** — reserve stock (and decrement on confirm).
2. **Payment** — authorize and capture the charge.
3. **Email** — send the order-confirmation email.
4. **Analytics** — record the purchase event for reporting.
5. **Fraud** — run a risk check.

End-to-end checkout latency is **3–5 seconds** in the steady state and spikes to 15+ seconds under load. Worse, the chain is **fragile**: when Payment times out, the whole order fails — even though the customer's money may have been captured on the processor side. On-call gets paged. Customer support gets tickets. The cart-abandonment rate during promotions is climbing.

**The meeting:** You're a senior backend engineer. The **backend lead** has pulled you into a planning meeting to redesign checkout. The lead has read about event-driven architecture and wants your take on whether it applies here — and a concrete sketch of the redesign if it does. The lead does *not* want buzzwords; the lead wants to know what changes, what the user sees, and what can break.

**Company context (revealed by the lead if you ask):**
- **Stack:** A modular monolith in Node.js; the five downstream services are a mix of owned services and third-party (a payment processor, an ESP, an analytics SaaS).
- **Team:** 6 backend engineers, 1 SRE. Observability is decent (metrics + tracing). There is *no* existing event broker in production.
- **Failure modes the lead cares about:** slow checkout, payment-timeout cascades, lost emails during outages, duplicated charges when customers retry.
- **Budget:** Cloud-conscious; the SRE is allergic to running Kafka unless it's justified.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **Backend lead (the persona)** | A concrete, low-risk redesign that ships in a quarter | Doesn't want a science project; wants to know what the user sees and what breaks |
| **Customers** | Instant confirmation; never get double-charged; never see a failure after paying | "Async" must not feel like "your order vanished" |
| **Payments team** | Payment auth must succeed before we promise the order | Pushing payment fully async risks promising goods we can't charge for |
| **Fraud team** | Want to block bad orders *before* fulfillment | Async fraud is fine for review, but you can't ship before the check clears |
| **Customer support** | A way to answer "where's my order?" | Async needs a status they can query — no black boxes |
| **On-call / SRE** | No new distributed-system nightmare | A broker is a new thing to operate; wants the *smallest* one that works |

## Constraints (surface these if relevant)

- **No broker in production today.** Introducing one is a real cost — the choice must be justified by the use case, not by resume appeal.
- **Payment auth is on the critical path.** You can't tell a customer "order confirmed" if their card was declined — so some part of payment is unavoidably synchronous.
- **Email/analytics are pure side-effects.** Nothing the customer sees depends on them succeeding instantly.
- **Fraud has a timing nuance:** it can run async for *review*, but high-risk orders shouldn't ship before the check completes.
- **Consumers must be idempotent.** Redelivery happens; a retried `OrderPlaced` event must not send two emails or capture two charges.
- **Failures need a home.** A consumer that can't process an event (bad data, downstream outage) can't just disappear it — it needs a dead-letter path and an alert.

## The Ask

The backend lead expects you to:

1. **Diagnose the root cause** — name the synchronous chain as the problem and explain *why* it makes checkout slow and fragile (latency addition, failure cascade, tight coupling).
2. **Propose an event-driven redesign** — what event is emitted, by whom, on what trigger; which downstream reactions become async consumers; and crucially, **what stays synchronous** and why.
3. **Describe the user experience** — what the customer sees and when (immediate "we're processing," deferred "confirmed," status polling/webhook), and how you avoid the "my order vanished" feeling.
4. **Pick a broker** — and justify it against the actual need (simple fan-out vs. high-throughput streaming). Defend not reaching for the heaviest option.
5. **Address operational reality** — idempotency of consumers, dead-letter handling for failed side-effects, at-least-once delivery and what it implies.
6. **Sketch the failure paths** — what happens when payment fails async? when email is down for an hour? when an event is delivered twice?

## Hidden Traps

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Root cause = the sync chain, not "we need Kafka."** A weak answer jumps to a technology. A strong answer first names the *architectural* problem: serial synchronous calls add latency and couple failure domains — any one slow/down service breaks the order.
> 2. **Sync vs async is per-reaction, not all-or-nothing.** The trap is making *everything* async. Payment **authorization** must stay synchronous (you can't confirm an order you can't charge). Email, analytics, and fraud-review are cleanly async. Inventory reservation is a judgment call. The candidate should reason per-reaction.
> 3. **The user-experience gap.** Async frees the request thread — but the customer still needs to know what happened. A strong answer specifies the contract: return "order received / processing" immediately, confirm via email/webhook/poll when payment settles, and expose a queryable order status for support.
> 4. **Idempotency is not optional.** At-least-once delivery means an `OrderPlaced` event may arrive twice. Each consumer (email, analytics, payment capture) must dedupe — typically by `orderId` (+ a processed-events table). A candidate who doesn't raise this hasn't operated an event system.
> 5. **Dead-letter queue for failed side-effects.** When the ESP is down, the email consumer must not block the stream and must not lose the event — it retries, then DLQs with an alert. "It'll just work" is the junior answer.
> 6. **Broker choice discipline.** The use case is **simple fan-out** of a handful of business events — this is textbook SNS+SQS (or EventBridge), *not* Kafka. Reaching for Kafka here is over-engineering the SRE would (rightly) reject. A strong candidate names the cheaper option and says *why* (no replay/retention/streaming need).
> 7. **Forgetting event ordering / causality.** If `OrderPlaced` and `OrderCancelled` can both exist, consumers must handle out-of-order arrival (a cancellation reaching email before the confirmation). At easy-difficulty, naming this as a *known* risk — even if deferring the full solution — signals seniority.
> 8. **The "confirmed but payment failed" landmine.** If you confirm to the user before payment fully settles async, you must have a reconciliation path: cancel the order, refund if captured, notify. The candidate should at least acknowledge this state machine.

## Reference (what a strong answer covers)

- **Diagnosis:** Checkout is slow and fragile because it's a **synchronous serial chain** — latencies add (5 calls × ~1s = multi-second p99), and a single timeout cascades to order failure. Each downstream service is also a failure domain coupled to the request.
- **The redesign:** The order service does only the work required to accept the order **synchronously** (validate cart, reserve inventory, **authorize payment**), then emits an **`OrderPlaced`** event to a topic and returns immediately to the customer with "order received, we're processing."
  - **Synchronous (stay in the request):** cart validation, inventory reservation, **payment authorization**. These define whether we can accept the order.
  - **Async consumers of `OrderPlaced`:** payment **capture**, confirmation email, analytics ingestion, fraud review, loyalty/points update.
- **User experience:** Return `202 Accepted` with an `orderId` and a "processing" state immediately. The customer gets an email on confirmation (and a pollable/webhook status). Support can query order status by `orderId`. Never a black box.
- **Broker:** A managed **pub/sub fan-out** is the right fit — **SNS + SQS** (or EventBridge) on AWS, or a managed topic on other clouds. **Not Kafka** — there's no need for retention, replay-at-offset, or high-throughput streaming; the SRE won't run a Kafka cluster for 5 events.
- **Reliability guarantees:**
  - **At-least-once delivery** — events may redeliver; every consumer is **idempotent** (dedupe on `orderId`/`eventId` against a processed-events record).
  - **Dead-letter queues** per consumer — after N retries, a poison event lands in a DLQ with an alert; nothing is silently dropped.
  - **Ordering** — acknowledged as a known risk; for the checkout flow, idempotency + status-machine guards (ignore `OrderPlaced` if the order is already `Confirmed`) handle the common cases.
- **Failure paths:** payment capture failing async flips the order to `Cancelled`, triggers a refund-if-captured + a cancellation email; ESP outage queues in SQS with retries then DLQ; a duplicate event is a no-op thanks to idempotency.

## Related Concept Notes
- [[Event Driven Overview]] — event-driven topology, brokers, producers/conducers
- [[Sync vs Async Systems]] — request-response vs event-driven; latency and coupling trade-offs
- [[Pub Sub and Event Streaming]] — when pub/sub (SNS/SQS) suffices vs. when you need streaming (Kafka)
- [[Event Driven Key Components]] — brokers, topics, consumers, DLQs
- [[Event Driven Challenges]] — idempotency, ordering, at-least-once delivery, dead-letter handling
- [[Monolithic Architecture]] — the starting point this is being redesigned from

---

*Last updated: 2026-07-15*
