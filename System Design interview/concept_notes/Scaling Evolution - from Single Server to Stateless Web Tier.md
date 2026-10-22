---
created: '2026-08-19'
difficulty: intermediate
mastery: growing
source: System Design Trainer, ch. 1.5 (Su vol.1 ch.1 pp.9-14, 25-28)
status: analyzing
tags:
- system-design
- scalability
- load-balancer
- architecture
- interview
- evolution
updated: '2026-08-19'
---

# 🧠 Topic: Scaling Evolution — from Single Server to Stateless Web Tier

> [!abstract] Executive Summary (ELI5)
> Su's ch.1 is a four-step ladder: one server → split web from DB → add a load balancer with a second web server → make the web tier stateless and replicate the DB. The interview pattern embedded in it: **start simple, add one component at a time, and only in answer to a specific failure**. For every rung, know what it solves — and what still kills the system.

---

## 🎯 The "Why"

- **Problem Solved:** Juniors draw "Kubernetes with 12 microservices" on the whiteboard; seniors draw an evolution and name the failure that forces each addition. The ladder is the skeleton of any "scale from zero" interview answer.
- Each step is bought **in response to a concrete pain**, never "just in case" — the anti-overengineering discipline.

## 🛠️ Core Principles

### The ladder (Su, ch.1)

| Stage | Adds | Solves | Still kills the system |
|---|---|---|---|
| **1. Single server** (fig 1.1) | — (starting point, max simplicity) | — | Failure of **any** component; vertical scaling is the only lever |
| **2. Web/DB split** (fig 1.3) | Separate machines for web tier and data tier | **Independent scaling** of web and DB (not fault tolerance!) | Web server OR DB failure still takes everything down |
| **3. Load balancer + 2nd web server** (fig 1.4) | LB in front; servers get private IPs (clients stop touching servers directly) | **First fault tolerance**: one web server can die, site lives | DB is single → its failure kills the site; the bottleneck moved to the DB. (Own observation, valid: the LB itself is a new SPOF — real systems run LB pairs with failover) |
| **4. Stateless web + DB replication** (fig 1.14) | Session store moved out (Redis/NoSQL); DB gets replicas | Web-server death loses no sessions; DB-node death is survived by replicas | Multi-DC/geoDNS (beyond this chapter); **replica consistency**; the shared session store itself (Redis down = everyone logged out — degradation, not death) |

**Fault tolerance appears only where there is more than one of a component.** Stage 2's split does NOT add fault tolerance — only independent scaling. That's the single most-missed point.

### Stage-3 subtleties
- LB gives clients one public IP; web servers live on private IPs (security + decoupling).
- The LB evens traffic **and** hides dead servers.
- Session stickiness (if state is still in server RAM) turns the LB into a router-by-ownership — see [[Stateless vs Stateful - the SAM Lens]].

### Stage-4 subtleties
- Autoscaling (add/remove servers automatically by load) is a **consequence** of statelessness, not a separate tech: replaceable servers make add/remove routine, hence automatable. The chain: static ×2 margin is bounded → need autoscaling → autoscaling needs stateless.
- Failure **severity** differs: Redis death = re-login (degradation); unreplicated DB death = no data (death). Grade failures, don't just count them.
- The stateful tier remains: the DB. That's why data scaling (replication, sharding) is the actual hard problem — trainer Module 4.

### Interview pattern
"Start with one server. What breaks first at 10×? Add exactly one thing. Repeat." — evolution beats big-bang architecture; every element must answer a named requirement (anti-overengineering, Su's red flag #1).

## ⚠️ Pitfalls

- Listing components without naming the failure each stage survives / still dies from.
- Thinking the web/DB split adds fault tolerance (it adds independent scaling only).
- Forgetting the LB is itself a single point of failure in the naive picture.
- Jumping straight to microservices + distributed DB "from day one" — Kleppmann: keep the DB single-node until cost or availability forces otherwise.

## 🔗 Related

- [[Stateless vs Stateful - the SAM Lens]] — stage 4's core move.
- [[Scaling Strategies]] — vertical/horizontal/diagonal vocabulary behind the ladder.
- [[Load_Balancers]] / [[Load_Balancing_Introduction]] — LB algorithms (trainer ch.2.3 goes deeper).

## ✅ Self-check

1. When does the system first survive a component failure? → Stage 3 (LB + second web server) — and only a web-server failure; DB still fatal.
2. What does the web/DB split actually buy? → Independent scaling, not fault tolerance.
3. Why is autoscaling a consequence of stateless? → Replaceable servers → add/remove is routine → automatable; sticky state makes it risky.
4. What still kills the system at stage 4? → Redis (degradation), replica divergence, multi-DC absence.