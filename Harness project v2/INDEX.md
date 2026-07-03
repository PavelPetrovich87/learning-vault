---
status: analyzing
tags: [agent-harness, project-index, navigation]
created: 2026-06-09
updated: 2026-06-09
---

# Agent Harness Skill — Project Index

> One-page navigation hub. All project documents linked from here.

---

## 🎯 Current State

**Scope:** Repo configuration for AI-agent convenience — context (`AGENTS.md`), feedback loop (linters/tests/types), architecture enforcement.
**Out of scope:** Deployment, project governance (sibling skill), reviewer-abandonment (social, no repo fix).
**Ladder:** L0 Blind → L1 Oriented → L2 Self-Correcting → L3 Self-Maintaining (4 levels, empirically anchored).
**Next blocker:** Assemble test corpus (5–6 repos).

---

## 📋 Core Documents (decisions & plans)

| Document | What it is | Last updated |
|----------|-----------|--------------|
| [[Agent Harness Skill — RFC]] | Architecture spec: topology, modules, contracts, execution flow, invariants | 2026-06-09 (v5.1) |
| [[Agent Harness Skill — Implementation Plan]] | Step-by-step build plan (P0–P8), critical path, risk register | 2026-06-09 |
| [[P0 — Foundations Decisions]] | Artifact form, 3 contracts, Investigation Playbook, interview criteria | 2026-06-09 (rev 4) |
| [[Test Corpus Specification]] | 6 repos with ground truth + evaluation rubric for classifier validation | 2026-06-09 |
| [[Test Corpus Workflow]] | Step-by-step process: how to run classifier on corpus, decision gates, when to stop | 2026-06-09 |

---

## 🔬 Research Notes (empirical anchors)

All live in `research_in_progress/` per vault rules. Do not move.

| Document | What it anchors | Status |
|----------|----------------|--------|
| [[Catalogue of Observed Agent Failure Modes — Study Block]] | **The ladder itself.** MSR'26 (33k PRs) + Columbia DAP Lab (9 patterns). 4-level ladder derived from Code + System failure layers. | ✅ Implemented (decision made) |
| [[AGENTS.md Spec & Agent Failure Modes — Study Block]] | L1 quality: Signal/Noise, Inferability Test, minimal viable template (<150 lines), format efficiency. | ✅ Analyzing |
| [[Context Engineering — Study Block]] | Positional bias ("Lost in the Middle"), progressive disclosure, token budget, Markdown vs YAML/JSON/XML. | ✅ Analyzing |

---

## 🏛️ Ancestor Documents (v4 dual-axis)

Preserved for reference. Not modified.

| Document | Note |
|----------|------|
| [[Repo Harness Skill — RFC v4 Architecture]] | Dual-axis ancestor (agent + deploy). Superseded by v5. |
| [[Repo Harness Skill — Step-by-Step Implementation Plan (RFC v4)]] | 10-phase dual-axis plan. Superseded by v5. |

---

## 🔗 Quick Cross-References

### Ladder definition
- RFC §7.1 → canonical table
- P0 Contract C → per-field LLM guidance for `roadmap.current_level`
- Catalogue research note → empirical justification

### Scope boundary
- RFC §3 → one-line definition + missing/ignored affordance split
- P0 Open Items → locked decisions

### Open questions status
| # | Question | Status | Where resolved |
|---|----------|--------|----------------|
| 1 | Defensible ladder? | ✅ Resolved | Catalogue research + RFC §11 |
| 2 | Context drift detection | ⏳ Open | P8 |
| 3 | Detection vs interview balance | ⏳ Open | P1 |
| 4 | Nested AGENTS.md granularity | ⏳ Open | P4 |
| 5 | Autonomous file-write policy | ⏳ Open | P7 |
| 6 | Measuring agent productivity | ⏳ Open | Post-P8 |

---

## 📁 Directory Structure

```
Harness project v2/
├── Agent Harness Skill — RFC.md              ← architecture
├── Agent Harness Skill — Implementation Plan.md ← build plan
├── P0 — Foundations Decisions.md              ← contracts & playbook
└── INDEX.md                                   ← this file

research_in_progress/
├── Catalogue of Observed Agent Failure Modes — Study Block.md ← ladder anchor
├── AGENTS.md Spec & Agent Failure Modes — Study Block.md      ← L1 quality
├── Context Engineering — Study Block.md                       ← CE fundamentals
├── Repo Harness Skill — RFC v4 Architecture.md                ← ancestor
├── Repo Harness Skill — Step-by-Step Implementation Plan (RFC v4).md ← ancestor
└── (other unrelated research)
```

## 🛠️ Implementation Workspace

| Document | What it is | Location |
|----------|-----------|----------|
| Skill working directory | Code, drafts, test runs | `agent-harness-skill/` |
| P2 Walking Skeleton | Minimal skill draft for validation | `agent-harness-skill/p2-walking-skeleton.md` |

---

*Last updated: 2026-06-10*