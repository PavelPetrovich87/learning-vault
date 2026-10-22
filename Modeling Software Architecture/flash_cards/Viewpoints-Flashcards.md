<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/architecture-modeling -->

# 🎴 C4 Model & Architecture Viewpoints — Flashcards

<!-- Source: [[Modeling Software Architecture]] viewpoint notes. -->

## Foundations

<!-- What is the difference between a model and a copy of the real system? -->
<!-- ? -->
<!-- A model is intentionally **simplified** — it removes irrelevant details to focus on the questions that matter right now. A copy would be as complex as reality and useless for reasoning. See [[Model]]. -->
<!-- #flashcards/architecture-modeling/foundations -->

<!-- What is a mental model, and why does it matter in architecture? -->
<!-- ? -->
<!-- A simplified internal representation of how something works, used to understand, predict, and reason. Architecture is abstract; mental models let architects reason and communicate without holding every detail in memory. See [[Mental Model]]. -->
<!-- #flashcards/architecture-modeling/foundations -->

<!-- Why is the system boundary important in architecture modeling? -->
<!-- ? -->
<!-- It separates what we control and are responsible for from the external world. It defines scope, ownership, and the interfaces that must be managed. See [[System]]. -->
<!-- #flashcards/architecture-modeling/foundations -->

<!-- What are the three levels of abstraction (Enterprise / Solution / System), and what question does each answer? -->
<!-- ? -->
<!-- **Enterprise** — "What systems exist and why?" (whole organization). **Solution** — "How do a few systems cooperate for one initiative?" **System** — "How is this single system built internally?" See [[Levels of Abstraction]]. -->
<!-- #flashcards/architecture-modeling/foundations -->

<!-- How do levels of abstraction (Enterprise/Solution/System) differ from C4 levels (Context/Container/Component/Code)? -->
<!-- ? -->
<!-- They are orthogonal. Enterprise/Solution/System zoom **across systems**; C4 levels zoom **inside one system**. They combine freely — e.g., a Container diagram is always at the System level of abstraction. See [[Levels of Abstraction]]. -->
<!-- #flashcards/architecture-modeling/foundations -->

<!-- What are the three key elements of an architecture view? -->
<!-- ? -->
<!-- **Components** (the things that do work), **Connectors** (the interaction mechanisms / rules of conversation), and **Configuration** (the overall arrangement — topology). See [[Key Elements of Architecture View]]. -->
<!-- #flashcards/architecture-modeling/foundations -->

<!-- Why should a connector be treated as a first-class element, not just a line? -->
<!-- ? -->
<!-- The connector carries the hardest architectural decisions — sync vs. async, protocol, reliability, data ownership, coupling. Two components with identical responsibilities but different connectors behave very differently. See [[Key Elements of Architecture View]]. -->
<!-- #flashcards/architecture-modeling/foundations -->

## Viewpoint vs. Perspective

<!-- What is the difference between a view and a viewpoint? -->
<!-- ? -->
<!-- A **viewpoint** is the reusable template/lens (defines concerns, notation, audience). A **view** is the concrete artifact you produce for a specific system by looking through that viewpoint. Viewpoint = recipe; view = cake. See [[Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What is a perspective, and how does it differ from a viewpoint? -->
<!-- ? -->
<!-- A **perspective** is a cross-cutting quality concern (security, performance) applied *to* a viewpoint. A viewpoint produces a view (structure); a perspective produces **no view of its own** — it evaluates and refines existing views. See [[Perspective]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Why is "show me the security view" a category error? -->
<!-- ? -->
<!-- Security is a *perspective*, not a *viewpoint* — it has no diagram of its own. You apply the Security perspective *to* a specific view (e.g., Deployment). There is no standalone "security diagram." See [[Perspective]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

## The C4 Zoom Levels

<!-- What does the Context Viewpoint show, and what does it deliberately hide? -->
<!-- ? -->
<!-- It shows the **system as a single box** plus actors and external systems. It hides all internals — databases, brokers, services, code. The single box *is* the boundary definition. See [[Context Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Why is "adding the database to the context diagram" a mistake? -->
<!-- ? -->
<!-- The database is **internal** — it belongs at the Container level (C4 L2). The value of the context viewpoint is the discipline of showing nothing inside; once you draw internals, it stops being a context diagram. See [[Context Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What does the Container Viewpoint show, and is a C4 "container" the same as a Docker container? -->
<!-- ? -->
<!-- It opens the Context box and shows the **major deployable/runnable units** (web app, API, database, queue). No — a C4 container is a logical concept (a single deployable unit); Docker is one deployment mechanism. See [[Container Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What's the test for whether something is a container? -->
<!-- ? -->
<!-- *Would this thing be deployed/started/stopped/replaced independently?* If yes, it's a container. A shared library is not (it doesn't run independently). See [[Container Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What does the Component Viewpoint show, and when do you draw it? -->
<!-- ? -->
<!-- It zooms into a **single container** and shows the significant code building blocks inside it (services, controllers, repositories). You draw it per-container, only for containers complex enough to warrant it (usually backends). See [[Component Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Can a C4 component span multiple containers? -->
<!-- ? -->
<!-- No — by definition a C4 component lives *inside* a single container. If something spans containers, you've mis-scoped a container boundary. See [[Component Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

## The Rozanski & Woods Viewpoints

<!-- Does C4 have a "Functional" level? Where does functionality live? -->
<!-- ? -->
<!-- No — C4 levels are structural zoom. The Functional Viewpoint is a **Rozanski & Woods** viewpoint. In C4 practice it's expressed via responsibility text on the Context box and, mainly, the supplementary **Dynamic diagram** tracing a use case through components. See [[Functional Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What's the difference between a capability and a use case? -->
<!-- ? -->
<!-- A **capability** is a high-level "what the system can do" ("Process payments") — stable, business-facing. A **use case** is a concrete scenario ("Customer checks out with a card") that *realizes* a capability. See [[Functional Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What are the three levels of data modeling, and which does the Information Viewpoint own? -->
<!-- ? -->
<!-- **Conceptual** (entities + relationships), **Logical** (attributes, keys, cardinalities), **Physical** (concrete schema). The Information Viewpoint owns **Conceptual and Logical**; Physical is an implementation/storage concern. See [[Information Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Why is it a mistake to jump straight to a physical database schema? -->
<!-- ? -->
<!-- It couples *what data means* to *how it's stored*. Meaning is stable for decades; storage technology changes. Modeling concepts first lets meaning survive storage changes. See [[Information Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Why must sensitive/regulated data be modeled explicitly in the Information Viewpoint? -->
<!-- ? -->
<!-- Because PII/PCI/PHI carry legal obligations (encryption, residency, retention) that propagate into every other viewpoint and perspective. If sensitive data is just "another attribute," the architecture gets built around wrong assumptions. See [[Information Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What does the Concurrency Viewpoint model, and how is it different from the Container Viewpoint? -->
<!-- ? -->
<!-- It models **runtime units** (processes, threads, async tasks) and shared state. The Container Viewpoint shows *static deployable units*; the Concurrency Viewpoint shows *runtime instances* — one container may map to many concurrent units. See [[Concurrency Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Why is "shared state" the central hazard of concurrency? -->
<!-- ? -->
<!-- Concurrency is only a problem when parallel units touch the same mutable state — that's where races and corruption arise. No shared state → no race. The viewpoint must make every piece of shared state explicit. See [[Concurrency Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What does the Development Viewpoint show, and how does it differ from the Container Viewpoint? -->
<!-- ? -->
<!-- It shows **source-code structure** — modules, packages, layers, dependency rules, build. The Container Viewpoint is logical/runtime; the Development Viewpoint is code/build. One container ← many modules. See [[Development Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Why must architectural dependency rules be enforced by tooling/CI? -->
<!-- ? -->
<!-- Because unenforced rules are aspirations, not constraints — violations accumulate until the structure rots. Lint rules and CI gates turn "we have a layering rule" into an actual constraint. See [[Development Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What does the Deployment Viewpoint show, and how does it differ from the Container Viewpoint? -->
<!-- ? -->
<!-- It shows **where** the system runs — environments, nodes, networks, and the container-to-node mapping. Container = logical (*what*); Deployment = physical (*where*). See [[Deployment Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Why must network constraints (firewalls, latency, trust boundaries) be shown explicitly on a deployment diagram? -->
<!-- ? -->
<!-- Those constraints are the deployment view's entire point — they're invisible in any logical view and they cause production incidents. A diagram that shows everything connected hides exactly the constraints you need to see. See [[Deployment Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What does the Operational Viewpoint model, and why is it called "the integrator"? -->
<!-- ? -->
<!-- It models **runtime behavior**: user journeys and automated sequences. It's "the integrator" because a journey threads through *all* other viewpoints (realizes a use case, crosses the boundary, moves data, runs on nodes). See [[Operational Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- Why is "operational" a misleading name for this viewpoint? -->
<!-- ? -->
<!-- It sounds like "operations/SRE," but those concerns live in the Deployment Viewpoint and Reliability perspective. The Operational Viewpoint is actually about **runtime behavior** — journeys and sequences, not how SREs operate the system. See [[Operational Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->

<!-- What are the two kinds of sequences the Operational Viewpoint models, and why do automated ones need special attention? -->
<!-- ? -->
<!-- **User journeys** (actor-driven) and **automated sequences** (time/event-triggered, no human in the loop). Automated ones run unattended, so their failures are **silent** until noticed. See [[Operational Viewpoint]]. -->
<!-- #flashcards/architecture-modeling/viewpoints -->
