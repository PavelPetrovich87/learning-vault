---
status: archived
exercise_type: planning_meeting
difficulty: intermediate
pattern: monolithic
source_concepts:
  - "[[Monolithic Architecture]]"
  - "[[Layered Architecture]]"
  - "[[Software Architecture]]"
  - "[[Factors Influencing Architecture Selection]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: The Two-Year-Old Monolith That's Starting to Hurt

## Type
**Planning meeting** — you are the architect invited to a tech-lead planning session. The persona is a pragmatic tech lead who wants a recommendation, not a lecture.

## Difficulty
**Intermediate.** Tests the hardest judgment call in the monolith→services journey: *when* is the pain real enough to justify extraction, and *what* to extract first. Easy to over-react (split everything); easy to under-react (live with the pain).

## The Situation

**Company:** BookLoop — a 2-year-old SaaS for independent bookstores. Inventory, POS integration, customer loyalty, email campaigns. Profitable, ~3000 stores, slow steady growth.

**Codebase:** A single Rails monolith ("the Big Book"). It started clean but is now ~180k LOC. Internally somewhat layered (controllers → services → models) but the service layer has thinned into pass-throughs, and several models cross-reference each other heavily.

**The pains the team is reporting:**
- **Merge conflicts:** 8 backend engineers in one repo. PRs regularly conflict; rebase cycles eat half a day.
- **Slow CI:** Full test suite takes ~25 minutes. Engineers context-switch while waiting; velocity is dropping.
- **Deployment risk:** Deploys are a nervous affair — one bad merge took the app down for 40 minutes last quarter. Deploys happen weekly at best.
- **Coupling creep:** The "Campaigns" (email marketing) module reaches deep into "Inventory" models. A change in Inventory unexpectedly breaks Campaigns tests.
- **On-call fatigue:** One outage last month was a memory leak in the reporting/job-queue module that crashed the whole app — including checkout.

**One specific signal worth noting:** the **Reporting & Analytics** module runs heavy batch jobs (rebuilding store reports nightly). It's CPU/memory-intensive, runs on a schedule, and is the source of most on-call incidents. It does *not* need to be deployed with every feature release.

**What's *not* broken:**
- Traffic is modest and predictable (~3000 stores, mostly business-hours). A couple of app servers handle it fine.
- The team genuinely likes working in one codebase — shared utilities, easy refactors across modules (when they don't conflict).
- The domain is now well-understood after 2 years. Boundaries are visible.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **Tech lead (the persona)** | A pragmatic path that reduces pain without a rewrite | Wants evidence-based, incremental change — allergic to "rewrite everything" or "do nothing" |
| **Backend engineers** | Fewer merge conflicts, faster CI, less coupling | But they value the ease of cross-module refactors in one codebase |
| **CTO** | Velocity up, deploy risk down, costs controlled | Skeptical of a big microservices migration (cost, headcount) |
| **On-call engineer** | Sleep — reporting jobs stop crashing checkout | Fault isolation matters more to them than architectural purity |
| **Finance** | No big new infra spend | A full K8s + platform-team migration is not in the budget |

## Constraints

- **Team:** 8 backend engineers, 2 frontend. No dedicated DevOps/SRE — one backend engineer "does the ops."
- **DevOps maturity:** CI/CD exists for the monolith (one pipeline). Dockerized deploys. Some monitoring, no distributed tracing (none needed yet). No Kubernetes.
- **Budget:** Profitable but cost-conscious. Can absorb *incremental* infra (one queue, one extracted service + its DB) but not a platform team.
- **Timeline:** No hard deadline, but velocity is dropping quarter-over-quarter and leadership wants a plan.
- **Domain:** Well-understood now. The Reporting module is an obvious candidate — heavy, isolated, batch-oriented, different scaling/release profile.

## The Ask

The tech lead has invited you in to help decide:

1. **Is the pain real enough to act on?** Or is this "normal" monolith friction that discipline alone can fix?
2. **If we act, what's the path?** Three options are on the table; the tech lead wants your recommendation with trade-offs:
   - **(a) Modular monolith refactor** — enforce module boundaries in the existing codebase (ArchUnit-style guards, data ownership per module, kill the pass-through service layer). Stay one deployable.
   - **(b) Extract one service** — pull the Reporting & Analytics module into its own service with its own DB. Keep the rest as a monolith.
   - **(c) Full microservices migration** — decompose the whole app into services (Orders, Inventory, Campaigns, Reporting, Loyalty...).
3. **What's the smallest change that buys the most relief?** — the tech lead values incremental wins over big bangs.
4. **What would you *not* do, and why?**

## Hidden Traps (the persona will probe for these)

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading and answer first.
>
> 1. **Over-reacting to microservices.** Does the candidate recognize that 8 engineers + modest traffic + no DevOps team does NOT justify a full microservices migration? That's a distributed-monolith-in-waiting. The microservices tax is unpaid.
> 2. **Under-reacting (do nothing).** "Just add process" (more code review, stricter PR rules) won't fix the reporting crash taking down checkout, nor the deep Campaigns↔Inventory coupling.
> 3. **Spotting the Strangler-Fig opportunity.** The Reporting module is the textbook extraction candidate: heavy, isolated, batch-scheduled, different scaling/release profile, source of incidents. A strong candidate proposes extracting *just that* first — not everything.
> 4. **Modular monolith as the foundation.** Even if they extract Reporting, the *rest* still needs internal module boundaries enforced — otherwise the remaining monolith keeps rotting.
> 5. **Data coupling.** Reporting reads from Inventory/Orders data. Extraction means deciding: does Reporting get its own copy (via events/CDC), or call the monolith's API? The candidate should surface this, not hand-wave "extract it."
> 5. **Cost/benefit framing.** Does the candidate weigh each option against the *actual* constraints (no DevOps team, cost-conscious, no K8s)? An answer that ignores budget and team size is incomplete.
> 6. **"Rewrite" instinct.** Anyone proposing a rewrite of a profitable 2-year-old app loses points. Evolution, not revolution.
> 7. **Sequencing.** A strong answer sequences the work: (1) enforce boundaries in the monolith first (cheap, immediate CI/conflict relief), (2) extract Reporting second (fault isolation + independent scaling), (3) re-evaluate before any further splits.

## Reference (what a strong answer covers)

- **Pain is real, but localized.** Merge conflicts and slow CI are friction; the reporting-crashes-checkout is a genuine fault-isolation problem. Different pains, different fixes.
- **Recommendation: (a) + (b), explicitly reject (c).** Enforce modular boundaries in the monolith (relieves conflicts, CI scope, coupling) AND extract the Reporting module (relieves fault isolation + lets it scale/release independently). Do NOT go full microservices — team, DevOps, and budget don't support it.
- **Why not (c):** 8 engineers, no DevOps/SRE, no K8s, cost-conscious. Full microservices = unpaid tax + likely distributed monolith (shared DB would creep back in).
- **Sequencing:** Boundaries first (days/weeks of work, immediate relief), Reporting extraction second (a focused project, fault isolation win), then *stop and re-evaluate* — don't auto-split further.
- **Reporting data:** surface the decision — event-carried state (Reporting consumes `InventoryChanged`, `OrderPlaced` events) is the clean decoupling; polling the monolith API is a simpler first step.
- **Exit criteria for further splits:** only when another module hits the same triggers (independent scaling need, different team ownership, stabilized boundary).

## Related Concept Notes
- [[Monolithic Architecture]] — modular monolith variant, scaling progression, when to extract
- [[Layered Architecture]] — fixing the pass-through service layer
- [[Factors Influencing Architecture Selection]] — team size, DevOps maturity as gates
- [[Software Architecture]] — anti-patterns (distributed monolith, premature decomposition)

---

*Last updated: 2026-07-15*
