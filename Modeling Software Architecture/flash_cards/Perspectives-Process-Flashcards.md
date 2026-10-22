<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/architecture-modeling -->

# 🎴 Architecture Perspectives & Modeling Process — Flashcards

<!-- Source: [[Modeling Software Architecture]] perspective and process notes. -->

## The Seven Perspectives

<!-- How many perspectives should you apply, and how do you choose? -->
<!-- ? -->
<!-- Typically **3–5** of the seven core perspectives, selected by **risk**: identify the system's biggest risks (data privacy for healthcare, load for e-commerce) and pick the perspectives that address them. Applying all seven to all views is analysis paralysis. See [[Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- What does a perspective produce, if not a view? -->
<!-- ? -->
<!-- It produces an **assessment** and a set of **refinements** to existing views ("add a firewall here", "encrypt this data store"). The output is documented decisions and changes to the architecture, not a new diagram. See [[Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

### Security

<!-- What does the Security Perspective protect, and what's the underlying model? -->
<!-- ? -->
<!-- The **CIA triad** — Confidentiality (only authorized can read), Integrity (data can't be tampered with), Availability (system resists DoS). Every security control maps to at least one of these. See [[Security Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why must threat modeling precede control selection? -->
<!-- ? -->
<!-- Controls without a threat model are cargo-cult security — you don't know which threats you're addressing. Threat modeling identifies assets, actors, and attack surfaces *first*, producing a prioritized threat list that drives specific controls. See [[Security Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- What does "least privilege" mean, and why is over-permissioning dangerous? -->
<!-- ? -->
<!-- Every component/user gets the minimum access needed, and no more. Over-permissioned services are the most common lateral-movement vector — once compromised, an attacker escalates freely because the service already has the keys. See [[Security Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

### Performance and Scalability

<!-- What's the difference between performance and scalability? -->
<!-- ? -->
<!-- **Performance** is behavior under *current* load (latency, throughput per request). **Scalability** is behavior *as load grows* — does the system absorb more users by adding capacity, or collapse? A fast system that doesn't scale breaks at 10×. See [[Performance and Scalability Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why must you identify the bottleneck before optimizing? -->
<!-- ? -->
<!-- Because of the Theory of Constraints — scaling the non-bottleneck does nothing. If the DB is the bottleneck, optimizing the API's CPU won't help. Profile first, then address *that* constraint. See [[Performance and Scalability Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- What's the difference between scaling up and scaling out, and which enables real growth? -->
<!-- ? -->
<!-- **Scale up** (vertical) = bigger machine. **Scale out** (horizontal) = more machines. Real growth usually requires scale-out, but it demands the architecture be **stateless and partitionable**. See [[Performance and Scalability Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why does "where you place state" determine how far you can scale? -->
<!-- ? -->
<!-- Statelessness enables horizontal scale — a stateless service can be replicated freely. A stateful service (sticky sessions, in-memory cache) is pinned to a node. Moving state to a shared store or eliminating it is often the key enabler of scale-out. See [[Performance and Scalability Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

### Availability and Resilience

<!-- What's the difference between availability and resilience? -->
<!-- ? -->
<!-- **Availability** is measured uptime. **Resilience** is failure-handling ability — when a dependency fails, the system degrades gracefully and recovers instead of cascading to a full outage. High availability without resilience gives rare but catastrophic outages. See [[Availability and Resilience Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why must distributed systems be designed assuming failure is normal? -->
<!-- ? -->
<!-- Because distributed systems are *defined* by partial failure — something is always degraded or restarting. Designing as if things always work guarantees cascading collapses. Designing for partial failure (retries, circuit breakers, fallbacks) contains failures. See [[Availability and Resilience Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why are retries without backoff or a cap dangerous? -->
<!-- ? -->
<!-- A failing dependency + aggressive retries = amplified load that takes the dependency down permanently. Retries need exponential backoff and a cap, plus a circuit breaker. See [[Availability and Resilience Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- What are RTO and RPO, and how do they drive architectural decisions? -->
<!-- ? -->
<!-- **RTO** (Recovery Time Objective) = how long until service is restored. **RPO** (Recovery Point Objective) = how much data you can afford to lose. These business-set targets drive the redundancy/recovery investment. See [[Availability and Resilience Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

### Evolution

<!-- What does the Evolution Perspective ensure, and why is "we'll add flexibility later" a myth? -->
<!-- ? -->
<!-- It ensures the system can change without breaking or requiring a rewrite — via low coupling, stable interfaces, clear extension points. "Flexibility later" is a myth because retrofitting it into a tangled system *is* a rewrite. See [[Evolution Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why is coupling the enemy of evolution? -->
<!-- ? -->
<!-- Coupling makes change ripple — the more one component knows about another's internals, the more a change breaks others. Loose coupling (stable interfaces, async contracts, bounded contexts) localizes change. See [[Evolution Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->
<!--SR:!2026-07-12,1,230-->

<!-- How does YAGNI relate to designing for change? -->
<!-- ? -->
<!-- YAGNI says don't build unneeded *features* prematurely — it does *not* say ignore *changeability*. You don't pre-build features, but you *do* keep seams clean so features can add cheaply later. See [[Evolution Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why is a shared database a problem for evolution? -->
<!-- ? -->
<!-- It's the highest-coupling integration — every consumer depends on the same schema, so any schema change risks breaking all of them. It makes the data model nearly impossible to evolve safely. See [[Evolution Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

### Development Resource

<!-- Why is "a good architecture is team-agnostic" a myth? -->
<!-- ? -->
<!-- An architecture assuming skills the team doesn't have, or complexity exceeding their capacity, will fail to ship regardless of technical elegance. The team is an architectural constraint. See [[Development Resource Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why is matching complexity to team capacity critical? -->
<!-- ? -->
<!-- A small team operating a hyper-complex architecture (3 people running 30 microservices) can't build, deploy, or maintain it — operational overhead exceeds capacity. Complexity must match what the team can operate. See [[Development Resource Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- What's "resume-driven design," and why is it a pitfall? -->
<!-- ? -->
<!-- Choosing technologies because they're impressive/trendy rather than because the team can use them effectively. It stalls projects — the team spends months learning instead of delivering. See [[Development Resource Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

### Cost

<!-- Why does run cost usually dominate Total Cost of Ownership? -->
<!-- ? -->
<!-- Build cost is paid once; run cost (infrastructure, APIs, operations) is paid every month for the system's lifetime. Over years, the recurring run cost typically exceeds the one-time build cost. See [[Cost Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why is network egress a common surprise cost? -->
<!-- ? -->
<!-- Data leaving your cloud (cross-region, internet egress) is priced per-GB and adds up quickly in chatty multi-region architectures. Teams budget for compute/storage but overlook egress. See [[Cost Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- How does the Cost Perspective interact with the other perspectives? -->
<!-- ? -->
<!-- It's the perspective most *in tension* — almost every quality investment (redundancy, capacity, encryption) raises cost. The architect balances cost against other qualities, matching spending to business value. See [[Cost Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

### Location

<!-- What three problems does the Location Perspective address? -->
<!-- ? -->
<!-- **Latency** (physical distance = time), **legal/regulatory** (data residency, cross-border transfer rules), and **presence** (where do users need the system to be fast/available?). All invisible in logical views. See [[Location Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why is "geography is just an ops detail" a dangerous assumption? -->
<!-- ? -->
<!-- Geography drives latency, legality (EU data in US violates GDPR), and resilience (single-region = one outage from failure). These shape the whole system, not deployment afterthoughts. See [[Location Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

<!-- Why might replication *worsen* a residency problem? -->
<!-- ? -->
<!-- Replication places data in *more* jurisdictions — and if that data is regulated, some placements may be illegal. You can't freely copy EU PII to a US region just because replication is technically easy. See [[Location Perspective]]. -->
<!-- #flashcards/architecture-modeling/perspectives -->

## The Modeling Process

<!-- What are the two cases for creating an architecture model, and how do they differ? -->
<!-- ? -->
<!-- **From scratch (greenfield):** you *decide* the shape, sourcing from requirements. **For an existing system (brownfield):** you *discover* the existing shape, sourcing from code, the running system, and people. Both produce the same artifact; they differ in where answers come from and difficulty. See [[Architecture Model]]. -->
<!-- #flashcards/architecture-modeling/process -->

<!-- What is the recommended sequence of viewpoints when modeling from scratch? -->
<!-- ? -->
<!-- Functional → Context → Information → Operational → Development → Concurrency → Deployment. Each viewpoint depends on the previous; skip the order and the views contradict each other. See [[Architecture Model]]. -->
<!-- #flashcards/architecture-modeling/process -->

<!-- Why is the modeling process a spiral rather than a straight line? -->
<!-- ? -->
<!-- Later viewpoints reveal gaps in earlier ones (Deployment may surface latency problems requiring revisiting Concurrency). After the first pass, applying perspectives exposes further gaps. Each loop refines the model. See [[Architecture Model]]. -->
<!-- #flashcards/architecture-modeling/process -->

<!-- What's the hardest part of modeling an existing system, and why? -->
<!-- ? -->
<!-- Recovering the **"why"** behind decisions. Code shows *what* was built but is silent about constraints and intent. Only the people who built it hold that rationale — capturing it before they leave is critical. See [[Architecture Model]]. -->
<!-- #flashcards/architecture-modeling/process -->

## Handling Complexity

<!-- What are the four techniques for handling model complexity? -->
<!-- ? -->
<!-- (1) **Choose the right abstraction level** — zoom out until readable. (2) **Prioritize viewpoints** — draw only the ones that answer real questions. (3) **Use levels within viewpoints** — C4 Context → Container → Component drill-down. (4) **Split overloaded views** by phase/context (BAU, batch, failure-handling). See [[Handling Complexity of Architecture Model]]. -->
<!-- #flashcards/architecture-modeling/process -->

<!-- Why does "putting everything on one diagram" fail? -->
<!-- ? -->
<!-- A real system has too many components to show at once — the result is an unreadable wall of boxes. A diagram is not a database; its job is to answer one question for one audience. The alternative is many small focused views. See [[Handling Complexity of Architecture Model]]. -->
<!-- #flashcards/architecture-modeling/process -->

<!-- Give an example of splitting an overloaded view by phase. -->
<!-- ? -->
<!-- An Operational view split into: **BAU** (normal operation), **post-processing/batch** (nightly invoicing, reporting), **onboarding/setup** (initial config), **failure handling** (incident response, recovery). Each phase has different behavior; splitting keeps each view focused. See [[Handling Complexity of Architecture Model]]. -->
<!-- #flashcards/architecture-modeling/process -->
