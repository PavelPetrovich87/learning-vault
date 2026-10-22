<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/devops/linux -->

# 🎴 Linux Processes — Flashcards

<!-- Source: [[Linux_Processes]] -->

<!-- **Design note:** Per the AI-era flashcard principle in `AGENTS.md`, these cards target the **mental model** layer — process genealogy (fork vs exec), state semantics (why D and Z ignore `kill -9`), the orphan/zombie distinction, and load-average interpretation. They deliberately *avoid* flag-table recall (signal numbers, `ps` option lists) — those are lookups that belong in the concept note. Most cards are **predict-the-outcome** and **debug-the-symptom** drills, because the competence they build — *reasoning about process states and family trees in your head* — is exactly the layer AI cannot install for you. -->

## fork + exec — the creation model

<!-- Running `ls` in bash: which call gives the new process its PID, and which changes its program? Are they the same call? -->
<!-- ? -->
<!-- **`fork()`** creates a new process (a clone of the shell) and assigns it a **new PID** — its PPID is the shell's PID. **`exec()`** then *replaces* that child's program image with `ls` while **keeping the same PID**. They are **two separate syscalls** — that's the point: the gap between them is where the shell wires up pipes, redirection, and the environment before the new program takes over. The `ls` PID comes from fork and is *unchanged* by exec. -->
<!-- #flashcards/devops/linux/processes -->

<!-- Why did Unix split "create a process" and "load a program" into two syscalls (`fork` + `exec`) instead of one combined call like Windows' `CreateProcess`? -->
<!-- ? -->
<!-- **Composability.** Because fork and exec are separate, the child can run setup code *between* them — redirect file descriptors, build pipelines, close inherited handles, set priority, change the environment — before becoming the new program. Pipes (`|`) and I/O redirection (`>`, `<`) are *built entirely* in that gap: the shell forks, rewires FDs in the child, then execs. A single combined call forces you to specify everything up front; the split makes the Unix pipeline model possible. -->
<!-- #flashcards/devops/linux/processes -->

<!-- After `fork()`, what (if anything) is shared between parent and child, and why is fork considered cheap on Linux despite cloning an address space? -->
<!-- ? -->
<!-- Memory is shared via **copy-on-write**: parent and child map the *same physical pages* read-only until one of them writes — only then does the kernel duplicate that page. So fork is cheap in practice because most of the cloned address space is never written. The child also **inherits the parent's open file descriptors** (and their offsets), environment, and working directory. Each gets its own PID and PPID. -->
<!-- #flashcards/devops/linux/processes -->

<!-- A daemon "double-forks." What is each fork achieving in process-model terms? -->
<!-- ? -->
<!-- **First fork:** the parent exits, the child is **reparented to PID 1** — this detaches it from the controlling terminal and the starting shell (it becomes an orphan on purpose). **Second fork:** the now-PID-1-parented child forks again and exits, leaving the grandchild with no controlling terminal and init as its ancestor — guaranteeing it can't reacquire a terminal and will be reaped by init when it dies. systemd does all this declaratively for you; manual double-fork is the pre-systemd daemonization dance. -->
<!-- #flashcards/devops/linux/processes -->

## Process states — what the STAT column means

<!-- `ps` shows a process in state `D`. You send `kill -9`. Nothing happens. Why — and what *will* end it? -->
<!-- ? -->
<!-- `D` is **uninterruptible sleep**, usually blocked on disk I/O. Signals (including **SIGKILL**) are only *delivered* when a process is scheduled, and a D-state process isn't schedulable until its I/O completes. So `kill -9` queues but never acts. It ends when the **I/O returns** (the hung NFS mount recovers, the disk responds) — then the process leaves D-state and the queued signal lands. If the I/O never returns, only a reboot clears it. Don't keep re-sending `-9`; diagnose the I/O. -->
<!-- #flashcards/devops/linux/processes -->

<!-- What's the difference between `S` (interruptible sleep) and `D` (uninterruptible sleep)? Which one is the *normal* idle state? -->
<!-- ? -->
<!-- **`S`** = waiting for an event (I/O, timer, signal) and **can be woken early by a signal** — this is the *normal* idle state; most processes spend most of their time here. **`D`** = waiting (usually on hardware/disk I/O) and **cannot be woken or killed by signals** until the operation completes — it's a "the kernel promised the I/O will finish" state. `S` honors `kill`; `D` ignores it. -->
<!-- #flashcards/devops/linux/processes -->

<!-- You see state `Z` in `ps`. The COMMAND shows `<defunct>`. What is this process doing — using CPU? Holding memory? Why does it exist at all? -->
<!-- ? -->
<!-- It's a **zombie**: the process has **already terminated** (called `exit()`), so it uses **no CPU and no memory**. It exists solely as a process-table entry holding its **exit status** and accounting stats, waiting for its **parent to call `wait()`** and collect them. The kernel can't free the slot until the status is handed off. It's not "running" in any sense — it's a receipt no one has picked up yet. -->
<!-- #flashcards/devops/linux/processes -->

<!-- What do the lowercase modifiers after a state code mean: `Ss`, `R+`, `Sl`? -->
<!-- ? -->
<!-- They refine the state. **`s`** = session leader (a shell that controls a terminal). **`+`** = in the **foreground** process group (the job currently attached to the terminal — gets your `Ctrl-C`). **`l`** = multi-threaded. So `Ss` = a sleeping session leader (a typical interactive shell), `R+` = running in the foreground, `Sl` = a sleeping multi-threaded process (e.g. a daemon). `<` = high priority, `N` = niced (low priority) are the others worth recognizing. -->
<!-- #flashcards/devops/linux/processes -->

## Zombies, orphans, and reaping — the family model

<!-- Distinguish an **orphan** from a **zombie**. Which is harmful, and why? -->
<!-- ? -->
<!-- **Orphan** = a *still-running* process whose parent died; the kernel **reparents it to PID 1**, which keeps it alive and reaps it later. Usually **benign** — daemons are deliberate orphans. **Zombie** = a *terminated* process whose parent hasn't called `wait()`; it lingers holding a process-table slot. **Zombies are the harmful one**: they cost no RAM/CPU but each holds a **PID**, and enough of them exhaust the process table so `fork()` fails. Orphans are auto-cleaned by init; zombies require the parent to reap (or die). -->
<!-- #flashcards/devops/linux/processes -->

<!-- A buggy parent is leaking zombies and you can't patch it. How do you clear the zombies, and *why* does that work? -->
<!-- ? -->
<!-- **Kill (or restart) the parent.** When the parent dies, the kernel reparents its children — including the already-dead zombies — to **PID 1 (init/systemd)**, whose defining job is to `wait()` on adopted children. PID 1 reaps them immediately, freeing the slots. This works because a zombie isn't "killed" (it's dead); it's **collected by a parent that calls `wait()`**, and PID 1 always does. Trying to `kill -9` the zombies themselves does nothing. -->
<!-- #flashcards/devops/linux/processes -->

<!-- Why is PID 1 (init/systemd) special *as a parent*, and why is a PID 1 crash catastrophic? -->
<!-- ? -->
<!-- PID 1 is the **default reaper**: every orphaned process gets reparented to it, and it `wait()`s on them so they don't accumulate as zombies. It's the **root of the process tree** — everything is its descendant. A crash is catastrophic because **no parent exists to restart PID 1** (it has no parent), and orphan/zombie reaping stops — the kernel usually panics. This is the core of the "systemd does too much" debate: PID 1 is a single point of failure *and* the process-table garbage collector. -->
<!-- #flashcards/devops/linux/processes -->

<!-- What is a **subreaper**, and why do container runtimes (Docker, containerd) set themselves as one? -->
<!-- ? -->
<!-- A subreaper (set via `prctl(PR_SET_CHILD_SUBREAPER)`) is a process that becomes the **reparenting target for its descendants' orphans** instead of PID 1. When a process inside a container dies and leaves orphans, they reparent to the container's subreaper, **not to the host's init**. This keeps the container's process tree self-contained so the runtime can reap and account for them — without it, orphaned container processes would "escape" up to the host's PID 1. -->
<!-- #flashcards/devops/linux/processes -->

## Signals and `kill` — requests, not commands

<!-- You type `kill 1234` and the process keeps running for a second, then exits. Why didn't it die instantly? -->
<!-- ? -->
<!-- `kill` with no flag sends **SIGTERM (15)** — a *request* to terminate, delivered asynchronously. Three things delay it: (1) `kill` returns immediately; the signal is only **delivered when the target is next scheduled**; (2) SIGTERM can be **caught** — a well-written program flushes buffers, closes files, and notifies peers before exiting; (3) it can even be **ignored**. That brief delay is the program doing graceful shutdown. If you need instant, uncatchable termination, `kill -9` (SIGKILL) — but you lose the cleanup. -->
<!-- #flashcards/devops/linux/processes -->

<!-- Why should `kill -9` (SIGKILL) be a *last resort*, not the default? Name two concrete risks. -->
<!-- ? -->
<!-- SIGKILL **cannot be caught**, so the process gets **no chance to clean up**. Risks: (1) **Unflushed buffers / half-written files** — a database may leave corrupt on-disk state; a writer may truncate a config file mid-write. (2) **Leaked resources** — temp files left in `/tmp`, IPC sockets/semaphores not unlinked, child processes orphaned unexpectedly. Always try plain `kill` (SIGTERM) first and give it a moment; escalate to `-9` only if it's genuinely stuck. -->
<!-- #flashcards/devops/linux/processes -->

<!-- Which two signals **cannot** be caught, blocked, or ignored — and what's the practical consequence? -->
<!-- ? -->
<!-- **SIGKILL (9)** and **SIGSTOP (19)**. Consequence: they are the **guaranteed** control signals — `kill -9` always terminates (when deliverable) and `kill -STOP` always pauses, regardless of what the process wants. This makes them your reliable backstop. The flip side: because they can't be caught, a process can't clean up before SIGKILL or save state before SIGSTOP — hence they're last-resort/blunt tools. (Note: even SIGKILL isn't delivered to a D-state process — see the D-state card.) -->
<!-- #flashcards/devops/linux/processes -->

<!-- A regular user runs `kill 1234` on a process owned by another user. What happens, and why? What if they used `sudo`? -->
<!-- ? -->
<!-- It **fails** with "Operation not permitted" — **signal permission follows ownership** (plus the root override). You may only signal processes you own. With `sudo`, the signal is sent as root, which can signal **any** process, so it succeeds. This is why running services under a shared/login account is a smell: everyone sharing that UID can kill each other's processes. Least privilege — a dedicated user per service — isolates this. -->
<!-- #flashcards/devops/linux/processes -->

## `ps aux` vs `ps -ef` — same data, different lens

<!-- `ps aux` and `ps -ef` both list every process. When would you reach for each, and what does each show by default that the other doesn't? -->
<!-- ? -->
<!-- Same underlying data (both read `/proc`), different **syntax families and columns**. **`ps aux`** (BSD, no dash) shows **`%CPU`, `%MEM`, `VSZ`, `RSS`** — reach for it for *resource* questions ("what's eating memory/CPU?"). **`ps -ef`** (SysV, with dash) shows **`PPID`** — reach for it for *parentage/tree* questions ("who spawned this?"). Add `--forest` to render the tree, `ps -eLf` to see threads (LWPs) separately. -->
<!-- #flashcards/devops/linux/processes -->

<!-- A junior reads `VSZ` and panics that a process is "using 50 GB of memory." Why are they wrong, and what number should they look at instead? -->
<!-- ? -->
<!-- **`VSZ` is virtual address space** — it includes lazy-allocated pages, memory-mapped files, and swapped-out regions that may never be touched. It's hugely inflated and tells you almost nothing about real footprint. The number that reflects actual physical RAM in use is **`RSS`** (resident set size; `RES` in `top`). A 50 GB VSZ with a 200 MB RSS is completely normal. Always judge memory by **RSS/RES**, never VSZ. -->
<!-- #flashcards/devops/linux/processes -->

<!-- In `ps -ef --forest`, where do all the `[bracketed]` kernel threads sit, and what is their common parent? -->
<!-- ? -->
<!-- They nest under **`kthreadd` (PID 2)**, which is itself a child of **PID 1**. The bracketed names (`[ksoftirqd/0]`, `[kworker/...]`, `[rcu_sched]`, `[migration/0]`) are **kernel threads** — schedulable processes that run entirely in kernel space doing housekeeping. `kthreadd` is the parent/spawner of all of them. They're normal kernel plumbing: no user program, no user address space, and they can't be meaningfully `kill`ed. -->
<!-- #flashcards/devops/linux/processes -->

## `top` — decoding the system from the header

<!-- `top` shows `load average: 4.50, 4.10, 3.90` on a **2-core** machine with `%Cpu(s): 70.0 id` (70% idle). Is the box CPU-bound? What's really happening? -->
<!-- ? -->
<!-- **No — it's I/O-bound, not CPU-bound.** Load average counts processes that are **runnable *or* in uninterruptible (D) sleep**. With 70% CPU idle yet load >2× the core count, the excess load comes from **D-state processes blocked on disk/IO**, not from CPU saturation. Confirm by checking the **`wa` (I/O-wait)** slice of `%Cpu(s)` — it'll be elevated. This is the classic load-average gotcha: high load ≠ high CPU when disk I/O dominates. -->
<!-- #flashcards/devops/linux/processes -->

<!-- What does the `Tasks: 230 total, 1 running, 229 sleeping, 0 stopped, 0 zombie` line in `top` map to in terms of process states? -->
<!-- ? -->
<!-- `running` = state **R** (on CPU or in the run queue). `sleeping` = **S + D** (interruptible *and* uninterruptible sleep). `stopped` = **T** (SIGSTOP/debugger). `zombie` = **Z** (terminated, unreaped). Note that **D-state counts as "sleeping" in the Tasks line but still inflates the load average** — that's the bridge between this line and the load-average gotcha. A non-zero `zombie` count here is an instant red flag for a leaking parent. -->
<!-- #flashcards/devops/linux/processes -->

<!-- A teammate says "load average 8 on an 8-core machine means it's overloaded." Are they right? What's the correct rule of thumb, and the caveat? -->
<!-- ? -->
<!-- **Not necessarily.** Rule of thumb: load ≈ core count means the CPUs are **fully utilized** (not overloaded) — each core has roughly one thing to do. **Above** core count, a backlog is forming (processes queueing); **below**, there's spare capacity. The caveat: load mixes **runnable + D-state** processes, so an 8.0 load could be 8 runnable (CPU-saturated) *or* 8 in disk-wait (IO-saturated, CPUs idle). Cross-check with `%Cpu id`/`wa` before concluding CPU overload. -->
<!-- #flashcards/devops/linux/processes -->

## systemctl — managing processes the clean way

<!-- You need to restart a misbehaving web server. Why `systemctl restart nginx` instead of hunting PIDs and `kill`-ing them? -->
<!-- ? -->
<!-- systemd tracks a service's processes as a **cgroup**, so `systemctl restart` stops the *whole tree* cleanly and starts fresh under supervision — no need to find master-vs-worker PIDs, and you won't leave stray children behind. It also honors `Restart=`, logs to journald, and survives reboot if enabled. Hand-`kill`-ing is the brittle pre-systemd way: you risk missing child processes, leaving orphans, and losing supervision. `systemctl status nginx` even shows the MainPID + full cgroup member list. -->
<!-- #flashcards/devops/linux/processes -->

<!-- `systemctl status nginx` shows a `Main PID` and a `CGroup` section listing more processes. What's the relationship, and why does it matter for `kill`? -->
<!-- ? -->
<!-- The **Main PID** is the primary process systemd tracks (the one `ExecStart` launched); the **CGroup** lists **all** processes systemd groups under this unit — master + workers + any children. It matters because `systemctl kill nginx` signals the **entire cgroup at once**, so you can't miss a worker. Hand `kill` on just the Main PID would leave the workers orphaned and running. The cgroup is systemd's answer to "which PIDs belong to this service?" — a process-tree boundary enforced by the kernel. -->
<!-- #flashcards/devops/linux/processes -->

## Predict-the-outcome drills

<!-- A shell (PID 1000) runs `sleep 100 &`, then immediately exits. Trace what happens to the `sleep` process and its parent. -->
<!-- ? -->
<!-- The shell forks a child (new PID, say 1050) which execs `sleep 100`; the `&` backgrounds it. When shell 1000 **exits**, the still-running `sleep` (1050) becomes an **orphan** and is **reparented to PID 1** (or a subreaper). It keeps running for the full 100s; when it finishes, its new parent (PID 1) `wait()`s on it, so it does **not** become a zombie. This is exactly why `nohup`/`systemd-run`/`disown` exist — to make this detached survival explicit and clean. -->
<!-- #flashcards/devops/linux/processes -->

<!-- Process A forks child B; B forks grandchild C; then A and B both exit immediately while C runs a 60s job. Whose child is C now, and who reaps C when it finishes? -->
<!-- ? -->
<!-- C is reparented step by step: when B (C's direct parent) exits, C is reparented — to A if A is alive, but since A also exited, C goes all the way up to **PID 1** (or the nearest **subreaper**). When C finishes its 60s job, **PID 1 reaps it** (calls `wait()`), so C does **not** become a zombie. PID 1 is the universal fallback reaper — that's why orphans are benign. -->
<!-- #flashcards/devops/linux/processes -->

<!-- A service's `top` shows one master process and four worker processes, all owned by the service's cgroup. You `kill -9` the master only. Predict what systemd does next, and why doing it via `systemctl` would've been better. -->
<!-- ? -->
<!-- The workers lose their master but are **still alive** (SIGKILL to the parent doesn't kill children — they're orphaned within the cgroup). systemd's service supervisor, seeing the **Main PID** die, treats it as a crash and (depending on `Restart=`) restarts the master — which may then spawn **new** workers while the old orphaned ones linger. You now have duplicate/orphaned workers. `systemctl restart` would have torn down the **whole cgroup** cleanly first. Lesson: kill services through systemd, not by PID. -->
<!-- #flashcards/devops/linux/processes -->
