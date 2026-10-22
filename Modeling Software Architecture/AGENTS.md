# Modeling Software Architecture: Agent Guidelines

## Purpose of This Folder

This folder is dedicated to the Udemy course **"Modeling Software Architecture: C4 viewpoints and perspectives"**.

Here we will:

- Create and maintain **concept notes** based on the course material.
- Discuss architecture topics, ask questions, and explore ideas together.
- Draw diagrams (C4 models, viewpoints, perspectives, and related visualizations).
- Learn and apply the C4 approach to modeling software architecture.

## 📁 Directory Structure & Purpose

Concept notes are grouped into **thematic subfolders** under `concept_notes/`, reflecting the structure of the course (foundations → viewpoints → perspectives → practice), with side reference folders for broader architecture frameworks/notations, cross-cutting best practices, and named anti-patterns beyond the course itself. This replaces a flat alphabetical listing so that related notes sit together and navigation mirrors the learning progression.

| Directory | Purpose | Contains |
|-----------|---------|----------|
| `concept_notes/foundations/` | **Base concepts** everything else builds on — what a model/system is, abstraction levels, mental models, communication. | `Model`, `System`, `System Architecture`, `System Modeling`, `Levels of Abstraction`, `Key Elements of Architecture View`, `Mental Model`, `Communicating the Model` |
| `concept_notes/viewpoints/` | **All viewpoints** — the structural lenses that produce views. Parent `Viewpoint` + 3 C4 levels (Context/Container/Component) + 7 Rozanski & Woods viewpoints (Functional, Information, Concurrency, Development, Deployment, Operational). | `Viewpoint` (parent), `Context Viewpoint`, `Container Viewpoint`, `Component Viewpoint`, `Functional Viewpoint`, `Information Viewpoint`, `Concurrency Viewpoint`, `Development Viewpoint`, `Deployment Viewpoint`, `Operational Viewpoint` |
| `concept_notes/perspectives/` | **All perspectives** — cross-cutting quality concerns applied *to* viewpoints. Parent `Perspective` + 7 core perspectives. | `Perspective` (parent), `Security`, `Performance and Scalability`, `Availability and Resilience`, `Evolution`, `Development Resource`, `Cost`, `Location` |
| `concept_notes/practice/` | **Process & application** — how to build a model, document it, version it, manage its complexity, and a worked example tying all viewpoints together. | `Architecture Model` _(how to create)_, `How to Document the Architecture Model` _(Proposal / ADD / ADR)_, `How to Version Control the Architecture Model` _(diagrams-as-code · drift)_, `Handling Complexity of Architecture Model`, `Worked Example - BookHub` |
| `concept_notes/frameworks-and-notations/` | **Broader architectural frameworks & notations** beyond the course itself. Top level holds the **meta-concepts** (parents + the four axes); concrete frameworks and notations live in `frameworks/` and `notations/` subfolders. | **Top-level meta:** `Architecture Framework` (parent), `Architecture Notation` (parent), and the four framework axes — `Architecture Process`, `Architecture Domains`, `Architectural Views`, `Architecture Metamodel`. **`frameworks/`:** `C4 Model`, `Viewpoints and Perspectives`, `4+1 View Model` _(+ `4+1/` deep dive)_, `TOGAF` _(+ `togaf/` deep dive: ADM, Content Framework, Capability Framework, BDAT domains, Partitioning, Time Horizons)_. **`notations/`:** `Flowchart`, `UML`, `ArchiMate`, `SysML`, `BPMN`, `Event-Storming` |
| `concept_notes/best-practices/` | **Cross-cutting how-to guidance** — practical rules for producing clear, audience-appropriate diagrams and artifacts. Sits alongside (not within) the course's conceptual framework. | `Best Practices for Diagrams` |
| `concept_notes/antipatterns/` | **Named anti-patterns** — the recurring ways architecture diagrams go wrong, as the inverse of best practices. Each entry maps 1:1 to the practice it violates. | `Anti-Patterns for Diagrams` |
| `flash_cards/` | Spaced-repetition cards for the material. Tagged `#flashcards/architecture-modeling/...`. | `Viewpoints-Flashcards`, `Perspectives-Process-Flashcards` |

### Where to put new notes

- New **base concept** (what something *is*) → `foundations/`
- New **viewpoint** (a structural lens producing a view) → `viewpoints/`
- New **perspective** (a quality concern applied to a view) → `perspectives/`
- New **process / worked example / how-to** → `practice/`
- New **cross-cutting practical guidance** (diagramming rules, communication tips, review checklists that apply across viewpoints and perspectives) → `best-practices/`
- New **named anti-pattern** (a recurring, catalogued failure — the inverse of a best practice) → `antipatterns/`
- New **architecture framework or visual notation** (e.g., a modeling approach, diagram standard, industry framework like C4 model, 4+1, TOGAF, UML, ArchiMate):
  - Meta-concept about frameworks/notations *as a category* (parents, the four axes: Process/Domains/Views/Metamodel) → `frameworks-and-notations/` top level
  - A **concrete framework** (C4, TOGAF, 4+1, R&W, …) → `frameworks-and-notations/frameworks/`
  - A **concrete notation** (UML, ArchiMate, SysML, BPMN, …) → `frameworks-and-notations/notations/`
- New **flashcards** → `flash_cards/` (follow the `#flashcards/architecture-modeling/<subsection>` tag convention)

> **Wikilinks are path-independent.** Notes use filename-only `[[WikiLinks]]` (e.g., `[[Viewpoint]]`, not `[[concept_notes/viewpoints/Viewpoint]]`), so Obsidian resolves them regardless of which subfolder a note lives in. Never add folder paths to wikilinks — keep them filename-only so notes can be reorganized freely.

## Note Conventions

- Use the **Technical-Deep-Dive** template from `Templates/` for evergreen concept notes.
- Link related notes with `[[WikiLinks]]`.
- Keep technical content in **English**; communication with the user is in **Russian**.
- Include proper frontmatter on every note:

  ```yaml
  ---
  status: researching | analyzing | brainstorming | implemented | archived
  tags: [c4-model, architecture, udemy, ...]
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  ---
  ```

- Update `updated:` on every edit.

## Anti-Patterns

- Do NOT dump raw course transcripts without analysis.
- Do NOT create notes without frontmatter or meaningful tags.
- Do NOT duplicate existing concepts — link to them instead.

---

*This folder is a learning space. Ask questions, challenge ideas, and build a clear understanding of software architecture modeling together.*

*Last updated: 2026-07-22*
