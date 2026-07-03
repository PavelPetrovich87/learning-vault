---
status: analyzing
mastery: beginner
tags: [agentic-engineering, harness-engineering, architectural-constraints]
source: OpenAI Blog (Harness Engineering)
created: 2026-03-18
updated: 2026-07-03
difficulty: advanced
---

# 🧠 Topic: Architectural Constraints (Harness Engineering)

> [!abstract] Executive Summary (ELI5)
> **Architectural Constraints** are physical guardrails—like strict linters and architectural tests—built into the CI/CD pipeline to mechanically block an AI agent from writing bad or unstructured code. Instead of just telling the AI "be good", we build systems that reject bad code and automatically tell the AI how to fix it.

---

## 🎯 The "Why"
- **Problem Solved:** AI agents are highly prone to making "local optimizations" (e.g., fixing a bug in the fastest way possible by bypassing layers) without "global awareness" (understanding how this breaks the overall system architecture). This rapidly leads to spaghetti code.
- **The Shift:** We must stop relying on prompts (e.g., "Please use hexagonal architecture") and start relying on deterministic, mechanical enforcement in the pipeline.
- **Trade-offs:** Increases the strictness and upfront complexity of the build pipeline. Human developers might find these constraints pedantic or annoying, but they are absolutely necessary to keep automated agents in check.

## 💡 Mental Model & Analogy
- **Analogy:** **Bowling Lane Bumpers.** The agent is throwing the bowling ball. You don't try to teach the agent to have perfect form; instead, you put up steel bumpers (architectural constraints) so that no matter how the ball is thrown, it bounces toward the pins.

## 🛠️ Core Principles & Mechanisms

### 1. Deterministic Linters with Remediation
When an agent breaks a rule, standard linters just output "Error: Rule X violated." A harness-engineered linter outputs the error **and the specific remediation instructions**.
- *Example:* Instead of `Error: Direct DB access in Controller`, the linter outputs: `Error: Controller accessing DB. Remediation: Move this query to the Repository layer and inject the Repository into the Controller.` This is fed straight back into the agent's context window.

### 2. Structural Tests
Automated tests that verify the dependency graph and folder structure of the codebase, not just the logic.
- Using tools to enforce that the `frontend/` folder cannot import from the `backend/database/` folder.
- Enforcing that `Entities` do not depend on `Infrastructure` components (e.g., Hexagonal Architecture constraints).

### 3. Layered Domain Architecture
OpenAI uses a rigid architectural model within each business domain:
`Types → Config → Repo → Service → Runtime → UI`
- Code can only depend "forward" through these layers.
- Cross-cutting concerns (auth, connectors, telemetry, feature flags) enter through a single explicit interface: **Providers**.
- Any other dependency direction is disallowed and **mechanically enforced**.
- This is the kind of architecture teams usually postpone until they have hundreds of engineers. With coding agents, it's an **early prerequisite** — constraints are what allow speed without decay.

### 4. The "Enforce Centrally, Allow Autonomy Locally" Principle
Like leading a large platform org: enforce boundaries, correctness, and reproducibility centrally. Within those boundaries, allow agents significant freedom in how solutions are expressed. For example: enforce boundary parsing with Zod-like validation at data edges, but don't prescribe which specific library to use.

### 5. Merge Philosophy Shift
When agent throughput far exceeds human attention:
- **Minimal blocking merge gates.** PRs are short-lived.
- **Corrections are cheap; waiting is expensive.** Test flakes are addressed with follow-up runs rather than blocking progress.
- This would be irresponsible in a low-throughput environment. With high agent throughput, it's often the right tradeoff.

> [!tip] Implementation Note
> Every time an agent breaks an architectural boundary, your reaction should be: "Why didn't the pipeline catch this?" You then add a structural test so that specific mistake becomes impossible to merge in the future.

## ⚠️ Pitfalls & Anti-patterns
- **"Vibes-based" Architecture:** Relying on code reviews or prompt engineering to maintain clean architecture, rather than automated, deterministic tests. "Human taste" that isn't encoded into tooling doesn't scale.
- **Unhelpful Errors:** Having a pipeline that fails with cryptic stack traces, leaving the agent guessing how to fix the issue.
- **Over-blocking CI:** Blocking merges on every flake when corrections are cheap and throughput is high. In agent-first environments, waiting often costs more than fixing forward.

## 🔗 Deep Connections
- **Parent Topic:** [[Harness Engineering]], [[Clean Architecture]]
- **Related Concepts:** [[Code Quality as AI Infrastructure]]

## ❓ Active Recall Questions
> [!faq]- In **Harness Engineering**, why are "Structural Tests" critical for managing AI agents?
> Because agents often make local optimizations that violate global architecture (resulting in spaghetti code). Structural tests mechanically enforce modular boundaries (e.g., preventing a Controller from importing a DB driver directly).

> [!faq]- How should a linter be modified to be "Agent-Friendly" within a harness?
> Instead of just outputting the failure (e.g., "Rule violated"), the linter should dynamically inject **specific remediation instructions** directly into the agent's context window so it knows exactly how to fix the error.
