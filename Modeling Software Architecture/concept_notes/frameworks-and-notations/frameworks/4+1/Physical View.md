---
status: archived
tags: [c4-model, architecture, 4+1, physical-view, kruchten, uml, deployment, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Physical View (4+1)

> [!abstract] Executive Summary (ELI5)
> The **Physical View** is the fourth of the four views in the [[4+1 View Model|4+1 View Model]]. It describes the system's **deployment topology** — the hardware nodes, networks, and environments the software runs on, and **which runtime element is mapped onto which node**. It answers "where does everything run, and how is it networked?" It is the view the **system engineer** cares about: deployment, networking, environments, and the physical/virtual infrastructure that hosts the [[Process View (4+1)|Process View's]] runtime units.

> [!warning] The #1 confusion — Physical View ≠ Process View
> They are different concerns. The **[[Process View (4+1)|Process View]]** shows *what runs concurrently* (processes, threads, replicas) — the runtime units. The **Physical View** shows *where* those runtime units are deployed on hardware (nodes, networks). Process = runtime concurrency; Physical = deployment mapping. The Physical View maps the Process View's elements onto nodes.

> [!note] Part of the 4+1 deep-dive
> One of four views in the [[4+1 View Model|4+1]] framework. The general "deployment" concept is covered in the cross-linked course viewpoint.

---

## 🎯 The "Why"

- **Problem Solved:** Software must run *somewhere*, and at any non-trivial scale the deployment topology — which nodes, which networks, which environments, which software goes where — is itself architecturally significant for availability, performance, security, and cost. The Physical View makes that topology explicit and checkable.
- **Alternatives:** Treating deployment as an ops afterthought. Result: unknown topology, security gaps, availability surprises, and cost overruns discovered only in production.
- **Trade-offs:** The Physical View is infrastructure-specific and changes when the deployment changes (cloud migrations, autoscaling). Its payoff is making deployment, HA topology, and network exposure visible and governable.

## 🛠️ Core Characteristics

| Aspect | Physical View |
|---|---|
| **Concern** | Deployment topology — hardware nodes, networks, environments, mapping of software to nodes |
| **Primary audience** | System engineer |
| **UML notation** | Deployment diagram |
| **Key elements** | Nodes, networks, environments, communication links, software-to-node mapping |
| **BDAT domain** | Technology |
| **Maps onto** | The [[Process View (4+1)\|Process View]]'s runtime units (each is deployed onto a node) |

> The canonical artifact is the **UML deployment diagram**: nodes (servers, VMs, regions), artifacts (deployable units) placed *on* nodes, and communication paths between nodes. The mapping of Process View elements (processes/threads) onto nodes is the heart of the Physical View.

## ↔️ Mapping to Rozanski & Woods

| Physical View contains | R&W equivalent | Note |
|---|---|---|
| Hardware nodes, networks, software-to-node mapping | [[Deployment Viewpoint]] | direct equivalent |

> Like the [[Development View (4+1)|Development View]], this is a clean 1:1 mapping: 4+1's Physical View and R&W's [[Deployment Viewpoint]] are essentially the same thing, for the same audience (system engineers), answering "where does everything run?" See [[4+1 View Model]].

## 🧱 Designing a Good Physical View

- **Identify the nodes** — physical servers, VMs, containers-as-hosts, cloud regions/availability zones.
- **Map the networks** — subnets, VPCs, load balancers, firewalls. Network topology carries security and availability decisions.
- **Show environments** — dev/test/stage/prod and their (often different) topologies.
- **Map software → nodes** — which process/replica runs on which node. This is the bridge from the [[Process View (4+1)|Process View]].
- **Capture failure domains** — single points of failure, HA topology (multi-AZ, replicas). Pair with [[Availability and Resilience Perspective]] and [[Security Perspective]].

## ⚠️ Pitfalls & Anti-patterns

- **Conflating with the Process View:** Process = *what runs* (concurrency); Physical = *where it runs* (deployment). The Physical View maps the Process View onto hardware.
- **Showing only production:** forgetting dev/test/stage environments. Their topology differs and matters for risk and cost.
- **No software-to-node mapping:** nodes drawn in isolation from what runs on them lose the traceability to the runtime architecture.
- **Ignoring network/firewall detail:** "everything talks to everything" hides the security and availability decisions that live in the network topology.
- **Static-only modeling for elastic systems:** drawing a fixed topology when the deployment is autoscaling. Note the variability (min/max replicas, autoscaler ranges).

## 🔗 Deep Connections

- **Parent Topic:** [[4+1 View Model]]
- **Sibling 4+1 views:** [[Logical View (4+1)]], [[Process View (4+1)]], [[Development View (4+1)]], [[Scenarios (4+1)]]
- **R&W equivalent:** [[Deployment Viewpoint]] (direct 1:1 mapping)
- **Related Concepts:** [[Viewpoint]], [[Container Viewpoint]] (C4's deployable unit, mapped onto nodes in the deployment view), [[Architecture Domains]] (Physical covers the Technology domain)
- **Perspectives that bite here:** [[Security Perspective]], [[Availability and Resilience Perspective]], [[Location Perspective]]
- **Prerequisites:** [[4+1 View Model]], [[Viewpoint]].

## ❓ Active Recall Questions

> [!faq]- What does the Physical View show, and who is it for?
> The system's **deployment topology** — hardware nodes, networks, environments, and the mapping of software (runtime units from the [[Process View (4+1)|Process View]]) onto nodes. It is for the **system engineer**, answering "where does everything run, and how is it networked?" Drawn with UML deployment diagrams.

> [!faq]- How does the Physical View differ from the Process View?
> The **[[Process View (4+1)|Process View]]** shows *what runs concurrently* (processes, threads, replicas) — the runtime units. The **Physical View** shows *where* those runtime units are deployed on hardware (nodes, networks). The Physical View maps the Process View's elements onto nodes. Process = concurrency; Physical = deployment mapping.

> [!faq]- Which R&W viewpoint is the Physical View equivalent to?
> R&W's [[Deployment Viewpoint]] — it is a direct 1:1 mapping (same concern, same audience of system engineers, same "where does it run?" question). Together with the [[Development View (4+1)|Development View]] → [[Development Viewpoint]], these are the cleanest correspondences between 4+1 and R&W.

---

## 📝 Related Notes

- [[4+1 View Model]]
- [[Logical View (4+1)]]
- [[Process View (4+1)]]
- [[Development View (4+1)]]
- [[Scenarios (4+1)]]
- [[Deployment Viewpoint]]
- [[Viewpoint]]
- [[Container Viewpoint]]
- [[Security Perspective]]
- [[Availability and Resilience Perspective]]
