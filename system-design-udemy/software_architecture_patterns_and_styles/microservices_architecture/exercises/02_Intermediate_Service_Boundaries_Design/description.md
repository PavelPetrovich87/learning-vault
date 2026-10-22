---
status: archived
exercise_type: interview_simulation
difficulty: intermediate
pattern: microservices
source_concepts:
  - "[[Identifying Microservices]]"
  - "[[Structuring Microservices]]"
  - "[[Microservices Communication]]"
  - "[[Microservices Overview]]"
  - "[[Factors Influencing Architecture Selection]]"
created: 2026-07-15
updated: 2026-10-21
---

# Exercise: Designing Service Boundaries for a Food-Delivery Platform

## Type
**Interview simulation** — system-design interview. The interviewer is a senior engineer who cares less about your boxes and more about *why each line is where it is*. They will probe every boundary decision.

## Difficulty
**Intermediate.** Tests the core microservices design skill: decomposing a real domain into services with the *right* granularity — not too fine (chatty), not too coarse (a mini-monolith) — with clear data ownership and defensible communication choices.

## The Situation

You're in a system-design interview. The prompt:

> *"Design the backend for a food-delivery platform — think Uber Eats or DoorDash. Customers browse restaurants and place orders; restaurants receive and prepare those orders; couriers pick up and deliver; there's payment, ratings, notifications, and real-time courier tracking. Walk me through how you'd break this into services, who owns what data, and how they talk to each other."*

**Domain detail (revealed by the interviewer if you ask):**
- **Customers** browse a catalog of **restaurants**, view their **menus** (items, prices, availability), place **orders**, pay, track the courier live on a map, and rate the experience afterward.
- **Restaurants** manage their own menus (items, prices, photos, on/off toggles for items), receive orders, mark them accepted/ready, and toggle their open/closed state.
- **Couriers** are independent contractors. The system **assigns** a courier to an order, the courier picks up, drives to the customer, and drops off. Couriers share their **GPS location** continuously while on a delivery.
- **Payments** charge the customer, pay the restaurant, and pay the courier. Fraud checks apply.
- **Ratings** are collected after delivery from both sides (customer rates restaurant + courier; restaurant rates courier).
- **Notifications** push order-state updates to customers, restaurants, and couriers (push, SMS, email).
- **Real-time courier tracking** must update the customer's map every few seconds during a delivery.

**Scale (revealed on request):** millions of customers, hundreds of thousands of restaurants, a fluctuating courier fleet, peak loads during meal windows (lunch + dinner spikes). Real-time location writes are high-volume and write-heavy; order placement is comparatively low-volume but critical.

**Greenfield.** No legacy. The candidate is designing from a blank page.

## Stakeholders & Their (Conflicting) Concerns

| Stakeholder | What they want | The tension |
|-------------|----------------|-------------|
| **Customer team** | A fast, cohesive browse→order→track experience | Wants to join catalog, order, and courier data easily — which pushes toward fewer services |
| **Restaurant team** | Full control of menu, availability, and order acceptance | Menu is their asset, but Orders needs to reference it — who owns "the price at order time"? |
| **Courier / Logistics team** | Efficient assignment + accurate live tracking | Live GPS is a high-volume firehose; mixing it into Order state bloats the order service |
| **Payments team** | Correctness, auditability, fraud isolation | Wants to own the money end-to-end; resists Orders "knowing" payment internals |
| **Platform / SRE** | Fewer chatty calls, clean contracts, independent deploys | Each new service is an operational unit they must run; pushes back on over-decomposition |
| **Engineering leadership** | Team boundaries that match Conway's law | Wants service seams that let autonomous teams ship without stepping on each other |

## Constraints

- **Decompose by business capability, not by entity or layer.** "A UserService / OrderService / DBService" is the wrong axis. The interviewer is watching for this.
- **Each piece of data has exactly one owner.** A strong answer assigns ownership explicitly (e.g., Restaurant owns the menu; Orders owns order state; Courier owns live location).
- **Granularity is a judgment call.** Too fine → chatty, operationally expensive. Too coarse → a distributed monolith with a famous name.
- **Sync vs. async must be justified per interaction.** Order placement, payment, and courier tracking have very different latency and consistency profiles.
- **Conway's law is real.** Service seams that ignore team boundaries tend to rot.

## The Ask

The interviewer expects the candidate to:

1. **Identify the bounded contexts / services** and justify each. (5–9 services is a reasonable range; defending the count matters more than the exact number.)
2. **Assign data ownership** — which service owns which data, and how other services get at data they need but don't own.
3. **Define the key APIs / interactions** between services, including whether each is synchronous or asynchronous and *why*.
4. **Justify granularity** — where did they draw a line, and what's the reasoning? Where did they deliberately *not* split?
5. **Handle the two tricky cases:** (a) where real-time courier tracking lives, and (b) where courier *assignment* lives, and why those are (or aren't) the same service.

## Hidden Traps

> [!danger] For the persona (AGENTS.md). If you're the candidate, stop reading here and answer first.
>
> 1. **Decomposing by entity or by layer.** "UserService, OrderService, PaymentService, DatabaseService, NotificationService" is entity-driven decomposition and a classic mistake. The right axis is **business capability / bounded context**. The interviewer will pounce on entity-driven splits.
> 2. **Ambiguous data ownership.** "The price at order time" is the canonical trap: the **Restaurant** owns the menu *now*, but the **Order** must own the *snapshot* of what was ordered (price, items) so a later menu change doesn't mutate historical orders. Strong candidates state this explicitly.
> 3. **Mixing live courier GPS into the Order service.** Live location is a high-volume write firehose; order state is low-volume and critical. Conflating them turns Order into a write hotspot. Strong candidates split **Courier Tracking** (the GPS firehose, time-series-ish) from **Order status** (the lifecycle). They are different concerns.
> 4. **Conflating courier *assignment* with courier *location*.** Assignment is a business decision (matching algorithm, supply/demand); location tracking is telemetry. Often separate services, though assignment *reads* location.
> 5. **Too-fine granularity.** A "PriceService," "TaxService," "FeeService," "DiscountService" inside checkout = a chatty distributed call graph for what is one decision. Strong candidates group these into a Pricing/Checkout capability unless scale forces otherwise.
> 6. **Too-coarse granularity.** One giant "OrderService" that also owns payments, courier assignment, notifications, and ratings is a mini-monolith. The interviewer will ask "what's the team structure?" to expose this.
> 7. **Sync-everywhere or async-everywhere.** Neither is right. Order placement may need synchronous validation (is the restaurant open? is the item available?) but payment and notification are better async. The candidate must justify *per interaction*, not globally.
> 8. **Ignoring Conway's law.** Boundaries that don't map to teams tend to decay into shared ownership → distributed monolith. Strong candidates explicitly connect service seams to team seams.
> 9. **Forgetting the read path.** The customer browse experience (list restaurants, search menus) is read-heavy and often served from a separate **read model / search service** (CQRS-flavored), not by querying the Restaurant service's transactional DB live.

## Reference (what a strong answer covers)

- **Decomposition by business capability** (a defensible service set):
  - **Customer / Identity** — profiles, auth, addresses, payment methods on file.
  - **Catalog / Restaurant** — owns restaurants, menus, item availability, open/closed state. *Owns the menu.*
  - **Order** — owns the order lifecycle (created → accepted → preparing → ready → picked_up → delivered). *Owns the order state and the price/item snapshot at order time.*
  - **Pricing & Checkout** — computes totals, taxes, fees, promos for an order. May fold into Order at small scale, but separates cleanly when promo/tax logic gets complex.
  - **Payment** — charges the customer, disburses to restaurant and courier, fraud checks. *Owns the money.*
  - **Courier / Logistics** — courier fleet state, availability, and the **assignment** of couriers to orders (the matching algorithm). *Owns assignment.*
  - **Courier Tracking** — the live GPS firehose and the customer-facing "where's my courier?" read model. Separated from Order because the load profile is totally different.
  - **Ratings & Reviews** — post-delivery feedback.
  - **Notifications** — a thin sender that subscribes to events (order state changes) and fans out to push/SMS/email. Never on the synchronous path.
- **Data ownership principles:**
  - Each entity has **one writer**. Restaurant owns menu; Order owns order state + the order-time price/item snapshot; Payment owns transactions; Courier Tracking owns live locations.
  - Services that need foreign data use **owned APIs** or **event-carried state** (e.g., Order keeps a snapshot of menu items at order time; Pricing subscribes to menu-change events if it needs them).
- **Communication choices (justified per interaction):**
  - *Synchronous:* Order placement validates "restaurant open? item available? price?" against Catalog via a sync call — the customer needs an immediate yes/no. (Accept the latency; make it resilient with timeouts + fallback.)
  - *Asynchronous / event-driven:* Payment processing, courier dispatch, notifications, ratings — all triggered by events (`OrderPlaced`, `OrderReady`, `CourierAssigned`, `OrderDelivered`). Decouples the write path and isolates failures.
  - *Streaming:* Courier GPS is a continuous stream into a tracking store (e.g., a time-series or geospatial store), read by the customer map view.
- **The two tricky cases:**
  - **Real-time tracking** lives in **Courier Tracking** (separate from Order) because the load profile (high-volume continuous writes, geospatial reads) is nothing like order state. Order publishes `CourierAssigned`; Tracking starts following that courier's stream.
  - **Courier assignment** lives in **Courier / Logistics** — it's a supply/demand matching decision that reads courier availability and location, not an order concern. Order publishes `OrderReady`; Logistics assigns and publishes `CourierAssigned`.
- **Granularity defense:** fewer than 5 services risks a mini-monolith; more than ~10 risks chatter and operational overhead. The candidate should name a count and defend it against both directions.
- **Conway's law:** service seams align to teams (Customer, Restaurant/Merchant, Logistics, Payments, Platform). Where one team owns two services, flag it — it may be a candidate to merge later.

## Related Concept Notes
- [[Identifying Microservices]] — bounded contexts, decomposition by business capability, granularity heuristics
- [[Structuring Microservices]] — data ownership, single-writer principle, database-per-service
- [[Microservices Communication]] — sync vs. async, API contracts, event-driven interaction, when to stream
- [[Microservices Overview]] — what services are and the granularity trade-off
- [[Factors Influencing Architecture Selection]] — Conway's law, team boundaries as a driver
- [[Event-Driven Architecture]] — events for decoupling writes and fan-out (notifications, dispatch)

---

*Last updated: 2026-07-15*
