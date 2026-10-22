---
status: archived
tags: [c4-model, architecture, togaf, partitioning, enterprise-architecture, enterprise, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Enterprise Architecture (Partition Level)

> [!abstract] Executive Summary (ELI5)
> At the **Enterprise** partition level, you model the **whole organization** at the **broadest scope and least detail**. The job is not to design systems — it is to set **strategy, principles, and governance** that the lower levels ([[Segment Architecture|Segments]] and [[Capability Architecture|Capabilities]]) must follow. Think of it as the **constitution** of the enterprise's architecture: short, foundational, and direction-setting, with the detailed laws made at lower levels.

> [!warning] Naming ambiguity — "Enterprise Architecture" means two things
> The term is overloaded. (1) **The whole discipline** of architecting an enterprise (what TOGAF as a whole is for). (2) **The top partition level** — the specific, broadest-scope/least-detail slice (this note). When TOGAF talks about partitioning, "Enterprise Architecture" means sense (2): the strategic-top partition, distinct from Segment and Capability.

> [!note] Part of the TOGAF deep-dive — the top partition level
> One of three levels in [[Architecture Partitioning]]. Scope: whole org; detail: least; timescale: longest.

---

## 🎯 The "Why"

- **Problem Solved:** Without an enterprise-level architecture, each Segment and project goes its own way — duplicating capabilities, violating shared principles, buying incompatible tech. The Enterprise level sets the **direction and guardrails** so the lower levels are consistent and the whole adds up.
- **Alternatives:** Letting direction emerge bottom-up from projects. Result: inconsistency, duplication, and no coherent target.

## 🛠️ Core Concerns

| Concern | What it sets |
|---|---|
| **Strategy** | The enterprise's strategic direction and business drivers |
| **Architecture principles** | Foundational rules every lower level must obey (e.g., "buy before build") |
| **Governance** | The Architecture Board and the conformance process (see [[Architecture Capability Framework]]) |
| **Enterprise-wide standards** | Shared tech standards, reference models ([[Technical Reference Model\|TRM]], [[III-RM]]) |
| **Target direction** | The high-level [[Target Architecture|Target]] for the whole enterprise |

> The Enterprise level is **deliberately high-level**. It should not contain Component-level detail — that belongs at the [[Capability Architecture|Capability]] level. Its output is principles, standards, and a strategic target, not solution designs.

## ↔️ Where it sits

- **Partition level:** top (broadest scope, least detail, longest timescale).
- **Directs:** [[Segment Architecture|Segments]] and [[Capability Architecture|Capabilities]] — they realize parts of it and must conform to its principles.
- **Governed by:** the Architecture Board at this level (enterprise-wide governance).
- **Runs an ADM cycle:** the [[Architecture Development Method|ADM]] at this level focuses on Phases A (principles), B (vision), and the strategic target.

## ⚠️ Pitfalls & Anti-patterns

- **Too much detail:** trying to design systems at the Enterprise level. It should set direction, not solutions.
- **No teeth:** principles with no governance to enforce them. Lower levels then ignore the direction.
- **Disconnected from Segments:** an enterprise target that no Segment realizes — pure aspiration.

## 🔗 Deep Connections

- **Parent Topic:** [[Architecture Partitioning]], [[TOGAF]]
- **Sibling levels:** [[Segment Architecture]], [[Capability Architecture]]
- **What it sets:** principles/governance via [[Architecture Capability Framework]]; reference models ([[Technical Reference Model|TRM]], [[III-RM]])
- **Prerequisites:** [[TOGAF]], [[Architecture Partitioning]].

## ❓ Active Recall Questions

> [!faq]- What is the Enterprise partition level, and what is its job?
> It models the **whole organization at the broadest scope and least detail**. Its job is to set **strategy, principles, governance, and enterprise-wide standards** — the direction and guardrails that Segments and Capabilities must follow. It is the "constitution," not the detailed laws.

> [!faq]- Why must the Enterprise level stay high-level?
> Because its scope is the entire organization — it cannot be detailed without becoming unreadable and never finishing. Detail belongs at the [[Capability Architecture|Capability]] level; the Enterprise level sets direction and principles that lower levels realize and conform to.

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Partitioning]]
- [[Segment Architecture]]
- [[Capability Architecture]]
- [[Architecture Capability Framework]]
- [[Technical Reference Model]]
- [[III-RM]]
