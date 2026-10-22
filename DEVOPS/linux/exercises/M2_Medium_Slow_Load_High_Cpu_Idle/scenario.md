---
status: archived
exercise_type: incident_diagnosis
difficulty: intermediate
pattern: integration_processes_services
source_concepts:
  - "[[Linux Processes]]"
  - "[[Linux Services and Systemd]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The Site Is Slow — But `top` Says CPU Is 80% Idle

## Type
**Integration diagnosis (Medium)** — two concepts woven together (process states/load average + systemd services), and the symptom is genuinely ambiguous. The team is chasing the wrong resource (CPU) because the headline metric is misleading. You are the senior who reads load-average-vs-CPU correctly and spots the *second* problem hiding behind the first. The candidate must hold two systems in mind at once.

## Difficulty
**Medium.** Two concepts, ambiguous symptom, *and* a layered incident: there's a primary cause (the app is I/O-bound, not CPU-bound — the high load + low CPU combination) *and* a secondary cause (a systemd-managed service is crash-looping, generating log I/O that's making the primary worse). The candidate has to (a) decode the misleading metric, (b) identify the primary bottleneck, (c) spot the secondary aggravator, and (d) prescribe fixes for both. The trap is treating "slow" as "CPU" because that's what people assume.

## The Situation

**Where:** Acme WebOps — the production web tier again. You are the senior SRE. **Jordan** is the on-call engineer.

**The system:** `web-app-07`, an Ubuntu server running the main web app under systemd (a service called `webapp`). It also runs a small sidecar log-shipper, a service called `log-shipper`. Both are managed by systemd. The web app reads and writes to a database on another host and to a local disk cache.

**The stakes:** Customer-visible slowness. The web app's p99 latency has tripled. Jordan has been paging through dashboards for an hour and is fixated on CPU — except CPU is mostly idle, which is *more* confusing, not less.

**The contradiction that drives the whole exercise:** `top` shows load average of ~12.0 on a 4-core box — that's "3× oversubscribed," which *feels* like the CPU should be pegged. But `%Cpu(s) … id` shows 80% idle. To Jordan, this is contradictory: "load is high, so the box is overloaded; but CPU is idle, so it isn't. Which is it?" The answer is neither "overloaded" in the way Jordan thinks nor "fine" — it's a specific kind of bottleneck that the load-average number hints at and the CPU-idle number confirms.

**The hidden second problem:** There's a crash-looping service on this box. It's not the cause of the slowness, but it's an aggravator Jordan hasn't noticed, and untangling the two is part of the work.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- Web app p99 latency has tripled in the last ~90 minutes.
- `uptime`/`top` shows load average ~12.0 on a 4-core box.
- `top`'s CPU summary shows ~80% idle (`%Cpu(s): … 80.0 id`).
- Jordan has been investigating CPU because load is high; the high idle confuses them.
- The box runs two services under systemd: `webapp` (the main app) and `log-shipper` (a sidecar).
- Disk I/O is not something Jordan has looked at yet.
- `/var/log/journal` is on the same disk as the app's cache.

## The Ask

Walk Jordan through producing:

1. **Decode the contradiction.** Load average 12.0, CPU 80% idle. What does load average *actually* count? What does high load + idle CPU tell you — what *kind* of bottleneck is this? Be precise.
2. **The single metric in `top` that confirms the hypothesis.** What column (beyond `%id`) should Jordan look at to validate the bottleneck class, and what would a high value there confirm?
3. **The diagnostic chain** — from "load is high, CPU is idle" → which resource is saturated → which processes are waiting on it. Name the tools (`iostat`, `iotop`, `pidstat`, `vmstat`) and what each shows.
4. **The *second* problem** — there's an aggravator on this box Jordan hasn't noticed. How would a careful pass through `systemctl status` (for all services, not just the obvious one) reveal it? What does a crash-loop look like in systemd output?
5. **The fix for both** — (a) the primary bottleneck (how to relieve the I/O pressure the web app is suffering), and (b) the secondary aggravator (how to stop a crash-looping service from making the primary worse, and what *caused* it to crash-loop in the first place — the systemd `Restart=` gotcha).

## Related Concept Notes
- [[Linux Processes]] — what load average actually counts (runnable + uninterruptible-sleep processes), the load-vs-CPU mental model, D-state and why it counts toward load but not CPU busy, `iostat`/`iotop`.
- [[Linux Services and Systemd]] — `systemctl status`, `Restart=always` without `RestartSec=`/`StartLimitBurst=`, the runaway-restart gotcha, why a crash-looping service fills the journal.

---

*Last updated: 2026-07-17*
