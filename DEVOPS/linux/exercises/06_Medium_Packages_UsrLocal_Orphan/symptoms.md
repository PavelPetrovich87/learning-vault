---
status: archived
artifact_type: exercise_symptoms
exercise: "[[06_Medium_Packages_UsrLocal_Orphan]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — The Binary `apt upgrade` Never Updates

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Lin's team observed.

**The incident:**

- **Wednesday** — CVE announced for `acme-cli`. Patched version **2.3.1** is in the repo.
- **Wednesday afternoon** — Lin runs `sudo apt update` (clean — refreshes lists, no errors), then `sudo apt upgrade`. Output: `0 upgraded, 0 newly installed, 0 to remove`. apt thinks the system is fully up to date.
- Lin runs `acme-cli --version`: still **2.1.0**. Two minor versions behind, unpatched.
- Lin assumes the upgrade "didn't take" and runs `apt upgrade` again. Same output: nothing to do.

**What the team has checked and found confusing:**

- `apt show acme-cli` returns a real package record — description, version, maintainer, dependencies. So the package *exists* in apt's eyes. The team is now sure apt manages this tool.
- `apt list --upgradable` shows nothing for `acme-cli` — nothing pending.
- `apt-cache policy acme-cli` shows the candidate version is 2.3.1 — so apt *knows* 2.3.1 is available. It just doesn't think it needs to install it. (This output is genuinely confusing to the team.)
- `which acme-cli` returns `/usr/local/bin/acme-cli`. The team hasn't registered that this path is significant.

**What the team has NOT checked (do not volunteer):**

- Whether the file at `/usr/local/bin/acme-cli` is actually owned by any dpkg package.
- The contents/history of `/usr/local`.
- Whether the departed engineer built from source or hand-copied a binary.

**What Lin reports feeling:**

- "The package exists in apt. The new version exists in apt. apt says nothing to upgrade. This is contradictory. Either apt is broken or we're missing something fundamental about how this install works."

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

**The `acme-cli` binary at `/usr/local/bin/acme-cli` is *not* owned by any dpkg package.** It was installed by the departed engineer via `make install` (or `cp`, or `wget` + manual placement) years ago — outside the package database. `apt` and `dpkg` have *no idea this file exists*. When Lin runs `apt upgrade`, apt checks its *own* database of installed packages, sees that no `acme-cli` package is recorded as installed, and reports "nothing to do" — correctly, from its point of view.

Meanwhile, `apt show acme-cli` and `apt-cache policy acme-cli` work because they query the *available* package metadata from the repo — they're telling Lin "yes, this package exists in the repo, and the candidate version is 2.3.1." They are **not** telling Lin "this binary on your disk is managed by apt." That's the team's misreading: they conflated "the package exists in apt's catalog" with "this file is managed by apt." Those are two completely different questions.

### The mechanism (why apt is blind to the binary)

- The Debian/Ubuntu package model has two layers: **`dpkg`** (the low-level database of "what packages are installed, what files they own") and **`apt`** (the high-level repo/dependency layer on top). Both layers consult the *same* installed-packages database.
- A file is "owned" by a package only if `dpkg` recorded it when that package was installed. `dpkg -S /path/to/file` answers "which installed package owns this file?"
- `/usr/local/` is, by FHS convention, the **sysadmin's territory** — for software compiled locally, copied manually, or otherwise installed outside the distro's package manager. Package-managed software goes in `/usr/`. Anything in `/usr/local/bin` is almost certainly *not* tracked by dpkg.
- `make install` typically writes to `/usr/local/` by default (specifically `/usr/local/bin`, `/usr/local/lib`, `/usr/local/share`). Those files are written directly to disk; dpkg is never told. The binary works fine — but it's an orphan from the package system's perspective.
- So when the binary is at `/usr/local/bin/acme-cli`, `apt upgrade` will *never* update it, no matter how many times Lin runs it. apt doesn't know it exists. The repo's `acme-cli` package, if Lin installed it, would land in `/usr/bin/acme-cli` — a *different file* — and shadow or be shadowed by the local one depending on `PATH` ordering.

### The single diagnostic

**`dpkg -S /usr/local/bin/acme-cli`** (or `dpkg -S $(which acme-cli)`) — "which installed package owns this file?"

- Expected result here: **`no path found matching pattern …`** (or `dpkg-query: no path found matching …`). This is the smoking gun: **no package owns this file.** It is outside dpkg's knowledge. apt therefore cannot upgrade it.
- For comparison, `dpkg -S /usr/bin/ls` → `coreutils: /usr/bin/ls` — a normal package-owned file.
- Supplementary: `dpkg -l | grep acme-cli` to confirm no `acme-cli` package is recorded as installed (the team will see nothing — confirming the package isn't installed even though the binary is on disk).
- Also useful: `apt-mark showmanual | grep acme-cli` (empty) and `ls -la /usr/local/bin/` (to see what other orphans live there).

### Why the team's checks were misleading

- `apt show acme-cli` → describes the package *available in the repo*. Says nothing about whether it's installed, let alone whether *this specific binary* is managed by apt.
- `apt-cache policy acme-cli` → shows candidate version 2.3.1 is available in the repo. Again — repo metadata, not "what's on your disk." The `Installed:` line would show `(none)` — which the team overlooked. That `(none)` is the actual answer.
- The version `acme-cli --version` reports comes from the *binary itself* (compiled-in or read from its own metadata) — not from apt. So it can be anything regardless of what apt thinks.

### A strong hypothesis for the historical cause

- The departed engineer probably hit a case where the packaged version of `acme-cli` was too old, missing a feature, or unavailable for this distro release. They cloned the repo, ran `./configure && make && sudo make install` — which dropped the binary (and libs, and mans) into `/usr/local/`. It worked. They moved on. No documentation. Years later, the package system has no record of it, and the team assumes apt manages everything on the box.
- This is the canonical "shadow IT install" — a functional binary that's invisible to the package manager, can't be upgraded or audited through normal channels, and silently drifts out of date.

### The remediation

**Immediate answer for security:** No, this server is not patched. The binary at `/usr/local/bin/acme-cli` is 2.1.0, vulnerable, and apt cannot touch it. Report truthfully and patch manually now.

**Durable fix — bring the binary under package management:**

1. **Remove the orphan.** Stop relying on `/usr/local/bin/acme-cli`. Move it aside (don't delete yet — keep a backup): `sudo mv /usr/local/bin/acme-cli /usr/local/bin/acme-cli.2.1.0.bak`.
2. **Install the packaged version.** `sudo apt install acme-cli`. This lands in `/usr/bin/acme-cli`, recorded in dpkg, upgradeable via apt, auditable.
3. **Verify `PATH` ordering resolves to the new binary.** `which acme-cli` should now return `/usr/bin/acme-cli`. (On Debian/Ubuntu, `/usr/local/bin` typically comes *before* `/usr/bin` in `PATH` — so you must remove/rename the orphan or the old binary still wins.) `hash -r` to clear the shell's command cache.
4. **Confirm version.** `acme-cli --version` → 2.3.1. `dpkg -S $(which acme-cli)` → `acme-cli: /usr/bin/acme-cli` (now package-owned).
5. **Audit for other orphans.** `for f in /usr/local/bin/*; do dpkg -S "$f" 2>/dev/null || echo "ORPHAN: $f"; done` — surface what else lives outside the package system on this box. There are probably more.
6. **Document.** Leave a note in the server's runbook so the next engineer doesn't repeat Lin's confusion.

**Alternative if packaging isn't available:** rebuild from source with `checkinstall` (creates a dpkg from a `make install` so the package DB tracks it), or vendor the binary under configuration management (Ansible/Puppet) so at least its provenance and version are recorded somewhere.

### Hidden diagnostic traps

1. **Conflating "the package exists in apt's catalog" with "this file is managed by apt."** Two different questions. `apt show`/`apt-cache policy` answer the first; only `dpkg -S` answers the second. The team's whole confusion stems from running the wrong query.
2. **Trusting `which`'s output without reading the path.** `/usr/local/bin/acme-cli` is itself the clue — `/usr/local` is the package manager's blind spot by design. The team saw the path and didn't register it.
3. **Reading `apt-cache policy`'s `Candidate:` line as "what's installed."** It isn't — it's "what apt *would* install." The `Installed:` line (showing `(none)`) is the actual answer, and the team overlooked it.
4. **Assuming "apt upgrade didn't take" rather than "apt doesn't manage this."** Re-running `apt upgrade` is the natural reflex; it's also useless. The discipline is to check `dpkg -S` before re-running.
5. **Forgetting `PATH` ordering after fixing.** Even after installing the packaged version into `/usr/bin`, if `/usr/local/bin` is earlier in `PATH`, the old orphan shadows the new binary. The version won't change until the orphan is moved/removed *and* the shell's hash is cleared.
6. **Missing the broader audit.** If one binary in `/usr/local/bin` is an orphan, others probably are too. Fixing one without auditing the rest leaves the box in a half-known state.

### The generalizable lesson

- **A package manager only knows what it installed.** Its database is a list of "files I put here and will manage." Anything installed outside that — `make install`, manual `cp`, `wget` + `tar`, a colleague's undocumented hack — is invisible to it. apt cannot upgrade what apt didn't install.
- **`dpkg -S <file>` (Debian/Ubuntu) or `rpm -qf <file>` (RHEL) is the single test** for "is this file package-managed?" Negative answer → orphan → outside the upgrade path → audit and bring it under management.
- **`/usr/local/` is the tell.** Files there are almost never package-owned. Seeing a tool you care about in `/usr/local/bin` should trigger "is this tracked?" immediately.
- **For security audits specifically**: the package database is necessary but not sufficient. Orphans exist. A CVE-confirmation process that only checks `apt` will miss them.

### Rubric — what a strong diagnosis covers

1. **Identifies the binary as outside dpkg's knowledge** — installed via `make install` (or manual copy) into `/usr/local/bin`, never recorded in the package database.
2. **Names `dpkg -S $(which acme-cli)` as the single diagnostic** — and predicts the "no path found" result that proves it's an orphan.
3. **Explains why the team's checks were misleading** — `apt show`/`apt-cache policy` describe the *repo catalog*, not "what's on disk"; the `Installed: (none)` line was overlooked.
4. **Accounts for `/usr/local/`'s significance** — FHS-reserved for sysadmin territory, package manager's blind spot by design.
5. **Gives both the immediate security answer** (no, not patched) **and the durable fix** (remove orphan, install packaged version, verify `PATH`/`hash`, audit other orphans).
6. **Generalizes the lesson** — package managers know only what they installed; `dpkg -S`/`rpm -qf` is the test; `/usr/local` is the tell; security audits must account for orphans.

## Part B — end
