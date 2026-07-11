---
status: researching
tags: [c4-model, architecture, viewpoint, component, c4-level-3, udemy, fundamentals]
created: 2026-07-10
updated: 2026-07-10
difficulty: intermediate
---

# 🧠 Topic: Component Viewpoint (C4 Level 3)

> [!abstract] Executive Summary (ELI5)
> The Component Viewpoint **zooms into a single container** and shows the building blocks inside it — the classes, modules, services, or packages that together implement that container. Where the Container Viewpoint showed "the API is one box," the Component Viewpoint opens that API box and shows its internals: the auth controller, the order service, the payment repository, etc. It's the level developers work in daily.

> [!warning] The #1 confusion — "Component" the C4 level vs "component" the generic term
> The word "component" is overloaded. In C4 Level 3, a **component** is a specific thing: a grouping of related code (classes/modules) *inside a single container* with a clear responsibility and interface. In the generic sense (see [[Key Elements of Architecture View]]), "component" means any building block at *any* level. When the lecture says "Component Viewpoint," it means the C4 L3 zoom — internals of one container.

---

## 🎯 The "Why"

- **Problem Solved:** For the containers that carry the most complexity (typically the API/backend services), showing them as a single box isn't enough to guide development. The Component Viewpoint reveals the internal structure — how responsibilities are divided, how the pieces depend on each other — so developers can navigate, test, and evolve the code.
- **Alternatives:** Letting internal structure live only in the code. Result: onboarding is slow, and the team has no shared picture of how the container is organized. The Component diagram externalizes that picture.
- **Trade-offs:** Component diagrams are the most volatile C4 level — code refactoring changes them constantly. The discipline is to model only the *significant* components (the ones with architectural weight), not every class. Many teams don't draw them at all unless the container is complex enough to warrant it.

## 💡 Mental Model & Analogy

- **Analogy:** If the Container Viewpoint is the **floor plan** (rooms of a building), the Component Viewpoint is the **furniture layout of one room** — the desks, shelves, and workstations inside a single room, who sits where, and how they collaborate. You don't furniture-layout every room in the building; you do it for the rooms where the arrangement matters.

```mermaid
graph TD
    subgraph "Container: API (opened up)"
        AC[Auth Controller] --> AS[Auth Service]
        AS --> AR[User Repository]
        OC[Order Controller] --> OS[Order Service]
        OS --> OR[Order Repository]
        OS -->|uses| AS
        OR -->|persists to| DB_OUT[("Database<br/>(external to this container)")]]
        AR -->|persists to| DB_OUT
    end
    style AC fill:#dbeafe,stroke:#1d4ed8
    style OS fill:#dcfce7,stroke:#15803d
```

Each box is a component — a code grouping with one responsibility. The database is *outside* this container (it's its own container), shown here only as a dependency.

## 🛠️ Core Principles

1. **Components live *inside* a single container.** A component never spans containers. It's a subdivision of one container's code.
2. **A component has a single, clear responsibility.** If you can't describe it in one sentence, it's under-decomposed. (See [[Key Elements of Architecture View]] → "a component with five responsibilities is five components.")
3. **A component has an explicit interface.** What it offers to other components, and what it requires from them. The interface is the contract; the implementation behind it is free to change.
4. **Model only significant components.** Not every class — the ones with architectural weight (services, controllers, repositories, domain logic). Trivial helpers don't need to be on the diagram.
5. **Component diagrams are per-container.** You draw a Component diagram for *one* container at a time — usually the most complex one (the API/backend). Not every container needs one.

## 🏗️ What Counts as a Component?

| Likely a component | Probably not (too granular) |
|---|---|
| A service class (OrderService, PaymentService) | A single DTO or entity class |
| A controller (AuthController, OrderController) | A utility/helper function |
| A repository / data-access module | A single config file |
| A domain logic module | A logger wrapper |
| A package/subsystem with a cohesive responsibility | Individual classes within such a package |

> The test: *does this grouping have a distinct responsibility and interface that other components depend on?* If yes, it's a component. If it's an internal implementation detail of another component, don't model it separately.

## ↔️ How the Component Viewpoint Connects

| Connects to | How |
|---|---|
| [[Container Viewpoint]] (C4 L2) | The Component Viewpoint opens one container. One container → many components. |
| [[Development Viewpoint]] | Components are often 1:1 with code modules/packages. This is where C4 L3 lives in practice — the Development Viewpoint's code structure. |
| [[Key Elements of Architecture View]] | Components (in the C4 sense) are the "components" of the triad; their dependencies are connectors. |
| [[Evolution Perspective]] | Clear component boundaries and interfaces are what make the container evolvable. |
| [[Development Resource Perspective]] | Component structure determines testability and cognitive load for developers. |

## ⚠️ Pitfalls & Anti-patterns

- **Modeling every class:** defeats the purpose. The diagram becomes a code map, not an architecture view. Model only significant components.
- **Components with fuzzy responsibilities:** "CoreService" that does everything. Under-decomposed; split it.
- **Cross-container components:** a component that spans two containers. By definition, components live inside one container — if it spans, you've mis-scoped a container boundary.
- **Drawing it for every container:** not every container needs a Component diagram. Reserve it for the complex ones (usually backends); a simple database container has nothing to show here.
- **Not updating it:** the most volatile C4 level. A stale component diagram is worse than none — it misleads. Either maintain it or don't create it.
- **Confusing C4 component with generic "component":** the lecture's "Component Viewpoint" is the C4 L3 zoom. Don't conflate with the generic term from [[Key Elements of Architecture View]].

## 🔗 Deep Connections

- **Parent Topic:** [[Viewpoint]], [[System Modeling]]
- **C4 neighbors:** [[Container Viewpoint]] (L2, one level up — the container being opened), [[Context Viewpoint]] (L1, two levels up)
- **Strongly related:** [[Development Viewpoint]] (components ≈ code modules; C4 L3 lives here in practice), [[Key Elements of Architecture View]]
- **Perspectives that bite here:** [[Evolution Perspective]] (changeability of component boundaries), [[Development Resource Perspective]] (testability, cognitive load)
- **Prerequisites:** [[Container Viewpoint]], [[Viewpoint]], [[System]].

## ❓ Active Recall Questions

> [!faq]- What does the Component Viewpoint show, and when do you draw it?
> It **zooms into a single container** and shows the significant code building blocks inside it (services, controllers, repositories, domain modules) — their responsibilities, interfaces, and dependencies. You draw it per-container, and only for containers complex enough to warrant it (usually the API/backend). A simple database container doesn't need one.

> [!faq]- What's the difference between a C4 "component" and the generic term "component"?
> In C4 L3, a **component** is specific: a grouping of related code *inside a single container* with one clear responsibility and interface. In the generic sense (see [[Key Elements of Architecture View]]), "component" means any building block at any level (a system, a container, and a code module are all "components" generically). The lecture's "Component Viewpoint" means the C4 L3 zoom.

> [!faq]- What's the test for whether something is a component worth modeling?
> *Does this grouping have a distinct responsibility and interface that other components depend on?* If yes, it's a component (a service, controller, repository). If it's an internal implementation detail (a DTO, a helper function, a config file), don't model it separately. Model only components with architectural weight.

> [!faq]- Can a component span multiple containers?
> No — by definition, a C4 component lives *inside* a single container; it's a subdivision of one container's code. If something seems to span containers, you've likely mis-scoped a container boundary, not discovered a cross-container component.

> [!faq]- How does the Component Viewpoint relate to the Development Viewpoint?
> They overlap heavily: components are often 1:1 with code modules/packages, so C4 L3 is where the Development Viewpoint's code structure becomes visible. The Development Viewpoint is the lens for *how code is organized and built*; the Component Viewpoint is the C4 zoom level that visualizes that organization inside one container.

---

## 📝 Related Notes

- [[Container Viewpoint]]
- [[Context Viewpoint]]
- [[Development Viewpoint]]
- [[Key Elements of Architecture View]]
- [[Evolution Perspective]]
- [[Development Resource Perspective]]
- [[Viewpoint]]
