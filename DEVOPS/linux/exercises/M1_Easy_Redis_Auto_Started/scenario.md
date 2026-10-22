---
status: archived
exercise_type: incident_diagnosis
difficulty: beginner
pattern: integration_package_service
source_concepts:
  - "[[Linux Package Management]]"
  - "[[Linux Services and Systemd]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: Nobody Started Redis — So Why Is It Running on Port 6379?

## Type
**Integration diagnosis (Easy)** — two concepts woven together (package management + systemd), but the symptom is clear and points cleanly at the domain. A junior installed a package for *development* purposes and a *production* instance appeared. You are the senior who recognizes that `apt install` quietly does more than unpack files. This is the on-ramp to the multi-concept tier: two concepts, no misdiagnosis, no prior damage.

## Difficulty
**Easy.** Two concepts but a clear symptom: "I installed a package, and a production service is now running that nobody started." The mental model being tested is that **installing a package can start — and enable — a service as a side effect of installation**, which sits at the intersection of the package layer and the service layer. A strong answer names the mechanism, distinguishes "started" from "enabled," and prescribes the preventive habit.

## The Situation

**Where:** Acme WebOps — a production app server. You are the senior SRE. **Riya** is a developer who was troubleshooting a local caching issue.

**The system:** A production box running the main web app. Redis is used in *staging* and *development* but is *not* supposed to run in this production environment — production caching is handled by a managed Redis elsewhere. The production box has port 6379 firewalled off and no Redis was deployed there.

**The stakes:** Security and correctness. Redis is now listening on port 6379 on a production box, with (almost certainly) no authentication configured (default Redis). It's a data-exfiltration and arbitrary-command-execution surface that wasn't supposed to exist. It's also consuming memory and could interfere with the production app under load. The urgency is "shut it down and figure out where it came from."

**The contradiction:** Nobody on the team ran `systemctl start redis`. Nobody deployed Redis to this box. The team's Terraform doesn't include it. Yet there it is: listening on 6379, in `systemctl status` as `active (running)`. To the team, this looks like either a breach or a ghost. It's neither — it's a side effect of a perfectly ordinary command one developer ran.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- Redis is listening on port 6379 on this production box (`ss -tlnp | grep 6379` shows redis-server).
- `systemctl status redis-server` shows `active (running)`.
- Nobody on the team ran `systemctl start redis-server`.
- The team's Terraform / config management does not deploy Redis to this box.
- Riya, earlier today, ran `sudo apt install redis-server` "to test something locally" — she did not connect it to the production service appearance.
- The Redis instance has its default config (no auth, default port).

## The Ask

Walk the team through producing:

1. **The mechanism.** How did Redis get from "Riya ran apt install" to "active and running as a service," without anyone ever running `systemctl start`? Name the specific package-install behavior and the specific systemd states involved.
2. **The two systemd states to check** — `active` vs `enabled`. What does each tell you, and why does it matter for "will it come back after a reboot"?
3. **The exact remediation**, in order: stop it now, then prevent it from coming back at boot. Name the commands and *why each is necessary* (stopping alone is insufficient if it's enabled).
4. **The generalizable habit** — how to install a package *without* its service auto-starting (the apt/dpkg policy mechanism), and the "did a package install just change my running system?" reflex.

## Related Concept Notes
- [[Linux Package Management]] — packages can start/enable services as a side effect of install; `apt`/`dpkg` post-install scripts run `systemctl enable --now` behind the scenes.
- [[Linux Services and Systemd]] — the four-state matrix (active/inactive × enabled/disabled); `enable` = boot-time, `start` = runtime; "started vs enabled" distinction.

---

*Last updated: 2026-07-17*
