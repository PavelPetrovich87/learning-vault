---
status: archived
tags: [c4-model, architecture, 4+1, logical-view, kruchten, uml, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Logical View (4+1)

> [!abstract] Executive Summary (ELI5)
> The **Logical View** is the first of the four views in the [[4+1 View Model|4+1 View Model]]. It describes the system's **functional structure** — the key abstractions (classes, objects, packages), their responsibilities, and how they collaborate to deliver the system's functionality. It is the view the **end-user and analyst** care about: "what does the system do, in terms of the problem domain?" It is the closest 4+1 comes to a "domain model," and it is drawn primarily with **UML class/object diagrams**.

> [!warning] The #1 confusion — Logical View ≠ only one course viewpoint
> The Logical View bundles together what [[Viewpoints and Perspectives|R&W]] keeps as **two separate viewpoints**: the [[Functional Viewpoint]] (capabilities) and the [[Information Viewpoint]] (data). In 4+1, both the functional decomposition *and* the persistent data model live in the Logical View's class diagrams. This is the key difference — R&W later separated data out.

> [!note] Part of the 4+1 deep-dive
> One of four views in the [[4+1 View Model|4+1]] framework. This note focuses on what's *distinctive about 4+1's* Logical View; the general concept of "functional structure" is covered in the cross-linked course viewpoints.

---

## 🎯 The "Why"

- **Problem Solved:** Before you can decide how the system runs concurrently ([[Process View (4+1)|Process]]) or where it deploys ([[Physical View (4+1)|Physical]]), you must model **what the system *is* functionally** — the domain objects and their collaborations. The Logical View provides that functional backbone, written in the vocabulary of the problem domain so end-users and analysts can read it.
- **Alternatives:** Jumping straight to a Process or Physical view without a functional model. The system then has no shared "what does this do?" picture, and stakeholders can't validate the design against their needs.
- **Trade-offs:** The Logical View is the most abstract of the four — it deliberately ignores concurrency, distribution, and code packaging. Its power is that abstraction; its limit is that it answers *only* the functional question.

## 🛠️ Core Characteristics

| Aspect | Logical View |
|---|---|
| **Concern** | Functional structure — what the system does, in domain terms |
| **Primary audience** | End-user, analyst |
| **UML notation** | Class diagram, object diagram, interaction/sequence diagrams |
| **Key elements** | Classes, objects, packages, interfaces, key abstractions |
| **BDAT domain** | Business (capabilities) + Data (entities), bundled |
| **Ignored** | Concurrency (→ Process), code packaging (→ Development), deployment (→ Physical) |

> The Logical View's class model typically includes **both** the behavioral classes (services, controllers) *and* the persistent data entities (Customer, Order). In R&W terms this is Functional + Information fused into one view.

## ↔️ Mapping to Rozanski & Woods viewpoints

This is the most important cross-reference: 4+1's Logical View spans **two** R&W viewpoints:

| Logical View contains | R&W viewpoint | Note |
|---|---|---|
| Functional decomposition (capabilities, services, collaborations) | [[Functional Viewpoint]] | direct equivalent |
| Persistent data entities and their relationships | [[Information Viewpoint]] | 4+1 *folds* data into Logical; R&W separates it |

> See [[4+1 View Model]] → "Views" for the full 4+1↔R&W↔C4 mapping. The takeaway: if you've modeled the Functional and Information viewpoints, you've essentially produced the 4+1 Logical View — but kept the two concerns separable, which 4+1 does not.

## 🧱 Designing a Good Logical View

Kruchten emphasizes a few patterns for keeping the Logical View readable:

- **Object-oriented decomposition** — model the problem domain as collaborating objects with clear responsibilities, not as a procedural flow.
- **Use of design patterns** — the Logical View is where patterns (Strategy, Observer, Façade…) clarify structure.
- **Keep it at the *significant* abstraction level** — model the architecturally meaningful classes, not every implementation class. (Compare [[Component Viewpoint]] → "Model only significant components.")
- **Separate interface from implementation** — what a class offers vs. how it's realized.

## ⚠️ Pitfalls & Anti-patterns

- **Modeling every class:** defeats the purpose. The Logical View should show architecturally significant abstractions, not be a code map.
- **Mixing in concurrency or deployment:** those belong to [[Process View (4+1)|Process]] and [[Physical View (4+1)|Physical]]. The Logical View is concurrency- and location-agnostic.
- **Ignoring the data model:** because 4+1 folds data here, teams sometimes under-model it. R&W's [[Information Viewpoint]] treats data as first-class — a useful corrective.
- **Procedural rather than object-oriented decomposition:** a flowchart of functions loses the domain-object clarity that is the Logical View's value.
- **Conflating with the Development View:** the Logical View shows *domain classes*; the [[Development View (4+1)|Development View]] shows *code modules/packages*. Same class can appear in both, but the concern differs.

## 🔗 Deep Connections

- **Parent Topic:** [[4+1 View Model]]
- **Sibling 4+1 views:** [[Process View (4+1)]], [[Development View (4+1)]], [[Physical View (4+1)]], [[Scenarios (4+1)]]
- **R&W equivalents:** [[Functional Viewpoint]], [[Information Viewpoint]] (the Logical View bundles both)
- **Related Concepts:** [[Viewpoint]] (each 4+1 view is a viewpoint in the general sense), [[Container Viewpoint]]/[[Component Viewpoint]] (C4's structural zoom — related but C4 separates layers 4+1 merges), [[Architecture Domains]] (Logical covers Business + Data)
- **Prerequisites:** [[4+1 View Model]], [[Viewpoint]].

## ❓ Active Recall Questions

> [!faq]- What does the Logical View show, and who is it for?
> The system's **functional structure** — the key domain abstractions (classes, objects, packages) and how they collaborate to deliver functionality. It is for the **end-user and analyst**, drawn primarily with UML class/object diagrams. It is the "what does the system do, in domain terms?" view.

> [!faq]- Which R&W viewpoints does the Logical View bundle together?
> It bundles the [[Functional Viewpoint]] (capabilities/services) and the [[Information Viewpoint]] (data entities) into one view. R&W keeps these separate; 4+1 folds data into the Logical View's class model. This is a key difference between the frameworks.

> [!faq]- Why is folding data into the Logical View a limitation?
> Because data often outlives the behavioral classes that manage it, and data-intensive systems need data modeled as a first-class concern (ownership, lineage, structure). 4+1 has no standalone data view; R&W later separated it as the [[Information Viewpoint]], and TOGAF elevates Data to a first-class BDAT domain. See [[Architecture Domains]].

> [!faq]- How does the Logical View differ from the Development View?
> The **Logical View** shows *domain classes/objects* (functional structure, for end-users/analysts). The **[[Development View (4+1)|Development View]]** shows *code modules/packages* (static code organization, for programmers). The same class can appear in both, but the concern is different — functional abstraction vs. code packaging.

---

## 📝 Related Notes

- [[4+1 View Model]]
- [[Process View (4+1)]]
- [[Development View (4+1)]]
- [[Physical View (4+1)]]
- [[Scenarios (4+1)]]
- [[Functional Viewpoint]]
- [[Information Viewpoint]]
- [[Viewpoint]]
- [[Architecture Domains]]
