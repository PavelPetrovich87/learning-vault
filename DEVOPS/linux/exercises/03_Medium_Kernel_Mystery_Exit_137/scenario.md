---
status: archived
exercise_type: incident_diagnosis
difficulty: intermediate
pattern: kernel
source_concepts:
  - "[[Unix Like Kernel]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The Process That Keeps Dying With Exit 137

## Type
**Incident diagnosis** — a production background worker keeps getting killed. The application logs show no error; the team has been chasing their own code for two days. You are the senior who gets called in to look *outside* the application. Medium ambiguity: the symptom (exit 137) could mean several things, and the team has already anchored on the wrong layer.

## Difficulty
**Medium.** Single concept (the kernel's role in process lifecycle), but the symptom is ambiguous: 137 *looks* like an application crash to anyone who doesn't know the convention `128 + signal`. The team has been debugging the wrong layer for 48 hours. A strong answer identifies where to look (kernel logs, not app code), names the specific kernel subsystem, and explains why the app log is silent.

## The Situation

**Where:** Acme WebOps — a SaaS analytics product. You are the senior SRE. **Marco** leads the data pipeline team.

**The system:** A worker process (`etl-worker`) running on a single beefy VM. It ingests batches of analytics events, transforms them, and writes to a warehouse. Java service, runs under systemd, nothing exotic. It had been stable for months.

**The stakes:** Real but bounded. Every time the worker dies, a batch is lost and must be replayed. Customers see stale dashboards for ~20 minutes. It's been dying 3–5 times a day for two days. Marco's team has been pulling late nights "fixing" it.

**The twist:** Marco's team has been **debugging the wrong thing**. They've spent two days in the application code, the JVM settings, the garbage collection logs, the dependency versions — because the symptom *looks* like an application problem. It isn't. The fault is one layer down, and the application log has nothing to say about it.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- `etl-worker` exits with **exit code 137** every time. Same code, every death.
- The **application log shows nothing** at the moment of death — no exception, no stack trace, no graceful shutdown message. The process is simply *gone* mid-batch.
- Restart is automatic (systemd brings it back), and the next batch usually succeeds — so it's intermittent, not deterministic per batch.
- Marco's team has spent ~48 hours in the application layer (code, JVM, GC, deps) and found nothing. They're frustrated and a little defensive.
- The deaths seem to cluster during heavy ingestion windows — but not every heavy window triggers one.

## The Ask

Walk Marco through producing:

1. **The meaning of exit 137.** Decode it. What does that number actually signify, and which layer of the system produced it?
2. **Where to look next** — and an explicit redirect away from the application layer. If the app log is silent, what *other* log on this machine would record the event that corresponds to a 137 exit? Name the specific place.
3. **A hypothesis** for the *underlying cause* that explains: silent app log, clustering during heavy windows, intermittent timing, JVM specifically.
4. **The corrective mental model** — why a whole engineering team spent two days in the wrong layer, and what single question would have pointed them at the right layer on day one.

## Related Concept Notes
- [[Unix Like Kernel]] — the kernel as the arbiter of process lifecycle, memory pressure, and the OOM killer; why "the kernel did this" is a default hypothesis when the app is silent and the exit looks signal-induced.

---

*Last updated: 2026-07-17*
