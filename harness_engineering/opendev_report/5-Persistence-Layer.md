# 2.5 Persistence Layer

The persistence layer stores conversation histories, configuration, model metadata, and file-operation logs using ordinary files on disk (JSON, JSONL, and plain text). No external database is required.

**Directory Structure.** Persistent state lives under two root directories:
- **User-global state** (`~/.opendev/`): settings, caches, installed plugins.
- **Project-scoped state** (`~/.opendev/projects/{encoded-path}/`): session transcripts, project-specific settings. This ensures isolation between different repositories.

### 2.5.1 Session Storage

Each conversation is stored as two files:
- **Metadata file** (`.json`): Records session identifier, timestamps, working directory, title, and summary.
- **Transcript file** (`.jsonl`): Stores the actual messages, one per line, serialized as JSON objects with roles, content, tool calls, and token counts.

**Safety and Durability.**
- **Atomic Renaming**: Sessions are written to temporary files and renamed via `os.rename()` to prevent data loss.
- **Exclusive Locks**: `fcntl.flock` prevents concurrent access issues.
- **Auto-save**: The conversation is auto-saved every 5 turns.
- **Session Index**: A lightweight index (`sessions-index.json`) enables instant session listing.

### 2.5.2 Operation Log and Undo

OPENDEV tracks every file operation (create, modify, delete) in a log to provide a single-command undo.

**Shadow git snapshots.** For comprehensive per-step undo (including shell command side effects), the system maintains a shadow git repository at `~/.opendev/snapshot/<project-id>/`. At every agent step, it records a tree hash. The `/undo` command can Diff and restore files from this shadow store without interfering with the user’s primary git history.

### 2.5.3 Configuration

Configuration follows a four-tier hierarchy (evaluated top-down, project settings override all):
1. **Project-local settings** (`<project>/.opendev/settings.json`).
2. **User-global settings** (`~/.opendev/settings.json`).
3. **Environment variables**: Supply API credentials and CI/CD overrides.
4. **Built-in defaults**: Provide baseline behavior.

**Context window limits.** Derived automatically from model capabilities fetched from the provider cache, preventing misconfiguration.

### 2.5.4 Provider and Model Cache

The system caches model capabilities (context length, pricing) locally under `~/.opendev/cache/`. It uses a **stale-while-revalidate** strategy with a 24-hour TTL, ensuring the agent can start even when offline.
