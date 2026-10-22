---
created: '2026-08-03'
updated: '2026-08-06'
status: analyzing
difficulty: intermediate
mastery: growing
source: "Wiegers & Beatty, Software Requirements (3rd ed., 2014), Ch. 1, pp. 2–25"
tags:
  - business-analysis
  - ba-dev-trainer
  - wiegers
  - chapter-1-1
---

# 1-1 Requirements Foundations — Levels, Types, Feature

> Consolidated note for Chapter 1.1 of the "BA for Developer" course
> (`ba-dev-trainer/` repo). Source: Wiegers & Beatty, *Software Requirements*,
> 3rd ed., 2014, Ch. 1 (book pp. 2–25). Tag: `[WB]`.

## 0. Core intuition (from the session)

> **Requirements cannot be guessed — they must be elicited.** People (and AI
> agents) are not mind readers. The majority of production bugs are not born
> in code, but in an incomplete or ambiguous understanding of *what* to build.
>
> **40–50% of all product defects** originate at the requirements stage
> (Davis, 2005). Rework consumes 30–50% of budget; requirements errors account
> for 70–85% of rework cost.

**Mindset shift for a developer:** a specification is a *hypothesis* to be
verified, not "the analyst's document that can be ignored". Defects in it cost
multiples more than code defects.

---

## 1. What is a requirement — three facets

After Sommerville/Sawyer (1997):

> A **requirement** is a specification of what should be built. It describes
> the system's behaviour, its properties, or its attributes. It can also serve
> as a constraint on the development process.

Three facets of one notion = three different places to go wrong:

| Facet | Question | Example | Marker |
|---|---|---|---|
| **Behaviour** | What does the system *do*? | "Sends an email on status change" | rephrases as an action verb |
| **Property** | What *is* the system? | "Available in 5 languages", "responds in 200 ms" | describes a characteristic |
| **Constraint** | Within what *bounds* is it built? | "Runs on PostgreSQL 15", "GDPR-compliant" | a frame, often imposed externally |

**Grey zone:** multilingual support is a property with a tinge of constraint.
If the customer dictates "5 languages mandatory for our market", it feels like
a constraint. The boundary is fluid — and that's normal.

**On the word "requirement" itself.** In the dictionary = "mandatory". In our
field — no: a low-priority / deferred requirement is still a requirement. It
is recorded so as not to be lost (the backlog is a container for deferred but
not forgotten items).

---

## 2. Three levels of requirements (the core)

This is a top-down **causal chain**, not three parallel boxes:

```
LEVEL 1. BUSINESS REQUIREMENTS        — answers "WHY"
  Why does the organisation need this system?
  Container: Vision & Scope document
  Example: "Cut front-desk staffing costs by 25%"
        │ generates
        ▼
LEVEL 2. USER REQUIREMENTS            — answers "WHY THE USER NEEDS IT"
  What the user must be able to do (a task, not a button).
  Container: User requirements document
  Form: use case / user story "As [role], I want [action], so that [value]"
  Example: "As a passenger, I want to check in for my flight, so I can board"
        │ generates (one → many)
        ▼
LEVEL 3. FUNCTIONAL REQUIREMENTS (FR) — answers "WHAT THE SYSTEM DOES"
  System behaviour, "shall…".
  Container: SRS (Software Requirements Specification)
  Example: "The system shall print the boarding pass"
```

**Mnemonic:** Why → Why-the-user → What-the-system-does.

**Key ratio:** one user requirement typically generates **several** functional
requirements. This is the requirements tree: root (business goal) → branches
(user tasks) → leaves (FRs).

---

## 3. Types that "permeate" all levels

These types **do not belong** to any single level — they apply to all three:

- **Quality attributes (NFRs)** — *how well* the system does what it does.
  Performance, availability, security, portability. "Responds in 200 ms."
- **External interfaces** — *what* the system interacts with: other programs,
  devices, users (UI).
- **Constraints** — bounds: technology, budget, regulations.
- **Business rules** — policies, regulations, standards. **Exist outside the
  software system.** The system obeys them, spawning FRs.

> **Common mistake:** "all non-functional = quality attributes". No.
> Constraints and external-interface requirements are also non-functional.
> Non-functional = "everything that is not functional".

> **Permeation is not membership.** The vertical purple rail on the concept
> map means "this type *can appear* at all three levels, in different
> formulations" — not "this type *belongs to* level X". Knowing the type
> does **not** tell you the level; the level (business / user / functional)
> is a separate coordinate.

---

## 4. Feature — a granularity notion, not a "how well"

> **Course terminology decision (2026-08-03).** The book translates *feature*
> as "характеристика", but that conflicts with quality attributes. *Feature*
> is a **granularity** notion: a named product capability = **a group** of
> related FRs. The book itself is inconsistent: table 1-1 says "характеристика",
> the caption to fig. 1-2 says "дерево функций". The course fixes
> **"function (feature)"**.

**The ladder of detail:**

| Level | What it is | Example ("Browser bookmarks") |
|---|---|---|
| **Feature (function)** | A named capability = a set of related FRs | "Bookmarks" as a whole |
| **Functional requirement** | One atomic behaviour | "The system shall allow renaming a bookmark" |
| **Quality attribute** | How well it works | "Bookmarks load in 200 ms" |

A feature sits **between** a user requirement and an individual FR: user
requirement → one or more features → each feature decomposes into several FRs.

**Recognition rule:** if a capability can be "shown to the user as a single
item" (bookmarks, dark theme, spell-check) and it decomposes into 3–10
"the system shall…" statements — it's a feature.

---

## 5. Business rule vs Functional requirement — the key distinction

|  | Business rule | Functional requirement |
|---|---|---|
| **Where it exists** | Outside the software system (company policy, law, standard) | Inside the system (its behaviour) |
| **Marker** | Can be enforced manually, without software | Requires the system to be implemented |
| **Example (StudyRoom)** | "The coworking charges a surcharge for exceeding the 4-hour/day booking limit" | "The system shall block bookings longer than 4 hours" |
| **Relation** | Generates → feature → many FRs | A consequence of a business rule |

**Why it matters:** a business rule *can change* (limit 4→6 hours). If you
mixed the rule with the FR, then when the rule changes you have to hunt down
and patch a pile of FRs across the SRS. If you kept them separate — you change
*one constant*.

> **Refined distinction (2026-08-06).** A business rule does **not** always
> generate a feature. Per the book (p. 11), a rule generates "functional
> requirements **or** quality attributes". Three real outcomes:
> - **Several related FRs → they group into a feature** (common for substantial
>   rules: "4-hour limit" → limit-control + surcharge + notifications features).
> - **One FR, no feature** (common: "passport at registration" → one FR
>   "passport field is required").
> - **A quality attribute** (common: "customer data is confidential" → a
>   security NFR, not an FR).
> - **Nothing** (the rule exists but the system simply ignores it — e.g. a
>   lunch-break policy).
>
> The red arrow on the concept map is therefore labelled **"can generate
> (via a group of FRs)"**, not "generates".

---

## 6. Complex requirements trees (food for thought)

### Tree 1: one business rule → several features → many FRs

```
Business rule: "Booking limit 4 hours/day per user"
   │
   ├─ Feature A: "Limit control"
   │    ├─ FR: system sums a user's booking time per day
   │    ├─ FR: system blocks new bookings on overflow
   │    └─ FR: system shows remaining limit in the UI
   │
   ├─ Feature B: "Surcharge for overflow"
   │    ├─ FR: system computes the surcharge by tariff
   │    ├─ FR: system charges via the payment gateway
   │    └─ FR: system sends a receipt
   │
   └─ Feature C: "Limit notifications"
        ├─ FR: system sends a push at 80% of the limit
        └─ FR: system sends an email on overflow
```

**Growth point:** if the limit changes from 4 to 6 hours — all three features
are affected, but the **business rule is one**. You change the constant → the
features read it.

### Tree 2: one user requirement → many FRs via different features

```
User requirement:
  "As a tenant, I want to book a meeting room, so I can hold a meeting"
   │
   ├─ Feature: "Room search"
   │    ├─ FR: filter by capacity
   │    ├─ FR: filter by equipment (projector, board)
   │    └─ FR: availability calendar
   │
   ├─ Feature: "Booking creation"
   │    ├─ FR: time-slot selection
   │    ├─ FR: participant entry
   │    └─ FR: slot-conflict check
   │
   └─ Feature: "Booking confirmation"
        ├─ FR: send email with details
        ├─ FR: generate door QR code
        └─ FR: add to participant's calendar
```

**Question to ponder:** is email confirmation (an FR in the "Confirmation"
feature) justified by the user requirement "hold a meeting"? Or is it a
"bow"? Depends on whether the user needs *proof / a pass*. Without that
"why" — it's gold plating.

### Tree 3: the trap — an FR without a user-requirement parent

```
??? (no user requirement)
   │
   └─ FR: "The system shall send every user a daily morning email
           with a weekly summary of their bookings"
```

**Diagnosis:** this is a "bow" (gold plating) until proven otherwise. A
developer added it "because it seemed useful". Every FR must have a user-
requirement parent. Without one — a candidate for deletion or priority
lowering.

**How to check:** ask "which user requirement does this FR serve?" If there
is no answer, or "well, it's generally useful" — red flag.

---

## 7. Every project has requirements; symptoms; economics (Module 6)

### Brooks — "the essential difficulty"

> "The hardest part of building software systems is deciding precisely *what*
> to build. … No other part of the work so damages the result if done poorly.
> No other part produces errors that are harder to fix later." — F. Brooks,
> "No Silver Bullet", 1987

Three parts to Brooks's thesis: (1) **the hardest thing is not to write, but
to elicit** a requirement; (2) this part **wrecks the entire result** if done
badly — no amount of beautiful code saves it; (3) errors here are **the
hardest to fix**, because they surface late and pull a cascade (change the
requirement → redesign → rewrite code → redo tests → re-release).

Consequence: time spent on elicitation is **not a release delay, it's an
investment**. It is often impossible / unnecessary to fully define requirements
before design starts — work **iteratively**: a portion of requirements →
stakeholder reaction → next portion. Critically: **iterations during coding
cost far more than iterations during concept development.**

### Defect economics — three threshold numbers

| What | Number | Source |
|---|---|---|
| Share of product defects seeded at the requirements stage | **40–50%** | Davis, 2005 |
| Share of budget consumed by rework | **30–50%** | Shull et al., 2002; GAO, 2004 |
| Share of rework cost attributable to requirements errors | **70–85%** | Leffingwell, 1997 |

**The defect-cost curve (Boehm/Grady):**

| When found | Relative cost | Why |
|---|---|---|
| While writing the requirement | **$1** | rewrite one SRS line |
| At design | **$2–3** | rewrite SRS + redo architecture |
| At coding | ~$5–10 | + rewrite code + tests |
| In production, by a user | **$100+** | + release, distraction, reputation, support |

Wiegers's case: a defect caught by internal review — $200; the same defect
noticed by a user — $4200. **21× more expensive.** Field data, not theory.

**Practical takeaway:** when arguing "spend another day clarifying the
requirement, or start coding now?" — you have the arithmetic. A day of
clarification now is cheaper than a week of rework later, with the
probability given by this table.

### Six symptoms of poor requirements

The book gives **symptoms** (what you *observe*), not causes. You see the
symptom first, then look for the root failure in the requirements process.

| #   | Symptom                           | What you see                                                                                   | Remedy                                                                                                                                                                                 |
| --- | --------------------------------- | ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Insufficient user involvement** | The analyst "invented" business requirements; the team built "ideal" requirements nobody needs | Constant contact with real users, not their managers                                                                                                                                   |
| 2   | **Careless planning**             | Vague requirements yield over-optimistic estimates; an estimate sounds like a commitment       | Main overspend drivers: frequent changes, missed requirements, poor analysis, under-detailed SRS — fight each by name                                                                  |
| 3   | **Scope creep**                   | The project grew beyond bounds; requirements keep growing                                      | Clarify business goals / vision / constraints; assess the impact of each new feature; buffers; in agile — fix iteration budget, adjust scope                                           |
| 4   | **Ambiguous requirements**        | One text, different understandings by different readers                                        | **Official review** with different stakeholders. Informal reading does *not* reveal ambiguity — everyone "understood their own way, and it made sense to each". Prototype + tests help |
| 5   | **Gold plating**                  | A developer added a feature "because it'd be nice", not in the spec                            | Trace every item to a source; agree ideas with the customer                                                                                                                            |
| 6   | **Missing user classes**          | An important group is undefined → its needs unaccounted for                                    | Don't forget support, data migrators, regulators (even "those unaware of the project")                                                                                                 |

### Subtle distinctions that trip people up

- **"Under-specified" vs "ambiguous"** — different symptoms.
  - *Under-specified* = not detailed enough → symptom #2 (careless planning).
  - *Ambiguous* = admits multiple interpretations → symptom #4.
  - Marker: one text yields different *sensible* readings by different
    readers → #4, not #2.

- **Elicitation failure vs specification failure** — different root causes.
  - If the requirement *was never known* (nobody knew about it) → **elicitation**
    failure.
  - If the requirement *was elicited but never persisted* (only verbal in a
    call) → **specification** failure.
  - Marker "verbal but not recorded" = specification failure, not elicitation.

- **Scope creep vs gold plating** — similar result (overspend), different
  source and direction.
  - *Scope creep* — the **customer** throws in new requirements; growth "from
    outside". Remedy: change management.
  - *Gold plating* — the **developer** adds "goodies" nobody asked for; growth
    "from inside". Remedy: discipline + traceability.

### Self-diagnosis template

When you see rework / conflict / "not what I asked for" → walk the chain:
1. **Symptom** — which of the six fired?
2. **Subprocess** — which of the four development subprocesses (elicitation /
   analysis / specification / validation) failed first?
3. **Cost curve** — which step ($1 → $100) are we at at now?
4. **Action** — what specifically would have prevented a repeat?

---

## 8. Concept map

The whole chapter on one diagram — three axes and the relations between them.
Click the image to open it in the Draw.io editor inside Obsidian.

![[concepts-map.drawio]]

**Three axes:**

1. **Center vertical — three LEVELS** (why → why-the-user → what-the-system-does).
   A causal chain: a business requirement *generates* a user requirement, which
   *generates* (one → many) functional requirements.
2. **Right — TYPES AROUND** (quality attributes, external interfaces,
   constraints, business rules). They do **not** belong to any one level — the
   vertical purple rail "all 3 levels" means they *permeate* every level, in
   different formulations. Knowing the type does not tell you the level.
3. **Left — PROCESS axis**: development (4 subprocesses) → management
   (baseline / impact analysis / traceability / status tracking).

**Reading the arrows:**

- **solid arrow** = "generates / leads to" (causal);
- **dashed arrow** = "applies to / permeates" (not causal, but applicable);
- **dashed red "can generate (via a group of FRs)"** = a business rule is the
  *source* of one possible chain (rule → feature → many FRs), but not every
  rule yields a feature — sometimes it yields a single FR, a quality attribute,
  or nothing.

**Key relations that used to feel "flat":**

1. **Business rule → feature → many FRs** — the chain where a rule is easy to
   confuse with an FR.
2. **Business requirement → user requirement → FR** — the main "tree" of the
   chapter.
3. **Types-around permeate the levels** — they are not "nearby", they apply to
   each level in their own formulation.
4. **An FR "lives in" the SRS → product**; migration / training / documentation
   → *project*.

---

## 9. Self-check questions

1. Name the three requirement levels top-down and the question each answers.
2. How does a feature differ from an FR? And from a quality attribute?
3. "Migrate data from the old system" — is it a product or a project
   requirement? (Answer: a *transition* requirement, a project category.)
4. Given the business rule "4-hour limit": which FR does it generate? Name a
   feature it would belong to.
5. You see the FR "the system shall send an email". What question do you ask
   to check whether it's a "bow"?
6. State the defect-cost curve ($1 → … → …) and the three economics
   thresholds (40–50%, 30–50%, 70–85%).
7. List the six symptoms of poor requirements. Which symptom describes "one
   text, different readings by readers"? Which — "every week the PM throws in
   a new feature"?
8. The team is reworking a module because "the customer meant something else",
   and there was no user requirement in the backlog — only a verbal
   description from a call. Which development subprocess failed first:
   elicitation or specification?

---

## 10. The classification algorithm (from the Exercise 2 session)

> Born from a real pain: given a mixed bag of Slack tasks, "I float — I see no
> clear separation algorithm". This section IS the algorithm. Two independent
> questions, applied in order — never jump to answers by keyword.

### Step 1 — Is the item a statement or an activity? (the fork)

Ask: **does this item say WHAT MUST BE (a property, a condition) — or WHAT
SOMEONE MUST DO (an action)?**

> «Система должна позволять отменять бронь» — a property of the system → a requirement.
> «Съездить на ресепшен и посмотреть» — an action → an activity, not a requirement.

**The X-rule (the mental trick that unstuck me):**
*an action over X is not X.* "Washing dishes" is work WITH dishes, but
"washing dishes" is not a plate. "Decompose the wish-list" is work WITH
requirements, but it is not a requirement.

### Step 2a — For statements: the six-question ladder

Run top-down, first hit wins:

| # | Question | If YES → |
|---|---|---|
| 1 | WHY does the org need this at all? | **Business requirement** |
| 2 | Is it a task of some user class? | **User requirement** |
| 3 | What the system does ("shall…")? | **Functional requirement** |
| 4 | Does it exist OUTSIDE the system (enforceable by hand, in a notebook)? | **Business rule** |
| 5 | Is it the environment/platform the system must live in? | **Constraint / external interface** |
| 6 | Is it about HOW WELL the system behaves? | **Quality attribute (NFR)** |

Plus one project-side branch: a *one-off* condition of rolling the product out
(migration, training, docs, hardware) → **project requirement** (transition
requirements live here, never in the SRS).

### Step 2b — For activities: the two independent axes

An activity is classified on **two axes that do NOT constrain each other**:

- **Axis A — is the deliverable a requirement?**
  product requirement / project requirement / neither (pure process action).
- **Axis B — is the object of the work REQUIREMENTS?**
  *development* (name the subprocess: elicitation / analysis / specification /
  validation) / *management* / «—» (object is code, video, hardware, demo…).

**"Neither" on A does NOT force «—» on B.** Watching admins work is not a
requirement (A: neither) but IS elicitation (B: development). Fixing the
agreed release list as unchangeable is not a requirement (A: neither) but IS
baseline-setting (B: management). Conversely, a migration script IS a project
requirement (A: project) while writing its code is «—» on B (object = code).

### Analysis vs impact assessment (the trap that cost me two attempts)

| **Analysis** (development) | **Impact assessment** (management) |
|---|---|
| works on the **content** of *forming* requirements: separate tasks from FRs, decompose, prioritise | works on **changing the agreed baseline**: what breaks, what it costs |
| product: structured/refined requirements | product: a consequence estimate for a change decision |
| requirements are still being born | requirements are already fixed; someone proposes to touch them |

One-line marker: **"calculate what breaks if we change X, where X is already
agreed" → management. "Take the raw pile apart and structure it" → analysis.**

### Worked examples (Exercise 2, variant B)

| Item | A | B |
|---|---|---|
| one-off export script from 1C | project (transition) | «—» (object = code) |
| agree the release list as an unchangeable basis | neither (action) | management (baseline) |
| put an FR into the SRS | product (FR) | development / specification |
| demo the current version to the board | neither | «—» (object = product demo) |
| go watch admins run the paper journal | neither | development / elicitation |
| calculate what breaks if the booking window goes 7→30 days | neither | management (impact) |
| record a training video for admins | project (user docs/training) | «—» (object = video) |
| sort the week's wishes: tasks vs solutions vs policies | neither | development / analysis |

---

## References

- Course: `ba-dev-trainer/` repo, chapter 1.1 (full package in
  `chapters/chapter-1-1-requirements-foundations/`).
- Book: `book/chapters/ch01.pdf` (book pp. 2–25, PDF pp. 21–44, offset +19).
- Chapter 1.1 fully covered (Modules 1–6); the "Reading" step is closed. Next
  by protocol: exercises → project → skill-check → acceptance.
- Next course chapter: 1.2 "Customer and the expectation gap".
