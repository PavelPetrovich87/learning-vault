---
status: analyzing
tags: [research, agent-harness, agents-md, context-engineering, failure-modes, in-progress]
created: 2026-06-09
updated: 2026-06-09
source:
  title: "AGENTS.md: Deep Dive into Context Layer for AI Coding Agents"
  author: self-directed research
  date_read: 2026-06-09
---

# AGENTS.md Spec & Agent Failure Modes — Study Block

> [!abstract] TL;DR
> Research into the `AGENTS.md` open-source format (Agentic AI Foundation / Linux Foundation), its adoption across 60,000+ repos, and empirical studies from 2026 on its effectiveness. Key paradox: `AGENTS.md` reduces runtime and token cost (↓28% median runtime, ↓20% tokens) but does **not** improve task success rate. LLM-generated files can even *decrease* success rate (−0.5–3%) while increasing cost (+20%). Effective `AGENTS.md` is not "more context" — it is "precise context." This directly anchors the L0→L4 agent-readiness ladder.

---

## 🔬 Phase 1: EXTRACTION — What Was Found

### Key Concepts

- **AGENTS.md as "README for agents"** — Standard Markdown, no required schema, nested scope (closest file wins), cross-agent compatible (Codex, Claude Code, Cursor, Gemini CLI, Aider, Qwen Code).
- **The Efficiency Paradox** — AGENTS.md makes agents *faster* and *cheaper* (fewer wasted steps guessing commands), but does **not** make them *more correct*. Extra context increases reasoning tokens without improving outcomes.
- **Signal vs Noise Framework** — High-signal sections (Agent Role, Tech Stack versions, Key Commands with full flags, Boundaries, Code Examples, Security Constraints) vs noise sections (Codebase Overview, Aspirational Guidelines, prose-only rules, duplicated docs).
- **Inferability Test** — If the agent can derive the info from `package.json`, tests, or configs, it does **not** belong in AGENTS.md.
- **Persistence Test** — If the instruction should not apply to 100% of future sessions, it belongs in a prompt, not in the persistent file.
- **Progressive Disclosure** — Agent Skills (DeerFlow 2.0) load only name + description at start; full instructions loaded on activation. Reduces context footprint.
- **Self-Maintaining Context** — Agent proposes file updates at end of session, but only generalizable principles, not bug-specific notes.

### Empirical Evidence

| Study | Key Metric | Finding |
|---|---|---|
| arXiv:2601.20404 (JAWs 2026) | Median runtime | ↓ 28.64% with AGENTS.md |
| arXiv:2601.20404 | Mean output tokens | ↓ 20.08% |
| arXiv:2602.11988 (ICML 2026) | Success rate (dev-written) | +4% vs baseline |
| arXiv:2602.11988 | Success rate (LLM-generated) | −0.5% (SWE-bench), −2% (AGENTbench) |
| arXiv:2602.11988 | Cost increase (LLM-generated) | +20% |
| arXiv:2602.11988 | Cost increase (dev-written) | +23% |
| arXiv:2602.11988 | Steps to completion | ↑ +3.5–4.6 (more exploration, same outcome) |

### High-Signal Sections (Include)

| Section | Why It Works |
|---|---|
| **Agent Role + Priorities** | Gives the agent a decision framework (e.g., "type safety > speed"). |
| **Tech Stack (with versions)** | Agent does not guess. Table format is scannable. |
| **Key Commands (full syntax)** | Copy-paste ready with flags, filters, env vars. |
| **Boundaries (Always / Ask First / Never)** | Prevents expensive mistakes ("Never commit to main", "Never touch .env"). |
| **Code Examples (from repo)** | "Show, don't tell." Real error-handling patterns. |
| **Security Constraints** | PII fields, auth flows, secrets management. |

### Pure Noise (Exclude)

| Section | Why It Hurts |
|---|---|
| **Codebase Overview** | Agent infers via `ls`, `find`, `grep`. Does not speed up file discovery. |
| **Aspirational Guidelines** | "We value clean code" — unverifiable, increases reasoning tokens. |
| **Commands without Flags** | Agent guesses options, wastes steps. |
| **Duplicated Docs** | Copies from README — violates SSoT, bloats context window. |
| **One-off Task Instructions** | Belongs in prompt, not persistent file. Creates pollution. |
| **Prose-only Rules** | Long paragraphs get "lost." Use tables, lists, three-tier boundaries. |

### Anti-Patterns & Fixes

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Vague persona | Agent doesn't know trade-offs | Concrete role + ordered priorities |
| Commands without flags | Agent guesses | Full syntax with flags |
| Prose-only rules | Hard to scan, easy to miss | Tables, lists, three-tier boundaries |
| No boundaries | Risky changes without control | Always / Ask First / Never |
| Stale examples | Agent follows outdated patterns | Quarterly review; update after refactors |
| Overly long file (>150 lines) | Agent loses focus | <150 lines; link to docs |
| Unmarked user decisions | Agent "improves" intentional choices | `<!-- BEGIN USER-SPECIFIED -->` blocks |

---

## 🧠 Phase 2: ANALYSIS — What It Means

### Strengths

- **Format is universal** — One file works across all major agents; no vendor lock-in.
- **Nested scope is powerful** — OpenAI uses 88 nested AGENTS.md files. Enables per-package context without root bloat.
- **Boundaries are the real guard rails** — "Never / Ask First" is more effective than complex policy-as-code for agent behavior.
- **Speed/cost savings are real** — Even without success-rate gains, ↓28% runtime and ↓20% tokens matter at scale.

### Weaknesses / Limitations

- **Success rate does not improve** — AGENTS.md is an efficiency tool, not a correctness tool. This limits how high it can push the maturity ladder.
- **LLM-generated files can hurt** — Autonomous generation without human review is risky. Our authoring sub-skill must gate file writes.
- **Redundancy hypothesis** — AGENTS.md helps most in *under-documented* repos. In well-documented projects it adds noise. Our classifier must detect documentation richness.
- **No strict schema** — Freedom is a double-edged sword. Quality varies wildly; "has AGENTS.md" ≠ "has good AGENTS.md."

### Comparison

| Criterion | AGENTS.md | CLAUDE.md | .cursorrules | SKILL.md (Agent Skills) |
|---|---|---|---|---|
| **Scope** | Repo-wide, nested | Repo-wide | Repo-wide | Task-specific, activated |
| **Format** | Free Markdown | Free Markdown | Free text | Structured (metadata + instructions + scripts) |
| **Disclosure** | Always loaded | Always loaded | Always loaded | Progressive (on activation) |
| **Agent support** | Universal (60k+ repos) | Claude-specific | Cursor-specific | DeerFlow 2.0+ |
| **Best for** | Persistent repo context | Anthropic agent tuning | Cursor agent tuning | Reusable, heavy instructions |

### Open Questions

- [ ] How does our classifier detect "documentation richness" to avoid advising AGENTS.md where README + configs already suffice?
- [ ] What is the exact quality threshold that distinguishes L1 (Oriented) from "has file but it's noise"?
- [ ] Should our authoring sub-skill generate a *single* root AGENTS.md or a *topic-split* set (core + docs/agent-*.md) by default?
- [ ] How do we validate that LLM-generated AGENTS.md passes the Inferability Test and does not harm performance?

---

## 🛠️ Phase 3: APPLICATION — How to Apply to Agent Harness Skill

### Current Work (Our Project)

| Plan Phase | Impact of This Research |
|---|---|
| **P1 — repo-classifier** | Add signal-quality assessment: not just "has AGENTS.md" but length, structure, has boundaries, has role, commands have flags. Detect documentation richness (README depth, config clarity) to avoid false L0→L1 advice. |
| **P2 — Walking Skeleton** | Advice for missing AGENTS.md must be the *minimal viable template* (~30–50 lines), not a comprehensive guide. Focus on non-inferable decisions only. |
| **P3 — Maturity Engine** | L1 "Oriented" = effective AGENTS.md (passes quality checks), not just "file exists." Define L2 "Navigable" = topic-based splitting or nested per-package context. L3 "Guarded" = explicit Always/Ask/Never boundaries. |
| **P4 — Authoring sub-skill** | Must follow strict template: Role + Tech Stack + Key Commands + Boundaries + Code Examples + Security. Must exclude noise sections. Must enforce <150 lines. Must gate writes (human approval for generation). |
| **P5 — Nest + Drift** | Topic-based splitting: root AGENTS.md (core) + `docs/agent-*.md` (testing, deployment). Nested AGENTS.md per package in monorepos. Drift detection checks for stale examples and aspirational bloat. |
| **P6 — Autonomous mode** | File-write policy is **critical** — LLM-generated files can decrease success rate. Overwrites always gated. Updates append/refine; never full rewrite without human sign-off. |
| **P8 — Guard Rails** | Boundaries (Always/Ask/Never) map directly to mode escalation: "Ask First" → guide mode gate; "Never" → autonomous mode hard stop. |

### Key Design Changes Triggered

1. **L1 definition tightens** — "Has root AGENTS.md" is insufficient. L1 = "Has root AGENTS.md that passes quality heuristics (role, boundaries, commands with flags, <150 lines, no noise)."
2. **LLM-generation is risky** — Our authoring sub-skill cannot autonomously write AGENTS.md in autonomous mode. It can *draft* in guide mode; human must approve.
3. **Documentation richness check** — Before advising "create AGENTS.md," the classifier must check if README + configs already provide sufficient inferable context. If yes, the repo may already be L1-equivalent without the file.
4. **Nested context is L2, not L1** — A single root file is L1. Topic-split or nested per-package is L2. This gives us a clean two-step progression.

---

## ✅ Phase 4: ACTION — What Next

### Next Steps

- [ ] Integrate quality heuristics into the L0→L4 ladder definition (update RFC + P0).
- [ ] Draft the minimal viable AGENTS.md template for the authoring sub-skill (P4).
- [ ] Add "documentation richness" signals to the classifier's 10 axes or as a meta-signal.
- [ ] Define the topic-based splitting strategy for monorepos (P5).
- [ ] Validate the minimal template against the test corpus: does it actually help (speed/cost) without hurting (success rate)?

### Related Notes

- [[Agent Harness Skill — RFC]] — architecture spec; needs L1/L2 definition update.
- [[Agent Harness Skill — Implementation Plan]] — build plan; P3/P4/P5 affected.
- [[P0 — Foundations Decisions]] — contracts + ladder; needs study block closure.
- [[Repo Harness Skill — RFC v4 Architecture]] — ancestor dual-axis spec.

### Status History

- `2026-06-09`: researching — collected sources, read empirical papers, analyzed community practices.
- `2026-06-09`: analyzing — synthesized Signal/Noise framework, Inferability/Persistence tests, mapped findings to plan phases.

---

## 🗣️ Raw Research Notes (Original Russian)

*The original research notes (in Russian) covered the following areas in depth:*

1. **Format & Stewardship** — AGENTS.md under Agentic AI Foundation / Linux Foundation. 60k+ repos. Nested scope. Override hierarchy.
2. **Ecosystem** — Codex, Claude Code, Cursor, Gemini CLI, Aider, Qwen Code. Alternatives: CLAUDE.md, SKILL.md, .cursorrules.
3. **Community Best Practices** — Michael Nigh's analysis of 2500+ repos (GitHub Blog 2025). Builder.io imperative-style guidelines.
4. **Empirical Studies** — arXiv:2601.20404 (JAWs 2026) and arXiv:2602.11988 (ICML 2026). Runtime, tokens, success rate, cost.
5. **Signal/Noise Breakdown** — Detailed table of high-signal vs noise sections with evidence.
6. **Minimal Context Principles** — Inferability Test, Persistence Test, Topic-Based Splitting, Progressive Disclosure, Self-Maintaining Context.
7. **Future Directions** — YAML frontmatter, AGENTS.db (SQLite + MCP), SKILL.md / Agent Skills.
8. **Anti-Patterns** — 7 common mistakes with fixes.
9. **Audit Checklist** — 7-point checklist for reviewing AGENTS.md quality.
10. **Minimal Viable Template** — Recommended ~30-line template with Role, Tech Stack, Key Commands, Boundaries, Code Examples, Security.

*For full detail, see the user's original research document (shared in conversation).*