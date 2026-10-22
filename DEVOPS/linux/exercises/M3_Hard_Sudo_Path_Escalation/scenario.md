---
status: archived
exercise_type: incident_diagnosis
difficulty: advanced
pattern: integration_sudo_permissions_services
source_concepts:
  - "[[Sudo]]"
  - "[[Linux Permissions]]"
  - "[[Linux Services and Systemd]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The CI Sudoers Rule, the `chmod 777` "Fix," and the Service That Won't Restart

## Type
**Integration diagnosis (Hard)** — three concepts woven together (sudoers + permissions + systemd), *and* the incident has been worsened by a prior misdiagnosis. This is a second-line diagnosis: someone already applied a plausible-but-wrong fix, and you have to untangle both the original fault and the damage. The hardest exercise in the set.

## Difficulty
**Hard.** Three concepts compounding, an ambiguous symptom, *and* a prior "fix" that made things worse. The candidate must reconstruct a sequence of events across three subsystems, distinguish the original fault from the secondary damage, and prescribe remediation for both — while resisting the temptation to treat any single symptom as the whole story.

## The Situation

**Where:** Acme WebOps — a CI runner that deploys to a production app server. You are the senior security engineer. **Sana** is the CI engineer who built the deploy pipeline; **Eric** is the junior who tried to fix it last night.

**The system:** A CI runner where a `deploy` user runs `sudo systemctl restart webapp` as part of the deploy pipeline. The sudoers rule granting this was added three months ago. The deploy worked fine until two days ago, when it started failing. Eric was paged last night, attempted a fix, and made things worse.

**The stakes:** High. Deploys are blocked. The production `webapp` service is in an unknown state. And — most concerning — the box's security posture has been degraded by Eric's "fix" in ways Eric doesn't fully grasp. Security needs to understand *what happened* before anything else changes, because a wrong next move could either lock everyone out or expand the exposure.

**The layering (what the candidate has to untangle):**
1. **The original fault** (two days ago): the deploy's `sudo systemctl restart webapp` started failing.
2. **The root cause of the original fault**: a sudoers gotcha about how the rule was written.
3. **Eric's misdiagnosis** (last night): Eric concluded the problem was filesystem permissions on something the deploy touched.
4. **Eric's worsening "fix"** (last night): `sudo chmod -R 777` on a directory that should never have been world-writable.
5. **The new symptom** that resulted: a service that now refuses to start for an *entirely different* reason than the original fault — and that new reason is itself a teaching moment about systemd and file permissions.

The candidate must separate all five layers and not conflate them.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- The deploy pipeline runs as user `deploy` and executes `sudo systemctl restart webapp`. This worked for ~3 months.
- Two days ago it started failing. The `sudo` invocation returned an error.
- Eric was paged last night. He concluded it was a permissions problem on a directory.
- Eric ran `sudo chmod -R 777` on that directory. (Which directory, and why this matters, is for the candidate to uncover.)
- After Eric's "fix," the situation is *worse*: the service now fails to start entirely, with a different error than the original.
- The box also now has a security exposure introduced by Eric's `chmod 777`.
- Nothing has been reverted. The box is in the state Eric left it.

## The Ask

Walk Sana and Eric through producing:

1. **The timeline reconstructed across three subsystems.** What was the *original* fault? What did Eric *misdiagnose*? What did Eric's "fix" *break*? Keep the three layers distinct.
2. **The original fault's root cause** — in the sudoers rule. What's wrong with how the rule was written, and why did it work for 3 months then suddenly start failing? (Hint: what changed wasn't the rule — it was something the rule depended on.)
3. **The diagnostic** that would have revealed the original fault without guessing — and why Eric's "it's filesystem permissions" conclusion was a leap.
4. **The damage Eric's `chmod -R 777` did** — both the immediate functional consequence (why the service now refuses to start) and the security exposure Eric introduced.
5. **The complete remediation**, in safe order: undo Eric's damage, fix the original sudoers fault properly, and verify the deploy works end-to-end. Cover both the *functional* restoration and the *security* restoration.
6. **The generalizable model** — second-line diagnosis discipline (distinguish original fault from secondary damage), sudoers absolute-path discipline, and why `chmod -R 777` is *never* the right response to "I don't understand this permission problem."

## Related Concept Notes
- [[Sudo]] — `NOPASSWD:` rules, the absolute-command-path requirement (no PATH-shadowing), `sudo -l` as the effective-policy diagnostic.
- [[Linux Permissions]] — why `chmod -R 777` is a security anti-pattern, what world-writable directories enable, the file-vs-directory mode distinction.
- [[Linux Services and Systemd]] — `systemctl status` and `journalctl` for service failures; what kinds of permission/path problems cause a service to refuse to start.

---

*Last updated: 2026-07-17*
