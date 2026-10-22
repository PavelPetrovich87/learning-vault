---
status: archived
tags: [c4-model, architecture, togaf, partitioning, capability-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Capability Architecture (Partition Level)

> [!abstract] Executive Summary (ELI5)
> At the **Capability** partition level, you model **a specific function or project** at the **narrowest scope and most detail**. This is where architecture meets implementation: detailed solution designs for one capability or project (e.g., "Online Checkout," "Payroll migration"). It takes the context from its [[Segment Architecture|Segment]] (and the [[Enterprise Architecture|Enterprise]] principles above that) and produces a concrete, buildable solution. It is the level developers and project teams work in.

> [!warning] Naming — "Capability Architecture" vs. "Business Capability" vs. the "Capability Framework"
> Three different "capabilities." (1) **Business Capability** — a BDAT Business-domain concept ("Process Payment"). (2) **[[Architecture Capability Framework]]** — TOGAF's *practice* (board, skills, governance). (3) **Capability Architecture** (this note) — the *bottom partition level*: detailed solutions for a specific function/project. Keep them distinct.

> [!note] Part of the TOGAF deep-dive — the bottom partition level
> One of three levels in [[Architecture Partitioning]]. Scope: one function/project; detail: most; timescale: shortest (project-length).

---

## 🎯 The "Why"

- **Problem Solved:** Strategy and segment-level architecture don't build systems — projects do. The Capability level is where **architecture becomes concrete enough to implement**: a detailed solution that a project team can build, test, and deliver, while still conforming to the segment and enterprise context above it.
- **Alternatives:** Projects architecting in isolation (no segment/enterprise context) — reinventing solutions, violating standards. Or trying to run all projects from the Enterprise level (too abstract to be buildable).

## 🛠️ Core Concerns

| Concern | What it captures |
|---|---|
| **Project / function scope** | The specific capability or project being delivered |
| **Detailed solution design** | Concrete components, interfaces, data, deployment — buildable detail |
| **Conformance to Segment/Enterprise** | Fits the segment context; obeys enterprise principles/standards |
| **Solution Building Blocks (SBBs)** | The concrete products/choices that realize the Architecture Building Blocks (ABBs) — see [[Architecture Content Framework]] |
| **Project-level Target & Transition** | The [[Target Architecture\|Target]] and [[Transition Architecture\|Transitions]] for this project |

> This is the level where [[Architecture Metamodel|ABBs become SBBs]] — where "we need a payment component" (ABB, from higher levels) becomes "we will use Stripe API v2" (SBB, here).

## ↔️ Where it sits

- **Partition level:** bottom (one function/project, most detail, project-length timescale).
- **Conforms to:** [[Segment Architecture]] (inherits area context) and [[Enterprise Architecture]] (inherits principles/standards).
- **Produces:** buildable solution designs, SBBs, project-level Transition Architectures.
- **Runs an ADM cycle:** a short, project-scoped [[Architecture Development Method|ADM]] pass — often trimmed.

> This is where the [[C4 Model]] and its Component-level detail naturally live — C4 is a notation for capability/project-level application architecture. See [[Application Architecture]].

## ⚠️ Pitfalls & Anti-patterns

- **Ignoring segment/enterprise context:** a project solution that violates enterprise principles or duplicates a segment capability. Conformance matters.
- **Over-reaching scope:** a "Capability" architecture that tries to redesign the whole segment. Keep it project-scoped.
- **No link to ABBs:** designing solutions without tracing to the Architecture Building Blocks defined above — loses the enterprise traceability chain.
- **Skipping Transition planning:** jumping to Target without intermediate Transitions — high-risk big-bang. See [[Architecture Time Horizons]].

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Partitioning]], [[TOGAF]]
- **Sibling levels:** [[Enterprise Architecture]] (top), [[Segment Architecture]] (middle)
- **Where ABBs → SBBs:** [[Architecture Content Framework]], [[Architecture Metamodel]]
- **Notation that fits here:** [[C4 Model]] (Component-level detail)
- **Prerequisites:** [[TOGAF]], [[Architecture Partitioning]].

## ❓ Active Recall Questions

> [!faq]- What is the Capability partition level, and how does it differ from a Business Capability?
> It is the **bottom partition level** — detailed, buildable solution architecture for **one specific function or project** (e.g., "Online Checkout"). It differs from a **Business Capability** (a BDAT Business-domain concept like "Process Payment") and from the **[[Architecture Capability Framework]]** (TOGAF's practice of board/skills/governance). Same word, three meanings.

> [!faq]- What happens at the Capability level that doesn't happen higher up?
> **ABBs become SBBs** — "we need a payment component" (Architecture Building Block, defined higher) becomes "we will use Stripe API v2" (Solution Building Block, chosen here). This is also where architecture becomes concrete enough to implement, often rendered in [[C4 Model|C4]] Component-level detail. See [[Architecture Content Framework]].

> [!faq]- Why must a Capability architecture conform to its Segment and Enterprise?
> Because it sits at the bottom of a nested hierarchy. It must realize the segment's area context and obey the enterprise's principles/standards — otherwise the project reinvents shared capabilities, violates standards, and the levels stop being coherent. Governance checks this conformance.

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Partitioning]]
- [[Enterprise Architecture]]
- [[Segment Architecture]]
- [[Architecture Content Framework]]
- [[Architecture Metamodel]]
- [[Architecture Time Horizons]]
- [[C4 Model]]
