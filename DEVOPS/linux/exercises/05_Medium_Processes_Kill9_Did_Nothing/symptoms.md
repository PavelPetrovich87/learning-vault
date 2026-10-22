---
status: archived
artifact_type: exercise_symptoms
exercise: "[[05_Medium_Processes_Kill9_Did_Nothing]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — `kill -9` Did Nothing — The Process Won't Die

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Ravi observed.

**Timeline:**

- **14:02** — Ravi kicked off `git clone` of a very large internal repo. The clone is going over the network (the repo is on an NFS-mounted share).
- **14:09** — The clone stops producing output. Cursor blinks. Ravi waits.
- **14:14** — Ravi gives up waiting, opens another terminal, finds the PID via `ps aux | grep git`, and sends `kill <pid>` (SIGTERM). Nothing happens.
- **14:15** — Ravi escalates: `kill -9 <pid>` (SIGKILL). Nothing happens. The process is still in `ps`.
- **14:16–14:20** — Ravi sends `kill -9 <pid>` **five more times**, each a little more forcefully. Each time `kill` itself reports no error (it accepts the command, the signal is sent). Each time, the process is unchanged in `ps`.
- **14:21** — Ravi is now in the team chat asking for permission to reboot the box "because kill -9 isn't working, the kernel must be broken."

**What Ravi has verified:**

- The PID is correct — he's killing the right number (`ps` confirms the git process still owns that PID).
- He has the authority to send the signal — the process runs as his own user.
- `kill` doesn't complain — it returns 0; the signals are being accepted/delivered as far as userspace can tell.
- The process entry in `ps` looks the same after each kill as before.

**What Ravi has NOT looked at (do not volunteer):**

- The `STAT` column of `ps` (or the state field of `top`/`/proc/<pid>/status`) — the single letter that tells you *why* SIGKILL is ineffective.
- The process's *parent* — the PID that spawned it.
- Anything about the underlying I/O the process was doing when it hung.

**What Ravi reports feeling:**

- Frustrated and a little betrayed by the OS. "`kill -9` is supposed to be the kill-it-with-fire option. I used it six times. If the strongest signal doesn't work, what else is there except rebooting?"

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause (a fork with two branches)

`kill -9` (SIGKILL) "did nothing" because the process is in one of **two states where SIGKILL is defined to have no immediate effect**. Ravi hasn't diagnosed *which* — and they have opposite fixes.

- **State `Z` — zombie (defunct).** The process has *already exited*. It is dead; the kernel is just holding its entry in the process table until its parent calls `wait()` to collect the exit status. You cannot kill a zombie — there's nothing left to kill. SIGKILL is a no-op because the process is already gone. The fix is to make the parent reap it (or kill the parent, which re-parents the zombie to PID 1, which reaps it).
- **State `D` — uninterruptible sleep.** The process is blocked inside a kernel-space I/O operation (typically disk or network/NFS) that has not returned. The kernel *cannot* deliver SIGKILL to a process in D-state — the signal is queued and only acted on once the I/O returns (which may be never, if the underlying I/O is genuinely hung). You cannot kill a D-state process — not with `kill`, not with `kill -9`, not with anything short of the I/O completing or the host rebooting. The fix is to *diagnose the I/O* (a hung NFS mount, a failing disk, a stuck kernel module), not to reach for kill.

Given Ravi's trigger (large `git clone` over NFS that hung), **D-state is the far more likely culprit** — the process is wedged inside an NFS syscall that hasn't returned. But the discipline is to *check*, not assume.

### The mechanism (why SIGKILL is defined to fail in both states)

- **SIGKILL is "can't be caught, blocked, or ignored" — but it still has to be *delivered*.** Delivery happens when the target process is scheduled and returns to user space (or hits a syscall boundary it can safely abort at). Two states prevent delivery:
  - **Zombie:** there is no user space left to return to. The process has exited. SIGKILL has nothing to act on.
  - **D-state:** the process is stuck *inside* the kernel on an uninterruptible I/O. It cannot be scheduled, and the kernel won't abort the I/O (the "uninterruptible" part) because aborting mid-operation could corrupt data or leave hardware in a bad state. So SIGKILL is queued and waits.
- This is why "more SIGKILL" is a category error. The signal isn't being ignored out of stubbornness; the kernel literally *cannot* act on it until the underlying condition changes. Repeating the command is identical to doing nothing.
- This is also why `kill -9` is **not** the "strongest possible" tool in the sense Ravi assumes. It's the strongest *userspace-deliverable* signal — but userspace delivery has prerequisites, and a zombie or D-state process fails them.

### The single diagnostic

Look at the **`STAT`** column of `ps` (e.g. `ps -o pid,ppid,stat,cmd -p <pid>`) or the **`S`** field in `/proc/<pid>/status` or `top`:

- **`Z`** → zombie. Already dead; needs reaping, not killing.
- **`D`** → uninterruptible sleep (usually disk/NFS I/O). Cannot be killed; the underlying I/O must be diagnosed.

(For completeness, the candidate might mention `T` = stopped/traced, which *also* won't respond cleanly — but that's a different fix, `SIGCONT`. Not the answer here.)

Note also the **PPID** column while you're there — if it's `Z`, you'll need the parent PID to act on it.

### The fix for each branch

**If `Z` (zombie):**
- You can't kill the zombie. Reap it by getting the **parent** to call `wait()`:
  - Often: the parent will reap it on its own eventually; if the parent is healthy, just wait.
  - If the parent is itself hung or buggy: sending `SIGCHLD` to the parent may nudge it to reap.
  - Last resort: kill the parent. The zombie (and its siblings) get re-parented to PID 1 (init/systemd), which reaps them automatically. *Only do this if killing the parent is safe* — killing the parent kills its other children too.
- Lesson: zombies are a *parent* problem, not a zombie problem.

**If `D` (uninterruptible sleep) — Ravi's likely case:**
- You cannot kill it. No signal will help. Do **not** reboot as a first resort (you'll lose three developers' work and the D-state may block the reboot anyway, requiring a hard reset).
- Diagnose the underlying I/O. For NFS:
  - Check the mount health: `mount | grep nfs`, `nfsstat`, `dmesg | grep -i nfs` — look for "server not responding" messages.
  - Check network: is the NFS server reachable? `ping`, `showmount`.
  - If the NFS server is genuinely gone, the standard fix is to fix/restore the server or the network path; the D-state process will unwind when I/O returns or times out.
- For disk D-state: check `dmesg` for disk/controller errors (`ATA error`, `I/O error`, `blocked for more than 120 seconds`).
- Lesson: D-state is an *I/O* problem, not a process problem. The fix is one layer down.

### Hidden diagnostic traps

1. **Treating `kill -9` as the nuclear option that always works.** It isn't. It's the strongest *userspace* signal, but it has prerequisites. A process that ignores it isn't "broken" — it's telling you which of two states it's in.
2. **More SIGKILL.** Sending it six times is identical to sending it once for these states. The signal isn't being resisted; the kernel can't act on it. Repetition is pure ritual.
3. **Reaching for reboot.** Ravi's about to reboot a shared box. The D-state process may block a clean shutdown (the kernel waits on it), forcing a hard reset and losing three developers' work. Diagnose first.
4. **Conflating zombie and D-state.** They look similar from a distance ("won't die") but have opposite fixes: zombie = act on the parent; D-state = diagnose the I/O. Applying the wrong fix wastes time and may cause collateral damage (killing a parent unnecessarily).
5. **Missing the `STAT` column.** The single-letter state is the whole diagnostic. People stare at `ps aux` output and miss the `STAT`/`S` column because it's a single letter buried among wider columns. `ps -o pid,ppid,stat,cmd -p <pid>` surfaces it.

### The generalizable lesson

- **`kill -9` is not magic.** It's the strongest signal *that userspace can deliver*, and it has prerequisites. If it "doesn't work," the process is in a state where delivery is impossible — and that state tells you the *real* problem.
- **The "won't die" symptom forks into two opposite diagnoses.** Always check `STAT` first: `Z` ⇒ parent problem; `D` ⇒ I/O problem. The fix for one is meaningless for the other.
- **"More force" is rarely the answer in systems work.** When the strongest tool fails, the problem is usually in a different layer. Diagnose before escalating.

### Rubric — what a strong diagnosis covers

1. **States that `kill -9` cannot touch** — names both `Z` (zombie) and `D` (uninterruptible sleep), and explains why SIGKILL is a no-op in each (already dead / can't deliver while inside uninterruptible kernel I/O).
2. **Names the `STAT`/`S` column as the single diagnostic** — and the two candidate letters (`Z`, `D`) — and ideally also notes `PPID` for the zombie branch.
3. **Gives the opposite fix for each branch** — zombie: reap via parent (or kill parent cautiously); D-state: diagnose the underlying I/O, don't reach for kill.
4. **Identifies the likely branch here** — D-state, given the NFS clone trigger — while insisting on checking `STAT` rather than assuming.
5. **Explains why "more SIGKILL" is a category error** — the kernel can't act on it, repetition is identical to doing nothing.
6. **Resists the reboot reflex** — diagnoses the I/O first; warns a D-state process may block clean shutdown, making reboot risky on a shared box.

## Part B — end
