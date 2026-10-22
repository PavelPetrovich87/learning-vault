---
status: archived
exercise_type: interview_simulation
difficulty: advanced
pattern: selection
source_concepts:
  - "[[Factors Influencing Architecture Selection]]"
  - "[[Software Architecture]]"
  - "[[Monolithic Architecture]]"
  - "[[Microservices Architecture]]"
  - "[[Event-Driven Architecture]]"
  - "[[Layered Architecture]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: Evolving a 15-Year-Old Global Monolith (Without Taking It Down)

## Type
**Interview simulation** — a staff/principal-level system-design interview conducted by a **panel**: the CTO (strategic vision, speed, board-facing) and a staff engineer (probes the technical migration plan, risk, phasing). The hardest exercise in the set — the candidate must synthesize monolith, microservices, *and* event-driven patterns into a multi-year evolution, with phasing, risk management, and explicit trade-offs, while keeping a live global system up throughout.

## Difficulty
**Advanced.** There is no single-architecture answer. This is a multi-year *evolution* combining all three patterns, driven by genuinely conflicting stakeholders (CTO wants speed, finance wants low cost, security wants lock-down, the legacy team defends the monolith) and a real team-scale problem (200 engineers, 15 teams) that actually justifies microservices. The candidate must avoid both traps: the "rewrite everything" reflex and the "don't touch the monolith" paralysis.

## The Situation

**Company:** Orion Global — a large enterprise SaaS platform. The core is **"the Citadel," a 15-year-old monolith** (Java + a sprawling Oracle database) serving **~5M users globally**. The Citadel works — it's profitable, stable-ish — but it's straining under scale and organizational pressure.

**200 engineers across 15 teams.** The Citadel is the single deployable: every team's work ships in one release train, and a deploy is a nervous, coordinated event.

**The pains driving the mandate:**
- **Global latency.** The Citadel runs in a single region (us-east). Users in APAC and EMEA experience 200–400ms+ latency; it's losing deals in those markets.
- **No real-time features.** Customers (and competitors) now expect live notifications, presence, collaborative editing, real-time dashboards. The Citadel is fundamentally request/response.
- **Teams blocked on the deploy cycle.** 15 teams queuing behind one release train. A feature a team finishes today ships in two weeks. Velocity is collapsing; teams are frustrated.
- **Blast radius.** Last quarter, one team's bad deploy (a memory leak in a reporting module) took the *entire* Citadel down for 47 minutes — globally. The post-mortem's headline finding: **no fault isolation.**
- **Regulatory pressure.** Expanding into the EU and parts of APAC means data-residency requirements the single-region Citadel can't satisfy.

**The mandate from the CEO, relayed by the CTO:** evolve the Citadel to support global scale, real-time features, and a growing multi-team engineering org — **while staying up throughout.** No big-bang rewrite. Multi-year horizon. Budget exists but Finance will scrutinize every dollar.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **CEO / board** | Global growth unlocked, real-time parity with competitors, no more global outages | Wants results in quarters, not a decade |
| **CTO (panel — strategic)** | A compelling, phased vision; speed of execution; a story for the board | Wants the strategic narrative and demonstrable progress fast |
| **Staff engineer (panel — technical)** | A concrete, risk-managed migration plan that won't break the Citadel mid-flight | Probes phasing, data decoupling, rollback, dual-run — kills hand-waves |
| **Finance** | Low cost; no open-ended "modernization" black hole | Skeptical of a multi-year, multi-region, platform-team build-out |
| **Security & compliance** | Data residency, least-privilege, reduced blast radius | Wants lock-down; wary of more distributed attack surface and multi-region data spread |
| **Legacy Citadel team** | Don't break what works; defend the monolith's reliability | Rightly proud of 15 stable years; fear a migration creates new instability |
| **The 15 product teams** | Ship independently, on their own cadence, without the release train | Desperate for autonomy; the #1 morale problem |
| **On-call / SRE** | No 3 AM migration fires; canary, rollback, dual-run, observability | Will resist any step without a safety net |
| **APAC / EMEA users** | Low latency, data residency | Single-region architecture structurally cannot deliver this today |

## Constraints

- **No big-bang rewrite.** The Citadel stays up throughout. Incremental evolution only.
- **Genuine team scale.** 200 engineers, 15 teams. This *actually* meets the bar where microservices are justified — the organizational-coordination cost of the monolith is now the dominant drag on velocity. (Unlike a 4-person team, this is not cargo-culting.)
- **Global latency & data residency.** Single-region request/response can't be patched into global low-latency. Requires multi-region deployment: edge/CDN for static + regional services for dynamic, with data-residency-aware routing.
- **Real-time features.** Request/response Citadel can't do live notifications/presence/collab. Requires an event-driven layer (websockets/SSE + an event backbone).
- **Transactional core.** Billing, identity, and the core order/account state are ACID and deeply coupled in Oracle. Distributing *this* across services is where migrations die. Keep the transactional core together; extract around it.
- **Oracle + legacy data model.** The Citadel's data is a 15-year-old schema with implicit relationships. Data decoupling is the central technical challenge — not the easy part.
- **Multi-year horizon, but quarterly checkpoints.** Finance and the board need visible progress: latency reduced, a team shipping independently, an outage blast-radius contained, a region live.
- **DevOps maturity:** mixed. CI/CD exists for the Citadel; observability is mediocre; feature flags partial. Part of the migration is *paying the platform tax* (CI/CD per service, tracing, flags) as a prerequisite, not an afterthought.

## The Ask

The panel expects a multi-year evolution strategy addressing:

1. **The overall arc.** How do monolith, microservices, and event-driven patterns combine here — and *where* does each earn its place? (No single-pattern answer survives this scenario.)
2. **Phasing.** What's the sequence over years? What's the goal of each phase? What's the "definition of done" for each?
3. **The migration mechanism.** How do you move work out of the Citadel incrementally without a flag-day cutover? (Name the pattern.)
4. **Global scale.** How do you get APAC/EMEA latency down and satisfy data residency?
5. **Real-time.** How do you add live features without rewriting the Citadel?
6. **Risk management.** Feature flags, canary, dual-run, rollback — what's the safety net at each step?
7. **The organizational dimension.** Conway's Law cuts both ways — how do you restructure teams to match the target architecture (and avoid the org fighting the architecture)?
8. **The cost/capability story for Finance.** What does this cost, what does it buy, and how do you avoid the open-ended black hole?
9. **What would you *not* do, and why?**

## Hidden Traps (the persona will probe for these — don't reveal them to yourself before answering)

> [!danger] These are for the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Single-pattern answer.** "Go microservices" or "go event-driven" misses the point. This is a *combination* over time: the monolith modularizes and shrinks, microservices get extracted where teams/scale justify it, EDA layers in for real-time + decoupling. The candidate must show *where each pattern applies*.
> 2. **The "rewrite everything" reflex.** "Let's build Orion 2.0 on the side and switch over" is a reject signal at staff level. Big-bang rewrites of a profitable 15-year system fail more often than they succeed. Evolution via the **Strangler Fig** is the answer.
> 3. **Ignoring the genuine team-scale signal.** Unlike a 4-person startup, 200 engineers / 15 teams genuinely meets the bar for microservices — the monolith's coordination cost is now the dominant velocity drag. A candidate who says "just keep the monolith, add modules" has under-weighted the organizational factor. Microservices here are justified, not cargo-culted.
> 4. **Distributing the ACID core.** Billing, identity, core order/account state are deeply coupled in Oracle. A candidate who proposes splitting these into separate services with eventual consistency is walking into a distributed-transaction nightmare. **Keep the transactional core together**; extract *around* it.
> 5. **Modularize before extracting.** The right first move is almost always: enforce module boundaries *inside* the Citadel first (clear ownership, data ownership per module, boundary tests). You can't cleanly extract a module that has no boundary. Skipping straight to extraction guarantees a torn coupling.
> 6. **Global latency needs multi-region, not microservices.** A candidate who thinks "microservices will fix latency" has confused two problems. Latency is a *geo* problem (multi-region + edge/CDN + regional services + data-residency routing), independent of service granularity. Microservices and multi-region are separate axes.
> 7. **Real-time needs EDA, not a rewrite.** Live notifications/presence/collab require an event-driven layer (event backbone + websocket/SSE fan-out). This can be added *alongside* the Citadel (events published from the core, consumed by a new real-time layer) without rewriting the request/response core.
> 8. **No phasing or milestones.** "We'll migrate over a few years" with no phases, definition-of-done, or quarterly checkpoints is unacceptable to Finance and the board. A strong answer sequences: (1) modularize monolith + pay platform tax (CI/CD, tracing, flags); (2) extract hot/team-owned services via Strangler Fig; (3) add EDA for real-time + decoupling; (4) multi-region rollout with data-residency routing.
> 9. **No risk management.** No feature flags, no canary, no dual-run, no rollback = no plan. The panel will kill any step without a safety net, especially around the ACID core and regional rollouts.
> 10. **Conway's Law ignored.** Architecture and org structure must co-evolve. Restructure teams to match target service boundaries; otherwise the org will re-create the monolith's coupling inside the "services" (a distributed monolith). The migration is *organizational as much as technical*.
> 11. **Cost black hole.** The candidate must frame cost vs. capability for Finance: what each phase costs, what it buys (latency → APAC revenue; team autonomy → velocity; fault isolation → no more 47-minute global outages). "Modernize everything" with no ROI framing will not get funded.
> 12. **Forgetting to pay the platform tax first.** You can't run microservices + multi-region + EDA without CI/CD per service, distributed tracing, feature flags, and observability. Part of *phase 1* is building the platform, not just refactoring code.

## Reference (what a strong answer covers)

- **Overall arc — three patterns, each in its place, evolved over years:**
  - **Monolith (Citadel):** modularize internally, then *shrink* it as services are extracted. The transactional core (billing, identity, core account/order) stays — possibly forever — as a well-modularized core service. Don't distribute the ACID heart.
  - **Microservices:** extract where a *team owns* a capability AND it needs independent scaling/release/deployment. 15 teams genuinely justifies this — it's not cargo-culting. Service seams aligned to team boundaries (Conway's Law).
  - **Event-driven:** add an event backbone for (a) real-time features (notifications/presence/collab via websocket/SSE fan-out), (b) decoupling extracted services from the core (event-carried state), and (c) feeding analytics/read models.

- **Phasing (multi-year, with definition-of-done per phase):**
  1. **Phase 1 — Stabilize & platform (months 0–9):** Enforce module boundaries inside the Citadel (ownership, data ownership, boundary tests). Build the platform: CI/CD per service, distributed tracing, feature flags, observability. *DoD: a module can be safely extracted; the team can deploy a service independently with safety nets.*
  2. **Phase 2 — Strangler-Fig extraction (months 6–24, overlapping):** Extract hot, team-owned, independently-scalable capabilities first (e.g., the reporting module that caused the outage; a team-owned domain). Route traffic incrementally via a facade; dual-run; canary; rollback flags. *DoD: that team ships on its own cadence; a fault in their service no longer takes down the Citadel.*
  3. **Phase 3 — Event-driven layer for real-time (months 12–30):** Stand up an event backbone. Publish domain events from the core. Build a real-time service (websocket/SSE) consuming them for notifications/presence/collab. Use event-carried state to decouple newly-extracted services from the Oracle core. *DoD: real-time features live; extracted services no longer synchronously read the Citadel's DB.*
  4. **Phase 4 — Multi-region & data residency (months 18–40):** Edge/CDN for static assets; regional service deployments for dynamic traffic; data-residency-aware routing (EU user data stays in EU). Replicate/edge-cache read models regionally; keep the transactional core's writes in a designated primary with regional read replicas. *DoD: APAC/EMEA latency under target; EU data-residency compliant.*

- **Migration mechanism: Strangler Fig.** A routing facade (API gateway) in front of the Citadel. New capabilities built as services; existing capabilities extracted incrementally. Traffic shifted behind feature flags, dual-run for correctness, canary for risk, instant rollback. The Citadel shrinks as services grow — never a flag day.

- **Global scale:** edge/CDN for static + regional service deployments + data-residency routing. This is a *geo* axis, orthogonal to service granularity. Regional read replicas / edge-cached read models for latency; writes to a designated primary or region-aware write shards where consistency permits.

- **Real-time:** an event backbone (the Citadel and extracted services publish events) + a real-time service fanning out via websocket/SSE. The request/response core isn't rewritten — it learns to publish events.

- **Risk management at every step:** feature flags on every cutover; canary deployments; dual-run with result diffing for correctness-critical extractions (especially around the core); instant rollback; SRE on-call during ramps; explicit rollback gates per phase.

- **Organizational co-evolution (Conway's Law):** restructure teams to own target services end-to-end; align service seams to team boundaries; fund a platform team to pay the DevOps tax so product teams don't each build their own infra. The migration is as much org-design as code.

- **Cost/capability for Finance:** frame each phase as ROI — Phase 1 (platform) → unblocks everything and stops the next global outage; Phase 2 (extraction) → team velocity recovers, blast radius contained; Phase 3 (real-time) → competitive parity, retention; Phase 4 (multi-region) → APAC/EMEA revenue unlock, regulatory compliance. Bounded phases with checkpoints prevent the open-ended black hole.

- **What NOT to do:** no big-bang rewrite; no splitting the ACID core; no extracting before modularizing; no microservices without first paying the platform tax; no multi-region rollout without data-residency routing; no service seams that cut across team boundaries.

## Related Concept Notes
- [[Factors Influencing Architecture Selection]] — the four factor families, and the org-scale threshold that justifies microservices
- [[Monolithic Architecture]] — modularizing the legacy core, when extraction is justified, "monolith first" in reverse
- [[Microservices Architecture]] — team-aligned service seams, the microservices tax and paying it via a platform team
- [[Event-Driven Architecture]] — event backbone for real-time fan-out, event-carried state for decoupling, read models
- [[Layered Architecture]] — internal modularization of the Citadel before extraction
- [[Software Architecture]] — Strangler Fig, Conway's Law, distributed-monolith anti-pattern, big-bang-rewrite anti-pattern

---

*Last updated: 2026-07-15*
