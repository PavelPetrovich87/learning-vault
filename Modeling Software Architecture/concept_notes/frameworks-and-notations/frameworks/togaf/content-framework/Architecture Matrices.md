---
status: archived
tags: [c4-model, architecture, togaf, content-framework, matrices, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Architecture Matrices

> [!abstract] Executive Summary (ELI5)
> A **matrix** in TOGAF is a **grid** showing the **relationships between two types of entity** — e.g., which Applications support which Business Functions, or which Data Entities are owned by which Applications. It is the middle of the three [[Architecture Content Framework|Content Framework]] artifact types (catalogs, matrices, diagrams). Its job is **traceability**: to make the connections between domains visible and checkable, so you can answer "which capability does this application serve?" and spot gaps ("a capability with no supporting application").

> [!note] Part of the TOGAF deep-dive
> One of three artifact types in the [[Architecture Content Framework]]. Sibling shapes: [[Architecture Catalogs]] (lists), [[Architecture Diagrams]] (pictures).

---

## 🎯 The "Why"

- **Problem Solved:** Catalogs tell you what entities exist, but not how they relate. Without a matrix, the question "which applications realize the 'Process Payment' capability?" requires hunting through prose. A matrix makes such cross-domain relationships **explicit, visible, and checkable** — and surfaces gaps (a capability with no application, an application with no data).
- **Alternatives:** Describing relationships in prose or scattering them across diagrams. Uncheckable and incomplete — you cannot mechanically find an orphaned capability in prose.
- **Trade-offs:** Matrices are precise and checkable but hard to read at scale (a 100-application × 50-function grid is dense). They are the **traceability backbone**, but they communicate poorly to non-technical stakeholders — that is what [[Architecture Diagrams|diagrams]] do.

## 🛠️ Core Principles

1. **A matrix relates two entity types.** Rows = one type (e.g., Applications), columns = another (e.g., Business Functions). The cells capture the relationship (used by, realizes, supports).
2. **Matrices make traceability explicit.** They are the tool for the BDAT traceability chain — every Technology artifact should trace up through Applications and Data to a Business capability. See [[TOGAF Architecture Domains]].
3. **Matrices expose gaps.** An empty row (an application that supports no function) or an empty column (a function with no application) is an immediate, visible architectural finding.
4. **Matrices derive from catalogs.** The entities in a matrix come from [[Architecture Catalogs|catalogs]]. The catalog is the source of truth; the matrix is a relationship projection.

## 🏗️ Common TOGAF Matrices

| Matrix | Rows × Columns | What it traces |
|---|---|---|
| **Business Footprint matrix** | Goal/Objective × Business Capability | Why a capability exists (its business driver) |
| **Role × Location matrix** | Role × Location | Where roles operate (feeds Location concerns) |
| **Application × Function matrix** | Application Component × Business Function | Which apps support which functions |
| **Application Communication matrix** | Application × Application | Which apps talk to which (interfaces) |
| **Data Entity × Application Component matrix** | Data Entity × Application | Which apps own/use which data |
| **Gap Analysis matrix** | Baseline entity × Target entity | What exists vs. what is needed (Phase E) |
| **Implementation Factor × Project matrix** | Factor × Project | Constraints/risks per planned project |

> The **Application × Function matrix** and the **Gap Analysis matrix** are the two most used. The first traces apps to business value; the second drives the migration plan by exposing the Baseline-to-Target gap.

## ↔️ Catalogs vs. Matrices vs. Diagrams

| | [[Architecture Catalogs\|Catalog]] | Matrix | [[Architecture Diagrams\|Diagram]] |
|---|---|---|---|
| Shape | List | Grid | Picture |
| Shows | Entities of one type | Relationships between two types | A visual subset, spatially |
| Strength | Completeness | **Traceability** | Communication |
| Analogy | Phone book | Cross-reference index | Map |
| Gap detection | Missing entity | **Orphaned entity / missing relationship** | (implicit, visual) |

## ⚠️ Pitfalls & Anti-patterns

- **Building a matrix before the catalogs exist:** the entities must be enumerated first. A matrix over an incomplete catalog inherits the gaps.
- **Overloading cells:** putting prose or multiple relationships in one cell. Keep the cell to a single, typed relationship (e.g., "owns" vs. "uses") for checkability.
- **Reading a matrix to a non-technical audience:** matrices are precise but dense. For communication, project the matrix into a diagram.
- **Ignoring empty rows/columns:** an empty row or column is a finding (an orphaned app, an unsupported function), not noise. Investigate it.
- **Too-large matrices:** a grid too big to read has lost its value. Split by domain or by segment, or summarize.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Content Framework]]
- **Sibling artifact types:** [[Architecture Catalogs]], [[Architecture Diagrams]]
- **Where matrices are produced:** [[Architecture Development Method]] (Phases C–E; Gap Analysis in E)
- **What matrices trace:** [[TOGAF Architecture Domains]] (the BDAT traceability chain)
- **Prerequisites:** [[TOGAF]], [[Architecture Content Framework]].

## ❓ Active Recall Questions

> [!faq]- What is an architecture matrix, and what is its primary strength?
> A **grid** showing relationships between two entity types (rows × columns), e.g., Applications × Business Functions. Its primary strength is **traceability** — making cross-domain connections explicit and checkable, and exposing gaps (orphaned entities, missing relationships).

> [!faq]- What is the Gap Analysis matrix, and when is it produced?
> It compares **Baseline entities × Target entities** to expose what exists vs. what is needed. It is produced in **Phase E (Opportunities & Solutions)** of the ADM and drives the migration plan. See [[Architecture Development Method]].

> [!faq]- Why is an empty row or column in a matrix a finding, not noise?
> An empty row (e.g., an application that supports no business function) or an empty column (a function with no supporting application) signals an architectural problem — an orphaned asset or an unsupported capability. Investigating these is one of the main values of building the matrix.

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Content Framework]]
- [[Architecture Catalogs]]
- [[Architecture Diagrams]]
- [[Architecture Development Method]]
- [[TOGAF Architecture Domains]]
