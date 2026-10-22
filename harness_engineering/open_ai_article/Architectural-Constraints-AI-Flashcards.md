<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
# Architectural Constraints for AI Agents — Flashcards

<!-- #flashcards/harness-engineering -->

<!-- In **AI-assisted development**, why is architectural decay *more dangerous* than in traditional development? -->
<!-- ? -->
<!-- AI agents optimize for **immediate task completion**, not long-term coherence. They violate boundaries at **scale** — a human might shortcut once, an agent does it every time. The decay is silent: code compiles, tests pass, but layer boundaries blur until your architecture is fiction. -->
<!--SR:!2026-05-05,21,250-->

---

<!-- What is the fundamental difference between **linting** and **architectural validation**, and why does it matter for AI agents? -->
<!-- ? -->
<!-- - **Linting** is **local** (one file) — checks syntax/style: naming, indentation, unused imports. -->
<!-- - **Architectural validation** is **global** (whole system) — checks structure: dependency direction, module isolation, layer boundaries. -->
<!-- AI agents need both, but architectural validation is harder to enforce and more critical — without it, an agent can produce perfectly linted code that destroys your system's structure. -->
<!--SR:!2026-05-02,18,250-->

---

<!-- In the context of **architectural constraints**, what are the three enforcement mechanisms, and at what stage does each operate? -->
<!-- ? -->
<!-- 1. **Static analysis before merge** (CI gate) — ArchUnit, ESLint custom rules. Violations block the PR. Feedback must be specific: not "violation" but "controllers.X cannot import db.Y — use a service layer." -->
<!-- 2. **Custom validators in the AI workflow** (pre-commit) — validate agent output *before* it's even committed. Fastest feedback loop — like pair programming with the agent. -->
<!-- 3. **Runtime guards** (production) — middleware, query interceptors, encryption wrappers for constraints that can't be checked statically (e.g., N+1 queries, unencrypted sensitive data). -->
<!--SR:!2026-05-01,9,210-->

---

<!-- When choosing tools and libraries for an **AI-scaffolded project**, what three criteria reduce agent hallucinations and errors? -->
<!-- ? -->
<!-- 1. **Visibility (ownership > dependency):** Code must live in the repo, not hidden in `node_modules`. If the agent can't read the source, it can't reason about it — it will guess, and guess wrong. -->
<!-- 2. **Declarative, finite vocabulary > complex APIs:** Tools with a bounded set of explicit primitives (e.g., utility classes) are easier for agents than imperative APIs with hidden state (e.g., theme providers, CSS-in-JS context). -->
<!-- 3. **Minimal abstraction layers:** Each abstraction layer is a hallucination surface. Fewer layers between intent and implementation = fewer places for the agent to misinterpret. -->
<!--SR:!2026-05-05,7,210-->

---

<!-- Why is **"code ownership" (source in your repo)** more AI-friendly than **"code dependency" (package in node_modules)**? -->
<!-- ? -->
<!-- A dependency is a **black box** — the agent sees the API surface but not the implementation. It must infer behavior from docs or training data, which leads to hallucinations. Owned source code is a **white box** — the agent reads the actual JSX, styles, and logic, then modifies them directly. This is the same principle as "the repo is the single source of truth": if it's not visible in the codebase, it doesn't exist for the agent. -->
<!--SR:!2026-05-06,22,250-->
