# Appendices

This section contains reference catalogs, prompt templates, and implementation details.

### A. Complete Tool Catalog
Catalog of 35 built-in tools organized by Category (File Ops, Process, Web, Symbols (LSP), Visual, PDF, Notebooks, Task Mgmt, User Input, Discovery, Batch, Subagents).

### B. LSP Language Server Matrix
Lists supported LSP servers for 30+ languages (Python/Pyright, Rust/rust-analyzer, etc.).

### C. Modular System Prompt Composition
Inventory of the 21 main agent sections and 4 thinking mode sections, including activation conditions and cacheability.

### D. Edit Tool Fuzzy Matching Chain
Details the nine replacer classes (Exact, Line-trimmed, Block-anchor, Whitespace-normalized, Indentation-flexible, Escape-normalized, Trimmed-boundary, Context-aware, Multi-occurrence).

### E. Shell Execution Pipeline
Details the six stages of `run_command` and the 16 server detection patterns for auto-promotion to background.

### F. System Reminder Catalog
Catalog of the 24 named system reminders (Phase control, Task lifecycle, Todo enforcement, Error recovery, Behavioral, JSON retry) and the 9-step injection timing.

### G. Subagent Capability Matrix
Matrix of tool access for the eight subagent types (Code-Explorer, Planner, PR-Reviewer, Security-Reviewer, Web-Clone, Web-Generator, Project-Init, Ask-User).

### H. Configuration Schema
Key fields in `AppConfig` (model, provider, max_tokens, thinking_model, auto_approve, etc.).

### I. Implementation Constants
Thresholds and limits used in OPENDEV (Compaction stages: 70/80/90/99%, Max undo: 50 ops, Max nudge: 3 attempts, etc.).

### J. CLI Command Reference
Launch options (`--continue`, `--working-dir`) and Slash commands (`/mode`, `/undo`, `/clear`, `/sessions`, `/thinking`, `/exit`).

### K. Full System Prompt Templates
Verbatim content of system prompt markdown files (main.md, mode-awareness.md, security-policy.md, tone-and-style.md, interaction-pattern.md, available-tools.md, tool-selection.md, code-quality.md, action-safety.md, read-before-edit.md, error-recovery.md, subagent-guide.md, git-workflow.md, task-tracking.md, output-awareness.md, scratchpad.md, code-references.md, reminders-note.md, thinking.md, available-tools.md, subagent-guide.md, code-references.md, output-rules.md, compaction.md, critique.md, init.md).
