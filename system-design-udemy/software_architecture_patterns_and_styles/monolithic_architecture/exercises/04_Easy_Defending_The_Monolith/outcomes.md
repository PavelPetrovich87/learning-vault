---
status: archived
artifact_type: session_outcomes
exercise: "[[04_Easy_Defending_The_Monolith]]"
pattern: monolithic
difficulty: beginner
session_date: 2026-07-16
verdict: solid_pass_with_depth_gaps
created: 2026-07-16
updated: 2026-10-21
---

# Outcomes — Defending the Monolith Against a Sophisticated Skeptic

> Self-assessment artifact from the interview simulation. A sequel to [[01_Easy_Startup_MVP_Decision]] — same scenario, same correct answer, but the pushback was evidence-cited and half-right. The whole point of the exercise was to close the three named gaps from the prior session.

## TL;DR

**Verdict: solid pass, with depth gaps behind the labels.** Closed two of the three targeted gaps cleanly (zero-anchor, unnamed-principle). The third (boring-tax) is partially closed — I hit two of the canonical three and had to be prompted for the third. The reasoning level was visibly higher than session 01; what remains is owning the *full vocabulary and mechanism* behind each named concept, not just the label.

---

## Explicit gap check vs. session 01

The whole point of this sequel. Three gaps were named in the prior outcomes. Grading each:

| Gap | Session 01 | Session 04 | Status |
|---|---|---|---|
| **1. Boring DevOps tax skipped** | Skipped entirely; named only exotic taxes | Named CI/CD-per-service + environment parity unprompted; added local dev friction ("20 containers"); **missed service discovery until prompted** | 🟡 **Partially closed** |
| **2. Zero anchors cited** | Zero | **GitHub, Stack Overflow, Basecamp** (monoliths at scale); **Fowler's MonolithFirst** by name; **Strangler Fig** for extraction | 🟢 **Closed** |
| **3. Unnamed principles** | Reasoned to Conway / monolith-first without naming | Named **Conway's Law**, **YAGNI** (×2), **MonolithFirst** unprompted and in-flow | 🟢 **Closed** |

The boring-tax gap is the one that didn't fully land. Next time the "microservices tax" question comes, force the canonical three *before* anything else: **CI/CD per service → service discovery → environment parity.** Only then layer the exotic ones.

---

## What I nailed ✓

| Rubric item | Evidence |
|---|---|
| **Named Conway's Law early and unprompted** | First turn — "there is a law, Conway's law, architecture should repeat team communication structure." This is the direct fix for session-01 gap #3. |
| **Brought anchors under pressure** | GitHub first turn; added Stack Overflow + Basecamp when argument 1 demanded companies. Cited Fowler's MonolithFirst by name. |
| **Reframed the skeptic's own evidence** | On argument 1 ("Twitter hit a wall"): reframed it as *org-scale* pain, not traffic pain, and chained it back to Conway's Law. Their evidence served my argument. |
| **Clean concessions on argument 4** | Conceded EDA's real benefits (loose coupling, independent scaling, audit log) before naming the costs. Didn't deny true premises. |
| **Strong YAGNI application ×2** | Used it on the DevOps-tax argument (don't build platform before you need it) *and* on EDA (don't wire Kafka before boundaries stabilize). Right principle, right moments. |
| **Anchor-chaining** | Connected Twitter → org scale → Conway's Law → MonolithFirst in one thread. This is the seniority signal interviewers listen for. |

The standout move: the **Twitter reframe**. The skeptic cited a real failure; I converted it into evidence for the monolith-first thesis by correctly diagnosing *why* Twitter hit the wall (org scale, not traffic) and chaining it to a named law.

---

## Pain points & level-ups ✗

### 1. Boring-tax gap — only partially closed

**What happened:** Named pipelines and environment parity unprompted, plus a genuinely good addition (local dev friction — "20 containers to test one change"). But I led with the *exotic* taxes (orchestration, key rotation, versioning) and missed **service discovery** entirely until prompted.

**The canonical three to drill until reflexive:**
1. **CI/CD per service** — one pipeline *per service*, each with build/test/deploy/rollback.
2. **Service discovery** — services must register, find, and health-check each other at runtime. Infra a 2-person team doesn't own today.
3. **Environment parity** — N dev/staging environments, not 1.

**Why it matters:** The boring taxes are what a 2-person team pays *every day*. The exotic taxes (distributed tracing, consistency models) are real but secondary. Leading with the boring ones shows you understand the *daily* cost, not just the textbook cost.

**Drill:** When "microservices tax" is asked, name the three boring taxes *out loud, in order*, before any other cost. Make it a verbal reflex.

---

### 2. YAGNI vs Premature Optimization — conflated the labels

**What happened:** Said "YAGNI or Premature Optimization" as if they were the same principle. They aren't.

**The distinction:**
- **YAGNI** (You Aren't Gonna Need It) — building *infrastructure or features* for needs you don't yet have. This is the one I meant, both times.
- **Premature Optimization** (Knuth) — micro-optimizing *code paths* for performance before profiling proves the need. Different scope, different author.

**Drill:** Keep them distinct. YAGNI is about scope/features; Premature Optimization is about performance/cost. A senior interviewer notices the conflation.

---

### 3. Candidate capabilities vs. validated bounded contexts — intuition without vocabulary

**What happened:** On argument 5 ("boundaries are knowable now"), I had the right instinct — *"ideas on paper are just ideas until they're proved by real life"* — but couldn't attach the labels. The interviewer handed me the vocabulary.

**The vocabulary I should own:**
- **Candidate capabilities** — a feature list (Projects, Tasks, Auth…). A *hypothesis* about what the system does.
- **Validated bounded contexts** — boundaries that have survived contact with real users, real edge cases, real change requests, and proven that changes inside them don't ripple across the others. *Evidence.*

**The answer to argument 5:** A feature list is a hypothesis, not evidence. Drawing services around hypotheses means redrawing them under load — and redrawing a service boundary in production is a *migration*, not a refactor. Fowler's MonolithFirst says: you discover boundaries by *running* the monolith, not by predicting them.

**Drill:** Internalize both labels. The next time "we know our boundaries" comes up, say: *"Those are candidate capabilities, not validated bounded contexts — let me explain the difference."* See [[Identifying Microservices]] (extended) and [[DDD Bounded Context]].

---

### 4. EDA costs — right shape, incomplete list

**What happened:** Named latency, distributed-flow debugging, eventual consistency. Right *shape* of the answer. But I led with latency (the weakest of the three) and missed the costs that actually bite daily.

**The full EDA cost ledger:**
- **Eventual consistency** (strongest — lead with this)
  - **Schema evolution** — the event *is* the contract; change a field and every consumer breaks. Versioning, dual-publish periods, coordinated deployments.
  - **Exactly-once semantics** — idempotency becomes *your* problem, on every consumer, forever. Retries, redeliveries, deduplication.
  - **Dead-letter queues** — failed events need monitoring, alerting, a reprocessing story. Platform work.
  - **Outbox pattern** — to publish reliably from your own DB transaction, you need an outbox table + relay. More moving parts.
- **Distributed-flow debugging** — one event fans across N services; no single stack trace, only a trail of logs to reconstruct.
- **Latency** — a broker hop is milliseconds; real but minor. Don't lead with it.

**Punchline I delivered cleanly:** EDA's benefits (loose coupling, independent scaling, audit log) all *presuppose* stable boundaries and a scaling problem worth optimizing. We have neither. So paying the EDA tax today buys benefits we can't yet collect. YAGNI.

**Drill:** Memorize the four eventual-consistency sub-costs. When EDA comes up, name eventual consistency first, then unpack it into schema/exactly-once/DLQ/outbox.

---

### 5. Strangler Fig — named it, couldn't describe the mechanism

**What happened:** Named the pattern (big win — anchor landed) but couldn't describe *how* it works or *when* to fire it. The interviewer had to teach the mechanism.

**The mechanism (Fowler, named after the strangler fig tree):**
1. Place a **routing facade** (API gateway / reverse proxy) in front of the monolith.
2. Build one new service for one extracted capability; route *just those requests* to it. Everything else still hits the monolith.
3. Repeat — capability by capability — until the monolith has nothing left to serve, then decommission it.
4. **The monolith keeps running throughout.** No big-bang rewrite, no feature freeze. Incremental and reversible.

**The exit criteria (when to fire it):**
1. A module develops an **independent scaling need** the rest doesn't share.
2. A module gets **distinct team ownership** (a real Conway unit forms around it).
3. A **boundary stabilizes** and demands a different **release cadence** than the core.

**Why this fully defends the monolith:** The modular monolith isn't "never extract" — it's "extract *cheaply* when an exit criterion fires, because the module seams are already pre-cut." Starting with services around hypothetical boundaries means there are no clean lines to follow — only ones you redraw under load.

**Drill:** See the new [[Strangler Fig]] concept note. Memorize the facade-and-incremental-routing mechanism plus the three exit criteria.

---

## Concession discipline — partial

A senior move is *leading* with the concession before the refutation. I did this cleanly on argument 4 (EDA) but not always proactively elsewhere.

**Strong concession shape:**
> *"You're right that [true premise]. And [name the true consequence]. But [the conclusion doesn't follow], because [named principle + evidence]."*

**Where I did it well:** Argument 4 — conceded loose coupling / independent scaling / audit log before naming the costs.
**Where I missed it:** Argument 3 (DevOps tax) — I listed taxes but didn't explicitly concede "infra is cheaper to build while small" before refuting the amortization conclusion. The cleaner refutation: *"Yes, building it now is cheaper than building it later — but the tax isn't a one-time build, it's a daily maintenance cost you pay every day for capacity you don't use. YAGNI."*

**Drill:** On every skeptic argument, lead with the concession. "You're right that X. And Y. But Z, because [principle]."

---

## Summary table — strengths vs. depth gaps

| Area | Strong (named it) | Outstanding gap (full depth) |
|---|---|---|
| Anchors | GitHub, Stack Overflow, Basecamp, Fowler, Strangler Fig | — (closed) |
| Principles | Conway, YAGNI, MonolithFirst | Distinguish YAGNI from Premature Optimization |
| Boring taxes | CI/CD, environment parity, local dev friction | **Service discovery** (missed) |
| Boundaries | Right intuition | **Candidate capabilities vs. validated bounded contexts** (labels) |
| EDA costs | Latency, debugging, eventual consistency | **Schema evolution, exactly-once, DLQ, outbox** |
| Extraction | Named Strangler Fig | **Mechanism + exit criteria** |
| Concessions | Argument 4 clean | Lead with concession on every argument |

---

## Where to dig next

### Immediate (plugs today's depth gaps)

| Artifact | Why | What to extract |
|---|---|---|
| **[[Strangler Fig]]** (new concept note, created this session) | I named the pattern but couldn't describe the mechanism or exit criteria. | The facade-and-incremental-routing mechanism + the three exit criteria. |
| **[[Identifying Microservices]]** (extended this session) | The candidate-capabilities vs. validated-boundaries distinction answers argument 5 — I had the intuition, not the labels. | The hypothesis-vs-evidence framing for service decomposition. |
| **[[DDD Bounded Context]]** (re-read) | Already exists; deepening makes "validated boundary" recur reliably. | What *validates* a context — stress-testing against real usage. |

### Proposed next exercise

> [!proposal] Proposed: **05_Intermediate_EDA_Cost_Modeling** *(event-driven, intermediate)*
>
> **Why this one, and why now:** This session exposed a recurring depth gap — I can *name* EDA's costs (eventual consistency, schema evolution, exactly-once, DLQ, outbox) but I haven't yet had to *quantify* them or *decide when EDA pays off*. Argument 4 ended on "EDA is premature generalization for us" — but the natural senior-level sequel is: *"Okay, so when DOES it pay off, and how do you know?"*
>
> **The setup:** Same indie-game-dev SaaS, 18 months on. One subsystem — say, real-time multiplayer sync events — has developed a genuinely different load profile and a stabilized boundary. The skeptic (now with a real case) argues it's time to introduce an event backbone.
>
> **What it would stress-test:**
> 1. **Quantifying EDA's benefits** against its costs — not hand-waving "loose coupling," but concrete scaling/team/release-cadence math.
> 2. **Deciding between EDA and synchronous extraction** — Strangler Fig with a synchronous service vs. an event-driven backbone. When is each right?
> 3. **Designing the event contract** — schema versioning, exactly-once, outbox pattern, DLQ strategy. The mechanics I named but haven't built.
> 4. **Naming the consistency model explicitly** — strong vs. eventual vs. causal; which operations demand which.
>
> **Why it closes this session's gaps:** It forces the depth behind the EDA cost list I half-named. By the end, "schema evolution" and "exactly-once" aren't vocabulary — they're design decisions I've made under pressure.
>
> **Sequencing:** After [[03_Advanced_Strangler_Fig_Extraction]] (which teaches the *synchronous* extraction mechanics). 05 would then layer the *asynchronous* variant on top, completing the extraction toolkit.

**Alternative proposals if EDA isn't the priority:**

- **05_Intermediate_Reverse_Conway** — Argument 2's sequel. The team has grown to 8; multiple Conway units now exist. Stress-test the *reverse-Conway maneuver* (designing org structure to steer architecture) and when it actually fires. Closes the "Conway cuts both ways" thread at depth.
- **05_Intermediate_Distributed_Monolith_Diagnosis** — Already referenced in session-01 outcomes as the highest-priority sequel. If not yet done, it remains high-value: it drills the boring DevOps taxes as *symptoms* of bad decomposition, which directly patches this session's partially-closed gap #1.

---

## One-sentence takeaway

> Closed the anchor and principle-name gaps cleanly; the boring-tax gap is partially closed; the remaining work is *depth behind the labels* — own the full Strangler Fig mechanism, the full EDA cost ledger, and the candidate-vs-validated-boundary vocabulary, not just the names.
