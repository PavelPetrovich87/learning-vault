---
status: archived
exercise_type: interview_simulation
difficulty: beginner
pattern: microservices
source_concepts:
  - "[[Structuring Microservices]]"
  - "[[Microservices Communication]]"
  - "[[Microservices Challenges]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The Read-Model Decision — Borrow, Ask, or Own?

## Type
**Interview simulation** — consultant follow-up. A short, focused engagement that isolates the single skill most candidates fumble after a distributed-monolith diagnosis: *how does a service get data it needs but doesn't own, once the database is no longer shared?* Friendly, direct, evidence-oriented. One decision, three consumers, one right framework.

## Difficulty
**Easy.** Tests one core skill: choosing, **per consumer**, between three read patterns — borrow (live cross-service read), ask (live API call), or own (event-fed read model) — and recognizing which consumers tolerate eventual consistency and which don't. Builds directly on the Ledgerline distributed-monolith exercise. A strong candidate frames the decision as a consistency-tolerance question, not a tooling question.

## Why this exercise exists

In the Ledgerline diagnosis, the candidate traced every coupling back to data ownership and sequenced the fix: ownership → library → physical DB split → selective async. But one skill kept arriving a beat late: **what does a read-heavy consumer (like Reporting) actually do once it can no longer join foreign tables?** This exercise isolates that decision so the candidate can lock it in. There is no diagnosis here — the system is already partially fixed. The only question is *how each of three consumers should get its data*.

## The Situation

You're back at Ledgerline — same fintech, six months later. The team took the Phase 1 advice: they've documented data ownership and routed all cross-service access through owning APIs. The shared `ledgerline-common` library has been carved into per-service contracts. So far, so good.

Now they're staring down Phase 3 — the **physical database split** — and they've hit a wall they didn't expect. Three consumers depend on data spread across the soon-to-be-split services, and each one breaks in a different way if the tables move. The VP wants your call on each, before they touch a single migration.

**The three consumers:**

1. **Customer Support Dashboard.** A real-time web UI used by support agents during live calls. An agent pulls up a customer and sees, on one screen: profile, current balances, recent transactions, and notification history. The data lives in four services (Customer, Accounts, Transactions, Notifications). Today the dashboard runs one giant SQL join across the shared DB. Agents need the view to feel **live** — a balance that changed 10 seconds ago should show on refresh. But "live" here means "fresh as of the last refresh," not "transactionally consistent."

2. **Nightly Regulatory Report.** A batch job, runs at 2 AM, produces a regulatory submission due by 6 AM. It aggregates four weeks of transaction history with customer and account context. Today it runs a long cross-table query. The regulator requires the numbers to reconcile to the cent against the source-of-truth — but **as of a fixed cutoff** (yesterday's end-of-day), not real-time.

3. **Welcome Email Sender.** Fires a welcome email when a new customer signs up. Today it's called synchronously from the Customer service on signup. It needs the customer's email, name, and language preference — all owned by Customer. It needs to fire within roughly a minute of signup. If it fires late, or twice, or not at all, that's a bug — but a one-minute delay is invisible to the user.

**The impending change:** the four owning services (Customer, Accounts, Transactions, Notifications) are about to move to **separate physical databases**. Cross-table joins will stop working overnight.

## The Ask

For **each of the three consumers**, decide and defend:

1. **Which read pattern** — borrow, ask, or own a copy — and *why that one* given the consumer's consistency tolerance and load profile.
2. **What physical change** the team must make *before* the DB split so the consumer survives it.
3. **Sync or async**, and the specific consistency contract (strong / eventual / at-a-cutoff).

Then answer the cross-cutting question: **what's the single principle that lets you decide all three quickly, without analyzing each from scratch?**

## The Three Read Patterns (framework)

Name them. The candidate should articulate these — don't hand them over until they've tried.

- **Borrow.** The consumer reads foreign tables directly (cross-service DB join or shared ORM model). Fast to build, fatal after a DB split. **Only valid when services share a database** — i.e., never, in a properly split system. This is the anti-pattern, not an option.
- **Ask.** The consumer calls the owning service's API live, every time it needs the data. Strong-ish consistency (as fresh as the owner), but **couples the consumer to the owner's availability and latency at read time**. Right when the consumer needs fresh data occasionally and can tolerate the owner being slow or down failing the request.
- **Own (read model / CQRS).** The consumer keeps its own copy of the data it needs, in its own store, refreshed by **events** the owning service emits (`TransactionCommitted`, `BalanceUpdated`, `CustomerSignedUp`). Decoupled at read time — the owner can be down and the consumer still serves. The cost: **eventual consistency** and the engineering overhead of maintaining a projection. Right when the consumer reads heavily, needs a reshaped/joined view, or needs to survive the owner being unavailable.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **Support agents** | A live, joined customer view that never feels stale | Pushes toward live cross-service calls — but four live calls per screen refresh is slow and fragile |
| **Compliance / regulator** | Cents-accurate reports reconciled to a fixed cutoff | Cares about *correctness at a point in time*, not freshness — a distinction weak candidates miss |
| **Customers** | A welcome email that arrives within a minute | Tolerates delay; does not tolerate the email firing twice or never |
| **On-call engineers** | No consumer that drags down a owning service, or vice versa | Pushes toward decoupling — borrow and even ask create blast-radius risk |
| **Data / DBA team** | No more cross-table joins against the shared DB | Wants every consumer off foreign tables before the split — no exceptions |
| **VP of Engineering** | One clear principle the team can apply to consumer N+1, not three bespoke answers | Wants the framework, not just the three decisions |

## Constraints

- **The DB split is happening.** No consumer may survive by keeping a cross-table join. That option is off the table by fiat.
- **Correctness is absolute on the Regulatory Report.** It must reconcile to the cent against the source-of-truth — but only *as of the cutoff*. This is the key distinction.
- **The Support Dashboard must feel live** — but "live" means "fresh on refresh," not "transactionally consistent across four services." No agent ever does a cross-service transaction from the dashboard; it's read-only.
- **The Welcome Email must be at-least-once**, not exactly-once, and a ~1-minute delay is acceptable. These two tolerances are the giveaway.
- **No big-bang.** Each consumer's fix must be shippable independently before its owner's DB splits.
- **Events already exist** (or can be added cheaply). The Phase 2 work introduced `TransactionCommitted`, `BalanceUpdated`, and the team can add `CustomerSignedUp` in a day.

## Hidden Traps

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Treating all three consumers the same.** The weak move is "give each one its own read model" or "make each one call the API." The three consumers have *different consistency tolerances and load profiles*, and the right answer differs for each. The exercise is testing whether the candidate *discriminates*.
> 2. **Missing the cutoff distinction on the Regulatory Report.** The report needs cents-accurate numbers — but *as of yesterday's end-of-day*. That's not strong consistency; that's **strong consistency at a fixed point in time**, which a snapshot/read model satisfies perfectly. Weak candidates hear "must reconcile to the cent" and reach for live cross-service calls, over-engineering it. A nightly snapshot fed by events is *more* correct, not less, because it's immutable.
> 3. **Confusing "live" with "consistent" on the Support Dashboard.** Agents want freshness on refresh, not cross-service transactional consistency. That's an **event-fed read model** — a denormalized "customer view" projection that refreshes on events. It reads in one query (fast), survives any owner being down, and is as fresh as the last event. A weak candidate proposes four synchronous API calls per screen refresh — slow, fragile, and drags every owner into the support path.
> 4. **Putting the Welcome Email on a synchronous call.** Today it *is* synchronous, and that's the bug. Signup should emit `CustomerSignedUp`; the email sender subscribes and fires async. This is the textbook event case: the consumer tolerates delay (1 min is fine), tolerates at-least-once (idempotency key on customer ID dedupes), and has no need for freshness. A weak candidate leaves it synchronous.
> 5. **Recommending "borrow" for any consumer.** Borrow (direct foreign-table read) is *never* correct after a split — it's the anti-pattern the split is meant to kill. If the candidate proposes it, the interviewer should stop them cold.
> 6. **Forgetting idempotency on the email sender.** If the email is event-driven and at-least-once, it *will* fire twice under retry. The sender must dedupe on a stable key (customer ID + event ID). Weak candidates miss this and ship a duplicate-email bug.
> 7. **Missing the unifying principle.** The cross-cutting answer: **the read-pattern decision is driven by the consumer's consistency tolerance and read load profile, not by the owner or the tooling.** Tolerates delay + reads often → own (read model). Needs freshness + reads occasionally → ask (live API). Never → borrow. State it cleanly.
> 8. **Conflating the read model with a cache.** A read model is an *owned, authoritative projection* with its own schema, not a Redis cache in front of the owner. The consumer owns it, maintains the projection logic, and trusts it as its source of truth at read time. Weak candidates reach for "just cache it."

## Reference (what a strong answer covers)

**The unifying principle** (state this first, then apply it): *for each consumer, ask two questions — (a) what consistency does it actually need, and (b) what's its read load profile? The pair determines the pattern. Borrow is never an option post-split.*

**Consumer 1 — Support Dashboard → Own (read model).**
- *Why:* read-heavy (every agent screen load), needs a joined/reshaped view across four services, tolerates seconds-of-staleness ("fresh on refresh"). Four live API calls per refresh would be slow and would couple support's availability to all four owners.
- *Pattern:* an **owned read model** — a denormalized "Customer View" projection (profile + balances + recent transactions + notification history) in the dashboard's own store, fed by events (`BalanceUpdated`, `TransactionCommitted`, `NotificationSent`).
- *Consistency:* eventual, seconds behind. Acceptable — agents read, they don't transact across services from this view.
- *Survives the split:* yes — reads from its own store; owners can move or be down.

**Consumer 2 — Nightly Regulatory Report → Own (snapshot read model).**
- *Why:* needs cents-accurate aggregation — but *as of a fixed cutoff*. A live cross-service query would be over-engineered and fragile; a **point-in-time snapshot** is more correct because it's immutable and auditable.
- *Pattern:* an **event-fed read model** that projects transaction + customer + account context into a reporting store continuously; the nightly job reads from that store with a `WHERE as_of <= <cutoff>` filter. The report reconciles because the projection is append-only and the cutoff freezes it.
- *Consistency:* strong *at the cutoff* (eventual up to the cutoff, then frozen). This satisfies the regulator.
- *Survives the split:* yes — the job reads from the reporting store, not the owners' tables.

**Consumer 3 — Welcome Email → Ask, but asynchronously (event-driven).**
- *Why:* needs almost-no data (email, name, language), tolerates ~1-min delay, tolerates at-least-once. This is the textbook **event** case — not because it needs a read model (the data is tiny), but because it should be *decoupled from the signup transaction*.
- *Pattern:* Customer service emits `CustomerSignedUp` (with the needed fields embedded — event-carried state transfer); the email sender subscribes and fires async. **Idempotency key** (customer ID + event ID) dedupes retries.
- *Consistency:* eventual, at-least-once, idempotent. Correct for email.
- *Survives the split:* yes — the email sender never touches Customer's tables; it reacts to an event with the payload it needs.

**What NOT to do:**
- Don't keep any cross-table join alive (borrow is dead post-split).
- Don't put the Support Dashboard on four synchronous API calls per refresh — slow and fragile.
- Don't make the Regulatory Report query live owners at 2 AM — over-engineered; a snapshot is more correct.
- Don't leave the Welcome Email synchronous — that's the existing bug.
- Don't call the read models "caches." They're owned projections with their own schema and lifecycle.

**Honest framing for the VP:** "You don't need three bespoke designs. You need one question — *what consistency does each consumer tolerate, and how often does it read?* — and the pattern falls out. Dashboard and Report own their data via event-fed read models; the email sender reacts to an event. None of them touch a foreign table after the split."

## Related Concept Notes
- [[Structuring Microservices]] — single-writer principle, database-per-service, the access-boundary enforcement this builds on
- [[Microservices Communication]] — sync vs. async, event-driven interaction, event-carried state transfer
- [[Microservices Challenges]] — read-model / CQRS as the answer to "how do I read foreign data after the split"
- [[Microservices Overview]] — why microservices decouple reads from writes via owned projections

---

*Last updated: 2026-07-17*
