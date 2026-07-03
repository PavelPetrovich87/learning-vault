# 2.3 Context Engineering Layer

LLM-based agents do not simply send user messages to a model and receive responses. The quality of every response is determined primarily by what the model is allowed to see in its context window: which instructions it receives, how much conversation history is retained, what it has learned from prior interactions, and how relevant external information is assembled before each call [39, 56, 101]. We refer to the set of mechanisms that manage this window collectively as the Context Engineering Layer [39, 56, 101].

Figure 9 shows the subsystems and their interactions. A user query enters via the QueryProcessor, which hands off to ContextPicker for context assembly. The assembled context passes through ContextCompactor for token-budget enforcement before entering the ReAct reasoning loop. Each turn, the result flows back to the SessionManager for persistence, and tool outcomes that represent learning opportunities are processed by the adaptive memory subsystem.

### 2.3.1 Dynamic System Prompt Construction

In an LLM-based agent, the system prompt is the primary instrument of behavioral control: it encodes the agent’s identity, capability boundaries, safety constraints, and task conventions. Every behavioral property that cannot be enforced through code is expressed as natural language in the system prompt. Getting this prompt right at startup is the central initialization problem.

**Priority-ordered conditional composition.** OPENDEV assembles each agent’s system prompt at runtime through a pipeline. Behavioral instructions are factored into independent sections, each stored as a separate markdown file and registered with a condition predicate and a priority integer.
1. **Filter.** Evaluate each section’s predicate against the current environment snapshot.
2. **Sort.** Order surviving sections by ascending priority.
3. **Load.** Read markdown files and resolve ${VAR} placeholders.
4. **Join.** Concatenate sections to produce the complete system prompt.

**Mode-specific variants and variable substitution.** Different reasoning phases require different prompts. Normal mode loads all tiers; thinking mode loads only a small set of purpose-built sections (available-tool awareness, subagent guidance). PromptVariables registry maps symbolic names to concrete tool identifiers, decoupling template prose from tool naming.

**Provider-specific conditional sections.** Different LLM providers have different capabilities. The PromptComposer addresses this with mutually exclusive sections gated on the model_provider field.

**Provider-level prompt caching.** For providers like Anthropic that support input caching, the PromptComposer splits the prompt into a stable part (cacheable) and a dynamic part. This yields substantial cost savings (approx. 88% reduction in input token cost for the cached portion).

**Two-tier fallback.** If an individual file is missing, the composer skips it. If modular composition fails entirely, it falls back to a monolithic core template.

### 2.3.2 Tool Result Optimization

Raw tool outputs consume far more tokens than their informational value warrants. Tool result optimization transforms raw outputs into compact, semantically preserving representations.

**Per-tool-type summarization.** Each tool result passes through a summarizer that applies a type-specific compression strategy:
- **File reads**: replaced with metadata (e.g., “Read file (142 lines, 4,831 chars)”).
- **Search results**: report match counts rather than matched lines.
- **Directory listings**: collapsed to item counts.
- **Command execution**: short outputs retained; longer outputs report line counts.
- **Errors**: truncated to 200 characters with a classified prefix.

**Large output offloading.** Outputs exceeding 8,000 characters are offloaded to scratch files. The conversation stores a 500-character preview and a reference path.

**Agent-aware truncation hints.** Offloaded outputs include tailored recovery hints (e.g., suggesting delegation to a Code Explorer).

**Interaction with compaction.** The compactor preferentially uses pre-computed summaries during LLM-based summarization, improving speed and quality.

### 2.3.3 Dual-Memory Architecture for Bounded Thinking

The thinking phase requires context but Providing unbounded history is infeasible. We resolve this through a dual-memory architecture.

**Episodic memory.** An LLM-generated summary of the full conversation history captures strategic, long-range context (goals, findings, file paths). It is regenerated every 5 messages from the full history to prevent summary drift.

**Working memory.** The most recent Exchanges (6 by default) are reproduced verbatim to provide fine-grained operational details.

**Combined injection.** The thinking context concatenates episodic memory summary, working memory messages, and the current user query.

### 2.3.4 Context-Aware System Reminders

System reminders address attention drift in long-running sessions. The model’s influence from the initial system prompt fades as the conversation grows.

**Event detectors.** The injection layer monitors eight conditions (tool failure, exploration spirals, premature completion, etc.) at the boundary between tool execution and the next LLM call.

**Template resolution.** Reminder text lives in plain text files (reminders.md), making them auditable and editable.

**Guardrail counters.** Each reminder type is governed by a counter or one-shot flag to prevent them from becoming noise.

**Rationale for role: user.** Reminders are injected as role: user messages. Recency makes the model treat them as something that just happened, requiring a response, resulting in higher compliance.

### 2.3.5 Context-Injected Error Recovery

When a tool fails, the system injects targeted recovery guidance.
1. **Classify** the error (permission, not found, mismatch, etc.).
2. **Retrieve** the corresponding template.
3. **Format** with context details.
4. **Inject** as a system message at maximum recency.

### 2.3.6 Adaptive Context Compaction (ACC)

Tool observations can quickly consume 70–80% of the token budget. ACC manages context pressure through a five-stage pipeline:
- **Stage 1 - Warning (70%)**: Log utilization trends.
- **Stage 2 - Observation Masking (80%)**: Replace older tool results with compact pointers.
- **Stage 2.5 - Fast Pruning (85%)**: Lightweight deletion of outputs beyond the recency budget.
- **Stage 3 - Aggressive Masking (90%)**: Preserve only the most recent outputs.
- **Stage 4 - Full Compaction (99%)**: Serialize history to disk and compress the middle portion via LLM summarization.

**Artifact Index.** ACC maintains a structured registry of all files touched, ensuring the agent remembers its actions even after compression.

### 2.3.7 Context Retrieval and Assembly Pipeline

Context retrieval is the most consequential capability. OPENDEV uses a four-layer pipeline:
- **Layer 1: Anchor-based retrieval tool selection.** Identify the strongest anchor (symbol name, string literal, etc.) and route to the most precise tool (find_symbol, text_search, etc.).
- **Layer 2: Multi-step agentic search via the Code Explorer.** Delegate to a subagent for exploratory retrieval. Context isolation prevents intermediate results from bleeding into the main agent's context.
- **Layer 3: Context assembly.** ContextPicker merges system prompt, memory, references, history, reminders, and query. Each piece tracks its provenance via ContextPiece.
- **Layer 4: Context optimization.** The assembled context passes through the staged compaction pipeline. budgets are calibrated against API-reported prompt_tokens for accuracy.
