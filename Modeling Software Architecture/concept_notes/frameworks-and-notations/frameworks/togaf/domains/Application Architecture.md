---
status: archived
tags: [c4-model, architecture, togaf, domains, application-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Application Architecture

> [!abstract] Executive Summary (ELI5)
> **Application Architecture** is the TOGAF domain that describes **which applications deliver the business capabilities and manage the data** — the software/services layer. It catalogs the application portfolio, the services/interfaces each app offers, and the dependencies between apps. It is the **bridge domain**: each application should trace *up* to a [[Business Architecture|business capability]] and *down* to the [[Data Architecture|data entities]] it owns, running on [[Technology Architecture|technology]]. It is described in Phase D of the ADM.

> [!note] Part of the TOGAF deep-dive
> One of the four BDAT domains in [[TOGAF Architecture Domains]]. It is the **Application** slice — the "software that processes data and delivers capabilities" layer.

---

## 🎯 The "Why"

- **Problem Solved:** In an enterprise, dozens or hundreds of applications overlap, duplicate, and depend on each other in opaque ways. Application Architecture brings order: a canonical portfolio, clear interfaces, and dependency maps, so you can rationalize (retire duplicates), plan change, and ensure each app has a business reason.
- **Alternatives:** Letting the application landscape grow organically with no inventory. Result: duplication, unknown dependencies, and apps nobody can justify retiring.

## 🛠️ Core Concerns & Artifacts

| Concern | What it captures | Example TOGAF artifacts |
|---|---|---|
| **Application portfolio** | Which applications exist | Application Portfolio catalog |
| **Services / interfaces** | What each app offers and how it connects | Interface catalog, Application Service/Function catalog |
| **Dependencies / communication** | Which apps talk to which | Application Communication matrix/diagram |
| **Capability support** | Which apps support which business capabilities | Application × Function matrix |
| **Data ownership** | Which apps own/use which data | Data Entity ↔ Application matrix |
| **Migration** | How apps change over time | Application Migration diagram |

> The **Application Portfolio catalog** and the **Application Communication diagram/matrix** are the most useful: the first enumerates what exists; the second shows how they integrate. Together they expose duplication and integration risk.

## ↔️ Where it sits in TOGAF

- **ADM phase:** Phase D (Information Systems Architectures) — alongside Data. See [[Architecture Development Method]].
- **Reference model:** the [[III-RM]] (integrated information infrastructure — application/data services supporting business via boundaryless information flow).
- **Metamodel entities:** Application Component, Application Service, Interface, Function (application). See [[Architecture Metamodel]].
- **Maps to course viewpoints:** [[Container Viewpoint]], [[Component Viewpoint]], [[Development Viewpoint]]. In C4 terms, the Application domain is where C4 lives — Containers and Components describe application structure. See [[Architecture Domains]].

> This is why the course pairs C4 with R&W: **C4 is essentially a notation for the Application domain** (containers/components), while R&W supplies the viewpoints for the domains C4 doesn't cover. See [[C4 Model]].

## ⚠️ Pitfalls & Anti-patterns

- **No portfolio inventory:** can't rationalize or plan change without knowing what exists.
- **Apps with no business traceability:** an application that supports no capability is a candidate for retirement — but you can't find it without the Application × Function matrix.
- **Unknown dependencies:** changing an app breaks others because the Application Communication matrix wasn't maintained.
- **Conflating Application with Technology:** an app (Application domain) vs. the platform it runs on (Technology domain). Keep them separate.

## 🔗 Deep Connections

- **Parent Topic:** [[TOGAF Architecture Domains]], [[TOGAF]]
- **Sibling domains:** [[Business Architecture]], [[Data Architecture]], [[Technology Architecture]]
- **ADM:** [[Architecture Development Method]] (Phase D)
- **Reference model:** [[III-RM]]
- **Course mapping (C4 lives here):** [[Container Viewpoint]], [[Component Viewpoint]], [[Development Viewpoint]], [[C4 Model]], [[Architecture Domains]]
- **Prerequisites:** [[TOGAF]], [[TOGAF Architecture Domains]].

## ❓ Active Recall Questions

> [!faq]- What is Application Architecture, and why is it the "bridge" domain?
> It describes **which applications deliver business capabilities and manage data**. It is the bridge because each application should trace *up* to a business capability and *down* to the data entities it owns, running on technology. It connects the business layer to the technology layer.

> [!faq]- How does the course's C4 model relate to the Application domain?
> C4 is essentially **a notation for the Application domain** — its Container and Component levels describe application structure. This is why the course pairs C4 (Application notation) with R&W viewpoints (which cover the other domains C4 omits). See [[C4 Model]].

> [!faq]- Which reference model supports the Application domain?
> The [[III-RM]] (integrated information infrastructure) — it provides a taxonomy of the application/data services that support the business through boundaryless information flow. The [[Technical Reference Model|TRM]] covers the Technology layer below. See [[Architecture Reference Models]].

---

## 📝 Related Notes

- [[TOGAF]]
- [[TOGAF Architecture Domains]]
- [[Business Architecture]]
- [[Data Architecture]]
- [[Technology Architecture]]
- [[Architecture Development Method]]
- [[III-RM]]
- [[Container Viewpoint]]
- [[Component Viewpoint]]
- [[C4 Model]]
