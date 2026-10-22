# BA for Developer — Vault Conventions

> This section holds notes and flashcards for the **"BA for Developer"** course
> (the `ba-dev-trainer/` repo, based on Wiegers & Beatty, *Software
> Requirements*, 3rd ed., 2014). Each course chapter becomes one folder
> directly under `Business Analysis/`.

## Folder structure

```
Business Analysis/
├── AGENTS.md                                  ← this file
└── <N-M-slug>/                                ← one folder per course chapter
    ├── concept-note.md                        ← the chapter concept note (EN)
    └── flashcards/
        └── flashcards.md                      ← chapter flashcards (EN)
```

### Rules

- **One chapter = one folder.** The folder slug mirrors the course chapter id
  in the form `<module>-<chapter>-<slug>`, e.g. `1-1-requirements-foundations`,
  `2-1-vision-and-scope`. Do **not** flatten multiple chapters into a single
  folder, and do **not** nest chapter folders inside each other.
- **Two files per chapter**, no more:
  1. `concept-note.md` — the consolidated chapter note.
  2. `flashcards/flashcards.md` — the chapter's flashcard file.
- **Flashcards live in their own subfolder** (`flashcards/`), not next to the
  concept note. The flashcard file name mirrors the folder: `flashcards.md`.
- **Do not pre-create empty chapter folders.** Create a chapter folder only
  when its note is ready (anti-pattern #6 from the vault conventions).
- **Language: English only inside the vault.** Russian quotes from the source
  book are allowed inside `<blockquote>` citations when the wording matters,
  but all explanation, headings, frontmatter values, and flashcards are in
  English. Course-side Russian lives in the `ba-dev-trainer/` repo, not here.

## Frontmatter (concept notes)

```yaml
---
created: 'YYYY-MM-DD'
updated: 'YYYY-MM-DD'
status: analyzing          # researching | analyzing | brainstorming | experimenting | implemented | archived
difficulty: beginner       # beginner | intermediate | advanced
mastery: beginner          # beginner | growing | mastered
source: "Wiegers & Beatty, Software Requirements (3rd ed., 2014), Ch. N, pp. X–Y"
tags:
  - business-analysis      # extra_tags from the section (always include)
  - ba-dev-trainer
  - wiegers
  - chapter-N-M            # e.g. chapter-1-1
---
```

## Flashcards

- Deck tag: `#flashcards/business-analysis` (single deck for the whole course;
  chapter is encoded in the `chapter-N-M` tag on the *concept note*, not in the
  deck tag).
- Format: `multi` (Q on one line, `?` separator, A below) — matches the rest
  of the vault.
- File header: `#flashcards/business-analysis` on the first line.
- Each card ends with `---` and a blank line.
- Source link: add `Source: [[concept-note]]` (filename-only wikilink) at the
  end of each card when the card is derived from a chapter note.

## Wikilinks

- Use **filename-only** wikilinks (`[[1-1 Requirements Foundations — Concept Note]]`),
  never path-based (`[[1-1-requirements-foundations/concept-note]]`).
  Filename-only links survive reorganisation.

## Cross-references

- The source repo: `~/learning/учебники/Kimi_Agent_Бизнес-Dev/ba-dev-trainer/`.
- Progress and chapter list: `progress.md` and `Карта привязок источников.md`
  in that repo.
- Each chapter note ends with a **References** section linking back to the
  repo chapter folder and the book chapter PDF.
