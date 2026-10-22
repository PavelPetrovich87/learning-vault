---
status: archived
tags: [c4-model, architecture, togaf, domains, business-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Business Architecture

> [!abstract] Executive Summary (ELI5)
> **Business Architecture** is the topmost TOGAF domain — it describes **what the organization does, for whom, and why**: its business capabilities, value streams, processes, organization, and business objects. It is the foundation that justifies everything below it: every [[Data Architecture|data entity]], [[Application Architecture|application]], and [[Technology Architecture|technology]] component should trace up to a business capability it serves. It is described first in the ADM (Phase C) precisely because the lower layers must be realized by it.

> [!note] Part of the TOGAF deep-dive
> One of the four BDAT domains in [[TOGAF Architecture Domains]]. It is the **Business** slice — the "what the org does" layer.

---

## 🎯 The "Why"

- **Problem Solved:** Without a Business Architecture, IT decisions float free of business value — you cannot answer "which capability does this application serve?" or justify its cost. Business Architecture makes the business-driver → IT chain explicit, so every lower-domain artifact has a reason to exist.
- **Alternatives:** Letting business context live only in people's heads or in strategy slides disconnected from IT. Result: IT that doesn't serve the strategy.

## 🛠️ Core Concerns & Artifacts

| Concern | What it captures | Example TOGAF artifacts |
|---|---|---|
| **Capabilities** | What the business *can do* (stable, not org-chart-bound) | Business Capability/Function catalog, Business Footprint matrix/diagram |
| **Value streams / processes** | How value is produced end-to-end | Value Stream catalog, process models, Functional Decomposition diagram |
| **Organization / actors / roles** | Who does what | Organization/Actor catalog, Role catalog |
| **Drivers / goals / objectives** | Why the business does this | Driver/Goal/Objective catalog |
| **Business objects / services** | What information the business cares about; services offered | Business Service/Function catalog, Contract/Measure catalog |

> The **Business Capability** is the anchor. Capabilities are relatively stable (a bank has had "Process Payment" for centuries); org structures and apps change frequently. Modeling capabilities first gives a stable target to trace apps and data to.

## ↔️ Where it sits in TOGAF

- **ADM phase:** Phase C (Business Architecture) — described *first*, before Data/Application/Technology. See [[Architecture Development Method]].
- **Reference model:** **none** — Business Architecture is organization-specific, so TOGAF provides no generic business reference model (unlike the [[Technical Reference Model|TRM]]/[[III-RM]] for the lower layers).
- **Metamodel entities:** Actor, Role, Organization Unit, Business Capability, Function, Process, Business Service, Business Object, Driver, Goal, Objective. See [[Architecture Metamodel]].
- **Maps to course viewpoints:** [[Context Viewpoint]] (actors, purpose) and [[Functional Viewpoint]] (capabilities). See [[Architecture Domains]].

## ⚠️ Pitfalls & Anti-patterns

- **Skipping Business Architecture (going straight to apps/tech):** the most common enterprise mistake. IT then has no business justification. Describe Business first.
- **Modeling org chart instead of capabilities:** org structures change; capabilities are stable. Anchor on capabilities, not reporting lines.
- **No traceability down:** capabilities with no apps/data realizing them, or apps with no capability they serve. The [[Architecture Matrices|matrix]] exposes these gaps.

## 🔗 Deep Connections

- **Parent Topic:** [[TOGAF Architecture Domains]], [[TOGAF]]
- **Sibling domains:** [[Data Architecture]], [[Application Architecture]], [[Technology Architecture]]
- **ADM:** [[Architecture Development Method]] (Phase C)
- **Course mapping:** [[Functional Viewpoint]], [[Context Viewpoint]], [[Architecture Domains]]
- **Prerequisites:** [[TOGAF]], [[TOGAF Architecture Domains]].

## ❓ Active Recall Questions

> [!faq]- What is Business Architecture, and why is it described first in the ADM?
> It describes **what the organization does** — capabilities, value streams, processes, org, business objects. It is described first (Phase C) because every lower-domain artifact (data entity, application, technology) should trace up to a business capability; you cannot justify the lower layers without the business foundation.

> [!faq]- Why anchor on capabilities rather than the org chart?
> Capabilities are relatively stable ("Process Payment" persists for decades); org structures change frequently. Anchoring on capabilities gives a stable target to trace apps and data to, while reorganizations don't invalidate the architecture.

> [!faq]- Does Business Architecture have a TOGAF reference model?
> No. The [[Technical Reference Model|TRM]] and [[III-RM]] cover Technology and Application/Data; Business Architecture is organization-specific, so TOGAF provides no generic business reference model.

---

## 📝 Related Notes

- [[TOGAF]]
- [[TOGAF Architecture Domains]]
- [[Data Architecture]]
- [[Application Architecture]]
- [[Technology Architecture]]
- [[Architecture Development Method]]
- [[Functional Viewpoint]]
- [[Context Viewpoint]]
