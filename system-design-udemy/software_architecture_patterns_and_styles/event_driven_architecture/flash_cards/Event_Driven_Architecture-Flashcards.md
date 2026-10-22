---
tags: [flashcards, system-design, software-architecture, event-driven]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Event-Driven Architecture — Interview Flashcards

## Overview & Definition

<!-- What is an event-driven architecture (EDA)? -->
<!-- ? -->
<!-- A style where components communicate by producing and consuming **events** — notifications that something happened. Producers emit an event and move on; consumers react asynchronously whenever ready. A broker (Kafka, RabbitMQ, SQS) mediates. The shift: from "do this and tell me when done" (synchronous command) to "this happened — react if you care" (asynchronous event). -->
<!-- #flashcards/system-design -->

<!-- What is an event, and how does it differ from a command? -->
<!-- ? -->
<!-- An **event** is an immutable record of something that **happened** (past tense: `OrderPlaced`, `PaymentFailed`) — a fact broadcast to anyone who cares. A **command** is a request to **do something** (imperative: `PlaceOrder`, `ChargeCard`) — directed at one specific recipient. Events are broadcast; commands are targeted. Mixing them is a common bug source. -->
<!-- #flashcards/system-design -->

<!-- What are the core characteristics of EDA? -->
<!-- ? -->
<!-- Producer–consumer decoupling, asynchronous by default, event as the unit of communication (immutable fact), reactive not imperative, eventual consistency, fan-out (one event → many reactions), and resilience via buffering (events queue when consumers are down). -->
<!-- #flashcards/system-design -->

<!-- Why use EDA — what are the main benefits? -->
<!-- ? -->
<!-- Loose coupling (add/remove consumers without touching producers), async responsiveness (producers don't block), scalability (consumers scale independently; brokers buffer spikes), resilience (events queue through consumer outages), extensibility (new reactions are purely additive), auditability (events form an immutable log), and replayability (rebuild state by replaying the log). -->
<!-- #flashcards/system-design -->

## Synchronous vs Asynchronous

<!-- What is the fundamental difference between synchronous and asynchronous communication? -->
<!-- ? -->
<!-- In **synchronous**, the caller **blocks** (waits) for a response — both parties must be available at the same time (temporal coupling). In **asynchronous**, the sender emits and **moves on immediately**; the receiver processes whenever ready via a broker. Sync is immediate but fragile; async is decoupled and resilient but eventually consistent. -->
<!-- #flashcards/system-design -->

<!-- What is temporal coupling, and why is it the main risk of synchronous systems? -->
<!-- ? -->
<!-- Both services must be **up and responsive at the same time**. If Service B is slow or down, Service A blocks, then times out — and if C called A, C blocks too. Failures cascade up the chain. Mitigate with timeouts, retries, circuit breakers, and by moving cross-service writes to async. -->
<!-- #flashcards/system-design -->

<!-- When would you choose synchronous over asynchronous? -->
<!-- ? -->
<!-- When you need an **immediate response** to proceed, for a **read** needing fresh data, for a simple query/response where the caller can't proceed without the answer, or when exposing a **public API**. Sync is simpler to reason about but creates temporal coupling. -->
<!-- #flashcards/system-design -->

<!-- Why should cross-service writes prefer async over sync? -->
<!-- ? -->
<!-- Synchronous write chains create cascading failures — if payment is down, order placement fails too, even though the order is valid. With async, the producer emits `OrderPlaced` and returns success; payment, inventory, and email react independently. If one is down, events buffer and catch up — no cascade. -->
<!-- #flashcards/system-design -->

<!-- How do most real systems combine sync and async? -->
<!-- ? -->
<!-- **Sync for reads** (a user needs an answer now) and **async for writes** with side effects (OrderPlaced → inventory, payment, email). The API gateway fronts sync client calls; internal cross-service writes flow through a broker. This gives immediate responses where needed and decoupling/resilience where possible. -->
<!-- #flashcards/system-design -->

<!-- What's the core trade-off between sync and async? -->
<!-- ? -->
<!-- **Synchronous trades resilience for immediacy** — confirmed response now, but both must be up (temporal coupling, cascades). **Asynchronous trades immediacy for resilience** — producer stays responsive and survives consumer outages, but eventual consistency and harder debugging. Choose by operation: reads → sync; writes with side effects → async. -->
<!-- #flashcards/system-design -->

## Pub/Sub vs Event Streaming

<!-- What is pub/sub, and when do you use it? -->
<!-- ? -->
<!-- A model where publishers emit events to a **topic**, and all current **subscribers** receive a copy — fan-out. Delivery is typically **ephemeral**: once delivered, the event is gone; late subscribers miss it. Use it for real-time broadcast and notifications. Tools: SNS, Google Pub/Sub, NATS. -->
<!-- #flashcards/system-design -->

<!-- What is event streaming, and when do you use it? -->
<!-- ? -->
<!-- A model where producers **append** events to a durable, ordered **log**, and consumers **read** at their own pace, tracking their position (offset). The log persists for a retention period and is **replayable**. Use it for event sourcing, audit, stream processing, and when consumers need to catch up or replay history. Tools: Kafka, Pulsar, Kinesis. -->
<!-- #flashcards/system-design -->

<!-- What's the key difference between pub/sub and event streaming? -->
<!-- ? -->
<!-- **Durability and replay.** Pub/sub is ephemeral — once delivered, the event is gone; late/restarting consumers miss it. Event streaming stores events in a durable, ordered log — consumers resume from their last offset and can replay the entire history. The decisive question: "Do consumers need to see events that happened while they were away?" No → pub/sub. Yes → streaming. -->
<!-- #flashcards/system-design -->

<!-- How does Kafka blur the line between pub/sub, queues, and streaming? -->
<!-- ? -->
<!-- Kafka uses **consumer groups**: within a group, each partition goes to exactly one consumer (**queue** — work distribution); across groups, every group gets all events (**pub/sub** — fan-out). Plus it's a durable log (**streaming** — replayable). One tool, three patterns, via partitions + consumer groups. Know which semantics you want before configuring. -->
<!-- #flashcards/system-design -->

<!-- When would you choose a message queue over pub/sub or streaming? -->
<!-- ? -->
<!-- When you need **work distribution**: one message processed by exactly **one** of N competing consumers (e.g., "process this video"). Pub/sub fans out to *all* subscribers; a queue distributes to *one* worker. Tools: RabbitMQ, SQS. -->
<!-- #flashcards/system-design -->

## Key Components

<!-- What are the four key components of an event-driven architecture? -->
<!-- ? -->
<!-- **Event Producers** (generate and publish events), **Event Brokers** (route and distribute), **Event Storage** (persist events durably), and **Event Consumers** (subscribe and react). Each plays a distinct role: producers detect and emit, brokers route and buffer, storage persists, consumers process. -->
<!-- #flashcards/system-design -->

<!-- What is the role of an event broker, and why is it critical infrastructure? -->
<!-- ? -->
<!-- The central middleware that **receives, routes, buffers, and optionally stores** events between producers and consumers — the heart of the EDA. It's critical (and a SPOF if not clustered) because if it goes down, events stop flowing. Always cluster and replicate it; monitor consumer lag, throughput, and error rates. -->
<!-- #flashcards/system-design -->

<!-- What's the difference between ephemeral and durable event storage? -->
<!-- ? -->
<!-- **Ephemeral** (pub/sub): events held only until delivered to active consumers, then discarded — late subscribers miss them (SNS). **Durable log** (streaming): events appended to a persistent, ordered log, retained for a configured period — consumers can catch up and replay (Kafka). Choose durable when consumers need replay or audit. -->
<!-- #flashcards/system-design -->

<!-- Why must consumers acknowledge (ACK) carefully? -->
<!-- ? -->
<!-- ACKing tells the broker the event is processed. **ACK before processing is done** → lost events if the consumer crashes. **ACK too late or never** → redelivery storms and broker storage fills. The correct pattern: process successfully, **then ACK**. If the consumer crashes before ACKing, the broker redelivers — which is why consumers must also be idempotent. -->
<!-- #flashcards/system-design -->

## Challenges

<!-- Why is eventual consistency a challenge in EDA, and how do you manage it? -->
<!-- ? -->
<!-- When a producer emits an event, the system is **not immediately consistent** — consumers haven't reacted yet. Manage it by: setting user expectations ("we're processing"), using **read models/projections** (CQRS) for queryable state, applying the **Saga pattern** for multi-step workflows, and accepting it for side-effects that don't need to be sync. -->
<!-- #flashcards/system-design -->

<!-- Why is ordering hard in EDA, and how do you preserve it? -->
<!-- ? -->
<!-- Events from different producers, partitions, or network paths can arrive out of order, and global ordering is expensive. Preserve order by **partitioning by entity ID** (e.g., `orderId`) so all events for one entity land in one partition in order; include **sequence numbers** so consumers discard stale events. Ordering is per-partition, not global. -->
<!-- #flashcards/system-design -->

<!-- Why must consumers be idempotent in EDA? -->
<!-- ? -->
<!-- At-least-once delivery means **duplicates are inevitable** — retries, network blips, consumer crashes before ACKing. A non-idempotent consumer will double-charge, double-send, or double-count. Deduplicate by `eventId`, design naturally idempotent operations, or track per-entity versions. At-least-once + idempotency = effectively exactly-once processing. -->
<!-- #flashcards/system-design -->

<!-- What is a poison message, and why is it dangerous? -->
<!-- ? -->
<!-- An event that **always crashes its consumer** (malformed data, null reference). Without protection, the broker redelivers it forever (consumer never ACKs), blocking the queue while other events pile up. Mitigate with a **dead-letter queue (DLQ)**: after N failed attempts, route the event to the DLQ for inspection. -->
<!-- #flashcards/system-design -->

<!-- Why is debugging harder in EDA than in synchronous systems? -->
<!-- ? -->
<!-- In a sync system, a stack trace shows the call chain. In EDA, "what happens when X fires?" is spread across many consumers reacting asynchronously — control flow is **implicit**. Without distributed tracing and correlation IDs, you can't tell which consumer processed or failed an event. Tracing is mandatory. -->
<!-- #flashcards/system-design -->

<!-- Why is schema evolution a challenge in EDA? -->
<!-- ? -->
<!-- Events are consumed by many services over years, and the producer doesn't know who consumes. Changing an event's structure breaks consumers you've forgotten about. Mitigate with **backward-compatible evolution** (add fields freely, never remove/rename without deprecation), a **schema registry**, and **schema versioning** in the event. -->
<!-- #flashcards/system-design -->

## Best Practices

<!-- What is idempotent event processing, and why is it the #1 EDA practice? -->
<!-- ? -->
<!-- A consumer is idempotent if **processing an event multiple times has the same effect as once**. It's #1 because at-least-once delivery means duplicates are inevitable. A non-idempotent consumer will double-charge, double-send, or double-count. Dedup by `eventId`, design naturally idempotent operations, or track per-entity versions. At-least-once + idempotency = effectively exactly-once. -->
<!-- #flashcards/system-design -->

<!-- Name three techniques for achieving idempotency in consumers. -->
<!-- ? -->
<!-- 1. **Deduplicate by `eventId`** — track processed IDs in Redis/DB; skip if already seen (general-purpose). 2. **Natural idempotency** — design inherently safe operations (`SET x = 5` is idempotent; `x += 1` is not). 3. **Versioned state** — use the event's sequence number; only apply if it advances the entity's state. -->
<!-- #flashcards/system-design -->

<!-- What is a dead-letter queue (DLQ), and why is it essential? -->
<!-- ? -->
<!-- A separate queue where messages that **fail to process repeatedly** are sent after max retries, so they don't block the main pipeline forever (the poison-message problem). Essential because without it, a single malformed event wedges the consumer indefinitely — other events pile up. Always configure a DLQ with max-retries, backoff, and monitoring. -->
<!-- #flashcards/system-design -->

<!-- How do you choose the right event broker? -->
<!-- ? -->
<!-- Match the broker to your need: **simple fan-out** → pub/sub (SNS); **work distribution** → queue (RabbitMQ, SQS); **durable log/replay/streaming** → Kafka/Pulsar; **complex routing** → RabbitMQ/EventBridge; **managed/low-ops** → cloud-native. Don't reach for Kafka for a simple notification — match the tool to the actual need. Start simple, upgrade when needed. -->
<!-- #flashcards/system-design -->

<!-- What is the "Kafka for everything" anti-pattern? -->
<!-- ? -->
<!-- Using Kafka (operationally heavy: clusters, partitions, offsets, retention) for workloads that don't need durable replay or high-throughput streaming — like a simple "send welcome email" notification. SQS/SNS or RabbitMQ would be simpler and cheaper. Start simple; move to Kafka only when you genuinely need replay, streaming, or event sourcing. -->
<!-- #flashcards/system-design -->

<!-- How do you handle event schema evolution safely? -->
<!-- ? -->
<!-- **Backward-compatible changes** (add fields freely, never remove/rename without deprecation), a **schema registry** (reject incompatible changes at publish time), **versioning in the event** (`schemaVersion`), **optional fields with defaults**, and **deprecation over removal**. Adopt a schema registry from day one — retrofitting is painful. -->
<!-- #flashcards/system-design -->

## Use Cases

<!-- When does EDA shine — what are the main use cases? -->
<!-- ? -->
<!-- 1. **Many independent reactions to one change** (fan-out: OrderPlaced → inventory, payment, email, analytics). 2. **Real-time/near-real-time processing** (streaming, dashboards, fraud). 3. **Decoupling producers from consumers at different scales**. 4. **Audit-heavy/compliance domains** (immutable event log). 5. **Tolerating consumer outages** (events buffer). 6. **Reactive UIs** (SSE, WebSockets, push). 7. **Event sourcing/CQRS** (replay, temporal queries). -->
<!-- #flashcards/system-design -->

<!-- When should you NOT use EDA? -->
<!-- ? -->
<!-- When you need **strong immediate consistency** (financial ACID), the flow is **simple and linear**, **sub-millisecond latency** is required, the system is **small/CRUD**, observability maturity is low (EDA without tracing is a black box), or the team isn't ready for eventual consistency. -->
<!-- #flashcards/system-design -->

<!-- Why is fan-out the canonical EDA use case? -->
<!-- ? -->
<!-- A single `OrderPlaced` event must trigger inventory, payment, email, analytics, and fraud check — many independent reactions. Doing this synchronously couples the order flow to five downstream services (any slow one hurts order placement). With EDA, the order service emits and returns; each consumer reacts independently. Adding a new reaction is purely additive. -->
<!-- #flashcards/system-design -->

<!-- How does EDA compose with other architectural styles? -->
<!-- ? -->
<!-- Rarely used alone. Most often paired with **microservices** (events instead of sync HTTP), **monoliths** (async side-effects), **serverless** (functions react to events — native model), and **CQRS** (commands write events, queries read from projections). Most real systems are hybrid: sync for reads, EDA for writes with side effects. -->
<!-- #flashcards/system-design -->

<!-- What's the decision framework for choosing EDA? -->
<!-- ? -->
<!-- Ask: 1. Need immediate confirmed response? (yes → sync). 2. Many independent reactions? (yes → EDA). 3. Need audit/replay? (yes → EDA). 4. Bursty traffic needing buffering? (yes → EDA). 5. Producer must survive consumer outages? (yes → EDA). 6. Domain tolerates eventual consistency? (no → sync). If 2+ point to EDA and the domain tolerates eventual consistency, EDA is likely right. -->
<!-- #flashcards/system-design -->

## Synthesis & Design

<!-- An interviewer asks: "How would you decide between sync, async pub/sub, and async streaming for a new service?" -->
<!-- ? -->
<!-- Decide by operation type and durability need: **Reads needing fresh data** → sync (REST/gRPC). **Writes with side effects needing fire-and-forget** → async pub/sub (SNS) if consumers don't need replay; **async streaming** (Kafka) if they need durable log, replay, audit, or event sourcing. For work distribution (one consumer per task) → queue (SQS/RabbitMQ). Match the tool to the need; don't default to Kafka for everything. -->
<!-- #flashcards/system-design -->

<!-- How would you ensure reliability in an event-driven system? -->
<!-- ? -->
<!-- Four pillars: 1. **Idempotent consumers** (dedup by `eventId` — at-least-once delivery means duplicates). 2. **Dead-letter queues** (poison messages don't wedge the pipeline). 3. **Distributed tracing** with correlation IDs (follow events across consumers). 4. **Schema registry + versioning** (evolve events without breaking consumers). Plus: monitor consumer lag, broker health, and DLQ depth. Assume events will be duplicated, out of order, and processed by consumers you can't see. -->
<!-- #flashcards/system-design -->

<!-- What's the Outbox pattern, and when do you need it? -->
<!-- ? -->
<!-- Solves the **dual-write problem**: when a service must update its DB *and* emit an event, doing them as two separate writes is unsafe (one may fail). The Outbox writes the event to an `outbox` table in the **same transaction** as the data change, then a separate process (polling publisher or CDC like Debezium) publishes to the broker. Atomicity guaranteed. Essential when DB writes and event emission must be consistent. -->
<!-- #flashcards/system-design -->

<!-- How do Kafka, RabbitMQ, and AWS EventBridge differ, and when would you pick each? -->
<!-- ? -->
<!-- **Kafka** — an **event streaming** platform: durable log, events stored for replay, ordered within partitions, highly scalable. Best for large-scale event processing, analytics, event sourcing. **RabbitMQ** — a **pub/sub + queue** broker: transient messages (consumed then gone), flexible routing, scales horizontally. Best for real-time messaging and task queues. **AWS EventBridge** — a **managed event bus**: no persistence, no strict ordering, scales with AWS infra. Best for integrating AWS services with cloud-native event-driven workflows. Match the tool: streaming → Kafka; pub/sub + queues → RabbitMQ; AWS glue → EventBridge. -->
<!-- #flashcards/system-design -->

<!-- What is an Event Transformation Layer, and when do you use it for schema evolution? -->
<!-- ? -->
<!-- An intermediary service (an "upgrader") that **translates older event versions to the new schema** before consumers see them. Use it when you can't do pure backward-compatible evolution (e.g., a field rename or structural change) and don't want every consumer to handle version logic. It sits between the broker and consumers, normalizing old events into the current format. Part of a broader schema-evolution toolkit alongside schema registries, versioning, and field deprecation. -->
<!-- #flashcards/system-design -->

<!-- How can Event Sourcing help manage eventual consistency? -->
<!-- ? -->
<!-- By making the **event log the source of truth** and deriving current state by **replaying events**. Instead of trusting that every consumer has caught up, any service can reconstruct the correct state by replaying the event history. This gives you: a complete audit trail, the ability to rebuild read models or fix corrupted state by replaying, and temporal queries ("what was the state on Tuesday?"). It doesn't eliminate eventual consistency, but it makes the system **self-healing** — state is always reconstructable from the log. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
