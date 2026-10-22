# DevOps Udemy Course — Agent Guidelines

This folder contains concepts, notes, and flashcards derived from the Udemy DevOps course.

## Folder Purpose

Store structured learning material tied to the course: key concepts, tools, commands, workflows, trade-offs, and extracted flashcards.

## Subfolder Structure

Course material is grouped by topic into subfolders. Each topic subfolder contains research notes and links out to stabilized concept notes.

Current subfolders:

- `linux/` — Linux fundamentals covered by the course (shell, filesystem, permissions, processes, networking, scripting, etc.).

Each topic subfolder contains:
- Research notes at the root of the subfolder
- `concept_notes/` — stabilized concept notes for that topic
- `flash_cards/` — spaced repetition flashcards for that topic

Add new topic subfolders here as new course sections are introduced.

## Note Lifecycle

### Research Notes
- **When to create:** While watching a course section, taking notes on a lecture, or analyzing a course demo.
- **Template:** `Templates/Research-to-Project Template.md`
- **Status:** `researching | analyzing | experimenting | implemented | archived`
- **Rule:** One document per course section or demo thread. Do not split a single lecture thread into multiple files.

### Concept Notes
- **When to create:** When a course concept stabilizes into a reusable, referenceable idea (e.g., "Linux File Permissions", "Process Management", "Bash Scripting").
- **Template:** `Templates/Technical-Deep-Dive.md`
- **Location:** Create the concept note in the relevant topic subfolder under `DEVOPS/<topic>/concept_notes/`.
- **Rule:** Link back to the originating research note here.

### Flashcards
- **When to create:** After completing a research note or concept note, extract key facts into flashcards.
- **Location:** `DEVOPS/<topic>/flash_cards/`
- **Syntax:** Obsidian Spaced Repetition (`Question :: Answer #flashcards/devops` or multi-line format).
- **Rule:** One card = one concept. Frame questions around problem-solving, tools, commands, trade-offs, and gotchas.

### Flashcards in the AI Era — Design Principle

Before generating flashcards, the agent MUST first reason explicitly about **which competence the card builds**, in the context of an AI-enhanced DevOps workflow where recall is cheap and on-demand. Knowledge separates into three layers with very different ROI:

1. **Recall (flags, syntax, exact options).** AI obsoletes this. Do NOT create flashcards for flag tables, option lists, or syntax memorization. These are lookups.
2. **Vocabulary (knowing a tool exists and what problem family it solves).** Necessary but insufficient. A small number of cards per topic is enough — the goal is recognition, not exhaustive coverage.
3. **Mental models & composition (data-flow reasoning, tool selection, ordering, debugging, gotchas).** This is the layer that compounds and that AI cannot install in the learner's head. **Prioritize this layer.**

When asked to create flashcards, the agent must:
- **First produce a short analysis** of the topic: which competences matter, how AI shifts their value, and which layer each candidate card targets.
- **Favor cards that build the model** — "why does `uniq` require sorted input?", "what's wrong with `command 2>&1 > file`?", "predict the output of this pipeline" — over cards that test recall ("what flag sorts numerically?").
- **Include composition and debug-the-AI cards** where relevant: predict-the-output drills, planted-bug detection, pipeline-ordering reasoning.
- **Store lookup material (flag tables, syntax references) in the concept note**, not in flashcards. Flashcards are the wrong home for reference data.

A good heuristic: if a card's answer can be rephrased as "look it up," it should not exist. If it builds judgment that survives the existence of AI, it should.

## Naming Conventions

- Research notes: `{SectionNumber}_{Topic_Name}.md` or `{CourseName}_{Topic_Name}.md`
  - Examples: `01_Linux_Basics.md`, `DevOps_Linux_File_System.md`
- Prefer `Title_Case_Snake_Case.md` for file names.

## Frontmatter

All notes MUST include:

```yaml
---
status: researching | analyzing | brainstorming | implemented | archived
tags: [devops, udemy, research]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Update `updated:` on every edit.

## Linking

- Link to related concept notes with `[[WikiLinks]]`.
- Link to the source lecture/section if applicable.
- Maintain a "Related Notes" section at the bottom of research notes.

## Anti-Patterns

- Do NOT use this folder for raw brainstorming; use `Заметки на будущее/` instead.
- Do NOT create flashcards in the root of this folder; use the `flash_cards/` subfolder within the relevant topic directory.
- Do NOT move notes out of this folder to "ripen" them. Create new concept notes elsewhere and link back.

---

*Last updated: 2026-07-11*
