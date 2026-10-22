---
status: archived
exercise_type: incident_diagnosis
difficulty: intermediate
pattern: packages
source_concepts:
  - "[[Linux Package Management]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The Binary `apt upgrade` Never Updates

## Type
**Incident diagnosis** — a security patch landed for an internal tool, the team ran `apt upgrade`, the tool's version number didn't change, and now they're confused about whether they're patched. You are the senior who recognizes what kind of install this is. Medium ambiguity: "upgrade didn't take" has several causes, and the team is about to chase the wrong one.

## Difficulty
**Medium.** Single concept (the two-layer package model and what falls outside it), but the symptom class is ambiguous: an "upgrade that didn't change the version" could be stale apt lists, a held package, a third-party repo lag, *or* — the real answer — a tool installed outside the package database entirely. A strong answer runs the diagnostic that distinguishes these and lands on "this binary isn't owned by any package."

## The Situation

**Where:** Acme WebOps — an internal tooling server. You are the senior. **Lin** runs the team that maintains an internal CLI called `acme-cli`.

**The system:** A Ubuntu server that hosts shared internal tooling. `acme-cli` is installed on it and used by several teams. The `acme-cli` maintainers ship a `.deb` for most installations, but this one server has a history — it was set up years ago by someone who left.

**The stakes:** A CVE was announced for `acme-cli` yesterday. The patched version (2.3.1) is in the repo. Security is asking each team to confirm they're patched by end of week. Lin's team ran `apt upgrade`, the version still shows 2.1.0, and they can't tell security whether they're patched. The wrong conclusion either way is bad: claiming patched when you're not (security risk), or claiming vulnerable and triggering a painful manual upgrade that isn't actually necessary.

**The contradiction:** `apt upgrade` reports the system is up to date — no errors, no pending upgrades. But `acme-cli --version` still reports 2.1.0, while the repo definitively has 2.3.1. The team's mental model is "I installed it with apt, so apt should upgrade it." Something in that model is wrong.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- `acme-cli --version` reports **2.1.0**. The repo has **2.3.1** (the patched version).
- `apt update` ran clean; `apt upgrade` reported "0 upgraded, 0 newly installed, 0 to remove" — i.e. apt sees nothing to do for `acme-cli`.
- `apt show acme-cli` returns a package record, so the team assumes apt manages this install.
- The server was set up years ago by a departed engineer; nobody currently on the team did the original install.
- Security wants confirmation by Friday on whether `acme-cli` here is patched.

## The Ask

Walk Lin through producing:

1. **The category of problem.** "apt says nothing to upgrade, but the version is old" — what are the candidate explanations? Name at least three.
2. **The single diagnostic** that distinguishes "apt manages this binary" from "this binary is outside apt's knowledge." What command, and what result would each candidate explanation produce?
3. **The likely root cause here**, and the *historical* reason this server is in this state — what did the departed engineer probably do, and why does it have this consequence years later?
4. **The remediation** — both the immediate "are we patched?" answer for security, and the durable fix so this binary becomes upgradeable via apt like everything else.
5. **The generalizable lesson** about what a package manager does and does not know — what lives outside its world, and how to spot it.

## Related Concept Notes
- [[Linux Package Management]] — the two-layer model (`dpkg`/`apt`), what the package database tracks, and what falls outside it (manual/source installs into `/usr/local`, `make install`, manually-copied binaries).

---

*Last updated: 2026-07-17*
