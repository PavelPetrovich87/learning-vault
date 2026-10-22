---
created: '2026-08-19'
difficulty: beginner
mastery: growing
source: System Design Trainer, ch. 1.3 (Su vol.1 ch.3 step 1 + Kleppmann ch.1.1)
status: analyzing
tags:
- system-design
- requirements
- fr-nfr
- interview
- process
updated: '2026-08-19'
---

# 🧠 Topic: Functional vs Non-Functional Requirements

> [!abstract] Executive Summary (ELI5)
> FR answers "**what** can the user do?" — you can demo it. NFR answers "**how well**, under what conditions?" — you measure it with a number or it shows up under load/failure. "Search with high speed" in one bullet is two requirements glued together, and neither is testable. Split them, and ground every NFR in a number — or mark it TBD and dig it out through behavior questions.

---

## 🎯 The "Why"

- **Problem Solved:** A vague problem statement is the point of a design interview, not a bug — the most-watched signal is whether you ask the right questions before drawing boxes. Fast answers without requirements ("Jimmy syndrome") = solving blind.
- **The artifact of step 1:** two lists (FR, NFR) + a list of assumptions. Without them, step 1 is a checkbox, not done.

## 🛠️ Core Principles

### The tests
- **FR test:** can you **demonstrate** it? "Look: I publish a post → it appears in the feed."
- **NFR test:** is it **measured by a number** or does it show up **under load/failure**? Scale, latency, availability, consistency, reliability.
- **Layer-sticking test:** "search with high speed" = stuck. Split → "search by messages" (FR) + "p95 search ≤ 500ms" (NFR).

### NFR without a number does not exist
"Fast", "scalable", "reliable" are wishes. Ground them:
- "Practically instant" → RTT ~150ms intercontinental → candidate `p95 ≤ 1s`; or intra-DC tens of ms → `p95 ≤ 100ms`. **Geography changes the number** — ask which one you're in.
- Latency NFRs go in **percentile form** (`p95 ≤ N ms`), never "average" — see [[Percentiles and Tail Latency]].

### Behavior, not solution (the key discipline)
Step 1 asks what the system should **do**, not how it's built inside:
- Solution-shaped (wrong): "How will we handle concurrent bookings?" (names a mechanism)
- Behavior-shaped (right): "What happens if two users book the same room at the same time?"
- Real marker: does the question **name a component/technology**? Not the presence of the word "how".

### Su's four step-1 questions + domain
1. What features? (harvests FR) 2. How many users? 3. Growth 3mo/6mo/1yr? (harvest NFR-scale) 4. Tech stack & existing services? Then add 2–3 **domain-specific behavior questions** ("is the feed chronological or ranked?" — every domain has an analogous one). Assumptions handed by the interviewer → write them down.

### Mapping to estimation
"How many users" is raw material: the MAU→DAU→RPS funnel ([[Back-of-the-Envelope Size Calculations]]) turns it into the peak QPS that sizes everything; the latency table turns "fast" into an ms budget ([[Latency Budget - System Design]]).

## ⚠️ Pitfalls

- Layer-sticking (FR+NFR in one bullet) — un-testable both ways.
- NFR without numbers — wishes, not requirements.
- Generic question lists (auth/payments/sessions) without domain specifics — "prepared from a cheat sheet".
- Solution-shaped questions hiding a mechanism.
- Over-engineering red flag #1 (Su): every design element must answer a specific requirement; "add Kafka just in case" without a requirement is the disease.

## 🔗 Related

- [[Percentiles and Tail Latency]] — the measurable form of a latency NFR.
- [[Latency Budget - System Design]] — grounding "instant" in numbers.
- [[Back-of-the-Envelope Size Calculations]] — the load funnel behind NFR-scale.

## ✅ Self-check

1. FR vs NFR in one line each? → FR = what, demonstrable; NFR = how well, measured/shown under load.
2. "History is stored forever" — FR or NFR? → NFR (storage property); the FR is "user can read old messages".
3. Interviewer says "practically instant"? → Not a requirement; ground via latency table (geo!) or TBD + behavior question.
4. Marker of a solution-shaped question? → It names a component/technology — not the word "how".