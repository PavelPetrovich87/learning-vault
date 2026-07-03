# 2 System Architecture

## 2.1 Overview

Figure 2 illustrates the architecture of OPENDEV across four principal layers: Entry & UI, Agent, Tool & Context, and Persistence. A user query flows through this pipeline sequentially, from the entry point through agent reasoning and tool execution, before results are persisted and rendered.

**Entry & UI Layer.** The CLI Entry Point parses arguments and bootstraps four shared managers (Config Manager, SessionManager, ModeManager, and ApprovalManager), which are injected into all downstream components. OPENDEV supports two frontends: a TUI built on Textual using blocking modal approvals, and a Web UI backed by FastAPI and WebSockets using asynchronous polling approvals. Both implement a shared UICallback contract, keeping the agent layer UI-agnostic.

**Agent Layer.** OPENDEV assigns five specialized model roles to distinct LLMs (Section 2.2.5), each lazily initialized and informed by a locally cached capability registry. The system operates in two modes: Normal Mode with full read-write tool access for execution, and Plan Mode restricted to read-only tools for safe planning (Section 2.2). Reasoning proceeds through the Extended ReAct Loop (Section 2.2.6), which runs four phases per turn: automatic context compaction when the token budget nears exhaustion; an optional thinking phase for pre-action reasoning at configurable depth; an optional self-critique phase; and the standard Reason-Act-Execute-Observe action phase.

**Tool & Context Layers.** The Tool Execution Layer is built around a ToolRegistry that dispatches calls to typed handlers covering file operations, process execution, and web access, with support for batch parallel execution and on-demand MCP tool discovery (Section 2.4.7). A Skills system lazily injects reusable, domain-specific prompt templates from a three-tier hierarchy (built-in, project, user). The Context Engineering Layer manages the LLM context window through four subsystems: System Reminders (Section 2.3.4) for context-aware behavioral guidance, Prompt Composer for modular system-prompt assembly, Memory for cross-session continuity, and Compaction (Section 2.3.6) for reclaiming token budget.

**Persistence Layer.** OPENDEV persists state across four stores: a Config Manager that resolves settings through a project-local, user-global, environment-variable, and built-in default hierarchy; a Session Manager that saves full conversation histories as JSON; a Provider Cache that stores model capability metadata locally; and an operation log that tracks file changes for rollback.

**Safety Architecture.** Because the agent can execute arbitrary shell commands, overwrite files, and spawn persistent processes, a single safety mechanism is insufficient. OPENDEV therefore employs a defense-in-depth architecture with five independent safety layers (Figure 3), each designed to prevent a class of harm independently so that no single point of failure compromises the system.

- Layer 1: Prompt-Level Guardrails (Section 2.3.1)
- Layer 2: Schema-Level Tool Restrictions (Section 2.4.1)
- Layer 3: Runtime Approval System (Section 2.4.1)
- Layer 4: Tool-Level Validation (Section 2.4.3, Section 2.4.2)
- Layer 5: Lifecycle Hooks (Section 2.4.1)

## 2.2 Agent Core Layer

The agent layer sits between the UI and the tool execution layers (Figure 2). At its center is a single entry point, MainAgent, which receives every user prompt and decides how to process it. Understanding this layer requires two perspectives: how agents are assembled before the first prompt arrives (scaffolding), and how the assembled agent processes user messages at runtime (the harness). In this context, a harness is the runtime orchestration layer that wraps the core reasoning loop and coordinates tool execution, context management, safety enforcement, and session persistence around it [102]. Where scaffolding is concerned with constructing the agent before the first prompt, the harness is concerned with everything that happens after: dispatching tools, compacting context, enforcing safety invariants, and persisting state across turns.

### 2.2.1 Agent Scaffolding

Before the agent can process user prompts, it must be fully assembled. Every agent in OPENDEV is constructed (system prompt compiled, tool schemas built, subagents registered) before the conversation lifecycle begins (Section 2.2.3). Understanding this construction pipeline clarifies why the runtime can treat all agents uniformly regardless of their role.

**Type foundation: BaseAgent and AgentInterface.** All agents inherit from BaseAgent, an abstract base class that accepts three constructor arguments (config, tool_registry, mode_manager) and defines four abstract methods: build_system_prompt() assembles the system prompt string, build_tool_schemas() returns OpenAI-format tool schemas, call_llm() executes a single LLM call, and run_sync() runs a full ReAct loop. The critical design choice is eager construction: BaseAgent.__init__() calls both build_system_prompt() and build_tool_schemas() before the constructor returns. By the time __init__() completes, the agent is fully ready to serve requests, with no lazy prompt assembly, no first-call latency. A complete refresh_tools() method re-invokes both build methods when the tool registry changes (e.g., after MCP server discovery or dynamic skill loading). Downstream code depends not on BaseAgent directly but on AgentInterface, a @runtime_checkable Protocol requiring the same surface (system_prompt, tool_schemas, refresh_tools, call_llm, run_sync), which decouples the factory from the concrete agent class.

**Single concrete agent class.** There is no class hierarchy of agent types. MainAgent is the only concrete subclass of BaseAgent, and every agent in the system (the main agent, all builtin subagents, and any user-defined custom agents) is an instance of this single class. Behavioral variation comes entirely from construction parameters: allowed_tools (a list that filters which tool schemas appear in the agent’s schema, or None for full access), _subagent_system_prompt (an override prompt set after construction), and an is_subagent flag derived from whether allowed_tools is non-null. Within __init__, MainAgent sets four HTTP client slots to None for lazy initialization, one each for the normal, thinking, critique, and VLM providers, deferring API key validation until the first LLM call. It creates a ToolSchemaBuilder(registry, allowed_tools) for schema generation and a bounded Queue(maxsize=10) for thread-safe message injection from the Web UI. The lazy client slots correspond to the model roles described in Section 2.2.5: each slot materializes a provider-specific HTTP client on first access, allowing agents to be constructed before credentials are configured.

**Factory assembly.** AgentFactory is the single entry point for agent construction. Both the TUI and Web UI invoke the same create_agents() method, ensuring identical setup regardless of frontend. The factory executes three phases in strict order:
- Phase 1 (Skills). The factory discovers skill definitions from three directories (builtin, user global, and project-local), creates a SkillLoader, and registers it with the tool registry so that the use_skill tool becomes available.
- Phase 2 (Subagents). The factory creates a SubAgentManager, calls register_defaults() to compile the builtin subagent specifications, then calls _register_custom_agents() to load any user-defined agents from configuration files. Finally, it registers the manager with the tool registry via set_subagent_manager(), which makes the spawn_subagent tool available.
- Phase 3 (Main agent). The factory constructs a MainAgent with no tool filtering (full access to all registered tools, including those added in Phases 1 and 2).

**Subagent compilation.** Each subagent begins as a SubAgentSpec, a TypedDict containing a name, description, system prompt, optional tool allowlist, optional model override, and optional Docker configuration. When SubAgentManager.register_subagent(spec) is called, it executes a four-step pipeline: (1) resolve the tool list, defaulting to a hardcoded set of safe tools if none is specified; (2) create an AppConfig copy with the model override if one is provided; (3) construct a MainAgent with allowed_tools set to the resolved list, triggering the eager build of a filtered system prompt and tool schema; and (4) set agent._subagent_system_prompt to the spec’s prompt override. The result is stored as a CompiledSubAgent (name, description, agent instance, tool list). Construction is cheap because all subagents share the same tool registry reference, with no cloning or deep copying.

**Dependency injection.** Agent construction produces agents; runtime execution requires services. AgentDependencies is a Pydantic model carrying seven fields that tools need at execution time: mode_manager, approval_manager, undo_manager, session_manager, working_dir, console, and config. Subagents receive a lightweight SubAgentDeps dataclass with only three fields: mode_manager, approval_manager, and undo_manager.

**Design evolution.** Three design pivots shaped the current scaffolding architecture. First, an early class hierarchy, with separate classes for planning agents, code-exploration agents, and web-generation agents, was replaced by the single parameterized MainAgent. Second, lazy prompt building (constructing the system prompt on the first run_sync call) was replaced by the eager-build pattern. Third, inline subagent definitions were replaced by the SubAgentSpec registration system.

### 2.2.2 Agent Runtime Architecture

With scaffolding complete, the assembled agent is ready to process user messages. The runtime behavior is governed by the agent harness, the orchestration infrastructure introduced in Section 2.2 that turns a stateless LLM into a persistent, tool-using, self-correcting agent.

**The central execution cycle.** The ReAct loop executes six phases per iteration: pre-check and compaction, thinking, self-critique, action, tool execution, and post-processing. Each phase is a distinct stage in the executor pipeline. The cycle repeats until the agent produces a final text response with no tool calls, or until a safety cap is reached.

**Input and output boundaries.** User messages enter through a thread-safe bounded queue. At the bottom, the Post-Processing path handles three responsibilities after the loop terminates: persisting the updated conversation to the session store, executing Stop hooks, and returning the agent’s final response to the UI layer.

**Supporting subsystems.** The Prompt Composition engine (Section 2.3.1) assembles the system prompt from modular sections. The Tool Registry (Section 2.4.1) dispatches each tool call to specialized handlers. The Safety System (Section 2.1) provides defense in depth. Context Engineering (Section 2.3.6) manages the conversation as a finite resource. Memory and Session services (Section 2.5) persist history and learned strategies. Subagent Orchestration (Section 2.2.7) enables delegation.

**Routing: plan or execute.** When a user prompt arrives, the MainAgent checks whether it should enter Plan Mode or proceed directly in Normal Mode. Two triggers activate Plan Mode: an explicit /plan command or a heuristic that detects planning intent. Routing is a one-time decision per prompt.

**Plan Mode: subagent-based planning.** Planning is delegated to a first-class Planner subagent. The Planner subagent executes in three stages: (1) explores the codebase using read-only tools; (2) analyzes findings and determines the sequence of changes; (3) writes a structured plan to a file in the scratch directory. Upon completion, the Planner returns the plan file path to the main agent, which displays it to the user.

**Normal Mode: full execution.** Normal Mode is the default operating state with full tool access. After a plan is approved, the agent works through the planned steps.

**Rationale for subagent-based planning.** The current design eliminates a brittle state machine by delegating planning to a subagent with a read-only tool schema. This ensures the Planner cannot write because write tools do not exist in its schema, and allows concurrent subagent spawning.

### 2.2.3 Conversation Lifecycle

Figure 6 traces the end-to-end path of a single user message through the system.

**Input ingestion and query processing.** User input arrives through a CLI invocation, the TUI, or the Web UI. All paths converge on the same agent execution core.

**The iteration cycle.** Each iteration follows a fixed sequence: draining injected messages, checking context pressure/compaction, thinking (if enabled), calling the action LLM, and dispatching tool calls.

**Completion and persistence.** The loop terminates when the agent signals done, the error budget is exhausted, or the iteration count reaches the limit. The final state is saved to the session store.

**Lifecycle hooks.** OPENDEV exposes lifecycle events (SessionStart, UserPromptSubmit, PreToolUse, etc.) that external scripts can observe or intercept.

**Cross-cutting concerns.**
- Interrupt tokens: propagate cancellation requests.
- Thread-safe injection queue: allows follow-up messages mid-execution.
- Session cost tracking: records cumulative token usage and cost.

### 2.2.4 REPL Command Dispatch

Deterministic operations like mode switching or model selection are handled directly by the REPL without invoking the agent. If the input begins with a “/” prefix, it is routed to a registered command handler.

**Command handler abstraction.** All handlers extend an abstract base class. The handler’s handle(args) method executes the operation and returns a CommandResult.

**Handler categories.** Nine handler classes cover session management (/clear, /compact), mode switching (/mode), configuration (/models), MCP commands (/mcp), agent management (/agents), skills, plugins, initialization (/init), and help.

**Side effects on system state.** Commands modify shared system state that subsequent agent interactions depend on (e.g., rebuilding the agent factory with a new model).

**Separation from agent tools.** The distinction between REPL commands and agent tools is architectural. Commands are synchronous and require no LLM; tools are selected by the LLM and subject to approval and undo tracking.

### 2.2.5 Workload-Optimized Multi-Model Architecture

Different execution phases benefit from different model capabilities. OPENDEV uses task-specific routing to optimize for cost, latency, and quality.

**Five model roles with fallback chains:**
- **Action model**: Primary model for tool-based reasoning.
- **Thinking model**: Optional model for extended reasoning without tool access.
- **Critique model**: Optional model for self-evaluation.
- **Vision model**: Vision-language model for processing images.
- **Compact model**: Smaller, faster model for summarization during context compaction.

**Provider abstraction and caching.** API clients are initialized lazily. Model capabilities are cached locally with a stale-while-revalidate pattern.

### 2.2.6 Extended ReAct Execution Loop

OPENDEV implements an extended ReAct loop with explicit thinking and optional self-critique.

**Initialization.** The executor clears queues, creates interrupt tokens, and bundles state into an IterationContext.

**Phase 0: Staged context management.** The compactor applies five progressively aggressive reduction strategies (Warning, Observation Masking, Fast Pruning, Aggressive Masking, Full LLM-based Compaction).

**Phase 1: Thinking.** If enabled, a separate thinking LLM call produces a reasoning trace without tool access. Separating thinking from action prevents premature tool use.

**Phase 2: Action.** The executor assembles the action prompt (system prompt, memory, tool schemas, conversation history, thinking trace) and calls the action LLM.

**Phase 3: Decision, dispatch, and doom-loop detection.**
- If no tool calls: handle errors with recovery nudges or signal completion.
- If tool calls: perform doom-loop detection using fingerprints (MD5 of tool name + args). Fingerprint-based detection catches stuck loops within 3 repetitions.
- Dispatch tools: read-only tools run in parallel; write tools run sequentially. Outcomes feed into the ACE memory pipeline.

**Termination and completion detection.** The loop ends through explicit tool calls, implicit completion (text-only response), nudge budget exhaustion, or safety limit.

### 2.2.7 Subagent Orchestration

The main agent can spawn specialized subagents for specific subtasks. Subagents execute in isolated contexts with filtered tool access and independent iteration budgets.

**Subagent specialization:**
- **Code explorer**: Read-only tools for codebase navigation.
- **Strategic planner**: Read-only tools plus extended reasoning.
- **Web tools**: Web fetching and file writing for cloning content.
- **User clarification**: Minimal toolset for gathering input.

**Tool filtering rationale.** Capabilities are restricted to prevent race conditions, reduce context size, and limit the blast radius of errors.

**Parallel execution.** The main agent can spawn multiple subagents concurrently. Results are synthesized into a single unified response.

**Subagent prompt refinements.** Prompts include explicit termination conditions (e.g., “stop when evidence is clear”) and anti-loop instructions.
