---
status: archived
exercise_type: planning_meeting
difficulty: intermediate
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

# Exercise: Architecting a HIPAA-Compliant Patient Records Platform

## Type
**Planning meeting** — you are the architect invited to brief the VP of Engineering. The persona is strategic, probes trade-offs and risk, and needs a recommendation they can present to the board. This is not an interview exam; it's a real decision with regulatory teeth.

## Difficulty
**Intermediate.** Tests the judgment that defeats single-pattern dogma: real systems rarely fit one box. The candidate must weigh monolith, microservices, *and* event-driven against a context that pulls in different directions (compliance favors EDA, team structure favors microservices, the transactional clinical core favors keeping data together) — and synthesize a **hybrid**. The failure modes are: over-decomposing with only 25 engineers, ignoring the audit/compliance lever, or treating the real-time notification as an afterthought.

## The Situation

**Company:** Meridian Health — a Series B HealthTech startup building a **patient records platform** for mid-sized clinics. **25 engineers across three teams: Clinical, Billing, Analytics.** HIPAA-regulated; PHI (protected health information) everywhere.

**Functional scope:**
- **Patient records** — demographics, medical history, problem list.
- **Appointments** — scheduling, reminders, check-in.
- **Prescriptions** — medication orders, drug-interaction checks, e-prescribing.
- **Lab results** — results arrive asynchronously from external labs; clinicians must be notified in near-real-time.
- **Billing & claims** — invoicing, insurance claim submission, payment posting.
- **Analytics** — clinical quality metrics, utilization dashboards, financial reporting.

**The non-negotiable: compliance & audit.** Under HIPAA, **every access to and every change of PHI must be recorded** in a tamper-evident audit trail. Who viewed what, when, from where, and what they changed. This is not a feature; it is a regulatory foundation. A failed audit means fines and lost contracts.

**The real-time requirement:** When a lab result arrives (async, from an external lab integration), the responsible clinician must be notified promptly — ideally within seconds — so time-sensitive results (critical values, abnormal imaging) aren't sitting in a queue. This is patient-safety-relevant.

**Team structure (already in place):** Clinical (12 eng, owns records/appointments/prescriptions/labs), Billing (7 eng, owns invoicing/claims/payments), Analytics (6 eng, owns reporting/quality metrics). Each team has a tech lead and ships independently.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **VP Engineering (the persona)** | An architecture that satisfies compliance, fits 3 teams, and is defensible to the board | Needs real trade-off analysis — will reject any "just do X" answer |
| **Compliance / privacy officer** | Bulletproof audit trail; PHI access logging that survives any audit; least-privilege data access | Favors immutable event log / event sourcing; suspicious of data spread across many services |
| **Clinical team** | A reliable, fast, ACID transactional core — a half-written prescription or a lost lab result is a patient-safety incident | Wants the tightly-coupled clinical data kept together; wary of distributed transactions |
| **Billing team** | Independent release cadence; billing rules change constantly with payers | Wants a clean service boundary; doesn't want clinical deploys to block billing |
| **Analytics team** | Heavy read access to everything without hammering the transactional DB | Wants read models / CQRS; running big queries on the clinical DB is a non-starter |
| **Clinicians (end users)** | Near-real-time lab result notifications; fast, correct records | Latency and correctness matter more than architectural elegance |
| **Security** | Strict service-data ownership, encryption everywhere, blast-radius containment | Favors few well-secured boundaries over a tangled shared-data free-for-all |

## Constraints

- **Regulatory:** HIPAA. Immutable, tamper-evident audit trail of all PHI access and mutation. Multi-region data residency may emerge as clinics expand across states.
- **Team:** 25 engineers across 3 teams. Mature enough to own services, but small enough that the microservices tax is real — they cannot afford 15 nano-services or a dedicated platform team.
- **Transactional core:** Patient record → prescription → (potential) drug-interaction check is a tightly-coupled, consistency-critical flow. Splitting it across services invites distributed-transaction pain.
- **Real-time:** Lab-result arrival is a classic async event from an external source, fanning out to one or more clinicians + the record + analytics. Synchronous polling is the wrong shape.
- **Analytics load:** Reporting queries (quality metrics, financial dashboards) are heavy reads; running them on the transactional clinical DB risks degrading the clinician-facing path.
- **Data sensitivity:** Strict data ownership — each service owns its data; no shared writable database; PHI access logged at the boundary.

## The Ask

The VP wants a recommendation addressing:

1. **Monolith, microservices, or event-driven — or a combination?** And *why*, mapped to the four factor families (business/compliance, technical, organizational, operational).
2. **How does the architecture serve the compliance/audit requirement?** (This is the regulatory backbone — do not hand-wave it.)
3. **Where do you draw service boundaries** — aligned to the teams? To the data? Both?
4. **How do you handle the real-time lab-result notification** without bolting it on as an afterthought?
5. **How do you keep analytics from crushing the transactional DB?**
6. **What's the risk of over-decomposing with only 25 engineers** — and how do you avoid it?

## Hidden Traps (the persona will probe for these — don't reveal them to yourself before answering)

> [!danger] These are for the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Single-pattern answer.** This is a HYBRID case. A candidate who picks only "microservices" or only "event-driven" misses that different sub-problems pull different ways. Strong answers combine patterns deliberately.
> 2. **Under-weighting compliance.** The audit requirement is the strongest architectural lever in the whole scenario. Event sourcing / an immutable event log gives you the audit trail *for free* as the system of record — every PHI access and mutation is an event, append-only, tamper-evident. A candidate who treats audit as "just add a logging table" has undersold the requirement.
> 3. **Distributing the ACID core.** Patient-record → prescription → interaction-check is tightly coupled and consistency-critical. A candidate who splits these into separate microservices with eventual consistency is creating patient-safety risk. Keep the transactional clinical core together (one service/one DB) — don't distribute the ACID heart.
> 4. **Ignoring team boundaries as a service-seam signal.** Three teams (Clinical/Billing/Analytics) already exist. Service boundaries aligned to team ownership (Conway's Law) reduce coordination friction. A candidate who draws boundaries that cut across teams is fighting the org.
> 5. **Over-decomposing.** 25 engineers cannot operate 15 services. The microservices tax (CI/CD, tracing, on-call per service, distributed-data bugs) is real. A few coarse-grained services aligned to teams — not a forest of nano-services.
> 6. **Lab-result notification as an afterthought.** This is a textbook EDA fan-out: external lab → event → (clinician notification + record update + analytics ingestion). A candidate who proposes "poll the labs API every minute" or "synchronous webhook to the clinician" has missed the shape. The event-driven piece belongs in the *core*, not the margins.
> 7. **Analytics hammering the transactional DB.** A candidate who doesn't address the read-side will get probed: "So analytics runs joins across the clinical DB during business hours?" The answer is CQRS-style read models / an analytics store fed by events — decouple heavy reads from the transactional core.
> 8. **Shared writable database.** A compliance red flag and a coupling trap. Each service owns its data; integration is via events/APIs, not shared tables. The audit story falls apart if services write to each other's DBs.
> 9. **No risk/trade-off articulation.** The VP needs to present trade-offs to the board. A strong answer names what this architecture *costs* (eventual consistency in places, operational complexity of an event backbone, audit-log growth) and how it's mitigated.

## Reference (what a strong answer covers)

- **Recommendation: a deliberate hybrid.** Not one pattern — three, each where it earns its keep:
  - **Microservices aligned to team boundaries** (Clinical, Billing, Analytics) — coarse-grained, each owning its data, independently deployable. This respects Conway's Law and lets Billing ship without Clinical.
  - **The transactional Clinical core stays together internally** (records, appointments, prescriptions, labs) — ACID, one database, one service — because splitting it invites distributed-transaction risk on patient-safety-critical flows. Inside, it can be a modular monolith-style service.
  - **Event-driven backbone** for: (a) the **immutable audit/compliance log** — every PHI access and mutation is an event, append-only, the system of record; (b) the **lab-result fan-out** — external lab → `LabResultReceived` event → consumed by notification, record-update, and analytics; (c) **cross-service integration** — billing consumes clinical events, analytics consumes everything.
  - **CQRS read models for Analytics** — analytics consumes the event stream and builds its own query-optimized read store. Heavy reporting never touches the transactional clinical DB.

- **Factor-family justification:**
  - *Business/compliance:* HIPAA audit trail is foundational → event sourcing for the record-access layer makes compliance a property of the architecture, not a bolt-on.
  - *Organizational:* 3 teams → service seams aligned to teams (Clinical/Billing/Analytics); 25 engineers total → coarse-grained services, no nano-service forest.
  - *Technical:* transactional clinical core needs ACID → keep together; lab fan-out is inherently async → EDA; analytics is read-heavy → CQRS read models.
  - *Operational:* independent deploy/release per team; fault isolation (a billing outage doesn't block clinical); clear on-call ownership per service.

- **Compliance as architecture, not feature:** the immutable event log *is* the audit trail. PHI access itself is logged as events. Tamper-evidence via append-only + hashing/chaining. Retention per regulation.

- **Service boundaries:** Clinical (transactional core, ACID), Billing (independent cadence, own data), Analytics (read-side, own store). Each owns its data; no shared writable DB. Integration via events + APIs.

- **Real-time labs:** external lab integration publishes `LabResultReceived`; notification service subscribes and pushes to the on-call/assigned clinician; clinical service records it; analytics ingests it. One event, multiple consumers, near-real-time.

- **Avoiding over-decomposition:** three coarse services (one per team), not fifteen. The Clinical service internally is modular/layered — it's not split further until a team splits off or a sub-domain needs independent scaling. The microservices tax is acknowledged and kept bounded.

- **Trade-offs named for the board:** eventual consistency between services (acceptable — clinical core stays consistent); operational complexity of an event backbone (mitigated by a managed broker); audit-log growth (retention + tiering policies); learning curve for teams new to events (training + pairing).

## Related Concept Notes
- [[Factors Influencing Architecture Selection]] — the four factor families driving the hybrid decision
- [[Event-Driven Architecture]] — event sourcing for audit/compliance, fan-out for lab notifications
- [[Microservices Architecture]] — service seams aligned to teams, coarse granularity, data ownership
- [[Monolithic Architecture]] — the transactional Clinical core kept together internally (modular)
- [[Layered Architecture]] — internal structure of the Clinical service
- [[Software Architecture]] — CQRS, Conway's Law, distributed-monolith anti-pattern

---

*Last updated: 2026-07-15*
