---
status: researching
mastery: beginner
tags: [business-analysis, requirements-engineering, ai-toolkit, roadmap, in-progress]
created: 2026-07-04
updated: 2026-07-04
source: personal learning plan
---

# 🧭 Requirements Engineering → AI Toolkit Roadmap

> [!abstract] Executive Summary
> A 2-month learning track (~48h, 6h/week) with a **dual deliverable**: (1) personal competence in requirements engineering for a Senior Full-Stack engineer, and (2) a public **AI toolkit** of 6 procedural skills + 3 role-based subagents, portable across Claude Code / LangGraph / n8n. Target: marketable as **Senior Full-Stack Engineer with BA + AI-tooling competencies** (EU + CIS market).

---

## 🎯 The "Why"

### Two hares, one shot
- **Hare 1 — Resume positioning.** Move beyond "full-stack dev" to "engineer who understands business." Concrete techniques (Event Storming, BDD, DDD Context Mapping) sell; "I gather requirements" does not.
- **Hare 2 — AI tooling trend.** Build a portfolio of AI agents for the requirements lifecycle. This is a rare cross-section (BA ∩ AI-dev) with few competitors.
- **Meta benefit.** Learning by coding = highest retention. Distilling a technique into a `SKILL.md` forces complete understanding.

### Hard rules (agreed with user)
1. **No certificate chase.** Not aiming for pure BA role. IREB CPRE only if a specific EU employer requires it.
2. **Quality > quantity.** 6 skills cap for the 2-month window. No cramming.
3. **Practice before code.** Each skill is distilled only after the technique is stable across 2-3 cases. Never code a skill after case 1.
4. **Hard mode in role-play.** Stakeholder subagent is explicitly permitted to be vague, contradictory, evasive, and to hide assumptions. No cheerleading.
5. **No BABOK/Volere deep dive.** They are reference, not core. Focus on techniques that ship in modern dev teams.

---

## 🏗️ Course Architecture — 4 Waves

```
Wave 1 (wk 1-2): Discovery           → skills 1, 2  + subagent: The Stakeholder
Wave 2 (wk 3-4): Structuring         → skills 3, 4  + subagent: The Devil's Advocate
Wave 3 (wk 5-6): Domain & Quality    → skills 5, 6  + subagent: The Tech Lead
Wave 4 (wk 7-8): Integration & Portfolio  → orchestrator + adapter + capstone
```

Each wave: **2-3 cases → distillation → 1-2 skills + 1 subagent.**

---

## 📦 Toolkit Inventory (11 artifacts)

### Procedural Skills (6)

| # | Skill | Wave | What it does | Depends on |
|---|---|---|---|---|
| 1 | `requirements-intake` | 1 | Structured interview; question bank per stakeholder type; surfaces assumptions; outputs requirements draft | — |
| 2 | `event-storming-facilitator` | 1 | Runs solo/group session; outputs domain events, commands, hotspots, bounded-context hints | — |
| 3 | `story-mapper` | 2 | Flat backlog → Patton story map (backbone, slices, releases) | feeds from 1, 2 |
| 4 | `spec-by-example` | 2 | Story → rules + examples → executable Gherkin (bridge to BDD/tests) | feeds from 3 |
| 5 | `context-mapper` | 3 | DDD Context Map: contexts + relationships (ACL, conformist, shared kernel, customer-supplier…) | feeds from 2 |
| 6 | `nfr-elicitor` | 3 | Vague quality claims → SMART SLOs per ISO 25010 | independent, applied on top of architecture |

### Role-Based Subagents (3)

| Persona | Wave | Function |
|---|---|---|
| **The Stakeholder** | 1 | Simulated client/domain expert for interview practice. Hard mode: hides info, contradicts self. |
| **The Devil's Advocate** | 2 | Reads spec, hunts for assumptions, missing cases, unstated risks. |
| **The Tech Lead** | 3 | Challenges NFRs and architectural fit. Realistic senior pushback. |

### Integration Layer (2)

| Artifact | Wave | Purpose |
|---|---|---|
| **Orchestrator + harness adapter** | 4 | Pipeline of skills; same knowledge layer, thin adapter per runtime (Claude Code / LangGraph / n8n) |
| **Capstone demo + portfolio packaging** | 4 | End-to-end run of one fresh case through all 6 skills; README + architecture diagram + writeup |

---

## 🔁 Practice Loop Procedure

The standard rhythm for every technique. (This procedure itself becomes a future meta-skill: `practice-loop-facilitator`.)

| Stage | Time | What happens |
|---|---|---|
| **0. Brief** | ~5 min | Agent issues project brief + stakeholder personas. **With hidden assumptions** to be surfaced. |
| **1. Study (homework)** | ~1-2h (off-session) | User reads the technique material before role-play. |
| **2. Role-play** | ~30-45 min | Agent plays roles (stakeholder, saboteur, dev). **Does not play along.** |
| **3. Replay analysis** | ~15 min | Agent critiques against explicit rubric: what was surfaced, missed, where solution-mode crept in. **No praise.** |
| **4. Second iteration** (opt.) | ~30 min | Same technique, **different domain**. Tests skill transfer. |
| **5. Distillation** (once per 2-3 topics) | ~60 min | When the technique is stable, write the skill/subagent. |

> [!warning] Learning happens at Stage 3 (analysis), not Stage 2 (role-play). Skipping analysis = wasted session.

---

## 🌊 Wave 1 — Discovery (wk 1-2, 12h)

### Goal
On exit you can:
- Run a 30-min elicitation interview while staying in problem space.
- Catch yourself in solution mode **in the moment**.
- Kick off an Event Storming Big Picture solo on an unfamiliar domain.

### Topics to master
1. Problem space vs solution space — recognize solution-in-disguise.
2. Question taxonomy: open / closed / clarifying / probing / funnel. Trigger for each.
3. Active listening: paraphrase, mirroring, silence, summarizing.
4. Stakeholder analysis: Power/Interest grid; shadow stakeholders; decision-maker vs influencer vs end-user.
5. Event Storming Big Picture: domain event (past-tense, business-meaningful), hotspot, timeline, actor.
6. Event Storming Process Level: command, policy, aggregate hint.
7. Events vs commands vs read models — never confuse them.

### Study materials (~5h core)

| Source | What to read | Time | Mark |
|---|---|---|---|
| Wiegers, *Software Requirements* 2/e | Stakeholder identification + elicitation techniques ("voice of the customer" chapters) | 2h | CANON |
| Wiegers, *Software Requirements* 2/e | Vision & Scope chapter | 45m | CANON |
| Brandolini, *Introducing Event Storming* | Part 1: Big Picture Event Storming | 2h | CANON |
| eventstorming.com primer | "What is Event Storming?" | 30m | CANON |
| Martin Fowler bliki — "EventStorming" (by Brandolini) | full post | 20m | CANON |

**Reference (consult, don't read cover to cover):**
- BABOK v3 — "Interviews" technique, Elicitation knowledge area. [SYNTHESIS — exact section number]
- Eric Evans, *DDD* — Chapter 2 "Communication and the Use of Language".

**Optional:**
- Robertson & Robertson, *Mastering the Requirements Process* — Volere stakeholder analysis section.

### Cases (3 × ~2h = 6h)
1. **Telemedicine — elderly GP resistant to new tools.** Trains: probing, silence, recognizing hidden anxieties.
2. **P2P lending founder — big vision, no details.** Trains: funnel questions, structured interview, vision extraction.
3. **Coffee chain inventory — ops person narrating in stories.** Trains: extracting events from narrative → bridge to Event Storming (dual-purpose: elicitation + first storming).

### Distillation contract — Wave 1

#### Skill `requirements-intake`

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
    question-bank.md          # templates per stakeholder archetype
    assumptions-template.md
    interview-plan.md
  examples/
    telemedicine-output.md
    p2p-lending-output.md
    coffee-chain-output.md
  README.md
```

#### Skill `event-storming-facilitator`

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
- [ ] Hotspots are specific, not generic ("Does billing happen before or after shipping?" — yes; "Need more info" — no).
- [ ] Distinguishes events / commands / read models correctly.
- [ ] Surfaces ≥ 1 implicit domain-expert question that the brief did not contain.

**File output:**
```
work/ba-ai-toolkit/skills/event-storming-facilitator/
  SKILL.md
  assets/
    event-checklist.md        # what makes a good event
    hotspot-template.md
  examples/
    telemedicine-storm.md
    p2p-lending-storm.md
    coffee-chain-storm.md
  README.md
```

#### Subagent `The Stakeholder`

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
  persona.md                  # system prompt + behavior rules
  archetypes/
    resistant-expert.md
    visionary-founder.md
    ops-storyteller.md
  README.md
```

---

## 🌊 Wave 2 — Structuring & Specification (wk 3-4, 12h)

### Goal
On exit you can:
- Turn a raw requirements draft into a Patton story map.
- Write executable specifications (Gherkin) from stories via Example Mapping.
- Tell the difference between a story that exposes business behavior and one that hides it.

### Topics to master
1. User Stories + INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable).
2. Anti-patterns: stories as tasks, too-large stories, disguised solutions.
3. Job Stories (JTBD): "When [situation], I want [motivation], so I can [outcome]".
4. **User Story Mapping**: backbone (activities/journeys), steps (slices), releases (horizontal).
5. Acceptance Criteria: Given-When-Then, Gherkin (Feature / Scenario / Background).
6. **Example Mapping** (Matt Wynne): story → rules → examples → questions (4 colors).
7. BDD: requirements as executable specs, shared language across dev/QA/business.
8. Declarative vs imperative scenarios — the #1 Gherkin sin.

### Study materials (~5h core)

| Source | What to read | Time | Mark |
|---|---|---|---|
| Jeff Patton, *User Story Mapping* | Parts I-II (the whole foundation) | 2.5h | CANON |
| Matt Wynne, *BDD in Action* (or his Example Mapping blog post on cucumber.blog) | Example Mapping technique | 1h | CANON |
| Gojko Adzic, *Specification by Example* | Ch. 1-3 (the why and the process) | 1.5h | CANON |

**Reference:**
- Wiegers, *Software Requirements* — chapters on user stories & acceptance criteria.
- cucumber.io docs — Gherkin reference.

**Optional:**
- Alistair Cockburn, *Writing Effective Use Cases* — use case levels (Cloud/Kite/Sea/Eel/Clam).

### Cases (3 × ~2h = 6h)
1. **Ed-tech SaaS — student onboarding flow as story map.** Greenfield; multiple user types (student, teacher, admin). Trains: backbone thinking, slicing by user journey.
2. **Food delivery — courier cancels mid-delivery.** Branching business logic: refunds, customer notification, courier reassignment, SLA breaches. Trains: example mapping on complex rules.
3. **HR tooling — leave request workflow (brownfield).** Existing process with edge cases nobody remembers (probation, half-days, cross-year). Trains: extracting examples from messy reality.

### Distillation contract — Wave 2

#### Skill `story-mapper`

**Inputs:**
- Requirements draft (from `requirements-intake`).
- Persona list (user types).
- Project goal / business outcome.

**Outputs:**
- Story map: backbone (user activities/journeys) → steps → slices.
- Release slices mapped to MVP → V1 → V2.
- Identified gaps / missing personas.
- Anti-pattern flags (where stories violate INVEST).

**Acceptance criteria — pass when:**
- [ ] Backbone is expressed as user activities, not features.
- [ ] Each slice is INVEST-compliant.
- [ ] Releases are sliced by business value, not technical layers.
- [ ] Surfaces ≥ 1 missing persona or activity per map.

**File output:**
```
work/ba-ai-toolkit/skills/story-mapper/
  SKILL.md
  assets/
    invest-checklist.md
    backbone-template.md
  examples/
    edtech-onboarding-map.md
    food-delivery-map.md
    hr-leave-map.md
  README.md
```

#### Skill `spec-by-example`

**Inputs:**
- Story card (one user story).
- Domain rules (from stakeholder or SME).
- Optional: existing test cases.

**Outputs:**
- Example map: rules (blue), examples (green), questions (yellow), constraints (red).
- Gherkin feature file with scenarios derived from examples.
- Open questions list for next stakeholder session.

**Acceptance criteria — pass when:**
- [ ] Each rule has ≥ 1 example; complex rules have positive + negative examples.
- [ ] Gherkin scenarios are declarative (no UI clicks), business-readable.
- [ ] Examples are concrete (with specific values), not abstract.
- [ ] Questions list captures what's unknown, not what's lazy.

**File output:**
```
work/ba-ai-toolkit/skills/spec-by-example/
  SKILL.md
  assets/
    gherkin-style-guide.md
    example-map-template.md
  examples/
    edtech-enrollment.feature
    food-delivery-courier-cancel.feature
    hr-leave-request.feature
  README.md
```

#### Subagent `The Devil's Advocate`

**Inputs:**
- Specification document (story map, Gherkin, requirements list).
- Domain context.

**Behavior contract (hard mode):**
- Searches for unstated assumptions ("You assume courier is always reachable — what if offline?").
- Identifies missing scenarios (edge cases, error paths, concurrent actions).
- Challenges ambiguous language ("'fast' — what does that mean here?").
- Flags contradictions between rules.
- Does NOT propose solutions — only surfaces gaps.

**Acceptance criteria — pass when:**
- [ ] Surfaces ≥ 5 assumptions per spec page.
- [ ] Identifies ≥ 3 missing scenarios per non-trivial story.
- [ ] All findings categorized: assumption / missing case / ambiguity / contradiction.

**File output:**
```
work/ba-ai-toolkit/subagents/the-devils-advocate/
  persona.md
  checklists/
    assumptions-audit.md
    edge-case-prompts.md
  examples/
    review-food-delivery.md
    review-hr-leave.md
    review-edtech.md
  README.md
```

---

## 🌊 Wave 3 — Domain & Quality (wk 5-6, 12h)

### Goal
On exit you can:
- Extract bounded contexts and their relationships from a storm/model.
- Turn vague quality claims into measurable SLOs.
- Reason about consistency / availability / latency trade-offs without hand-waving.

### Topics to master
1. DDD strategic: domain vs subdomain vs bounded context — the differences that matter.
2. Subdomain types: core / supporting / generic.
3. Ubiquitous Language: why it matters, how to form it.
4. **Context Mapping**: 9 relationship types (customer-supplier, conformist, ACL, shared kernel, published language, open-host service, partners, big ball of mud, separated ways).
5. Domain Storytelling (Eber/Widmann) — elicitation technique complementing Event Storming.
6. **ISO/IEC 25010** quality model: 8 characteristics.
7. SMART-ifying NFRs: "fast" → p99 < 200ms under 1000 RPS.
8. SLO/SLI/SLA: engineering language for NFRs.
9. Trade-off thinking: consistency / availability / latency / cost.

### Study materials (~5h core)

| Source | What to read | Time | Mark |
|---|---|---|---|
| Vaughn Vernon, *Implementing DDD* | Ch. 2 "Domains, Subdomains, and Bounded Contexts" + Ch. 3 "Context Mapping" | 2.5h | CANON |
| Eric Evans, *DDD* | Ch. 14 "Maintaining Model Integrity" (the context mapping origin) | 1.5h | CANON |
| Google SRE Book, Ch. 4 "SLO/SLI" | full chapter, free online | 1h | CANON |

**Reference:**
- ISO/IEC 25010 specification (official quality model) — the 8 characteristics.
- Nick Tune / Scott Wlaschin — Context Mapping patterns blog posts.

**Optional:**
- Brandolini, *Introducing Event Storming* — Design Level storming part.
- domainstorytelling.org — Domain Storytelling primer.

### Cases (3 × ~2h = 6h)
1. **Smart home / IoT — many contexts, few people.** Device management, user, scene automation, billing, telemetry. Trains: identifying contexts at scale; shared kernel vs ACL decisions.
2. **Legacy monolith migration — brownfield DDD.** Extract contexts from an existing e-commerce monolith. Trains: recognizing "big ball of mud" and seams for extraction.
3. **High-load real-time (food delivery tracking).** Trains: NFR elicitation under stress; latency / throughput / consistency trade-offs.

### Distillation contract — Wave 3

#### Skill `context-mapper`

**Inputs:**
- Domain events from `event-storming-facilitator`.
- Project brief.
- Organization structure (teams, existing systems).

**Outputs:**
- Bounded context inventory with names + ubiquitous language hints.
- Context map (visual + table) with relationship types.
- Risk flags: where ACL needed, where shared kernel is tempting but dangerous.

**Acceptance criteria — pass when:**
- [ ] Identifies ≥ 4 bounded contexts for non-trivial domain.
- [ ] Each relationship labeled with one of the 9 patterns + rationale.
- [ ] Surfaces ≥ 1 "big ball of mud" risk area.
- [ ] Names contexts using ubiquitous language, not technical jargon ("Ordering" not "OrderService").

**File output:**
```
work/ba-ai-toolkit/skills/context-mapper/
  SKILL.md
  assets/
    context-map-templates.md     # the 9 patterns with decision guide
    ubiq-language-template.md
  examples/
    smart-home-context-map.md
    ecommerce-monolith-map.md
    food-delivery-context-map.md
  README.md
```

#### Skill `nfr-elicitor`

**Inputs:**
- Stakeholder statements ("the system should be fast", "secure", "always available").
- Architecture summary or constraints.
- Domain context (real-time? batch? regulated?).

**Outputs:**
- NFR table classified by ISO 25010 category.
- Each NFR as: attribute / stimulus / response measure / target / trade-off note.
- SLO/SLI candidates for engineering teams.

**Acceptance criteria — pass when:**
- [ ] Maps every vague quality claim to ≥ 1 ISO 25010 category.
- [ ] Every NFR has a measurable target (no "fast" without a latency number).
- [ ] Flags ≥ 2 trade-offs per non-trivial architecture (e.g., consistency vs availability).
- [ ] Distinguishes hard constraints (regulatory) from soft targets (UX).

**File output:**
```
work/ba-ai-toolkit/skills/nfr-elicitor/
  SKILL.md
  assets/
    iso-25010-cheatsheet.md
    slo-template.md
  examples/
    food-delivery-nfrs.md
    iot-nfrs.md
    ecommerce-nfrs.md
  README.md
```

#### Subagent `The Tech Lead`

**Inputs:**
- NFR set + architecture sketch.
- Context map.
- Project constraints (team size, timeline, budget).

**Behavior contract (hard mode):**
- Challenges NFR targets: "p99 < 200ms — at what load? at what cost?"
- Questions architecture choices: "Why microservices here? What's the team size?"
- Pushes back on missing trade-offs.
- Demands measurement strategy, not just claims.
- Cites realistic constraints: team size, time-to-market, operational burden.

**Acceptance criteria — pass when:**
- [ ] Challenges ≥ 3 NFR targets per review.
- [ ] Surfaces ≥ 1 cost / operational concern per architecture.
- [ ] Asks "how will you measure this?" for every claim.

**File output:**
```
work/ba-ai-toolkit/subagents/the-tech-lead/
  persona.md
  checklists/
    nfr-challenge-questions.md
    architecture-fit-questions.md
  examples/
    review-smart-home.md
    review-monolith-migration.md
    review-food-delivery.md
  README.md
```

---

## 🌊 Wave 4 — Integration & Portfolio (wk 7-8, 12h)

### Goal
On exit you can:
- Wire 6 skills into a coherent pipeline.
- Reuse the same knowledge layer across 2+ harnesses with thin adapters.
- Hand the toolkit to a stranger and they understand it in 10 minutes.

### Topics to master
1. Orchestrator patterns: linear pipeline, fan-out/fan-in, human-in-the-loop checkpoints.
2. Skill invocation protocol (input/output contract between skills).
3. Harness portability: markdown knowledge layer vs runtime-specific glue.
4. Capstone demo: end-to-end run on a fresh project.
5. Technical writing for portfolio: README, architecture diagram, design writeup.

### Study materials (~3h core)

| Source | What to read | Time | Mark |
|---|---|---|---|
| Claude Code `SKILL.md` authoring guide (official docs) | full | 1h | CANON |
| LangGraph tutorial — basic agent orchestration | quickstart | 1h | CANON |
| Michael Nygard — "ADR" original paper + ThoughtWorks ADR template | canonical primer | 30m | CANON |
| n8n AI Agent node documentation | reference | 30m | CANON |

**Reference:**
- Existing public skill collections (Anthropic, community) — study structure, not copy.

### Distillation contract — Wave 4

#### Orchestrator + Adapter

**Inputs:**
- All 6 skills + 3 subagents.
- Choice of harnesses (Claude Code primary; LangGraph + n8n secondary).

**Outputs:**
- Pipeline definition: skill invocation order + data flow.
- Adapter pattern doc: shared knowledge layer + harness-specific wrappers.
- Demo run on capstone case.

**Acceptance criteria — pass when:**
- [ ] Pipeline runs end-to-end on a fresh domain.
- [ ] Same knowledge layer (markdown) reused across ≥ 2 harnesses without content changes.
- [ ] Adapter per runtime ≤ 50 LOC of glue.
- [ ] Documented invocation protocol (input/output schema between skills).

**File output:**
```
work/ba-ai-toolkit/
  pipeline.md                   # the directed graph + data flow
  adapters/
    claude-code/                # skill manifests + .claude config
    langgraph/
      graph.py
      nodes/                    # each wraps a SKILL.md
    n8n/
      workflows/
  examples/
    capstone-run/               # end-to-end artifacts from one fresh project
      01-intake.md
      02-storming.md
      03-story-map.md
      04-specs.feature
      05-context-map.md
      06-nfrs.md
```

#### Portfolio packaging

**Acceptance criteria — pass when:**
- [ ] README explains: what / why / how to use / how to extend.
- [ ] Architecture diagram shows pipeline + adapter pattern.
- [ ] Capstone run has 6 artifacts demonstrating the full lifecycle.
- [ ] Writeup explains portability claim with concrete example.

---

## 📐 Toolkit Architecture (Pipeline View)

```
[requirements-intake] ──┐
                        ├──→ [story-mapper] ──→ [spec-by-example] ──→ tests
[event-storming] ───────┤
                        └──→ [context-mapper] ──→ architecture

[nfr-elicitor] ─────────────────────────────────────→ SLOs, ADRs
```

Skills are **not an independent set** but a **directed graph**. This is itself a resume point: "designed an AI-agent pipeline for the requirements lifecycle."

### Portability contract
- **Portable:** prompts, procedures, knowledge, role definitions (text/markdown).
- **Non-portable:** tool integration, state, orchestration (rewritten per runtime).

Architecture: **harness-agnostic knowledge layer + thin adapter per runtime.**

---

## 🗓️ Schedule (2026-07-04 → ~2026-08-28)

| Week | Wave | Cases | Skills built | Subagent |
|---|---|---|---|---|
| 1 (Jul 4-10) | W1 Discovery | Telemedicine, P2P lending | — | — |
| 2 (Jul 11-17) | W1 Discovery | Coffee chain | `requirements-intake`, `event-storming-facilitator` | The Stakeholder |
| 3 (Jul 18-24) | W2 Structuring | Ed-tech, Food delivery | — | — |
| 4 (Jul 25-31) | W2 Structuring | HR tooling | `story-mapper`, `spec-by-example` | The Devil's Advocate |
| 5 (Aug 1-7) | W3 Domain & Quality | Smart home, Legacy migration | — | — |
| 6 (Aug 8-14) | W3 Domain & Quality | High-load real-time | `context-mapper`, `nfr-elicitor` | The Tech Lead |
| 7 (Aug 15-21) | W4 Integration | Capstone case | Orchestrator + adapter | — |
| 8 (Aug 22-28) | W4 Portfolio | (packaging) | README, demos, diagram | — |

---

## 🚫 Explicitly Out of Scope (2-month constraint)

To avoid scope creep, these are **deliberately excluded**:
- BPMN deep (only reading, not drawing).
- Use Cases Cockburn in detail (touched in W2, not flagship).
- ATAM and formal architecture methods (this is system design territory).
- Decision Tables / State Machines as separate topics (mentioned only).
- Regulatory deep-dives (GDPR/HIPAA) — only as a factor inside cases.
- BABOK/Volere as study material — reference only.

---

## 📊 Progress Tracker

### Wave 1 — Discovery
- [ ] Study materials read
- [ ] Case 1: Telemedicine intake
- [ ] Case 2: P2P lending founder
- [ ] Case 3: Coffee chain inventory
- [ ] Skill: `requirements-intake`
- [ ] Skill: `event-storming-facilitator`
- [ ] Subagent: The Stakeholder

### Wave 2 — Structuring
- [ ] Study materials read
- [ ] Case 4: Ed-tech onboarding
- [ ] Case 5: Food delivery courier-cancel
- [ ] Case 6: HR leave request (brownfield)
- [ ] Skill: `story-mapper`
- [ ] Skill: `spec-by-example`
- [ ] Subagent: The Devil's Advocate

### Wave 3 — Domain & Quality
- [ ] Study materials read
- [ ] Case 7: Smart home contexts
- [ ] Case 8: Legacy monolith migration
- [ ] Case 9: High-load real-time NFRs
- [ ] Skill: `context-mapper`
- [ ] Skill: `nfr-elicitor`
- [ ] Subagent: The Tech Lead

### Wave 4 — Integration & Portfolio
- [ ] Orchestrator design
- [ ] Harness adapter (Claude Code → LangGraph → n8n)
- [ ] Capstone case (end-to-end)
- [ ] Toolkit README
- [ ] 2-3 demo artifacts
- [ ] Architecture diagram + writeup

---

## 📚 Source Material (priority order)

**Canonical, read in original:**
1. Karl Wiegers — *Software Requirements* 2/e (foundation, easy read).
2. Jeff Patton — *User Story Mapping* (changes backlog thinking).
3. Gojko Adzic — *Specification by Example* + *Impact Mapping* (modern view).
4. Vaughn Vernon — *Implementing DDD* (Ch. 2-3 for Wave 3).
5. Eric Evans — *DDD* (Ch. 14 for context mapping origin).
6. Google SRE Book — Ch. 4 (for SLO/SLI language).

**Reference (consult, do not read cover to cover):**
- BABOK Guide v3.
- Robertson & Robertson — *Mastering the Requirements Process* (Volere).
- Alistair Cockburn — *Writing Effective Use Cases*.

**Context / mindset:**
- Don Norman — *The Design of Everyday Things* (JTBD thinking).

> [!info] Source-marking convention
> When the agent (me) outputs module material, it marks each claim as either:
> - **[CANON]** — well-established, confident.
> - **[SYNTHESIS]** — agent's synthesis from general knowledge; verify against source.

---

## 🔗 Related Notes
- **Parent:** [[Business Analysis/README|Business Analysis README]]
- **Future links:** individual skill notes (`requirements-intake.md`, etc.) as they are built.
- **Code:** `work/ba-ai-toolkit/` (separate working directory for the toolkit repo).

---

## ❓ Active Recall

> [!faq]- Why practice before coding the skill?
> A skill encodes **stable competence**. Coding after case 1 freezes a clumsy first-attempt version. 2-3 cases reveal the invariant pattern worth distilling.

> [!faq]- Why varied domains instead of one capstone project?
> Varied practice (cognitive science) extracts the **invariant** across contexts — this is what transfers to any future real project. Single-project practice trains the project, not the skill.

> [!faq]- Why is the analysis stage more important than the role-play?
> Stage 2 (role-play) is performance; stage 3 (analysis) is where the gap between intent and execution becomes visible. Skipping analysis = no learning signal.

> [!faq]- Why are acceptance criteria checklists, not tests?
> A skill's quality is judged by **what it produces against a rubric**, not by unit tests. The checklist is the acceptance contract — if the artifact fails the checklist, the skill is not done.

---

*Last updated: 2026-07-04 — full per-wave distillation contracts added.*
