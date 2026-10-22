---
status: archived
artifact_type: exercise_symptoms
exercise: "[[M2_Medium_Slow_Load_High_Cpu_Idle]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — The Site Is Slow — But `top` Says CPU Is 80% Idle

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Jordan observed.

**The incident, as Jordan reports it:**

- **~90 minutes ago** — The web app's p99 latency started climbing. Dashboards show it's now ~3× the normal baseline. Customers are noticing.
- Jordan pulls up `top` on `web-app-07`:
  ```
  load average: 12.05, 11.80, 9.40
  ```
  (1-min, 5-min, 15-min — all high and rising.)
  And the CPU summary line:
  ```
  %Cpu(s):  3.0 us,  1.5 sy,  0.0 ni, 80.2 id, 14.8 wa,  0.0 hi,  0.5 si,  0.0 st
  ```
  CPU is **80% idle**. There's also a small column called `wa` at 14.8% — Jordan hasn't registered what that means.
- Jordan has been paging through dashboards looking at CPU utilization graphs, convinced the answer is there because "load average is 12 on a 4-core box — that's 3× oversubscribed, the CPU must be the problem." The high idle is, in Jordan's words, "confusing, probably a dashboards lag."

**What Jordan has checked:**

- CPU utilization per process in `top` — no single process is eating CPU. The web app processes are at single-digit percent.
- Memory — plenty free, no swapping.
- Network to the database host — round-trips normal, bandwidth normal.
- The `webapp` service status — `systemctl status webapp` shows `active (running)`, looks healthy from this angle.

**What Jordan has NOT checked (do not volunteer):**

- Disk I/O — `iostat`, `iotop`. Hasn't occurred to them.
- The `wa` column in the CPU summary (I/O wait).
- The *other* service on this box: `log-shipper`. Jordan only looked at `webapp`.
- `/var/log/journal` size — the journal is on the same disk as the app's local cache.

**What Jordan reports feeling:**

- "Load is 12, so the box is overloaded. CPU is 80% idle, so it isn't. These can't both be true. I've been staring at CPU graphs for an hour and I'm going in circles."

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause — and there are two layered problems

**Problem 1 (the primary): the web app is I/O-bound, not CPU-bound.** Load average counts processes that are *runnable* OR *in uninterruptible sleep (D-state, waiting on I/O)*. A high load average with idle CPU means many processes are stuck in D-state — waiting on I/O that isn't completing fast enough. They count toward "load" (they exist, they want to run, they're just blocked) but they don't burn CPU (they can't — they're asleep in the kernel waiting for the disk). Hence: load 12, CPU 80% idle. That's the signature of an I/O-bound system, *not* a CPU-bound one.

The `wa` (I/O wait) column in the CPU summary — 14.8% — is the confirmation Jordan overlooked. `wa` is the percentage of time the CPU spent idle *because it was waiting for outstanding I/O to complete*. 14.8% wa + 80% id means even when the CPU has nothing to do, a meaningful chunk of that "idle" is "idle because we're stalled on disk." On an I/O-bound system under heavier pressure, `wa` can climb much higher.

**Problem 2 (the aggravator): a crash-looping service is generating log I/O that competes with the web app.** The `log-shipper` service on this box is in a crash-loop. Each crash generates a stack trace and restart cycle in the journal; the journal lives on the same disk as the web app's local cache. The crash-loop is producing a continuous stream of disk writes (the journal filling with crash/restart entries), which competes with the web app's reads/writes — aggravating the I/O bottleneck from Problem 1. Jordan never looked at `log-shipper` because they were focused on `webapp`.

The crash-loop itself is caused by a classic systemd gotcha: `log-shipper.service` has `Restart=always` with no `RestartSec=` and no `StartLimitBurst=`/`StartLimitIntervalSec=`. So when the shipper hits a transient failure (a misconfigured downstream, a bad config flag), it crashes, systemd restarts it *immediately*, it crashes again — a tight loop that pegs the disk with log writes and burns CPU on restart cycles. This is the "runaway restart" anti-pattern from the Services concept note.

### The mechanism (why load-average vs CPU-idle is the smoking gun)

- **Load average** = exponential moving average of (number of runnable processes + number of processes in uninterruptible sleep). On a 4-core box, a sustained load of 12 means roughly 12 processes are simultaneously wanting to make progress. If CPU is 80% idle, those 12 processes are *not* waiting for CPU (they'd be running). They must be waiting for something else — and D-state (uninterruptible sleep on I/O) is by far the most common "something else."
- **Why D-state counts toward load but not CPU.** A process in D-state is asleep inside a kernel I/O syscall. It can't be scheduled (so it doesn't burn CPU), but it exists in the runqueue accounting as "wanting to make progress" (so it counts toward load). High load + low CPU is the textbook signature of "many processes blocked on I/O."
- **The `wa` column** formalizes it: it's the fraction of time the CPU was idle *with outstanding I/O*. A high `wa` says "the CPU would have been doing work, but processes were stalled waiting on I/O." It's the bridge between "load is high" and "CPU is idle."
- **The crash-loop aggravates it** by adding I/O pressure: every restart cycle writes journal entries, every stack trace hits disk. On a shared disk, the web app's I/O gets slower, more of its processes stall in D-state, load climbs further. A positive-feedback loop.

### The diagnostic chain

From the misleading metric to the root cause:

1. **Decode the contradiction** — load 12 + CPU 80% idle ⇒ I/O-bound, not CPU-bound. Read the `wa` column (14.8% ⇒ significant I/O wait) to confirm.
2. **`iostat -xz 1`** — per-device I/O stats, refreshed each second. Look for a device with `%util` near 100 (saturated disk) and high `await` (latency per I/O). This pinpoints *which* disk is the bottleneck. (`-x` extended, `-z` skips idle devices.)
3. **`iotop -oP`** (or `pidstat -d 1`) — per-*process* I/O. Shows which processes are reading/writing the most. Expect to see `webapp` processes waiting on reads, and *also* the journal-related writes from the crash-looping `log-shipper`.
4. **`vmstat 1`** — the `wa` column again (CPU time in I/O wait), plus `bo`/`bi` (blocks in/out) — corroborates I/O pressure at the system level.
5. **`systemctl status log-shipper`** — reveals the crash-loop: the status output shows recent restart activity; `Active: activating (auto-restart)` or rapidly-flipping active/inactive; the log section is full of `Started Log Shipper…` / `Stopped Log Shipper…` / stack traces repeating every second or two. `systemctl status webapp` alone (which Jordan checked) misses this — you have to check *all* services, or `systemctl --failed` / `systemctl list-units --state=activating`.
6. **`journalctl -u log-shipper --since '15 min ago' | tail`** — confirms the crash-loop rate and what the shipper is dying on (config error, downstream unreachable, etc.).

### The fix for both

**Problem 1 — I/O-bound web app:**
- *Immediate*: relieve the disk pressure. If the local cache can be moved to faster storage (or a different spindle), do it. If the cache can be shrunk or warmed differently, do that. Check that the cache isn't thrashing (evicting and re-reading the same entries) — a bad cache policy can look exactly like I/O-bound.
- *Diagnostic*: identify *what* the web app is waiting on. Is it the local cache disk, or the DB round-trips? `iostat`/`iotop` distinguish local-disk I/O from network latency.
- *Structural*: separate the journal (and logs generally) from the app's working set onto different storage. Same disk = shared bottleneck, which is exactly how Problem 2 aggravated Problem 1.

**Problem 2 — crash-looping log-shipper:**
- *Immediate*: stop the crash-loop. `sudo systemctl stop log-shipper` halts the loop and the journal-write pressure *now*. The web app's I/O should improve measurably within seconds — itself a confirmation that the aggravator was real.
- *Then*: read the journal to find out *what* log-shipper is dying on. `journalctl -u log-shipper --since '30 min ago'` — it's probably a transient downstream failure (log aggregator unreachable), a bad config flag, or an OOM (shipper memory leak). Fix the underlying cause.
- *Structural*: fix the unit file. `Restart=always` alone is a footgun — pair it with:
  - `RestartSec=5s` (or higher) — a delay between restarts so a crash-loop can't spin tightly.
  - `StartLimitBurst=5` and `StartLimitIntervalSec=60` — after 5 restarts in 60s, systemd gives up and marks the unit `failed` instead of looping forever. This converts a runaway loop into a single, visible, alertable failure.
  - `Restart=on-failure` (instead of `always`) if clean exits shouldn't restart.
  - Use `systemctl edit log-shipper` to add these as a drop-in (per the Services exercise — don't edit the package file).
- *Verify*: after stopping log-shipper, watch `iostat -xz 1` — the saturated disk's `%util` should drop and the web app's latency should recover. That's the end-to-end confirmation that you found *both* problems.

### Hidden diagnostic traps

1. **Treating "high load" as "CPU-bound."** The most common misread. Load counts runnable *and* D-state processes. High load + low CPU is I/O-bound, by definition. Jordan spent an hour on CPU graphs because of this assumption.
2. **Overlooking the `wa` column.** It's right there in `top`'s CPU summary and it directly answers "is the idle CPU actually I/O wait?" 14.8% wa is a strong signal that Jordan stepped past.
3. **Checking only the obvious service.** Jordan checked `webapp` (the customer-facing one) and skipped `log-shipper`. On any multi-service box, *all* services matter — a sidecar's pathology can tank the primary through shared resources. `systemctl status` every service, or `systemctl --failed` / `list-units --state=activating` to surface the sick ones.
4. **Missing the shared-disk coupling.** The journal and the app cache on the same disk is the mechanism by which Problem 2 aggravated Problem 1. Without that coupling, the crash-loop would be noisy but not fatal. Coupling is the multiplier.
5. **Treating the crash-loop as the primary cause.** It isn't — the web app was already trending I/O-bound. The crash-loop *aggravated* it. A candidate who fixes only the crash-loop and declares victory will see the latency partially recover but not fully, because Problem 1 is still there. Fix both.
6. **`Restart=always` without limits.** The unit-file root cause of the crash-loop being *tight* rather than *occasional*. Without `RestartSec` and `StartLimitBurst`, any failure becomes a tight spin.

### The generalizable model

- **Load average = runnable + uninterruptible-sleep processes.** High load + idle CPU ⇒ I/O-bound. High load + low `%id` ⇒ CPU-bound. The two metrics together diagnose the bottleneck class. Always read them as a pair, never either alone.
- **The `wa` column is the bridge** between "load is high" and "CPU is idle." A high `wa` says the idle CPU is idle *because* of I/O stall.
- **`iostat`/`iotop`/`pidstat -d`** are the I/O equivalents of `top` — learn them as a cluster. `top` alone is a CPU-and-memory tool; it cannot fully diagnose I/O.
- **On a multi-service box, audit *all* services**, not just the one the customer sees. A crash-looping sidecar is invisible in the customer-facing dashboard and devastating on shared disk.
- **`Restart=always` is a footgun without rate limits.** Always pair with `RestartSec` and `StartLimitBurst`. The drop-in override (`systemctl edit`) is the right place.
- **Shared resources couple failures.** Journal and app cache on one disk makes any disk-heavy failure (crash-loop, log flood) into an app outage. Separate critical I/O paths.

### Rubric — what a strong diagnosis covers

1. **Decodes load-vs-CPU** — load counts runnable + D-state; high load + idle CPU ⇒ I/O-bound, not CPU-bound. Names the `wa` column as the confirmation Jordan overlooked.
2. **Names the I/O diagnostic chain** — `iostat -xz 1` (per-device `%util`/`await`), `iotop -oP` or `pidstat -d 1` (per-process), `vmstat 1` (system-level `wa`/`bo`/`bi`).
3. **Identifies the *second*, hidden problem** — the crash-looping `log-shipper`; checks all services, not just the obvious one; reads `systemctl status log-shipper` and `journalctl -u log-shipper`.
4. **Explains the shared-disk coupling** — journal + app cache on the same disk is how Problem 2 aggravated Problem 1; separate critical I/O paths.
5. **Names the `Restart=always` gotcha** — without `RestartSec`/`StartLimitBurst`, a transient failure becomes a tight crash-loop; fix via `systemctl edit` drop-in.
6. **Prescribes fixes for both problems** and the verification (stop log-shipper → watch `iostat` recover); resists "fix only the crash-loop and declare victory."

## Part B — end
