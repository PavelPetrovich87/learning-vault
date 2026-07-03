---
status: analyzing
tags: [agent-harness, workflow, test-corpus, validation, process, openclaw]
created: 2026-06-09
updated: 2026-06-10
---

# Test Corpus Workflow (OpenClaw Edition)

> Step-by-step process for validating the Agent Harness Skill using the OpenClaw Skill Evolution Loop framework.
>
> **Prerequisites:** [[Test Corpus Specification]] · [[OpenClaw Skill Evolution Loop — Implementation Plan]]

---

## 🎯 Big Picture

```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenClaw Skill Evolution Loop                │
│                                                                 │
│  User ──► Orchestrator ──► spawns Runners (clean room)        │
│                              └──► repo 1, repo 2, ... repo N    │
│                              └──► M replicas per repo           │
│                              └──► writes output.md to runs/     │
│                                                                 │
│  Orchestrator ──► spawns Judges (separate knowledge)            │
│                              └──► scorecard.yaml per repo       │
│                              └──► consistency across replicas   │
│                                                                 │
│  Orchestrator ──► Aggregates weighted score + regression check  │
│                              └──► vs golden/<prev-SHA>/         │
│                              └──► Decision gate: PASS/FIX/STOP  │
│                                                                 │
│  Evolution (guide/autonomous) ──► Patch SKILL.md ──► Re-run     │
└─────────────────────────────────────────────────────────────────┘
```

**Three roles, separation of knowledge:**

| Role | Sees | Must NOT see |
|------|------|--------------|
| **Orchestrator** | Everything: skill git history, corpus, ground truth, runs, golden | — |
| **Runner** | SKILL.md + ONE corpus repo | Ground truth, rubric, other runners' outputs, vault docs |
| **Judge** | One runner output + ground truth + rubric | Skill internals, other judges' scores |

> [!important] Clean-room invariant
> Runner must never read `ground-truth/`, `rubric/`, `golden/`, or this vault. If runner sees expected answers, every score is worthless.

---

## Phase A: Corpus Creation (by separate agent)

**Who:** Separate agent (not orchestrator, not runner, not judge).
**Input:** [[Test Corpus Specification]]
**Output:** 6 repos in `~/skill-lab/corpus/`

**Steps:**
1. Give the agent only the Test Corpus Specification.
2. Tell it: "Create these 6 repos in `~/skill-lab/corpus/` per the spec. Don't ask questions, follow exactly."
3. Verify: check that all 6 folders exist and match directory trees.
4. Commit + tag: `git tag corpus-v1`

**Ground truth extraction:**
- Extract per-repo expected values from spec tables into `~/skill-lab/ground-truth/*.yaml`
- Include synonym lists (e.g., `api-backend ≈ backend-api`)
- Mark which axes are "clearly detectable" (undetermined on these = fail)

**Rubric extraction:**
- Write `~/skill-lab/rubric/scoring.md` (rubric §1–8 + weights verbatim from spec)
- Write `~/skill-lab/rubric/anti-patterns.txt` (deterministic forbidden-pattern list)

---

## Phase B: P2 — Walking Skeleton Validation (the truth detector)

**Goal:** Validate the core hypothesis — does the pipeline produce **signal or noise**?

**Repo:** `greenfield-nextjs/` only (L0)

**Why only one repo:** P2 is not comprehensive testing. It's a thin slice to validate the ladder before investing in full layers.

### Step B1: Configure runner (clean room)

Runner workspace: `~/skill-lab/runner-ws/`
- SKILL.md symlinked from `~/skill-lab/skills/agent-harness-p2/`
- No access to ground-truth/, rubric/, golden/
- Model: fixed target model (production condition)

### Step B2: Spawn runner

```
task: |
  Run skill `agent-harness-p2` on repository:
  REPO_PATH: ~/skill-lab/corpus/greenfield-nextjs
  Write complete output to:
  OUTPUT_PATH: ~/skill-lab/runs/<RUN_ID>/greenfield-nextjs/r1/output.md
agentId: harness-runner
```

### Step B3: Spawn judge

```
task: |
  Score runner output for repo greenfield-nextjs.
  OUTPUT_PATH: runs/<RUN_ID>/greenfield-nextjs/r1/output.md
  TRUTH_PATH: ground-truth/greenfield-nextjs.yaml
  RUBRIC: rubric/scoring.md
  SCORECARD_PATH: runs/<RUN_ID>/greenfield-nextjs/scorecard.yaml
agentId: harness-judge
```

### Step B4: Evaluate scorecard

**Deterministic checks (zero-variance):**
- [ ] Format compliance: all required sections present
- [ ] Anti-pattern scan: no test/lint/CI/drift/nested-context/L2/L3 mentions
- [ ] Roadmap cardinality: exactly 1 item
- [ ] Advice length: ≤3 paragraphs
- [ ] Axis exact/synonym match vs ground truth

**LLM-judged dimensions:**
- [ ] Evidence quality: A/B/C/F
- [ ] Gap identification: A/B/C/F
- [ ] Level recommendation: A/B/C/F
- [ ] Greenfield protection: Pass/Fail

### Step B5: Decision gate

| Scorecard | Verdict | Action |
|-----------|---------|--------|
| All deterministic pass + gap/level = A + greenfield protection pass | **SIGNAL** | ✅ Proceed to Phase C (P1) |
| Any deterministic fail, or gap/level < A, or greenfield protection fail | **NOISE** | ❌ **STOP.** Ladder is wrong. Return to research. Do NOT proceed. |

**Critical rule:** If P2 produces noise, no amount of P1–P8 engineering will fix a broken ladder. The problem is conceptual.

---

## Phase C: P1 — Classifier Encoding + Validation

**Goal:** Classifier produces accurate, evidence-backed profiles on all 6 repos.

**Repos:** All 6 — spawn one runner per repo.

### Step C1: Fan out runners (parallel or sequential)

```
For each repo in [greenfield-nextjs, express-api, legacy-monolith, 
                   nx-monorepo, microservices-partial, mature-reference]:
  
  spawn runner:
    REPO_PATH: ~/skill-lab/corpus/<repo>
    OUTPUT_PATH: ~/skill-lab/runs/<RUN_ID>/<repo>/r1/output.md
```

**Order (sequential mode):**
1. `greenfield-nextjs` — simplest, baseline
2. `express-api` — slightly more complex
3. `legacy-monolith` — edge cases
4. `nx-monorepo` — monorepo detection
5. `microservices-partial` — stale context
6. `mature-reference` — high-maturity recognition

### Step C2: Fan out judges

After all runners complete, spawn one judge per repo:

```
For each repo:
  spawn judge:
    OUTPUT_PATH: runs/<RUN_ID>/<repo>/r1/output.md
    TRUTH_PATH: ground-truth/<repo>.yaml
    SCORECARD_PATH: runs/<RUN_ID>/<repo>/scorecard.yaml
```

### Step C3: Orchestrator aggregates

1. Read all 6 scorecards.
2. Compute weighted score per spec:
   - Profile completeness: 10%
   - Evidence quality: 15%
   - Axis accuracy: 25%
   - Gap identification: 25%
   - Level recommendation: 15%
   - Special cases (greenfield/legacy/drift): 10%
3. Write `runs/<RUN_ID>/report.md`.

### Step C4: Decision gate

| Weighted score | Verdict | Action |
|----------------|---------|--------|
| ≥75%, no F grades | **PASS** | ✅ Proceed to Phase D (P3). Promote run to `golden/<SKILL_SHA>/`. |
| 60–75%, no F | **FIX** | ⚠️ Adjust classifier heuristics. Re-run failed repos. |
| <60% or any F | **STOP** | ❌ Heuristics or ladder fundamentally wrong. Debug which axes fail. Return to P1 or research. |

**What to fix (common failures):**
- Axis consistently wrong → add/adjust heuristics
- Missing signals → tighten Investigation Playbook Phase 1–2
- False gaps → add "what NOT to recommend" rules

---

## Phase D: P3 — Maturity Engine + Level Validation

**Goal:** `target-profiler` + `gap-engine` recommend correct next level for each repo.

**Repos:** All 6 — but focus shifts from axis values to level recommendations.

### Step D1: Run full pipeline

Same runner/judge/orchestrator flow as Phase C, but now judge evaluates level recommendation accuracy:

| Repo | Expected current | Expected recommended |
|------|------------------|----------------------|
| `greenfield-nextjs` | L0 | L1 |
| `express-api` | L0 | L1 |
| `nx-monorepo` | L1 | L2 |
| `microservices-partial` | L2 | L3 |
| `legacy-monolith` | L0 | L1 (honest) |
| `mature-reference` | L3 | L3 (minimal gaps) |

### Step D2: Decision gate

| Result | Verdict | Action |
|--------|---------|--------|
| All 6 correct level | **PASS** | ✅ Proceed to Phase E (P4–P8) |
| 1–2 repos off by one | **FIX** | ⚠️ Tune target-profiler. Re-run. |
| >2 wrong, or greenfield gets L2/L3 | **STOP** | ❌ Ladder or target-profiler broken. |

---

## Phase E: P4–P8 — Feature Building + Regression

**Goal:** Build remaining sub-skills. Use corpus for regression testing.

### Regression test protocol (after each phase)

**After each phase (P4, P5, P6, P7, P8):**

1. Run full pipeline on all 6 repos (runner → judge → aggregate).
2. **Regression check:** diff axis values, gap lists, levels against `golden/<prev-SHA>/`.
3. Any metric that *dropped* on any repo = **regression flag**, even if aggregate passes.

| Condition | Verdict | Action |
|-----------|---------|--------|
| ≥75%, no F, no regression | **PASS** | Promote to golden. Proceed to next phase. |
| ≥75%, but regression detected | **PASS-WITH-REGRESSION** | Report with side-by-side diff. Do NOT auto-promote. Human decides. |
| <75% or any F | **FIX/STOP** | Per Phase C/D gates. |

**New outputs validation:**
- Check only NEW outputs for the phase.
- Previous phases' outputs must be identical (regression test).

---

## 🔧 The Evolution Step (skill self-improvement, gated)

On **FIX** verdict, the orchestrator (strong model, full context):

1. Collects all `failure_notes` across scorecards.
2. Classifies failures into noise taxonomy: hallucination / over-generalization / scope creep / ladder violation / missing evidence / format drift.
3. Maps failures to SKILL.md sections.
4. Proposes a **minimal diff** with failure evidence + hypothesis + expected metric movement.

### Anti-overfitting guardrails

| Guardrail | Rule |
|-----------|------|
| No ground-truth leakage | Patch must never contain corpus-specific strings (`greenfield-nextjs`, exact expected values) |
| Generalize, don't memorize | Change heuristics/constraints, not per-repo special cases |
| One patch per cycle | Never batch unrelated fixes — attribution requires one-change-one-measurement |
| STOP never auto-patches | <60% / F-grades = conceptual problem. Human research territory. |

### Evolution policy (advise / guide / autonomous)

| Mode | Who approves | Behavior |
|------|-------------|----------|
| **advise** | nobody | Report + proposed diff in chat. Default. |
| **guide** | human, per patch | Orchestrator posts diff + evidence. Waits for "apply". On apply: commit, re-run failed repos. |
| **autonomous** | policy | Apply → commit → re-run failed → if improved, full re-run all 6; if not, `git revert` and stop. Max K=3 iterations. |

Run autonomous only after guide mode has shown ≥3 sensible patches in a row.

---

## 📋 Master Checklist

### Phase A (Corpus)
- [ ] 6 repos created per Test Corpus Specification
- [ ] `corpus-v1` tag applied
- [ ] Ground-truth YAMLs extracted for all repos
- [ ] Rubric + anti-patterns written

### Phase B (P2 Walking Skeleton)
- [ ] Runner spawned on `greenfield-nextjs`
- [ ] Judge produced scorecard
- [ ] Deterministic checks all pass
- [ ] Greenfield protection passes
- [ ] **Decision:** SIGNAL → proceed / NOISE → stop and research

### Phase C (P1 Classifier)
- [ ] Runners spawned on all 6 repos
- [ ] Judges produced 6 scorecards
- [ ] Orchestrator aggregated weighted score
- [ ] Regression check vs golden (if exists)
- [ ] **Decision:** PASS → proceed / FIX → patch + re-run / STOP → debug

### Phase D (P3 Maturity Engine)
- [ ] Full pipeline on all 6 repos
- [ ] Level recommendations validated
- [ ] **Decision:** PASS → proceed / FIX → tune / STOP → debug

### Phase E (P4–P8, each phase)
- [ ] Feature built
- [ ] Regression test on all 6 repos
- [ ] New outputs validated
- [ ] **Decision:** PASS / PASS-WITH-REGRESSION / FIX / STOP

---

## 🚨 Emergency Brakes

**STOP and research (never auto-patch):**
1. P2 produces noise on greenfield → ladder is wrong
2. P1 scores <60% → heuristics fundamentally wrong
3. P3 recommends L3 for greenfield → target-profiler or ladder wrong
4. Any phase breaks previous phases → regression, fix before proceeding

**FIX and re-run (patch allowed):**
1. P1 scores 60–75% → adjust heuristics
2. P3 off by one level on 1–2 repos → tune target-profiler
3. New phase breaks 1 repo → fix edge case

---

## 🔗 Related Documents

- [[Test Corpus Specification]] — ground truth + rubric source
- [[OpenClaw Skill Evolution Loop — Implementation Plan]] — infrastructure + agent config + build phases
- [[Agent Harness Skill — Implementation Plan]] — build phases P1–P8
- [[P0 — Foundations Decisions]] — Investigation Playbook, contracts
- [[Agent Harness Skill — RFC]] — classifier axes, module definitions

---

*Updated: 2026-06-10 (OpenClaw integration)*