# Wave 4 — Integration & Portfolio

> Canonical detailed description for Wave 4 of the [[Requirements-Engineering-Roadmap|Requirements Engineering Toolkit]].

**Status:** Not started
**Window:** Week 7-8 (Aug 15 – Aug 28, 2026)
**Budget:** ~12h (3h study + 6h build + ~3h portfolio packaging)

---

## Goal
On exit you can:
- Wire 6 skills into a coherent pipeline.
- Reuse the same knowledge layer across 2+ harnesses with thin adapters.
- Hand the toolkit to a stranger and they understand it in 10 minutes.

---

## Topics to master
1. Orchestrator patterns: linear pipeline, fan-out/fan-in, human-in-the-loop checkpoints.
2. Skill invocation protocol (input/output contract between skills).
3. Harness portability: markdown knowledge layer vs runtime-specific glue.
4. Capstone demo: end-to-end run on a fresh project.
5. Technical writing for portfolio: README, architecture diagram, design writeup.

---

## Study materials (~3h core)

| Source | What to read | Time | Mark |
|---|---|---|---|
| Claude Code `SKILL.md` authoring guide (official docs) | full | 1h | CANON |
| LangGraph tutorial — basic agent orchestration | quickstart | 1h | CANON |
| Michael Nygard — "ADR" original paper + ThoughtWorks ADR template | canonical primer | 30m | CANON |
| n8n AI Agent node documentation | reference | 30m | CANON |

**Reference:**
- Existing public skill collections (Anthropic, community) — study structure, not copy.

---

## Distillation contract — Wave 4

### Orchestrator + Adapter

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

### Portfolio packaging

**Acceptance criteria — pass when:**
- [ ] README explains: what / why / how to use / how to extend.
- [ ] Architecture diagram shows pipeline + adapter pattern.
- [ ] Capstone run has 6 artifacts demonstrating the full lifecycle.
- [ ] Writeup explains portability claim with concrete example.

---

## Wave 4 progress
- [ ] Orchestrator design
- [ ] Harness adapter (Claude Code → LangGraph → n8n)
- [ ] Capstone case (end-to-end)
- [ ] Toolkit README
- [ ] 2-3 demo artifacts
- [ ] Architecture diagram + writeup
