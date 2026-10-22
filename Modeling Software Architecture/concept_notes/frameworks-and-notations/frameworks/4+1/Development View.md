---
status: archived
tags: [c4-model, architecture, 4+1, development-view, kruchten, uml, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Development View (4+1)

> [!abstract] Executive Summary (ELI5)
> The **Development View** is the third of the four views in the [[4+1 View Model|4+1 View Model]]. It describes the system's **static code organization** — the modules, packages, layers, and subsystems the source code is divided into, and their compile-time dependencies. It is the view the **programmer** cares about: "how is the codebase laid out, and what depends on what?" It says nothing about runtime concurrency or deployment — only how the code is structured on disk.

> [!warning] The #1 confusion — Development View = code structure, NOT the dev process
> "Development" here means **the code's static structure** (modules/packages/layers), *not* the process of developing software (agile, CI/CD, etc.). It maps to R&W's [[Development Viewpoint]] — same idea, same audience (programmers). The name refers to the developer's perspective on the codebase, not to development activities.

> [!note] Part of the 4+1 deep-dive
> One of four views in the [[4+1 View Model|4+1]] framework. This note focuses on 4+1's framing; the general concept of "code structure modeling" is in the cross-linked course viewpoint.

---

## 🎯 The "Why"

- **Problem Solved:** A new developer joining the project needs to know *how the code is organized* — what the modules are, what the layers are, what may depend on what. The Development View externalizes that picture, so onboarding is fast and the team shares a common map of the codebase.
- **Alternatives:** Letting code structure live only in the code itself. Result: slow onboarding, hidden coupling, and architectural drift nobody can see.
- **Trade-offs:** The Development View is the **most volatile** of the four — refactoring changes it constantly. The discipline is to model the *architecturally significant* structure (the major modules, the layer rules), not every package. Many teams update it rarely or not at all for simple systems.

## 🛠️ Core Characteristics

| Aspect | Development View |
|---|---|
| **Concern** | Static code organization — modules, packages, layers, subsystems |
| **Primary audience** | Programmer (developer) |
| **UML notation** | Component diagram, package diagram |
| **Key elements** | Modules, packages, layers, subsystems, compile-time dependencies |
| **BDAT domain** | Application |
| **Static or dynamic?** | **Static** (compile-time; contrast with [[Process View (4+1)\|Process View]] which is runtime) |

> The Development View answers questions like: "Is there a UI → Domain → Data Access layering? May the UI call the database directly? Which modules depend on which?" These are **architectural rules**, not just descriptions.

## ↔️ Mapping to Rozanski & Woods

| Development View contains | R&W equivalent | Note |
|---|---|---|
| Code modules, packages, layers, dependencies | [[Development Viewpoint]] | direct equivalent |
| Allowed dependency rules (layering constraints) | [[Development Viewpoint]] | part of the same |

> This is the cleanest 1:1 mapping in 4+1 — 4+1's Development View and R&W's [[Development Viewpoint]] are essentially the same thing, for the same audience (programmers), answering the same question ("how is the code organized?"). See [[4+1 View Model]].

## 🧱 Designing a Good Development View

- **Show the layering** — UI, domain, data access, infrastructure. The layer rules *are* the architecture.
- **Map the major modules/packages** — not every namespace; the architecturally significant ones.
- **Capture dependency rules** — "UI may not call Data Access directly," "Domain depends on nothing outward." These constraints are the value.
- **Show subsystem boundaries** — cohesive groups of modules that could be developed/released independently.
- **Keep it honest** — a Development View that doesn't match the real folder layout is worse than none.

## ⚠️ Pitfalls & Anti-patterns

- **Conflating with the Logical View:** the [[Logical View (4+1)|Logical View]] shows *domain classes* (functional); the Development View shows *code modules/packages*. The same class can appear in both, but the concern differs.
- **Conflating with the development process:** "Development" = code structure, not agile/CI/CD.
- **Modeling every package:** over-granular diagrams become unmaintainable. Model architecturally significant structure only.
- **Not updating it:** the most volatile view. A stale Development View misleads; either maintain it or don't create it.
- **No dependency rules:** just a graph of "depends on" with no constraints. The *rules* (what's allowed/forbidden) are the architectural content.
- **Drawing runtime/concurrency here:** that belongs to the [[Process View (4+1)|Process View]]. The Development View is compile-time/static.

## 🔗 Deep Connections

- **Parent Topic:** [[4+1 View Model]]
- **Sibling 4+1 views:** [[Logical View (4+1)]], [[Process View (4+1)]], [[Physical View (4+1)]], [[Scenarios (4+1)]]
- **R&W equivalent:** [[Development Viewpoint]] (direct 1:1 mapping)
- **Related Concepts:** [[Viewpoint]], [[Component Viewpoint]] (C4's code-level zoom — analogous to a component/module decomposition), [[Levels of Abstraction]]
- **Prerequisites:** [[4+1 View Model]], [[Viewpoint]].

## ❓ Active Recall Questions

> [!faq]- What does the Development View show, and who is it for?
> The system's **static code organization** — modules, packages, layers, subsystems, and their compile-time dependencies. It is for the **programmer** (developer), answering "how is the codebase laid out, and what may depend on what?" It is drawn with UML component/package diagrams.

> [!faq]- Does "Development View" refer to the development process (agile, CI/CD)?
> No. "Development" here means **the code's static structure** as seen by a developer — modules, packages, layers — not the process of developing software. It maps directly to R&W's [[Development Viewpoint]] (same concern, same audience).

> [!faq]- How does the Development View differ from the Logical View?
> The **[[Logical View (4+1)|Logical View]]** shows *domain classes/objects* (functional structure, for end-users/analysts). The **Development View** shows *code modules/packages* (static code organization, for programmers). A class can appear in both, but the concern is different — functional abstraction vs. code packaging.

> [!faq]- Why is the Development View the most volatile of the four?
> Because code structure changes constantly through refactoring — far more often than the functional model, runtime topology, or deployment. The discipline is to model only architecturally significant structure (major modules, layer rules), and to either keep it current or not create it at all (a stale Development View misleads).

---

## 📝 Related Notes

- [[4+1 View Model]]
- [[Logical View (4+1)]]
- [[Process View (4+1)]]
- [[Physical View (4+1)]]
- [[Scenarios (4+1)]]
- [[Development Viewpoint]]
- [[Viewpoint]]
- [[Component Viewpoint]]
