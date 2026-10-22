---
status: archived
exercise_type: interview_simulation
difficulty: beginner
pattern: monolithic
source_concepts:
  - "[[Monolithic Architecture]]"
  - "[[Software Architecture]]"
  - "[[Factors Influencing Architecture Selection]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: The "We Should Be Like Netflix" Startup

## Type
**Interview simulation** — system-design interview, friendly tone.

## Difficulty
**Easy.** Tests the single most common judgment call in early architecture: monolith vs. microservices when you're small and new.

## The Situation

You're in a system-design interview. The interviewer has just given you this prompt:

> *"Design the backend for a new SaaS product — a niche project-management tool for indie game developers. Tell me how you'd architect it."*

Before you can even sketch a box, the candidate-next-to-you-in-your-head (or, in a panel variant, your enthusiastic co-founder) jumps in:

> *"We should go microservices! That's what Netflix and Uber do. We'll need an auth service, a project service, a task service, a notification service, a search service..."*

**Company context (revealed by the interviewer if you ask):**
- **Team:** 2 engineers (you and a co-founder). No DevOps hire planned for at least a year.
- **Product stage:** Pre-product-market-fit. You don't yet know which features will stick.
- **Traffic:** Modest — a few hundred indie studios as early users. Worst-case first-year traffic is well within what a single app server handles.
- **Budget:** Bootstrapped. Cloud bill matters; you can't afford a platform team.
- **Domain:** Evolving. You have a feature list, but you expect to pivot parts of it based on early user feedback.
- **Legacy:** None — greenfield.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **You (candidate/co-founder)** | Ship an MVP fast, learn what users want, stay sane | Microservices would slow you down enormously at 2 people |
| **Co-founder (enthusiastic)** | "Modern" architecture; resumes; impressed by Netflix talks | Cargo-culting a pattern designed for 1000-engineer orgs |
| **Interviewer** | A reasoned, trade-off-aware recommendation | Hears "Netflix did it" as a red flag; wants to see you push back with evidence |
| **Future you (implicit)** | An architecture that can evolve when you grow | Don't paint yourself into a corner — but don't over-engineer now |

## Constraints (surface these if relevant)

- **No DevOps maturity:** No CI/CD per service, no observability platform, no Kubernetes. Microservices tax is unpaid.
- **Unknown domain boundaries:** You can't draw service boundaries you haven't discovered yet — the product is still finding its shape.
- **Speed is survival:** Every week to MVP matters; microservices add months of infra setup.
- **Budget is tight:** Many services = many replicas, more monitoring, more cost.

## The Ask

The interviewer expects you to:

1. **Recommend an architecture** (monolith vs. microservices vs. something else) and **justify it** with the specific constraints above.
2. **Push back respectfully** on the "be like Netflix" instinct — using reasoning, not just opinion.
3. Name **what would change your mind later** — i.e., the triggers for eventually extracting services.
4. (Bonus) Sketch the **internal structure** of your recommended approach so it doesn't become a Big Ball of Mud.

## Hidden Traps (the persona will probe for these — don't reveal them to yourself before answering)

> [!danger] These are for the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Cargo-cult engineering.** Does the candidate recognize that copying Netflix's architecture at 2 people is the #1 anti-pattern? They should reference *organizational scale*, not traffic, as the real driver of microservices.
> 2. **"Monolith = bad" misconception.** A strong candidate defends the monolith as a legitimate, often-correct choice — not as a compromise. They should name real companies (Basecamp, Shopify, GitHub) running monoliths at scale.
> 3. **Modular monolith.** Does the candidate proactively suggest a *modular* monolith (clear internal boundaries, data ownership per module) rather than a tangled one? This shows seniority.
> 4. **The "monolith first" principle.** Does the candidate articulate *why* you can't draw service boundaries before the domain stabilizes — and that refactoring modules is cheap, while fixing wrong service boundaries is a migration?
> 5. **Ignoring the DevOps tax.** Does the candidate explicitly note that microservices demand CI/CD, tracing, service discovery — none of which this team has?
> 6. **No exit criteria.** A weak answer says "we'll start monolithic." A strong answer says *when* and *why* they'd extract a service (independent scaling, different team, stabilized boundary).
> 7. **Conflating "monolith" with "no structure."** If they choose monolith, do they still design internal layers/modules, or do they hand-wave "one big app"?

## Reference (what a strong answer covers)

- **Recommendation:** Start with a **modular monolith** (single deployable, internal module boundaries aligned to candidate capabilities like Projects, Tasks, Auth, Notifications).
- **Justification:** 2 engineers, no DevOps, unknown domain, speed-to-MVP is survival, budget-constrained — every one of these points away from microservices.
- **Pushback on "Netflix":** Netflix uses microservices because of ~organizational scale (hundreds of teams), not traffic. Cargo-culting their architecture at 2 people is the #1 startup architecture mistake.
- **Evolution trigger:** Extract a service only when a module has a stabilized boundary AND needs independent scaling, a different team, or a different release cadence. State this explicitly.
- **Internal structure:** Layered or modular internally — don't let it become a Big Ball of Mud.

## Related Concept Notes
- [[Monolithic Architecture]] — variants (modular monolith), pros/cons, "monolith first" principle
- [[Software Architecture]] — trade-offs, anti-patterns (cargo-cult, distributed monolith)
- [[Factors Influencing Architecture Selection]] — the four factor families, decision framework

---

*Last updated: 2026-07-15*
