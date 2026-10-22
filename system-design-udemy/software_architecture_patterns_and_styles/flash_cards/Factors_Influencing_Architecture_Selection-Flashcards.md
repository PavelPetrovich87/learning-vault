---
tags: [flashcards, system-design, software-architecture, decision-making, trade-offs]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Factors Influencing Architecture Selection — Interview Flashcards

## The Four Factor Families

<!-- What are the four families of factors that influence architecture selection? -->
<!-- ? -->
<!-- **Business** (quality attributes / NFRs, time to market, budget, domain complexity), **Technical** (scale/traffic, performance/latency, data/consistency, tech stack/legacy), **Organizational** (team size/structure, skills, Conway's Law, autonomy needs), and **Operational** (DevOps maturity, CI/CD, observability, compliance). No single factor decides — they must be weighed together, with the most important weighted heaviest for your context. -->
<!-- #flashcards/system-design -->

<!-- Why are non-functional requirements (quality attributes) usually the strongest driver of architecture? -->
<!-- ? -->
<!-- Functional requirements tell you *what* the system does; quality attributes (scalability, availability, latency, maintainability) tell you *how well* it does it under constraints. Architecture is primarily the answer to quality-attribute demands. Two systems with identical features but different scale/latency/availability needs may warrant completely different architectures. -->
<!-- #flashcards/system-design -->

## Organizational Factors

<!-- How does team size influence architecture selection? -->
<!-- ? -->
<!-- Small teams (1–10 engineers) should use a monolith — microservices overhead swamps the benefits. Large orgs (10+ teams) benefit from microservices so teams can own services and work in parallel. Key insight: microservices exist to solve the **team scaling** problem, not the traffic problem — a monolith can serve enormous traffic. -->
<!-- #flashcards/system-design -->

<!-- What is Conway's Law, and how can you use it deliberately? -->
<!-- ? -->
<!-- "Organizations produce systems that mirror their communication structure." If teams are organized by business domain, microservices aligned to those domains feel natural. If organized by technical layer, a layered architecture emerges regardless of intent. Use it via **reverse Conway**: structure teams the way you want the architecture to look, then let the architecture follow. -->
<!-- #flashcards/system-design -->

## The DevOps Gate

<!-- Why is DevOps maturity a "gate" for microservices and EDA? -->
<!-- ? -->
<!-- Microservices and EDA come with a non-negotiable operational tax: per-service CI/CD, centralized logging, distributed tracing, metrics, containerization, orchestration, secrets management. If the org can't pay this tax, these styles become an operational nightmare. Build DevOps maturity first; adopt microservices second. It's a prerequisite, not an afterthought. -->
<!-- #flashcards/system-design -->

## Data and Scale Factors

<!-- How do data consistency requirements influence architecture selection? -->
<!-- ? -->
<!-- **Strong ACID across entities** → keep them in a monolith (avoid distributed transactions/sagas). **Eventual consistency acceptable** → microservices (DB per service) or EDA. **Audit trail / temporal queries** → EDA with event sourcing. If a domain needs ACID across entities (e.g., financial ledgers), don't invite saga complexity — keep it together. -->
<!-- #flashcards/system-design -->

<!-- How does the traffic/scale profile influence architecture selection? -->
<!-- ? -->
<!-- Low/predictable traffic → monolith. High, uniform traffic → monolith behind a load balancer + caching. High, uneven traffic (hot spots) → microservices (scale only the hot component). Bursty/spiky traffic → EDA or serverless (brokers buffer spikes). The key question isn't "how much traffic?" but "do different parts need to scale differently?" -->
<!-- #flashcards/system-design -->

## Decision Process

<!-- What's the recommended process for making an architecture decision? -->
<!-- ? -->
<!-- 1. Gather requirements (functional + quality attributes). 2. Identify constraints (budget, team, deadline, legacy). 3. List candidate styles (don't fixate on one — consider 2–3). 4. Score each against your top 2–3 quality attributes. 5. Document the trade-offs in an Architecture Decision Record (ADR). 6. Build or evolve. 7. Revisit when context changes. -->
<!-- #flashcards/system-design -->

<!-- What is an Architecture Decision Record (ADR), and why use one? -->
<!-- ? -->
<!-- A short document capturing a single architecture decision: the context, options considered, the decision, and its consequences. ADRs force explicit trade-off thinking and leave a trail so future teams understand *why* a choice was made. "Why did we choose X?" should always have a written answer. -->
<!-- #flashcards/system-design -->

## Anti-Patterns and Mindset

<!-- What is cargo-cult engineering, and how do you avoid it in architecture selection? -->
<!-- ? -->
<!-- Copying the architecture of a company (Netflix, Google) at a completely different scale, team size, or domain — "they did it, so should we." Avoid it by analyzing *your own* factors (team, scale, domain, maturity) and choosing what fits your context. "Netflix uses microservices" is not a reason; it's cargo cult. -->
<!-- #flashcards/system-design -->

<!-- Why is architecture selection continuous rather than a one-time decision? -->
<!-- ? -->
<!-- The factors change over time — teams grow, traffic spikes, domains clarify, budgets shift. A system that starts as a monolith may extract microservices; a microservices system may consolidate services that were split too eagerly. Good architects treat the architecture as evolving, not frozen at the initial decision. Revisit when context changes. -->
<!-- #flashcards/system-design -->

<!-- Why do real production systems rarely use a single architectural style? -->
<!-- ? -->
<!-- Because different parts of a system have different needs. A common production architecture: client-server front door (API gateway), microservices for the core, each service layered internally, communicating via events for cross-service writes. Styles compose — you match each layer of the decision (deployment, internal structure, communication) to the relevant factors rather than forcing a single label. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
