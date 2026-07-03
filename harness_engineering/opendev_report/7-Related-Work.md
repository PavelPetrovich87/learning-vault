# 4 Related Work

The development of terminal-centric AI coding agents draws upon several research streams:

### 4.1 Code Generation and Code LLMs
Progressed from function-level generation (HumanEval) to repository-level challenges. Key models include DeepSeek-Coder, StarCoder, and CodeLlama. OPENDEV builds on this by embedding LLMs within an agentic loop for real-world navigation and editing.

### 4.2 Autonomous Issue Resolution
SWE-bench defined the task of autonomous issue resolution. Frameworks like SWE-Agent and AutoCodeRover explore single-agent navigation, while OpenHands orchestrates heterogeneous agents.

### 4.3 Code as a Core Medium for Generalist Agents
Shift from natural language reasoning to code-driven interactions.
- **Interaction protocols**: ReAct and ReWOO.
- **Thinking and acting in code**: PAL, CodeAct, TaskWeaver.
- **Memory with code**: Voyager (stores skills as code), MemGPT (virtual paging).

### 4.4 Agentic Software Engineering Workflows
- **PDAR loop**: Plan–Do–Assess–Review.
- **Spec-driven development**: GitHub’s Spec Kit, OpenSpec (governing changes via `spec.md`).
- **Multi-agent and agile-inspired**: BMAD (organizing agents into Scrum roles).

### 4.5 Agent Tool Systems and Modular Components
Training-free frameworks rely on specialized tools for bug reproduction (AEGIS), fault localization (SBFL), and patch generation (AutoDiff).

### 4.6 Context Engineering for Long-Horizon Agents
Context Engineering (CE) formalizes the supplied context as a structured tuple of instructions, knowledge, tools, memory, state, and query.
- **Theoretical principles**: Entropy reduction, minimal sufficiency, and semantic continuity.
- **Compression**: Tree-of-thought, In-context Autoencoder.
- **Meta-level optimization**: Meta Context Engineering (MCE) co-evolves context configurations and evaluation setups.

### 4.7 Benchmarks for Evaluating Agentic Coding Systems
Benchmarks have widened from APPS to repository-level issue resolution (SWE-bench) and environment-grounded interaction (WebArena, OSWorld). Specialized benchmarks evaluate tool-use proficiency (Berkeley Function Calling Leaderboard).
