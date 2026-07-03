---
status: researching
tags: [research, in-progress, agent-harness, agent-readiness, rfc, decisions, p0]
created: 2026-06-09
updated: 2026-06-09
revision: 4
source:
  title: Agent Harness Skill — P0 Foundations Decisions
  author: self-directed design session
  date_read: 2026-06-09
---

# Agent Harness — P0 Foundations: Decisions & Findings

> [!abstract] TL;DR
> P0 decisions locked. **Scope = agent-readiness only** (deployment is a future sibling skill — see [[Agent Harness Skill — RFC]]). **Artifact form = pure skill (no scripts).** Classification and gap analysis are done by an LLM, not deterministic code, because repo setups are heterogeneous, evolving, and combinatorial. The three contracts (`project-profile`, `capability`, `roadmap`) are **not files** — they are **structured agent outputs** emitted into the conversation so downstream sub-skills can consume them. Each schema field carries an LLM-facing explanation ("what to look for, what to emit"). Interview-question criteria are defined explicitly.
>
> 📄 **Architecture:** [[Agent Harness Skill — RFC]] · 📋 **Build plan:** [[Agent Harness Skill — Implementation Plan]]

---

## Decision 1 — Artifact Form: Pure Skill (no scripts)

**Chosen:** Skill format (markdown instructions + LLM reasoning). **No deterministic scripts.**

**Rationale:**
- Repo setups are **heterogeneous and evolving**. Any given combination of infra/framework/tooling can be arbitrary. A deterministic scanner would need endless special-casing and would rot as ecosystems shift.
- Classification is fundamentally a **judgment task**, not a parsing task. "Is this a microservices repo or a modular monolith?" is rarely answerable by a single manifest grep — it needs holistic reading of signals.
- We deliberately want this step **as non-deterministic as possible** — the intelligence lives in the LLM, not in frozen rules.

**Implications:**
- The skill is a set of instructions that guide an LLM to **read the repo, reason, and emit structured output**.
- No JSON Schema validators, no scanner binaries, no config parsers as code.
- Determinism, where needed (e.g., reproducibility of output shape), comes from **strict output contracts in the prompt**, not from code.
- Trade-off accepted: output is only as reliable as the model. We mitigate with explicit per-field guidance and interview fallback (see below).

---

## Decision 2 — Contracts Are Agent Outputs, Not Files

The three contracts from the RFC stay, but they are reframed:

- They are **NOT** separate `.json` / `.schema.json` files on disk.
- They are **structured outputs an agent emits into the chat/conversation** at the end of each step.
- Downstream steps / sub-skills **read the prior output from the conversation window** and continue from it.
- Therefore each contract is documented as a **shape + per-field LLM instructions**: for every field we explain *what the LLM must look for in the repo* and *what it must emit*.

This keeps the whole pipeline in-context and serializable through the conversation, with no filesystem coupling.

---

## Contract A — `project-profile` (output of `repo-classifier`)

The classifier reads the repo and emits this. It is the foundation for everything downstream.

```yaml
project_profile:
  classification:        # the 10 axes (see per-axis guidance below)
    - axis_id: string
      value: string | "undetermined"
      detected: detected | undetermined | ambiguous
      confidence: high | medium | low
      signals: [ ... ]            # evidence the LLM used
      reasoning: string           # short "why I concluded this"
  interview_questions:            # generated from undetermined/ambiguous axes
    - axis_id: string
      question: string
      why_it_matters: string
      options_hint: [ ... ]       # likely answers to make it a one-tap choice
```

### Per-field LLM guidance

| Field | What the LLM must do |
|---|---|
| `axis_id` | Use the fixed axis identifiers (table below). All 10 axes MUST appear in every profile, even if `undetermined`. |
| `value` | The concrete classification for the axis (e.g., `kubernetes`, `monorepo`, `ssr`). If you cannot determine it from evidence, emit exactly `"undetermined"`. Never guess to fill a blank. |
| `detected` | `detected` = clear evidence; `ambiguous` = conflicting/partial evidence (e.g., both Vite and Webpack configs present); `undetermined` = no evidence at all. |
| `confidence` | Honest self-assessment. `low` confidence on a `detected` value should still trigger a confirmation interview question. |
| `signals` | List the actual files/manifest entries you read (e.g., `next.config.js`, `dependencies.next`, `turbo.json`, `k8s/*.yaml`). This is the audit trail — it makes the recommendation explainable (RFC invariant #4). |
| `reasoning` | One or two sentences: "Found `turbo.json` + multiple `packages/*` → monorepo." Keep it terse but falsifiable. |

### The 10 axes (fixed IDs) and what to look for

| `axis_id` | Priority | What the LLM looks for (signals) |
|---|---|---|
| `infra` | 🔴 Critical | `Dockerfile`, `docker-compose.yml`, `k8s/`, `*.tf` (Terraform), `serverless.yml`, Vercel/Netlify config, GitOps (`argocd`, `flux`). Distinguish Docker vs K8s vs serverless vs static host. |
| `backend_arch` | 🔴 Critical | Service directory count, separate deployables, API gateways/BFF, service mesh configs, single app vs many. Monolith / modular-monolith / microservices / BFF / mesh. |
| `code_org` | 🔴 Critical | `turbo.json`, `nx.json`, `pnpm-workspace.yaml`, `lerna.json`, `packages/`/`apps/` layout. Monorepo vs polyrepo. |
| `client_server` | 🔴 Critical | `next.config.js` (SSR/SSG), pure SPA (CRA/Vite SPA), MPA, PWA manifest/service worker. |
| `delivery_rendering` | 🟡 Important | SSG vs CSR vs ISR vs Edge vs RSC. Next.js `app/` (RSC), `getStaticProps` (SSG), edge runtime configs. |
| `frontend_scaling` | 🟡 Important | Module Federation config, micro-frontend orchestration, `@module-federation/*`, multiple independent FE apps. |
| `auth_connectivity` | 🟡 Important | JWT libs, OIDC/OAuth providers (`next-auth`, `passport`, `@auth0`), BFF auth pattern, session strategy. |
| `interaction_protocols` | 🟢 Optional | REST (Express/Fastify routes), GraphQL (`apollo`, `graphql`), gRPC (`.proto`), WebSockets (`socket.io`, `ws`). |
| `data_model_sync` | 🟢 Optional | CRDT libs (`yjs`, `automerge`), optimistic UI, SWR/React Query, local-first signals. |
| `build_runtime` | 🟢 Optional | `vite.config.*`, `webpack.config.*`, ESM/`"type": "module"`, bundler/runtime (Bun/Node/Deno). |
| `agents_md_quality` | 🔴 Critical (meta) | Assess root `AGENTS.md`: existence + quality. Signals: length (<150 lines?), has `## Role`/`## Boundaries`/`## Key Commands` sections? Commands have full flags? Prose-only or table-based? Aspirational guidelines present? See `agents_md_quality` heuristics in Contract C. |
| `doc_richness` | 🟡 Important (meta) | README depth, inline comments, config clarity. Used to avoid advising AGENTS.md where README + configs already provide sufficient inferable context (redundancy hypothesis). |

> **Rule:** Detection gap = interview question. Any axis that ends up `undetermined` or `ambiguous` (or `detected` with `low` confidence) MUST produce an entry in `interview_questions`.

---

## Investigation Playbook (the `repo-classifier` core)

> [!important] The framing
> The real choice is **not** "scripts vs prose." It is **"vague instruction vs disciplined investigation procedure."** We avoid scripts for the *interpretation* of a repo (that is a judgment task → LLM), but we do not hand the model a flat "figure it out" prompt. We give it a **phased investigation playbook**: gather facts deterministically (with standard read/grep/glob tools), interpret them with judgment (heuristics below), and escalate everything uncertain to interview questions.

### The two things the skill must keep separate

1. **Fact-gathering — deterministic.** "Does `Dockerfile` exist?", "what is in `dependencies`?" — answered by tools (read/grep/glob), in a fixed order. This is *not* a script we maintain; it is plain tool use. Determinism here is good and must be preserved.
2. **Fact-interpretation — judgment.** "These signals → microservices, not a modular monolith." This is what the LLM owns, guided by the axis heuristics (not rigid rules).

The skill instructs the model: *gather facts deterministically (this is how and in what order), interpret them with judgment (these are the heuristics and how to explain your conclusion).*

### Why discipline replaces determinism

A scaffolder (Backstage/Nx) earns trust through **reproducibility**: same repo → same verdict. Dropping code sacrifices that. The playbook's discipline is what buys it back: because every `value` MUST be backed by recorded `signals`, two independent runs converge on the same evidence — and any genuine ambiguity is forced into `interview_questions` rather than a silent guess.

### The phases

**Phase 0 — Orient.**
Read the cheapest high-signal artifacts first: `README`, root manifest (`package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml`), and the top-level directory layout. Form an initial hypothesis about the project class. Do not classify yet.

**Phase 1 — Cheap signals (manifests).**
Walk the 10 axes **in priority order (🔴 → 🟡 → 🟢)**. For each axis, look for the signals listed in the axis table above — treat them as **heuristics, not rules**. Record what you find into `signals`. Most axes should resolve here.

**Phase 2 — Targeted deep-read.**
ONLY for axes that came out `ambiguous` or `low` confidence in Phase 1, open the specific config files needed to disambiguate. Do **not** read the whole repo. Read with a question in mind (e.g., "Vite or Webpack?" → open the bundler config, not everything).

**Phase 3 — Synthesize & self-check.**
Fill the `project-profile`. For every one of the 10 axes emit `value` + `signals` + `reasoning`. Self-check: is any `value` set without a corresponding `signals` entry? If yes, downgrade it to `undetermined` — never keep an unbacked guess.

**Phase 4 — Escalate.**
Route everything still `undetermined` / `ambiguous` / `detected`-but-`low` into `interview_questions`, following the Interview Question Criteria (below).

### Hard rules layered over the phases

- **Evidence-first (load-bearing, not optional).** A `value` may not be set without a recorded `signals` entry. No evidence → `undetermined`, not a guess. This is the mechanism that makes runs converge and keeps the tool explainable.
- **Stop conditions.** Each axis has a "good enough" point: once a decisive signal is found, stop digging on that axis (e.g., `turbo.json` + `packages/*` → `monorepo`, done).
- **Detect beats ask.** Never raise an interview question for something already evidenced at `high` confidence. Ask only what the repo genuinely cannot reveal.
- **Prioritize budget.** Spend deep-read effort on 🔴 Critical axes first; a `low`-confidence 🟢 Optional axis can simply ship as an interview question without deep-reading.

### Accepted residual risk

Even with the playbook, two runs may diverge on genuine edge cases. This is tolerable **only because** divergence is forced into `interview_questions` (via evidence-first), never into a silent differing guess. If we ever observe silent divergence, evidence-first is being violated.

---

## Contract B — `capability` (output of `capability-registry`)

A tool described once, with a **single projection: the agent feedback loop**. (v4 had a second "deploy gate" projection — dropped with the deploy axis. There is now exactly one consumer, so the dual-projection model collapses to this.) Emitted per capability the repo needs or has.

```yaml
capability:
  capability_id: string          # lint | typecheck | test | format | ...
  provision:
    install: [ ... ]
    config_path: string | "none"
    tool: string                 # concrete tool detected/recommended
  feedback:                       # the agent self-check loop (module C)
    command: string              # how the agent runs it
    output_format: string        # how the agent parses results
    fix_command: string | null   # auto-fix command if one exists
```

### Per-field LLM guidance

| Field | What the LLM must do |
|---|---|
| `capability_id` | Canonical slug for the capability. Stable across repos so downstream sub-skills can match. |
| `provision.install` | The command(s)/packages that install it. If already present, record what is installed (read lockfile/`package.json`). |
| `provision.config_path` | Path to the tool's **real** config file (e.g., `.eslintrc.json`, `tsconfig.json`). `"none"` if not configured yet. The agent references this file; it never forks or embeds a copy. |
| `provision.tool` | The concrete tool (e.g., `eslint`, `tsc`, `vitest`, `prettier`). Detect from manifests; recommend if missing. |
| `feedback.command` | The exact command the agent runs for self-check (e.g., `npm run lint`, `tsc --noEmit`). |
| `feedback.output_format` | How an agent reads results for self-correction — e.g., `eslint --format json`, `tsc` diagnostics, `vitest --reporter=json`. This IS the feedback loop. |
| `feedback.fix_command` | Auto-fix command if one exists (`eslint --fix`, `prettier --write`), else `null`. |

> **Invariant (RFC #6):** The agent references the tool's **real config** (`config_path`); it never embeds a forked copy. This is the slimmed-down remnant of v4's single-source rule — now about agent↔repo consistency, not gate↔loop drift.

---

## Contract C — `roadmap` (output of `gap-engine`)

Compares the profile against the target for the next maturity level and emits a prioritized plan with rationale.

```yaml
roadmap:
  current_level: L0 | L1 | L2 | L3
  target_level: L0 | L1 | L2 | L3          # the NEXT reasonable level, not final
  items:
    - module_id: string
      capability_id: string | null
      status: present | partial | missing
      priority: critical | important | optional
      rationale: string          # why this matters for THIS project class
      next_step: string          # concrete human-readable action
```

### Per-field LLM guidance

> **Ladder decided (2026-06-09):** 4 levels, empirically anchored in the failure-mode catalogue. Each level removes a class of *observed* failure, not an invented criterion. See [[Catalogue of Observed Agent Failure Modes — Study Block]]. The old 5th rung ("Navigable") folded into L1 — it was just good context for a complex repo, not a distinct stage.

| Field | What the LLM must do |
|---|---|
| `current_level` | Infer from what already exists, using the **agent-readiness ladder**: **L0 Blind** = no root `AGENTS.md` or file exists but is noise (>150 lines, no boundaries, no role, aspirational prose); **L1 Oriented** = effective `AGENTS.md` (passes quality heuristics: Agent Role + Priorities, Tech Stack versions, Key Commands with full flags, Boundaries [Always/Ask/Never], <150 lines, no noise per Inferability Test), scaled to **nested** per-package for monorepos, **plus** a repo map + conventions so the agent can locate work — removes Code-layer failures (~5%); **L2 Self-Correcting** = feedback loop wired (lint/typecheck/test + architecture fitness as agent-runnable self-checks with parseable output + fix commands) **and** boundaries enforced (protected paths) — removes the largest addressable failure, CI/test (17%); **L3 Self-Maintaining** = drift detection/resync + advanced tooling (MCP/hooks/slash-commands) — prevents context poisoning / stale-context misdirection. |
| `agents_md_quality` | Assess the root `AGENTS.md` if present: `effective` (passes heuristics), `bloated` (noise sections, >150 lines, missing boundaries), `minimal` (has role + commands but lacks boundaries or security), or `none`. Prevents false L1 classification when a file exists but hurts performance. |
| `target_level` | Always the **next** reasonable level up from current — never jump to L3 on a greenfield. Protects against over-engineering (RFC invariant #2). |
| `module_id` | Which agent-harness module the gap belongs to: `context` (A), `navigability` (B), `feedback` (C), `tooling` (D), `boundaries` (E). |
| `status` | `present` = exists and adequate; `partial` = exists but incomplete/misconfigured/stale; `missing` = absent. |
| `priority` | Weight by axis priority and level requirement. Critical-axis gaps at the target level → `critical`. |
| `rationale` | MUST explain why this matters **for this specific project class** — not generic advice. This is the explainability invariant (RFC #4). Generic rationale = a bug. |
| `next_step` | One concrete action the user (or skill in guide/autonomous mode) can take. |

---

## Interview Question Criteria

The classifier falls back to asking the user when it cannot decide. An interview question MUST be generated when **any** of these hold:

1. **No evidence** — axis is `undetermined` (no signals found at all).
2. **Conflicting evidence** — axis is `ambiguous` (e.g., both `vite.config.ts` and `webpack.config.js`, or both SSR and pure-SPA signals).
3. **Low confidence** — axis is `detected` but `confidence: low`; confirm rather than assume.
4. **Decision-defining ambiguity** — the axis drives a 🔴 Critical downstream choice (e.g., `infra`, `backend_arch`) and the evidence is anything less than `high` confidence.

### What makes a GOOD interview question

- **One decision = one question.** Never bundle two axes. (Progressive disclosure.)
- **Closed over open.** Provide `options_hint` so the user taps a choice instead of writing prose.
- **Carries `why_it_matters`.** The user must understand the downstream impact (e.g., "This determines whether we generate one root `AGENTS.md` or nested per-package context").
- **Self-contained.** No assumed prior context; readable standalone.
- **Only ask what you cannot detect.** Detection always beats asking. Never ask about something already evidenced with high confidence.

### What makes a BAD interview question (avoid)

- Bundling multiple axes into one prompt.
- Open-ended "tell me about your architecture" prompts.
- Asking something already detectable from manifests.
- Missing rationale ("why are you asking me this?").

---

## Open Items Carried Forward

- **Scope pivot (2026-06-09):** project narrowed from dual-axis (agent + deploy) to **agent-readiness only**. Deploy becomes a future sibling skill. Consequences captured in [[Agent Harness Skill — RFC]]: 3-package topology → single skill; capability dual-projection → single agent-feedback projection; "bridge" sub-skills → ordinary sub-skills.
- **Scope boundary locked (2026-06-09):** scope = **repo configuration for AI-agent convenience** (context `AGENTS.md`, feedback loop, architecture enforcement). Explicitly **out of scope**: reviewer abandonment (38% — social, no repo fix); project governance (`CONTRIBUTING`/`ROADMAP`/PR templates/duplicate-PR prevention → candidate **sibling skill**); agent-capability assessment. We own only the *missing-affordance* side of Code + System failure layers. See [[Catalogue of Observed Agent Failure Modes — Study Block]].
- **Ladder simplified (2026-06-09):** 5 levels → **4** (L0 Blind / L1 Oriented / L2 Self-Correcting / L3 Self-Maintaining). Old "Navigable" folded into L1. Empirically anchored; RFC open question #1 resolved.
- **Study Block** — owner: **user**. ✅ AGENTS.md ([[AGENTS.md Spec & Agent Failure Modes — Study Block]]); ✅ context-engineering ([[Context Engineering — Study Block]]); ✅ failure-mode catalogue ([[Catalogue of Observed Agent Failure Modes — Study Block]]). Dropped: Cursor/aider conventions (env-specific tooling, low value). Remaining/optional: scaffolder prior art (Backstage/Nx) for UX inspiration only — not blocking.
- **Test corpus** — still to assemble (5–6 repos of varying maturity). Now the main blocker before P1.

---

## P0 Exit Criteria — Status

- [x] Artifact form decided → **pure skill, no scripts**.
- [x] Three contracts defined as **agent outputs** with per-field LLM guidance.
- [x] Investigation Playbook drafted (phased fact-gathering + judgment + evidence-first).
- [x] Interview-question criteria defined.
- [x] Study block — **core complete** (AGENTS.md + context-engineering + failure-mode catalogue). Ladder anchored, #1 resolved. Scaffolder UX study optional/non-blocking.
- [ ] Test corpus — **now the main blocker** before P1.

---

## Related Documents

- [[Agent Harness Skill — RFC]] — architecture spec (agent-only scope).
- [[Agent Harness Skill — Implementation Plan]] — phased build plan.
- [[Catalogue of Observed Agent Failure Modes — Study Block]] — empirical anchor for the 4-level ladder.
- [[AGENTS.md Spec & Agent Failure Modes — Study Block]] — `AGENTS.md` quality heuristics.
- [[Context Engineering — Study Block]] — positional bias, progressive disclosure, token budget.
- [[Repo Harness Skill — RFC v4 Architecture]] — dual-axis ancestor (deploy + agent).
- [[Repo Harness Skill — Step-by-Step Implementation Plan (RFC v4)]] — dual-axis build plan.

---

*Last updated: 2026-06-09 (revision 4 — study block core complete; 4-level ladder anchored in failure-mode catalogue; scope boundary locked, governance + reviewer-abandonment out of scope)*
