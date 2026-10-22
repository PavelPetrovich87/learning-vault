---
tags: [flashcards, system-design, software-architecture, layered]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Layered Architecture — Interview Flashcards

## Definition and the Dependency Rule

<!-- What is a layered (n-tier) architecture, and what are the classic layers? -->
<!-- ? -->
<!-- A style that organizes an application into horizontal tiers, each with a single responsibility, with strict top-down dependencies. The classic four-layer model: **Presentation** (UI/API) → **Application/Business** (use-case orchestration) → **Domain** (core business rules) → **Data Access** (persistence). A higher layer may call the one directly below; a lower layer must never call back up. -->
<!-- #flashcards/system-design -->

<!-- What is the dependency rule in layered architecture, and what happens when you break it? -->
<!-- ? -->
<!-- Dependencies always point **downward and inward** — presentation depends on application, application on domain, domain on data access. Lower layers must never reach up to higher ones. Break it and you get circular and upward dependencies, which break compilation, testing, and reasoning — the fastest path to a Big Ball of Mud. -->
<!-- #flashcards/system-design -->

<!-- What's the difference between closed and open layers? -->
<!-- ? -->
<!-- **Closed (strict):** a layer may only call the one directly below it — maximizes decoupling and swappability. **Open (relaxed):** a layer may skip intermediate layers (e.g., presentation calling data access for a fast read) — reduces pass-through overhead but weakens the abstraction. Most production systems default to closed and open layers only where a skip measurably improves performance. -->
<!-- #flashcards/system-design -->

## Pros and Cons

<!-- What are the main advantages of layered architecture? -->
<!-- ? -->
<!-- 1. **Clear separation of concerns** — each layer has one job. 2. **Layer swappability** — swap the DB or UI without touching others (they communicate via interfaces). 3. **Independent testability** — mock the layer below. 4. **Predictable dependency direction** — easy to navigate. 5. **Parallel team work** — teams own different layers. 6. **Familiar** — the most widely understood style; fits MVC web apps. -->
<!-- #flashcards/system-design -->

<!-- What are the main disadvantages of layered architecture, and which would you lead with in an interview? -->
<!-- ? -->
<!-- I'd lead with **pass-through / anemic middle layers** — in CRUD apps, the business layer often just forwards calls to data access, adding no value. Then: **vertical features touch every tier** (adding a field end-to-end ripples through all layers), **indirection and latency** from cascading down the stack, **monolithic deployment by default**, and (in the traditional form) the **domain depending on the database** — the most valuable code coupled to the most volatile. -->
<!-- #flashcards/system-design -->

## Dependency Inversion

<!-- How do hexagonal, onion, and clean architecture relate to layered architecture? -->
<!-- ? -->
<!-- They're **dependency-inverted refinements** of layered. The key difference: in traditional layered, the *domain depends on data access* (it knows about the DB). In hexagonal/onion/clean, the **domain defines an interface** (a port, like `UserRepository`) and the data-access layer *implements* it — so the domain depends on nothing and the DB is swappable. It's one inverted dependency, and it fixes layered's biggest flaw. -->
<!-- #flashcards/system-design -->

<!-- In an interview, how would you draw the contrast between traditional layered and clean/hexagonal dependency direction? -->
<!-- ? -->
<!-- **Traditional:** Domain → Data Access (domain calls the DB, depends on infrastructure). **Inverted (clean/hexagonal):** Domain defines a `UserRepository` interface; the Data Access layer *implements* it. Dependencies point **inward toward the domain**. Result: the business rules are protected from infrastructure churn, and you can swap Postgres for MongoDB without touching the domain. -->
<!-- #flashcards/system-design -->

## Use Cases and Anti-Patterns

<!-- When would you use layered architecture, and when is it a poor fit? -->
<!-- ? -->
<!-- **Use it** for: CRUD / line-of-business apps, MVC web applications, internal tools, systems with a stable domain, teams new to architecture, and apps needing swappable persistence or multiple frontends. **Poor fit** when: the domain is complex and central (prefer hexagonal/clean), components need independent scaling/deployment (prefer microservices), features are mostly vertical slices, or the system is event-heavy and real-time. -->
<!-- #flashcards/system-design -->

<!-- What's an "anemic domain model," and why is it a problem? -->
<!-- ? -->
<!-- A domain layer that contains only data (getters/setters) with no behavior — all logic lives in the application layer or controllers. The layer exists in name only and provides none of the benefit of a true domain model. It's a common anti-pattern in naive layered apps and a sign the layering is artificial. -->
<!-- #flashcards/system-design -->

<!-- What's a "pass-through layer," and what does it signal? -->
<!-- ? -->
<!-- A middle layer (often the business/application layer) that merely forwards calls to the layer below without adding logic. It signals the layer is artificial — either remove it or give it a real responsibility (orchestration, validation, transaction management). -->
<!-- #flashcards/system-design -->

## Composition

<!-- How does layered architecture compose with other styles like microservices or monoliths? -->
<!-- ? -->
<!-- Layered is usually an **internal structure**, not a deployment decision. A monolith is typically layered inside; each microservice is typically layered inside. Layered answers "how do I organize code?" while the outer style (monolith, microservices, event-driven) answers "how do I deploy and scale?" They compose cleanly — almost every well-built service has layered internals. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
