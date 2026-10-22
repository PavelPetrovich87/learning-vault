<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/devops/linux -->

# 🎴 Linux Permissions — Tricky Flashcards

<!-- Source: [[Linux_Permissions]] · [[Root_User_vs_Regular_User]] · [[Users_and_Groups]] -->

<!-- **Design note:** Per the AI-era flashcard principle in `AGENTS.md`, these cards target the **mental model** behind two questions: (1) *is root truly omnipotent?* and (2) *the non-additive, first-match-wins permission check*. Flag/syntax recall (octal tables, `chmod` flags) is deliberately excluded — those are lookups that belong in the concept note. The cards here are mostly **predict-the-output** and **debug-the-AI** drills, because the competence they build — *executing the kernel's permission algorithm in your head* — is exactly the layer AI cannot install for you. -->

## Root is not omnipotent — the layered model

<!-- Does the root user bypass the Unix rwx (DAC) permission model? And is that the same as being omnipotent? -->
<!-- ? -->
<!-- Root **bypasses the DAC model entirely** — it ignores owner/group/other bits, the sticky bit, and can `chown` anything. But it is **not omnipotent**: MAC systems (SELinux, AppArmor) still enforce policy on root; immutable files (`chattr +i`) block root until the attribute is removed; read-only mounts block root at the VFS layer; and locked-down kernels refuse module loads or read-only sysctls even to root. Correct framing: root ignores *one layer* (DAC), not *every* layer. -->
<!-- #flashcards/devops/linux/permissions -->

<!-- A junior admin says "root can do anything, so we don't need SELinux." What's the flaw? -->
<!-- ? -->
<!-- Root bypasses **DAC** (rwx ownership checks) but **not MAC**. SELinux/AppArmor enforce *policy* (e.g., "nginx may read `/var/www` but not `/home`") that applies **even to root**. A compromised root process running nginx is still confined by SELinux type enforcement. MAC is precisely the layer that keeps a root compromise from becoming a full system compromise — that's why hardened servers enable it *on top of* root. -->
<!-- #flashcards/devops/linux/permissions -->

<!-- Name three things the root user cannot do despite "having every right." -->
<!-- ? -->
<!-- (1) Modify/delete a file with the **immutable attribute** (`chattr +i`) until it's removed. (2) Write to a **read-only mount** (must `mount -o remount,rw` first). (3) Pass an SELinux/AppArmor **policy denial** — root is still subject to MAC. (Also: load modules on a kernel with module-loading disabled, write read-only sysctls, etc.) These show root ignores the rwx layer but not the layers above/below it. -->
<!-- #flashcards/devops/linux/permissions -->

<!-- Why can root delete a file in `/tmp` that belongs to another user, when a regular user cannot? -->
<!-- ? -->
<!-- `/tmp` is mode `1777` — world-writable plus the **sticky bit**. The sticky bit says "only the file's owner (or root) can delete or rename entries here." A regular user is blocked by the sticky bit even though the directory is writable. **Root ignores the entire DAC model, including the sticky bit**, so it can delete anyone's `/tmp` files. This is a concrete example of root bypassing DAC — and of the sticky bit being the *only* thing protecting regular users from each other in a shared writable dir. -->
<!-- #flashcards/devops/linux/permissions -->

## Ownership is independent metadata

<!-- Is a file's owner required to be a member of the file's group? Can root assign a group the owner doesn't belong to? -->
<!-- ? -->
<!-- **No, and yes.** Owner and group are just two numbers (UID and GID) stored in the inode — independent metadata, not live references to `/etc/passwd` or `/etc/group`. The kernel performs **no consistency check** when `chown` runs. Root can assign `alice:finance` even if alice isn't in `finance`, and can even assign a GID that **no group uses** (the file then displays the raw number). This is also why `userdel` without `-r` leaves orphaned files: ownership is by UID number, not by the user's existence. -->
<!-- #flashcards/devops/linux/permissions -->

<!-- Can a regular user run `chown alice:finance file` to give the file to alice? Can they `chgrp file finance`? -->
<!-- ? -->
<!-- **`chown` to another user: no** — Linux blocks regular users from giving files away (prevents quota/billing abuse); only root can change the owner freely. **`chgrp` to `finance`: only if the user is already a member of `finance`** — regular users may set the group only to one they belong to. The asymmetry exists because giving away ownership has system-wide implications, while changing the group is a constrained delegation. The "ownership is independent metadata" freedom is a **root privilege**. -->
<!-- #flashcards/devops/linux/permissions -->

## First-match-wins — the non-additive rule (predict the output)

<!-- Alice owns file `f`. The mode is `----rwx--- alice finance` and alice IS in the finance group. Can alice `cat f`? -->
<!-- ? -->
<!-- **No — permission denied.** The kernel checks buckets **in order**: owner → group → others, and stops at the first match. Alice is the owner, so it uses the owner triplet `---` and **never consults the group triplet**, even though `finance` would have granted `rwx`. This is the **non-additive / first-match-wins rule**: permissions are not summed or maxed. The triplet that matches you first is the only one that applies, even if a later triplet is more permissive. -->
<!-- #flashcards/devops/linux/permissions -->

<!-- Same file as above (`----rwx--- alice finance`, alice in `finance`). Root runs `chown root:finance f` — nothing else changes. Can alice `cat f` now? -->
<!-- ? -->
<!-- **Yes — full access (read/write/execute).** The mode is unchanged (`----rwx---`), but alice is no longer the owner, so the owner triplet doesn't match her. The kernel falls through to step 2: "is alice in `finance`?" → yes → use the group triplet `rwx`. **The only thing that changed is ownership, and alice's access went from denied to full.** Transferring ownership *to* a user can reduce their access; transferring it *away* can grant it. This is why "owner = most access" is a false intuition. -->
<!-- #flashcards/devops/linux/permissions -->

<!-- File is `-r-------- alice alice` (owner-only read, nothing else). Alice is the owner. Can alice read it? Can she write to it? -->
<!-- ? -->
<!-- **Read: yes. Write: no.** Owner triplet is `r--` → read granted, write denied. Since alice matches the owner bucket first, the group (`---`) and others (`---`) triplets are never consulted. Note: this is *not* about the other triplets being empty — even if they were `rwx`, alice-as-owner would still only get `r--`. The first-match-wins rule means the owner triplet alone decides her access. -->
<!-- #flashcards/devops/linux/permissions -->

<!-- Alice owns `f` with mode `-rwxr-xr-x`. She runs `chmod u-r f`. New mode: `-rwx` becomes `-wx` for owner. Can alice still read the file via the group triplet (`r-x`)? -->
<!-- ? -->
<!-- **No.** Alice is the owner, so the kernel uses the owner triplet only — now `-wx` (write+execute, **no read**). The group triplet `r-x` is unreachable to her because she matched the owner bucket first. Removing a bit from the owner triplet **cannot be compensated** by a more permissive group triplet. This is the practical trap: "I'll just read it via the group" does not work when you're the owner. Fix: either restore `u+r`, or `chown` the file away so you fall through to group. -->
<!-- #flashcards/devops/linux/permissions -->

## Debug-the-AI — find the planted bug

<!-- A teammate claims: "The file is `-rw-r----- alice finance`, alice is in `finance`, so alice definitely has read access." What's wrong with this reasoning? -->
<!-- ? -->
<!-- It's **correct in outcome but the reasoning is incomplete and fragile.** Alice has read access — but **because the owner triplet `rw-` grants it**, not because she's in `finance`. The group membership is irrelevant here: as owner, she matches bucket 1 and the group triplet is never consulted. If someone later ran `chmod u-r` (owner triplet → `--`), alice would lose read access **even though she's still in `finance` and the group triplet still says `r--`.** The correct mental model is "which single triplet matches first?", not "which groups am I in?" -->
<!-- #flashcards/devops/linux/debug -->

<!-- An AI-generated script does `chown alice:finance f` to "make sure alice can read the file." The file was previously `----r----- root finance` and alice is in `finance`. Did this help or hurt alice's access? -->
<!-- ? -->
<!-- **It hurt — alice lost access.** Before: alice wasn't owner, fell through to group `finance` → `r--` → she could read. After `chown alice:finance`: alice is now the owner, so the kernel uses the owner triplet `---` → **permission denied**. The "fix" broke her access because making her the owner moved her from the permissive group bucket to the empty owner bucket. The lesson: `chown` can *reduce* a user's access when the owner triplet is stricter than the group triplet they benefited from. -->
<!-- #flashcards/devops/linux/debug -->

<!-- An AI suggests `chmod -R 644 /var/www` to "fix permissions" on a web directory. Why will this break things? -->
<!-- ? -->
<!-- `644` = `rw-r--r--` has **no execute bit**. Directories need the `x` bit to be traversable (`cd`, or accessing a file by path). After this command, no subdirectory under `/var/www` can be entered — web server returns 403, and even the admin can't `cd` in. The correct fix is the capital-`X` form: `chmod -R u+rwX,go+rX /var/www` — `X` adds execute only to directories (and already-executable files), preserving traversal without marking every file executable. -->
<!-- #flashcards/devops/linux/debug -->

## Why the rule exists — the design reason

<!-- Why does the kernel use first-match-wins instead of giving you the union (best) of all matching triplets? -->
<!-- ? -->
<!-- **Predictability and least-surprise within each audience.** Union semantics would make access hard to reason about: combining owner `r--`, group `-w-`, others `--x` into `rwx` would silently grant capabilities no single triplet intended. First-match-wins gives each file a clear **priority order** (owner > group > public) and lets the owner's triplet be the definitive statement for that user. It also makes the check **O(1)** — one comparison, then decide — rather than scanning and merging all triplets. The cost is the counterintuitive cases above, which are rare in practice because owners usually set their own triplet to be at least as permissive as the group. -->
<!-- #flashcards/devops/linux/permissions -->

## Meta — what AI changes about this topic

<!-- In the AI era, which permission competences still matter, and which does AI safely absorb? -->
<!-- ? -->
<!-- **AI absorbs:** octal↔symbolic conversion, exact `chmod`/`chown` flags, the rwx bit table, default modes — all lookups. **Still matters:** *executing the permission algorithm* (first-match-wins, file-vs-directory semantics, the deletion-needs-directory-write trap), *predicting* whether a given user can access a given file, recognizing when `chmod -R` or a misplaced `chown` will break things, and knowing that root isn't omnipotent across MAC/VFS/immutable layers. These are judgment calls AI cannot make for you at approval time — you must be able to read a proposed permission change and foresee its effect. -->
<!-- #flashcards/devops/linux/meta -->

<!-- Why is "let the AI set the permissions" risky without you understanding first-match-wins? -->
<!-- ? -->
<!-- Because permission changes have **non-obvious downstream effects** that AI confidently gets wrong: an AI may `chown` a file *to* a user and accidentally remove their access (owner triplet stricter than group), or `chmod -R 644` a directory and brick traversal, or claim "alice is in the group so she can read it" when she's the owner of a more-restrictive triplet. You must be able to **trace the kernel's check** (owner? → group? → others?) and predict the outcome before approving the change. Critical reading of permission proposals is the layer AI cannot give you. -->
<!-- #flashcards/devops/linux/meta -->
