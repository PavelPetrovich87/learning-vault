# Frontend Directory: Agent Guidelines

This file defines local conventions for the `Frontend/` directory of the learning vault.

## Scope

This directory contains frontend engineering learning modules. Each module is a self-contained topic folder that holds evergreen concept notes directly inside it.

Notes here are **framework-agnostic by default** — when a concept applies across React/Vue/Angular, write the principle generically and show framework-specific code side by side under "Examples Across Frameworks" (replacing the `Technical-Deep-Dive` template's "Examples Across Domains" section). Prefer TypeScript for all examples.

## Module Structure

Each module folder (e.g., `Layout Components/`) follows this structure:

```
<Module Name>/
├── <concept-note-1>.md
├── <concept-note-2>.md
└── flashcards/      # Spaced repetition flashcards (Obsidian SR)
```

## Current Modules

| Module | Purpose |
|--------|---------|
| `Layout Components/` | Layout & page-shell composition patterns — header/sidebar/main/footer regions, slots, sticky containers. Framework-agnostic. |
| `senior-front/` | Курс «Фронтенд senior+ в агентную эру»: концепт-ноты и флешки по главам. React как основной стек. Источник — глава 1.1 и далее (рендеринг-стратегии, архитектура, агентный фронтенд). |

## Note Rules

- Create **concept notes** directly inside the module folder.
- Create **flashcards** inside the module's `flashcards/` folder **when content warrants it** — do NOT pre-create empty folders (vault anti-pattern #6).
- Follow the vault-wide `AGENTS.md` for frontmatter, linking, and anti-patterns.
- Use `Templates/Technical-Deep-Dive.md` for concept notes.
- Tag concept notes with `frontend` plus topic-specific slugs (e.g., `layout-components`, `ui-architecture`).
- Code examples: show React, Vue, and Angular side by side when the concept is portable. Prefer TypeScript.

## Future Modules

Additional frontend learning modules (e.g., `State Management/`, `Forms/`, `Styling/`) will be added alongside `Layout Components/` as content matures.
