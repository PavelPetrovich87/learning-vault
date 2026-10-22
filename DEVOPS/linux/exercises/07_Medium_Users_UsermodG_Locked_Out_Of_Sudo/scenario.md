---
status: archived
exercise_type: incident_diagnosis
difficulty: intermediate
pattern: users
source_concepts:
  - "[[Users and Groups]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: One Character in `usermod` Locked Alice Out of `sudo`

## Type
**Incident diagnosis + recovery** — a senior engineer added herself to the `docker` group with a `usermod` command, and now `sudo` rejects her. She's mid-outage, her SSH session is her only way in, and she's terrified of logging out in case she can't get back. You are the senior walking her through both the *diagnosis* (what the command actually did) and the *recovery* (how to fix it without losing access). Medium difficulty: the cause is a famous one-character trap, but the *recovery* is where the real judgment lives.

## Difficulty
**Medium.** Single concept (supplementary group management, the `-a` flag), but the recovery raises the stakes: a wrong move bricks her access permanently, and the "obvious" recovery (re-run `usermod` correctly) requires sudo she no longer has. A strong answer names the exact mechanism, explains why closing the SSH session is dangerous, and walks the *safe* recovery path.

## The Situation

**Where:** Acme WebOps — production web tier. You are the senior on call. **Alice** is a senior engineer who was diagnosing a container issue.

**The system:** A production web server. Alice SSH'd in as herself (a `sudo`-capable account). She needed to run `docker` commands without `sudo`, so she added herself to the `docker` group. One command, she thought, harmless.

**The stakes:** High. Alice is mid-incident on a production box. Her current SSH session still works — her shell was authenticated *before* the group change. But if she logs out, the next login reads her (now-crippled) group membership, and she may lose sudo permanently on this box. The box has no other human with active sudo access right now (her teammate's shift ended; console/ILO access exists but is slow).

**The trap:** The command Alice ran *looked* correct. It used `usermod`, it named the group, it named her user. It would have worked if she'd added one character. The absence of that character is the entire incident.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- Alice ran a single `usermod` command to add herself to the `docker` group.
- Immediately after, `sudo` started rejecting her: `Sorry, user alice is not allowed to execute '/usr/bin/whoami' as root.` (or similar).
- Her *current* SSH session is still functional — she can run non-sudo commands, her shell is the same one from before the change.
- She has NOT yet logged out. She's afraid to.
- The box is reachable over SSH; console/ILO access exists but is slow and inconvenient.
- No other human with sudo is currently available (teammate off-shift).

## The Ask

Walk Alice through producing:

1. **The diagnosis.** What did Alice's `usermod` command actually do — beyond adding her to `docker`? Name the exact mechanism and the one missing character.
2. **The confirmation.** What command would show Alice *exactly* what her supplementary group list now is — and what it would have been before?
3. **The recovery — safely, in priority order.** She has a live session and sudo is broken. List the recovery options from least-risky to most-risky, and explicitly explain why "just log out and log back in" is dangerous *right now*.
4. **The preventive lesson.** Why does this one-character mistake have such outsized consequences, and what's the safe alternative command she should always use?

## Related Concept Notes
- [[Users and Groups]] — primary vs supplementary groups, `usermod -aG` vs `-G` (replace vs append), why this is "one of the most destructive one-character mistakes in Linux administration," and the supplementary-groups-read-at-login gotcha.

---

*Last updated: 2026-07-17*
