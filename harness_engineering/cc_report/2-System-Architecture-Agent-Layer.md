# 2 System Architecture & Agent Layer

The agent layer in Claude Code represents the cognitive core of the application, responsible for planning, reasoning, and routing tasks to the appropriate execution channels.

## 2.1 The Query Engine (`QueryEngine.ts`)
The `QueryEngine` is the operational loop of Claude Code. When a user input is received, the system does not simply invoke the API once; it enters an event loop:
1. **Context Assembly:** Aggregates user intent, file contents, conversational memory context, and terminal states into a formatted prompt array.
2. **LLM Invocation:** Calls the Anthropic services (`services/api/`) with tools exposed as functions.
3. **Stream Handling & UI:** Progressively streams the response to the user via the CLI's React-based UI (Ink).
4. **Tool Evaluation:** Pauses generation to execute any called tools locally.
5. **Cycle Return:** Appends the tool outputs as observations back into the context window and returns to step 2 until the model deems the task complete.

## 2.2 Subagents and the Coordinator (`coordinator/`)
Claude Code employs a compound multi-agent system where distinct tasks can be handed off:
- **AgentTool:** Used natively by the main agent to offload specific tasks. Subagents are forked with their own context bounds and tools constraints. 
- **Forking and Resuming:** `forkSubagent.ts` and `resumeAgent.ts` show that agents maintain state and memory trees, allowing complex overarching tasks like debugging and refactoring to not pollute the primary context window with intermediate trace data.

## 2.3 The IDE Bridge (`bridge/`)
To transcend the limitations of pure terminal UX without leaving the CLI flow, Claude Code includes an IDE bridge.
- Uses local sockets/communication proxies to read open files directly from a paired VS Code or JetBrains instance.
- This allows the context compiler to intelligently auto-inject the file the developer was just looking at in the IDE prior to typing the command.

## 2.4 Server & Remote Execution
The `server/` and `remote/` packages indicate Claude Code has the capability to decouple the local CLI client from the heavy execution engine. This caters to remote-dev container scenarios, allowing the intelligent orchestration logic to run in disparate environments.
