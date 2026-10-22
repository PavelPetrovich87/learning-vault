---
status: archived
tags: [c4-model, architecture, togaf, domains, data-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Data Architecture

> [!abstract] Executive Summary (ELI5)
> **Data Architecture** is the TOGAF domain that describes **what information the business needs, how it is structured, where it flows, and who owns it**. It is a first-class BDAT domain (not folded into Application, unlike in [[4+1 View Model|4+1]]) because data often outlives the applications that manage it — "Customer," "Order," "Product" persist for decades while apps come and go. It is described in Phase D of the ADM, after [[Business Architecture|Business]] and alongside [[Application Architecture|Application]].

> [!note] Part of the TOGAF deep-dive
> One of the four BDAT domains in [[TOGAF Architecture Domains]]. It is the **Data** slice — the "information needed" layer.

---

## 🎯 The "Why"

- **Problem Solved:** Without a Data Architecture, every application invents its own view of "Customer" or "Order," producing duplicated, inconsistent definitions and integration spaghetti. A first-class Data Architecture fixes the **common definitions, ownership, and lineage** so data is a managed asset, not a byproduct.
- **Alternatives:** Folding data into Application Architecture (the [[4+1 View Model|4+1]] approach). Works for small systems; fails at enterprise scale where data must be shared and governed across apps.

## 🛠️ Core Concerns & Artifacts

| Concern | What it captures | Example TOGAF artifacts |
|---|---|---|
| **Data entities** | The things the business cares about (Customer, Order, Product) | Data Entity/Business Information catalog |
| **Structure / models** | How entities are structured (conceptual/logical/physical) | Entity-relationship models, data models |
| **Flows** | Where data moves between systems | Data Dissemination diagram, Data Entity ↔ Application matrix |
| **Ownership / stewardship** | Who owns and is accountable for each data entity | (in the catalog attributes) |
| **Lineage** | Where data originates and how it transforms | Data Migration diagram, lineage maps |
| **Quality / security constraints** | Integrity, privacy, retention | Data Security diagram, constraints in the catalog |

## ↔️ Where it sits in TOGAF

- **ADM phase:** Phase D (Information Systems Architectures) — alongside Application. See [[Architecture Development Method]].
- **Reference model:** partially supported by the [[III-RM]] (the integrated information infrastructure — how information flows). No standalone data reference model.
- **Metamodel entities:** Data Entity, Data Element, Information Flow, Logical/Application Component (data-flavored). See [[Architecture Metamodel]].
- **Maps to course viewpoints:** [[Information Viewpoint]] (how data is structured, stored, moved). See [[Architecture Domains]].

> The **Data Entity catalog** and the **Data Entity ↔ Application matrix** are the two most useful artifacts: the first enumerates what data exists; the second shows which apps own/use it — exposing duplication and ownership gaps.

## ⚠️ Pitfalls & Anti-patterns

- **Folding Data into Application:** data outlives apps; keep it a peer domain.
- **No ownership:** data entities with no accountable steward become nobody's responsibility and erode in quality.
- **Ignoring lineage:** not knowing where data originates makes change risky and compliance hard.
- **Only physical models:** jumping to table schemas without conceptual/logical models loses the shared business meaning.

## 🔗 Deep Connections

- **Parent Topic:** [[TOGAF Architecture Domains]], [[TOGAF]]
- **Sibling domains:** [[Business Architecture]], [[Application Architecture]], [[Technology Architecture]]
- **ADM:** [[Architecture Development Method]] (Phase D)
- **Reference model:** [[III-RM]]
- **Course mapping:** [[Information Viewpoint]], [[Architecture Domains]]
- **Prerequisites:** [[TOGAF]], [[TOGAF Architecture Domains]].

## ❓ Active Recall Questions

> [!faq]- Why is Data a first-class domain in TOGAF (not folded into Application)?
> Because data often outlives the applications that manage it — "Customer," "Order," "Product" persist for decades while apps come and go. Treating Data as a peer domain preserves common definitions, ownership, and lineage independently of whichever app currently touches it. (Contrast [[4+1 View Model|4+1]], which folds data into the Logical View.)

> [!faq]- What are the two most useful Data Architecture artifacts?
> The **Data Entity catalog** (enumerates what data exists, with ownership attributes) and the **Data Entity ↔ Application matrix** (shows which apps own/use each entity — exposing duplication and ownership gaps). See [[Architecture Catalogs]], [[Architecture Matrices]].

> [!faq]- Which reference model supports the Data domain?
> Partially the [[III-RM]] (integrated information infrastructure — how information flows). There is no standalone data reference model; Business has none, and Technology has the TRM. See [[Architecture Reference Models]].

---

## 📝 Related Notes

- [[TOGAF]]
- [[TOGAF Architecture Domains]]
- [[Business Architecture]]
- [[Application Architecture]]
- [[Technology Architecture]]
- [[Architecture Development Method]]
- [[III-RM]]
- [[Information Viewpoint]]
