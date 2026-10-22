---
tags: [flashcards, system-design, software-architecture, microservices]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Microservices Architecture — Interview Flashcards

## Overview & Characteristics

<!-- What is a microservices architecture, and what are its core characteristics? -->
<!-- ? -->
<!-- A collection of small, autonomous services, each **built around a business capability**, **independently deployable**, **owning its own private database**, and communicating over the network via APIs or events. Characteristics: business-capability alignment, independent deployment, data ownership (DB per service), network communication, decentralization (team-owned), failure-aware design, and "smart endpoints, dumb pipes." -->
<!-- #flashcards/system-design -->

<!-- What does "smart endpoints, dumb pipes" mean, and why does it matter? -->
<!-- ? -->
<!-- Business logic lives **in the services**, not in an intelligent middleware layer. The network (message bus, HTTP) is a simple, dumb transport. This avoids a fragile "smart bus" (the old enterprise-service-bus model) that becomes a bottleneck and single point of failure. Services own their logic; the pipe just carries bytes. -->
<!-- #flashcards/system-design -->

<!-- Why do companies like Netflix and Amazon use microservices — is it about traffic? -->
<!-- ? -->
<!-- No — it's about **organizational scale**. Hundreds of engineers needing autonomy, independent release cycles, and clear ownership ("you build it, you run it"). A monolith can serve enormous traffic; the driver toward microservices is team scaling. Copying Netflix's architecture at a 10-person startup is cargo-cult engineering. -->
<!-- #flashcards/system-design -->

## Identifying Microservices

<!-- What are the four lenses for identifying microservices? -->
<!-- ? -->
<!-- 1. **Business Capabilities** — what does the business do? 2. **Single Responsibility Principle** — one reason to change. 3. **Data Ownership** — who owns what data? 4. **Independent Deployability** — can it ship on its own? Apply all four together — no single lens is sufficient. -->
<!-- #flashcards/system-design -->

<!-- Why is decomposing by business capability better than by technical layer or entity? -->
<!-- ? -->
<!-- A business capability (Orders, Payments, Catalog) is a coherent function the business recognizes — it maps to a DDD bounded context, keeps changes that belong together in one service, and aligns with team ownership. Decomposing by technical layer ("Data Access Service") produces the wrong unit; by entity ("a service per table") produces chatty, anemic CRUD wrappers. -->
<!-- #flashcards/system-design -->

<!-- What does "data ownership" mean for a microservice, and why is it critical? -->
<!-- ? -->
<!-- Each service owns its **private database** — no shared schemas, no cross-service joins. Other services access its data only via its API (sync) or via event-carried copies (async). It's critical because data ownership both *defines* the boundary and *enforces* independence. Sharing a database is the fastest path to a distributed monolith. -->
<!-- #flashcards/system-design -->

<!-- How do you test whether a service boundary is correct? -->
<!-- ? -->
<!-- The independent-deployability test: **can I deploy this service alone, at 3 AM, without coordinating with another team?** If yes, the boundary is right. If no, you have temporal or data coupling and likely a distributed monolith. A correct boundary also maps to a business capability, has one reason to change (SRP), and owns its data. -->
<!-- #flashcards/system-design -->

<!-- What is a "distributed monolith," and what causes it? -->
<!-- ? -->
<!-- Services that look like microservices but behave like a monolith: share a database, must deploy together, or sit in long synchronous chains. All costs, no benefits. Caused by drawing boundaries that split tightly-coupled data, ignoring data ownership, or over-decomposing into chatty nanoservices. The #1 microservices anti-pattern. -->
<!-- #flashcards/system-design -->

<!-- Should you start with fine-grained or coarse-grained services, and why? -->
<!-- ? -->
<!-- **Coarse-grained.** It's much easier to split a too-big service later than to merge two too-small ones (merging means reconciling databases and release cycles). Start with fewer, coarser services and split when a clear, stable boundary emerges. "Make it coarse, split when it hurts." -->
<!-- #flashcards/system-design -->

## Structuring Microservices

<!-- What are the four pillars of structuring a microservice? -->
<!-- ? -->
<!-- 1. **Decompose by business domain** (bounded context). 2. **Define clear APIs** (explicit, stable, versioned contract). 3. **Choose the right granularity** (not too big, not too small). 4. **Implement observability** (logs, metrics, distributed tracing). Together they enable business alignment, independent deployment, right-sized scaling, and operability. -->
<!-- #flashcards/system-design -->

<!-- What makes a good microservice API? -->
<!-- ? -->
<!-- An **explicit contract** (documented via OpenAPI/Protobuf), **stable and backward-compatible**, **versioned** (breaking changes ship as `/v2`), **cohesive** (serves one capability), and **coarse-grained** (fewer, richer endpoints over chatty tiny ones). It exposes DTOs, never internal entities or DB schema — don't leak your internal model. -->
<!-- #flashcards/system-design -->

<!-- How do you know if a service is the right granularity? -->
<!-- ? -->
<!-- **Small enough to be owned by one team** and **big enough to do its job without constantly calling other services**. Too fine = CRUD wrapper, chatty, ops overhead > logic (merge it). Too coarse = multiple teams own it, multiple reasons to change, risky deploys (split it). Start coarse; split only when a clear sub-capability demands independence. -->
<!-- #flashcards/system-design -->

<!-- What are the three pillars of observability, and why are they mandatory in microservices? -->
<!-- ? -->
<!-- **Logging** (discrete events with context), **Metrics** (numeric time-series: rate, errors, latency), and **Distributed Tracing** (a request's journey across services, hop by hop). Mandatory because in a distributed system you can't attach a debugger — without tracing and correlation IDs, debugging a cross-service issue is a multi-hour forensic exercise. -->
<!-- #flashcards/system-design -->

<!-- Why must trace IDs be propagated across every service hop? -->
<!-- ? -->
<!-- A single user request may cross 5+ services. Without a **trace ID propagated at every hop**, you can't reconstruct the request's path, see where time was spent, or find which service failed. Trace IDs tie logs, metrics, and traces together into one coherent story across the whole system. -->
<!-- #flashcards/system-design -->

## Communication

<!-- What are the two fundamental communication modes in microservices, and when do you use each? -->
<!-- ? -->
<!-- **Synchronous** (caller blocks until response — REST, gRPC, GraphQL) and **Asynchronous** (sender emits and moves on — Kafka, RabbitMQ, SNS/SQS). Use **sync for reads** needing fresh data or immediate responses. Use **async for writes with side effects**, to decouple producer from consumer, to buffer spiky traffic, and for fan-out (many reactions to one event). -->
<!-- #flashcards/system-design -->

<!-- When would you choose REST vs gRPC? -->
<!-- ? -->
<!-- **REST** for public APIs, browser/mobile clients, and human-readable contracts — ubiquitous and cacheable. **gRPC** for internal service-to-service calls needing low latency and high throughput — binary (Protobuf), strongly typed, multiplexed over HTTP/2, supports streaming. Common split: gRPC between services, REST at the public API gateway. -->
<!-- #flashcards/system-design -->

<!-- What is temporal coupling, and why is it the main risk of synchronous communication? -->
<!-- ? -->
<!-- Both services must be **up and responsive at the same time**. If Service B is slow or down, Service A blocks, then times out — and if C called A, C blocks too. Failures cascade up the chain. Mitigate with timeouts, retries, circuit breakers, and by moving cross-service writes to async. Avoid long synchronous chains (A→B→C→D). -->
<!-- #flashcards/system-design -->

<!-- What's the difference between a message queue and pub/sub? -->
<!-- ? -->
<!-- **Queue (point-to-point):** one message processed by **exactly one** of N competing consumers — work distribution, background jobs (RabbitMQ, SQS). **Pub/Sub (topic):** one message fans out to **many subscribers**, each getting its own copy — broadcasting state changes (Kafka, SNS). Kafka supports both via partitions and consumer groups. -->
<!-- #flashcards/system-design -->

<!-- Why should cross-service writes prefer async over sync? -->
<!-- ? -->
<!-- Synchronous write chains create cascading failures — if payment is down, order placement fails too, even though the order is valid. With async, Orders emits `OrderPlaced` and returns success immediately; payment, inventory, and email react independently. If one is down, events buffer and catch up — no cascade. -->
<!-- #flashcards/system-design -->

<!-- What is service discovery, and why is it needed? -->
<!-- ? -->
<!-- Because service instances come and go (scaling, restarts, containers), callers need a way to find them. A **service registry** (Consul, Eureka, etcd) tracks live instances; callers use **client-side discovery** (pick an instance) or **server-side discovery** (a load balancer/sidecar picks). In Kubernetes, a `Service` gives a stable DNS name backed by dynamic pods. -->
<!-- #flashcards/system-design -->

## Challenges

<!-- Why is data consistency harder in microservices than in a monolith? -->
<!-- ? -->
<!-- With database-per-service, there are **no cross-service ACID transactions**. A single business operation (place an order) may need to update Orders, Payments, and Inventory — each in its own DB. If one fails mid-flow, you can't roll them all back. In a monolith, one transaction handles it; in microservices, you need sagas and eventual consistency. -->
<!-- #flashcards/system-design -->

<!-- What is the Saga pattern, and how does it solve cross-service consistency? -->
<!-- ? -->
<!-- A sequence of **local transactions** across services, where each step has a **compensating action** to undo it on failure. If step 2 (charge card) fails after step 1 (reserve inventory), the saga runs step 1's compensation (release inventory). No global ACID — consistency through compensations. Two flavors: **choreography** (events) and **orchestration** (central driver). -->
<!-- #flashcards/system-design -->

<!-- What is the dual-write problem, and how does the Outbox pattern solve it? -->
<!-- ? -->
<!-- When a service must update its DB *and* emit an event, doing them as two separate writes is unsafe — one might fail, leaving inconsistent state. The **Outbox pattern** writes the event to an `outbox` table in the **same transaction** as the data change, then a separate process (polling publisher or Change Data Capture like Debezium) publishes to the broker. Atomicity guaranteed. -->
<!-- #flashcards/system-design -->

<!-- Why is distributed tracing mandatory in microservices? -->
<!-- ? -->
<!-- A single request may cross 5+ services. Without a **trace ID propagated across every hop**, you can't tell which service was slow or failed — debugging is a multi-hour forensic exercise. Tracing (Jaeger, Zipkin, OpenTelemetry) assembles spans into a tree showing the full path, timing, and failure point. Mandatory, not optional. -->
<!-- #flashcards/system-design -->

<!-- What is a circuit breaker, and when does it help? -->
<!-- ? -->
<!-- A pattern that **stops calling a failing downstream service**. When Service B fails repeatedly, the circuit "opens" — A stops calling B and returns a fallback (cached data, default, or fast error) instead of blocking and cascading. After a cooldown, it "half-opens" to test recovery. Prevents one slow/down service from taking down its callers. -->
<!-- #flashcards/system-design -->

<!-- Why does microservices security need Zero Trust? -->
<!-- ? -->
<!-- A monolith has one trust boundary; microservices have many — every service-to-service call crosses a boundary. "Internal" doesn't mean "trusted." **Zero Trust** means every call authenticates and authorizes, never assumed safe because of network location. Implement with mTLS (often via a service mesh), OAuth2/JWT, network segmentation, and a vault for secrets. -->
<!-- #flashcards/system-design -->

## Scaling Strategies

<!-- What is horizontal scaling, and why is it the default for microservices? -->
<!-- ? -->
<!-- Adding **more instances** of a service behind a load balancer — each identical and interchangeable. It's the default because it has no hard ceiling, uses cheap commodity hardware, provides resilience, and enables zero-downtime deploys. The critical requirement is **statelessness** — any instance must be able to handle any request. -->
<!-- #flashcards/system-design -->

<!-- Why is statelessness critical for horizontal scaling? -->
<!-- ? -->
<!-- If an instance holds session state in memory, the load balancer must use **sticky sessions** (route the same user to the same instance) — which defeats even distribution and blocks elastic scaling. If that instance dies, the session is lost. Externalize state to Redis or use JWTs so **any instance can serve any request**, then the load balancer distributes freely and you add/remove instances at will. -->
<!-- #flashcards/system-design -->

<!-- What is auto-scaling, what triggers it, and what's its main caveat? -->
<!-- ? -->
<!-- Adding/removing instances automatically based on conditions: **metric-based** (CPU > 70%), **request-rate** (RPS threshold), **queue depth** (SQS backlog), or **schedule-based** (business hours). Benefits: cost efficiency and automatic spike handling. **Main caveat: auto-scaling a service doesn't fix a database bottleneck** — if 20 instances all hit one DB, you've just moved the problem. Also watch for cold-start latency and thrashing (use cooldowns). -->
<!-- #flashcards/system-design -->

<!-- What is sharding, and when should you use it? -->
<!-- ? -->
<!-- Splitting a database's **rows across multiple instances** by a shard key (user_id, tenant_id, region). Each shard holds a subset, scaling both reads AND writes. Use it only when a single primary DB truly can't keep up despite caching and read replicas — sharding adds major complexity: cross-shard queries/joins are hard, cross-shard transactions lose ACID, re-sharding is painful (use consistent hashing), and a bad key causes hot spots. -->
<!-- #flashcards/system-design -->

<!-- How does consistent hashing help with sharding? -->
<!-- ? -->
<!-- It minimizes **data movement** when adding or removing shards. With naive `hash(key) % N`, adding one shard requires rehashing nearly all data. Consistent hashing places shards on a "ring" — adding or removing a shard only moves the keys in that segment, not the entire dataset. It's the production standard for dynamically scalable sharding. -->
<!-- #flashcards/system-design -->

<!-- What's the recommended scaling progression, and why does order matter? -->
<!-- ? -->
<!-- 1. Optimize code & queries → 2. Caching → 3. Vertical scale DB → 4. Read replicas → 5. Auto-scale stateless services → 6. Shard → 7. CQRS. Order matters because each rung is cheaper and simpler than the one above. **Don't jump to sharding** before exhausting caching and replicas — they often solve the problem at 1/100th the effort. Sharding is near-irreversible complexity. -->
<!-- #flashcards/system-design -->

<!-- Why is caching the "cheapest scaling multiplier"? -->
<!-- ? -->
<!-- A well-placed cache (Redis/Memcached + CDN) can reduce database load by **90%+** for read-heavy workloads — most reads never reach the DB. It's far cheaper and simpler than sharding, often delaying or eliminating the need to shard. Always exhaust caching before considering physical database scaling. -->
<!-- #flashcards/system-design -->

## Synthesis & Design Questions

<!-- What is the "microservices tax," and how do you decide if you can pay it? -->
<!-- ? -->
<!-- The non-negotiable costs of distribution: per-service CI/CD, centralized logging, distributed tracing, metrics/alerting, containerization + orchestration, service discovery, secrets management, and contract testing. You pay it because the benefits (independent scaling, team autonomy) outweigh it *at organizational scale*. If your org can't pay it — small team, low DevOps maturity — stay monolithic; microservices will hurt more than they help. -->
<!-- #flashcards/system-design -->

<!-- How would you explain the trade-off of microservices to an interviewer? -->
<!-- ? -->
<!-- You trade **operational simplicity for independent scaling, deployment, and team autonomy**. Gain: scale and ship each service independently, with fault isolation. Pay: distributed-systems complexity (network failures, eventual consistency, sagas), per-service CI/CD and observability, harder testing/debugging. The honest framing: microservices solve the *team scaling* problem, not the traffic problem. -->
<!-- #flashcards/system-design -->

<!-- An interviewer asks: "Should we use microservices for our startup?" — how do you answer? -->
<!-- ? -->
<!-- Probably not yet. Start with a **modular monolith** — it's fastest to build, cheapest to operate, and you can't draw service boundaries until the domain stabilizes. Move to microservices only when you hit multiple of: large org (10+ teams), stable bounded contexts, independent scaling needs, different tech-stack requirements, and mature DevOps. "Monolith first" — extract services when a module demands independence. -->
<!-- #flashcards/system-design -->

<!-- What's the difference between microservices and n-tier architecture? -->
<!-- ? -->
<!-- N-tier decomposes by **technical concern** (presentation, cache, search, data), while microservices decompose by **business capability** (orders, payments, catalog). They often coexist: a microservices system may use an n-tier structure internally (each service has cache, app, data layers). N-tier is about *infrastructure tiers*; microservices are about *service boundaries* aligned to the business. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
