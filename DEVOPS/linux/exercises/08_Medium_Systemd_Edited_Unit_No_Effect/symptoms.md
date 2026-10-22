---
status: archived
artifact_type: exercise_symptoms
exercise: "[[08_Medium_Systemd_Edited_Unit_No_Effect]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — Edited the systemd Unit, Restarted — Nothing Changed

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Tomás observed.

**Timeline:**

- **Monday** — Security finding: nginx worker processes on `web-app-07` should not run as the legacy user; they want them under a dedicated `webapp-svc` account.
- **Monday afternoon** — Tomás finds the unit:
  ```
  $ systemctl cat nginx.service
  # /usr/lib/systemd/system/nginx.service
  [Service]
  …
  # (no User= directive — nginx defaults apply)
  ```
- Tomás opens the file directly: `sudo vim /usr/lib/systemd/system/nginx.service` and adds a line `User=webapp-svc` under `[Service]`. Saves. The edit is visibly present in the file.
- Tomás restarts: `sudo systemctl restart nginx`. No error. Service comes up "active (running)".
- Tomás checks the worker processes: `ps -eo user,pid,cmd | grep nginx`. The workers are still running as the *old* user — **not** `webapp-svc`. The change did not take.
- Tomás re-opens the unit file. His edit is *still there*. He restarts again. Same result.

**What Tomás has verified:**

- The line `User=webapp-svc` is unambiguously present in `/usr/lib/systemd/system/nginx.service`.
- `systemctl restart nginx` succeeds each time — service is active, no errors in `systemctl status`.
- `systemctl cat nginx.service` shows the file he edited — and his `User=` line is in the output.
- The worker processes still run as the old user, after multiple restarts.
- He did **not** run any `systemctl daemon-reload` (he doesn't know that command exists).

**The looming worry:**

- `apt list --upgradable` shows `nginx/nginxed` is pending an upgrade. Tomás has a vague, uneasy feeling that the upgrade is going to do *something* to the file he just edited, but he can't articulate what.

**What Tomás has NOT realized (do not volunteer):**

- That systemd caches unit files in memory and a `daemon-reload` is required to pick up changes.
- That the file he edited is a *package-shipped* unit file that a package upgrade will silently overwrite.
- That there's an idiomatic override mechanism (`systemctl edit`) that solves both problems at once.

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The two stacked root causes

Tomás has tripped **two** distinct systemd gotchas at once. Both have to be named for the diagnosis to be complete.

**Cause 1 — he skipped `systemctl daemon-reload`.** systemd does *not* re-read unit files from disk on every operation. It caches the parsed unit configuration in memory (in the manager process). When you edit a unit file on disk, systemd has no idea anything changed until you tell it to re-read — that's `systemctl daemon-reload`. `systemctl restart nginx` restarts the *service* using systemd's *cached* (old) configuration; it does **not** reload systemd's own view of the unit files. So even though Tomás's edit is on disk, systemd is still running with the pre-edit in-memory copy. The change is invisible to the running manager.

This is the canonical "#1 'why didn't my change take effect' mistake in systemd." The service restart and the daemon reload are two *different* operations on two *different* things.

**Cause 2 — he edited the wrong file (a package-shipped unit file that will be overwritten).** `/usr/lib/systemd/system/nginx.service` is the unit file *shipped by the nginx package*. It is owned by dpkg. When `apt upgrade` installs a new nginx package version, dpkg will silently overwrite this file with the package's new version — Tomás's `User=` edit will vanish, with at best a "conffile changed" prompt that's easy to miss in an automated upgrade. Editing `/usr/lib/systemd/system/` is the systemd equivalent of editing a file under `/usr/local/bin` that a package will later stomp: the package owns it.

systemd's unit-file precedence (highest priority wins) is, roughly:

1. Runtime overrides via `systemctl edit --runtime` (in `/run/systemd/system/`) — transient, gone on reboot.
2. **`/etc/systemd/system/<unit>.d/*.conf` drop-in overrides** — local admin overrides, highest persistent priority. This is where `systemctl edit` writes.
3. `/etc/systemd/system/<unit>.service` — full local override of the unit.
4. **`/usr/lib/systemd/system/<unit>.service`** — package-shipped defaults (what Tomás edited). Lowest priority.
5. Defaults built into systemd.

So even if Tomás *had* run `daemon-reload`, his edit would still be in the wrong file — surviving only until the next package upgrade. And an override in `/etc/systemd/system/` would *shadow* his edit anyway if one existed. The idiomatic fix solves both causes at once.

### The mechanism (why these are two distinct things)

- **systemd's two-tier state.** There's the *on-disk* unit files (the source of truth systemd reads at load time) and the *in-memory* parsed configuration (what systemd actually uses to manage services). `daemon-reload` is the bridge: it tells systemd "throw away your in-memory copy, re-read the on-disk files." Without that bridge, on-disk edits are invisible.
- **`systemctl restart <unit>` vs `systemctl daemon-reload`.** Restart operates on a *service* using the currently-cached unit definition. daemon-reload operates on *systemd itself*, refreshing its cache from disk. They are orthogonal — you almost always need both when you edit a unit: `daemon-reload` first (so systemd sees the change), *then* `restart` (so the service picks up the new config).
- **The precedence model exists precisely so local overrides survive package updates.** The package ships `/usr/lib/systemd/system/`; the admin puts overrides in `/etc/systemd/system/<unit>.d/`. Drop-in files in `/etc` *augment* the package file (they don't replace it) — they add or override specific directives while leaving the rest of the package file intact. A package upgrade refreshes `/usr/lib/...`; the admin's `/etc/...` drop-ins are untouched.

### The systemd-idiomatic override

- **`sudo systemctl edit nginx.service`** — opens a drop-in file at `/etc/systemd/system/nginx.service.d/override.conf` in the editor. Tomás types only what he wants to change:
  ```
  [Service]
  User=webapp-svc
  ```
  Save, exit. systemd writes the drop-in, and (modern systemd) automatically performs a `daemon-reload` for you. Then `sudo systemctl restart nginx` to apply to the running service.
- The drop-in *overrides* the `User=` directive from the package file without touching the package file. The next nginx package upgrade refreshes `/usr/lib/systemd/system/nginx.service` — Tomás's `/etc/systemd/system/nginx.service.d/override.conf` is untouched and continues to win by precedence.
- Verify: `systemctl cat nginx.service` now shows *both* the package file and the drop-in (with a comment line naming the drop-in file). `systemctl show nginx.service -p User` confirms the effective value. `ps -eo user,pid,cmd | grep nginx` confirms the workers run as `webapp-svc`.

### The recovery sequence for Tomás right now

1. **Revert the package-file edit** so the pending upgrade doesn't trip on a conffile conflict. Easiest: `sudo apt install --reinstall -o Dpkg::Options::="--force-confmiss" nginx` to restore the pristine package unit, *or* `sudo apt-get --reinstall install nginx` (with appropriate conffile handling) — or, if an upgrade is imminent anyway, let the upgrade restore the file. The point: get `/usr/lib/systemd/system/nginx.service` back to pristine so the package owns it cleanly.
2. **Apply the override the right way**: `sudo systemctl edit nginx.service`, add `User=webapp-svc` under `[Service]`, save.
3. **Reload + restart**: `sudo systemctl daemon-reload && sudo systemctl restart nginx`. (systemctl edit already triggers a daemon-reload, but being explicit is fine and harmless.)
4. **Verify three ways**: `systemctl cat nginx.service` (drop-in visible), `systemctl show nginx.service -p User` (effective value = webapp-svc), `ps -eo user,pid,cmd | grep nginx` (workers as webapp-svc).
5. **Now run `apt upgrade`** — the package file under `/usr/lib/...` refreshes cleanly; the drop-in under `/etc/...` survives.

### Hidden diagnostic traps

1. **Confusing `restart` with `daemon-reload`.** The most common systemd mistake. They operate on different things (service vs manager) and you need both when editing a unit.
2. **Editing `/usr/lib/systemd/system/` directly.** It's the package's territory. Local overrides belong in `/etc/systemd/system/` (or via `systemctl edit`). Direct edits to package files get clobbered on upgrade.
3. **Assuming "the edit is in the file `systemctl cat` shows, so it must be effective."** `systemctl cat` reads from disk. systemd's *running* configuration is the in-memory cache, which only updates on `daemon-reload`. The file showing the edit is necessary but not sufficient.
4. **Forgetting precedence when debugging.** If a directive seems "stuck" on an old value, check for a higher-precedence override in `/etc/systemd/system/<unit>.d/` or `/etc/systemd/system/<unit>.service` — those shadow `/usr/lib/...`. `systemctl cat` shows the full stack; `systemctl show` shows the *effective* resolved value.
5. **Overwriting instead of overriding.** Copying the package file to `/etc/systemd/system/nginx.service` and editing *that* works, but it forks the entire unit — you stop tracking package improvements to the unit. Drop-ins (`systemctl edit`) are strictly better: override only what you need, inherit the rest.
6. **Missing the upgrade interaction.** Tomás's edit + pending upgrade is a conffile-prompt landmine. The safe sequence is: revert the package-file edit, apply a drop-in override, *then* upgrade.

### The generalizable mental model

- **systemd has on-disk unit files and an in-memory cache; `daemon-reload` is the bridge.** When you change a unit on disk, you must reload systemd before the change takes effect. `restart` alone won't do it.
- **Unit-file precedence exists to separate "package defaults" from "local overrides."** Package files live in `/usr/lib/systemd/system/`; local overrides live in `/etc/systemd/system/` (full override) or `/etc/systemd/system/<unit>.d/*.conf` (drop-in). The `/etc` paths win. Never edit `/usr/lib` — it's the package's.
- **`systemctl edit <unit>` is the idiomatic override tool** — it creates a drop-in, survives package upgrades, and (modern systemd) auto-reloads.
- **`enable`/`start`/`daemon-reload` are three different operations on three different things**: `enable` controls *boot-time* start (the symlink in the `wants` target); `start` controls *runtime* state now; `daemon-reload` refreshes systemd's *config cache*. Conflating them is the root of most "systemd didn't do what I expected" tickets.

### Rubric — what a strong diagnosis covers

1. **Names `daemon-reload` as cause #1** — systemd caches unit config in memory; `restart` uses the cache; on-disk edits need `daemon-reload` to be seen.
2. **Names "edited a package file" as cause #2** — `/usr/lib/systemd/system/` is dpkg-owned; the pending upgrade will overwrite it.
3. **Explains unit-file precedence** — `/etc/systemd/system/` (and its `.d/` drop-ins) beats `/usr/lib/systemd/system/`; overrides belong in `/etc`.
4. **Prescribes `systemctl edit`** as the idiomatic override (drop-in in `/etc/systemd/system/<unit>.d/override.conf`); notes it survives upgrades and auto-reloads.
5. **Distinguishes `enable` / `start` / `daemon-reload`** as three operations on three different things (boot state / runtime state / config cache).
6. **Gives the safe recovery sequence** — revert the package-file edit, apply the drop-in, reload + restart, verify three ways, *then* run the upgrade.

## Part B — end
