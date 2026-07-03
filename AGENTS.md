# Obsidian Vault: Agent Guidelines

This file defines the structure, conventions, and rules for any AI agent interacting with this vault. All agents MUST read this file before creating, modifying, or moving notes.

## Vault Overview

This is a **personal technical learning vault** dedicated to deep knowledge management for a Senior Software Engineer. Content is primarily in **English**, communication with the user is in **Russian**.

---

## 📁 Directory Structure & Purpose

| Directory | Purpose | Agent Action |
|-----------|---------|--------------|
| `research_in_progress/` | **Active research notes.** Article analysis, deep dives, experiments, PoC ideas. One document per research thread. | Create research notes here using `Templates/Research-to-Project.md`. |
| `System Design interview/concept_notes/` | **System Design & Architecture** concepts — evergreen notes on distributed systems, scalability, patterns. | Create system-design-specific concept notes here. |
| `Backend/` | Backend engineering learning modules. Contains topic-specific subfolders (e.g., `Nest.js/`). | Create backend/API concept notes in the relevant module folder. |
| `Backend/Nest.js/` | NestJS framework concepts, CLI usage, architecture, and patterns. | Create NestJS evergreen notes using `Templates/Technical-Deep-Dive.md`. |
| `Backend/Microservices/` | Backend architecture & design concept notes (Clean Architecture, Hexagonal Architecture, DDD). | Create architecture-specific concept notes here; group related topics into subfolders such as `Hexagonal Architecture/`. |
| `Backend/Microservices/Hexagonal Architecture/examples/` | Runnable code examples for Hexagonal Architecture concepts. Each project is self-contained and includes a README with run instructions. | Create a new subfolder per example project inside the relevant topic folder. Link back to related concept notes in README. |
| `Backend/Nest.js/flashcards/` | NestJS spaced repetition flashcards. | Generate from NestJS concept notes using Obsidian SR syntax. |
| `Business Analysis/` | Business analysis, requirements engineering, domain modeling, and process design. | Create BA-related concept notes here using the appropriate template. |
| `System Design interview/flash_cards/` | Spaced Repetition flashcards in Obsidian SR syntax. | Generate flashcards from concept notes and research notes. |
| `system-design-udemy/` | **Udemy System Design Course** material — research notes, concepts, and flashcards tied to the course. | Default location for all system-design course notes unless user specifies otherwise. |
| `DEVOPS/` | **Udemy DevOps Course** material — research notes, concepts, and flashcards tied to the course. | Read `DEVOPS/AGENTS.md` for subfolder conventions and templates. |
| `Заметки на будущее/` | **Raw ideas, brainstorming fragments, quick thoughts.** Low structure, high entropy. | Place ephemeral ideas here. Do NOT place structured research here. |
| `Templates/` | Note templates. | Use appropriate template when creating notes. |
| `articles_to_analyze/` | External articles and documents pending analysis. | Read from here, but do NOT create new notes here. |
| `books/` | Book notes and summaries. | — |
| `agentic_ai/` | Agentic AI concepts, patterns, and implementation guides for LLM-based agents. Use subfolders like `patterns/`, `llm_fundamentals/`, `memory_and_state/`, `tool_use_and_mcp/`, `observability/` to group related topics.
| `harness_engineering/` | Project-specific notes for the harness engineering project. | — |
| `playwright/` | Playwright end-to-end testing — architecture, patterns, CI/CD integration, debugging, and tooling. | Read `playwright/AGENTS.md` for subfolder conventions and templates.
| `project_august/` | Project-specific notes. | — |

---

## 📝 Note Lifecycle & Rules

### 1. Research Notes (`research_in_progress/`)
- **When to create:** After reading an article, watching a talk, or starting a deep investigation.
- **Template:** `Templates/Research-to-Project Template.md`
- **Rule:** ONE document per research thread. Do NOT split into multiple files by phase. All phases (Extraction, Analysis, Application, Action) live in the same note.
- **Rule:** Do NOT move research notes between folders to "ripen" them. They stay here. If a concept matures, create a new Concept Note and link back.
- **Status field:** Use frontmatter `status: researching | analyzing | experimenting | implemented | archived`

### 2. Concept Notes (Domain Folders)
- **When to create:** When a research topic has stabilized into a reusable, referenceable concept.
- **Template:** `Templates/Technical-Deep-Dive.md`
- **Rule:** These are "Evergreen Notes" — atomic, self-contained, linked to other concepts.
- **Rule:** Create concept notes in the **most specific domain folder** that matches the topic (e.g., `Backend/` for NestJS, `agentic_ai/` for LLM agents, `Business Analysis/` for DDD). Use `System Design interview/concept_notes/` **only** for system-design and architecture-specific topics that are **not** tied to the Udemy course. For Udemy course notes, use `system-design-udemy/concept_notes/` by default.

### 3. Future Notes (`Заметки на будущее/`)
- **When to create:** For raw ideas at 3 AM, quick thoughts, brainstorming fragments.
- **Template:** `Templates/Future-Note.md`
- **Rule:** Do NOT place structured research or article analysis here. This is for ephemeral thoughts only.
- **Rule:** If a Future Note matures, copy/re-create it as a Research Note or Concept Note. Do NOT move the original (preserve backlinks).

### 4. Flashcards (`System Design interview/flash_cards/` and `system-design-udemy/flash_cards/`)
- **When to create:** After completing a research note or concept note, extract key facts into flashcards.
- **Syntax:** Obsidian Spaced Repetition
  - Single-line: `Question :: Answer #flashcards/system-design`
  - Multi-line:
    ```markdown
    Question text
    ?
    Answer text
    #flashcards/system-design
    ```
- **Rule:** One card = one concept. No cards requiring more than 3 items.
- **Rule:** Frame questions around problem-solving, architecture, trade-offs, and "gotchas" (Senior-level framing).
- **Rule:** Do NOT duplicate existing flashcards.

---

## 🏷️ Frontmatter Conventions

All notes MUST include frontmatter:

```yaml
---
status: researching | analyzing | brainstorming | implemented | archived
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

- `status`: Current state of the note.
- `tags`: Relevant topic tags. Research notes should include `[research, in-progress]`.
- `created`: Date of creation.
- `updated`: Date of last modification (update on every edit).

---

## 🔗 Linking & Navigation

- Use `[[WikiLinks]]` to connect related notes.
- When creating a research note, link to:
  - The source article/file (e.g., `[[Claude_Code_Dynamic_Workflows_Анализ]]`)
  - Related concept notes
  - Any relevant future ideas
- Maintain a "Related Notes" section at the bottom of each research note.

---

## 🚫 Anti-Patterns (What NOT to Do)

1. **Do NOT move notes between folders to "ripen" them.** Create new notes and link back.
2. **Do NOT create research notes in `Заметки на будущее/`.** That's for raw ideas only.
3. **Do NOT create flashcards outside designated flash_cards folders.** Use `System Design interview/flash_cards/` for general system-design topics, and `system-design-udemy/flash_cards/` for Udemy course flashcards.
4. **Do NOT split a research thread into multiple files by phase.** Keep it in one note.
5. **Do NOT use corporate speak.** Speak like a senior engineer talking to a colleague.
6. **Do NOT write in Russian in vault files.** All technical content is in English; communication with the user is in Russian.

---

## ✅ Agent Checklist (Before Any Session)

- [ ] Read this `AGENTS.md` file.
- [ ] Read `CLAUDE.md` if you are Claude (personality and style guidelines).
- [ ] Check `Templates/` for the appropriate template before creating a note.
- [ ] Verify the target folder matches the note type.
- [ ] Update `updated:` in frontmatter on every edit.

---

## 📚 Key Files for Context

| File | Purpose |
|------|---------|
| `AGENTS.md` | This file — structure and rules for all agents. |
| `CLAUDE.md` | Personality, tone, and style guidelines for Claude specifically. |
| `DDD_CONTEXT.md` | Domain-Driven Design context for specific projects. |
| `GEMINI.md` | Guidelines for Gemini interactions (if applicable). |

---

*Last updated: 2026-07-01*
