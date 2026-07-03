# System Design interview — Agent Guidelines

This folder holds **evergreen concept notes** and **spaced-repetition flashcards** for system design and architecture topics that are **not** tied to the Udemy course. For Udemy course material, use `system-design-udemy/` instead.

## Folder Purpose

Atomic, self-contained, referenceable notes on distributed systems, scalability, architecture patterns, and interview-grade system design. This is the home for "I want to deeply understand X" notes that have stabilized beyond the research phase.

## Subfolder Structure

```
System Design interview/
├── concept_notes/      ← Evergreen concept notes (Technical-Deep-Dive template)
│   ├── DDD/            ← Domain-Driven Design notes
│   ├── Patterns/       ← Design pattern application notes
│   ├── calculations/   ← Back-of-the-envelope estimation notes
│   └── front_end/      ← Frontend architecture notes
├── flash_cards/        ← Spaced repetition cards (Obsidian SR syntax)
└── AGENTS.md           ← This file
```

The topic subfolders under `concept_notes/` (DDD, Patterns, calculations, front_end) are **organic groupings** — create a new one only when ≥3 notes share a clear theme. Do not pre-create empty topic folders.

## What belongs here vs elsewhere

| Topic | Belongs in | NOT here |
|-------|-----------|----------|
| Distributed systems, CAP/PACELC, replication, partitioning | ✅ here | — |
| URL shortener, KV store, latency budget design | ✅ here | — |
| NestJS / framework specifics | ❌ | `Backend/Nest.js/` |
| CI/CD, GitHub Actions, deployment auth | ❌ | `DEVOPS/ci-cd/` |
| LLM observability, Langfuse, prompt ops | ❌ | `agentic_ai/observability/` |
| Agentic patterns (orchestrator-worker, evaluator-optimizer) | ❌ | `agentic_ai/patterns/` |
| Udemy course sections | ❌ | `system-design-udemy/` |

If a note drifts from system design into another domain, **move it** to the right home and leave a short link behind if useful.

## Note Lifecycle

### Concept Notes
- **When to create:** When a research topic stabilizes into a reusable, referenceable concept.
- **Template:** `Templates/Technical-Deep-Dive.md`
- **Rule:** One concept per note. Atomic. Link to parent/related concepts via `[[WikiLinks]]`.
- **Status field:** `analyzing | experimenting | implemented | archived` (work lifecycle).
- **Mastery field (optional):** `beginner | growing | mastered` (your learning progress).

### Flashcards
- **When to create:** After a concept note is stable, extract key facts into flashcards.
- **Syntax:** Obsidian Spaced Repetition
  - Single-line: `Question :: Answer #flashcards/system-design`
  - Multi-line: `Question` / `?` / `Answer` / `#flashcards/system-design`
- **Rule:** One card = one concept. Senior-level framing (trade-offs, gotchas, "when NOT to").
- **Rule:** Filenames MUST end in `.md` (no extensionless files) and ideally match the parent concept note name with a `-Flashcards` suffix.
- **Rule:** Link back to the source concept note at the top of the file: `Source: [[Concept Note Name]]`.

## Anti-patterns specific to this folder
1. **Do NOT** dump framework-specific notes here (NestJS, Langfuse, etc.) — they have proper homes.
2. **Do NOT** create extensionless flashcard files — always `.md`.
3. **Do NOT** pre-create empty topic subfolders under `concept_notes/`.
4. **Do NOT** leave stub/placeholder concept notes (unfilled templates). Either fill them or delete.

*Last updated: 2026-07-03*
