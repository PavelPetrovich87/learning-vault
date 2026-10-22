---
status: archived
tags: [c4-model, architecture, togaf, content-framework, diagrams, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Architecture Diagrams

> [!abstract] Executive Summary (ELI5)
> A **diagram** in TOGAF is a **visual picture** of a selected set of entities and their relationships. It is the most communicative of the three [[Architecture Content Framework|Content Framework]] artifact types (catalogs, matrices, diagrams). Its job is **communication**: to let a stakeholder *see* the architecture — the big picture, the flows, the boundaries — in a way that [[Architecture Catalogs|catalogs]] (lists) and [[Architecture Matrices|matrices]] (grids) cannot. A diagram is a **projection** of the underlying metamodel, drawn for a specific audience.

> [!warning] The #1 confusion — TOGAF diagrams are not notation-defined
> TOGAF names many diagram types (Business Footprint diagram, Application Communication diagram, etc.) but it is **notation-agnostic** — it does not define *how to draw the boxes*. You render TOGAF diagrams in [[C4 Model|C4]], ArchiMate, or UML. The diagram *type* comes from TOGAF; the *symbols* come from a notation. See [[Architectural Views]].

> [!note] Part of the TOGAF deep-dive
> One of three artifact types in the [[Architecture Content Framework]]. Sibling shapes: [[Architecture Catalogs]] (lists), [[Architecture Matrices]] (grids).

---

## 🎯 The "Why"

- **Problem Solved:** Catalogs and matrices are precise but unreadable to most stakeholders. A sponsor, a business owner, or a new team member needs to *see* the architecture — the boundary, the flows, the clusters — to understand it. Diagrams turn the metamodel data into a picture tuned to a specific audience and concern.
- **Alternatives:** Showing stakeholders the raw catalog or matrix. Accurate, but incomprehensible to non-architects. The diagram is the **communication layer** on top of the data layer.
- **Trade-offs:** Diagrams communicate well but **lose completeness and precision** — a diagram is a *subset* (you cannot draw 500 applications legibly). The risk is that a diagram *hides* entities that the catalog shows. Always pair diagrams with the catalog/matrix that generated them.

## 🛠️ Core Principles

1. **A diagram is a projection, not the source of truth.** The entities come from [[Architecture Catalogs|catalogs]] and the relationships from [[Architecture Matrices|matrices]]. The diagram shows a *selected subset*, visually. Change the catalog → regenerate the diagram.
2. **A diagram targets an audience and a concern.** Like a [[Viewpoint|view]], each diagram is drawn for specific stakeholders answering a specific question. The Business Footprint diagram is for executives; the Application Communication diagram is for integration architects.
3. **TOGAF names diagram types but not their symbols.** TOGAF defines *which* diagrams to produce (by purpose); the *notation* (C4, ArchiMate, UML) is your choice. This is why TOGAF + ArchiMate is a common pairing.
4. **Diagrams trade completeness for readability.** You cannot draw everything legibly. A good diagram shows the architecturally significant subset and explicitly notes what was elided.

## 🏗️ Common TOGAF Diagrams (by domain)

| Domain | Common diagrams | What they show | Audience |
|---|---|---|---|
| **Business** | Business Footprint diagram, Functional Decomposition diagram, Value Chain diagram, Goal/Objective/Service diagram | Capabilities, goals, value flow | Executives, business owners |
| **Data** | Data Dissemination diagram, Data Migration diagram, Data Security diagram | Where data flows/migrates, who sees it | Data architects, security |
| **Application** | Application Communication diagram, Application and User Location diagram, Software Engineering diagram, Application Migration diagram | Which apps talk to which, where they sit, migration path | Integration architects, devs |
| **Technology** | Computing Hardware diagram, Networked Computing/Hardware diagram, Platform Service diagram | Nodes, networks, infrastructure | Infrastructure, ops |
| **Cross** | Solution Concept diagram, Enabling and Support diagram | High-level solution, support view | Mixed |

> The **Business Footprint diagram** (shows capabilities and the goals they serve) and the **Application Communication diagram** (shows which apps talk to which) are the two most universally useful — they are the "executive" and "integration" entry points respectively.

## ↔️ Catalogs vs. Matrices vs. Diagrams

| | [[Architecture Catalogs\|Catalog]] | [[Architecture Matrices\|Matrix]] | Diagram |
|---|---|---|---|
| Shape | List | Grid | Picture |
| Shows | Entities of one type | Relationships between two types | A visual subset, spatially |
| Strength | Completeness | Traceability | **Communication** |
| Source of truth? | ✅ Yes | Projection of catalog | Projection of catalog+matrix |
| Audience | Architects (governance) | Architects (analysis) | **All stakeholders** |

> The diagram is the only one of the three that a business sponsor can read — which is why it is the communication artifact, but also why it must never be the *only* artifact (it hides what the catalog reveals).

## ⚠️ Pitfalls & Anti-patterns

- **Treating the diagram as the source of truth:** the catalog/matrix is the data; the diagram is a projection. Editing only the diagram leaves the catalog stale and inconsistent.
- **Drawing without a notation:** freehand boxes-and-lines lose the type discipline. Pick a notation (C4, ArchiMate, UML) and apply it consistently. See [[Architectural Views]].
- **Trying to show everything:** a diagram with 200 boxes is unreadable and hides structure. Show the significant subset; elide the rest explicitly.
- **One diagram for all audiences:** a single diagram cannot serve executives, devs, and ops. Draw audience-specific diagrams (the [[Viewpoint]] principle).
- **No link back to the catalog:** a diagram that doesn't derive from a catalog will drift from reality. Keep the catalog as the source and regenerate.
- **Confusing TOGAF diagram types with notation:** TOGAF says *which* diagram (Business Footprint, Application Communication); C4/ArchiMate/UML say *how to draw it*. Don't expect TOGAF to define the symbols.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Content Framework]]
- **Sibling artifact types:** [[Architecture Catalogs]], [[Architecture Matrices]]
- **Notation choices:** [[C4 Model|C4]], [[Viewpoints and Perspectives|R&W]] (the course's notation), `ArchiMate`, `UML` (future notes)
- **Diagram-as-view:** [[Architectural Views]], [[Viewpoint]] (each TOGAF diagram is a view produced for an audience/concern)
- **Where diagrams are produced:** [[Architecture Development Method]] (throughout)
- **Prerequisites:** [[TOGAF]], [[Architecture Content Framework]].

## ❓ Active Recall Questions

> [!faq]- What is an architecture diagram in TOGAF, and what is its primary strength?
> A **visual picture** of a selected set of metamodel entities and relationships, drawn for a specific audience and concern. Its primary strength is **communication** — it is the only one of the three artifact types that a non-architect stakeholder can read.

> [!faq]- Does TOGAF define how to draw its diagrams?
> **No.** TOGAF is notation-agnostic — it names diagram *types* (Business Footprint, Application Communication, etc.) by purpose, but the *symbols* come from a notation you choose: C4, ArchiMate, or UML. This is why TOGAF + ArchiMate is a common pairing.

> [!faq]- Why must a diagram not be the source of truth?
> Because a diagram is a **projection** — a selected, readable subset. It necessarily elides entities for legibility. The [[Architecture Catalogs|catalog]] is the source of truth (completeness); the diagram communicates a view of it. Editing only the diagram leaves the catalog inconsistent and hides what was elided.

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Content Framework]]
- [[Architecture Catalogs]]
- [[Architecture Matrices]]
- [[Architectural Views]]
- [[Viewpoint]]
- [[Architecture Development Method]]
