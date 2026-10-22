---
status: archived
artifact_type: exercise_symptoms
exercise: "[[09_Medium_Archiving_Tarbomb_In_Home]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — The Tarball That Exploded Across the Home Directory

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Mei experienced.

**Timeline:**

- **Tuesday morning** — Mei needs to install "Acme Analyzer," a vendor-provided toolkit, on her build account. She downloads `toolkit.tgz` from the vendor support portal into her home directory.
- **Tuesday 10:14** — She runs, from `~`:
  ```
  tar -xvzf toolkit.tgz
  ```
- **Tuesday 10:14:05** — The terminal scrolls. And scrolls. Roughly 50 entries fly by — `README`, `analyzer`, `analyzer.conf`, `libcore.so`, `utils/`, `scripts/setup.sh`, `LICENSE`, `CHANGELOG`, and on and on.
- **Tuesday 10:15** — Mei looks at her home directory with fresh eyes: it now contains her pre-existing project folders and notes *interleaved* with ~50 new files she doesn't recognize, some loose in `~`, some in partial subdirectories like `utils/` and `scripts/` that didn't exist before.

**What Mei observes:**

- She has extracted tarballs many times before. The usual pattern: one new directory appears (e.g. `toolkit-1.2/`), and *everything* lands inside it. This time, the archive produced no top-level directory at all — files went straight into her current working directory.
- The new files are mixed with her own. Some names (`README`, `LICENSE`) are generic enough that she can't be sure whether they're hers or the tarball's without checking timestamps. (Hers aren't, but she hasn't confirmed that yet.)
- The `utils/` and `scripts/` subdirectories didn't exist before the extraction — she's fairly sure of that, but "fairly sure" isn't good enough for a `rm`.

**What Mei has NOT done (do not volunteer):**

- She has not deleted anything. She's frozen because she doesn't know the exact file list.
- She has not run any command to *list* what the tarball deposited. She's been staring at `ls ~` trying to remember what was there before.

**What Mei reports feeling:**

- "I've used `tar -xvzf` a hundred times and it always made a folder. This one just threw files everywhere. Did I do something wrong? And how do I clean this up without nuking my own files?"

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

**The tarball is a "tarbomb"** — an archive whose entries are stored *without* a common top-level directory. When the vendor built it, they `tar`'d the contents of the project directory rather than the project directory itself:

```
# vendor did something like this (produces a tarbomb):
cd toolkit-project
tar -czf ../toolkit.tgz .          # archives the *contents* — no wrapper

# instead of this (well-formed):
tar -czf toolkit.tgz toolkit-project/   # archives the directory — wrapper preserved
```

So the archive's entry list looks like `./README`, `./analyzer`, `./libcore.so`, `./utils/parse.so`, … — every path is relative to *the current directory at extraction time*, with no enclosing folder. `tar` faithfully extracts each entry into whatever directory you run it from. Mei ran it from `~`, so 50 entries landed directly in `~`.

This is not a bug in `tar`, and Mei didn't do anything wrong at extraction time. The fault is in how the archive was built. The defensive habit on the *receiving* end is to inspect before extracting — see "preventive habit" below.

### The mechanism (why well-formed archives don't do this)

- `tar` records the path of each entry *as it was added*. If you add `toolkit-project/README`, the entry is `toolkit-project/README`, and extraction creates `toolkit-project/` first, then puts `README` inside it. The wrapper directory is just another entry in the archive — the top-level one.
- If you add `.` (the contents of the directory) instead of the directory itself, the entries are `./README`, `./analyzer`, etc. There is no wrapper entry. Extraction puts them straight into CWD.
- So the "single tidy folder" behavior Mei expects is *not* a feature of `tar` — it's a property of how the archive was *authored*. A badly-authored archive produces a tarbomb. `tar` has no opinion; it extracts what it was given.
- The same applies to `unzip`: a zip with no top-level folder behaves identically. It's an archive-construction issue, not an extraction-tool issue.

### The single command that lists exactly what the tarball deposited

**`tar -tzf toolkit.tgz`** (list, no extraction) — the `-t` flag lists the archive's entries without writing anything to disk. This is the authoritative list of what the tarball would (and did) deposit. Mei can diff this list against her home directory to identify exactly which files came from the archive.

- Better, sorted and de-duplicated for review: `tar -tzf toolkit.tgz | sort`.
- To strip the leading `./` and get clean names for comparison: `tar -tzf toolkit.tgz | sed 's|^\./||' | sort`.
- This is the *only* safe basis for cleanup — it comes from the archive itself, not from Mei's memory of what her directory used to look like.

### The cleanup — safely

The danger: `rm *` or pattern-based deletion risks files Mei *cannot* identify by name alone. A generic `README` or `LICENSE` in the tarball shares a name with files that might have pre-existed in `~`. `rm README LICENSE` could delete Mei's own files if she's wrong about provenance.

Safe cleanup sequence:

1. **Generate the canonical file list from the archive** (not from memory, not from `ls`):
   ```
   tar -tzf toolkit.tgz | sed 's|^\./||' | sort > /tmp/toolkit-files.txt
   ```
2. **Review the list** before doing anything. Look for suspicious entries: paths starting with `/` (absolute — a malicious or broken archive could try to write outside CWD; tar normally strips leading `/` as a safety measure, but check), paths with `..` (path traversal — same concern), or names that collide with files you know are yours.
3. **Build a removal script that operates on exactly the archive's entries**, into a scratch location you can inspect:
   ```
   mkdir /tmp/toolkit-cleanup
   cd ~
   while IFS= read -r f; do
     [ -e "$f" ] && echo "rm -rf -- \"$f\""
   done < /tmp/toolkit-files.txt > /tmp/toolkit-cleanup/remove.sh
   ```
   Inspect `/tmp/toolkit-cleanup/remove.sh` *before* running it. Every line should be a tarball entry; nothing of Mei's should appear.
4. **Run the reviewed script.** Then verify with another `tar -tzf toolkit.tgz | sed 's|^\./||' | sort | while read f; do [ -e "$f" ] && echo "LEFTOVER: $f"; done` — clean output means everything's gone.
5. **Remove the tarball itself** (`toolkit.tgz`) once cleanup is verified.
6. **Re-extract into a containment directory** (see preventive habit) if Mei still needs the tool.

The discipline: the file list comes from the *archive*, not from the filesystem. The filesystem is the contaminated crime scene; the archive is the manifest.

### Why `rm *` and pattern-based cleanup are dangerous

- `*` matches everything in the CWD — including Mei's pre-existing files. Catastrophic.
- Pattern-based (`rm *.so`, `rm -rf utils`) is safer but still risky: if Mei had her own `utils/` directory or her own `.so` files, they'd be deleted too.
- Generic-name collisions (`README`, `LICENSE`, `Makefile`, `install.sh`) are the silent killer — these names are common in *both* tarballs and home directories. Name-based deletion without the archive's manifest is guessing.
- The archive's entry list is the only ground truth. Build cleanup from it, review it, then execute.

### Hidden diagnostic traps

1. **Treating this as a `tar` bug.** It isn't. `tar` extracted exactly what it was told. The fault is in the archive's construction. The fix is in extraction *hygiene*, not in switching tools.
2. **Trusting memory over the manifest.** "I think these files are new" is not a cleanup plan. The archive's entry list is authoritative; memory is not.
3. **`rm *` or pattern `rm`.** Either can delete Mei's own files. Cleanup must be built from the archive's manifest and reviewed.
4. **Missing the malicious-archive case.** A *deliberately* malicious tarball can include entries with absolute paths (tar usually strips leading `/`) or `..` traversal segments to write *outside* the CWD — overwriting `~/.ssh/authorized_keys`, `~/.bashrc`, etc. The `tar -t` inspection step catches this. A vendor tarball is probably just sloppy, not malicious — but the inspection habit defends against both.
5. **Forgetting to re-extract safely.** Once cleaned up, Mei still needs the tool. Re-extracting the same way reproduces the problem. She needs the containment pattern (below).

### The preventive habit

- **Always list before extracting: `tar -tvzf toolkit.tgz` (or `unzip -l`)** — inspect the entry list. If the first few entries are `toolkit-1.2/`, `toolkit-1.2/README`, … you have a well-formed archive and safe extraction. If they're `./README`, `./analyzer`, … it's a tarbomb — extract into a containment directory.
- **Extract into a fresh subdirectory always**, regardless of whether you trust the archive:
  ```
  mkdir toolkit-extract && tar -xzf toolkit.tgz -C toolkit-extract
  ```
  The `-C` flag changes tar's extraction directory. Even a tarbomb is contained — it litters `toolkit-extract/`, not `~`. After extraction, inspect; if there's a single wrapper dir inside, `mv toolkit-extract/toolkit-1.2 ~/toolkit-1.2` and `rmdir toolkit-extract`. If it's a tarbomb, you've isolated it and can decide what to do.
- **`tar -k` (keep existing files)** is a useful safety flag: tar will refuse to overwrite existing files during extraction. If extraction would clobber something of Mei's, tar errors instead. Belt-and-suspenders alongside `-C`.
- **The general rule**: never extract into a directory you care about. Always extract into a fresh empty directory and move things out deliberately. This single habit makes tarbombs and malicious archives harmless.

### Rubric — what a strong diagnosis covers

1. **Names the "tarbomb" pattern** — an archive without a top-level wrapper directory; fault is in archive construction, not extraction.
2. **Explains the mechanism** — `tar` records entry paths as added; archiving `.` instead of the directory yields no wrapper; extraction is faithful to the entry list.
3. **Names `tar -tzf toolkit.tgz` as the authoritative manifest** — the safe basis for cleanup; not memory, not `ls`.
4. **Walks the safe cleanup** — build removal list from the archive, review it, execute, verify; explicitly warns against `rm *` and pattern `rm` because of generic-name collisions.
5. **Calls out the malicious-archive dimension** — `..` traversal and absolute paths; inspection defends against both sloppy and malicious archives.
6. **Prescribes the preventive habit** — `tar -t` before `tar -x`; extract into a containment directory (`-C`); `tar -k` to refuse overwrites.

## Part B — end
