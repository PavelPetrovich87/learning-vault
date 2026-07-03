# `_reference/` — External reference repos (NOT vault notes)

This folder holds **cloned external repositories** used as reference material
(claude-code source, opendev, agent-architecture-atlas, etc.).

## ⚠️ This folder is excluded from vault conventions

- These are **not** user-authored notes. Do **not** apply AGENTS.md rules
  (frontmatter, English-only, linking) here.
- Excluded from git tracking (see root `.gitignore`) due to size (~335 MB)
  and the fact these repos can be re-cloned from their remotes at any time.
- Excluded from Obsidian graph/search indexing considerations.

## Contents

| Folder | What it is |
|--------|-----------|
| `CC_leaked/` | Claude Code source code (reference) |
| `opendev-repo/` | OpenDev source repo (Rust) |
| `opendev_report/` | Report notes derived from OpenDev analysis |
| `cc_report/` | Report notes derived from Claude Code analysis |
| `agent-architecture-atlas/` | Agent architecture research project (with node_modules) |

## To re-clone

If lost, re-clone from original remotes. URLs were not recorded here — check
git history of this folder or the original sources.
