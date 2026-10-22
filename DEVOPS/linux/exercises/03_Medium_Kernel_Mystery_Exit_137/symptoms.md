---
status: archived
artifact_type: exercise_symptoms
exercise: "[[03_Medium_Kernel_Mystery_Exit_137]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — The Process That Keeps Dying With Exit 137

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Marco and his team observed.

**The incident, as Marco reports it:**

- `etl-worker` has been crashing 3–5 times per day for two days. Each crash: exit code **137**.
- **The application log is silent at the moment of death.** No exception. No stack trace. No graceful-shutdown hook firing. No "I am about to exit" line. The process is there processing a batch one second, and gone the next. The next log line is from systemd restarting it ~5 seconds later.
- systemd dutifully restarts it (`Restart=on-failure`), and the *next* batch usually succeeds. So it's not a deterministic data bug — same kind of batch sometimes lives, sometimes dies.
- The deaths **cluster during heavy ingestion windows** (mid-morning and late afternoon, when event volume spikes). But not every heavy window produces a death. Some quiet nights have had one anyway.

**What Marco's team has already done (48 hours of work in the wrong layer):**

- Pored over the application code around the crash timestamps — nothing.
- Tuned JVM flags: heap sizes (`-Xmx`), GC policy (`-XX:+UseG1GC`), metaspace. No change in crash pattern.
- Inspected GC logs — no out-of-memory exception from the JVM, no GC death spiral, no `OutOfMemoryError` stack trace anywhere.
- Bumped dependency versions (the ingestion library, the DB driver). Nothing.
- Added more application-level logging. The new logs confirmed the same thing: silent death, no exception, process simply absent mid-batch.

**Marco's current hypotheses (all application-layer, all wrong):**

- "Maybe an uncaught native exception in a JNI library."
- "Maybe the JVM is hitting some internal limit we haven't found."
- "Maybe the ingestion library has a native-memory leak."

**What Marco has NOT looked at (do not volunteer):**

- Anything outside the application/JVM layer.
- Any system log that isn't the app's own log file.
- The possibility that *something else on the system* killed the process on purpose.

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

**Exit code 137 = 128 + 9 = killed by SIGKILL (signal 9).** The process did not crash — it was *killed* by something with the authority to send SIGKILL. Since SIGKILL can't be caught, handled, or logged by the application (the process is terminated immediately, no shutdown hooks, no chance to write a final log line), the application log is *guaranteed* silent. That silence is itself the clue: a graceful app crash leaves a stack trace; a SIGKILL leaves nothing.

On this box, the killer is the **kernel's OOM (Out-Of-Memory) killer**. Under memory pressure — which lines up exactly with the clustering during heavy ingestion windows — the kernel picks a process to murder to save the system, and a JVM with a large heap (`etl-worker`) is a prime target. The OOM killer sends SIGKILL. Hence 137, silent app log, intermittent timing tied to memory pressure.

### The mechanism (why the JVM didn't report it, and why `OutOfMemoryError` never fired)

- The JVM's `OutOfMemoryError` fires when the JVM's *own* heap allocation fails — i.e. when the JVM tries to grow beyond `-Xmx` and the GC can't free enough. That's an *in-process* OOM, and it produces a stack trace.
- The kernel OOM killer is a completely different beast. It fires when **the system as a whole** (or the cgroup) runs out of memory. The kernel then picks a victim (often the biggest RSS consumer, scored by `oom_score`) and SIGKILLs it. The JVM never sees it coming — no chance to throw `OutOfMemoryError`, no chance to run shutdown hooks, no chance to log. The process is reaped instantly.
- This is why Marco's team found no `OutOfMemoryError`: they were looking for the JVM's OOM. The kernel's OOM leaves its signature *in the kernel logs*, not in the app log.
- JVMs are frequent OOM-killer victims because their committed/ RSS footprint is large and grows predictably. A JVM with `-Xmx4g` plus off-heap (metaspace, thread stacks, direct buffers, JIT code cache) can have a real footprint well above the `-Xmx` headline number.

### Where to look (the redirect away from the app layer)

- **`dmesg -T | grep -i -E 'killed process|out of memory|oom'`** — the kernel ring buffer. The OOM killer logs its decision here. Expect lines like:
  ```
  [timestamp] python[1234] invoked oom-killer: …
  [timestamp] Out of memory: Killed process 5678 (java) total-vm:8GB, anon-rss:5GB, …
  ```
- **`journalctl -k --since '2 days ago' | grep -i oom`** — same data via the journal, persistent across reboots if `Storage=persistent`.
- **`/var/log/kern.log`** (Debian/Ubuntu) or `/var/log/messages` (RHEL) — the on-disk kernel log.
- **`oom_score` and `oom_score_adj`** on the running process (`cat /proc/<pid>/oom_score`) — to see how the kernel ranks `etl-worker` as a future victim.

### A strong hypothesis for the underlying cause

- Memory pressure on the box (or in the cgroup) during heavy ingestion, with `etl-worker`'s JVM as the largest consumer → kernel OOM killer selects it → SIGKILL → exit 137 → silent app log → systemd restarts → next batch usually succeeds because the restart freed the JVM's footprint, relieving pressure temporarily.
- Intermittency is explained: pressure has to cross a threshold, which depends on what else is running and the exact batch size. Not every heavy window crosses it.
- The fix lives at the *system* layer, not the app layer: (a) size `-Xmx` to leave headroom for the OS and other processes; (b) cap the worker's cgroup memory and let the JVM live within it; (c) add swap if appropriate (debated — swap can turn OOM-kill into slow-thrash, a trade-off); (d) move other memory-hungry services off the box; (e) set `oom_score_adj` to *protect* the worker or *sacrifice* a less critical process first.

### Hidden diagnostic traps

1. **Reading 137 as an application crash.** It isn't. `128 + signal` is the POSIX convention; 137 unambiguously means SIGKILL from outside. The single most important decode in this incident.
2. **Treating the silent app log as evidence the app is at fault.** Inverse: a silent death *with a signal exit code* is strong evidence the app had no say in it. The silence points *away* from the app layer.
3. **Looking for `OutOfMemoryError` and concluding "not memory."** Two different OOMs: JVM-internal (throws, logs a stack trace) vs kernel (SIGKILLs, logs to dmesg). Absence of the first says nothing about the second.
4. **Anchoring bias / sunk cost.** 48 hours in the app layer makes the team *want* the answer to be there. The discipline: when the app layer yields nothing, go one layer down — kernel, cgroups, dmesg — before doubling down.
5. **Missing cgroup-level OOM.** On systemd-managed services, the OOM kill may be a *cgroup* OOM (memory.max exceeded) rather than the global kernel OOM. Check `systemctl status etl-worker` (shows `Memory:` and sometimes OOM notes) and `journalctl -u etl-worker`. Same SIGKILL, slightly different log location.

### The corrective mental model

- **"The kernel did this" is a default hypothesis** when (a) the exit code is signal-induced (≥128), and (b) the app log is silent. Don't blame the app for a death it couldn't log.
- The one question that would have redirected on day one: *"What does exit 137 mean, and where does the system record the event that caused it?"* Decoding 137 points at the kernel layer instantly.
- Memory problems have **two faces** — in-process (JVM throws, app can log) and system-level (kernel SIGKILLs, app cannot log). Searching only the app log finds only the first face.

### Rubric — what a strong diagnosis covers

1. **Decodes 137 = 128 + 9 = SIGKILL**, and states that this means the process was killed by an external authority, not that it crashed internally.
2. **Redirects to the kernel/system logs** — names `dmesg`, `/var/log/kern.log` or `journalctl -k`, and explains that SIGKILL is logged by the kernel, not the app.
3. **Names the kernel OOM killer** as the specific subsystem, and explains the JVM-vs-kernel OOM distinction (no `OutOfMemoryError` because the kill is external and immediate).
4. **Explains the silent app log as a positive signal** — SIGKILL can't be caught, so silence is expected; this points *away* from the app layer, not toward it.
5. **Accounts for all the observations**: clustering under memory pressure, intermittency, JVM as prime victim, survival of next batch.
6. **Names the corrective discipline** — when exit is signal-induced and the app is silent, go to the kernel layer *before* doubling down on app debugging; decode the exit code first.

## Part B — end
