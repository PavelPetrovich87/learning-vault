---
status: archived
exercise_type: interview_simulation
difficulty: intermediate
pattern: event_driven
source_concepts:
  - "[[Event Driven Overview]]"
  - "[[Sync vs Async Systems]]"
  - "[[Pub Sub and Event Streaming]]"
  - "[[Event Driven Key Components]]"
  - "[[Event Driven Challenges]]"
  - "[[Event Driven Best Practices]]"
  - "[[Event Driven Use Cases]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: The Event Backbone of a Ride-Sharing App

## Type
**Interview simulation** — a system-design interview. The interviewer is an experienced engineer who probes broker choice, event design, ordering, and the operational realities of running a high-throughput event-driven system at scale.

## Difficulty
**Intermediate.** Tests the central event-driven design judgment call at "internet company" scale: choosing the right broker(s) for mixed workloads, partitioning for ordering, designing idempotent consumers under at-least-once delivery, and separating high-volume telemetry from business events.

## The Situation

You're in a system-design interview. The interviewer has given you this prompt:

> *"Design the event backbone for a ride-sharing platform — think Uber or Lyft. Riders request rides, drivers are matched in real time, GPS streams continuously from phones, and trip events drive billing, ratings, fraud detection, surge pricing, and the live map. Walk me through how the events flow, what carries them, and how you keep it correct and observable at scale."*

**Scale and workload (revealed by the interviewer if you ask):**
- **Geo footprint:** dozens of metro regions; each metro is largely independent operationally.
- **Concurrency:** ~200k active rides at peak globally; ~2M drivers and ~5M riders with the app open.
- **GPS stream:** every active driver phone emits a location update every 1–4 seconds. This is the dominant event volume by orders of magnitude — hundreds of thousands of events/second globally, low individual value, highly time-sensitive, mostly consumed by the matching engine and the live map and then discarded.
- **Business events:** `RideRequested`, `DriverMatched`, `TripStarted`, `TripCompleted`, `PaymentCaptured`, `RatingSubmitted`, plus surge-pricing recomputation signals. Far lower volume than GPS, but each one has audit, billing, and regulatory weight.
- **Consumers:** each business event fans out to several consumers — e.g., `TripCompleted` drives billing, driver/rider ratings prompts, fraud review, analytics, and surge-model updates.
- **Latency expectations:** driver-to-rider matching and map updates are near-real-time (low hundreds of ms end-to-end). Billing and fraud are not latency-sensitive but are correctness-sensitive.
- **Existing infra:** the company runs Kafka already for the data warehouse pipeline. The candidate may or may not choose to reuse it.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **Matching team** | Fresh driver locations, fast; ride requests dispatched in ms | Wants low-latency fan-out; hates anything that adds hops |
| **Billing** | Every chargeable event captured exactly once, in order, durably | A missed or duplicated trip event is real money |
| **Fraud** | A full, replayable history of trip + GPS events for investigation | Wants long retention and the ability to re-run detection |
| **Surge / pricing** | Continuous aggregate signal from supply and demand | Needs throughput, but can tolerate some lag |
| **Live map (rider/driver UX)** | Fresh, cheap location data, dropped if stale | Doesn't care about durability — stale GPS is worthless |
| **Data / analytics** | A clean, replayable stream into the warehouse | Wants Kafka with retention; retention costs money |
| **On-call / SRE** | One backbone to operate, not five; visibility into consumer lag | More brokers = more pager pain |

## Constraints (surface these if relevant)

- **Two fundamentally different workloads** live in the same system: a high-volume, ephemeral, time-sensitive GPS stream, and lower-volume, durable, correctness-critical business events. Conflating them is a classic mistake.
- **Ordering matters within an entity.** The events for a single ride must be processed in order (`TripStarted` before `TripCompleted`), and a driver's GPS trace must be ordered — but there's no meaningful global ordering across all rides.
- **Delivery is at-least-once.** Any broker choice will redeliver under failure. Consumers must tolerate it.
- **Audit / replay is a real requirement** for fraud and billing — events can't be pure fire-and-forget pub/sub with no retention.
- **Throughput is lopsided.** GPS dwarfs business events; letting GPS starve or delay billing events (or ballooning storage by retaining GPS for years) are both failure modes.
- **The company already operates Kafka.** Introducing a *second* broker must be justified; reusing the existing one must also be justified against the workloads.

## The Ask

The interviewer expects you to:

1. **Identify the event types and their consumers** — name the core events, who produces each, who consumes each, and what the fan-out looks like.
2. **Choose your broker(s)** — and justify it. Likely candidates: Kafka for the durable business stream and possibly a separate path for ephemeral GPS (a dedicated high-throughput Kafka topic with short retention, or a purpose-built pub/sub/WebSocket fan-out). Explain pub/sub-vs-streaming reasoning.
3. **Partition for ordering** — by what key (`rideId`, `driverId`), and why. Explain how you get per-ride ordering without paying for global ordering.
4. **Handle the GPS firehose** — separate the high-volume, low-value, ephemeral telemetry stream from the durable business stream; describe retention, consumer design, and back-pressure.
5. **Design idempotent consumers** — under at-least-once delivery, how does each consumer avoid double-billing / double-counting (dedupe by event id + ride id, idempotency keys downstream).
6. **Address schema and evolution** — how events are described (schema registry, Avro/Protobuf), and how you evolve them without breaking consumers.
7. **Make it observable** — consumer lag, throughput, DLQs, and alerting. What does on-call watch?

## Hidden Traps

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Pub/sub vs streaming is a real fork — not "Kafka for everything."** Real-time fan-out of ephemeral location data looks like pub/sub territory, but the audit/replay/surge-analytics needs of business events point at streaming with retention. A senior answer names *both* needs and either uses Kafka for both with sharply different topic configs, or splits the GPS firehose onto a separate mechanism. A junior answer says "Kafka" without justifying retention/throughput trade-offs.
> 2. **The GPS firehose is a different beast.** Conflating high-volume ephemeral GPS with durable business events starves billing under load and wastes storage on stale locations. A strong answer isolates GPS: dedicated high-throughput topic with short retention (hours), consumers that tolerate drops, possibly a separate fan-out (pub/sub or direct WebSocket/edge) for the live map. GPS is *cheap and lossy*; `TripCompleted` is *expensive and exact*.
> 3. **Partitioning by the right key.** Ordering across the whole system is impossible and unnecessary; ordering *within a ride* is mandatory. Partition by `rideId` for trip events (or `driverId` for the GPS trace). Partitioning by the wrong key (or by nothing) breaks ordering guarantees; a strong candidate states the guarantee they need and keys to it.
> 4. **The exactly-once myth.** Candidates sometimes claim "Kafka gives exactly-once." The honest statement: brokers offer at-least-once (and Kafka's transactions give effectively-once across its own produce+consume, but not end-to-end to your DB/API). The practical answer is **at-least-once + idempotent consumers**. A candidate who claims effortless exactly-once gets probed.
> 5. **Idempotency is the real correctness mechanism.** Under redelivery, billing must not double-charge, ratings must not double-submit, surge must not double-count. Dedupe on a stable event id (+ ride id), with a processed-events table per consumer. For payments specifically, propagate an idempotency key to the downstream processor.
> 6. **Schema and versioning are ignored at your peril.** With many consumers and a schema that evolves for years, an unversioned JSON event is a time bomb. A strong answer raises **schema registry** + a backward-compatible format (Avro/Protobuf) and a strategy (additive changes, consumer rollout before producer rollout).
> 7. **Consumer lag is the SLI.** An event-driven system fails *silently* — events pile up in the broker while consumers stall. A strong answer names consumer-lag monitoring, per-consumer DLQs with alerts, and throughput SLOs. "It's all async, we're done" misses operations entirely.
> 8. **Cross-metro isolation.** A hot partition or a bad consumer in one metro shouldn't degrade the global backbone. A strong answer at least mentions per-region/metro topic partitioning or regional clusters.

## Reference (what a strong answer covers)

- **Two streams, one backbone (Kafka):**
  - **Business events** (`RideRequested`, `DriverMatched`, `TripStarted`, `TripCompleted`, `PaymentCaptured`, `RatingSubmitted`) on durable Kafka topics, partitioned by `rideId`, multi-day/week retention for replay, consumed by billing, ratings, fraud, surge, analytics, data warehouse. Each fans out to multiple consumer groups.
  - **GPS telemetry** on a dedicated, high-partition-count Kafka topic, short retention (hours), cheap storage, consumed by the matching engine and the live-map fan-out service. Consumers are designed to tolerate drops and staleness; the live map may use a separate low-latency pub/sub or WebSocket edge layer fed from this topic.
- **Partitioning:** trip/business events keyed by `rideId` so all events for one ride land on one partition and are consumed in order per consumer group. GPS keyed by `driverId` so each driver's trace is ordered. No global ordering — and none needed.
- **Broker justification:** Kafka because the platform needs *durability, replay, multi-consumer fan-out, and ordering within a key* — pure pub/sub (SNS/SQS-style) can't replay for fraud and can't key-partition for ordering. The existing Kafka footprint makes reuse operationally cheaper than a second broker. GPS gets its own isolated topic config so it can't starve business events.
- **Idempotent consumers:** every consumer dedupes on event id (+ `rideId`) against a processed-events record; billing propagates an idempotency key to the payment processor; ratings and surge use the same dedupe pattern. The system is **at-least-once + idempotent** — "effectively once" — not relying on broker-level exactly-once.
- **Schema evolution:** a **schema registry** with Avro/Protobuf; changes are additive (new optional fields) and backward-compatible; new schema versions roll out consumers-first, then producers; incompatible changes go through a new topic or a versioned event type.
- **Observability:** per-consumer-group **lag** as the primary SLI, with alerting on sustained lag growth; throughput and error-rate dashboards; per-consumer **DLQs** for poison messages with paging alerts; capacity headroom on partition count for GPS.
- **Operational notes:** regional/metro-aware deployment to contain blast radius; separate consumer groups per concern so one slow consumer (e.g., fraud batch) doesn't block another (e.g., live map).

## Related Concept Notes
- [[Event Driven Overview]] — broker topologies, producers/consumers, fan-out
- [[Pub Sub and Event Streaming]] — the pub/sub vs streaming fork; Kafka topic/partition model; retention and replay
- [[Sync vs Async Systems]] — near-real-time vs batch consumers in the same system
- [[Event Driven Key Components]] — topics, partitions, consumer groups, schema registry, DLQs
- [[Event Driven Challenges]] — ordering via partitioning, idempotency, consumer lag, the exactly-once myth
- [[Event Driven Best Practices]] — schema evolution, consumer isolation, observability
- [[Event Driven Use Cases]] — real-time/telemetry-heavy domains

---

*Last updated: 2026-07-15*
