---
created: '2026-08-19'
difficulty: intermediate
mastery: growing
source: System Design Trainer, ch. 1.5 (Su vol.1 ch.1 pp.25-28; SAM synthesis)
status: analyzing
tags:
- system-design
- stateless
- stateful
- sessions
- sam
- scalability
- interview
updated: '2026-08-19'
---

# 🧠 Topic: Stateless vs Stateful — the SAM Lens

> [!abstract] Executive Summary (ELI5)
> HTTP has no memory — every request arrives from a stranger. To "recognize" users, the app adds memory on top: a session dictionary. The whole stateful/stateless choice is **where that dictionary lives**: in each server's RAM (stateful) or in a shared store like Redis (stateless). Where it lives decides whether you can scale, survive failures, and deploy painlessly — the three axes of **SAM**.

---

## 🎯 The "Why"

- **Problem Solved:** "Why do some services scale easily and others are misery?" — because statefulness quietly breaks all three operational axes. Naming them (SAM) turns a vague feeling into a checklist.
- **Origin:** Su's ch.1 shows the sticky-sessions picture and lists the pains; SAM (Scalability / Availability / Modifiability) is a course synthesis structuring those pains into axes.

## 💡 Mental Model & Analogy

- The naive approach: `sessions = {}` — **a dictionary in the server's RAM**. Login writes `sessions["abc123"] = {user, cart}` and hands the browser a cookie with **only the key** (`session_id=abc123`). The cookie is a coat-check ticket, not the coat.
- Add a second server → two dictionaries, never synced → login on server1, next request to server2 → "who are you?" → forced re-login. That's why stateful needs **sticky sessions**, and that's where all three axes start hurting.

## 🛠️ Core Principles

### The SAM axes (stateful vs stateless)

| Axis | A: stateful (dict in server RAM) | B: stateless (dict in shared Redis) |
|---|---|---|
| **S — Scalability** | A new server only takes **new** clients; old ones are pinned. The LB must route by ownership, **not by load** — it stops balancing. Rebalancing = session migration. | New server immediately serves anyone; LB routes by load; no migrations. |
| **A — Availability** | Server dies → its dictionary dies with it → that share of users is logged out (degradation), or worse. | Server dies → LB redirects; sessions live in Redis — users don't notice. A loses **sessions**, B loses only **capacity**. |
| **M — Modifiability** | Deploy = restart process = RAM wiped = that server's users logged out. Options: rolling (portioned logouts), maintenance window (everyone at once), or session drain/migration (complex). | Deploy = restart doesn't touch sessions (they're in Redis); rolling is invisible to users. |

**Key mechanic for M:** deploying a new version changes **code**, not sessions — but a restart clears RAM. In stateful, code and state share the same RAM, so they die together; in stateless they are separated, so they die independently.

### The stateless test (one sentence)
Kill any server at any moment — any in-flight request can be replayed on another server and the user keeps their session. If true → stateless.

### Cache vs state — the loss test
- **Cache**: tolerable to lose — there's a source of truth behind it; a miss just recomputes slower. May live in server RAM even in stateless architectures.
- **State**: no source of truth behind it — losing it means re-login, lost carts. Must move to a shared store.

### Why Redis and not the DB for sessions?
The session lookup is overhead on **every** request: at peak 2000 rps that's 2000+ extra reads/sec on your most precious resource. Redis = RAM (~0.1–1ms); a DB = disk + B-tree + transaction machinery (~1–10ms+). Every extra ms sits in W, and by Little's Law inflates required capacity ([[Little's Law and Capacity Planning]]). Sessions are ephemeral key-value data with TTL — no joins, no transactions — the textbook case for "extremely low latency" stores (Su's own NoSQL criterion). Honest cost: Redis is +1 component and a new failure point — for a small app, DB sessions are simpler. Trade-off, not dogma.

## ⚠️ Pitfalls

- "Stateless = no data" — wrong: the dictionary still exists, it just **moved**.
- Thinking cookies hold session data — they hold the **key** only.
- Deploying a stateful system like it's stateless (mass logouts).
- Keeping stateful tiers because "we'll scale it later" — DB is inherently stateful; that's exactly why data scaling (replication/sharding) is its own hard topic (Module 4 of the trainer).

## 🔗 Related

- [[Scaling_Session_Management]] — the three implementation families (sticky / shared store / JWT) with pros-cons; JWT extends beyond the trainer's ch.1.5.
- [[Scaling Evolution - from Single Server to Stateless Web Tier]] — the architecture ladder this fits into.
- [[HTTP_Statelessness]] — the protocol-level fact that makes all of this necessary.

## ✅ Self-check

1. One-sentence stateless test? → Kill any server; replay in-flight requests elsewhere; sessions survive.
2. What breaks in stateful? → SAM: can't rebalance (S), lose sessions on failure (A), deploy logs users out (M).
3. Why is deploying stateful painful? → Restart clears RAM where sessions live; code and state die together.
4. Sessions in Redis vs DB? → Per-request lookup cost (RAM vs disk), TTL, key-value shape; +1 component is the price.