# System Design Udemy Course — Agent Guidelines

This folder contains concepts, notes, and flashcards derived from the Udemy System Design course.

## Folder Purpose

Store structured learning material tied to the course: key concepts, architectural patterns, trade-offs, case studies, and extracted flashcards.

## Subfolder Structure

Course material is grouped by topic into subfolders. Each subfolder contains research notes and links out to stabilized concept notes.

Current subfolders:

- `networking/` — networking fundamentals covered by the course (protocols, DNS, TCP/UDP, load balancing, CDNs, etc.).
  - `networking/ip/` — IP addresses: IPv4, IPv6, public/private IPs, NAT, CIDR.
  - `networking/dns/` — DNS concepts, resolution process, caching, security, and large-scale DNS strategies.
  - `networking/client_server_model/` — client-server architecture, request-response, stateless vs stateful, communication patterns.
- `protocols/` — communication protocols covered by the course (TCP, UDP, HTTP, WebSocket, gRPC, etc.).

Each topic subfolder contains:
- Research notes at the root of the subfolder
- `concept_notes/` — stabilized concept notes for that topic
- `flash_cards/` — spaced repetition flashcards for that topic

Add new topic subfolders here as new course sections are introduced.

## Note Lifecycle

### Research Notes
- **When to create:** While watching a course section, taking notes on a lecture, or analyzing a course case study.
- **Template:** `Templates/Research-to-Project Template.md`
- **Status:** `researching | analyzing | experimenting | implemented | archived`
- **Rule:** One document per course section or case-study thread. Do not split a single lecture thread into multiple files.

### Concept Notes
- **When to create:** When a course concept stabilizes into a reusable, referenceable idea (e.g., "Consistent Hashing", "Rate Limiting", "CQRS").
- **Template:** `Templates/Technical-Deep-Dive.md`
- **Location:** Create the concept note in the relevant topic subfolder under `system-design-udemy/<topic>/concept_notes/`.
- **Rule:** Link back to the originating research note here.

### Flashcards
- **When to create:** After completing a research note or concept note, extract key facts into flashcards.
- **Location:** `system-design-udemy/<topic>/flash_cards/`
- **Syntax:** Obsidian Spaced Repetition (`Question :: Answer #flashcards/system-design` or multi-line format).
- **Rule:** One card = one concept. Frame questions around problem-solving, architecture, trade-offs, and gotchas.

## Naming Conventions

- Research notes: `{SectionNumber}_{Topic_Name}.md` or `{CourseName}_{Topic_Name}.md`
  - Examples: `01_Introduction_To_System_Design.md`, `Consistent_Hashing_Deep_Dive.md`
- Prefer `Title_Case_Snake_Case.md` for file names.

## Frontmatter

All notes MUST include:

```yaml
---
status: researching | analyzing | brainstorming | implemented | archived
tags: [system-design, udemy, research]
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

*Last updated: 2026-07-02*
