# Wave 1 — Discovery

> Canonical detailed description for Wave 1 of the [[Requirements-Engineering-Roadmap|Requirements Engineering Toolkit]].

**Status:** Not started
**Window:** Week 1-2 (Jul 4 – Jul 17, 2026)
**Budget:** ~12h (5h study + 6h cases + ~2h distillation)

---

## Goal
On exit you can:
- Run a 30-min elicitation interview while staying in problem space.
- Catch yourself in solution mode **in the moment**.
- Kick off an Event Storming Big Picture solo on an unfamiliar domain.

---

## Topics to master
1. Problem space vs solution space — recognize solution-in-disguise.
2. Question taxonomy: open / closed / clarifying / probing / funnel. Trigger for each.
3. Active listening: paraphrase, mirroring, silence, summarizing.
4. Stakeholder analysis: Power/Interest grid; shadow stakeholders; decision-maker vs influencer vs end-user.
5. Event Storming Big Picture: domain event (past-tense, business-meaningful), hotspot, timeline, actor.
6. Event Storming Process Level: command, policy, aggregate hint.
7. Events vs commands vs read models — never confuse them.

---

## Study materials (~5h core)

| Source | What to read | Time | Mark |
|---|---|---|---|
| Wiegers, *Software Requirements* 2/e | Stakeholder identification + elicitation techniques ("voice of the customer" chapters) | 2h | CANON |
| Wiegers, *Software Requirements* 2/e | Vision & Scope chapter | 45m | CANON |
| Brandolini, *Introducing Event Storming* | Part 1: Big Picture Event Storming | 2h | CANON |
| eventstorming.com primer | "What is Event Storming?" | 30m | CANON |
| Martin Fowler bliki — "EventStorming" (by Brandolini) | full post | 20m | CANON |

**Reference (consult, don't read cover to cover):**
- BABOK v3 — "Interviews" technique, Elicitation knowledge area.
- Eric Evans, *DDD* — Chapter 2 "Communication and the Use of Language".

**Optional:**
- Robertson & Robertson, *Mastering the Requirements Process* — Volere stakeholder analysis section.

---

## Preparation — Subagent `The Stakeholder` (builds BEFORE cases)

The subagent is **infrastructure** for the learning process, not a learning product. It must exist before Case 1 so role-play is possible. Build it first, improve it iteratively as cases expose weaknesses.

**Inputs (runtime):**
- Domain + role.
- Knowledge level (expert / casual / hostile).
- Hidden constraints to plant (3-5 facts the user must dig out).

**Behavior contract (hard mode):**
- Talks in vague terms until probed.
- Contradicts self if user doesn't catch.
- Hides key constraints until ≥ 2 follow-ups.
- Responds to direct leading questions with "I think so?" not facts.
- Does NOT reveal hidden items on first mention.

**Acceptance criteria — pass when:**
- [ ] Withholds ≥ 80% of hidden facts unless explicitly probed with right angle.
- [ ] Behaves differently for different personas (resistant ≠ vision-rich).
- [ ] Calibration test: 5 users, all must surface different facts.

**File output:**
```
work/ba-ai-toolkit/subagents/the-stakeholder/
  persona.md
  archetypes/
    resistant-expert.md
    visionary-founder.md
    ops-storyteller.md
  README.md
```

---

## Cases (3 × ~2h = 6h)
1. **Telemedicine — elderly GP resistant to new tools.** Trains: probing, silence, recognizing hidden anxieties.
2. **P2P lending founder — big vision, no details.** Trains: funnel questions, structured interview, vision extraction.
3. **Coffee chain inventory — ops person narrating in stories.** Trains: extracting events from narrative → bridge to Event Storming (dual-purpose: elicitation + first storming).

Case briefs and replays live in this folder as they are produced.

---

## Distillation contract — Wave 1

### Skill `requirements-intake`

**Inputs:**
- Stakeholder persona (role, background, attitude, hidden constraints).
- Project brief (domain, problem statement, scope hint).
- Session goal (what to extract: vision / pain points / process / constraints).

**Outputs (artifacts):**
- Pre-interview plan with question bank grouped by category.
- Assumptions register (the #1 output).
- Requirements draft (functional, NFRs as stated, business rules).
- Open questions for follow-up.

**Acceptance criteria — pass when:**
- [ ] Produces ≥ 5 probing questions per stated high-level requirement.
- [ ] Question bank adapts to stakeholder archetype (resistant → empathy-first; vision-rich → funnel).
- [ ] Flags ≥ 3 hidden assumptions per session.
- [ ] Distinguishes problem-space vs solution-mode statements in transcript.
- [ ] Does NOT propose features.

**File output:**
```
work/ba-ai-toolkit/skills/requirements-intake/
  SKILL.md
  assets/
    question-bank.md
    assumptions-template.md
    interview-plan.md
  examples/
    telemedicine-output.md
    p2p-lending-output.md
    coffee-chain-output.md
  README.md
```

### Skill `event-storming-facilitator`

**Inputs:**
- Domain brief.
- Storming level (Big Picture / Process / Design).
- Optional: list of actors / stakeholders.

**Outputs:**
- Timeline of domain events (past tense, business-meaningful).
- Hotspots (open questions, frictions) with owner.
- Commands (user intentions) and policies (if Process level).
- Initial bounded-context candidates (grouping by semantic affinity).
- Action items / follow-up questions.

**Acceptance criteria — pass when:**
- [ ] Produces ≥ 15 events for a non-trivial domain.
- [ ] Events are past tense + business-meaningful (reject "User clicked button").
- [ ] Hotspots are specific, not generic.
- [ ] Distinguishes events / commands / read models correctly.
- [ ] Surfaces ≥ 1 implicit domain-expert question that the brief did not contain.

**File output:**
```
work/ba-ai-toolkit/skills/event-storming-facilitator/
  SKILL.md
  assets/
    event-checklist.md
    hotspot-template.md
  examples/
    telemedicine-storm.md
    p2p-lending-storm.md
    coffee-chain-storm.md
  README.md
```

---

## Wave 1 progress
- [ ] Study materials read
- [ ] **Subagent: The Stakeholder (preparation, before cases)**
- [ ] Case 1: Telemedicine intake
- [ ] Case 2: P2P lending founder
- [ ] Case 3: Coffee chain inventory
- [ ] Skill: `requirements-intake`
- [ ] Skill: `event-storming-facilitator`

Case artifacts and replay analyses accumulate in this folder.
