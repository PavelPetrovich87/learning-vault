---
status: archived
exercise_type: interview_simulation
difficulty: advanced
pattern: monolithic
source_concepts:
  - "[[Monolithic Architecture]]"
  - "[[Microservices Architecture]]"
  - "[[Microservices Challenges]]"
  - "[[Factors Influencing Architecture Selection]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: Extracting the Hot Module from a Shopify-Scale Monolith

## Type
**Interview simulation** — staff-level system-design interview. The interviewer is rigorous and expects senior-level reasoning about data coupling, migration risk, and incremental extraction.

## Difficulty
**Advanced.** Tests the hardest monolith scenario: extracting a *single* service from a large, successful monolith where the data is tightly coupled — and doing it safely without a big-bang rewrite, without downtime, and without breaking the rest of the system.

## The Situation

**Company:** Stitchline — a large e-commerce platform (~$2B GMV, ~50k merchants, ~10M daily active shoppers). The core is a single, mature monolith ("the Loom") that has scaled remarkably well — à la Shopify/GitHub/Basecamp. It's internally disciplined: modular structure, clear module boundaries, owned data per module. Traffic scales fine with horizontal replication and caching.

**The problem child:** the **Pricing & Promotions Engine.** It re-evaluates cart totals, applies promo codes, computes dynamic/flash-sale prices, and calculates merchant payouts. It's:
- **Computationally heavy** — every cart update triggers a re-pricing pass; during flash sales it's CPU-bound.
- **Under enormous, spiky load** — flash sales drive 10–20x normal traffic concentrated on pricing.
- **On a different release cadence** — the pricing team ships rule changes weekly; the rest of the monolith ships daily. Coordinating deploys is painful.
- **Owned by a dedicated team** — the Pricing team (12 engineers) is functionally independent but blocked behind the monolith's deploy cycle.
- **Tightly coupled to the data** — Pricing reads from and writes to the **Orders, Cart, Catalog, and Merchant** tables, all in the monolith's single shared Postgres database. Pricing logic is woven through 40+ code paths in Orders and Cart.

**The trigger:** last Black Friday, Pricing consumed so much CPU during a flash sale that it degraded the *entire* monolith — including checkout — for 18 minutes. The business cost of that incident was significant. The mandate from the CTO: **extract Pricing into its own independently scalable service within two quarters.** No big-bang rewrite. No downtime. The rest of the monolith stays.

**The catch:** Pricing's logic is deeply woven into Orders and Cart code. It shares the database with them. A "lift and shift" will tear the coupling and break things.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **CTO** | Pricing extracted in 2 quarters, no downtime, no repeat of Black Friday | Hard deadline; high stakes |
| **Pricing team (12 eng)** | Own their service, ship on their cadence, scale independently | Eager to extract — but they own the migration risk too |
| **Orders / Cart teams** | Don't break checkout, don't add latency to the hot path | Pricing extraction must not slow down cart operations |
| **On-call / SRE** | No 3 AM migration fires, ability to roll back | Wants feature flags, canary deploys, dual-run windows |
| **Data platform team** | Clean data boundaries, no dual-write inconsistency | Worried about Pricing writing to shared tables post-extraction |
| **Business / merchants** | Pricing always correct — no wrong totals, no promo errors | Correctness is non-negotiable; a wrong price is a legal/PR issue |

## Constraints

- **No big-bang rewrite.** Incremental migration only — the Strangler Fig pattern.
- **No downtime.** Checkout must stay up throughout. Migrations happen behind feature flags with dual-run windows.
- **Shared database is the core coupling.** Pricing reads/writes Orders, Cart, Catalog, Merchant tables. Untangling data ownership is the central challenge.
- **Latency budget:** cart operations currently run in-process (nanoseconds). Post-extraction, Pricing calls become network hops (milliseconds). The cart hot path is latency-sensitive.
- **Correctness is absolute.** A wrong price is a business/legal incident. Dual-run must produce identical results before cutover.
- **Timeline:** 2 quarters (~6 months). Pricing team leads the migration with support from Orders/Cart teams.
- **Infrastructure is mature:** CI/CD, Kubernetes, observability, distributed tracing, feature flags — all in place. The DevOps tax is paid.

## The Ask

The interviewer expects a migration plan that addresses:

1. **The Strangler Fig sequence** — how do you incrementally route pricing traffic from the monolith to the new service, without a flag day?
2. **Data decoupling** — how do you untangle Pricing's reads/writes from the shared database? (API calls? Event-carried state? CDC? A copy?) What are the trade-offs?
3. **The dual-run / cutover strategy** — how do you verify the new service produces identical results before flipping traffic? How do you roll back?
4. **Latency** — how do you avoid turning in-process pricing calls into slow network hops on the cart hot path?
5. **Risk management** — feature flags, canaries, shadow traffic, rollback plan. What's your safety net?
6. **The sequencing across 6 months** — what's the order of operations? What's the "definition of done" for each phase?

## Hidden Traps (the persona will probe for these)

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading and answer first.
>
> 1. **Big-bang temptation.** "Rewrite Pricing as a service and switch over" is rejected on sight. The candidate must propose incremental, Strangler-Fig extraction.
> 2. **Data coupling hand-wave.** "Just give Pricing its own database" ignores that Pricing reads/writes four other modules' tables. The candidate must address *how* Pricing gets that data (API? events? CDC? duplicated snapshot?) and the trade-offs of each.
> 3. **Dual-write problem.** If Pricing keeps writing to shared tables during transition, you have dual-write inconsistency. Strong candidates raise the **Outbox pattern** or a clear single-writer transition plan.
> 4. **Latency blindness.** Turning a synchronous cart re-pricing into a network call may blow the latency budget. Strong candidates consider: keep hot-path pricing *in-process* via a library/shared module initially; use async/cached pricing; or co-locate the service.
> 5. **No cutover safety.** No feature flag, no shadow/dual-run, no rollback = unacceptable at this scale. The candidate must propose dual-run (run old + new, diff results) before shifting traffic.
> 6. **Ignoring correctness validation.** Pricing errors are business/legal incidents. The candidate must specify *how* dual-run verifies identical outputs (shadow traffic, result diffing, sampled comparison).
> 7. **No phased plan.** A single "we'll migrate it" with no phases, milestones, or rollback gates is weak. Expect: (1) stand up service with no traffic, (2) shadow run + diff, (3) canary a % of traffic, (4) ramp with rollback ready, (5) retire old path.
> 8. **Forgetting the people/coordination.** Pricing, Orders, Cart, and Data teams must coordinate. Cross-team contracts, API versioning, and a clear ownership transition matter.

## Reference (what a strong answer covers)

- **Approach: Strangler Fig, incremental.** Build the new Pricing service alongside the monolith; route traffic gradually; retire the old code path only after validation.
- **Data decoupling — staged:**
  - *Phase 1 (read-side):* Pricing service reads via the monolith API initially (simplest), then migrate to **event-carried state** — Pricing subscribes to `CatalogChanged`, `CartUpdated`, `MerchantUpdated` events and maintains its own read model. Eliminates synchronous cross-module reads.
  - *Phase 2 (write-side):* Pricing stops writing to shared tables. The **single-writer principle** — each piece of data has exactly one owner. Pricing writes only to its own DB; it emits `PriceCalculated` events that Orders/Cart consume. Use the **Outbox pattern** to avoid dual-write inconsistency.
- **Dual-run / cutover:**
  - Run old (in-monolith) and new (service) pricing in **shadow mode**: same inputs, diff the outputs. No traffic served by new yet.
  - When diffs are zero (or understood) for a sustained window, **canary** a small % of pricing traffic to the new service behind a feature flag.
  - Ramp gradually (1% → 10% → 50% → 100%), monitoring error rates and latency. Roll back instantly via flag if metrics degrade.
  - Retire the old code path only after 100% sustained + a soak period.
- **Latency:** avoid network hops on the *cart hot path*. Options: keep a thin pricing "calculator" shared library for the synchronous hot path initially; pre-compute/cache prices for common carts; co-locate the service in the same cluster/zone; reserve full network calls for non-hot-path (batch/promo-rule) pricing.
- **Correctness:** shadow-traffic diffing is the gold standard — run both, compare every output, alert on any divergence. Sampled deep comparison if full dual-run is too costly.
- **Phasing over 6 months:**
  1. Q1: Stand up service, read via API, shadow-run + diff (no traffic).
  2. Q1 late / Q2 early: Migrate reads to event-carried state; canary traffic behind flags.
  3. Q2: Ramp to 100%, transition write ownership (Outbox), retire old path.
  - Each phase has explicit rollback gates.
- **Risk:** feature flags on every cutover step, canary deployments, shadow diffing, instant rollback, and a dedicated on-call during ramps.

## Related Concept Notes
- [[Monolithic Architecture]] — scaling progression, Strangler Fig, when to extract
- [[Microservices Architecture]] — independent scaling, the extraction decision
- [[Microservices Challenges]] — data consistency, Outbox pattern, dual-write problem
- [[Factors Influencing Architecture Selection]] — extraction triggers

---

*Last updated: 2026-07-15*
