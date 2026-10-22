---
status: archived
artifact_type: exercise_symptoms
exercise: "[[M3_Hard_Sudo_Path_Escalation]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — The CI Sudoers Rule, the `chmod 777` "Fix," and the Service That Won't Restart

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Sana and Eric observed, in the order they observed it.

**Background (the setup, three months ago):**

- The deploy pipeline runs as user `deploy`. As the last step of a deploy, it runs `sudo systemctl restart webapp` to bounce the production service.
- Sana added a sudoers rule to allow this without a password (CI can't type passwords). She put it in `/etc/sudoers.d/deploy`:
  ```
  deploy ALL=(ALL) NOPASSWD: systemctl restart webapp
  ```
- This worked for ~3 months. Deploys ran cleanly, the service bounced, no sudo prompts.

**The original fault (two days ago):**

- Deploys start failing at the restart step. The pipeline output:
  ```
  $ sudo systemctl restart webapp
  Sorry, user deploy is not allowed to execute '/usr/bin/systemctl restart webapp' as root.
  ```
  (Note for the candidate's reasoning: the path `/usr/bin/systemctl` in the error is sudo's own resolution of the command — not necessarily what the rule said.)
- Nothing in the sudoers file changed. Sana and Eric both verify the rule is still in `/etc/sudoers.d/deploy`, exactly as written.
- The deploy had been working the day before. The rule is identical. And yet `deploy` can no longer run `systemctl restart webapp` via sudo.

**Eric's misdiagnosis and "fix" (last night):**

- Eric is paged. He stares at the sudoers rule, stares at the error, and concludes: "the `deploy` user must not have filesystem permission to read something `systemctl` needs — probably the service's working directory or its config. I'll just open it up."
- Eric runs: `sudo chmod -R 777 /opt/webapp` (the service's install directory, containing binaries, configs, and a `data/` subdirectory with runtime state and a small embedded SQLite DB).
- Eric re-tries the deploy. The `sudo systemctl restart webapp` now returns a *different* error:
  ```
  Job for webapp.service failed because the control process exited with error code.
  See "systemctl status webapp" and "journalctl -xe" for details.
  ```
- `systemctl status webapp` shows `Result: exit-status` and references the journal; `journalctl -u webapp` shows the service's binary refusing to start with a message about its data directory:
  ```
  webapp[12345]: FATAL: refusing to run: data directory /opt/webapp/data has insecure permissions (0777)
  ```
  (Many databases and security-conscious services — PostgreSQL, Redis, SSH, etc. — refuse to start if their data directory is group- or world-writable. This is a deliberate safety check.)

**The current state of the box:**

- The original sudoers fault is *still there* — Sana's rule in `/etc/sudoers.d/deploy` is unchanged.
- `/opt/webapp` and everything under it is now mode `777` — Eric's "fix" is in place and hasn't been reverted.
- The `webapp` service refuses to start because of the insecure permissions on its data directory.
- Deploy is blocked, production is in an unknown state, and `/opt/webapp/data` (which includes a database file) is now world-writable — a security exposure Eric introduced.
- Nothing has been rolled back.

**What Eric and Sana have verified:**

- The sudoers rule is present, syntactically valid, in `/etc/sudoers.d/deploy`. `visudo -c` passes.
- `deploy` genuinely cannot run `sudo systemctl restart webapp` — the original error persists.
- `/opt/webapp` is `777` (Eric's change); the service's data-dir check fails on that.
- The service was starting fine *before* Eric's chmod.

**What neither has realized (do not volunteer):**

- That the original sudoers rule has a non-absolute path (`systemctl`, not `/usr/bin/systemctl`), and that something about `deploy`'s environment changed two days ago (a PATH update, a new shell profile, a wrapper script, an `alias`/function — pick one) that altered what `systemctl` resolved to *for the deploy user*, breaking the sudoers match. sudo requires absolute paths in sudoers exactly to prevent PATH-shadowing; Sana's rule worked for 3 months by luck, until `deploy`'s environment shifted.
- That Eric's chmod had *nothing to do* with the original fault.
- That the chmod 777 has *two* consequences: the service won't start (functional), and the data directory is now writable by any user on the box (security) — including potentially overwriting the SQLite DB.

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### Reconstructing the three layers (the heart of the exercise)

There are **three distinct things** going on. A strong diagnosis keeps them rigorously separate.

**Layer 1 — The original fault (two days ago).** Sana's sudoers rule is:
```
deploy ALL=(ALL) NOPASSWD: systemctl restart webapp
```
The command is written as `systemctl` — **a bare name, not an absolute path**. sudoers requires (and `sudo` resolves commands against) absolute paths precisely to avoid PATH-shadowing attacks. When `deploy` runs `sudo systemctl restart webapp`, sudo takes the command name `systemctl`, resolves it to a path (using `secure_path` — sudo's own compiled-in search path, *not* the user's `$PATH`), and checks whether the sudoers rule permits `deploy` to run *that resolved path*. If the rule says `systemctl` (bare) and sudo resolves it to `/usr/bin/systemctl`, the match may or may not work depending on sudo's version-specific handling of bare command names — but more importantly, if `systemctl` resolves to a *different* path than the rule implicitly allows, the match fails.

What changed two days ago: something in the `deploy` user's environment (a new shell profile pushed by config management, a `~/bin/systemctl` wrapper script, a PATH reordering) caused `systemctl` to resolve — for the deploy invocation context — to a path the sudoers rule's bare-name match didn't accommodate. The rule worked for 3 months because the resolution was stable; it broke when the environment shifted. The *robust* form of the rule uses an absolute path explicitly, which is immune to this whole class of failure:
```
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart webapp
```
(Note: even better is *exactly* specifying the command + args, as above, so `deploy` can run only `restart webapp`, not arbitrary systemctl subcommands.)

This is the canonical sudoers absolute-path gotcha: **bare command names depend on resolution at runtime; absolute paths don't.** Sana's rule was fragile; it worked until it didn't.

**Layer 2 — Eric's misdiagnosis (last night).** Eric saw "user deploy is not allowed to execute '/usr/bin/systemctl restart webapp'" and concluded the problem was filesystem permissions on `/opt/webapp`. This is a leap with no evidence. The error is a *sudoers policy* rejection — it's saying sudo's rule evaluation denied the command, not that the filesystem blocked anything. `deploy` isn't even getting *to* `/opt/webapp`; the rejection happens at the sudo-policy layer, before `systemctl` runs, before the service is touched. Eric confused a sudoers-policy error with a filesystem-permission error.

**Layer 3 — Eric's worsening "fix" and its two consequences (last night).** Eric ran `sudo chmod -R 777 /opt/webapp`. This has *nothing to do* with the original fault — the sudoers error would still occur regardless of `/opt/webapp`'s modes. But it creates two new problems:

- *Functional*: `webapp` (like many security-conscious services — PostgreSQL, Redis, SSH, anything with a data-dir safety check) refuses to start when its data directory is group- or world-writable. The new error — `refusing to run: data directory /opt/webapp/data has insecure permissions (0777)` — is the service's own defensive check rejecting the state Eric created.
- *Security*: `/opt/webapp/data` is now world-writable. Any user on the box can modify the SQLite DB (and the binaries, and the configs). On a multi-user CI runner, that's a real privilege-integrity exposure — a low-privilege user could trojan the binary or corrupt/tamper the DB.

So the box now has: (a) the original sudoers fault (unfixed), (b) a service that won't start (Eric's functional damage), and (c) a world-writable data directory (Eric's security damage). Three problems, one incident.

### The diagnostic that would have revealed Layer 1 without guessing

- **`sudo -l -U deploy`** (run as root, or `sudo -l` as `deploy`) — shows the *effective* policy for `deploy`, after sudo evaluates every rule. This is what Eric should have run *first*. It would have shown exactly what `deploy` is and isn't permitted to run, and crucially *which path* the `systemctl` rule resolves to. The output would have made clear that the policy itself was the problem, not the filesystem.
- **Reading the sudoers rule and noticing the bare `systemctl`** (not `/usr/bin/systemctl`) — the absolute-path discipline. The bare name is the smell.
- **`visudo -c`** only checks *syntax*. Eric conflated syntax-validity with effective-permission (exactly the same trap as Exercise 10). `visudo -c` passing told him the rule was grammatically fine; it told him *nothing* about whether the rule effectively permitted the resolved command.

Why Eric's "it's filesystem permissions" was a leap: the error message is a sudo *policy* rejection, not an I/O or EACCES from the filesystem. The two layers produce different errors. Reading carefully — "Sorry, user deploy is not allowed to *execute* …" — points at sudo's authorization layer, not the filesystem.

### The complete remediation, in safe order

The order matters: undo Eric's damage first (the box is actively worse because of it), then fix the original fault, then verify end-to-end.

**Step 1 — Undo Eric's chmod (restore `/opt/webapp`'s permissions).**

The hard part is knowing what the *correct* modes were. `chmod -R 777` overwrote them; there's no built-in undo. Options:

- If you have a baseline (configuration management — Ansible/Puppet/Terraform defines the modes), re-apply it. This is the cleanest restore: the IaC is the source of truth.
- If you have a recent backup or a twin box, `stat`/`find -printf '%m %p\n'` the reference and replicate.
- If neither, reconstruct from principle: application binaries should be `755` (owner root, readable/executable by all, writable by no one but root); config files typically `640` (root:webapp, readable by the service user); the *data directory* specifically should be `700` or `750` owned by the service user (this is what the safety check expects — and the reason `777` failed).
- **The critical sub-step**: `/opt/webapp/data` must end up owned by the service user with no group/world write — `chmod 750 /opt/webapp/data` (or `700`), `chown -R webapp:webapp /opt/webapp/data`. That's what unblocks the service's safety check.

A reasonable one-shot (assuming root-owned binaries, service-user-owned data):
```
chown -R root:root /opt/webapp                      # binaries/configs back to root
chmod -R u+rwX,go+rX /opt/webapp                    # capital X: dirs +x, files not
chown -R webapp:webapp /opt/webapp/data             # data owned by service user
chmod 750 /opt/webapp/data                          # data dir: tight perms — passes safety check
```
(The capital-`X` discipline from Exercise 01 — `chmod -R 755` would mark every config file executable, which is sloppy.)

**Step 2 — Fix the original sudoers fault.**

Rewrite Sana's rule with an absolute path, and scope it tightly to the exact command + args:
```
# /etc/sudoers.d/deploy
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart webapp
```
This: (a) uses the absolute path (no PATH-shadowing, no environment-dependent resolution), (b) scopes to exactly `restart webapp` — `deploy` cannot run `systemctl stop`, `systemctl disable`, or `systemctl restart anything-else`. Edit with `visudo -f /etc/sudoers.d/deploy` (never a plain editor — protects against syntax corruption that could break sudo wholesale).

**Step 3 — Verify the policy.**

`sudo -u deploy sudo -l` should now list `/usr/bin/systemctl restart webapp` as permitted. Then `sudo -u deploy sudo /usr/bin/systemctl restart webapp` should actually run (the deploy step, by hand).

**Step 4 — Verify the service.**

`systemctl status webapp` — `active (running)`, no insecure-permissions error. `journalctl -u webapp --since '5 min ago'` — clean startup.

**Step 5 — Verify the security posture.**

`stat -c '%a %U:%G %n' /opt/webapp /opt/webapp/data` — confirm modes are tight, no `777` anywhere. Audit for any other `777` Eric may have created: `find /opt/webapp -perm -0777` should return nothing. Consider rotating credentials/keys that lived under `/opt/webapp` while it was world-writable — if there were secrets there, assume they were exfiltratable during the window.

**Step 6 — End-to-end deploy test.** Run the actual pipeline and confirm a clean deploy + restart.

### Hidden diagnostic traps

1. **Conflating the three layers.** The single biggest failure mode. A candidate who treats "service won't start" as the whole story misses the original sudoers fault and the security exposure. The discipline of *second-line diagnosis*: separate original fault from secondary damage, and ask "what was wrong *before* someone touched it?"
2. **Reading a sudo policy error as a filesystem error.** Eric's mistake. "Sorry, user X is not allowed to execute …" is sudoers authorization; it has nothing to do with filesystem modes. Different error classes, different layers.
3. **`visudo -c` passing ⇒ rule effective.** Same trap as Exercise 10. Syntax ≠ outcome. `sudo -l` is the outcome test.
4. **Bare command name in sudoers.** The root of Layer 1. Bare names depend on runtime resolution; absolute paths don't. Always use absolute paths in sudoers — both for reliability (Layer 1) and for security (PATH-shadowing privilege escalation is the classic exploit this guard prevents).
5. **`chmod -R 777` as a debugging tool.** *Never* the right response to "I don't understand this permission problem." It opens the box to tampering, it breaks safety checks (Layer 3's functional damage), and it destroys the information needed to restore correct modes. It's the system-administration equivalent of disabling a failing smoke detector.
6. **Forgetting the security dimension of Eric's fix.** Even after `/opt/webapp`'s modes are restored and the service starts, the *window* during which the data directory (and any secrets within) was world-writable is a real incident. Rotate credentials. Audit access logs.
7. **Over-scoping or under-scoping the rewritten sudoers rule.** `NOPASSWD: /usr/bin/systemctl` (no args) would let `deploy` run *any* systemctl subcommand — too broad. `NOPASSWD: /usr/bin/systemctl restart webapp` is right — least privilege, exactly the deploy's need.
8. **Restoring `/opt/webapp` modes with `chmod -R 755`.** Marks every config file executable; sloppy. Use capital `X` or the find-based split (Exercise 01 discipline).

### The generalizable model

- **Second-line diagnosis: separate original fault from secondary damage.** When called to a box someone already "fixed," the first question is "what was wrong before they touched it?" The current state conflates original fault + well-intentioned damage; they must be untangled before any new change.
- **Read the error class carefully.** A sudoers policy rejection ("not allowed to execute") and a filesystem permission error (EACCES, "permission denied") are different layers producing different messages. The message tells you which layer to debug.
- **Absolute paths in sudoers, always.** Bare names are environment-dependent and a PATH-shadowing security risk. `NOPASSWD: /usr/bin/systemctl restart webapp` — absolute, scoped, exact.
- **`sudo -l` is the effective-policy test; `visudo -c` is the syntax test.** Use the right one. They answer different questions.
- **`chmod -R 777` is never a debugging tool.** It destroys information, breaks safety checks, and opens security holes. The disciplined response to "I don't understand the permissions here" is to *diagnose*, not to flatten.
- **Security-conscious services check their own data-dir modes.** PostgreSQL, Redis, SSH, many others refuse to start on group/world-writable data dirs. This is a feature, not a bug — it's the service defending itself against exactly the kind of tampering `chmod 777` enables. When a service complains about "insecure permissions," it's telling you the modes are wrong, not that the service is broken.
- **Least privilege in sudoers.** Scope `NOPASSWD:` rules to the exact command + args the automation needs. `NOPASSWD: /usr/bin/systemctl` is too broad; `NOPASSWD: /usr/bin/systemctl restart webapp` is right.

### Rubric — what a strong diagnosis covers

1. **Separates the three layers** — original sudoers fault, Eric's misdiagnosis, Eric's worsening damage — and refuses to conflate them.
2. **Identifies the bare-command-name root cause** in Layer 1 — sudoers requires absolute paths; bare names depend on runtime resolution and break when the environment shifts; fix is `/usr/bin/systemctl restart webapp`.
3. **Names `sudo -l -U deploy` as the diagnostic Eric should have run** — and explains why "it's filesystem permissions" was a leap (the error is a sudo *policy* rejection, not a filesystem EACCES).
4. **Explains both consequences of `chmod -R 777`** — functional (the service's data-dir safety check rejects 777) and security (world-writable data dir enables tampering); names the safety check as a *feature*.
5. **Prescribes complete remediation in safe order** — restore `/opt/webapp` modes (capital-X discipline, tight data-dir perms), rewrite the sudoers rule with an absolute path + scoped args via `visudo`, verify policy + service + security posture, rotate any exposed secrets.
6. **Generalizes the second-line-diagnosis discipline and the "chmod -R 777 is never debugging" rule**, plus absolute-path/least-privilege in sudoers.

## Part B — end
