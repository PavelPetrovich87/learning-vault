---
status: archived
exercise_type: incident_diagnosis
difficulty: intermediate
pattern: archiving
source_concepts:
  - "[[Archiving Commands]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The Tarball That Exploded Across the Home Directory

## Type
**Incident diagnosis + cleanup** — an operator downloaded a vendor tarball, extracted it, and instead of a tidy project folder they got fifty-odd loose files scattered across their working directory. You are the senior who recognizes the pattern, explains the mechanism, and — crucially — walks them through the *safe* cleanup (because the obvious cleanup is itself dangerous). Medium difficulty: the cause is a famous anti-pattern, but the cleanup has a trap.

## Difficulty
**Medium.** Single concept (the "tarbomb" — an archive without a top-level wrapper directory), but the symptom is alarming (files everywhere) and the cleanup has a real trap: the instinct to `rm *` is exactly how people delete their own pre-existing files that happened to share names. A strong answer names the pattern, explains *why* well-formed archives don't do this, and prescribes the disciplined cleanup + the inspection-first habit that prevents it.

## The Situation

**Where:** Acme WebOps — a build/CI workstation. You are the senior. **Mei** is a dev who needed to install a vendor's analysis toolkit.

**The system:** Mei's user account on a shared build box. Her home directory was a normal working directory — a few project folders, some notes, a couple of personal scripts. She downloaded `toolkit.tgz` from the vendor's support portal, extracted it, and the contents went *everywhere*.

**The stakes:** Low customer impact but high anxiety for Mei — she now has fifty unknown files mixed into her home directory and she can't tell, at a glance, which ones are hers and which came from the tarball. The dangerous move is to start deleting things by pattern; the safe move requires knowing exactly what the tarball contained.

**The contradiction:** Mei has extracted tarballs before, and they usually produce a *single* new directory containing the project. This one didn't — it dumped files directly into her current directory. To Mei, this looks like "tar went wrong." It didn't. The archive was built without a wrapper directory, and tar faithfully did exactly what its content list said.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- Mei downloaded `toolkit.tgz` and ran `tar -xvzf toolkit.tgz` from her home directory.
- Instead of one new subdirectory, ~50 files (some loose, some in oddly-named partial subdirs) appeared directly in her home directory alongside her existing files.
- Mei has not deleted anything yet. She's frozen because she doesn't know which files came from the tarball.
- Her home directory contained pre-existing files of her own before the extraction.
- The tarball is still on disk (`toolkit.tgz`); she hasn't removed it.

## The Ask

Walk Mei through producing:

1. **The pattern name and mechanism.** What is this called? Why did `tar` extract *here* instead of into a subdirectory? What does this tell you about how the archive was built?
2. **The safe, exact list of files the tarball deposited** — what single command produces that list *without* relying on Mei's memory or `ls`? (This is the key to safe cleanup.)
3. **The cleanup — safely.** Walk through removing exactly the tarball's files and *only* those. Explicitly explain why `rm *` is the wrong move and what specifically could go wrong.
4. **The preventive habit** — the one command Mei should always run *before* extracting any tarball, and the safer extraction invocation that contains the blast even if the archive is badly formed.

## Related Concept Notes
- [[Archiving Commands]] — archive vs compression, the "always `tar -t` before `tar -x`" habit, the tarbomb anti-pattern, safe extraction patterns.

---

*Last updated: 2026-07-17*
