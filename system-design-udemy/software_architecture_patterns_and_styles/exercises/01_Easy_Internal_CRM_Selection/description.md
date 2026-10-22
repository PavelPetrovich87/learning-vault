---
status: archived
exercise_type: planning_meeting
difficulty: beginner
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

# Exercise: Which Architecture for Our Internal CRM?

## Type
**Planning meeting** — you are the engineer the CTO pulled into a 30-minute slot. The CTO wants a clear, defensible recommendation they can approve today. Friendly, practical, cost-conscious.

## Difficulty
**Easy.** Tests the most fundamental selection judgment call: gathering real requirements and choosing the simplest architecture that fits — then defending that choice with evidence, not vibes. The trap is cargo-culting "modern" architectures onto a problem that asks for the boring answer.

## The Situation

**Company:** Northwind Metrics — a profitable B2B SaaS company (their main product is a marketing-analytics dashboard). ~80 employees total, **4 engineers** on the internal-tools track.

**What they want to build:** A new **internal CRM** for their own sales team (~50 sales reps, business-hours usage). The current process is "our reps live in a spreadsheet and a shared inbox," and it's breaking down. The CTO has greenlit a fresh build.

**Functional scope (what the reps need):**
- **Contacts** — leads, accounts, contact details, notes.
- **Deals** — pipeline stages, deal value, expected close date.
- **Activities** — log calls, emails, meetings; timeline per contact/deal.
- **Basic reporting** — pipeline by stage, rep activity, win/loss, simple charts.
- **Email integration** — sync with the reps' inbox (read/send/log), basic templates.

**Reporting detail worth noting:** A few times a week, a manager runs queries that join deals × activities × contacts to produce a pipeline summary or a quarterly rep-performance view. These are not real-time; they're "run it, get a number." Nothing exotic — no ML, no streaming analytics, no sub-second dashboards. The dataset is small — the entire pipeline fits in memory comfortably.

**What the CTO has heard on podcasts:** "Everyone's going microservices and event-driven now." They don't want to build something they'll regret — but they also don't want to waste money. They're asking you to cut through the noise.

**Explicitly NOT in scope (the CTO will confirm if asked):**
- No customer-facing exposure — this is internal-only, behind the corporate VPN/SSO.
- No mobile app — reps use it in the browser.
- No multi-tenancy — single company, single tenant.
- No global users — one office, one time zone, business hours.
- No real-time collaboration features — no "two reps editing the same deal live."
- No integrations beyond email and the account-list read sync.
- No compliance regime (no HIPAA, SOC 2 for *this* tool specifically, no audit-log mandate beyond normal SaaS hygiene).

This is, on paper, the most boring system an architect could be handed. That's the point.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **The CTO (the persona)** | A clear recommendation they can approve in this meeting — cheap to build, cheap to run, won't embarrass them in a year | Wants to feel "modern" but is fundamentally cost- and risk-averse; will probe "why not microservices?" |
| **The 4 engineers** | Something they can actually build and operate on-call without losing their minds | A microservices split across 4 people is a distributed-monolith nightmare for them |
| **Sales manager** | Fast features, reliable reporting queries, no data loss | Doesn't care about architecture — cares that deals don't disappear |
| **Sales reps (50)** | A tool that's fast during business hours and doesn't lose their notes | Light, predictable load; zero need for global scale or 24/7 SLAs |
| **Finance** | Low cloud bill, no platform team hire | Microservices + K8s + observability stack = ongoing spend they don't want for an internal tool |
| **Future Northwind (implicit)** | An architecture that can evolve *if* the CRM ever becomes customer-facing | Possible, not probable — don't optimize for a future that may never come |

## Constraints (surface these explicitly)

- **Team:** 4 engineers. No dedicated DevOps/SRE. No platform team. No on-call rotation beyond those four people.
- **Traffic:** ~50 concurrent users during business hours, near-zero off-hours. Reporting queries are bursty but infrequent (a few times a week). A single app server comfortably handles this; peak load is utterly predictable.
- **Budget:** Profitable but frugal on internal tooling. The CRM is not revenue-generating — Finance will question any infra that looks like a startup-burn cloud bill. A platform team hire is not on the table.
- **Domain maturity:** Well-understood (it's a CRM — contacts, deals, activities, reports). Boundaries are obvious from day one. This is not an exploratory product hunting for PMF.
- **Operations:** Minimal on-call expectation. Whatever you build, 4 engineers run it on rotation. If a service goes down at 2 AM, one of those four gets woken. Operational complexity is paid for in human sleep.
- **Integration:** Email sync (probably an IMAP/SMTP or provider API poll, or a provider like Gmail/Outlook); possibly read-only sync with the main product's account list so reps don't re-key accounts.
- **Reliability expectation:** Internal tool, business-hours. A 15-minute outage during the day is annoying, not catastrophic. There is no 24/7 SLA.

## The Ask

The CTO wants:

1. **A recommendation.** Monolith, microservices, or event-driven? (Or "it depends" — but you'd better resolve the dependence fast.)
2. **Justification via the four factor families** — business, technical, organizational, operational. The CTO wants reasoning they can repeat to Finance, not "trust me."
3. **An explicit rejection of microservices *and* event-driven** — with evidence, not "they're too complex." Why specifically don't they fit here?
4. **Internal structure** — if you recommend a monolith, what does it look like inside? (Layers? Modules?) How do you keep it from rotting?
5. **Exit criteria** — when would you *reconsider*? What signal would make you extract a service or add eventing later?

## Hidden Traps (the persona will probe for these — don't reveal them to yourself before answering)

> [!danger] These are for the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Recommending before gathering.** Does the candidate ask about team size, traffic, budget, domain, operations *before* naming an architecture — or do they jump straight to "monolith"? The CTO is testing whether you gather requirements first, even for an "obvious" case.
> 2. **Cargo-culting "modern."** Does the candidate recognize this is a textbook monolith case (small team, internal tool, CRUD-heavy, modest predictable traffic)? The #1 failure mode is reaching for microservices/EDA because they sound sophisticated.
> 3. **Justifying via the four factor families — explicitly.** Weak answers say "monolith is simpler." Strong answers map the factors: *organizational* (4 engineers), *technical* (low, uniform, CRUD-heavy load), *operational* (no DevOps team to pay the microservices tax), *business* (internal, cost-conscious, not revenue-generating).
> 4. **Rejecting microservices *with evidence*.** "Too complex" is not enough. The candidate should name the actual tax: CI/CD per service, service discovery, distributed tracing, network-failure modes, data consistency across boundaries — all borne by 4 people for 50 users.
> 5. **Rejecting event-driven *with evidence*.** EDA shines for fan-out, decoupling, real-time streams. This system has none of that — it's synchronous CRUD with the occasional batch report. Forcing events in adds an async mental model, an event bus, and eventual consistency for zero benefit.
> 6. **Modular monolith, not Big Ball of Mud.** A strong candidate recommends a *modular* monolith with internal layers/modules (Contacts, Deals, Activities, Reporting, Email Sync) and clear data ownership per module — so the "exit" to services later is cheap if ever needed.
> 7. **Reporting is not a reason to split.** A candidate might panic at "reporting queries" and propose a separate analytics service. The right answer: these are infrequent, business-hours, on modest data — a read replica or even just indexed queries on the same DB is plenty. CQRS/streaming is wildly overkill here.
> 8. **No exit criteria.** "Monolith forever" is as naive as "microservices from day one." A strong answer names the triggers: CRM becomes customer-facing, a team splits off, a module needs independent scaling, real-time features appear.
> 9. **Forgetting cost.** An internal tool's cloud bill is visible to Finance. The candidate should note a monolith keeps infra (one app server, one DB, maybe a read replica, a job runner) minimal and predictable.

## Reference (what a strong answer covers)

- **Recommendation:** A **modular monolith** with a clean internal layering. Single deployable, single database, internal modules (Contacts, Deals, Activities, Reporting, Email Sync) with enforced boundaries and per-module data ownership.
- **Justification by factor family:**
  - *Organizational:* 4 engineers. A microservices split would have each engineer owning (and on-calling) multiple services — a distributed monolith with no one awake to run it.
  - *Technical:* Low, uniform, CRUD-heavy traffic from ~50 business-hours users; reporting queries are infrequent and modest. No independent-scaling need, no real-time streaming, no fan-out.
  - *Operational:* No DevOps/platform team. The microservices tax (CI/CD per service, tracing, service discovery, network failure handling) is unpaid and unjustified.
  - *Business:* Internal, non-revenue, cost-conscious. A monolith keeps the cloud bill and headcount minimal. Choosing microservices here is spending money to buy complexity.
- **Explicit rejection of microservices:** The team can't operate it (4 engineers, no platform team), the traffic doesn't require it, and the domain — while well-bounded — doesn't need independent deployment/scaling per module. Net result would be a distributed monolith.
- **Explicit rejection of event-driven:** No fan-out, no real-time stream, no async workflow that benefits from decoupling. EDA would impose an event bus, eventual consistency, and an async mental model on a synchronous CRUD app — pure cost, zero benefit.
- **Reporting handled in-place:** read replica for the heavier join queries (or even just proper indexes), not a separate analytics service or CQRS pipeline.
- **Internal structure:** Layered internally (API/handlers → domain modules → data access), with module boundaries enforced (e.g., ArchUnit-style tests) so a future extraction is possible without a rewrite.
- **Exit criteria for reconsidering:** (a) CRM becomes customer-facing with real scale/SLA; (b) a team grows large enough to own a module independently; (c) a specific module needs independent scaling or release cadence; (d) genuine real-time or fan-out features appear (then eventing for *that* part, not everything).

## Related Concept Notes
- [[Factors Influencing Architecture Selection]] — the four factor families used as the decision framework
- [[Monolithic Architecture]] — modular monolith variant, "monolith first," when it's the right answer
- [[Microservices Architecture]] — the microservices tax and when org scale actually justifies it
- [[Event-Driven Architecture]] — where EDA shines (fan-out, real-time, decoupling) and why that's absent here
- [[Layered Architecture]] — internal structure of the recommended monolith
- [[Software Architecture]] — trade-offs, cargo-cult and distributed-monolith anti-patterns

---

*Last updated: 2026-07-15*
