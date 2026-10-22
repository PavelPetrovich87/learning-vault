---
status: archived
tags: [c4-model, architecture, notation, sysml, omg, systems-engineering, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: SysML — Systems Modeling Language

> [!abstract] Executive Summary (ELI5)
> **SysML** (Systems Modeling Language, OMG, 2007) is a **systems-engineering notation** — a UML profile (a customized subset + extensions of [[UML]]) designed for modeling *systems*, not just software. Where UML assumes the system is software, SysML models **hardware, software, people, facilities, and data** together as one system-of-systems. It adds things UML lacks (requirements diagrams, parametrics for engineering equations, block definition) and trims things irrelevant to systems (no object diagrams). It is the notation of choice for aerospace, automotive, defense, and other systems-engineering domains.

> [!warning] The #1 confusion — SysML is UML-derived but not "UML for systems"
> SysML is **not** simply UML applied to systems. It is a **deliberate redesign**: it *reuses part* of UML (around half the diagram kinds), *removes* software-specific bits (object diagrams, communication diagrams), and *adds* systems-engineering essentials (requirements, parametrics, block definition, allocations). The relationship is "SysML is based on a subset of UML, extended for systems engineering."

> [!note] Part of the notations side of this folder
> One of four standalone notations in the `notations/` subfolder (parent: [[Architecture Notation]]). Analyzed through the four notation axes.

---

## 🎯 The "Why"

- **Problem Solved:** Systems engineers build things that mix hardware, software, mechanics, electronics, and people (a spacecraft, a car, a medical device). [[UML]] assumes the system is software and cannot express "the engine must weigh ≤ 200 kg" (a requirement) or "thrust = mass × acceleration" (a parametric constraint). SysML extends the model to cover **the whole engineered system**, with first-class requirements, blocks (any system part), and parametrics (engineering analysis).
- **Alternatives:** **UML** (software-only — can't model the hardware/requirements side), bespoke engineering drawings (non-standardized). For pure software, UML; for systems, SysML.
- **Trade-offs:** SysML is **heavy and complex**, aimed at large, mission-critical systems engineering. For a web service it is gross overkill. The payoff is precise, multi-discipline modeling that catches integration issues early (the cost of a hardware/software mismatch in aerospace is enormous).

## 🧭 SysML Through the Four Notation Axes

### 1. Symbol set — *UML-derived + systems extensions*

SysML's central element is the **Block** — a generic system part (hardware, software, person, facility, anything) — which generalizes UML's Class. Key element types:

| Category | SysML elements |
|---|---|
| **Structure** | Block, Internal Block Diagram (parts/connections inside a block), Port, Connector |
| **Behavior** | Activity, State Machine, Sequence (reused from UML) |
| **Requirements** (new vs UML) | Requirement, with text, ID, and verify relationships — **first-class** |
| **Parametrics** (new vs UML) | Constraint blocks — engineering equations binding system properties |
| **Allocation** | Which software/hardware/person realizes which requirement or activity |

> The **Requirement** and **Parametric** diagrams are the headline additions over UML. Requirements diagrams let you tie system properties to verifiable requirements; parametrics let you bind physical/engineering constraints (mass, thrust, power) into the model.

### 2. Diagram types — *9 kinds (half of UML's 14, plus systems additions)*

SysML defines **9 diagram kinds** — a trimmed and extended version of UML's 14:

| Removed from UML (software-specific) | Retained from UML | Added for systems |
|---|---|---|
| Object, Communication, Deployment, Component, Composite Structure, Timing, Interaction Overview | Class (→ Block Definition), Sequence, State Machine, Activity, Use Case, Package | **Requirement**, **Parametric**, Internal Block |

> Note: SysML has **no Deployment diagram** — because SysML's Block/Internal Block diagrams already represent physical layout generically. This is a notable difference from [[UML]].

### 3. Metamodel coupling — *strong (model-first, like UML)*

SysML has a **formal metamodel** (built on UML's, via the UML profile mechanism) and is used **model-first** in serious tools.

| Coupling | Reality |
|---|---|
| **Strong by design** | Blocks, requirements, parametrics are typed; relationships follow rules |
| **Tools enforce it** | Cameo/Sparx/CATIA Magic/Siemers — full system-model checking |
| **Often a "single source of truth"** | In aerospace/automotive, the SysML model is the master from which designs are derived |

> Like [[ArchiMate]], SysML in serious use is **strongly metamodel-coupled** — the model is the truth, the views are projections. See [[Architecture Metamodel]].

### 4. Tooling & format — *specialized, model-based*

- **Authoring tools:** Cameo Systems Modeler, Sparx EA, CATIA Magic, Siemens Polarion, IBM Rational Rhapsody.
- **Interchange format:** **XMI** (shared with UML — SysML is a UML profile, so it interchanges via XMI).
- **Simulation/analysis:** parametrics support engineering analysis (e.g., trade studies, performance simulation).

## ↔️ SysML vs. UML — when to use which

| | SysML | [[UML]] |
|---|---|---|
| Domain | Systems engineering (HW + SW + people + facilities) | Software |
| Central element | Block (generic system part) | Class (software structure) |
| Requirements | **First-class** (Requirement diagram) | Not first-class |
| Parametrics/engineering analysis | **Yes** (Constraint blocks) | No |
| Deployment diagram | No (Block handles physical layout) | Yes |
| Diagram count | 9 | 14 |
| Use when | Aerospace, automotive, defense, medical devices | Pure software |

> Rule of thumb: if your system is **only software**, use UML (or [[C4 Model|C4]]). If it **mixes disciplines** (a car, a satellite, a robotic system), use SysML.

## ⚠️ Pitfalls & Anti-patterns

- **Using SysML for pure software:** it's overkill and lacks UML's software-specific diagrams (no deployment diagram). Use UML/C4 instead.
- **Treating SysML as "UML with extra diagrams":** it's a deliberate redesign — Block (not Class), first-class requirements, parametrics. The mindset is systems engineering, not software.
- **Ignoring parametrics:** they're the unique value for engineering analysis. Modeling structure without parametrics misses the systems-engineering payoff.
- **Drawing-only SysML:** like UML, SysML loses its value if sketched without a model. Use a model-first tool.
- **Over-modeling:** SysML can model *everything* about a system. Model what's architecturally and contractually significant.
- **Confusing Block with Class:** a Block generalizes Class — it can be hardware, software, or person. Don't assume "Block = software class."

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Notation]] (the four notation axes), [[Architecture Framework]]
- **Concrete notation siblings:** [[UML]] (SysML is UML-derived), [[ArchiMate]], [[BPMN]]
- **Contrast:** [[UML]] (software-only), [[ArchiMate]] (enterprise), [[C4 Model]] (software-system, simpler)
- **Related Concepts:** [[Architecture Metamodel]] (SysML is strongly coupled), [[Architectural Views]] (SysML diagrams are views), [[System]] (SysML models the whole system, not just software), [[Levels of Abstraction]]
- **Prerequisites:** [[Architecture Notation]], [[Architecture Framework]], basic familiarity with [[UML]].

## ❓ Active Recall Questions

> [!faq]- What is SysML, and how does it relate to UML?
> SysML (Systems Modeling Language) is a **systems-engineering notation** — a UML profile that *reuses part of UML*, *removes* software-specific diagrams (object, communication, deployment), and *adds* systems essentials (requirements, parametrics, block definition, allocation). It models hardware + software + people + facilities as one system, whereas [[UML]] assumes the system is software.

> [!faq]- What are the four notation axes' values for SysML?
> **Symbol set:** UML-derived + systems extensions — Block (generic system part), first-class Requirement, parametric Constraint blocks, allocations. **Diagram types:** 9 kinds (trimmed from UML's 14 + systems additions: Requirement, Parametric, Internal Block). **Metamodel coupling:** strong — model-first, often the single source of truth in aerospace/automotive. **Tooling/format:** specialized tools (Cameo, Sparx, CATIA), XMI interchange.

> [!faq]- What does SysML add that UML lacks, and why do those matter?
> **Requirement diagrams** (first-class requirements tied to verifiable system properties) and **Parametric diagrams** (engineering constraint blocks binding physical/mathematical relationships like mass, thrust, power). They matter because systems engineering is contract-driven (requirements) and analysis-driven (engineering trade studies) — neither of which UML can express.

> [!faq]- When should you choose SysML over UML?
> When the system **mixes disciplines** — hardware + software + people + facilities (a car, a satellite, a medical device, a robotic system). For pure software, [[UML]] (or [[C4 Model|C4]]) is simpler and has software-specific diagrams SysML dropped (like deployment). SysML is overkill for a web service.

---

## 📝 Related Notes

- [[Architecture Notation]]
- [[Architecture Framework]]
- [[UML]]
- [[ArchiMate]]
- [[BPMN]]
- [[Architecture Metamodel]]
- [[Architectural Views]]
- [[System]]
- [[Levels of Abstraction]]
