---
status: archived
exercise_type: interview_simulation
difficulty: beginner
pattern: microservices
source_concepts:
  - "[[Microservices Overview]]"
  - "[[Microservices Challenges]]"
  - "[[Structuring Microservices]]"
  - "[[Monolithic Architecture]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: Diagnosing the "Distributed Monolith"

## Type
**Interview simulation** — consultant-style assessment. You've been brought in to diagnose an ailing system. Friendly but direct; you want evidence and a prioritized fix list, not opinions.

## Difficulty
**Easy.** Tests the single most common microservices failure mode: a system that *looks* like microservices but *behaves* like a monolith. A strong candidate spots the coupling, names why it happened, and sequences the remediation correctly.

## The Situation

You're a consultant. The VP of Engineering at **Ledgerline** — a mid-size fintech (~30 engineers) — has pulled you in for a two-week assessment. The pitch:

> *"Two years ago we 'moved to microservices' to scale the org. It was supposed to let teams move independently. Instead, every deploy is a coordinated multi-service circus, a single slow service takes the whole platform down, and nobody can ship a feature without a war room. We look like we have microservices, but we feel more monolithic than when we started. What went wrong, and what do we do?"*

**What you find when you look at the system:**

- **Five "services," one database.** Auth, Accounts, Transactions, Notifications, and Reporting are each deployed independently — but they all read and write to a single shared Postgres instance, including each other's tables. Transactions reads `accounts.balance`; Reporting joins across all tables; Notifications polls `transactions` for triggers.
- **Synchronous call chains everywhere.** A `POST /transfer` request flows synchronously: Auth → Accounts → Transactions → Notifications, all blocking HTTP. If Transactions slows down, Accounts and Notifications pile up threads and eventually exhaust their connection pools.
- **Coordinated deploys.** A schema change to `accounts` requires shipping Accounts, Transactions, Notifications, and Reporting together. The "independent deploy" promise is fiction. There's a release manager whose full-time job is herding deploys.
- **Shared libraries as a hidden coupling vector.** All services depend on a `ledgerline-common` library containing ORM models, DTOs, and validation logic. Bumping the library version means rebuilding and redeploying every service.
- **No clear data ownership.** Nobody can answer "who owns the `accounts.balance` column?" — it's written by Accounts, read by Transactions, mutated by Reporting batch jobs, and implicitly relied on by Notifications.

**Key context:**
- 30 engineers, 5 teams roughly aligned to the 5 services in name only.
- The migration was driven by "we need to scale the engineering org," not by traffic or a specific scaling bottleneck.
- The original monolith still exists as a reference but is no longer served.
- Infra is decent: Kubernetes, CI/CD per service, basic metrics — but no distributed tracing across the synchronous chains.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **VP of Engineering** | The promised microservices benefits — independent deploys, team autonomy, resilience | Got the *costs* of microservices without the *benefits*; wants a credible recovery path, not a rewrite |
| **Team leads (5)** | Ship features without a war room; own their service end-to-end | Currently blocked on coordinated deploys and shared-DB schema changes they don't control |
| **Release manager** | Predictable, low-risk releases | Exists only *because* deploys are coupled — a successful remediation eliminates this role |
| **On-call engineers** | Sleep at night; isolate failures | One slow service cascades to all others; no blast walls exist today |
| **Data / DBA team** | Clean data ownership, schema stability | The shared DB is a free-for-all; every team treats foreign tables as fair game |
| **Business / compliance** | No downtime, correct balances, auditability | Coordinated-deploy failures and cascading outages threaten both uptime and data integrity |

## Constraints

- **No big-bang rewrite.** The business will not fund "start over." Remediation must be incremental and reversible.
- **The shared database is the core coupling.** Any fix that doesn't address data ownership is cosmetic.
- **Synchronous chains create temporal coupling.** You can't fix deploy independence while runtime calls are chained.
- **30 engineers can't all pivot.** Remediation must be absorbable by the existing team structure over months, not weeks.
- **Correctness is absolute.** It's fintech — balances and transactions must stay consistent throughout any migration.

## The Ask

The VP expects a **diagnosis and a prioritized remediation plan:**

1. **Diagnose the root cause(s)** — name, with evidence, *why* this system is a distributed monolith rather than true microservices. Be specific: which couplings are structural, which are accidental.
2. **Prioritize the fixes** — what do you fix first, second, third? What's the single highest-leverage change? What can wait?
3. **Address the database** — recommend an approach to data ownership. (Database-per-service? Something staged?) Justify the sequence.
4. **Address the runtime coupling** — what do you do about the synchronous chains and the shared library?
5. **Flag what *not* to do** — which "obvious" fixes would make things worse?

## Hidden Traps

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **The shared database is the smoking gun.** Weak candidates dwell on tooling or team structure. The structural root cause is one shared DB: it re-couples everything services were supposed to decouple. A strong answer leads with this.
> 2. **"Looks like microservices, behaves like a monolith" is the named anti-pattern.** The candidate should articulate *why*: services deployed independently but coupled via shared data and sync calls have all the costs (network, ops, latency) and none of the benefits (independent change, failure isolation).
> 3. **The shared library is hidden coupling.** `ledgerline-common` shared ORM models silently re-couple every service. Strong candidates catch this and recommend extracting owned contracts / interface packages per service.
> 4. **Do NOT recommend more services.** The problem is wrongly-bounded services with wrong coupling, not too few services. Adding services *increases* coupling surface here.
> 5. **Sequence: data decoupling before runtime decoupling.** You can't meaningfully break sync chains while the DB is shared (the data still has to be consistent). Fix ownership first, then introduce async where appropriate.
> 6. **Sync vs. async is a *consequence*, not the root cause.** Recommending "just go async" without addressing data ownership rearranges the failure modes without removing them.
> 7. **Database-per-service is the destination, not day one.** Naively "give every service its own DB" tomorrow breaks every cross-table read. The transition is staged: agree ownership → route access through owning APIs → physically split.
> 8. **The migration was org-driven, not load-driven.** Acknowledge the original *intent* (scale the org) was reasonable but the execution skipped the hard part (data ownership).

## Reference (what a strong answer covers)

- **Diagnosis:** The system is a **distributed monolith** — independently deployed services that are coupled at the data layer (shared Postgres), the runtime layer (synchronous chains), and the code layer (`ledgerline-common`). It pays the microservices tax (network, ops, coordination) without realizing the benefits (independent change, failure isolation, team autonomy).
- **Root cause, prioritized:**
  1. **Shared database (structural, highest leverage).** Five services, one DB, no ownership. Every other coupling traces back here.
  2. **Synchronous call chains (temporal coupling).** A consequence of the shared-data assumption — services call each other synchronously to keep data consistent in-process.
  3. **Shared library (`ledgerline-common`).** Accidental but pervasive coupling vector; bumps force redeploy of everything.
- **Remediation, sequenced:**
  - *Phase 1 — Establish data ownership (no physical split yet).* Agree and document the single owner of each table/column. `accounts.*` → Accounts; `transactions.*` → Transactions; etc. Enforce the boundary at the *access* layer: other services stop hitting foreign tables directly and instead call the owning service's API. This is reversible and low-risk.
  - *Phase 2 — Break the shared library.* Carve `ledgerline-common` into per-service owned contracts. Bumps become local, not global.
  - *Phase 3 — Physically split the database.* Once all cross-service access goes through owning APIs, move each service's tables to its own schema/DB. The single-writer principle now holds.
  - *Phase 4 — Decouple runtime selectively.* Only now does it make sense to convert cross-service writes to async (events) where failure isolation matters — e.g., Notifications subscribing to `TransactionCommitted` rather than being called synchronously. Add distributed tracing first so you can see what you're changing.
- **What NOT to do:** Don't add more services. Don't "just go async" before data ownership is fixed. Don't attempt a big-bang database split. Don't leave the shared library in place.
- **Honest framing for the VP:** The intent (scale the org) was sound; the execution skipped the hard part — data ownership. Two years in, the fix is incremental and recoverable, not a rewrite.

## Related Concept Notes
- [[Microservices Overview]] — what microservices are supposed to deliver, and the costs when done wrong
- [[Microservices Challenges]] — distributed monolith anti-pattern, shared-database coupling, temporal coupling
- [[Structuring Microservices]] — service boundaries, data ownership, database-per-service
- [[Monolithic Architecture]] — comparison point: this system *is* a monolith, just distributed across a network
- [[Factors Influencing Architecture Selection]] — org-driven vs. load-driven migration triggers

---

*Last updated: 2026-07-15*
