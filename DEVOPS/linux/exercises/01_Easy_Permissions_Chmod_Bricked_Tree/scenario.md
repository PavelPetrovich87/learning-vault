---
status: archived
exercise_type: incident_diagnosis
difficulty: beginner
pattern: permissions
source_concepts:
  - "[[Linux Permissions]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The `chmod -R` That Took Down the Site

## Type
**Incident diagnosis** — a junior admin is reporting a website outage they caused. You are the senior on call. They will describe what they did and what they see; you must figure out *why* it broke and *how to fix it correctly*. Friendly, low-stakes, one isolated fault.

## Difficulty
**Easy.** Single concept, symptom points fairly clearly at the domain. The trap is the **fix**: the obvious "just put the permissions back" is itself a common mistake. A strong answer names the *mechanism* (which permission bit was lost, and why that matters for a directory) and the *correct remediation* (not the cargo-culted one).

## The Situation

**Where:** Acme WebOps — a small hosting shop. You are the senior sysadmin. **Sam** is a junior who joined three weeks ago.

**The system:** A single Ubuntu web server running Apache, serving a company marketing site from `/var/www/marketingsite/`. Static HTML, a few PHP scripts, an `uploads/` directory. Nothing fancy. It was healthy this morning.

**The stakes:** The marketing site is public. Every minute it's down is visible to customers. But the box itself is fine — no data loss, no security incident. Just an outage caused by a well-intentioned edit.

**What happened (high level — the persona fills in the detail):** Sam noticed some permission warnings in a security scan, decided to "tidy up" the permissions under `/var/www`, ran one recursive command, and the site went 403 Forbidden a minute later. Sam is now standing at your desk (metaphorically) with the terminal open.

## What You Know

Neutral facts available before the persona speaks. Ask the persona to confirm or elaborate:

- The site was serving normally this morning before Sam's change.
- Sam ran exactly **one** command on `/var/www` to "fix the permissions."
- Apache's error log is filling with `AH00035: Access denied` / `(13)Permission denied` messages.
- The browser shows **403 Forbidden**, not 404, not 500 — Apache can see the docroot exists but cannot serve from it.
- The files themselves are all still there and uncorrupted.

## The Ask

You need to walk Sam through producing:

1. **A diagnosis** — what specifically about the permissions is now wrong, and *why* that breaks Apache's ability to serve the site. Name the exact mechanism, not "the permissions are messed up."
2. **The single most useful diagnostic command** to *confirm* the hypothesis before changing anything — what would you look at, and what would you expect to see?
3. **The correct remediation** — and an explicit warning about the fix that *looks* right but makes things worse (or marks every file executable).
4. **A preventive habit** — how Sam should approach "tidying permissions" on a tree of mixed files and directories next time.

## Related Concept Notes
- [[Linux Permissions]] — the read/write/execute triplet model, why directories need the execute bit to be traversable, and the capital-`X` fix.

---

*Last updated: 2026-07-17*
