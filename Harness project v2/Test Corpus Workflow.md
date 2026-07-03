---
status: analyzing
tags: [agent-harness, workflow, test-corpus, validation, process]
created: 2026-06-09
updated: 2026-06-09
---

# Test Corpus Workflow — How to Validate the Skill

> Step-by-step process for using the 6 test repositories to build and validate the Agent Harness Skill.
>
> **Prerequisite:** [[Test Corpus Specification]] — the repos must exist in `agent-harness-test-corpus/`.

---

## 🎯 Big Picture

```
Corpus created ──► P2 Walking Skeleton ──► P1 Classifier ──► P3 Maturity Engine
     │                │ (repo #1 only)       │ (all 6 repos)     │ (all 6 repos)
     │                │                      │                   │
     │                ▼                      ▼                   ▼
     │           Signal or noise?       Axis accuracy       Level correct?
     │                │                      │                   │
     │           ┌────┴────┐            ┌───┴───┐          ┌───┴───┐
     │           │ Noise   │            │ Fail  │          │ Fail  │
     │           │ └──►    │            │ └──►  │          │ └──►  │
     │           │ Return  │            │ Fix   │          │ Tune  │
     │           │ to      │            │ heuristics       │ ladder
     │           │ research│            │                   │
     │           └────┬────┘            └───┬───┘          └───┬───┘
     │                │ Signal              │ Pass             │ Pass
     │                ▼                      ▼                   ▼
     │           Proceed to            Proceed to         Proceed to
     │           P1                    P3                 P4–P8
```

---

## Phase A: Corpus Creation (by separate agent)

**Who:** Separate agent (not you, not the skill-building agent).
**Input:** [[Test Corpus Specification]]
**Output:** 6 repos in `agent-harness-test-corpus/`

**Your action:**
1. Give the separate agent only the [[Test Corpus Specification]] document.
2. Tell it: "Create these 6 repos in `agent-harness-test-corpus/` per the spec. Don't ask questions, follow the spec exactly."
3. Verify: check that all 6 folders exist and match the directory trees in the spec.

**Time estimate:** 10–15 minutes (agent creates stubs + minimal file contents).

---

## Phase B: P2 — Walking Skeleton (the truth detector)

**Goal:** Validate the core hypothesis — does `classify → target → gap → roadmap` produce **signal or noise**?

**Repo:** `greenfield-nextjs/` only (L0)

**Why only one repo:** P2 is not about comprehensive testing. It's a thin slice to validate that the ladder produces actionable advice before we invest in full classifier encoding.

### Step B1: Run the thin pipeline

**Prompt to the skill-building agent:**

```
Run the agent harness pipeline on `agent-harness-test-corpus/greenfield-nextjs/` in advise mode.

Use ONLY these components (the rest are stubbed or hardcoded):
1. repo-classifier: classify the repo across all 10 axes
2. target-profiler: hardcoded target = "L1 requires root AGENTS.md"
3. gap-engine: detect "no root AGENTS.md"
4. roadmap: one item — create root AGENTS.md

Output format:
- project-profile (all 10 axes)
- identified gap
- roadmap (1 item)
- advice text (what the user should do)
```

### Step B2: Evaluate against rubric

Use the [[Test Corpus Specification]] §6 "Greenfield Protection" criterion.

**Checklist:**
- [ ] Classifier correctly identifies all 10 axes (profile completeness)
- [ ] `agents_md_quality` = `none` (correct — no AGENTS.md exists)
- [ ] Gap is exactly: "missing root AGENTS.md"
- [ ] Roadmap has exactly 1 item: create root AGENTS.md
- [ ] Advice is minimal and actionable (not mentioning L2, L3, drift, nested context)
- [ ] No false positives (doesn't say "add tests" or "add linting" unless they are truly missing — but these are secondary on L0)

### Step B3: Decision gate

| Result | Action |
|--------|--------|
| **Signal** — advice is clean, minimal, correct | ✅ Proceed to Phase C (P1) |
| **Noise** — advice is vague, over-engineered, or wrong | ❌ **STOP.** The ladder is wrong. Return to research. Do NOT proceed to P1. |

**Critical rule:** If P2 produces noise, no amount of P1–P8 engineering will fix a broken ladder. The problem is conceptual, not implementation.

---

## Phase C: P1 — Classifier Encoding + Validation

**Goal:** The classifier produces accurate, evidence-backed profiles on all 6 repos.

**Repos:** All 6 — run classifier on each independently.

### Step C1: Encode the classifier

**Prompt to the skill-building agent:**

```
Encode the repo-classifier as skill instructions.

Requirements:
1. Follow the Investigation Playbook from [[P0 — Foundations Decisions]]:
   Phase 0 (Orient) → Phase 1 (Cheap signals) → Phase 2 (Targeted deep-read) → Phase 3 (Synthesize) → Phase 4 (Escalate to interview)
2. Per-axis detection heuristics from [[Agent Harness Skill — RFC]] §7.2
3. Evidence-first: every non-obvious axis value must have ≥1 recorded signal
4. `undetermined` is valid — when evidence is ambiguous, mark undetermined and generate an interview question
5. Never guess without signals
```

### Step C2: Run classifier on all 6 repos

**Order matters:** Run in this order to catch cascade errors:
1. `greenfield-nextjs` — simplest, baseline
2. `express-api` — slightly more complex
3. `legacy-monolith` — tests edge cases
4. `nx-monorepo` — tests monorepo detection
5. `microservices-partial` — tests stale context detection
6. `mature-reference` — tests high-maturity recognition

**For each repo, capture:**
- Full `project-profile` output (all 10 axes)
- Recorded signals per axis
- `undetermined` axes + interview questions
- Time taken (rough — just note if it's instant or slow)

### Step C3: Score against rubric

Use the evaluation rubric from [[Test Corpus Specification]].

**Scoring worksheet (copy this for each repo):**

```
Repo: _______________

1. Profile completeness: ___ / 10 axes complete
2. Evidence quality: ___ (A/B/C/F)
3. Axis accuracy: ___ / 10 correct
4. Gap identification: ___ (A/B/C/F)
5. Level recommendation: ___ (A/B/C/F)

Special cases (if applicable):
- Greenfield protection: Pass / Fail
- Legacy honesty: Pass / Fail
- Drift detection: Pass / Fail

Weighted score: ___%
```

### Step C4: Decision gate

| Weighted score | Action |
|----------------|--------|
| ≥75%, no F grades | ✅ Proceed to Phase D (P3) |
| 60–75% | ⚠️ Fix classifier heuristics, re-run on failed repos. Do NOT proceed. |
| <60% or any F | ❌ **STOP.** Classifier heuristics are wrong or ladder is wrong. Debug: which axes fail? Are signals being recorded? Return to P1 or research. |

**What to fix in classifier (common failures):**
- Axis consistently wrong → add/adjust heuristics for that axis
- Missing signals → tighten Investigation Playbook Phase 1–2
- False gaps → add "what NOT to recommend" rules to gap-engine
- Over/under-leveling → adjust target-profiler logic

---

## Phase D: P3 — Maturity Engine + Level Validation

**Goal:** `target-profiler` + `gap-engine` recommend the correct next level for each repo.

**Repos:** All 6 — but now you care about level recommendation, not just axis values.

### Step D1: Run full pipeline (classify → target → gap → roadmap)

**Prompt:**

```
Run the FULL pipeline on each repo:
1. repo-classifier → project-profile
2. target-profiler → ideal harness per level
3. gap-engine → actual vs ideal
4. maturity engine → recommend NEXT level (not final)
5. roadmap → prioritized gaps with "why"

Output for each repo:
- Detected level (L0/L1/L2/L3)
- Recommended next level
- Top 3 gaps
- Rationale for level recommendation
```

### Step D2: Validate level recommendations

| Repo | Expected current | Expected recommended | What to check |
|------|------------------|----------------------|---------------|
| `greenfield-nextjs` | L0 | L1 | Minimal advice, no over-engineering |
| `express-api` | L0 | L1 | Context + tests + linting |
| `nx-monorepo` | L1 | L2 | Feedback loop + boundaries |
| `microservices-partial` | L2 | L3 | Drift detection + resync |
| `legacy-monolith` | L0 | L1 (honest) | Context first, warn about modernization cost |
| `mature-reference` | L3 | L3 (minimal gaps) | Tiny gaps only, no major changes |

### Step D3: Decision gate

| Result | Action |
|--------|--------|
| All 6 repos get correct level recommendation | ✅ Proceed to Phase E (P4–P8) |
| 1–2 repos off by one level | ⚠️ Tune target-profiler logic, re-run. |
| >2 repos wrong, or greenfield gets L2/L3 | ❌ **STOP.** Ladder or target-profiler is broken. Return to P3 or research. |

---

## Phase E: P4–P8 — Feature Building (thickening)

Once P2 + P1 + P3 are validated, build the remaining sub-skills. Use the corpus for regression testing.

### Regression test protocol

**After each phase (P4, P5, P6, P7, P8):**

1. Run the full pipeline on all 6 repos.
2. Check: do previous phases still pass?
3. New phase adds new capabilities — check only the NEW outputs.

**Example after P4 (context-authoring):**
- P1–P3 outputs should be identical (regression)
- NEW: Does generated `AGENTS.md` for `greenfield-nextjs` look correct?
- NEW: Does `microservices-partial` AGENTS.md get resynced (billing removed, notification added)?
- NEW: Does `mature-reference` get minimal, correct updates?

**Example after P6 (feedback + boundaries):**
- P1–P5 outputs should be identical (regression)
- NEW: Are lint/test commands correctly detected for `express-api` and `nx-monorepo`?
- NEW: Are protected paths flagged for `legacy-monolith` (hardcoded creds)?

---

## 📋 Master Checklist

### Before you start
- [ ] Separate agent created all 6 repos per [[Test Corpus Specification]]
- [ ] You verified repo structure matches spec

### P2 Walking Skeleton
- [ ] Ran thin pipeline on `greenfield-nextjs`
- [ ] Evaluated against greenfield protection criterion
- [ ] **Decision:** Signal → proceed / Noise → stop and research

### P1 Classifier
- [ ] Encoded classifier with Investigation Playbook
- [ ] Ran on all 6 repos in order
- [ ] Scored all repos against rubric
- [ ] **Decision:** ≥75% + no F → proceed / otherwise → fix or stop

### P3 Maturity Engine
- [ ] Ran full pipeline on all 6 repos
- [ ] Validated level recommendations
- [ ] **Decision:** All correct → proceed / otherwise → tune or stop

### P4–P8 (each phase)
- [ ] Built the feature
- [ ] Ran regression on all 6 repos (previous phases unchanged)
- [ ] Validated NEW outputs on relevant repos
- [ ] **Decision:** Pass → next phase / Fail → fix

---

## 🚨 Emergency Brakes

**Stop and research if:**
1. P2 produces noise on greenfield → ladder is wrong
2. P1 scores <60% → heuristics are fundamentally wrong
3. P3 recommends L3 for greenfield → target-profiler or ladder is wrong
4. Any phase breaks previous phases → regression, fix before proceeding

**Stop and fix (don't research) if:**
1. P1 scores 60–75% → adjust heuristics, re-run
2. P3 off by one level on 1–2 repos → tune target-profiler
3. New phase breaks 1 repo → fix edge case, re-run

---

## 🔗 Related Documents

- [[Test Corpus Specification]] — what the repos are + evaluation rubric
- [[Agent Harness Skill — Implementation Plan]] — build phases P1–P8
- [[P0 — Foundations Decisions]] — Investigation Playbook, contracts
- [[Agent Harness Skill — RFC]] — classifier axes, module definitions

---

*Created: 2026-06-09*