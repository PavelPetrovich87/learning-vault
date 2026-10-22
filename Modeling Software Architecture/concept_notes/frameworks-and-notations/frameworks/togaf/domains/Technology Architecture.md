---
status: archived
tags: [c4-model, architecture, togaf, domains, technology-architecture, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Technology Architecture

> [!abstract] Executive Summary (ELI5)
> **Technology Architecture** is the bottommost TOGAF domain — it describes **the infrastructure that hosts the applications**: platforms, networks, runtimes, compute, storage, and deployment environments. It is the *physical/virtual foundation* of the stack: [[Business Architecture|Business]] capabilities are delivered by [[Application Architecture|Applications]] that process [[Data Architecture|Data]], all running on Technology. It is described last in the ADM's domain phases (Phase D) and is the domain most directly served by a TOGAF reference model — the [[Technical Reference Model|TRM]].

> [!note] Part of the TOGAF deep-dive
> One of the four BDAT domains in [[TOGAF Architecture Domains]]. It is the **Technology** slice — the "infrastructure that hosts apps" layer (bottom of the BDAT stack).

---

## 🎯 The "Why"

- **Problem Solved:** Applications must run *somewhere*, and at enterprise scale the infrastructure (cloud regions, networks, environments, platforms) is itself complex and expensive. Technology Architecture makes the infrastructure explicit — what runs where, on what platform, in which environment — so deployment, cost, security, and availability can be reasoned about.
- **Alternatives:** Treating infrastructure as an afterthought left to ops. Result: unknown deployment topology, security gaps, and cost surprises.

## 🛠️ Core Concerns & Artifacts

| Concern | What it captures | Example TOGAF artifacts |
|---|---|---|
| **Nodes / platforms** | Where compute/storage runs | Computing Hardware diagram, Technology Standards catalog |
| **Networks** | How nodes are connected | Networked Computing/Hardware diagram |
| **Platform services** | What the platform provides (DB, messaging, identity…) | Platform Service diagram (maps to [[Technical Reference Model\|TRM]] categories) |
| **Environments** | dev/test/stage/prod and their mapping | Environment maps, deployment views |
| **Mapping apps → nodes** | Which app is deployed onto which node | (the app↔node mapping, often a matrix) |

## ↔️ Where it sits in TOGAF

- **ADM phase:** Phase D (Information Systems Architectures) — alongside Application/Data, but it is the *lowest* layer. See [[Architecture Development Method]].
- **Reference model:** the [[Technical Reference Model|TRM]] — a generic, vendor-neutral taxonomy of the technology platform (OS, network, data management, security, etc.). This is the domain the TRM directly serves.
- **Metamodel entities:** Node, Technology Component, Platform Service, Network, Artifact. See [[Architecture Metamodel]].
- **Maps to course viewpoints:** [[Deployment Viewpoint]] (where things run) and [[Concurrency Viewpoint]] (runtime instances). See [[Architecture Domains]].

> This is where deployment, security, and availability concerns land most concretely. Apply the [[Security Perspective]] and [[Availability and Resilience Perspective]] here to ask "is this deployment secure / resilient?"

## ⚠️ Pitfalls & Anti-patterns

- **Conflating Technology with Application:** the platform (Technology) vs. the apps running on it (Application). Keep them in separate domains.
- **No reference to the TRM:** reinventing platform categories that the [[Technical Reference Model|TRM]] already provides. Map your tech onto the TRM taxonomy.
- **Ignoring environments:** treating "production" as the only environment. Dev/test/stage mappings matter for deployment and risk.
- **No app↔node mapping:** Technology described in isolation from what runs on it loses the traceability chain up to the business.

## 🔗 Deep Connections

- **Parent Topic:** [[TOGAF Architecture Domains]], [[TOGAF]]
- **Sibling domains:** [[Business Architecture]], [[Data Architecture]], [[Application Architecture]]
- **ADM:** [[Architecture Development Method]] (Phase D)
- **Reference model:** [[Technical Reference Model]] (the TRM directly serves this domain)
- **Course mapping:** [[Deployment Viewpoint]], [[Concurrency Viewpoint]], [[Architecture Domains]]
- **Perspectives that bite here:** [[Security Perspective]], [[Availability and Resilience Perspective]], [[Location Perspective]]
- **Prerequisites:** [[TOGAF]], [[TOGAF Architecture Domains]].

## ❓ Active Recall Questions

> [!faq]- What is Technology Architecture, and where does it sit in the BDAT stack?
> It is the **bottommost** domain — the infrastructure (platforms, networks, runtimes, environments) that hosts the applications. Business capabilities → Applications → Data → Technology: each layer runs on the one below. It is described in Phase D of the ADM.

> [!faq]- Which reference model directly serves the Technology domain?
> The [[Technical Reference Model|TRM]] — a generic, vendor-neutral taxonomy of the technology platform (OS, network, data management, security, etc.). You map your specific products onto its categories. See [[Architecture Reference Models]].

> [!faq]- Which course viewpoints map to the Technology domain?
> [[Deployment Viewpoint]] (where things run) and [[Concurrency Viewpoint]] (runtime instances). Security, availability, and location concerns land here most concretely. See [[Architecture Domains]].

---

## 📝 Related Notes

- [[TOGAF]]
- [[TOGAF Architecture Domains]]
- [[Business Architecture]]
- [[Data Architecture]]
- [[Application Architecture]]
- [[Architecture Development Method]]
- [[Technical Reference Model]]
- [[Deployment Viewpoint]]
- [[Concurrency Viewpoint]]
- [[Security Perspective]]
- [[Availability and Resilience Perspective]]
