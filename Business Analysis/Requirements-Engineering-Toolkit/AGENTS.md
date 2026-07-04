# Requirements Engineering Toolkit — Project Guide

> A 2-month learning-by-building project. Goal: Senior Full-Stack engineer becomes marketable as **dev + business analyst + AI-tooling author** for EU+CIS market.

## What is this project?

A dual-deliverable learning track:
1. **Personal competence** in requirements engineering (Discovery → Structuring → Domain/Quality → Integration).
2. **A public AI toolkit** of 6 procedural skills + 3 role-based subagents, portable across Claude Code / LangGraph / n8n.

Result goes into the resume as a concrete artifact (open-source toolkit repo), not as a vague claim.

---

## 🗺️ Navigation

| File | Purpose |
|---|---|
| [[Requirements-Engineering-Roadmap]] | **The plan.** Architecture, all 4 waves, distillation contracts, schedule, source materials. Read first. |
| [[Progress-Tracker]] | **Visual progress table.** Updated as cases and skills are completed. |
| [[wave-1-discovery/AGENTS]] | Wave 1 detailed description: Discovery (intake, Event Storming). |
| [[wave-2-structuring/AGENTS]] | Wave 2 detailed description: Structuring (story mapping, BDD). |
| [[wave-3-domain-quality/AGENTS]] | Wave 3 detailed description: Domain (DDD) + Quality (NFRs). |
| [[wave-4-integration/AGENTS]] | Wave 4 detailed description: Integration (orchestrator + adapter + capstone). |

Each wave folder also contains `concept-notes/` for study material and case artifacts as they are produced.

---

## 🎯 The 4 Waves at a Glance

| Wave | Window | Theme | Skills built | Subagent |
|---|---|---|---|---|
| **1 — Discovery** | wk 1-2 (Jul 4-17) | Interviews, Event Storming | `requirements-intake`, `event-storming-facilitator` | The Stakeholder |
| **2 — Structuring** | wk 3-4 (Jul 18-31) | Story mapping, BDD | `story-mapper`, `spec-by-example` | The Devil's Advocate |
| **3 — Domain & Quality** | wk 5-6 (Aug 1-14) | DDD Context Mapping, NFRs | `context-mapper`, `nfr-elicitor` | The Tech Lead |
| **4 — Integration** | wk 7-8 (Aug 15-28) | Orchestrator, adapter, portfolio | Orchestrator + adapter + capstone | — |

---

## 🔄 Practice Loop (the standard rhythm for every technique)

| Stage | Time | What happens |
|---|---|---|
| **0. Brief** | ~5 min | Agent issues a **detailed** project brief: domain, problem, persona, session goal, and **hidden assumptions** (3-5 facts the user must dig out). The brief is as detailed as possible. |
| **0.5. Pre-interview plan** (elicitation cases) | ~15 min | User drafts a pre-interview plan + question bank before role-play. Agent reviews before role-play starts. This is the first output that feeds skill distillation. |
| **1. Study (homework)** | ~1-2h | User reads technique material off-session. |
| **1.5. Study quiz** | ~5 min | User signals `готов к квизу`. Agent gives 3-5 short questions. Pass ≥ 70% → role-play. Fail → re-read flagged sections. |
| **2. Role-play** | ~30-45 min | **Subagent** plays the role with a system prompt. Agent does not play along — **hard mode**. See protocol below. |
| **3. Replay analysis** | ~15 min | Agent critiques against **role-play rubric** (separate from skill acceptance criteria). **No praise.** |
| **4. Iterate / next case** | ~30 min | User incorporates feedback (edits plan/artifact), then moves to next case on a new domain. |
| **5. Distillation** (after 3 cases on a topic) | ~60 min | **User authors the skill**; agent critiques and iterates. Only after technique is stable across multiple cases. |

> Learning happens at Stage 3 (analysis), not Stage 2 (role-play).

### Practice Protocol — explicit rules

**Subagents before cases.** Subagents (`The Stakeholder`, `The Devil's Advocate`, `The Tech Lead`) are **infrastructure for the learning process**, not learning products. They are built in the **Preparation phase of each wave**, before any case. They are improved iteratively as cases expose weaknesses. One product, iteratively — no v1/v2 split.

**Role-play mechanics:**
- The role is played by a **subagent with a system prompt** (not the main agent ad-hoc).
- Communication during role-play is in **Russian**.
- Artifacts (intake drafts, storming outputs, story maps, Gherkin, NFR tables) are in **English** — they are resume-facing.
- Turn-by-turn: one question → one answer. Trains active listening and adaptation.
- **Out-of-character prefix:** `// meta: <question>` — main agent responds as trainer, role is paused.
- **Solution-mode rule (variant B — immediate stop):** If the user proposes a feature/solution instead of probing the problem, the main agent **stops the role-play immediately** and calls it out. Role resumes after the user recalibrates.
- **Closing signal:** User types `всё, резюмирую` or `закрываю интервью` → subagent falls silent, main agent moves to Stage 3.

**Role-play rubric** (agent fills this at Stage 3, separate from skill acceptance criteria):

| Category | What it scores | Red flag |
|---|---|---|
| Question mix | open/closed/probing ratio | closed > 50% |
| Funnel discipline | broad → narrow, or jumping | chaotic jumps |
| Active listening | paraphrase / mirroring present | none |
| Solution-mode creep | feature proposals count | ≥ 1 |
| Silence tolerance | didn't fill every pause | spammed questions |
| Stakeholder adaptation | matched persona/age/resistance | generic |
| Assumption surfacing | hidden facts uncovered (of N planted) | < 30% |
| Closing | summarized, verified understanding | abrupt end |

**Distillation rules:**
- Trigger: technique is stable across the wave's cases (qualitative judgment by both sides).
- **User is the author** of the skill. Agent is critic + helper.
- No `Templates/Skill-Template.md` — user presents the skill draft, agent evaluates against the wave's acceptance criteria.
- Skills live in `work/ba-ai-toolkit/`.

**Source fallback:** If the user cannot access a canonical source, the agent either finds an alternative or writes a `[SYNTHESIS]`-marked summary itself.

---

## 📐 Toolkit Architecture

```
[requirements-intake] ──┐
                        ├──→ [story-mapper] ──→ [spec-by-example] ──→ tests
[event-storming] ───────┤
                        └──→ [context-mapper] ──→ architecture

[nfr-elicitor] ─────────────────────────────────────→ SLOs, ADRs
```

**Portability contract:**
- **Portable:** prompts, procedures, knowledge, role definitions (markdown).
- **Non-portable:** tool integration, state, orchestration (rewritten per runtime).

Architecture: **harness-agnostic knowledge layer + thin adapter per runtime.**

Code lives in `work/ba-ai-toolkit/` (separate working directory, intended to become a public repo).

---

## ⚙️ Hard Rules (non-negotiable)

1. **Practice before code.** A skill is distilled only after the technique is stable across 2-3 cases. Never after case 1.
2. **Hard mode in role-play.** The Stakeholder subagent is explicitly authorized to be vague, contradictory, evasive, and to hide assumptions. No cheerleading.
3. **6 skills cap.** Quality > quantity. No cramming into the 2-month window.
4. **Source-marking convention.** When the agent outputs module material, it marks each claim as `[CANON]` (well-established) or `[SYNTHESIS]` (verify against source).
5. **Update `updated:` in frontmatter and the Progress Tracker on every meaningful change.**

---

## 🚫 Out of Scope (2-month constraint)

- BPMN deep (reading only).
- Use Cases Cockburn in detail.
- ATAM, formal architecture methods (system design territory).
- Decision Tables / State Machines as separate topics.
- Regulatory deep-dives (GDPR/HIPAA) — only as case factors.
- BABOK/Volere as study material — reference only.

---

## 📚 Canonical Sources (priority order)

1. Karl Wiegers — *Software Requirements* 2/e.
2. Jeff Patton — *User Story Mapping*.
3. Gojko Adzic — *Specification by Example* + *Impact Mapping*.
4. Vaughn Vernon — *Implementing DDD* (Ch. 2-3).
5. Eric Evans — *DDD* (Ch. 14).
6. Google SRE Book — Ch. 4 (SLO/SLI).

Full per-wave reading lists are in each wave's `AGENTS.md`.

---

*Last updated: 2026-07-04 — project structure created.*
