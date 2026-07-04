# Wave 2 — Structuring & Specification

> Canonical detailed description for Wave 2 of the [[Requirements-Engineering-Roadmap|Requirements Engineering Toolkit]].

**Status:** Not started
**Window:** Week 3-4 (Jul 18 – Jul 31, 2026)
**Budget:** ~12h (5h study + 6h cases + ~2h distillation)

---

## Goal
On exit you can:
- Turn a raw requirements draft into a Patton story map.
- Write executable specifications (Gherkin) from stories via Example Mapping.
- Tell the difference between a story that exposes business behavior and one that hides it.

---

## Topics to master
1. User Stories + INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable).
2. Anti-patterns: stories as tasks, too-large stories, disguised solutions.
3. Job Stories (JTBD): "When [situation], I want [motivation], so I can [outcome]".
4. **User Story Mapping**: backbone (activities/journeys), steps (slices), releases (horizontal).
5. Acceptance Criteria: Given-When-Then, Gherkin (Feature / Scenario / Background).
6. **Example Mapping** (Matt Wynne): story → rules → examples → questions (4 colors).
7. BDD: requirements as executable specs, shared language across dev/QA/business.
8. Declarative vs imperative scenarios — the #1 Gherkin sin.

---

## Study materials (~5h core)

| Source | What to read | Time | Mark |
|---|---|---|---|
| Jeff Patton, *User Story Mapping* | Parts I-II (the whole foundation) | 2.5h | CANON |
| Matt Wynne, *BDD in Action* (or Example Mapping blog post on cucumber.blog) | Example Mapping technique | 1h | CANON |
| Gojko Adzic, *Specification by Example* | Ch. 1-3 (the why and the process) | 1.5h | CANON |

**Reference:**
- Wiegers, *Software Requirements* — chapters on user stories & acceptance criteria.
- cucumber.io docs — Gherkin reference.

**Optional:**
- Alistair Cockburn, *Writing Effective Use Cases* — use case levels (Cloud/Kite/Sea/Eel/Clam).

---

## Preparation — Subagent `The Devil's Advocate` (builds BEFORE cases)

The subagent is **infrastructure** for the learning process, not a learning product. Build it first, improve it iteratively as cases expose weaknesses.

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

## Cases (3 × ~2h = 6h)
1. **Ed-tech SaaS — student onboarding flow as story map.** Greenfield; multiple user types (student, teacher, admin). Trains: backbone thinking, slicing by user journey.
2. **Food delivery — courier cancels mid-delivery.** Branching business logic: refunds, customer notification, courier reassignment, SLA breaches. Trains: example mapping on complex rules.
3. **HR tooling — leave request workflow (brownfield).** Existing process with edge cases nobody remembers (probation, half-days, cross-year). Trains: extracting examples from messy reality.

---

## Distillation contract — Wave 2

### Skill `story-mapper`

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

### Skill `spec-by-example`

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

---

## Wave 2 progress
- [ ] Study materials read
- [ ] **Subagent: The Devil's Advocate (preparation, before cases)**
- [ ] Case 4: Ed-tech onboarding
- [ ] Case 5: Food delivery courier-cancel
- [ ] Case 6: HR leave request (brownfield)
- [ ] Skill: `story-mapper`
- [ ] Skill: `spec-by-example`
