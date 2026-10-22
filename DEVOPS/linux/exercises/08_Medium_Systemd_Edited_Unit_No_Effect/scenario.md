---
status: archived
exercise_type: incident_diagnosis
difficulty: intermediate
pattern: systemd
source_concepts:
  - "[[Linux Services and Systemd]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: Edited the systemd Unit, Restarted — Nothing Changed

## Type
**Incident diagnosis** — an operator needed to run a service under a different user. They edited the unit file, restarted the service, and the change didn't take. Worse, a package update is now pending and they're worried it'll wipe their edit. You are the senior who recognizes the *two* distinct systemd gotchas this operator has tripped at once. Medium difficulty: "edited a file, nothing changed" could be several things; the answer layers two famous systemd mistakes.

## Difficulty
**Medium.** Single domain (systemd), but the symptom has two *stacked* root causes: (1) the operator edited the wrong file in the wrong way, and (2) even the right edit needs a reload step. A strong answer names both, explains why the path they edited is wrong, and prescribes the systemd-idiomatic way to override a unit.

## The Situation

**Where:** Acme WebOps — a web tier box. You are the senior. **Tomás** is an engineer hardening a service.

**The system:** An Ubuntu server running `nginx` under systemd. Tomás's security team told him nginx should not run as root in its worker processes; he needs to set `User=` for the service. He opened the unit file, made the change, restarted — and nginx is still running workers as the old user.

**The stakes:** A security finding is open against this box. Tomás needs to show evidence the change is live. But the change *isn't* live, despite his edit being clearly present in the file he edited. To make matters worse, an `apt upgrade` is now pending that includes the nginx package — and Tomás has a nagging worry (which he should) that the upgrade will interact badly with his edit.

**The contradiction:** Tomás opens the unit file — his edit is *right there* in the file. He restarts the service. The change doesn't take effect. To Tomás, this means systemd is "ignoring" the file. It isn't. He edited a file systemd gives lower priority, and he skipped a step that makes systemd re-read its configuration.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- Tomás edited a unit file *under `/usr/lib/systemd/system/`* (the path of the file he opened).
- The edit is visibly present in that file (`User=` set to the new value).
- After `systemctl restart nginx`, the workers still run as the *old* user.
- Tomás did not run any `systemctl daemon-*` command before or after the restart.
- An `apt upgrade` is pending that includes the `nginx` package.
- Tomás is worried the upgrade will overwrite or otherwise interfere with his edit.

## The Ask

Walk Tomás through producing:

1. **The two stacked root causes.** Why didn't the edit take? (There are *two* distinct systemd mechanisms at play — name both.)
2. **Why the path he edited is the wrong place** for a local override, and what *will* happen to that file when the pending `apt upgrade` runs.
3. **The systemd-idiomatic way** to override a package-shipped unit file — the command, the location it writes to, and why this approach survives package updates.
4. **The reload step** he skipped, why it's necessary, and how it differs from `systemctl restart`.
5. **The generalizable mental model** — systemd's unit-file precedence (which file wins when several define the same unit), and the `enable`/`start`/`daemon-reload` distinction.

## Related Concept Notes
- [[Linux Services and Systemd]] — units vs scripts, the unit-file precedence (`/usr/lib` < `/etc/systemd/system` < runtime overrides), `systemctl edit` for drop-ins, the `daemon-reload` requirement after editing unit files, the package-update-overwrites-direct-edits gotcha.

---

*Last updated: 2026-07-17*
