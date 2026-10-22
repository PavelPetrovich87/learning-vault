---
status: archived
tags: [c4-model, architecture, notation, archimate, togaf, enterprise, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: ArchiMate

> [!abstract] Executive Summary (ELI5)
> **ArchiMate** (The Open Group, 2004) is the **enterprise-architecture notation** — the visual language most often paired with [[TOGAF]]. Where [[UML]] is software-focused, ArchiMate spans the **whole enterprise**: it has symbols for business actors and processes, applications and services, *and* technology nodes and networks — all in one coherent notation, organized into three layers. It also defines ~25 **viewpoints** (named diagram kinds for specific concerns/audiences). It is a **notation** (with strong metamodel coupling) that TOGAF uses to render its BDAT domains.

> [!warning] The #1 confusion — ArchiMate is a notation, not a framework
> Despite defining ~25 viewpoints, ArchiMate is **a notation**, not an [[Architecture Framework|architecture framework]]. It does not prescribe a process, nor does it tell you which views *must* exist for a given enterprise. Its viewpoints are *reusable diagram templates*; choosing which to produce is still a framework/team decision. The common pairing — ArchiMate + TOGAF — combines ArchiMate's notation with TOGAF's framework/method.

> [!note] Part of the notations side of this folder
> One of four standalone notations in the `notations/` subfolder (parent: [[Architecture Notation]]). Analyzed through the four notation axes.

---

## 🎯 The "Why"

- **Problem Solved:** Enterprise architects need to draw business, application, *and* technology on coherent diagrams that trace across layers — and [[UML]] (software-only) can't, while ad-hoc drawings don't. ArchiMate provides **one notation spanning all enterprise layers**, so a Business Actor can be shown realizing a Business Service, realized by an Application Component, deployed on a Technology Node — all in one connected model.
- **Alternatives:** **UML** (software-only, no first-class business layer), **BPMN** (business-process-only), **C4** (software-system only). For enterprise scope, ArchiMate is the standard choice; it pairs naturally with TOGAF.
- **Trade-offs:** ArchiMate is **comprehensive but complex** — its 3-layer × aspect matrix and many element types have a steep learning curve. Its precision and layer-spanning traceability are the payoff; for a single software system, UML or C4 is usually simpler.

## 🧭 ArchiMate Through the Four Notation Axes

### 1. Symbol set — *three layers × aspects, with explicit "active/behavior/passive"*

ArchiMate organizes its elements into a **layer × aspect matrix**:

| Layer | Active structure (who does) | Behavior (what they do) | Passive structure (what they use) |
|---|---|---|---|
| **Business** | Business Actor, Business Role, Business Collaboration | Business Process, Business Function, Business Service, Business Event | Business Object, Representation |
| **Application** | Application Component, Application Collaboration, Interface | Application Function, Application Service, Application Process | Data Object |
| **Technology** | Node, Device, System Software, Communication Path, Network | Technology Function, Technology Service | Artifact |

> Each element has a **precise shape** (rectangle for active structure, rounded for behavior, etc.) and a **precise meaning**. The three-layer structure is what lets ArchiMate trace a technology node up through an application to a business actor.

### 2. Diagram types — *~25 named viewpoints*

ArchiMate defines about 25 **viewpoints** — reusable diagram templates, each aimed at a specific stakeholder/concern:

| Viewpoint category | Examples |
|---|---|
| **Introductory / high-level** | Introductory, Landscape Map |
| **Business** | Business Function, Business Process, Business Cooperation, Product |
| **Application** | Application Behavior, Application Cooperation, Application Structure, Application Usage |
| **Technology** | Infrastructure, Implementation & Deployment, Technology |
| **Strategic / Motivation** | Strategy, Capability, Value Stream, Outcome Realization |
| **Cross-layer** | Stakeholder, Goal Realization, Motivation |

> Unlike UML's 14 *diagram kinds*, ArchiMate's viewpoints are stakeholder-driven templates — each names its audience and concerns. This is why ArchiMate pairs so well with the concern-driven philosophy of [[Viewpoints and Perspectives|R&W]].

### 3. Metamodel coupling — *strong (model-first)*

ArchiMate has a **formal metamodel** and is designed to be **model-first**: you build a typed model, and views are projections of that model.

| Coupling | Reality |
|---|---|
| **Strong by design** | Elements are instances of typed layer×aspect classes; relationships follow defined rules (serving, realization, assignment, etc.) |
| **Tools enforce it** | Archi modelers (Archi, BiNoM, vendor tools) check the model against the metamodel |

> ArchiMate is the strongest-coupled notation in this folder — the picture is genuinely generated from a typed, checkable model. This embodies the [[Architecture Metamodel]] idea most fully.

### 4. Tooling & format — *specialized, with an open exchange format*

- **Authoring tools:** **Archi** (free, open-source, the de facto standard), plus vendor tools (BiArchitect, Avola, MEGA, BiZZdesign).
- **Interchange format:** **ArchiMate Model Exchange File Format** (The Open Group standard) — exchanges models between tools.
- **Pairs with TOGAF:** ArchiMate models often live in a TOGAF Architecture Repository.

## ↔️ ArchiMate vs. UML

| | ArchiMate | [[UML]] |
|---|---|---|
| Scope | **Enterprise** (business + app + technology) | Software (some deployment) |
| Layers | 3 (Business/Application/Technology) | No layered model |
| Diagram kinds | ~25 viewpoints (stakeholder-driven) | 14 diagram kinds (concern-driven) |
| First-class business | **Yes** (Business Actor, Process, Service) | No |
| Metamodel coupling | Strong (model-first) | Medium (often drawing-only) |
| Framework pairing | TOGAF | 4+1 |

> UML is deeper on software internals; ArchiMate is broader across enterprise layers. For an enterprise architecture, ArchiMate; for software design detail, UML.

## ↔️ Why ArchiMate + TOGAF is the canonical pairing

[[TOGAF]] is **notation-agnostic** (it defines domains, metamodel entities, and a process, but not how to draw). ArchiMate is **framework-agnostic** (it defines symbols and viewpoints, but not which to produce or how to run an architecture practice). They fit together perfectly:

- TOGAF's **BDAT domains** ↔ ArchiMate's **three layers** (Business, Application, Technology — Data is folded into Application/Data Object).
- TOGAF's **metamodel entities** (Capability, Application Component, Node) ↔ ArchiMate's **element types**.
- TOGAF's **Architecture Repository** stores ArchiMate models.

> See [[TOGAF]] → "Views" (TOGAF defers to a notation for drawing; ArchiMate is the standard choice).

## ⚠️ Pitfalls & Anti-patterns

- **Treating ArchiMate as a framework:** its viewpoints are reusable diagram templates, not a mandate of which views must exist. Pair with TOGAF or your own framework.
- **Drawing all 25 viewpoints:** most enterprises use 5–10 viewpoints that match their concerns. Pick by stakeholder/concern.
- **Forgetting the layered traceability:** the whole value is Business→Application→Technology tracing. Drawing one layer in isolation loses it.
- **Confusing ArchiMate layers with TOGAF domains:** they overlap (Business/App/Tech) but ArchiMate folds Data into Application (as Data Object); TOGAF keeps Data as a separate BDAT domain. Note the difference.
- **Using ArchiMate for software-internal detail:** it's coarse-grained. Use [[UML]] or [[C4 Model|C4]] Component diagrams for code-level detail.
- **Drawing-only ArchiMate:** if you're sketching ArchiMate without a model, you've lost its main benefit (strong metamodel coupling). Use a model-first tool like Archi.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Notation]] (the four notation axes), [[Architecture Framework]]
- **Concrete notation siblings:** [[UML]], [[SysML]], [[BPMN]]
- **Framework it pairs with:** [[TOGAF]] (the canonical enterprise pairing)
- **Contrast:** [[UML]] (software-deep), [[C4 Model]] (software-system, simpler), [[BPMN]] (business-process-only)
- **Related Concepts:** [[Architecture Metamodel]] (ArchiMate is the strongest-coupled notation), [[TOGAF Architecture Domains]] (BDAT ↔ ArchiMate layers), [[Architectural Views]] (ArchiMate viewpoints are views), [[Viewpoint]]
- **Prerequisites:** [[Architecture Notation]], [[Architecture Framework]].

## ❓ Active Recall Questions

> [!faq]- What is ArchiMate, and why is calling it a framework a mistake?
> ArchiMate is an **enterprise-architecture notation** — a visual language with elements for Business, Application, and Technology layers, ~25 named viewpoints, and a strong metamodel. It is a notation, not a framework: it doesn't prescribe a process or mandate which views must exist. Pair it with a framework — most commonly [[TOGAF]].

> [!faq]- What are the four notation axes' values for ArchiMate?
> **Symbol set:** three-layer × aspect matrix (Business/Application/Technology × Active structure/Behavior/Passive structure). **Diagram types:** ~25 stakeholder-driven viewpoints. **Metamodel coupling:** strong — model-first; tools check models against the metamodel. **Tooling/format:** specialized tools (Archi is the de facto free one) with the ArchiMate Model Exchange File Format.

> [!faq]- Why is ArchiMate + TOGAF the canonical pairing?
> Because [[TOGAF]] is notation-agnostic (it defines domains/metamodel/process but not how to draw), and ArchiMate is framework-agnostic (it defines symbols/viewpoints but not which to produce). TOGAF's BDAT domains map onto ArchiMate's three layers, its metamodel entities map onto ArchiMate's element types, and its Architecture Repository stores ArchiMate models.

> [!faq]- How do ArchiMate layers relate to TOGAF BDAT domains?
> They overlap but are not identical. ArchiMate has Business, Application, Technology layers; TOGAF has Business, Data, Application, Technology. The key difference: ArchiMate folds **Data** into the Application layer (as Data Object), while TOGAF keeps **Data** as a first-class separate domain. See [[TOGAF Architecture Domains]].

> [!faq]- Is ArchiMate or UML better for an enterprise architecture?
> **ArchiMate** — because it spans business + application + technology in one coherent notation, with first-class business elements (Business Actor, Process, Service) that UML lacks. UML is deeper on software internals but has no business layer. For an enterprise, ArchiMate; for software design detail, UML.

---

## 📝 Related Notes

- [[Architecture Notation]]
- [[Architecture Framework]]
- [[UML]]
- [[SysML]]
- [[BPMN]]
- [[TOGAF]]
- [[TOGAF Architecture Domains]]
- [[Architecture Metamodel]]
- [[Architectural Views]]
- [[Viewpoint]]
