# Playwright: Agent Guidelines

This folder is dedicated to **Playwright end-to-end testing** — test architecture, patterns, CI/CD integration, debugging, and tooling.

## 📁 Directory Structure

| Directory | Purpose | Agent Action |
|-----------|---------|--------------|
| `concept_notes/` | Evergreen Playwright concepts: selectors, fixtures, page object model, parallelization, retries, tracing, sharding, CI integration. | Create concept notes using `Templates/Technical-Deep-Dive.md`. |
| `flashcards/` | Spaced repetition cards for Playwright facts and gotchas. | Generate from concept notes using Obsidian SR syntax. |
| `research_in_progress/` | Active investigations, article deep dives, experiments. | Use `Templates/Research-to-Project Template.md`. |

## 📝 Conventions

- Content is in **English**.
- Use `[[WikiLinks]]` to connect to related backend/frontend testing notes.
- Link related System Design notes where relevant (e.g., CI/CD, visual regression).
- Tag Playwright notes with `[playwright, testing, e2e]` plus topic-specific tags.

## 🏷️ Frontmatter

Same as the root `AGENTS.md`:

```yaml
---
status: researching | analyzing | brainstorming | implemented | archived
tags: [playwright, testing, e2e]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

---

*Last updated: 2026-06-29*
