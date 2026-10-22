---
status: archived
tags: [c4-model, architecture, togaf, partitioning, segment-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Segment Architecture

> [!abstract] Executive Summary (ELI5)
> At the **Segment** partition level, you model **one business area** (e.g., "Retail," "HR," "Manufacturing," "Customer Service") at **medium scope and medium detail**. It sits between the [[Enterprise Architecture|Enterprise level]] (which sets direction for the whole org) and the [[Capability Architecture|Capability level]] (which designs specific projects). A Segment architecture takes the enterprise's principles and target, specializes them to one business area, and provides the context that the area's Capability architectures fit into.

> [!note] Part of the TOGAF deep-dive — the middle partition level
> One of three levels in [[Architecture Partitioning]]. Scope: one business area; detail: medium; timescale: medium.

---

## 🎯 The "Why"

- **Problem Solved:** The Enterprise level is too high-level to guide specific projects, but modeling the whole org at project detail is impossible. The Segment level is the **missing middle**: it makes one business area coherent — its capabilities, applications, data — so projects in that area have a consistent context and don't duplicate or contradict each other.
- **Alternatives:** Jumping from Enterprise principles straight to project-level (Capability) architectures. Result: projects in the same area reinvent shared capabilities and diverge.

## 🛠️ Core Concerns

| Concern | What it captures |
|---|---|
| **Business area scope** | The boundaries of the segment (e.g., "all of Retail") |
| **Capability map for the area** | The capabilities within this segment (specialized from the enterprise map) |
| **Area-specific applications & data** | The apps/data serving this segment |
| **Conformance to enterprise** | How this segment realizes enterprise principles/target |
| **Context for capabilities** | The framework that the segment's [[Capability Architecture\|Capability]] projects fit into |

> A Segment architecture covers all four [[TOGAF Architecture Domains|BDAT domains]] *for its area* — but only for that area, and at medium detail. It is the scope where a business area becomes coherent.

## ↔️ Where it sits

- **Partition level:** middle (one business area, medium detail, medium timescale).
- **Conforms to:** [[Enterprise Architecture]] (inherits principles, standards, target direction).
- **Provides context for:** [[Capability Architecture]] projects within the segment.
- **Runs an ADM cycle:** a focused [[Architecture Development Method|ADM]] pass scoped to the segment.

## ⚠️ Pitfalls & Anti-patterns

- **Segment scope too vague:** "the customer area" with no clear boundary. Define the segment's extent.
- **Segment disconnected from Enterprise:** ignoring enterprise principles/standards, becoming a silo.
- **Too much detail:** designing project solutions at the Segment level — that's Capability's job.
- **Overlapping segments:** two segments claiming the same capabilities. Segment boundaries should be clean.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Partitioning]], [[TOGAF]]
- **Sibling levels:** [[Enterprise Architecture]] (above), [[Capability Architecture]] (below)
- **What it spans:** [[TOGAF Architecture Domains]] (all four, for its area), [[Architecture Time Horizons]] (Baseline/Target for the area)
- **Prerequisites:** [[TOGAF]], [[Architecture Partitioning]].

## ❓ Active Recall Questions

> [!faq]- What is the Segment partition level, and what gap does it fill?
> It models **one business area** at **medium scope and detail** — the "missing middle" between the [[Enterprise Architecture|Enterprise level]] (too high to guide projects) and the [[Capability Architecture|Capability level]] (too narrow to keep an area coherent). It specializes enterprise principles to one area and provides context for that area's projects.

> [!faq]- Does a Segment architecture cover all four BDAT domains?
> Yes — but **only for its business area**, and at medium detail. A Segment covers the Business, Data, Application, and Technology of its area (e.g., "all of Retail"), providing a coherent context for the Capability projects within it. See [[TOGAF Architecture Domains]].

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Partitioning]]
- [[Enterprise Architecture]]
- [[Capability Architecture]]
- [[TOGAF Architecture Domains]]
- [[Architecture Development Method]]
