---
status: archived
tags: [c4-model, architecture, togaf, capability-framework, reference-model, enterprise-architecture, udemy]
created: 2026-07-14
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: Architecture Reference Models

> [!abstract] Executive Summary (ELI5)
> **Architecture Reference Models** are TOGAF's **pre-packaged, reusable baselines** — generic, vendor-neutral taxonomies that give you a starting point instead of a blank page. TOGAF ships two: the **[[Technical Reference Model|TRM]]** (a taxonomy of the technology platform — the infrastructure layer) and the **[[III-RM|III-RM]]** (a taxonomy of the integrated information infrastructure — the application/data space that supports business). You adopt them as the *generic* end of the [[TOGAF|Enterprise Continuum]] and specialize them to your organization.

> [!note] Part of the TOGAF deep-dive
> These reference models are bundled within the [[Architecture Capability Framework]] (they are reusable assets the practice starts from). This note is the parent; the two models have their own deep dives.

---

## 🎯 The "Why"

- **Problem Solved:** Every enterprise has a technology platform and an application landscape, and if each team invents its own taxonomy from scratch, nothing is comparable. The reference models provide a **shared, generic vocabulary** for the technology and information-infrastructure layers, so teams start from a common baseline and specialize rather than reinvent.
- **Alternatives:** Building a bespoke taxonomy per enterprise. More tailored, but high effort and non-comparable across organizations; you lose the benefit of a common reference.
- **Trade-offs:** The reference models are *generic* — they cover the common case, not your specific industry quirks. You adopt them as a foundation and extend. The payoff is a fast, comparable starting point.

## 🛠️ The Two Reference Models

| Model | Layer it covers | Core idea | Deep dive |
|---|---|---|---|
| **[[Technical Reference Model\|TRM]]** (Technical Reference Model) | Technology / infrastructure | A generic taxonomy of the **platform**: operating systems, networks, databases, security, management services. Vendor-neutral. | [[Technical Reference Model]] |
| **[[III-RM\|III-RM]]** (Integrated Information Infrastructure Reference Model) | Application / Data (the "information infrastructure") | A taxonomy of the **business-supporting application/data services**: how information flows from business operations back to business. Focused on boundaryless information flow. | [[III-RM]] |

> The TRM covers the *bottom* of the stack (the platform everything runs on); the III-RM covers the *middle* (the application/data services that turn business activity into managed information). Together they span the Technology + Application/Data space. They do not cover [[Business Architecture|Business Architecture]] — that has no generic reference model because it is organization-specific.

## ↔️ Where they sit — the Enterprise Continuum

The reference models live at the **generic (left) end** of the [[TOGAF|Enterprise Continuum]]:

```
Generic ←————————————————————————————————→ Specific
Foundation Architecture        Common Systems        Industry        Organization-Specific
   ├─ TRM (technology taxonomy)
   └─ III-RM (information infrastructure)
```

- You **start** an initiative by consulting the TRM/III-RM for reusable categories before designing from scratch.
- You **specialize** them to your organization, moving rightward along the continuum (adding your tech choices, your industry patterns, your org specifics).

## ⚠️ Pitfalls & Anti-patterns

- **Ignoring them and reinventing the taxonomy:** every team builds its own platform categories. Non-comparable, high effort.
- **Treating them as prescriptive technology choices:** they are *taxonomies*, not mandated products. The TRM has a "Database" category; it does not mandate PostgreSQL.
- **Expecting them to cover Business Architecture:** they don't. Business is organization-specific; only Technology and Information Infrastructure have generic reference models.
- **Adopting them verbatim without specialization:** the generic model is a starting point, not an end state. Specialize it to your org along the Enterprise Continuum.

## 🔗 Deep Connections

- **Parent Topic:** [[TOGAF]], [[Architecture Capability Framework]]
- **The two models:** [[Technical Reference Model]], [[III-RM]]
- **Where they live:** the [[TOGAF|Enterprise Continuum]] (generic end)
- **What they accelerate:** [[Architecture Development Method]] (Phase D in particular)
- **Prerequisites:** [[TOGAF]].

## ❓ Active Recall Questions

> [!faq]- What are the TOGAF Architecture Reference Models, and what problem do they solve?
> Two pre-packaged, reusable, vendor-neutral taxonomies — the **[[Technical Reference Model|TRM]]** (technology platform) and the **[[III-RM|III-RM]]** (integrated information infrastructure) — that give teams a shared generic starting point instead of a blank page. They solve the "every team invents its own non-comparable taxonomy" problem.

> [!faq]- Do the reference models cover Business Architecture?
> No. They cover the **Technology** (TRM) and **Application/Data** (III-RM) layers. Business Architecture has no generic reference model because it is organization-specific — capabilities and value streams differ per enterprise.

> [!faq]- Where do the reference models sit in the Enterprise Continuum?
> At the **generic (Foundation) end** — they are the most reusable, vendor-neutral baselines. You adopt them there and specialize rightward (Common Systems → Industry → Organization-Specific) as you add your tech choices and constraints.

---

## 📝 Related Notes

- [[TOGAF]]
- [[Architecture Capability Framework]]
- [[Technical Reference Model]]
- [[III-RM]]
- [[Architecture Development Method]]
