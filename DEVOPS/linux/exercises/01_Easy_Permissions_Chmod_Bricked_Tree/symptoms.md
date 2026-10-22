---
status: archived
artifact_type: exercise_symptoms
exercise: "[[01_Easy_Permissions_Chmod_Bricked_Tree]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — The `chmod -R` That Took Down the Site

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Sam experienced and reported.

**Timeline:**

- **09:12** — Sam ran a security scanner against `/var/www`. It flagged a handful of files as "group-writable" and "world-readable" and emitted a scary-looking summary.
- **09:14** — Sam decided to "lock it all down properly" and ran:
  ```
  sudo chmod -R 644 /var/www
  ```
  reasoning that "644 means the owner can read/write and everyone else can read — that's the safe default for a website, right?"
- **09:15** — Sam refreshed the marketing site in the browser. **403 Forbidden.**
- **09:16** — Sam tailed the Apache error log:
  ```
  [error] AH00035: Access denied because search permissions are missing on a component of the path
  ```
  Repeated for every request.

**What Sam has already tried:**

- Restarted Apache (`sudo systemctl restart apache2`). It came up clean, no config errors. The 403s continued unchanged.
- Verified the files are all still there: `ls /var/www/marketingsite/` lists the expected `index.html`, PHP files, and `uploads/` directory. Nothing was deleted.
- Checked `/var/www` itself: it's still owned by `root:root`, as it always was.

**What Sam reports feeling:**

- Confused and a little panicky. "I literally only changed permissions — I didn't touch the configs, didn't delete anything, didn't restart anything except Apache to try to fix it. How did making things *more* secure break serving the site?"

**What Sam does NOT yet realize (do not volunteer):**

- That directories and files need *different* permission bits.
- That `644` lacks a bit that directories absolutely require for Apache to do anything useful.

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

`chmod -R 644 /var/www` recursively set **every entry — files *and* directories** — to `rw-r--r--`. Directories need the **execute (`x`) bit** to be *traversable* (searchable). Without `x` on a directory, no process — Apache included — can look up entries inside it or reach anything beneath it, regardless of the file permissions below. The Apache error verbatim says it: *"search permissions are missing on a component of the path."*

`644` is the correct mode for regular files, but it is the *wrong* mode for directories. Directories should be `755` (or `750`). This is the single most common way `chmod -R` bricks a directory tree.

### The mechanism (why the fix isn't "just chmod 755 everything")

- A directory's `x` bit = permission to **traverse/enter** (look up names, `stat` files inside, `cd` in). It has nothing to do with "running" the directory.
- Apache serves `/var/www/marketingsite/index.html` by walking the path component by component: `/var` → `/var/www` → `/var/www/marketingsite` → `index.html`. Every directory in that chain needs `x` for the Apache user. Lose `x` on *any* one of them → 403 on everything below.
- Files at `644` are correct and desirable — you do **not** want every HTML/PHP file marked executable.

### The correct diagnostic (to confirm before changing anything)

- `ls -l /var/www` and `ls -ld /var/www/marketingsite` — `ls -ld` on the directory to see the directory's own mode (a bare `ls -l` shows the directory's *contents*, not the directory).
- Expected to see `drw-r--r--` (i.e. mode `644`) on the directories — the missing `x` is the smoking gun.
- Alternatively `namei -l /var/www/marketingsite/index.html` walks each component and shows the mode of each — perfect for spotting *which* directory lost the bit.

### The correct remediation

The fix must restore the execute bit on **directories only**, without marking every file executable:

```
chmod -R u+rwX,go+rX /var/www
```

The **capital `X`** is the key: it sets execute *only* on directories and on files that are already executable — it will **not** mark `index.html` or the PHP scripts as executable. (Lowercase `x` would mark everything, which is wrong.)

Equivalent `find` form, more explicit:
```
find /var/www -type d -exec chmod 755 {} +
find /var/www -type f -exec chmod 644 {} +
```

### The cargo-culted wrong fix to warn against

- ❌ `chmod -R 755 /var/www` — "fixes" the directories but marks **every file** executable. Sloppy, trips security scanners, and is wrong for the same reason Sam's original instinct was wrong: it treats files and directories the same way.
- ❌ `chmod -R 777` — the panicked "just make it work" move. Opens the site to tampering and is a textbook security anti-pattern.
- ❌ Restoring from backup — overkill. The files are intact; only the mode bits are wrong. Backup restore risks losing anything written between the last backup and now (e.g. fresh uploads).

### Hidden diagnostic traps

1. **Treating files and directories uniformly.** The entire incident stems from this. Any fix that does the same (`-R 755`, `-R 777`) repeats the original error in a new shape.
2. **Looking at the wrong `ls` output.** A plain `ls -l /var/www/marketingsite` shows the *contents' modes*, not the directory's. You must `ls -ld` the directory itself to see the missing `x`.
3. **Restarting Apache expecting it to help.** Permissions are enforced by the kernel at access time, per syscall. Restarting the daemon changes nothing — the error log even tells you the cause ("search permissions are missing").
4. **Forgetting the uploads/ directory may need different perms.** A 755/644 split works for a static site, but `uploads/` usually needs to be *writable* by the web user — confirm the app's actual requirements before declaring victory.

### The preventive habit

- Never run a blanket recursive `chmod` over a mixed tree. Decide the policy for *files* and *directories* separately.
- Prefer `chmod -R …X…` (capital `X`) for "make directories traversable, leave files alone."
- Before "tidying" production permissions, snapshot the current modes (`find /var/www -printf '%m %p\n' > /tmp/perms.before`) so you can diff/restore if it goes sideways.
- Treat a security scanner's output as a *lead*, not a *command*. Understand why each flagged file is flagged before changing it.

### Rubric — what a strong diagnosis covers

1. **Identifies the missing execute bit on directories** as the root cause — and explains that directory `x` means *traverse/search*, not "execute the directory."
2. **Connects the Apache error message** ("search permissions are missing on a component of the path") to the mechanism — recognizes the kernel is enforcing directory traversal at each path component.
3. **Names a targeted diagnostic** (`ls -ld`, or `namei -mo`) that would *confirm* the hypothesis before changing anything — doesn't just guess-and-fix.
4. **Warns against the cargo-cult fix** — explicitly says `chmod -R 755` is wrong because it marks files executable, and `chmod -R 777` is a security hole.
5. **Specifies the capital-`X` fix** (`chmod -R u+rwX,go+rX`) or the equivalent `find -type d/-type f` split — restoring directory `x` *without* touching file executability.
6. **Frames it as a files-vs-directories mental-model failure** on Sam's part, not just "you typed the wrong number" — the lesson is the model, not the magic mode.
