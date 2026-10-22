---
status: archived
tags: [c4-model, architecture, togaf, content-framework, catalogs, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Architecture Catalogs

> [!abstract] Executive Summary (ELI5)
> A **catalog** in TOGAF is a **structured list** of entities of a single type, with their key attributes. It is the simplest of the three [[Architecture Content Framework|Content Framework]] artifact types (catalogs, matrices, diagrams). Its job is **completeness** — to make sure you have enumerated *all* the things of a kind (all applications, all data entities, all capabilities) so none is forgotten and all are governed. Think of it as the **inventory** of the architecture: a phone book, not a map.

> [!note] Part of the TOGAF deep-dive
> One of three artifact types in the [[Architecture Content Framework]]. Sibling shapes: [[Architecture Matrices]] (relationships), [[Architecture Diagrams]] (pictures).

---

## 🎯 The "Why"

- **Problem Solved:** Without catalogs, you don't know what you have. "Which applications does the enterprise run?" becomes an unanswerable question, and you can't govern, rationalize, or deduplicate what you haven't enumerated. A catalog forces systematic enumeration.
- **Alternatives:** Keeping the inventory in scattered PowerPoint slides and spreadsheets. Non-canonical, non-comparable, and always stale.
- **Trade-offs:** Catalogs are thorough but unvisual. They are the *backbone* of completeness and governance, but they don't communicate relationships or big-picture structure — that is what [[Architecture Matrices|matrices]] and [[Architecture Diagrams|diagrams]] are for.

## 🛠️ Core Principles

1. **A catalog lists entities of one type.** One catalog per entity type — an Application Portfolio catalog, a Data Entity catalog, a Capability catalog — not a mix. This keeps each catalog focused and queryable.
2. **Each entry has key attributes.** A catalog is not just names; each row carries the attributes that matter for governance (owner, status, lifecycle, technology, criticality).
3. **Catalogs ensure completeness.** Their primary value is that *nothing is missed* — if an application exists, it should be in the Application Portfolio catalog. This is the basis for rationalization and gap analysis.
4. **Catalogs are the data source for matrices and diagrams.** The entities in a matrix or diagram come *from* a catalog. The catalog is the source of truth; the matrix/diagram is a projection.

## 🏗️ Common TOGAF Catalogs (by domain)

| Domain | Common catalogs | What they enumerate |
|---|---|---|
| **Business** | Organization/Actor catalog, Driver/Goal/Objective catalog, Role catalog, Business Service/Function catalog, Value Stream catalog, Contract/Measure catalog | Who does what, why, and how it's measured |
| **Data** | Data Entity/Business Information catalog, Data Dissemination (who sees what data) | What information exists and where it flows |
| **Application** | Application Portfolio catalog, Interface catalog, Function (application) catalog | Which apps exist, what they offer, how they connect |
| **Technology** | Technology Standards catalog, Technology Component catalog | Which tech is approved/in use |
| **Cross** | Consolidated Reusable Building Blocks catalog, Solution Building Block catalogue | Reusable assets and chosen solutions |

> The **Application Portfolio catalog** and the **Data Entity catalog** are the two most universally useful — they answer "what do we have?" for the Application and Data domains respectively. Most enterprises start there.

## ↔️ Catalogs vs. Matrices vs. Diagrams

| | Catalog | [[Architecture Matrices\|Matrix]] | [[Architecture Diagrams\|Diagram]] |
|---|---|---|---|
| Shape | List | Grid | Picture |
| Shows | Entities of one type | Relationships between two types | A visual subset, spatially |
| Strength | Completeness | Traceability | Communication |
| Analogy | Phone book | Cross-reference index | Map |

> A catalog tells you *what exists*; a matrix tells you *how they relate*; a diagram tells you *what it looks like*. You need all three for a complete architecture description.

## ⚠️ Pitfalls & Anti-patterns

- **Cataloging without attributes:** a list of names with no owner/status/technology is an index, not a governance-ready catalog. Capture the attributes that matter.
- **Mixing entity types in one catalog:** "the enterprise catalog" with apps, data entities, and capabilities mixed together. Keep one type per catalog for queryability.
- **Letting catalogs go stale:** an Application Portfolio catalog that hasn't been updated in two years is fiction. Catalogs need a maintenance owner and a refresh cadence.
- **Cataloging everything down to the class level:** over-granular catalogs become unmaintainable. Catalog architecturally significant entities, not every implementation detail.
- **Treating the catalog as the final deliverable:** the catalog is the *data source*; its value is realized when matrices and diagrams project it for stakeholders.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Content Framework]]
- **Sibling artifact types:** [[Architecture Matrices]], [[Architecture Diagrams]]
- **What populates catalogs:** [[Architecture Metamodel]] (the typed entities)
- **Where catalogs are produced:** [[Architecture Development Method]] (Phases B–E)
- **Prerequisites:** [[TOGAF]], [[Architecture Content Framework]].

## ❓ Active Recall Questions

> [!faq]- What is an architecture catalog, and what is its primary strength?
> A **structured list** of entities of a single type, with key attributes (e.g., an Application Portfolio catalog, a Data Entity catalog). Its primary strength is **completeness** — ensuring every architecturally significant entity of a kind is enumerated so none is missed and all can be governed.

> [!faq]- How does a catalog differ from a matrix and a diagram?
> A **catalog** lists entities of one type (strength: completeness). A **matrix** shows relationships between two types as a grid (strength: traceability). A **diagram** is a visual picture (strength: communication). The catalog is the data source; matrices and diagrams project it.

> [!faq]- Name two universally useful TOGAF catalogs.
> The **Application Portfolio catalog** (what applications exist) and the **Data Entity catalog** (what data entities exist). Most enterprises start with these because they answer the foundational "what do we have?" question for the Application and Data domains.

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Content Framework]]
- [[Architecture Metamodel]]
- [[Architecture Matrices]]
- [[Architecture Diagrams]]
- [[Architecture Development Method]]
