---
tags: [flashcards, system-design, software-architecture, layered, multi-tier]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Multi-Layered Architecture — Interview Flashcards

## Two-Tier (Two-Layered)

<!-- What is a two-tier (two-layered) architecture, and when might you still see it? -->
<!-- ? -->
<!-- An application split into a **client layer** (UI + business logic — a "fat client") and a **data layer** (database), with the client connecting **directly** to the database. No middle tier. You still see it in 1990s-era desktop business apps (VB6 + SQL Server), Microsoft Access apps, legacy POS systems, and small internal inventory tools. A web app with inline SQL is technically two-tier — and an anti-pattern. -->
<!-- #flashcards/system-design -->

<!-- What are the main advantages and disadvantages of two-tier? -->
<!-- ? -->
<!-- **Pros:** simplest layered structure, low latency for few users (direct client-to-DB connection), few moving parts, fast initial development, cheap to run. **Cons:** tight client-to-DB coupling (schema changes break every client), hard-to-update fat clients, security exposure (clients hold DB credentials and direct access), no reuse of business logic, poor scalability (connection-limited to a few hundred users), and no platform independence. -->
<!-- #flashcards/system-design -->

<!-- Why does two-tier scale poorly, and what's the ceiling? -->
<!-- ? -->
<!-- Every client holds a **direct database connection**, and the DB server has a hard ceiling on concurrent connections (hundreds, not thousands). The database is a shared, stateful bottleneck that's hard to scale horizontally. There's also no way to scale the logic tier independently because it's embedded in the clients. The ceiling is typically a few hundred concurrent users. -->
<!-- #flashcards/system-design -->

<!-- How would you mitigate two-tier's performance and scalability limits, and what's the honest long-term fix? -->
<!-- ? -->
<!-- **Short term:** connection pooling, caching, batch queries/stored procedures, read replicas, and database sharding. **Long term:** evolve toward three-tier by extracting business logic into a middle application layer — this unlocks centralized pooling, caching, independent scaling, and a security boundary, solving most of two-tier's problems at once. Don't keep band-aiding a two-tier system that's hit its wall. -->
<!-- #flashcards/system-design -->

## Three-Tier (Three-Layered)

<!-- What is a three-tier architecture, and what's the key advance over two-tier? -->
<!-- ? -->
<!-- An application split into **presentation** (thin UI), **application** (centralized business logic / API — the middle tier), and **data** (database). The presentation tier calls the application tier's API; only the application tier talks to the data tier. The key advance over two-tier is the **centralized, independently scalable middle logic tier** — shared by all clients, hiding the DB behind a trust boundary. -->
<!-- #flashcards/system-design -->

<!-- What are the main advantages of three-tier over two-tier? -->
<!-- ? -->
<!-- Centralized, reusable business logic (one API serves all clients), independent scalability (scale the app tier horizontally without touching the DB), a clear security boundary (clients never touch the DB), easier updates (change logic in one place), tech-stack flexibility per tier, parallel team work, and independent testability of each tier. -->
<!-- #flashcards/system-design -->

<!-- What are the main disadvantages of three-tier? -->
<!-- ? -->
<!-- More moving parts (three deployable tiers), network hops adding latency (Presentation → Application → Data), deployment complexity across tiers, higher infrastructure cost, distributed debugging (a bug can be in any tier), API versioning burden, and over-engineering for simple apps. -->
<!-- #flashcards/system-design -->

<!-- Why does three-tier scale better than two-tier? -->
<!-- ? -->
<!-- The application tier can scale **horizontally** (add more app servers behind a load balancer) independently of the database, and business logic is centralized so caching is effective for all clients. **Connection pooling** in the app tier means thousands of requests share hundreds of DB connections, instead of each client holding one. The DB is still the long-term ceiling, but three-tier delays that wall significantly. -->
<!-- #flashcards/system-design -->

<!-- How does keeping the application tier stateless improve scalability? -->
<!-- ? -->
<!-- A stateless app server can handle any request from any client, so a load balancer can distribute requests freely (round-robin) and you can add or remove app servers at will. If sessions live in app-server memory, you're forced into sticky sessions, which unbalance load and block elastic scaling. Externalize session state to Redis or use JWTs. -->
<!-- #flashcards/system-design -->

<!-- How would you mitigate the latency added by tier-to-tier network hops in three-tier? -->
<!-- ? -->
<!-- Colocate tiers in the same data center/VPC (low network latency), use fast binary protocols like gRPC/Protobuf for internal calls, batch requests to reduce round-trips, and cache hot data at the application tier so many reads never reach the database. -->
<!-- #flashcards/system-design -->

<!-- Why is letting the presentation tier talk directly to the database an anti-pattern? -->
<!-- ? -->
<!-- It re-creates two-tier's coupling and security holes: the client learns the DB schema, holds DB credentials, and bypasses the centralized business logic. It breaks the trust boundary, prevents logic reuse, and makes schema changes break clients. The presentation tier must always go through the application tier's API. -->
<!-- #flashcards/system-design -->

## N-Tier

<!-- What is an n-tier architecture, and how does it differ from three-tier? -->
<!-- ? -->
<!-- A generalization of layered architecture to **any number of tiers** (n > 3). Where three-tier fixes presentation, application, and data, n-tier adds dedicated tiers for concerns that need their own infrastructure: caching, search, integration, edge/security, background processing. You add a tier when a concern deserves its own scaling profile, technology, or security zone. -->
<!-- #flashcards/system-design -->

<!-- What are the main advantages and disadvantages of n-tier? -->
<!-- ? -->
<!-- **Pros:** granular per-tier scalability (scale only the hot tier), specialized technology per concern, strong separation of concerns, independent deployment/evolution per tier, fault isolation, flexible security zones, supports very high scale. **Cons:** latency compounds with every hop, high operational complexity, expense, harder debugging without distributed tracing, deployment coordination, over-engineering risk, and a larger failure surface. -->
<!-- #flashcards/system-design -->

<!-- When does three-tier become n-tier? -->
<!-- ? -->
<!-- When a concern needs its own scaling profile (search spikes independently of writes), needs its own technology (relational DB can't do fast full-text search), is cross-cutting enough to centralize (caching, auth, rate limiting at an edge tier), involves complex integration (many external systems → a messaging tier), or demands security separation (DMZ/edge tier). You grow into n-tier — you don't start there. -->
<!-- #flashcards/system-design -->

<!-- How does n-tier affect performance, and how do you mitigate the latency from many hops? -->
<!-- ? -->
<!-- Every tier adds a network hop and serialization, so latency grows with tier count. Mitigate by: colocating chatty tiers in the same data center, using gRPC (HTTP/2, binary) for internal calls, batching/aggregating requests, using async/event-driven handoffs for non-critical paths, and caching aggressively so most reads never reach the data tier. Specialized tiers (cache, search) can actually *reduce* net latency for the right workload. -->
<!-- #flashcards/system-design -->

<!-- How do you prevent one tier's failure from cascading across the system in n-tier? -->
<!-- ? -->
<!-- Use **circuit breakers** (stop calling a failing tier), **timeouts** (don't block forever), **retries with backoff**, and **graceful degradation** (serve stale cache when the data tier is down). Every cross-tier call should assume the callee can fail. Without these, one slow tier cascades and brings down the entire request path. -->
<!-- #flashcards/system-design -->

<!-- What's the over-engineering risk with n-tier, and how do you avoid it? -->
<!-- ? -->
<!-- Adding tiers "for completeness" that don't earn their keep — pure indirection and latency. Avoid it with a **cost/benefit gate**: a new tier must justify itself with an independent scaling profile, a distinct technology need, or a security requirement. If it doesn't, keep the concern inside an existing tier. Start at three-tier and add tiers only when a concern demands it. -->
<!-- #flashcards/system-design -->

<!-- How does n-tier differ from microservices? -->
<!-- ? -->
<!-- N-tier decomposes by **technical concern** (presentation, cache, search, data), while microservices decompose by **business capability** (orders, payments, catalog). They often coexist: a microservices system may use an n-tier structure internally (each service has cache, app, data layers). N-tier is about *infrastructure tiers*; microservices are about *service boundaries*. -->
<!-- #flashcards/system-design -->

<!-- Why is distributed tracing essential in n-tier systems? -->
<!-- ? -->
<!-- Because a single user request may cross 5+ tiers, and without tracing (Jaeger, Zipkin) plus correlation IDs propagated across every hop, root-causing a slow or failed request is guesswork. Tracing lets you see the full path, where time was spent, and which tier failed. It's mandatory, not optional, at n-tier scale. -->
<!-- #flashcards/system-design -->

## Comparison and Decision-Making

<!-- How would you decide between two-tier, three-tier, and n-tier in an interview? -->
<!-- ? -->
<!-- Match the tier count to the **scale and complexity of the concern**: **two-tier** for tiny internal tools with few users (direct client-to-DB is fine). **Three-tier** for virtually any web/mobile app — it's the modern default, with centralized logic, a security boundary, and horizontal app-tier scaling. **N-tier** when specific concerns (search, cache, integration, security) need their own scaling profile or technology. Start at three-tier and add tiers only when a concern demands it — never start at n-tier. -->
<!-- #flashcards/system-design -->

<!-- What's the single biggest scalability difference between two-tier and three-tier? -->
<!-- ? -->
<!-- In two-tier, every client holds a direct DB connection and the logic lives in the clients — you can't scale either independently, and the DB connection ceiling is hit at a few hundred users. In three-tier, the **application tier scales horizontally** behind a load balancer and uses **connection pooling** to share DB connections across thousands of requests. That one architectural change pushes the ceiling from hundreds to thousands of users. -->
<!-- #flashcards/system-design -->

<!-- In all three styles, what's the recurring performance/scalability mitigation theme? -->
<!-- ? -->
<!-- **Reduce load on the bottleneck tier.** In two-tier and three-tier the bottleneck is the database; in n-tier it can be any tier. The toolkit is consistent across all three: **caching** (absorb reads), **connection pooling** (share DB connections), **read replicas** (scale reads), **sharding** (scale writes), **batching** (reduce round-trips), **circuit breakers + graceful degradation** (survive failures), and — the honest long-term fix — **extracting overloaded concerns into their own tier** (which is exactly how two-tier becomes three-tier, and three-tier becomes n-tier). -->
<!-- #flashcards/system-design -->

## Real-World Design & Security

<!-- What is multi-tier architecture, and why is it used? -->
<!-- ? -->
<!-- A software design pattern that divides an application into multiple layers (tiers), each with a specific responsibility — commonly Presentation (UI), Business Logic (API/backend), and Data (database/storage). It's used to **improve scalability, maintainability, and modularity** by separating concerns: each tier can be developed, scaled, and deployed independently. Used in web applications, enterprise systems, and distributed applications. The tier count (2, 3, or n) is chosen based on the scale and complexity of the system. -->
<!-- #flashcards/system-design -->

<!-- How does scalability improve in a multi-tier system? -->
<!-- ? -->
<!-- Through four mechanisms: **Horizontal scaling** — add more instances of a tier (e.g., multiple web servers behind a load balancer). **Vertical scaling** — increase resources (CPU/RAM) of individual servers. **Decoupling layers** — each tier scales independently (e.g., the caching tier scales separately from the app tier). **Load balancing** — distributes requests across multiple servers. The key win over a monolith: you scale *only the tier under load* rather than the whole system. -->
<!-- #flashcards/system-design -->

<!-- Walk me through a real-world n-tier architecture — e.g., a banking system. -->
<!-- ? -->
<!-- **Presentation:** mobile app / web portal (React, Swift, Kotlin). **Business:** API services for transactions and authentication (Java, .NET, Node.js). **Data:** customer data and transaction history in SQL databases. **Additional specialized tiers:** a **security tier** (OAuth, JWT), a **caching tier** (Redis/Memcached for performance), and a **load-balancing tier** (NGINX, AWS ELB to distribute traffic). This tier separation ensures high availability, performance, and security — and each tier can be hardened and scaled to its own profile. -->
<!-- #flashcards/system-design -->

<!-- How does load balancing work in an n-tier architecture? -->
<!-- ? -->
<!-- It distributes traffic across multiple servers to prevent overload, applied at tier boundaries. **Between presentation and business:** a load balancer (NGINX, AWS ELB) directs traffic across multiple backend servers. **Between business and data:** requests route to replicated database servers to spread load. Common algorithms: **Round Robin** (each request to the next available server), **Least Connections** (to the least busy server), and **IP Hashing** (routes a user to the same backend for session consistency / sticky sessions). -->
<!-- #flashcards/system-design -->

<!-- When would you choose microservices over a traditional n-tier architecture? -->
<!-- ? -->
<!-- Choose microservices when: you need **independent scaling** (each service scales separately), you have a **large team** (different teams own different services), you want **technology flexibility** (different languages/frameworks per service), or you need **faster independent deployments**. Remember the distinction: n-tier decomposes by *technical concern* (cache, search, data), microservices by *business capability* (orders, payments). N-tier is simpler and preferred for small-to-medium apps; microservices pay off at organizational scale. They often coexist. -->
<!-- #flashcards/system-design -->

<!-- How would you secure inter-tier communication in a distributed multi-tier system? -->
<!-- ? -->
<!-- 1. **TLS/HTTPS** — encrypt all data between tiers. 2. **Authentication & Authorization** — JWT, OAuth, or API keys to secure APIs. 3. **Zero Trust security** — enforce strict identity-based access between every layer; never assume an internal call is trusted. 4. **Firewalls & network segmentation** — isolate critical services (the DB must never be directly reachable from the internet; put it in a private subnet). 5. **Intrusion Detection Systems (IDS)** — monitor unusual traffic between tiers. Defense in depth: each tier boundary is a checkpoint. -->
<!-- #flashcards/system-design -->

<!-- How would you design a fault-tolerant multi-tier system for high traffic (e.g., Netflix or Amazon)? -->
<!-- ? -->
<!-- **Multiple data centers / regions** — distribute traffic globally for disaster recovery. **Auto-scaling** — dynamically add/remove servers based on traffic. **Load balancing** — even traffic distribution across all services. **Database replication** — primary-replica setup so a replica promotes if the primary fails. **Circuit breaker pattern** — prevent cascading failures when a downstream tier is down. **Retry mechanisms** — automatically retry transient failures with backoff. **Event-driven architecture** — decouple tiers with Kafka/RabbitMQ so producers survive consumer outages. The theme: assume every tier *will* fail, and design each to degrade gracefully. -->
<!-- #flashcards/system-design -->

<!-- If you had to build a highly available banking application, what architecture would you choose and why? -->
<!-- ? -->
<!-- A **distributed n-tier architecture with microservices** — because banking needs all of: **security** (each tier has strict access control, defense in depth), **scalability** (independent services for transactions, fraud detection, customer data — each scales to its own load), **resilience** (failover mechanisms and replication ensure 24/7 uptime), and **data consistency** (ACID-compliant databases with replication for the transactional core). The n-tier backbone (presentation, API, cache, data) gives the structure; microservices aligned to banking capabilities give the team autonomy and independent scaling. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
