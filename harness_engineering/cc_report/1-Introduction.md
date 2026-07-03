# 1 Introduction

This report details the architectural design and implementation patterns of **Claude Code**, a terminal-native, autonomous AI coding assistant. Operating directly within the developer's command-line environment, Claude Code is designed to bridge the gap between high-level reasoning and rigorous local execution.

## 1.1 The Shift to Terminal-Native Agents
Unlike traditional IDE plugins (which offer reactive suggestions) or browser-based sandboxes, a terminal-native agent lives where development happens. It natively interacts with source control, local file systems, and build commands. Claude Code leverages this position to provide complex, multi-step interventions while relying on robust safety and permission models.

## 1.2 High-Level Architecture Overview
Claude Code operates as a sophisticated compound AI system primarily implemented in TypeScript and React (Ink for CLI UI). The core architecture is modular and highly extensible:

### Architectural Pillars
- **CLI Entrypoint (`main.tsx`):** Handled via Commander.js for arg parsing, utilizing Ink for terminal UI components.
- **The Query Engine (`QueryEngine.ts`):** The beating heart of the system. It manages the core reasoning loop, processing prompts, streaming outputs from the Anthropic API, and orchestrating the ReAct-style tool execution cycle.
- **Context & Memory (`context.ts`, `state/`):** Dynamic context collection handles prompt assembly, injecting relevant environment variables, git state, and conversational history while managing the context window.
- **Tool System (`Tool.ts`, `tools/`):** A rigid registry pattern exposing over 40 distinct capabilities, ranging from bash command execution and file manipulation to MCP resource consumption.
- **Command Registry (`commands.ts`, `commands/`):** Over 100 slash commands that function as fixed user-driven workflows (e.g., `/commit`, `/review`), bypassing or interacting directly with the agent.
- **Coordinator (`coordinator/`):** Handles multi-agent architectures, allowing subagents and specialized routines to be forked and resumed.
- **Bridge (`bridge/`):** Offers deeper integrations with conventional IDEs (VS Code, JetBrains) beyond raw terminal access.
- **Tasks & Skills (`tasks/`, `skills/`):** Infrastructure for background process management and behavior extensions.

By modularizing responsibilities across these components, Claude Code achieves a balance between raw computational power and careful, deterministic system orchestration.
