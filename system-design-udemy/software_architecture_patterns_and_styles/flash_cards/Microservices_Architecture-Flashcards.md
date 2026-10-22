---
tags: [flashcards, system-design, software-architecture, microservices]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Microservices Architecture — Interview Flashcards

## Definition and Core Rules

<!-- What is a microservices architecture? -->
<!-- ? -->
<!-- A style that structures an application as a collection of small, autonomous services, each **built around a business capability** (a bounded context), **independently deployable**, **owning its own private database**, and communicating over the network via APIs (sync) or events (async). Decomposition is by business capability — a vertical slice — not by technical layer. -->
<!-- #flashcards/system-design -->

<!-- What is the "database per service" rule, and why is it so important? -->
<!-- ? -->
<!-- Each service owns its private database; no service reads or writes another's tables directly. This enables loose coupling, independent deployment (no coordinated migrations), and polyglot persistence (catalog on Elasticsearch, orders on Postgres). The cost: no cross-service joins or ACID transactions, data duplication, and eventual consistency between services. -->
<!-- #flashcards/system-design -->

<!-- How do microservices communicate, and how do you choose between sync and async? -->
<!-- ? -->
<!-- **Synchronous (HTTP/gRPC):** caller waits; tight temporal coupling; good for reads needing fresh data. **Asynchronous (events via Kafka/RabbitMQ):** producer emits and moves on; decoupled and resilient; good for cross-service writes with side effects. Rule of thumb: **prefer async for cross-service writes** (avoids cascading failures from sync chains); use sync for reads or when an immediate response is essential. -->
<!-- #flashcards/system-design -->

## Pros and Cons

<!-- What are the main advantages of microservices? -->
<!-- ? -->
<!-- 1. **Independent scalability** — scale only the hot service. 2. **Independent deployment** — per-service release cycles, small low-risk releases. 3. **Team autonomy** — teams own services end-to-end. 4. **Fault isolation** — one crash doesn't take down others. 5. **Tech-stack heterogeneity** — right tool per job. 6. **Independent evolution** — rewrite/replace one service without a system-wide effort. 7. **Clear ownership** — "the Orders team owns the Orders service." -->
<!-- #flashcards/system-design -->

<!-- What are the main disadvantages of microservices, and which would you lead with in an interview? -->
<!-- ? -->
<!-- I'd lead with **distributed-systems complexity** — every call can fail; you need timeouts, retries, circuit breakers. Then: **operational overhead** (per-service CI/CD, monitoring, on-call), **eventual consistency** (no cross-service ACID), **distributed transactions become sagas**, **harder testing and debugging** (need distributed tracing), **data duplication**, **network latency** (in-process nanoseconds → network milliseconds), and **higher infrastructure cost**. -->
<!-- #flashcards/system-design -->

## Decision-Making

<!-- When should you use microservices? Name the conditions that must hold. -->
<!-- ? -->
<!-- Multiple should hold: large engineering org (10+ teams), complex domain with **stable** bounded contexts, need for independent scaling, different tech-stack requirements, mature DevOps practices (CI/CD, K8s, observability), and high availability requirements. One condition alone is rarely enough — and if DevOps maturity is low, microservices will hurt more than they help. -->
<!-- #flashcards/system-design -->

<!-- When should you NOT use microservices? -->
<!-- ? -->
<!-- Small team (1–10 engineers) — overhead swamps benefits. Unknown/evolving domain — you can't draw boundaries you haven't discovered. Low DevOps maturity — no CI/CD or observability. Simple CRUD system with modest traffic. Need for strong ACID across entities. Optimizing for early development speed (microservices slow you down early). -->
<!-- #flashcards/system-design -->

<!-- Explain the "monolith first" principle and why it matters for microservices. -->
<!-- ? -->
<!-- Start as a monolith and extract services only when a specific module demands independence. You learn the right boundaries by living with the domain first — refactoring between monolith modules is cheap, but fixing wrong service boundaries is a migration. Extract when a module needs independent scaling, belongs to a different team, needs a different release cadence/tech stack, or has stabilized boundaries. -->
<!-- #flashcards/system-design -->

## Key Patterns and Concepts

<!-- What is the "microservices tax," and why is it a gate? -->
<!-- ? -->
<!-- The non-negotiable platform capabilities you must build or buy before microservices pay off: per-service CI/CD, centralized logging, distributed tracing, metrics/alerting, containerization + orchestration, service discovery, secrets management, and contract testing. If your org can't pay this tax, microservices become an operational nightmare. DevOps maturity is a prerequisite, not an afterthought. -->
<!-- #flashcards/system-design -->

<!-- What is a saga, and why is it needed in microservices? -->
<!-- ? -->
<!-- A sequence of local transactions across services, where each step has a **compensating action** to undo it on failure. It replaces the impossible cross-service ACID transaction (with DB-per-service). Example: order placed → reserve inventory → charge card → confirm order, with compensations (release inventory, refund card) if any step fails. Sagas are far harder than a `BEGIN/COMMIT`. -->
<!-- #flashcards/system-design -->

<!-- Why is an API gateway used in microservices? -->
<!-- ? -->
<!-- It's a single entry point for clients: routes requests to the right service, handles cross-cutting concerns (auth, rate limiting, TLS, logging), can aggregate responses from multiple services (API composition / BFF pattern), and shields clients from the internal, ever-changing service topology. -->
<!-- #flashcards/system-design -->

## Anti-Patterns and Gotchas

<!-- What is a "distributed monolith," and how do you avoid it? -->
<!-- ? -->
<!-- Services that look like microservices but behave like a monolith: sharing a database, requiring coordinated deploys, or sitting in long synchronous chains. All the costs, none of the benefits. Avoid it by: giving each service its own data store, decoupling deployment cycles, and replacing synchronous chains with async events. It's the #1 microservices anti-pattern. -->
<!-- #flashcards/system-design -->

<!-- An interviewer asks "why do companies like Netflix and Amazon use microservices?" — what's the honest answer? -->
<!-- ? -->
<!-- Because of **organizational scale**, not raw traffic. Hundreds of engineers needing autonomy, independent release cycles, and clear ownership ("you build it, you run it"). A monolith can serve enormous traffic — the driver toward microservices is team scaling. This is why copying Netflix's architecture at a 10-person startup is cargo-cult engineering. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
