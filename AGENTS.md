# Obsidian Vault: Agent Guidelines

This file defines the structure, conventions, and rules for any AI agent interacting with this vault. All agents MUST read this file before creating, modifying, or moving notes.

## Vault Overview

This is a **personal technical learning vault** dedicated to deep knowledge management for a Senior Software Engineer. Communication with the user is in **Russian**.

### Language policy (bilingual by zone)
- **Concept notes, flashcards, evergreen reference** → English (these are long-lived reference material).
- **Working zones** (`research_in_progress/`, `daily-plans/`, `Заметки на будущее/`, `harness_engineering/planning/`) → **bilingual: Russian or English**, author's choice. These are living documents, not reference artifacts.
- **Templates** → English for structure; Russian is fine inside callouts/placeholders if that matches how the author drafts.

---

## 📁 Directory Structure & Purpose

| Directory | Purpose | Agent Action |
|-----------|---------|--------------|
| `research_in_progress/` | **Active research notes.** Article analysis, deep dives, experiments, PoC ideas. One document per research thread. | Create research notes here using `Templates/Research-to-Project Template.md`. |
| `notes/` | **Current evergreen concept notes** tied to ongoing work (eloomi-next architecture etc.). Latest note format: Russian, `Technical-Deep-Dive` structure, full frontmatter incl. `mastery`. | Create concept notes here when they grow out of current work discussions; use `Templates/Technical-Deep-Dive.md`. |
| `System Design interview/` | **System Design & Architecture** — evergreen concept notes and flashcards on distributed systems, scalability, patterns. NOT tied to the Udemy course. | Read `System Design interview/AGENTS.md`. Create concept notes in `concept_notes/` (use `Templates/Technical-Deep-Dive.md`), flashcards in `flash_cards/`. |
| `Backend/` | Backend engineering learning modules. Contains topic-specific subfolders (e.g., `Nest.js/`, `Microservices/`). | Create backend/API concept notes in the relevant module folder. |
| `Backend/Nest.js/` | NestJS framework concepts, CLI usage, architecture, and patterns. The canonical home for ALL NestJS notes (do not use a root-level `NestJS/` folder). | Create NestJS evergreen notes using `Templates/Technical-Deep-Dive.md`. |
| `Backend/Microservices/` | Backend architecture & design concept notes (Clean Architecture, Hexagonal Architecture, DDD). | Create architecture-specific concept notes here; group related topics into subfolders such as `Hexagonal Architecture/` and `Domain-Driven Design/`. |
| `Backend/Microservices/Hexagonal Architecture/examples/` | Runnable code examples for Hexagonal Architecture concepts. Each project is self-contained and includes a README with run instructions. | Create a new subfolder per example project inside the relevant topic folder. Link back to related concept notes in README. |
| `Backend/Microservices/Domain-Driven Design/` | DDD concept notes — Bounded Context, Aggregates, Aggregate Root, Domain Services, Application Services, Entities, Value Objects, Domain Events, and the parent DDD overview. | Create DDD-specific concept notes here using `Templates/Technical-Deep-Dive.md`. The parent note is `Domain-Driven Design.md`. |
| `Business Analysis/` | Business analysis, requirements engineering, domain modeling, and process design. | Create BA-related concept notes here using the appropriate template. |
| `system-design-udemy/` | **Udemy System Design Course** material — research notes, concepts, and flashcards tied to the course. | Read `system-design-udemy/AGENTS.md`. Default location for all system-design course notes unless user specifies otherwise. |
| `system-design-course/` | **System Design Trainer course** (repo `~/learning/учебники/system-design`) — Russian chapter summaries with tables, mermaid diagrams, and cross-links. | Read `system-design-course/AGENTS.md`. One file per chapter in `module-N/`; synced by the repo agent after each chapter acceptance. |
| `ai-consulting-course/` | **Coursera: IBM Enterprise AI Transformation and Consulting Professional Certificate** — по папке на курс. Курс 1 *The AI Transformation Advisor* (неделя с 2026-10-15), курс 2 *Identify and Prioritize AI Opportunities*. | Read `ai-consulting-course/course-N-*/AGENTS.md`. Транскрипт из чата → конспект `N.N-*.md` + `*-Flashcards.md` в `module-N-*/` внутри папки курса, карта в `INDEX.md` курса. |
| `DEVOPS/` | **Udemy DevOps Course** material plus general CI/CD concepts. Topic subfolders (`linux/`, `ci-cd/`) each contain `concept_notes/` + `flash_cards/`. | Read `DEVOPS/AGENTS.md` for subfolder conventions and templates. |
| `DEVOPS/ci-cd/` | CI/CD, GitHub Actions, deployment, visual regression (Chromatic). | Non-Udemy CI/CD concept notes go here (in `concept_notes/`). |
| `agentic_ai/` | Agentic AI concepts, patterns, and implementation guides for LLM-based agents. Subfolders: `patterns/`, `llm_fundamentals/`, `memory_and_state/`, `tool_use_and_mcp/`, `observability/`, `tooling/`, `OpenAI SDK/`, `crewAI/`. | LLM-observability notes (Langfuse, prompt ops, eval) go in `observability/`. CrewAI framework notes go in `crewAI/`. |
| `Modeling Software Architecture/` | Software architecture modeling concepts (C4, system context, etc.). | Create modeling concept notes here. |
| `git/` | Git internals, advanced workflows, shadow trees. Concept notes in `git/concept_notes/`. | Create git concept notes here. |
| `playwright/` | Playwright end-to-end testing — architecture, patterns, CI/CD integration, debugging, and tooling. | Read `playwright/AGENTS.md` for subfolder conventions and templates. |
| `harness_engineering/` | Project-specific notes for the harness engineering project: `open_ai_article/`, `designing-css-architecture/`, `learning/`, `planning/` (RFCs, plans, specs). | Project planning docs go in `harness_engineering/planning/`. |
| `project_august/` | **DELETED 2026-10-21** (project closed; recoverable via `git checkout HEAD -- project_august`). | — |
| `daily-plans/` | Daily planning and review notes (one file per day). **Exception zone:** lightweight frontmatter only (`date`, `type`, `created`, `updated`); no `status`/`tags`/`mastery` required. | Use `Templates/Daily-Plan-and-Review.md`. |
| `Заметки на будущее/` | **Raw ideas, brainstorming fragments, quick thoughts.** Low structure, high entropy. | Place ephemeral ideas here using `Templates/Future-Note.md`. Do NOT place structured research here. |
| `Templates/` | Note templates. | Use appropriate template when creating notes. |
| `articles_to_analyze/` | External articles and documents pending analysis (input only — `.docx`, `.pdf`, source articles). | Read from here, but do NOT create new notes here. |
| `books/` | Book source files (PDFs, docx) and summaries. | Add `.md` summaries when reading a book. |
| `projects/` | Project card templates (`project_cards/`). | — |
| `Skill Creation/` | Notes on authoring agent skills (description structure, context models). | — |
| `Self-hypnosis-app-DDD/` | DDD exploration for the self-hypnosis app project. | — |
| `sleight-of-mouth/` | **Дилтс «Фокусы языка» (Sleight of Mouth)** — проект модульного представления книги: модули с алгоритмами работы с убеждениями и их композиции под задачи. Distilled knowledge base included at `sleight-of-mouth/knowledge/`; extraction pipeline in `~/work/Dielts/`. | Read `sleight-of-mouth/AGENTS.md` first. Modules are created only on explicit user request. |
| `_reference/` | **EXCLUDED ZONE.** Cloned external reference repositories (claude-code source, opendev, agent-architecture-atlas, etc.). NOT vault notes. | Do NOT apply any vault convention here. Do NOT edit, link, or count these files. Gitignored. |

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
status: researching | analyzing | drafting | brainstorming | experimenting | implemented | stable | archived
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
reviewed: YYYY-MM-DD
---
```

- `status`: **Work lifecycle** of the note. Allowed values: `researching | analyzing | drafting | brainstorming | experimenting | implemented | stable | archived`. Required on all notes except exception zones (see below). `stable` = verified, maintained reference (evergreen); `archived` = finished course/project material, kept for reference only.
- `reviewed`: **Optional.** Date of the last content review (an evergreen-note "spaced repetition"). Set/update it when you re-read and confirm or refresh a note; see the audit dashboard for `reviewed`-based queries.
- `mastery`: **Learning progress** of the author for this topic. Optional, concept notes only. Allowed values: `beginner | growing | mastered`. Use this instead of overloading `status` (e.g., the old `status: new`/`growing` were this concept misfiled).
- `tags`: Relevant topic tags. Research notes should include `[research, in-progress]`.
- `created`: Date of creation (`YYYY-MM-DD`).
- `updated`: Date of last modification. **Update on every edit.**

### Exception zones (lighter frontmatter)
The following zones do NOT require `status`/`tags`/`mastery` because they are not reference notes:
- `daily-plans/` — needs only `date`, `type`, `created`, `updated`.
- `System Design interview/flash_cards/`, `system-design-udemy/**/flash_cards/`, `DEVOPS/**/flash_cards/` — flashcards are SR cards; frontmatter optional (a `Source: [[note]]` link at top is preferred over frontmatter).
- `_reference/` — fully excluded, see above.

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
3. **Do NOT create flashcards outside designated `flash_cards/` folders.** Use `System Design interview/flash_cards/`, `system-design-udemy/**/flash_cards/`, or `DEVOPS/**/flash_cards/`. Flashcard files MUST end in `.md` (no extensionless files).
4. **Do NOT split a research thread into multiple files by phase.** Keep it in one note.
5. **Do NOT use corporate speak.** Speak like a senior engineer talking to a colleague.
6. **Do NOT pre-create empty "placeholder" folders.** Only create a folder when you have content for it. (Empty top-level folders `AI-Core/`, `Learning_notes/` pre-date this rule and are pending either content or deletion.)
7. **Do NOT create duplicate copies of a note across folders.** One canonical home per note; link to it with `[[WikiLinks]]` instead of copying.
8. **Do NOT overload `status` with learning-progress values** (`new`, `growing`, `mastered`). Use the separate `mastery` field for that.
9. **Language:** Concept notes, flashcards, and evergreen reference MUST be in English (anti-pattern #6 from prior revision is relaxed for working zones — see Language policy above).

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
| `DEVOPS/AGENTS.md` | DevOps folder sub-conventions. |
| `system-design-udemy/AGENTS.md` | Udemy SD course sub-conventions. |
| `System Design interview/AGENTS.md` | System Design evergreen notes sub-conventions. |
| `Backend/AGENTS.md` | Backend modules sub-conventions. |
| `Modeling Software Architecture/AGENTS.md` | Architecture modeling sub-conventions. |
| `playwright/AGENTS.md` | Playwright sub-conventions. |

---

*Last updated: 2026-10-21 (vault cleanup: project_august deleted, enums normalized, zone statuses set, notes/ documented)*
