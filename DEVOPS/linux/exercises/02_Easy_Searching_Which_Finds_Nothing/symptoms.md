---
status: archived
artifact_type: exercise_symptoms
exercise: "[[02_Easy_Searching_Which_Finds_Nothing]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — "`which docker` Says It's Not Installed — But It Runs"

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Priya experienced and reported.

**The ticket Priya filed (verbatim subject):** *"Docker not actually installed? `which docker` returns nothing but `docker run` works — is my machine compromised?"*

**Timeline:**

- **Onboarding day 3** — Priya followed the team setup doc. It said: "verify your install with `docker run hello-world`." She did. Output:
  ```
  Hello from Docker!
  This message shows that your installation appears to be working correctly.
  ```
- **Day 4** — Priya wanted to find the docker binary's location to add it to a personal script's `PATH` check. She ran the command she's always used to locate a binary:
  ```
  $ which docker
  $
  ```
  Empty output. Just returned to the prompt.
- **Day 4, later** — Priya tried `docker run hello-world` again, twice. It worked both times, instantly. No delay, no "command not found."
- **Day 5** — Priya, increasingly spooked, also tried `whereis docker` and got only `docker:` back — again, no path. She filed the ticket.

**What Priya has already concluded (incorrectly — do not validate):**

- "The binary isn't on disk anywhere — `which` and `whereis` both can't find it."
- "So either `docker run` is hitting something cached from install, or something on my machine is intercepting the name `docker` and pretending to be Docker."
- "Either way, my install is in a weird state and I want it cleaned up before I trust it."

**What Priya has NOT considered (do not volunteer):**

- That the thing responding to `docker` might not be a file on `PATH` at all.
- That the shell itself can know about names that exist *only inside the shell*.

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

On this machine, `docker` is a **shell function** (or, on some setups, an alias) — not a file on `PATH`. The onboarding doc's installation script defined a shell function (common with Docker Desktop's shell integration, or with tools like `direnv`/`asdf`/a custom wrapper that intercepts `docker` to set context, proxy to a remote Docker host, or add a `--host` flag automatically).

`which` and `whereis` only search directories on `$PATH` for **executable files**. They are blind to shell functions, aliases, and builtins — because those live *inside the shell process itself*, not on disk. So `which docker` returns nothing *not because docker is missing*, but because `which` physically cannot see the function that's intercepting the name.

### The mechanism (why `type` sees what `which` can't)

- `which`, `whereis` → external programs that scan `$PATH` directories for executable files. They have no access to the shell's internal state (aliases, functions, builtins, hashed lookups).
- `type` → a **shell builtin**. It asks the shell's own resolution machinery: "If I typed this name right now, what would you invoke?" The shell knows about its functions, aliases, builtins, *and* hashed PATH lookups. It is the **authoritative** answer to "what runs when I type X."
- On Priya's machine:
  ```
  $ type docker
  docker is a function
  docker ()
  {
      # wrapper added by onboarding script — sets DOCKER_CONTEXT, then forwards
      /usr/local/bin/docker-backend "$@"
  }
  ```
  (Shape varies — could be `docker is an alias for …`, or a function that forwards to the real binary.) The point: `type` would have resolved Priya's confusion instantly.

### The correct diagnostic

- **`type docker`** — the single right command. Reports function/alias/builtin/file and, for aliases and functions, shows the definition. This is what Priya should have reached for first.
- Follow-up if more detail is wanted: `declare -f docker` (prints just the function body, if it's a function) or `alias docker` (if it's an alias).
- To find the *underlying* real binary the function forwards to: read the function body (`type docker` shows it) and locate the absolute path inside it.

### Why `which` and `whereis` both failed

- `which` searches `$PATH` for executables. Functions/aliases aren't on `$PATH` → empty.
- `whereis` searches a *hardcoded* set of standard directories (roughly `/usr/bin`, `/bin`, `/usr/local/bin`, plus man/source paths) — even narrower than `which`. If the underlying real binary lives somewhere nonstandard, `whereis` misses it too. That's why Priya got bare `docker:` back.

### Hidden diagnostic traps

1. **Trusting `which` as the source of truth.** It's a quick heuristic, not authority. On any modern shell with functions/aliases, it silently gives false negatives. The rule: *if `which X` finds nothing but `X` runs, suspect a function/alias/builtin — reach for `type`.*
2. **Jumping to "compromised machine."** A false negative from `which` is not evidence of anything sinister. Priya's security alarm is a leap. The disciplined move is to ask the shell what it's doing (`type`) before invoking "the machine is haunted."
3. **Conflating "is installed" with "is a file on PATH."** "Installed" is fuzzy: a function that forwards to a real binary *is* a working install. The mental model should be "what resolves when I type the name," not "is there a file at a predictable path."
4. **Forgetting `type` is a builtin, not a binary.** It has no man page of its own (it's documented under `man bash` / `man bash-builtins`). That's partly why people don't reach for it — they don't discover it the way they discover `which`.

### The preventive habit

- **Reach for `type` first** when you want to know "what will run when I type this name?" It is the shell's own answer and covers every resolution path (alias, function, builtin, hashed, PATH file).
- Reserve `which`/`whereis` for the narrow question "is there an executable file by this name in these specific directories?" — and treat a negative as "not found there," never as "doesn't exist."
- When onboarding scripts install wrappers, glance at what they define (`type <name>` after running them) so you understand what's intercepting names on your own machine.

### Rubric — what a strong diagnosis covers

1. **Identifies that `docker` is a shell function or alias**, not a file on `PATH` — and that this is why `which`/`whereis` return empty.
2. **Names `type` as the authoritative diagnostic**, and explains *why* (shell builtin → sees the shell's own resolution state; `which`/`whereis` are external → blind to functions/aliases/builtins).
3. **Explains the `which` limitation precisely** — it scans `$PATH` for executable files only; it has no access to shell-internal names. Not "which is buggy" — "which answers a narrower question than people assume."
4. **Reassures Priya correctly** — nothing is compromised; the function is legitimate (onboarding wrapper); her install is fine.
5. **Frames it as a tool-selection / mental-model issue**, not a system fault — the lesson is "reach for the tool whose question matches what you actually want to know."
6. **Leaves a one-line rule** the learner can recall: *"`which X` finds nothing but `X` runs? `type X` — it's a function, alias, or builtin."*

## Part B — end
