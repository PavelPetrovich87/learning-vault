# 3 The Tool System

Claude Code relies on a strictly typed, permission-bound Tool System to interact with its host environment. Unlike simple LLM wrappers, Claude Code maintains complex behavioral and security invariants over its tool executions.

## 3.1 Tool Schema and Abstractions (`Tool.ts`)
All tools extend a base definition that mandates:
1. Strict schema validation for inputs (typically Zod or JSON schemas).
2. Defined UX rendering strategies (e.g., streaming output previews vs. hidden background outputs).
3. Explicit permission models (whether the tool can auto-execute or requires explicit user approval).

## 3.2 Primary Tool Categories
Claude Code has 40 registered tools, roughly categorized as follows:

### Local Execution Tools
- **BashTool / PowerShellTool:** The ultimate capability interfaces. They implement sophisticated rules around reading, writing, and permissions. For example, `bashPermissions.ts` and `destructiveCommandWarning.ts` evaluate commands against safety heuristics before auto-running.
- **REPLTool:** For interactive environments.

### Filesystem & Context Gathering
- **FileReadTool & FileEditTool:** Basic capabilities for inspecting or modifying code. File reading enforces size limits (`limits.ts`) to prevent blowing out the context window. 
- **GlobTool & GrepTool:** Codebase exploration capabilities, often routing to fast native tools like `rg` (ripgrep) underneath.
- **LSPTool:** Allows the agent to query language servers for symbol references, type definitions, and diagnostics (`symbolContext.ts`).

### Modularity & Extensibility
- **MCPTool, ListMcpResourcesTool, ReadMcpResourceTool:** Standardized implementations of the Model Context Protocol (MCP), allowing the agent to dynamically register external tools served by arbitrary MCP servers without hardcoding them into the main binary.
- **SkillTool:** Uses a framework for agent behaviors or scripts stored locally.

### Agent Workflow Management
- **EnterPlanModeTool & ExitPlanModeTool:** A toggle that restricts the agent to exploration and drafting, explicitly blocking mutations to safeguard against impulsive automated changes.
- **EnterWorktreeTool & ExitWorktreeTool:** Suggests isolated workspace testing.
- **Task Tools (`TaskCreateTool`, `TaskUpdateTool`, `TaskStopTool`):** Mechanisms for spawning long-running background tasks.

## 3.3 The Slash Commands System (`commands/`)
Alongside AI-driven tools, Claude Code implements over 100 deterministic CLI slash commands for direct user interaction. 
- Lifecycle commands: `/clear`, `/exit`, `/login`, `/logout`.
- Debugging capabilities: `/ant-trace`, `/bughunter`, `/heapdump`.
- Git/PR management: `/commit`, `/commit-push-pr`, `/autofix-pr`.
- Integrations: `/install-github-app`, `/install-slack-app`.
- Modes/State: `/plan`, `/fast`, `/cost`, `/review`.

The command registry provides escape hatches from autonomous agency into direct developer control.
