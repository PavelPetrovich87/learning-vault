---
tags: [flashcards, system-design, software-architecture, patterns]
created: 2026-07-11
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Software Architecture — Interview Flashcards

## Core Concepts

<!-- What's the difference between software architecture and software design, and why does the distinction matter in an interview? -->
<!-- ? -->
<!-- **Architecture** is the high-level structure: how components are organized, how they interact, and the decisions that are hard to reverse (deployment topology, data ownership, communication style). **Design** is the internals of each component — classes, functions, algorithms. The distinction matters because architecture is about the expensive-to-change decisions you most want to get right early; design is the daily refactoring you do inside those boundaries. -->
<!-- #flashcards/system-design -->

<!-- An interviewer asks "why does architecture matter?" — what's your structured answer? -->
<!-- ? -->
<!-- Architecture determines the system's quality attributes: scalability, maintainability, testability, resilience, deployability. Bad architecture makes every change expensive, every outage catastrophic, and every team step on each other. Good architecture isolates change, enables independent scaling, and lets teams move in parallel. I'd frame it as: architecture is the set of decisions whose cost compounds over time if you get them wrong. -->
<!-- #flashcards/system-design -->

## Architectural Styles — Trade-offs

<!-- When would you choose a monolith over microservices in a system design interview? What would you say to justify it? -->
<!-- ? -->
<!-- Start with the context: small team (1–10 engineers), early-stage product / MVP, unknown or evolving domain, modest traffic, or strong ACID consistency needs. Then the justification: a monolith is simplest to build, test, deploy, and operate — no distributed-systems complexity, fast in-process calls, trivial ACID transactions. The principle is "monolith first": you can't draw service boundaries you haven't discovered yet, so refactor cheaply while monolithic and extract services only when a module demands independence. -->
<!-- #flashcards/system-design -->

<!-- What's the core trade-off of microservices, and how do you frame it to an interviewer? -->
<!-- ? -->
<!-- You trade **operational simplicity for independent scaling, deployment, and team autonomy**. You gain the ability to scale and ship each service on its own schedule, with fault isolation between them. You pay with distributed-systems complexity (network failures, eventual consistency, distributed transactions/sagas), per-service CI/CD and observability overhead, and harder testing/debugging. The honest framing: microservices solve the *team scaling* problem, not the traffic problem. -->
<!-- #flashcards/system-design -->

<!-- What's the core trade-off of event-driven architecture, and when would you recommend it? -->
<!-- ? -->
<!-- You trade **immediate consistency and easy debugging for loose coupling, async responsiveness, and extensibility**. Producers emit and move on; consumers react whenever ready — you can add consumers without touching producers. Recommend it when many independent reactions must fire from one change (e.g., OrderPlaced → inventory, payment, email, analytics), for streaming/real-time workloads, or to buffer spiky traffic. Avoid it when you need strong immediate consistency or sub-millisecond latency. -->
<!-- #flashcards/system-design -->

## Anti-Patterns and Gotchas

<!-- What's a "distributed monolith," and why is it a red flag an interviewer might probe? -->
<!-- ? -->
<!-- A system that *looks* like microservices — many services, network calls — but behaves like a monolith: services share a database, must deploy together, or sit in long synchronous chains. It incurs all the costs of microservices (network, ops complexity) with none of the benefits (independent scaling/deployment). It usually comes from splitting before boundaries were understood or sharing a database. I'd flag it as the #1 microservices anti-pattern. -->
<!-- #flashcards/system-design -->

## Big-Picture Framing

<!-- How would you answer "are real systems usually one architectural style?" -->
<!-- ? -->
<!-- No — real systems blend styles. A common production architecture: a client-server front door (API gateway), microservices for the core, each service layered internally, communicating via events for cross-service writes. The principle is composition: match each layer of the decision (deployment, internal code structure, communication) to the relevant factors rather than forcing a single label. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-11* -->
