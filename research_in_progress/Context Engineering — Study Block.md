---
status: analyzing
tags: [research, agent-harness, context-engineering, llm-context, progressive-disclosure, lost-in-middle, in-progress]
created: 2026-06-09
updated: 2026-06-09
source:
  title: "Context Engineering: Structural Practices for LLMs in 2026"
  author: self-directed research
  date_read: 2026-06-09
---

# Context Engineering — Study Block

> [!abstract] TL;DR
> Comprehensive survey of context engineering as a discipline distinct from prompt engineering. Key insight: **context is a finite attention budget**, not an expandable container. Empirical findings validate our minimal-AGENTS.md approach and provide architectural foundations for the L0→L4 ladder: "Lost in the Middle" (U-shaped attention) enforces short context files with critical info at the top; Progressive Disclosure supports topic-based splitting (L2); Context Rot / Poisoning validates drift detection (L4); Markdown is the most token-efficient format (−34% vs JSON); 5–7 tools is the performance sweet spot.

---

## 🔬 Phase 1: EXTRACTION — What Was Found

### Key Concepts

- **Context as Finite Budget** — Every token competes for limited attention. Irrelevant tokens dilute signal. Teams mastering context engineering see **40–70% API cost reduction** and **2–3x task completion improvement** [(inkeep)](https://inkeep.com/blog/context-engineering-why-agents-fail).
- **Four Pillars** — Sourcegraph/Anthropic framework: (1) Instructions / System Prompt, (2) Retrieval, (3) Memory, (4) Tools. Our AGENTS.md maps to Pillar 1 (Instructions); our Investigation Playbook maps to Pillar 2 (Retrieval).
- **Lost in the Middle** — Liu et al. (Stanford/UC Berkeley). LLMs exhibit a **U-shaped attention curve**: information at the start (primacy) and end (recency) is retained; middle content is statistically lost. With only 20 retrieved documents (~4K tokens), accuracy drops from 70–75% to **55–60%** when relevant info is in the middle [(ACL Anthology)](https://aclanthology.org/2024.tacl-1.9/).
- **Context Rot** — Systematic quality degradation as context grows. Attention dilution + zero-sum competition + sudden performance cliffs (often before advertised limits, e.g., 130K real vs 200K claimed) [(Zylos)](https://zylos.ai/research/2026-01-19-llm-context-management/).
- **Context Poisoning** — Hallucinations or errors enter long-term memory / conversation history and are cyclically cited as fact, creating positive feedback loops. Fix: structured note-taking with explicit verified vs. speculative markers [(dbreunig)](https://www.dbreunig.com/2025/06/26/how-to-fix-your-context.html).
- **Progressive Disclosure** — UX pattern applied to LLMs: load context in phases (Index → Scout → Deep-read) rather than upfront. Anthropic: "progressive disclosure through exploration." Gives **40–70% cost reduction** [(mindstudio)](https://www.mindstudio.ai/blog/progressive-disclosure-ai-agents-context-management/).
- **Hierarchical Structuring** — LongRefiner (ACL 2025): XML-like hierarchical document representation reduces tokens to **1/10** while preserving structure. Hierarchy-aware chunking improves RAG accuracy by **+14.8%** [(contextual.ai)](https://contextual.ai/blog/document-parser-for-rag/).

### Format Efficiency (Nested Data)

| Format | Token Efficiency | Accuracy (GPT-5/Gemini) | Best Use |
|---|---|---|---|
| **Markdown** | **Highest** (−34% vs JSON) | Good | Documentation, instructions |
| **YAML** | Good (~+10% vs MD) | **Highest** | Complex nested structures, configs |
| **JSON** | Low (+34% vs MD) | Low | API integrations (Llama-agnostic) |
| **XML** | **Lowest** (+80% vs MD) | Low | Strict sectioning, legal docs |

> **Implication:** AGENTS.md as Markdown is optimal for token efficiency. Our skill instructions should also prefer Markdown over JSON/XML.

### Tool Count Sweet Spot

- **Each model performs worse with >1 tool** (Berkeley Function-Calling Leaderboard).
- **46 tools → failure** (Llama 3.1 8b experiment); **19 tools → success**.
- **Degradation beyond 5–10 tools per agent** (LangGraph research).
- **Solution:** Specialized sub-agents with 5–7 tools each + routing agent.

### Position Bias (Architectural Roots)

MIT 2025 identified two architectural causes:
1. **Causal Attention Masking** — Early tokens are visible to all later tokens; middle tokens are only visible to subsequent ones. Early tokens accumulate more attention weight.
2. **Positional Encoding Decay** — RoPE distance-based decay. Middle tokens are too far from start for primacy, too far from end for recency. They sit in a "dead zone."

### Optimal Prompt Structure (by Model)

| Model | Key Structuring Principle |
|---|---|
| **Claude** | XML tags (`<instructions>`, `<context>`, `<example>`) genuinely best. Aggressive language ("CRITICAL!", "YOU MUST") actively hurts. |
| **GPT-5** | Router-based (multiple models behind one endpoint). Zero-shot preferred; "think hard about this" literally triggers reasoning model. |
| **Gemini** | Always include few-shot. Place specific questions **at the end**, after data. Prefers shorter, direct prompts. |

### Priority Ordering (Universal)

1. **System instructions** — always first
2. **Critical facts & constraints** — at start or clearly marked
3. **High-priority retrieved docs** — after instructions
4. **Few-shot examples** — after context, before query
5. **Low-priority material** — at end, marked "optional"
6. **User query** — at the very end

---

## 🧠 Phase 2: ANALYSIS — What It Means

### Strengths

- **Validates our minimal-AGENTS.md stance** — Both studies (AGENTS.md + Context Engineering) converge: less but precise context beats more context.
- **Explains why stale AGENTS.md is worse than none** — Context poisoning: stale instructions become "verified facts" that the agent cyclically cites.
- **Gives architectural justification for L2 splitting** — Progressive disclosure + "Lost in the Middle" mean a single 300-line AGENTS.md is actively harmful. Topic-splitting (core + docs/agent-*.md) is not a convenience — it's a performance requirement.
- **Markdown choice validated** — AGENTS.md as Markdown is the most token-efficient choice.

### Weaknesses / Limitations

- **Most studies focus on RAG / chat, not repo context** — We are extrapolating from general context engineering to repo-specific context. The underlying attention mechanisms are identical, but repo context has different patterns (file trees, manifests, config files).
- **"Lost in the Middle" studies use retrieval tasks** — Our use case is instruction-following, not retrieval. The positional bias still applies, but the magnitude may differ.
- **Context compression research is nascent** — Adaptive compression (arXiv:2603.29193) works in labs but may not be practical for our skill-level implementation.

### Comparison with AGENTS.md Research

| Finding | AGENTS.md Study | Context Engineering Study | Combined Implication |
|---|---|---|---|
| **Length** | <150 lines optimal | Middle content is lost | **Hard ceiling: 150 lines.** Critical info (Role, Boundaries) at the **top**. |
| **Splitting** | Topic-based splitting | Progressive disclosure | **L2 = not optional.** Single large file is architecturally harmful. |
| **Format** | Standard Markdown | Markdown most token-efficient | **AGENTS.md as Markdown = optimal choice.** |
| **Stale content** | Stale examples anti-pattern | Context poisoning | **Drift detection (L4) is safety-critical.** Stale file actively misleads. |
| **Redundancy** | Inferability Test | Zero-sum attention budget | **If agent can infer it, it competes for attention.** Exclude it. |

### Open Questions

- [ ] Does the 150-line ceiling hold for all models, or is it model-specific? Claude 1M window vs Gemini 2M — does the "dead zone" scale?
- [ ] How do we detect "context poisoning" in a repo context? (Stale AGENTS.md being treated as ground truth.)
- [ ] Should our skill enforce a hard tool-count limit (5–7) when recommending capabilities?
- [ ] Does the priority ordering (system → critical → examples → query) apply to AGENTS.md internal structure? (Role first, then Boundaries, then Tech Stack, then Examples?)

---

## 🛠️ Phase 3: APPLICATION — How to Apply to Agent Harness Skill

### Current Work (Our Project)

| Plan Phase | Impact of This Research |
|---|---|
| **P1 — repo-classifier** | + Signal: tool count per agent (5–7 sweet spot, >10 bloated). + Signal: context complexity / file count that suggests topic-splitting need. + Signal: documentation richness (redundancy check). |
| **P2 — Walking Skeleton** | Advice must follow **priority ordering**: Role + Boundaries at the **top** of AGENTS.md (not bottom — they'd be lost in the middle). |
| **P3 — Maturity Engine** | L1 = effective AGENTS.md (<150 lines, Role + Boundaries at top, passes Inferability Test). L2 = topic-split = **progressive disclosure** (not just organization). L4 = structured note-taking + compaction + context poisoning detection. |
| **P4 — Authoring sub-skill** | Template must enforce: (1) Role first, (2) Boundaries second, (3) Tech Stack, (4) Key Commands, (5) Code Examples, (6) Security. Nothing after line 150. Markdown format confirmed. |
| **P5 — Nest + Drift** | Drift detection must check for **context poisoning** (stale instructions treated as fact). Compaction: summarize outdated sections rather than append. |
| **P6 — Autonomous mode** | Must not add tools beyond 5–7 per agent. Must not append to AGENTS.md indefinitely — compaction required. |
| **P8 — Guard Rails** | Boundaries at top = hard stops. "Ask First" = mode gate. Critical constraints must never be in the middle of the file. |

### Key Design Changes Triggered

1. **AGENTS.md internal ordering is load-bearing** — Not just "what sections to include" but **in what order**. Role + Boundaries must be at the top (positions 1–2 in the file) to survive positional bias. Tech Stack and Commands can follow. Examples and Security go later but before any "optional" material.
2. **L2 is architecturally required, not cosmetic** — "Lost in the Middle" + Context Rot mean a single 300-line file is harmful. Topic-splitting (core + agent-*.md) is the only way to keep each file under the attention threshold.
3. **Drift detection = poison detection** — Stale AGENTS.md doesn't just become "less useful" — it becomes actively misleading. Drift detection must validate facts against current codebase, not just check timestamps.
4. **Tool count is a maturity signal** — A repo with 20+ tool definitions in its agent config is exhibiting the same anti-pattern as bloated AGENTS.md. Classifier should flag this.

---

## ✅ Phase 4: ACTION — What Next

### Next Steps

- [ ] Update P0 / RFC to specify AGENTS.md **section ordering** (Role → Boundaries → Tech Stack → Key Commands → Code Examples → Security).
- [ ] Define "context poisoning" detection heuristic for drift module (P5): how does the agent verify that AGENTS.md instructions still match current codebase?
- [ ] Add `tool_count` and `context_complexity` signals to classifier (P1).
- [ ] Validate the 150-line ceiling empirically against test corpus — does a 200-line AGENTS.md perform measurably worse than 100-line?

### Related Notes

- [[AGENTS.md Spec & Agent Failure Modes — Study Block]] — empirical findings on AGENTS.md effectiveness.
- [[Agent Harness Skill — RFC]] — architecture spec; needs section-ordering update.
- [[Agent Harness Skill — Implementation Plan]] — build plan; P3/P4/P5 affected.
- [[P0 — Foundations Decisions]] — contracts + ladder; needs AGENTS.md ordering guidance.

### Status History

- `2026-06-09`: researching — read comprehensive context engineering survey.
- `2026-06-09`: analyzing — synthesized positional bias, progressive disclosure, format efficiency, and mapped to plan phases.

---

## 🗣️ Key Quotes for Reference

> "Context engineering is the discipline of deciding what information the model needs to know right now." — Sourcegraph

> "Every token you add has a cost — not just in API price, but in attention budget." — Inkeep

> "Early tokens are visible to all later tokens... Middle tokens sit in a dead zone." — MIT / Dev.to

> "46 tools → failure; 19 tools → success." — Llama 3.1 8b experiment

> "Adding tokens is a zero-sum game. Each relevant token competes with each irrelevant one." — Zylos
