# 2.4 Tool System

The tools through which the agent interacts with the development environment form an extensible ecosystem that balances comprehensive capability with context efficiency, safety with flexibility, and built-in tools with dynamic discovery.

### 2.4.1 Registry Architecture and Schema Construction

A flat namespace of tools becomes unmanageable as capabilities grow. OPENDEV adopts a registry with handler categories, organizing tools into handler classes with schema-based registration.

**Three separated concerns.**
- **ToolSchemaBuilder**: Assembles JSON schemas from static built-in definitions, dynamically discovered MCP schemas, and subagent schemas.
- **ToolRegistry**: Serves as the central dispatcher, mapping tool names to handler methods across 12 handler classes.
- **Lifecycle hooks**: Provides extensibility without modifying handler code (synchronous PreToolUse, asynchronous PostToolUse).

**Runtime approval.** Before any tool call, the runtime approval system gates execution based on trust boundaries (Manual, Semi-Auto, Auto). Default danger rules (e.g., `rm -rf /`) are always active and cannot be overridden.

### 2.4.2 File Operations

Five tools handle file-system interactions:
- **read_file**: Line-numbered reading with output truncation (head-tail strategy) and stale-read tracking.
- **write_file**: New file creation only; rejects overwrites to prevent accidental full-file wipes.
- **edit_file**: 9-pass fuzzy matching to handle LLM formatting drift (whitespace, indentation, etc.). Implements a chain-of-responsibility pattern with nine replacer classes.
- **list_files**: Directory listing and glob search with ignore patterns.
- **search**: Dual-mode content search (text via ripgrep and structural via ast-grep).

### 2.4.3 Shell Execution and Background Tasks

- **run_command**: Six-stage execution pipeline including safety gates, command prep, server detection, execution fork (PTY for background, pipes for foreground), output management, and timeout handling.
- **Background task management**: Commands promoted to background are assigned unique IDs and their output is streamed to files.
- **get_process_output**: Retrieves the last 100 lines from a background task.
- **kill_process**: Graceful termination using SIGTERM followed by SIGKILL.

### 2.4.4 Web Interaction

- **fetch_url**: Browser-engine web fetching via Playwright/Crawl4AI to handle JavaScript-heavy sites.
- **web_search**: Privacy-respecting search via DuckDuckGo.
- **capture_web_screenshot**: Full-page screenshots at different breakpoints.
- **open_browser**: Launches the system’s default browser for visual workflow integration.

### 2.4.5 Multi-Language Semantic Code Analysis via LSP

OPENDEV adopts Language Server Protocol (LSP) integration to provide semantic understanding (type resolution, cross-file references) across 30+ languages.

**Four-layer LSP abstraction.** Agent Tool Layer -> Symbol Retriever -> LSP Server Wrapper -> Solid Language Server. This translates agent-facing tool calls into language-server-specific protocol messages.

**Core tools:**
- **find_symbol**: Symbol definition lookup with location and body preview.
- **find_referencing_symbols**: Cross-file reference search.
- **rename_symbol**: Workspace-wide semantic rename.
- **replace_symbol_body**: Body-preserving rewrite.
- **insert_before/after_symbol**: Positional code insertion.

### 2.4.6 User Interaction, Task Management, and Planning

- **ask_user**: Structured multi-choice questions with free-text fallback. Rendering adapts to TUI (modal) or Web UI (polling).
- **Task tracking**: Lightweight kanban list (write_todos, update_todo, complete_todo, list_todos) that persists across agent iterations.
- **present_plan**: Plan review and approval tool. Plan steps are extracted into the todo list upon approval.
- **task_complete**: Explicit signal that the current task is finished.

### 2.4.7 Token-Efficient External Tool Discovery via MCP

- **search_tools**: Token-efficient discovery of external tools via the Model Context Protocol. Adopts lazy discovery: only schemas for tools explicitly searched for or invoked are included in the context. This reduces baseline context overhead from 40% to <5%.

### 2.4.8 Subagent Delegation, Skills, and Batch Execution

- **spawn_subagent**: Launches an isolated subagent with its own ReAct loop and filtered tool registry. Supports automatic parallelization via asyncio.
- **invoke_skill**: On-demand loading of modular knowledge units (git conventions, checklists, etc.) from a three-tier hierarchy (project, user, built-in).
- **batch_tool**: Enables multiple tool calls in a single turn (parallel or serial mode), reducing round-trip overhead.
