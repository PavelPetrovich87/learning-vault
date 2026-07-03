---
status: implemented
tags: [research, agent-harness, failure-modes, empirical, msr26, columbia-dap, agent-capability]
created: 2026-06-09
updated: 2026-06-09
source:
  title: "Catalogue of Observed Agent Failure Modes — Empirical Anchor for L0→L4"
  author: self-directed research
  date_read: 2026-06-09
---

# Catalogue of Observed Agent Failure Modes — Study Block

> [!abstract] TL;DR
> Analysis of two major empirical studies on agent failures: MSR'26 (33,000 agent PRs) and Columbia DAP Lab (9 failure patterns in vibe-coding apps). The research reveals a **fundamental tension** with our current ladder: observed failures are primarily **social-technical and coordination-related** (38% reviewer abandonment, 23% duplicate PRs, 17% CI failures), not context-comprehension failures. The user proposes an alternative L0→L4 ladder based on **agent capability/autonomy levels** (Grounded → Collaborative), which is orthogonal to our current **repo readiness** ladder (Blind → Self-maintaining). This requires architectural reconciliation.

---

## 🔬 Phase 1: EXTRACTION — What Was Found

### Key Sources

| Study | Scale | Focus |
|---|---|---|
| **MSR'26** | 33,000 agent PRs (Codex, Copilot, Devin, Cursor, Claude Code) | Repo-level agent behavior in open-source projects |
| **Columbia DAP Lab** | 15+ apps built iteratively with 5 agents | Vibe-coding failure patterns across the stack |

### MSR'26: Hierarchical Taxonomy of Rejected PRs (600 manually labeled)

| Level | Pattern | Frequency | Root Cause |
|---|---|---|---|
| **Reviewer** | Abandoned / Not Reviewed | **38%** (228 PR) | Agent cannot initiate meaningful interaction or argue PR value |
| **Pull Request** | Duplicate PR | 23% (142) | No contextual awareness of ongoing work |
| | Unwanted Feature | 4% (24) | Misunderstands project priorities/boundaries |
| | Non-Functional / Wrong Branch / Wrong Description | 3% (22) | Lack of procedural discipline |
| **Code** | CI / Test Failure | 17% (99) | Does not validate changes before submitting PR |
| | Incorrect Implementation | 3% (19) | Semantic misunderstanding of requirements |
| | Incomplete Implementation | 2% (15) | No self-check for completeness |
| **Agentic** | Misalignment (ignores reviewer) | 1.6% (9) | Loses context across review iterations |
| | License / CLA Issues | <1% (4) | No legal awareness |

**Critical insight:** The #1 failure is **not code quality** — it's **social-technical**. Agents create PRs that humans ignore because the agent cannot "sell" the change, coordinate, or demonstrate awareness of project context.

### Columbia DAP Lab: 9 Critical Failure Patterns

1. **Presentation & UI Grounding Mismatch** — spatial/visual reasoning failures
2. **State Management Failures** — consistency lost during refactoring
3. **Business Logic Mismatch** — code runs but produces wrong results
4. **Data Management Errors** — schema confusion, ID mismatches
5. **API & External Service Integration Failures** — edge cases unhandled
6. **Security Vulnerabilities** — invisible to unit tests
7. **Repeated Code** — duplication instead of refactoring
8. **Codebase Awareness & Refactoring Issues** — forgets to update related components as scale grows
9. **Exception & Error Handling** — prefers **suppressing** errors over surfacing them

**Critical insight:** Agents prioritize **runnable code over correctness**. This is a systematic cognitive bias, not fixable by better prompts alone.

### Unified Failure Mode Catalogue (5 Layers)

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 4: SOCIO-TECHNICAL (coordination with humans)       │
│  • Reviewer Abandonment (38%)                               │
│  • Misalignment with reviewer (1.6%)                        │
│  • License / CLA violations                                 │
├─────────────────────────────────────────────────────────────┤
│  LAYER 3: WORKFLOW (fitting into process)                  │
│  • Duplicate PR (23%)                                       │
│  • Unwanted Feature (4%)                                    │
│  • Wrong Branch / Description (3%)                          │
├─────────────────────────────────────────────────────────────┤
│  LAYER 2: SYSTEM (component interaction)                   │
│  • CI / Test Failure (17%)                                  │
│  • API / External Integration Failures                      │
│  • State / Data Management Errors                           │
│  • Security Vulnerabilities                                 │
├─────────────────────────────────────────────────────────────┤
│  LAYER 1: CODE (local correctness)                         │
│  • Incorrect Implementation (3%)                            │
│  • Incomplete Implementation (2%)                           │
│  • Business Logic Mismatch                                  │
│  • Exception Suppression                                    │
│  • Repeated Code                                            │
├─────────────────────────────────────────────────────────────┤
│  LAYER 0: GROUNDING (intent comprehension)                 │
│  • UI Grounding Mismatch                                    │
│  • Wrong Task Description                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧠 Phase 2: ANALYSIS — What It Means

### Strengths

- **Empirically grounded** — 33k PRs + 15 apps. Not opinions.
- **Surprising distribution** — 38% social, 23% coordination, 17% CI. Only 5% actual code correctness. This reshapes priority.
- **Actionable** — Each failure mode maps to a specific repo-provided capability or agent behavior.

### Weaknesses / Limitations

- **Selection bias** — MSR'26 studies *rejected* PRs (28.5% of total). 71.5% merged PRs are not analyzed. We don't know what *successful* agents do differently.
- **Open-source bias** — Corporate repos may have different failure patterns (stricter CI, less reviewer abandonment due to assigned reviewers).
- **Vibe-coding ≠ production** — Columbia study focuses on app prototyping, not maintenance of mature repos.

### The Fundamental Tension: Two Ladders

The research proposes an **agent capability ladder**:

| Level | Name | What Agent CAN Do | Eliminates |
|---|---|---|---|
| L0 | Grounded | Correctly interprets task | Layer 0 |
| L1 | Semantically Correct | Code solves the right problem | Layer 1 |
| L2 | System-Valid | Passes CI before PR; no integration breakage | Layer 2 |
| L3 | Project-Aware | No duplicates; reads CONTRIBUTING; right branch | Layer 3 |
| L4 | Collaborative | Engages reviewer; incorporates feedback | Layer 4 |

**Our current ladder** is a **repo readiness ladder**:

| Level | Name | What Repo PROVIDES |
|---|---|---|
| L0 | Blind | Nothing |
| L1 | Oriented | Effective AGENTS.md |
| L2 | Navigable | Repo map + nested context |
| L3 | Guarded | Feedback commands + boundaries |
| L4 | Self-maintaining | Drift resync + advanced tooling |

**These are orthogonal dimensions.**
- A repo can be L4 Self-maintaining but the agent in it might only be L1 Semantically Correct.
- An L4 Collaborative agent might succeed in an L0 Blind repo.

### Open Questions

- [ ] Which ladder does our skill measure? Repo readiness, agent capability, or both?
- [ ] If repo readiness: does our ladder actually address the observed failure modes? (AGENTS.md helps with Layer 1–2, but not Layer 3–4.)
- [ ] If agent capability: how does a repo "prepare" an agent to be more capable?
- [ ] Can we map the two ladders? E.g., "L2 Navigable repo enables L2 System-Valid agent behavior"?

---

## 🛠️ Phase 3: APPLICATION — How to Apply to Agent Harness Skill

### Current Work (Our Project)

The empirical data forces a strategic choice. Three options:

#### Option A: Keep Repo Readiness Ladder, But Redefine Levels by Failure Modes

Map each repo readiness level to the failure modes it prevents:

| Repo Level | Prevents | How |
|---|---|---|
| **L0 → L1** | Layer 1 (Incorrect/Incomplete Implementation) | AGENTS.md with Role, examples, boundaries |
| **L1 → L2** | Layer 2 (CI Failure, Integration, State/Data) | Repo map + command registry + test runner config |
| **L2 → L3** | Layer 3 (Duplicate PR, Unwanted Feature, Wrong Branch) | CONTRIBUTING.md, ROADMAP, issue templates, project conventions |
| **L3 → L4** | Layer 4 (Reviewer Abandonment, Misalignment) | PR templates, review guidelines, feedback loop config |

**Problem:** This makes L3–L4 about project governance, not agent infrastructure. It drifts toward the "deploy" axis we explicitly cut.

#### Option B: Switch to Agent Capability Ladder

Measure what the agent can do in the repo. The skill becomes an **agent assessment + enablement** tool.

**Problem:** Our skill is called "Agent Harness" and its charter is "prepare repositories." Measuring agent capability is out of scope unless we define "preparation" as "enabling the agent to reach higher capability levels."

#### Option C: Dual-Axis (Reconciled)

A 2×2 matrix:
- **X-axis:** Repo Readiness (what infrastructure exists)
- **Y-axis:** Agent Capability (what the agent can do given that infrastructure)

The skill's job: assess repo readiness (X), then recommend infrastructure upgrades that unlock the next agent capability level (Y).

**Example:**
- Repo is L1 Oriented (has AGENTS.md) but agent behavior is L0 Grounded (keeps misinterpreting tasks).
- Recommendation: improve AGENTS.md with clearer task descriptions + examples (move repo to L1.5) to unlock agent L1.

**Problem:** This is more complex. Is the complexity justified?

### Recommendation

**Option A with a pivot:** Keep the repo readiness ladder, but acknowledge that L3–L4 are about **project governance infrastructure** (CONTRIBUTING, ROADMAP, PR templates), not just agent tooling. This is within "agent-readiness" because an agent cannot be L3 Project-Aware without the repo providing project context.

The key insight from the empirical data:
- **L1–L2** are about **context comprehension** (AGENTS.md, repo map, commands) — this is what we've been designing.
- **L3–L4** are about **project integration** (CONTRIBUTING, ROADMAP, review etiquette) — this is new and was underweighted.

**Design change:** Add a "Project Governance" module to the classifier and maturity engine. The 10 axes should include signals for:
- `contributing_guide` (CONTRIBUTING.md, PR template)
- `project_priorities` (ROADMAP, milestones, issue labels)
- `review_process` (CODEOWNERS, review guidelines)

---

## ✅ Phase 4: ACTION — What Next

### Next Steps

- [x] **Architectural decision (2026-06-09): Option A, scope-narrowed.** Keep the **repo readiness** ladder, but address **only the Code + System layers** (the missing-affordance failures a repo can fix). Carve-outs:
  - **Workflow layer** (duplicate PR 23%, unwanted feature) → **separate sibling governance skill**, out of scope now.
  - **Socio-technical layer** (reviewer abandonment 38%, misalignment) → **fully out of scope** (no repo artifact fixes maintainer trust/bandwidth).
  - **Agent capability** (Option B) → rejected; we measure repo affordance, not agent behavior.
- [x] Ladder simplified to **4 levels** (L0 Blind / L1 Oriented / L2 Self-Correcting / L3 Self-Maintaining). L1 = Code layer, L2 = System layer, L3 = maintenance discipline. Encoded in [[Agent Harness Skill — RFC]], [[P0 — Foundations Decisions]], [[Agent Harness Skill — Implementation Plan]].
- [x] Reconciled with AGENTS.md + Context Engineering: a "perfect" AGENTS.md (L1) only prevents Code-layer failures (~5%); the feedback loop (L2) is what kills the largest addressable failure (CI/test 17%). This is why L2, not L1, is the high-value rung.
- [ ] *(Future)* Spec the sibling governance skill (CONTRIBUTING/ROADMAP/PR-template/duplicate detection) when this skill ships.

### Related Notes

- [[AGENTS.md Spec & Agent Failure Modes — Study Block]] — AGENTS.md effectiveness research.
- [[Context Engineering — Study Block]] — positional bias, progressive disclosure, token budget.
- [[Agent Harness Skill — RFC]] — architecture spec; needs ladder decision.
- [[P0 — Foundations Decisions]] — contracts + ladder; needs revision based on this research.

### Status History

- `2026-06-09`: researching — analyzed MSR'26 and Columbia DAP Lab studies.
- `2026-06-09`: analyzing — identified fundamental tension between repo readiness and agent capability ladders.
- `2026-06-09`: implemented — scope decision made (Option A, narrowed to Code + System layers); 4-level ladder anchored and propagated to RFC/P0/Plan.

---

## 🗣️ Key Quotes for Reference

> "The #1 failure is not code quality — it's social-technical. Agents create PRs that humans ignore." — MSR'26 analysis

> "Agents prioritize runnable code over correctness." — Columbia DAP Lab

> "38% reviewer abandonment + 23% duplicate PRs + 17% CI failures = 78% of rejections have nothing to do with code correctness." — MSR'26
