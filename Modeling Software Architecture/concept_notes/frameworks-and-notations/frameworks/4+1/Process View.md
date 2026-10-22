---
status: archived
tags: [c4-model, architecture, 4+1, process-view, kruchten, uml, concurrency, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Process View (4+1)

> [!abstract] Executive Summary (ELI5)
> The **Process View** is the second of the four views in the [[4+1 View Model|4+1 View Model]]. It describes the system's **runtime concurrency** — the processes, threads, and tasks that exist when the system is running, and how they communicate, synchronize, and share resources. It captures the **dynamic, runtime behaviour**: performance, scalability, availability, and fault tolerance all live here. It is the view the **integrator and performance engineer** care about.

> [!warning] The #1 confusion — "Process" = runtime concurrency, NOT business process
> The word "process" collides with "business process" (the workflows the business performs). In 4+1, **Process View means runtime concurrency**: operating-system processes, threads, tasks, and their interactions. It maps to R&W's [[Concurrency Viewpoint]], *not* to business process modeling. This collision trips many readers.

> [!note] Part of the 4+1 deep-dive
> One of four views in the [[4+1 View Model|4+1]] framework. The general concept of "runtime concurrency modeling" is covered in the cross-linked course viewpoint.

---

## 🎯 The "Why"

- **Problem Solved:** The [[Logical View (4+1)|Logical View]] shows the functional classes, but a single class diagram says nothing about **what runs concurrently**, how the system handles load, or what happens when a thread or process fails. The Process View answers those questions: which runtime instances exist, how they coordinate, and where the performance and availability risks are.
- **Alternatives:** Hoping concurrency "works out" at implementation time. Result: race conditions, deadlocks, throughput bottlenecks, and unavailability discovered only in production.
- **Trade-offs:** The Process View is the most behaviorally complex of the four — runtime interactions are harder to model than static structure. The payoff is that the expensive runtime bugs (contention, deadlocks, single points of failure) become visible on the diagram.

## 🛠️ Core Characteristics

| Aspect | Process View |
|---|---|
| **Concern** | Runtime concurrency — processes, threads, tasks, their communication/synchronization |
| **Primary audience** | Integrator, performance engineer |
| **UML notation** | Activity diagrams, concurrency diagrams, state diagrams |
| **Key elements** | Processes, threads, tasks, messages, shared resources, communication paths |
| **BDAT domain** | Application + Technology (runtime layer) |
| **Quality concerns captured** | Performance, scalability, availability, fault tolerance |

> This is the view where **quality attributes are most explicit** in 4+1. Whereas R&W would apply a [[Performance and Scalability Perspective]] or [[Availability and Resilience Perspective]] *to* a view, 4+1 embeds these concerns *inside* the Process View by design.

## ↔️ Mapping to Rozanski & Woods

| Process View contains | R&W equivalent | Note |
|---|---|---|
| Runtime concurrency (processes, threads) | [[Concurrency Viewpoint]] | direct equivalent |
| Performance/scalability concerns | [[Performance and Scalability Perspective]] | 4+1 *embeds*; R&W *applies as a perspective* |
| Availability/fault-tolerance concerns | [[Availability and Resilience Perspective]] | 4+1 *embeds*; R&W *applies as a perspective* |

> The key contrast: 4+1 **embeds** performance and availability inside the Process View; R&W **separates** them as cross-cutting perspectives that can be applied to *any* view (including but not limited to concurrency). R&W's move gives quality concerns a first-class home across all views, not just the concurrency one. See [[4+1 View Model]].

## 🧱 Designing a Good Process View

- **Identify the runtime units** — processes, threads, tasks, replicas. How many instances of each exist at runtime?
- **Map communication** — synchronous RPC, async messaging, shared memory, event channels.
- **Capture control flow** — startup/shutdown sequences, request handling, error paths.
- **Model failure behaviour** — what happens when a process/thread dies? Retries, failover, dead-letter queues. (Pair with [[Availability and Resilience Perspective]].)
- **Address the bottlenecks** — shared resources, locks, contention points, single points of failure.

## ⚠️ Pitfalls & Anti-patterns

- **Confusing Process View with business process:** the #1 trap. Process = runtime concurrency (threads/processes), not business workflows.
- **Only the happy path:** showing startup and request handling but not failure/recovery. The most expensive bugs live in the failure paths.
- **Ignoring the runtime topology:** forgetting replicas, pools, autoscalers. The static [[Logical View (4+1)|Logical]] class diagram hides that you may have 50 instances.
- **Static-only modeling:** drawing the Process View as a fixed snapshot when runtime instances are elastic (autoscaling). Note the variability.
- **Conflating with the Physical View:** the Process View shows *what runs concurrently* (logical runtime units); the [[Physical View (4+1)|Physical View]] shows *where* those run on hardware. Different concerns.

## 🔗 Deep Connections

- **Parent Topic:** [[4+1 View Model]]
- **Sibling 4+1 views:** [[Logical View (4+1)]], [[Development View (4+1)]], [[Physical View (4+1)]], [[Scenarios (4+1)]]
- **R&W equivalents:** [[Concurrency Viewpoint]] (direct), with [[Performance and Scalability Perspective]] and [[Availability and Resilience Perspective]] as embedded concerns
- **Related Concepts:** [[Viewpoint]], [[Perspective]] (R&W's quality axis — the thing 4+1 embeds instead), [[Container Viewpoint]] (C4's runtime unit ≈ a process), [[Architecture Domains]]
- **Prerequisites:** [[4+1 View Model]], [[Viewpoint]].

## ❓ Active Recall Questions

> [!faq]- What does the Process View show, and why is the word "process" misleading?
> It shows **runtime concurrency** — processes, threads, tasks, and their communication/synchronization. "Process" is misleading because it collides with "business process"; here it means OS/runtime processes and threads, not business workflows. It maps to R&W's [[Concurrency Viewpoint]].

> [!faq]- Which quality concerns does the Process View capture, and how does that differ from R&W?
> 4+1 **embeds** performance, scalability, availability, and fault tolerance *inside* the Process View. R&W instead **separates** these as cross-cutting perspectives ([[Performance and Scalability Perspective]], [[Availability and Resilience Perspective]]) that can be applied to *any* view. 4+1 bundles them into one view; R&W gives them a first-class home across all views.

> [!faq]- Why is "only modeling the happy path" a pitfall in the Process View?
> Because the most expensive runtime bugs — race conditions, deadlocks, single points of failure, retry storms — live in the failure and recovery paths. A Process View that shows only startup and request handling hides exactly where production incidents originate.

> [!faq]- How does the Process View differ from the Physical View?
> The **Process View** shows *what runs concurrently* — logical runtime units (processes, threads, replicas) and their interactions. The **[[Physical View (4+1)|Physical View]]** shows *where* those runtime units are deployed on hardware (nodes, networks). Process = runtime concurrency; Physical = deployment mapping.

---

## 📝 Related Notes

- [[4+1 View Model]]
- [[Logical View (4+1)]]
- [[Development View (4+1)]]
- [[Physical View (4+1)]]
- [[Scenarios (4+1)]]
- [[Concurrency Viewpoint]]
- [[Performance and Scalability Perspective]]
- [[Availability and Resilience Perspective]]
- [[Viewpoint]]
- [[Perspective]]
