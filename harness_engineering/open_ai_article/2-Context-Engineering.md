---
status: new
tags: [agentic-engineering, harness-engineering, context-engineering]
source: OpenAI Blog (Harness Engineering)
created: 2026-03-18
difficulty: intermediate
---

# 🧠 Topic: Context Engineering (Harness Engineering)

> [!abstract] Executive Summary (ELI5)
> **Context Engineering** is the practice of ensuring an AI agent has exactly the right information at the right time to make accurate coding decisions. It acts as the "eyes and ears" for the agent, preventing it from guessing or hallucinating about your system's state.

---

## 🎯 The "Why"
- **Problem Solved:** Models natively lack knowledge of your specific business logic, active database state, or runtime errors. Without context engineering, agents hallucinate APIs, invent nonexistent database tables, or fail to understand why their code is breaking in production.
- **Alternatives:** Manually copy-pasting logs, documentation, and file contents into the chat window every time you want the agent to do something.
- **Trade-offs:** Requires building sophisticated pipelines to fetch, filter, and format data so it fits efficiently within the LLM's context window without overwhelming it with noise.

## 💡 Mental Model & Analogy
- **Analogy:** **The New Hire Onboarding + Live Dashboard.** 
  - *Static context* is like giving a new engineer the company handbook and architecture diagrams on day one. 
  - *Dynamic context* is like giving them access to Datadog and AWS logs so they can see what's actually happening in production right now.

## 🛠️ Core Principles & Code Snippets

Context Engineering is divided into two primary types:

### 1. Static Context
Curated, persistent knowledge bases injected into the agent's prompt.
- **`AGENTS.md` as Table of Contents, NOT Encyclopedia:** OpenAI tried "one big AGENTS.md" and it failed — context is scarce, too much guidance becomes non-guidance, monolithic files rot, and they're hard to verify mechanically. Instead, AGENTS.md (~100 lines) serves as a **map with pointers** to deeper sources of truth in a structured `docs/` directory.
- **Progressive Disclosure:** Agents start with a small, stable entry point and are taught *where to look next*, rather than being overwhelmed up front.
- **Repository as System of Record:** Design docs, execution plans, quality grades, product specs, and references all live versioned in-repo. A dedicated `docs/` directory is the system of record, organized into `design-docs/`, `exec-plans/`, `product-specs/`, `references/`, etc.
- **Plans as First-Class Artifacts:** Execution plans with progress and decision logs are checked into the repository. Active plans, completed plans, and known tech debt are all versioned and co-located.

### 2. Dynamic Context
Giving agents read-only access to real-time, runtime data.
- **Observability Hooks:** Piping live logs, metrics (via PromQL/LogQL), and stack traces directly back to the agent when a test or deployment fails.
- **Environment Access:** Allowing the agent to query database schemas, execute read-only API calls, or use the Chrome DevTools Protocol to visually navigate the browser and see rendering issues.

### 3. The "Invisible Knowledge" Principle
> **What Codex can't see doesn't exist.**

From the agent's POV, anything not accessible in-context while running effectively doesn't exist. Knowledge in Google Docs, Slack threads, or people's heads is invisible. Only repository-local, versioned artifacts (code, markdown, schemas, executable plans) count.
- That Slack discussion aligning the team on an architectural pattern? If it isn't discoverable to the agent, it's as unknown as it would be to a new hire joining 3 months later.
- **Boring Tech Preference:** Technologies that are composable, API-stable, and well-represented in the training set are easier for agents to model. Sometimes it's cheaper to **reimplement** a subset of functionality than to work around opaque upstream behavior (e.g., building a custom concurrency helper with full OTel instrumentation instead of pulling in `p-limit`).

> [!tip] Implementation Note
> Don't just dump all logs into the context window. Build a middle layer that aggregates, summarizes, or filters the dynamic context (e.g., retrieving only the stack trace of the exact failing test) to conserve tokens and improve the agent's reasoning focus.

## ⚠️ Pitfalls & Anti-patterns
- **The "One Big AGENTS.md":** A monolithic instruction file crowds out the task and code, turns into a graveyard of stale rules, and can't be mechanically verified. Agents pattern-match locally instead of navigating intentionally.
- **The "Data Dump":** Flooding the context window with the entire repository or unparsed megabytes of raw logs, causing the LLM to lose the signal in the noise ("Lost in the Middle" phenomenon).
- **Stale Static Context:** Failing to update the `AGENTS.md` file when architectural patterns change, causing the agent to aggressively write outdated code. Fix: dedicated "doc-gardening" agents that scan for stale docs and open fix-up PRs.
- **Invisible Knowledge Debt:** Keeping architectural decisions in Slack threads or Google Docs instead of encoding them into the repo where agents can find them.

## 🔗 Deep Connections
- **Parent Topic:** [[Harness Engineering]]
- **Related Concepts:** [[Retrieval-Augmented Generation (RAG)]], [[Observability]]

## ❓ Active Recall Questions
> [!faq]- In **Context Engineering**, what is the difference between Static Context and Dynamic Context?
> **Static Context** refers to persistent repository knowledge (like `AGENTS.md` and documentation). **Dynamic Context** refers to real-time, runtime data injected into the loop (like live logs, PromQL metrics, or browser navigation states).

> [!faq]- Why did OpenAI's "one big AGENTS.md" approach fail?
> Four reasons: (1) Context is scarce — a giant file crowds out the actual task; (2) When everything is "important," nothing is; (3) Monolithic files rot instantly and agents can't tell what's still true; (4) A single blob can't be mechanically verified for freshness or coverage. The fix: AGENTS.md as a ~100-line table of contents pointing to a structured `docs/` directory.

> [!faq]- What does "what Codex can't see doesn't exist" mean in practice?
> Knowledge in Google Docs, Slack threads, or people's heads is invisible to the agent. Only repository-local, versioned artifacts (code, markdown, schemas, executable plans) matter. If context isn't in-repo, it might as well not exist.
