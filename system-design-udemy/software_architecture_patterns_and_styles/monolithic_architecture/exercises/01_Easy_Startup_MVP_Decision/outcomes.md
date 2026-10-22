---
status: archived
artifact_type: session_outcomes
exercise: "[[01_Easy_Startup_MVP_Decision]]"
pattern: monolithic
difficulty: beginner
session_date: 2026-07-16
verdict: solid_pass
created: 2026-07-16
updated: 2026-10-21
---

# Outcomes — "We Should Be Like Netflix" Startup

> Self-assessment artifact from the interview simulation. Pins what stuck, what didn't, and where to dig next.

## TL;DR

**Verdict: solid pass.** Came in with the right instincts, defended them under Socratic pressure, and left nothing major on the floor. Three genuine strengths; three concrete gaps to drill before a real interview.

---

## What I nailed ✓

| Rubric item | Evidence |
|---|---|
| **Gathered requirements first** | Asked about scale (DAU) *before* sketching boxes — textbook reflex. |
| **Recommended *modular* monolith** | Did not hand-wave "monolith." Went straight to **modular monolith + DDD + bounded context = module**. The seniority signal. |
| **Recognized cargo-cult engineering** | Named the real driver: *org scale, not traffic* — "hundreds of teams, independent planning, deadlines, releases." This is the #1 thing interviewers listen for. |
| **"Monolith first" reasoning** | Argued the principle from first principles even without the label: unknown domain boundaries → split → re-split → merge = pain. |
| **Exit criteria (full triad)** | Named three concrete extraction triggers, each with a *why*. Most candidates say "we'll start simple" and stop. |
| **Internal structure** | DDD bounded contexts as modules = pre-cut seams for future extraction. |

The standout insight: **"a feature used 1000× more than others should be split for independent scaling"** — surfaced unprompted, before the interviewer handed over the constraints. That's the seed of the entire exit-criteria discussion.

---

## Pain points & level-ups ✗

### 1. Skipped the *boring* DevOps taxes — named only the exotic ones

**What happened:** When asked to spell out the microservices tax, I jumped to the sophisticated costs — distributed transactions, consistency models, distributed tracing, network latency, attack surface. All correct, all senior-level. But I skipped the *everyday* taxes that actually eat small teams alive.

**The gap:** The boring taxes are what interviewers expect *first*, because they're what actually kills a 2-person team. Lead with them, then layer the exotic ones on top.

**The list I should have led with:**
- **CI/CD per service** — not one pipeline, one *per service*, each with its own build/test/deploy/rollback.
- **Service discovery & configuration** — services must find each other; config must live somewhere reachable. Infra I don't own today.
- **Environment parity** — spinning up realistic dev/staging for *N* services, not 1.

**Drill:** Next time the "microservices tax" question comes, force myself to name the three unglamorous taxes *before* reaching for distributed tracing.

---

### 2. Argued from reasoning, not evidence — cited zero anchors

**What happened:** Argued cleanly against cargo-culting Netflix but didn't drop a single company name or named principle.

**The gap:** Evidence converts a good answer into an unforgettable one. "Netflix's architecture is driven by org scale" is strong; "Basecamp, Shopify, and GitHub all run monoliths at far greater scale than we'll ever hit" is devastating.

**Anchors to memorize:**
- **Companies:** Basecamp, Shopify, GitHub (GitHub's modular-monolith Ruby rewrite is *the* canonical counterexample to "Netflix or bust").
- **Named principle:** **Martin Fowler's "MonolithFirst"** — *most successful microservice stories start with a monolith that got too painful, not a greenfield microservices system.*
- **Named law:** **Conway's Law** — I reasoned to it ("separate teams, separate services"); I should *name* it.

**Drill:** Build a one-line "companies running monoliths at scale" and a one-line "Fowler's MonolithFirst" muscle memory.

---

### 3. Used the right concepts — didn't always *name* them

**What happened:** I reasoned *to* several named principles (Conway's Law, monolith-first, bounded contexts) but only named DDD/bounded context explicitly. The reasoning was sound; the labels were missing.

**The gap:** Naming the principle shows you've read the literature, not just that you can reason. Interviewers can't give credit for a principle you implied but didn't name.

**Drill:** When I reason to a named concept, name it in the same breath. "Different teams, different release cadences → extract (this is **Conway's Law** in action)."

---

## Exit criteria triad (the answer I delivered)

Captured here so I can refine it:

1. **Independent scaling** of a hot subsystem (the 1000× insight).
2. **Team ownership** — a module gains a distinct owning team (Conway's Law).
3. **Independent release cadence** — different teams shipping the same module at different speeds, surfacing as deployability risk + velocity drag (merge conflicts, blocked PRs, coordination overhead).

> Test for #3: *when the cost of sharing a deployable exceeds the cost of splitting one, the seam cuts itself.*

This is a strong, complete set. Next exercise should stress-test *how* to extract once a trigger fires.

---

## Where to dig next

In rough order of payoff, given today's gaps:

### Immediate (plugs today's gaps)

| Artifact | Why | What to extract |
|---|---|---|
| **[[Monolithic Architecture]]** (concept note) | Formalizes the "monolith first" principle I reasoned to but didn't name; covers modular monolith variant + pros/cons. | The *label* for what I argued. |
| **[[Factors Influencing Architecture Selection]]** | Turns my intuition into a repeatable decision framework — the four factor families. | A checklist I can run under pressure instead of reasoning from scratch. |
| **[[Software Architecture]]** | Anti-patterns: cargo-cult, **distributed monolith** (define this — it's the natural sequel to this exercise). | Vocabulary for what goes wrong when microservices are done badly. |

### Natural next exercises (sequenced)

1. **[[01_Easy_Distributed_Monolith_Diagnosis]]** *(microservices, easy)* — **highest priority.** This exercise ended on "don't cargo-cult microservices." The immediate sequel is: *what does it look like when someone did, and how do you diagnose it?* It will also drill the boring DevOps taxes I skipped — they show up as symptoms there.
2. **[[02_Intermediate_Growing_Monolith_Pains]]** *(monolithic, intermediate)* — **direct stress-test of my exit criteria.** Same domain posture one step on: the monolith is 2 years old and hurting. I'd be forced to decide *which* of my three triggers has actually fired, and sequence the response. This is where my "modular monolith first, extract second" thesis gets put on trial.
3. **[[03_Advanced_Strangler_Fig_Extraction]]** *(monolithic, advanced)* — **the "how" my exit criteria only gestured at.** I named *when* to extract; this exercise forces me to design *how* — Strangler Fig, data decoupling, dual-run cutover. Staff-level. Save for after #2.

### Supporting concept notes worth a focused read

- **[[Conway's Law]]** — I used it correctly; worth formalizing so I name it instinctively.
- **[[DDD Bounded Context]]** — I invoked it correctly as "bounded context = module"; deepen so it recurs reliably in every architecture decision.
- **[[Microservices Challenges]]** — the exotic taxes I named (data consistency, distributed tracing) live here formally. Pair with the boring-tax list above.

---

## One-sentence takeaway

> Strong on reasoning and instinct; the gap is *vocabulary and evidence* — name the principles I reason to, cite the companies that prove the point, and lead with the boring DevOps taxes before the exotic ones.
