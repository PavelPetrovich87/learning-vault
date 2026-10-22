---
tags: [flashcards, system-design, software-architecture, event-driven]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Event-Driven Architecture — Interview Flashcards

## Definition and Core Concepts

<!-- What is an event-driven architecture (EDA)? -->
<!-- ? -->
<!-- A style where components communicate by producing and consuming **events** — notifications that something happened. Producers emit an event and move on; consumers react asynchronously whenever they're ready. A broker (Kafka, RabbitMQ, SQS) mediates. The shift: from "do this and tell me when done" (synchronous command) to "this happened — react if you care" (asynchronous event). -->
<!-- #flashcards/system-design -->

<!-- What's the difference between an event, a command, and a message? -->
<!-- ? -->
<!-- A **command** is "do this" — directed at one specific recipient that must act (`ChargeCard`). An **event** is "this happened" — a fact broadcast to anyone who cares (`OrderPlaced`). A **message** is the generic envelope carrying either. Naming test: events are past-tense, commands are imperative. Mixing them — treating an event as a command by assuming exactly one consumer handles it — is a common bug source. -->
<!-- #flashcards/system-design -->

## Topologies and Patterns

<!-- What's the difference between pub/sub and queue topologies? -->
<!-- ? -->
<!-- **Pub/sub (topic):** one event fans out to many subscribers — each gets its own copy. Used for broadcasting state changes. **Queue (point-to-point):** one event handled by exactly one of N competing workers — used for work distribution and parallel processing. Kafka supports both via partitions (queue within a consumer group) and consumer groups (pub/sub across groups). -->
<!-- #flashcards/system-design -->

<!-- What are the three event patterns (notification, state transfer, sourcing), and when would you use each? -->
<!-- ? -->
<!-- **Event Notification:** event carries only an ID; consumer must call back for details — smallest payload, but creates a sync dependency. **Event-Carried State Transfer:** event carries full state; consumer is self-sufficient — fully decoupled but larger payload and schema coupling. **Event Sourcing:** the event log *is* the source of truth; current state is derived by replaying — complete audit + replay, but high complexity. Often paired with CQRS. -->
<!-- #flashcards/system-design -->

## Delivery Semantics

<!-- What are at-least-once, at-most-once, and exactly-once delivery? -->
<!-- ? -->
<!-- **At-most-once:** may lose events, never duplicates (lowest cost — no retries/acks). **At-least-once:** never loses, may duplicate — the most common default. **Exactly-once:** delivered exactly one time — the hardest, rarely truly end-to-end. In practice, "exactly-once" is usually **at-least-once delivery + idempotent consumers** that deduplicate by `eventId`. -->
<!-- #flashcards/system-design -->

<!-- Why must consumers be idempotent in EDA, and how do you achieve it? -->
<!-- ? -->
<!-- Because duplicates are inevitable — retries, at-least-once delivery, and network blips all cause them. A non-idempotent consumer will double-charge, double-send, or double-count. Achieve it by: deduplicating by `eventId` (track processed IDs in Redis/DB), designing naturally idempotent operations (`SET x = 5` is safe; `x += 1` is not), or tracking per-entity version numbers and only applying advancing versions. -->
<!-- #flashcards/system-design -->

<!-- How does ordering work in event-driven systems, and how do you preserve it? -->
<!-- ? -->
<!-- Most brokers guarantee order only **within a partition**, not globally. To keep one entity's events in order, partition by the entity ID (e.g., `orderId`) so all its events land in the same partition. Across entities or partitions, order is not guaranteed. Consumers must also track per-entity versions to discard stale or out-of-order events. -->
<!-- #flashcards/system-design -->

## Pros and Cons

<!-- What are the main advantages of event-driven architecture? -->
<!-- ? -->
<!-- 1. **Loose coupling** — producers don't know who consumes. 2. **Async responsiveness** — producers don't block. 3. **Independent scalability** — consumers scale separately; brokers buffer spikes. 4. **Resilience** — if a consumer is down, events queue up; producer is unaffected. 5. **Easy extensibility** — add a consumer without touching producers. 6. **Auditability** — events form an immutable log. 7. **Replayability** — rebuild state by replaying the log. -->
<!-- #flashcards/system-design -->

<!-- What are the main disadvantages of event-driven architecture, and which would you lead with? -->
<!-- ? -->
<!-- I'd lead with **eventual consistency by default** — the system settles over time, and reasoning about *current* state is hard. Then: **implicit control flow** (hard to trace what happens), **difficult debugging** without distributed tracing, **notoriously hard ordering/idempotency/exactly-once**, **schema evolution** across many consumers, **poison messages** blocking queues, and **testing complexity** (async, slow, flaky). -->
<!-- #flashcards/system-design -->

## Key Patterns and Problem-Solving

<!-- What is the dual-write problem, and how does the Outbox pattern solve it? -->
<!-- ? -->
<!-- When a service must update its database *and* emit an event, doing them as two separate writes is unsafe — one might fail, leaving inconsistent state. The **Outbox pattern**: write the event to an `outbox` table **in the same transaction** as the data change, then a separate process (a "polling publisher" or Change Data Capture like Debezium) reads the outbox and publishes to the broker. Atomicity guaranteed. -->
<!-- #flashcards/system-design -->

<!-- What is a dead-letter queue (DLQ), and why is it essential? -->
<!-- ? -->
<!-- A queue where messages that fail to process repeatedly are sent, so they don't block the main queue forever. Without a DLQ, a single poison message can wedge a consumer indefinitely. The DLQ lets problematic events be inspected and handled separately while the pipeline keeps flowing. Always configure one. -->
<!-- #flashcards/system-design -->

## Use Cases and Composition

<!-- When would you recommend event-driven architecture? -->
<!-- ? -->
<!-- When many independent reactions must fire from one change (OrderPlaced → inventory, payment, email, analytics), for real-time/streaming workloads, to decouple producers from consumers that scale differently, for audit-heavy domains (event sourcing), for fan-out workflows, to tolerate consumer outages, and for reactive UIs. Avoid it when you need strong immediate consistency, simple linear flows, or sub-millisecond latency. -->
<!-- #flashcards/system-design -->

<!-- How does EDA compose with other architectural styles? -->
<!-- ? -->
<!-- EDA is rarely used alone. Most often paired with **microservices** (services communicate via events instead of sync HTTP), **monoliths** (async side-effects like emails/analytics), **serverless** (functions react to events — the native serverless model), and **CQRS** (commands write events, queries read from projections). Styles compose — match each layer to the relevant factors. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
