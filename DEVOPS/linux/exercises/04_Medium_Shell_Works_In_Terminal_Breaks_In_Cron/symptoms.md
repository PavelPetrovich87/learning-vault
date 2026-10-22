---
status: archived
artifact_type: exercise_symptoms
exercise: "[[04_Medium_Shell_Works_In_Terminal_Breaks_In_Cron]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — The One-Liner That Works at the Prompt But Dies in Cron

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Dana experienced.

**The report pipeline (Dana built it iteratively at the prompt):**

```
psql -h warehouse -U reporter -d analytics -c "$(cat ~/bin/q3-revenue.sql)" | csvtool && \
  mail -s "Q3 Revenue" leadership@acme.io < report.csv
```

(Shape matters more than exact bytes — a pipeline that calls several tools, uses `~` and a relative path into `~/bin/`, and reads a file from the home directory.)

**Timeline:**

- **Monday afternoon** — Dana builds the one-liner, tests it at the prompt six times. Every run: query succeeds, CSV produced, mail arrives in leadership's inbox, report is correct. She's delighted.
- **Monday 17:00** — Dana pastes the line into `crontab -e`, schedules `0 2 * * *` (02:00 nightly), saves, confirms the entry is there.
- **Tuesday morning** — No report arrived. Dana runs the line by hand from her terminal: works perfectly, mail arrives. Concludes "cron must have had a hiccup."
- **Wednesday morning** — Still no report. Dana runs by hand again: works. Starts to worry.
- **Thursday–Saturday** — Same pattern. By Saturday Dana is re-reading the command obsessively, tweaking quotes, swapping `&&` for `;`, adding `set -x` mentally. Nothing she changes to the *command* suggests a bug, because by hand it works every time.

**What Dana has verified:**

- The crontab entry is syntactically correct (she ran `crontab -l` and stared at it).
- The schedule is right (`0 2 * * *`).
- The user the crontab belongs to is the same user she tests as by hand.
- The script — by hand — has now succeeded ~15 times across the week.

**What Dana has NOT checked (do not volunteer):**

- Anything about *how* cron launches its shell vs how her login shell is set up.
- Whether the tools in her pipeline are even findable by cron's shell.
- Where cron's stderr goes (or whether it goes anywhere at all).

**What Dana reports feeling:**

- Mystified and a little embarrassed. "The command is correct. I've proven it correct fifteen times. Why does the same command fail at 2 AM?"

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

**Cron runs commands in a minimal non-interactive shell that does not load the interactive shell's environment.** The exact failure in Dana's pipeline is almost certainly one (or more) of:

1. **`PATH` differs.** Dana's interactive shell loads `~/.bashrc` / `~/.bash_profile`, which extends `PATH` to include `~/bin`, `/usr/local/bin`, conda/pyenv/nvm shims, etc. Cron's shell sees only a tiny default `PATH` (often `/usr/bin:/bin`). Tools Dana invokes by bare name — `psql`, `csvtool`, `mail`, a helper in `~/bin/` — are silently "command not found." Cron's stderr typically goes nowhere (or to a dead local mailbox Dana never reads), so the failure is *silent*.
2. **`~` and relative paths resolve, but to a different CWD.** Cron sets `HOME` but the working directory may not be what Dana assumes. `~/bin/q3-revenue.sql` resolves via `HOME` (usually fine), but any *relative* path (e.g. `report.csv` written to CWD) lands somewhere Dana didn't test from.
3. **No aliases, no functions, no shell options.** Anything Dana relied on from `~/.bashrc` — an alias, a shell function, `set -o pipefail`, `shopt -s globstar` — is absent in cron's shell.
4. **Environment variables missing.** Anything set in her interactive session (`PGPASSWORD`, API tokens, `LANG`, `TZ`) is not present in cron unless explicitly exported in the crontab or the script.

For Dana's specific pipeline, the most likely single culprit is **#1 — a tool on her extended `PATH` that cron's minimal `PATH` cannot find**, with the silent failure making it invisible.

### The mechanism (why the same bytes behave differently)

- **Interactive login shell** (`ssh dana@host`): reads `/etc/profile` → `~/.bash_profile` (or `~/.profile`) → `~/.bashrc`. These files append to `PATH`, define aliases/functions, export env vars. Dana's terminal session is *rich*.
- **Non-interactive non-login shell** (what cron spawns): reads almost nothing. bash reads `~/.bashrc` *only if* `BASH_ENV` points at it (cron doesn't set this). In practice, cron's shell is a near-bare environment: minimal `PATH`, no aliases, no functions, no custom env.
- Cron also defines a small set of variables itself: `SHELL` (default `/bin/sh`, often dash on Debian — *not* bash!), `HOME`, `LOGNAME`, `PATH=/usr/bin:/bin`. Everything else must be supplied explicitly.
- So the *command bytes* are identical, but the *environment those bytes execute in* is radically different. A name that resolves in Dana's rich interactive `PATH` is "command not found" in cron's minimal `PATH`. She tested in the wrong environment.

### How to find which difference bit her (the disciplined way)

Don't guess. Capture what cron's shell actually sees:

1. **Make cron tell you.** Add a one-shot diagnostic cron entry that dumps the environment to a file:
   ```
   * * * * * env > /tmp/cron-env.txt; echo "PATH=$PATH" >> /tmp/cron-env.txt; which psql csvtool mail >> /tmp/cron-env.txt 2>&1
   ```
   Wait a minute. Read `/tmp/cron-env.txt`. The difference between that `PATH` and Dana's interactive `echo $PATH` is the smoking gun.
2. **Capture cron's stderr.** Cron failure is silent because stderr goes nowhere by default. Redirect it in the crontab entry:
   ```
   0 2 * * * /home/dana/bin/nightly-report.sh >> /home/dana/log/nightly.log 2>&1
   ```
   The "command not found" errors will appear.
3. **Reproduce cron's environment locally.** Test the command under the same minimal `PATH` cron uses:
   ```
   env -i HOME="$HOME" PATH=/usr/bin:/bin /bin/sh -c '...the command...'
   ```
   If it fails the same way cron fails, you've reproduced the bug *without* waiting for 02:00.
4. **Read the mail.** Cron mails command output to the local user by default (if `MAILTO` isn't unset and an MTA is configured). `mail` or `mutt` as Dana may show days of "command not found" reports she never saw because she never reads local mail.

### The correct remediation

Three layers, in order of robustness:

- **Don't depend on the environment.** In the script (turn it into a script, not a crontab one-liner), set `PATH` explicitly at the top, source only what you need, use absolute paths for every tool (`/usr/bin/psql`, `/usr/local/bin/csvtool`), `cd` to a known directory before any relative path. Make the script self-contained.
- **Set variables in the crontab** if you must keep it a one-liner:
  ```
  PATH=/usr/local/bin:/usr/bin:/bin
  MAILTO=dana@acme.io
  0 2 * * * /home/dana/bin/nightly-report.sh
  ```
- **Always redirect stderr to a logfile** you actually read. Silent failure is the default in cron; logging is opt-in.

### Hidden diagnostic traps

1. **Debugging the command instead of the environment.** Dana spent a week tweaking a correct command. The command was never the problem. The category "works interactively, fails automated" almost always points at *environment*, not *logic*.
2. **Assuming cron failure is loud.** It isn't. By default cron swallows stderr and mails it to a local mailbox most users never open. The symptom is silence, and silence is ambiguous.
3. **`/bin/sh` ≠ `/bin/bash`.** Cron's default `SHELL` is `/bin/sh`, which on Debian is `dash` — a POSIX shell without bashisms. If Dana's command used bash arrays, `[[ ]]`, process substitution, or `source`, it'll fail in `sh` even if `PATH` were fine. Either set `SHELL=/bin/bash` in the crontab or write portable POSIX.
4. **Forgetting `HOME` vs CWD.** `~` expands via `HOME` (cron sets it), but relative paths resolve against cron's CWD (often `$HOME`, but not guaranteed and not where Dana tested). Don't conflate the two.
5. **Testing in the wrong environment and declaring victory.** Dana's "15 successful hand-runs" tested the command in an environment that does not exist at 02:00. The test was valid for her terminal; it was invalid for cron.

### The generalizable habit

- **Test cron-bound commands in a cron-like environment, not your interactive shell.** `env -i HOME="$HOME" PATH=/usr/bin:/bin /bin/sh -c '…'` reproduces cron's world. If it works there, it'll work in cron.
- **Make scripts self-contained**: explicit `PATH`, absolute tool paths, explicit `cd`, no reliance on aliases/functions/interactive env.
- **Always log stderr.** Silent failure is the cron default; opt into logging every time.
- **The category rule**: when something works by hand but fails automated (cron, systemd, CI, boot scripts), suspect *environment* first, *logic* second. The bytes are usually fine.

### Rubric — what a strong diagnosis covers

1. **Names the category** — interactive vs non-interactive shell; cron runs in a minimal environment that doesn't load the interactive startup files.
2. **Lists the concrete environment differences** — minimal `PATH`, no aliases/functions, no custom env vars, `/bin/sh` vs `/bin/bash`, possibly different CWD.
3. **Identifies `PATH` as the most likely specific culprit** for Dana's pipeline (tools on her extended `PATH` not found by cron's minimal one).
4. **Names a disciplined way to capture the difference** — dump cron's `env`, redirect cron's stderr to a logfile, reproduce with `env -i … /bin/sh -c …`. Not "tweak and hope."
5. **Calls out the silent-failure trap** — cron swallows stderr by default; the missing report tells you nothing without logging.
6. **Generalizes the lesson** — "works interactively, fails automated" ⇒ suspect environment first, not logic; test in the target environment.

## Part B — end
