---
status: archived
artifact_type: exercise_symptoms
exercise: "[[M1_Easy_Redis_Auto_Started]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — Nobody Started Redis — So Why Is It Running on Port 6379?

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what the team observed.

**Timeline:**

- **10:00** — Riya, a developer with SSH access to the production box for debugging, is chasing a caching issue she's seeing in staging. She wants a local Redis on her *dev* machine, but she's SSH'd into `prod-app-04` by habit. She runs `sudo apt install redis-server` and steps through the prompts. Install completes, "Setting up redis-server … done." She's mildly surprised it was so quick, then goes back to her debugging and forgets about it.
- **11:30** — A different engineer, doing a routine security/port audit, runs `ss -tlnp` on `prod-app-04` and spots **redis-server listening on port 6379**. This should not be there. Production uses managed Redis elsewhere; this box has nothing to do with Redis.
- **11:35** — He checks `systemctl status redis-server`: **`active (running)`**, recent start time, looks healthy. Nobody started it.
- **11:40** — The team's Terraform is checked: no `redis-server` resource for this box. Config management checked: no Redis in the manifests. The chat lights up: "did we get breached?"
- **11:50** — Someone connects to it: `redis-cli -h localhost -p 6379`. It responds. No auth required (default Redis config). Anyone who can reach 6379 can read/write arbitrary keys and, with the default config, run `CONFIG SET dir …` tricks to write files. On a production box. That wasn't supposed to run Redis at all.

**What the team has verified:**

- Redis is unambiguously running and listening on 6379.
- `systemctl status` says `active (running)` with a start timestamp ~90 minutes ago.
- Nobody recalls starting it. Terraform/config-management don't deploy it.
- Riya, when asked, mentions she ran `sudo apt install redis-server` earlier — but she says "I only installed it, I didn't *start* it. apt install doesn't start services, does it?" (She genuinely isn't sure.)
- The Redis config is the default (`/etc/redis/redis.conf` is the pristine package version — `requirepass` is commented out, `bind 127.0.0.1` … maybe; depends on version).

**What the team has NOT realized (do not volunteer):**

- That the package's post-install script ran `systemctl enable --now redis-server` automatically.
- The distinction between the service being `active` (running now) and `enabled` (will start at boot).
- That `apt install` of a package shipping a systemd unit commonly starts and enables the service.

**What the team reports feeling:**

- A mix of "are we breached?" and "where did this come from?" The two flavors of unease feed each other.

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

**The Redis package's post-install script ran `systemctl enable --now redis-server` as part of `apt install`.** This is standard Debian/Ubuntu packaging behavior for service-shipping packages: the package includes a systemd unit file, and the postinst script (invoked by dpkg at the end of installation) registers and activates the service. The `--now` flag is the key: `enable --now` both **enables** the unit (creates the boot-time symlink in `.wants/`) and **starts** it immediately.

So Riya's `apt install redis-server` did, in order: (1) unpack the Redis binaries, (2) drop the unit file at `/usr/lib/systemd/system/redis-server.service`, (3) run the postinst script, which (4) called `systemctl enable --now redis-server` — enabling it for boot *and* starting it right now. Riya never typed `systemctl start`. She didn't have to. The package did it for her.

This is the canonical "a package install in production can start a service" gotcha. It's not a bug — it's the packaging convention (the package author assumes you *want* the service running after install). But on a box where you installed the package for development or library reasons and didn't expect a live service, it's a surprise.

### The mechanism (the package → systemd bridge)

- **Debian/Ubuntu packages can ship systemd unit files** in `/usr/lib/systemd/system/`. When dpkg installs the package, it places the unit file. The package's `postinst` maintainer script then typically runs `deb-systemd-helper enable <unit>.service` and `deb-systemd-invoke start <unit>.service` (or, equivalently, `systemctl enable --now`).
- This means **`apt install <service-package>` is, by default, a *deploy* of that service**, not just an installation of its files. The service goes from "not present" to "active and enabled" in one command.
- This is true of many packages: `nginx`, `apache2`, `redis-server`, `postgresql`, `docker.io`, `snmpd`, many others. Any package that ships a unit and follows the convention will auto-start on install.
- The behavior is *configurable* (see "preventive habit" below) but the default is "start and enable." That default is what bit Riya.

### The two systemd states to check

The four-state matrix (active/inactive × enabled/disabled) is the mental model:

| State | Means | Effect |
|-------|-------|--------|
| **active** | The service is running *right now*. | The process exists, the port is open. |
| **inactive** | The service is not running right now. | No process. |
| **enabled** | systemd is configured to start the service at *boot*. | It'll come back after reboot. |
| **disabled** | systemd is not configured to start it at boot. | It won't auto-start on reboot. |

`redis-server` here is **active AND enabled** — both. The candidate needs to check both because the remediation requires addressing both:

- `systemctl is-active redis-server` → `active`.
- `systemctl is-enabled redis-server` → `enabled`.

A common mistake: stopping the service (`systemctl stop`) but leaving it enabled. It's gone now — but it'll come back at the next reboot, and the team will be back here in a week. Conversely, disabling without stopping leaves it running until the next reboot. Both must be addressed.

### The exact remediation

In order:

1. **Stop it now** (runtime):
   ```
   sudo systemctl stop redis-server
   ```
   This terminates the running process; the port closes immediately. The security exposure ends here.
2. **Prevent it from coming back at boot**:
   ```
   sudo systemctl disable redis-server
   ```
   This removes the boot-time symlink; the service will not auto-start on reboot.
3. **(Optional) Mask it** if you want to be *certain* no one (including other services with `Requires=`) can start it accidentally:
   ```
   sudo systemctl mask redis-server
   ```
   Masking makes the unit a symlink to `/dev/null` — `systemctl start` becomes a no-op. Stronger than disable. Use this if the box should *never* run Redis.
4. **Verify** the port is closed: `ss -tlnp | grep 6379` should return nothing.
5. **Decide what to do with the package itself.** If Redis is genuinely unneeded on this box, `sudo apt purge redis-server` removes the package *and* its config. (`purge` not `remove` — `remove` leaves `/etc/redis/redis.conf` behind.) If Riya needed the Redis *client tooling* (`redis-cli`) for debugging but not the server, install `redis-tools` instead — which doesn't ship a service.

### Why "stop" alone is insufficient

- `stop` affects only the *current* runtime. Because the postinst `enable`d the service, it's in the boot-time `.wants/` of `multi-user.target`. On the next reboot, systemd brings it up again automatically. The team would see Redis "mysteriously reappear" next patch Tuesday.
- This is the `enable`/`start` distinction from the Services concept note made concrete: `stop` is the inverse of `start`; `disable` is the inverse of `enable`. They operate on different things (runtime vs boot). You need both to fully remediate.

### Hidden diagnostic traps

1. **Assuming "apt install" only installs files.** It doesn't. For service packages it also deploys the service. This is the central surprise. Treating install as "just files" is how production gets an unexpected Redis.
2. **Treating "breach" as the leading hypothesis.** The team jumped to "are we compromised?" The simpler explanation — a teammate ran `apt install` — is both more likely and easier to verify. (That said, the unauthenticated Redis *is now* a genuine security issue that needs remediation regardless of how innocent the cause was.)
3. **Stopping without disabling.** The service comes back on reboot. Half-fix.
4. **Disabling without stopping.** The service keeps running until reboot. Half-fix in the other direction.
5. **`apt remove` vs `apt purge`.** `remove` leaves config; a later reinstall (or a different package pulling redis-server as a dep) silently resurrects with the old config. `purge` is the clean remove.
6. **Forgetting `redis-tools` vs `redis-server`.** If Riya only needed the CLI, `redis-tools` is the right package and doesn't ship a service. The "server" suffix was the clue she missed.

### The generalizable habit

- **`apt install` of a service-shipping package is a deploy, not just an install.** Treat it as such in production. The reflex on any `apt install <pkg>` on a production box should be: "what does this package's postinst do? Will it start a service?"
- **Install without auto-starting**: there's a dpkg policy mechanism to prevent service start on install:
  ```
  sudo systemctl mask redis-server        # before install — strongest
  # or, transiently for one install:
  sudo RUNLEVEL=1 apt install redis-server
  # or set the policy-rc.d "unattended" hook to deny service start
  ```
  The cleanest preventive approach: if you don't want a service running, mask its unit before install, or use `policy-rc.d` to forbid package-initiated service starts globally.
- **After any production `apt install`, audit**: `systemctl list-unit-files --state=enabled` — see what's now enabled that wasn't before. Compare against a known-good baseline.
- **The four-state matrix is the model**: active/inactive × enabled/disabled. "Is it running?" and "will it run at boot?" are different questions with different answers and different commands.
- **Prefer `redis-tools` (or the `-client` / `-cli` variant) when you need the client**, reserving `redis-server` (and its kin) for boxes where you actually want the daemon.

### Rubric — what a strong diagnosis covers

1. **Identifies package postinst `systemctl enable --now` as the mechanism** — `apt install` of a service package starts and enables the service as a side effect.
2. **Names the four-state matrix** (active/inactive × enabled/disabled) and checks both — `is-active` and `is-enabled`.
3. **Gives the complete remediation in order** — `stop` (runtime) + `disable` (boot), optionally `mask`, then verify the port; explains why `stop` alone is insufficient.
4. **Recommends `purge` over `remove`** if the package is genuinely unneeded; flags `redis-tools` as the no-service alternative for Riya's actual need.
5. **Names the "apt install is a deploy" reflex** for production — audit `systemctl list-unit-files --state=enabled` after any prod install; consider `mask`/`policy-rc.d` to prevent auto-start.
6. **De-escalates the "breach" hypothesis** — the innocent cause is more likely and is verified; the unauth Redis is still a real issue to remediate regardless.

## Part B — end
