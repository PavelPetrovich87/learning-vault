---
tags: [flashcards, system-design, software-architecture, monolithic]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Monolithic Architecture — Interview Flashcards

## Definition and Core Characteristics

<!-- A junior engineer says "monoliths are bad architecture." How do you respond? -->
<!-- ? -->
<!-- That's a misconception. A monolith is a *style*, not a quality judgment. It's a single deployable unit with all functionality — UI, business logic, data access — in one process. It's the simplest thing to build, test, deploy, and operate, with fast in-process calls and trivial ACID transactions. Companies like Basecamp, GitHub, Shopify, and Stack Overflow run monoliths at scale. The right framing: a monolith is often the *correct* choice for small teams, early products, and modest traffic — it's premature decomposition that's usually the mistake. -->
<!-- #flashcards/system-design -->

<!-- What are the defining characteristics of a monolithic architecture? -->
<!-- ? -->
<!-- 1. Single deployable artifact (one binary, JAR, or Docker image). 2. Single process / runtime. 3. In-process communication (function calls, no network). 4. Shared infrastructure — typically one database, one config, one tech stack. 5. Atomic deployment — any change ships the whole app. -->
<!-- #flashcards/system-design -->

## Pros and Cons

<!-- What are the main advantages of a monolith, and how would you list them in an interview? -->
<!-- ? -->
<!-- 1. **Simplicity** — one codebase, one build, one deploy. 2. **Fast in-process calls** — no serialization or network latency. 3. **Trivial ACID transactions** — one transaction spans the whole domain. 4. **Easy testing** — end-to-end tests just boot the app. 5. **Low operational overhead** — one process to monitor and scale. 6. **Lower cost** — less infrastructure, fewer engineers needed. -->
<!-- #flashcards/system-design -->

<!-- What are the main disadvantages of a monolith, and which one would you lead with in an interview? -->
<!-- ? -->
<!-- I'd lead with **no independent scalability** — you scale the whole app even if only one module is hot. Then: **no fault isolation** (one bug crashes everything), **tech-stack lock-in**, **team friction** at scale (merge conflicts, long builds), **risky whole-app deployments**, and tendency toward a **Big Ball of Mud** without internal discipline. -->
<!-- #flashcards/system-design -->

## Use Cases and Decision-Making

<!-- When would you choose a monolith, and when would you move away from one? -->
<!-- ? -->
<!-- **Choose it** for: MVPs / early-stage products, small teams (1–10 engineers), modest predictable traffic, an evolving/unknown domain, strong ACID consistency needs, internal/CRUD tools. **Move away** when: a component needs independent scaling, multiple teams are stepping on each other, parts need different tech stacks or release cadences, or fault isolation becomes critical. -->
<!-- #flashcards/system-design -->

<!-- Explain the "monolith first" principle and why it's widely recommended. -->
<!-- ? -->
<!-- Most systems should start as a monolith and extract microservices only when a specific module demands independence. You can't know the right service boundaries until the domain stabilizes — refactoring between monolith modules is cheap (a few keystrokes), but fixing wrong service boundaries is a migration. Extract a service when a module needs independent scaling, belongs to a different team, needs a different release cadence, or has stabilized boundaries. -->
<!-- #flashcards/system-design -->

## Variants and Gotchas

<!-- What's a modular monolith, and why might you recommend it over a "pure" monolith? -->
<!-- ? -->
<!-- A monolith run as a single process but with **strictly enforced module boundaries** (often aligned to DDD bounded contexts), each owning its own data. It keeps the simplicity of a monolith while making a future move to microservices a clean extraction rather than a rewrite. If you're starting monolithic, this is the variant to build — it's the best of both worlds until you genuinely need distribution. -->
<!-- #flashcards/system-design -->

<!-- What's a "distributed monolith," and how do you avoid building one? -->
<!-- ? -->
<!-- Multiple services that are tightly coupled — sharing a database, requiring coordinated deploys, or calling each other synchronously in long chains. It has all the costs of microservices (network, ops) and none of the benefits. Avoid it by: giving each service its own data store, decoupling deployment cycles, and replacing synchronous chains with async events. It's the worst-of-both-worlds anti-pattern. -->
<!-- #flashcards/system-design -->

## Scaling a Monolith

<!-- How do you scale a monolith as traffic grows, without rewriting it as microservices? -->
<!-- ? -->
<!-- Progressively: 1. **Vertical scaling** (bigger machine). 2. **Horizontal scaling** (multiple instances behind a load balancer). 3. **Caching** (Redis, CDN) to offload reads. 4. **Read replicas** to split DB reads from writes. 5. **Extract only the hot module** into its own service, leaving the rest as a monolith. Most systems never need to go beyond step 2–3 — the monolith rarely dies, it just gets augmented. -->
<!-- #flashcards/system-design -->

<!-- Why do companies like Basecamp, GitHub, and Shopify run monoliths at enormous scale? -->
<!-- ? -->
<!-- Because raw traffic rarely forces microservices — monoliths scale far with horizontal replication, caching, and discipline. The real driver toward microservices is **organizational scale** (many teams needing autonomy), not requests-per-second. These companies prove the monolith's ceiling is much higher than people assume, as long as you keep internal structure disciplined (modular monolith). -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
