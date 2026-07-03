# Backend Directory: Agent Guidelines

This file defines local conventions for the `Backend/` directory of the learning vault.

## Scope

This directory contains backend engineering learning modules. Each module is a self-contained topic folder that holds evergreen concept notes directly inside it.

## Module Structure

Each module folder (e.g., `Nest.js/`) follows this structure:

```
<Module Name>/
├── <concept-note-1>.md
├── <concept-note-2>.md
└── flashcards/      # Spaced repetition flashcards (Obsidian SR)
```

## Current Modules

| Module | Purpose |
|--------|---------|
| `Nest.js/` | NestJS framework learning — concepts, patterns, CLI usage, and architecture. |

## Note Rules

- Create **concept notes** directly inside the module folder.
- Create **flashcards** inside the module's `flashcards/` folder.
- Follow the vault-wide `AGENTS.md` for frontmatter, linking, and anti-patterns.
- Use `Templates/Technical-Deep-Dive.md` for concept notes.

## Future Modules

Additional backend learning modules will be added alongside `Nest.js/` as needed.
