# OpenDev: Engineering Implementation Details #flashcards/system-design/opendev

## Behavioral Steering & Nudges

What is the architectural distinction between **System Reminders** and **Nudges** in the OpenDev (2026) Harness?
?
- **The System (Infrastructure/Pipes):** The code mechanism for message delivery, placement (Recency Bias), and model routing (Thinking vs. Action).
- **The Behavior (Content/Water):** The `Nudge` itself—the tactical instruction (e.g., "Empty Action detected") aimed at steering **Claude 4.6 Sonnet's** immediate behavior.
**Hierarchy:** Pipes (System) must exist and position correctly for the Water (Nudge) to reach the model.
#flashcards/system-design/opendev/harness
<!--SR:!2026-03-27,1,230-->

---

## Architecture & Traits

In the `opendev-agents` crate, how is the separation of concerns between Agent and Harness enforced at the code level?
?
Through the **`BaseAgent` trait** (defining the interface for reasoning/prompts) and the **`AgentResult` struct** (capturing the outcome of the Harness-managed ReAct loop, including backgrounding and interruption states).
<!--SR:!2026-03-27,1,230-->

What is the role of `AgentResult::backgrounded` in long-running sessions?
:: it acts as a **"soft yield"**, allowing the harness to pause a task, persist its state, and yield execution to the foreground/UI while the task continues asynchronously.
<!--SR:!2026-03-27,1,230-->

## Context Compaction (ACC) Implementation

What are the 5 specific optimization thresholds used by the `ContextCompactor` in `opendev-context`?
?
1. **70% (Warning):** Logging and tracking begins.
2. **80% (Mask):** Observation masking starts.
3. **85% (Prune):** Fast pruning of old tool outputs.
4. **90% (Aggressive):** Intensive masking and trimming.
5. **99% (Compact):** Full LLM-powered middle-history summarization.
<!--SR:!2026-03-27,1,230-->

Which specific tool types are **protected** from Stage 3 Pruning (85%) to prevent losing critical knowledge anchors?
:: `read_file`, `skill`, `invoke_skill`, `present_plan`, `vlm`, and `web_screenshot`.
<!--SR:!2026-03-27,1,230-->

What is the `PRUNE_PROTECTED_TOKENS` constant and its default value?
:: **40,000 tokens**. It ensures the most recent part of the conversation is never pruned, preserving immediate operational context.
<!--SR:!2026-03-27,1,230-->

## Prompt Engineering & Steering

What are the five key fields of a `PromptSection` in the dynamic composition engine?
?
1. **Name:** Identifier.
2. **File Path:** Template location.
3. **Condition:** Predicate function for inclusion.
4. **Priority:** Integer for sorting (lower = earlier).
5. **Cacheable:** Boolean flag for provider-level prompt caching (e.g., Anthropic).
<!--SR:!2026-03-27,1,230-->

What is the difference between a `Directive` and a `Nudge` in the `MessageClass` system?
?
- **Directive:** Strategy/Error context that reaches **both** Thinking and Action models.
- **Nudge:** Behavioral guardrails (like todo reminders) that reach **only** the Action model.
- **Internal:** Metadata/logs stripped from all LLM calls.
<!--SR:!2026-03-27,1,230-->

How does the `ReactLoop` prevent "Nudge Loops" with a stubborn model?
:: By enforcing a **Nudge Budget** (default: `max_nudge_attempts: 3`), after which the harness stops prompting and accepts the agent's current state as finished.
<!--SR:!2026-03-27,1,230-->

## Orchestration & Skill Deployment

What are the two primary deployment strategies for specialized instructions in OpenDev (2026)?
?
1. **Inline Skill (Swiss Army Knife):** Instructions (e.g., Mermaid) are injected into the **current** agent's prompt for tactical, low-context tasks.
2. **Delegated Subagent (Specialist Squad):** A **new isolated session** is spawned for strategic or "messy" tasks (e.g., security audits) to prevent context pollution.
<!--SR:!2026-03-29,3,250-->

How does the Harness decide which skill strategy to use (Hybrid Intelligence)?
?
- **Deterministic (Fast):** Rust-based `ConditionFn` scans the environment (e.g., `.git` presence).
- **LLM-Driven (Smart):** The **Thinking Model (o2)** analyzes the task and explicitly calls a subagent if a deep dive is required.
#flashcards/system-design/opendev/orchestration
<!--SR:!2026-03-27,1,230-->

---

## Reliability & Safety

What are the **5 Layers of Safety (Defense in Depth)** in the OpenDev architecture?
?
1. **L1 (Prompt):** Role-play guardrails (e.g., "Don't leak internal thoughts").
2. **L2 (Schema):** Tool-stripping (if a tool isn't in the JSON schema, the agent can't call it).
3. **L3 (Approval):** Human-in-the-loop for dangerous operations (`rm`, `write_file`).
4. **L4 (Validation):** Sanity checks on arguments (file size limits, forbidden paths like `.env`).
5. **L5 (Hooks):** Post-execution audit (automatic lint/test) with **Shadow Git rollback** on failure.
<!--SR:!2026-03-27,1,230-->

Why is **L2 (Schema Filtering)** considered the most effective safety layer?
:: Because it creates a **Zero-Trust** environment where the model physically cannot even "see" or "reason about" unauthorized tools, regardless of its internal instructions.
<!--SR:!2026-03-27,1,230-->
#flashcards/system-design/opendev/safety

---

How does the `9-Pass Fuzzy Matching` system handle LLM indentation errors?
:: Through the **`indentation_flexible_find`** pass, which ignores leading indentation and blank lines, focusing purely on the trimmed content of the lines.
<!--SR:!2026-03-27,1,230-->

What algorithm is used for the `context_aware_find` pass (Pass 8) to calculate match quality?
:: **LCS (Longest Common Subsequence)** similarity. It requires a threshold (usually >0.5) to validate a match based on surrounding code anchors.
<!--SR:!2026-03-27,1,230-->

Why does the `SnapshotManager` use `git tree hash` instead of full commits for the Shadow Git?
:: To maximize performance and reduce storage bloat. A tree hash captures the workspace state instantly without the overhead of commit metadata or messages.
<!--SR:!2026-03-27,1,230-->

## Performance & Truncation

What are the default limits for tool output truncation in `truncation.rs`?
?
- **MAX_LINES:** 2000 lines.
- **MAX_BYTES:** 50 KB.
Anything exceeding these is offloaded to `~/.opendev/tool-output/`.
<!--SR:!2026-03-27,1,230-->

When is `TruncateDirection::Tail` used instead of `Head`?
:: For **Shell commands and Test logs**, where the most important information (the error or final result) is located at the very end of the output.
<!--SR:!2026-03-27,1,230-->

## Thinking Phase Logic

How does OpenDev support models with "Native Reasoning" (like o1 or DeepSeek-R1)?
?
By extracting the **`reasoning_content`** field from the API response and using the **`reasoning_effort`** parameter to scale the model's internal thinking budget.
<!--SR:!2026-03-27,1,230-->
