---
status: archived
exercise_type: interview_simulation
difficulty: beginner
pattern: monolithic
source_concepts:
  - "[[Monolithic Architecture]]"
  - "[[Software Architecture]]"
  - "[[Factors Influencing Architecture Selection]]"
  - "[[Conway's Law]]"
created: 2026-07-16
updated: 2026-10-21
---

# Exercise: Defending the Monolith Against a Sophisticated Skeptic

## Type
**Interview simulation** — system-design interview, friendly but firm tone. A *second* pass at the monolith-vs-microservices judgment call, deliberately targeting the gaps left by [[01_Easy_Startup_MVP_Decision]].

## Difficulty
**Easy.** Same single judgment call — monolith vs. microservices when you're small and new — but the skeptic comes armed with *real arguments and real evidence*, not the lazy "Netflix does it" instinct. This forces the candidate to defend the monolith with **named principles and cited anchors**, not just clean reasoning.

## Why this exercise exists

> [!info] Sequel to [[01_Easy_Startup_MVP_Decision]]. That exercise ended with three named gaps: (1) the candidate skipped the *boring* DevOps taxes, (2) argued from reasoning but cited **zero evidence/anchors**, and (3) reasoned *to* named concepts (Conway's Law, monolith-first) without *naming* them. This exercise is engineered to close those three gaps. The skeptic here is sophisticated enough that hand-waving and unnamed reasoning won't survive the conversation.

## The Situation

You're in a system-design interview. The setup mirrors the first exercise — but this time the pushback is **sharp and evidence-backed**.

> *The prompt:* "Design the backend for a new SaaS product — a niche project-management tool for indie game developers. It's early days, small team, bootstrapped."

**Company context (revealed by the interviewer if asked — same as exercise 01):**
- **Team:** 2 engineers (you and a co-founder). No DevOps hire planned for at least a year.
- **Product stage:** Pre-product-market-fit. You don't yet know which features will stick.
- **Traffic:** Low hundreds of indie studios; worst-case first-year growth is low thousands. Single app server territory.
- **Budget:** Bootstrapped. Cloud bill matters; no platform team.
- **Domain:** Evolving. Feature list exists; expect to pivot parts of it on feedback.
- **Legacy:** None — greenfield.

## The Twist — a sophisticated skeptic, not a cargo-culter

The co-founder has clearly *read the literature*. They don't say "Netflix does it." They say things like this — and the interviewer will role-play each one as a genuine challenge:

> **Skeptic, argument 1 — "Monoliths don't scale."**
> *"Every major platform that started as a monolith hit a wall — Twitter's Rails monolith, Shopify's early architecture, the classic 'rewriting Twitter' post-mortems. Aren't we just deferring an inevitable, expensive migration?"*

> **Skeptic, argument 2 — "Microservices let teams move independently."**
> *"Isn't the whole point of microservices that two teams can ship without coordinating? Even at 2 people, that's less context-switching. Conway's Law cuts both ways — if we structure as a monolith, we're locked into one release train forever."*

> **Skeptic, argument 3 — "We can pay the DevOps tax up front and amortize it."**
> *"Yes, microservices need CI/CD per service, service discovery, observability. But setting that up now, while the system is small, is cheap. Doing it later, under pressure, when we're already big — that's when it's expensive. Build the platform before you need it."*

> **Skeptic, argument 4 — "Event-driven architecture decouples us for free."**
> *"We don't need synchronous microservices. An event-driven backbone — Kafka, services publishing and consuming events — gives us loose coupling, independent scaling, and a clean audit log, all at once. Why wouldn't we start there?"*

> **Skeptic, argument 5 — "Boundaries are knowable now."**
> *"You keep saying 'we don't know our domain boundaries yet.' But we have a feature list: Projects, Tasks, Auth, Notifications, Billing. Those *are* boundaries. Why not draw them as services now and avoid the painful extraction later?"*

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **You (candidate)** | Defend the modular-monolith choice with *evidence*, not just reasoning | The skeptic's arguments are real — they can't be waved away |
| **Co-founder (sophisticated skeptic)** | A "modern," decoupled architecture; has read the case studies | Each argument is half-right; the candidate must concede what's true and refute what isn't |
| **Interviewer** | Evidence-cited, principle-named defense; ability to *distinguish* good arguments from cargo-culting | Watching for unnamed reasoning and missing anchors — the exact gaps from exercise 01 |
| **Future you** | An architecture that isn't a future rewrite | Must address the "deferred migration" argument head-on |

## Constraints (surface these if relevant)

- Same as [[01_Easy_Startup_MVP_Decision]]: no DevOps maturity, unknown domain boundaries, speed-to-MVP is survival, budget-constrained, greenfield.
- **New constraint:** the skeptic is right about *some* things. A strong candidate separates the true premises from the false conclusions. "Conway's Law cuts both ways" is true; "therefore microservices now" is the false conclusion.

## The Ask

The interviewer expects you to:

1. **Defend the modular-monolith recommendation against all five arguments** — not by repeating exercise-01 reasoning, but by *citing evidence and naming principles*.
2. **Concede what's true** in each skeptic argument. A candidate who denies a true premise loses credibility; one who concedes the premise but refutes the conclusion earns it.
3. **Name the principles** — at minimum: **Conway's Law**, **Martin Fowler's "MonolithFirst" rule**, **the YAGNI / premature-generalization trap**, and the **modular monolith as a pre-cut seam** strategy.
4. **Cite the anchors** — at minimum: name **two or three companies running monoliths at scale** (Basecamp, Shopify, GitHub), and reference the actual pattern name for incremental extraction (**Strangler Fig**).
5. **Spell out the boring DevOps tax** — CI/CD per service, service discovery, environment parity — *before* the exotic taxes. The skeptic's "pay it up front" argument (argument 3) is the test.

## Hidden Traps (the persona will probe for these — don't reveal them to the candidate)

> [!danger] These are for the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Unnamed reasoning (the exercise-01 gap).** Does the candidate *name* Conway's Law, MonolithFirst, YAGNI, Strangler Fig — or do they reason to the ideas without the labels? The whole point of this exercise. Push hard: *"Is there a name for what you just described?"*
> 2. **Zero anchors (the exercise-01 gap).** Does the candidate cite Basecamp/Shopify/GitHub running monoliths at scale? Do they reference Fowler by name? Watch for reasoning with no evidence.
> 3. **The boring-tax gap (the exercise-01 gap).** When refuting argument 3 ("pay the tax up front"), does the candidate lead with CI/CD-per-service, service discovery, environment parity? Or jump to the exotic taxes (distributed tracing, consistency)? The boring taxes are the right answer here because they're the ones a small team actually pays daily.
> 4. **Failing to concede true premises.** Argument 2 ("Conway's Law cuts both ways") and argument 3 ("infra is cheaper to build while small") contain true premises. A weak candidate denies them; a strong one concedes the premise and refutes the *conclusion* (true ≠ sufficient justification now).
> 5. **Hand-waving the "boundaries are knowable" argument (argument 5).** A feature list is *not* a set of stabilized bounded contexts. The candidate must distinguish **candidate capabilities** (a list) from **validated boundaries** (stress-tested by real usage). This is the deepest trap.
> 6. **Event-driven over-reach (argument 4).** Event-driven ≠ free. The candidate should name the costs: eventual consistency, schema evolution, debugging distributed flows, exactly-once semantics. The skeptic is treating EDA as a silver bullet; the candidate must break the silver-bullet framing.
> 7. **Confusing "monolith" with "no extraction plan."** The strongest defense isn't "never extract" — it's *"we'll extract via Strangler Fig when an exit criterion fires, and the modular monolith keeps that option cheap."* A candidate who can't describe the *evolution path* hasn't fully defended the monolith.

## Reference (what a strong answer covers)

- **Recommendation:** **Modular monolith** — same as exercise 01 — but defended with named principles and cited evidence this time.
- **Defense against argument 1 ("monoliths don't scale"):** Cite **Basecamp, Shopify, GitHub** as monoliths operating at scale orders of magnitude beyond this product's ceiling. The "Twitter hit a wall" story is real but misapplied — Twitter's wall was *org* scale and *velocity*, not traffic, and their migration was years-long and painful, which is an argument *against* premature decomposition, not for it. Name **Strangler Fig** as the documented incremental-extraction pattern that keeps the exit option open.
- **Defense against argument 2 ("Conway cuts both ways"):** **Concede the premise** — Conway's Law is bidirectional. But refute the conclusion: at 2 engineers, there is one Conway unit; structuring as microservices doesn't create team independence that doesn't exist organizationally. Name **Conway's Law** explicitly. The reverse-Conway move (designing org structure) is a tool for *later*, when teams exist.
- **Defense against argument 3 ("pay the tax up front"):** Name the **boring taxes first** — CI/CD per service, service discovery/configuration, environment parity — then the exotic ones. Refute the amortization argument: building platform infra for a system that doesn't need it yet is **YAGNI / premature generalization**; the cost isn't just the build, it's the *maintenance and cognitive load* every day you don't need it. The tax compounds; you pay it *every day*, not once.
- **Defense against argument 4 ("event-driven decouples for free"):** Break the silver-bullet framing. Event-driven architecture trades synchronous coupling for **asynchronous complexity**: eventual consistency, schema evolution, distributed-flow debugging, exactly-once semantics, dead-letter queues. None of that is "free." EDA is a *later* optimization once boundaries stabilize — and even then, it's a choice with real costs, not a default.
- **Defense against argument 5 ("boundaries are knowable now"):** Distinguish **candidate capabilities** (a feature list: Projects, Tasks, Auth…) from **validated bounded contexts** (stress-tested by real users, real edge cases, real change patterns). The list is a *hypothesis*; validated boundaries are *evidence*. Drawing services around hypotheses means redrawing them under load — which is a migration, not a refactor. Name **Fowler's "MonolithFirst"**: you discover boundaries by running the monolith, not by predicting them.
- **The evolution path (closes the defense):** The modular monolith isn't "never extract" — it's "extract via **Strangler Fig** when an exit criterion fires": independent scaling need, distinct team ownership, or stabilized boundary with a different release cadence. The modular structure keeps the seams pre-cut, so extraction is a project, not a rewrite.

## Related Concept Notes
- [[Monolithic Architecture]] — modular monolith variant, "monolith first" principle, Strangler Fig extraction
- [[Software Architecture]] — anti-patterns (cargo-cult, distributed monolith, premature decomposition, silver-bullet thinking)
- [[Factors Influencing Architecture Selection]] — the four factor families; turns intuition into a repeatable framework
- [[Conway's Law]] — bidirectional org↔architecture coupling; reverse-Conway as a *later* tool

## Source Exercise
- Sequel to [[01_Easy_Startup_MVP_Decision]] — see its [[outcomes]] for the three gaps this exercise targets.

---

*Last updated: 2026-07-16*
