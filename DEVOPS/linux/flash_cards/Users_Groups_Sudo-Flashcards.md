<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/devops/linux -->

# 🎴 Users, Groups & Sudo — Flashcards

<!-- Source: [[Users_and_Groups]] · [[Sudo_and_Sudoers]] · [[Root_User_vs_Regular_User]] -->

<!-- **Design note:** Per the AI-era flashcard principle in `AGENTS.md`, these cards target **security reasoning, destructive-trap awareness, and the identity model** — the competences that compound and that AI cannot install. Flag-syntax recall is deliberately excluded (look it up). This is a security topic, so "why" matters far more than "what flag." -->

## The identity model — three files

<!-- What three files define user and group identity on a local Linux system, and what does each contain? -->
<!-- ? -->
<!-- **`/etc/passwd`** — login name, UID, GID, GECOS, home dir, login shell (world-readable). **`/etc/shadow`** — password hashes and aging policy (root-only). **`/etc/group`** — group names, GIDs, and member lists (world-readable). See [[Users_and_Groups]]. -->
<!-- #flashcards/devops/linux/users -->

<!-- Why is the password hash stored in `/etc/shadow` rather than `/etc/passwd`, given that `/etc/passwd` must be world-readable? -->
<!-- ? -->
<!-- Early Unix tools (`ls`, `mail`) need to map UIDs to usernames, so `/etc/passwd` must be readable by all. Leaving password hashes there made offline cracking trivial. `/etc/shadow` was introduced as a **root-only** file for hashes, leaving `/etc/passwd` world-readable for name lookups only. The `x` in passwd field 2 means "hash is in shadow." -->
<!-- #flashcards/devops/linux/users -->

<!-- What does the `x` in field 2 of a `/etc/passwd` entry mean? -->
<!-- ? -->
<!-- It is a **placeholder** indicating that the real password hash lives in `/etc/shadow`, not in `/etc/passwd`. It is not the hash itself. This split lets `/etc/passwd` stay world-readable without exposing hashes to offline cracking. -->
<!-- #flashcards/devops/linux/users -->

<!-- Why are UID 0, UIDs 1–999, and UIDs 1000+ significant, and what can go wrong when UIDs are reused after deletion? -->
<!-- ? -->
<!-- UID **0** is root. **1–999** are usually system/service accounts (daemon, www-data). **1000+** are regular humans. Reusing a UID after `userdel` (without `-r`) causes a new user to **unexpectedly inherit** the old user's files, because file ownership is by UID number, not name. Reserve retired UIDs or scrub files first. -->
<!-- #flashcards/devops/linux/users -->

## Primary vs supplementary groups

<!-- What is the difference between a user's primary group and supplementary groups, and where is each stored? -->
<!-- ? -->
<!-- The **primary group** is stored in `/etc/passwd` (4th field) and is the group used when the user **creates new files**. **Supplementary groups** are listed in `/etc/group` and grant additional memberships (e.g., `sudo`, `docker`) **without** affecting default file ownership. Same user, two different storage locations, two different effects. -->
<!-- #flashcards/devops/linux/users -->

<!-- Alice is in groups `alice`, `sudo`, `docker`. She creates a new file. Which group owns it, and why does this matter for collaboration? -->
<!-- ? -->
<!-- The **primary group** — `alice` — owns it (unless the parent directory has the setgid bit). This matters because teammates in `docker` cannot collaborate on her files unless she explicitly `chgrp`s them or the shared directory uses setgid to force group inheritance. -->
<!-- #flashcards/devops/linux/users -->

## The `-aG` trap — the destructive one-character mistake

<!-- What is the destructive difference between `usermod -G docker,ssh alice` and `usermod -aG docker,ssh alice`? -->
<!-- ? -->
<!-- Without `-a`, `-G` **replaces** Alice's entire supplementary group list with only `docker,ssh` — silently removing her from every other group, including `sudo`, potentially locking you out of admin access. With `-a` (append), the groups are **added** to her existing memberships. Always use `-aG`. This is one of the most destructive one-character mistakes in Linux administration. -->
<!-- #flashcards/devops/linux/users -->

<!-- You ran `usermod -G docker alice` and Alice can no longer use `sudo`. What happened, and how do you avoid it next time? -->
<!-- ? -->
<!-- `-G` without `-a` **replaced** her group list with only `docker`, stripping her from `sudo`. You are likely now locked out unless you have a root shell or another sudoer. Next time use `-aG` (append). Recovery requires root console/recovery mode. This is why the `-a` flag is not optional — it is the safety mechanism. -->
<!-- #flashcards/devops/linux/users -->

## Account lifecycle — creation and deletion

<!-- What is the difference between `useradd` and `adduser`, and which belongs in scripts? -->
<!-- ? -->
<!-- `useradd` is the **low-level, portable** command — minimal by default, needs explicit flags (`-m`, `-s`). `adduser` is an **interactive Perl wrapper** (Debian/Ubuntu only) that prompts for details and is friendlier for manual use. In scripts, always use `useradd` with explicit flags for predictability and cross-distro portability. -->
<!-- #flashcards/devops/linux/users -->

<!-- Why should you always pass `-m` to `useradd` in scripts, even if your current distro creates home dirs by default? -->
<!-- ? -->
<!-- Because the default depends on `/etc/login.defs` (`CREATE_HOME`), which differs across distros — Debian/Ubuntu's `useradd` does **not** create a home dir without `-m`, while RHEL/CentOS does. Explicit `-m` makes behavior **consistent across distros** and prevents broken logins (GUI sessions, `~/.config`-dependent tools). -->
<!-- #flashcards/devops/linux/users -->

<!-- Why is `userdel bob` without `-r` risky, and what problem surfaces later? -->
<!-- ? -->
<!-- The account is removed but the **home directory and files remain**, now owned by an unassigned UID. When a new user is later created with that same recycled UID, they **unexpectedingly inherit** bob's old files — a security and privacy breach. Use `-r` to remove the home dir, or archive it first. -->
<!-- #flashcards/devops/linux/users -->

<!-- Why might a freshly created user be unable to log in immediately, even though the account exists? -->
<!-- ? -->
<!-- A new `useradd` account has **no password set** — the account is locked (not empty, locked). Until you run `passwd <user>` (or add an SSH key to `~/.ssh/authorized_keys`), login is impossible. This is by design: account creation and credential assignment are separate steps. -->
<!-- #flashcards/devops/linux/users -->

## Locking accounts — the SSH-key gap

<!-- You lock alice's account with `usermod -L alice`. Is she now fully locked out? (Two scenarios.) -->
<!-- ? -->
<!-- **No, not necessarily.** `-L` only disables **password** login (prepends `!` to the hash). If alice has an SSH key in `~/.ssh/authorized_keys`, **key-based login still works**. To fully disable the account: also remove her authorized keys, set her shell to `/usr/sbin/nologin`, or expire the account with `chage -E 0 alice`. -->
<!-- #flashcards/devops/linux/users -->

<!-- What are three ways to fully disable a user's login (not just password login)? -->
<!-- ? -->
<!-- (1) Set the shell to `/usr/sbin/nologin` or `/bin/false` (`usermod -s /usr/sbin/nologin alice`). (2) Expire the account (`sudo chage -E 0 alice`). (3) Lock the password **and** remove `~/.ssh/authorized_keys`. Password-locking alone (`usermod -L`) leaves SSH-key login open — a common security gap. -->
<!-- #flashcards/devops/linux/users -->

## Safe editing — `vipw`/`vigr`/`visudo`

<!-- Why should you use `vipw` and `vigr` instead of editing `/etc/passwd` and `/etc/group` directly? -->
<!-- ? -->
<!-- They **acquire a file lock** to prevent concurrent edits and run **syntax validation** before committing. A malformed passwd/group file breaks logins system-wide. `vipw -s` edits `/etc/shadow` safely. If you must edit by hand, these are the only safe tools; otherwise use the dedicated commands (`useradd`, `usermod`, `groupadd`). -->
<!-- #flashcards/devops/linux/users -->

<!-- Why must you edit `/etc/sudoers` with `visudo` and never a plain text editor? -->
<!-- ? -->
<!-- `visudo` **validates the syntax** after saving and rejects the change if it would not parse, restoring the previous version. A malformed sudoers file breaks `sudo` entirely — and since you need sudo to fix it, you may be **locked out of root access**, requiring recovery mode. This is the single most common sudo disaster. -->
<!-- #flashcards/devops/linux/sudo -->

## `sudo` vs `su` — the accountability model

<!-- What three problems does `sudo` solve that `su -` does not? -->
<!-- ? -->
<!-- (1) **Fine-grained delegation** — sudoers rules allow specific users to run specific commands as specific users; `su` is all-or-nothing. (2) **Auditability** — sudo logs every invocation (who/what/when) to `/var/log/auth.log`; `su` does not. (3) **Temporary escalation** — one command, then privileges drop; `su` opens a full root session. -->
<!-- #flashcards/devops/linux/sudo -->

<!-- `sudo` asks for your password; `su` asks for root's. Why does this matter operationally, and what modern best practice does it enable? -->
<!-- ? -->
<!-- With `sudo`, the root password is **never shared** — every escalation is tied to a named user in the logs. This enables **disabling the root password entirely** (modern best practice) while still allowing delegated administration. With `su`, everyone who escalates must know the root password, defeating individual accountability. -->
<!-- #flashcards/devops/linux/sudo -->

## Sudoers rule syntax and the PATH-shadowing attack

<!-- What is the full form of a sudoers rule, and what does each field mean? -->
<!-- ? -->
<!-- `who host=(runas:group) commands`. **who** = user/`%group` allowed to escalate. **host** = which machine (enables one config for a fleet). **(runas:group)** = the user and group the command may execute as. **commands** = absolute paths of permitted binaries, or `ALL`. Optional: `NOPASSWD:` skips the password prompt. -->
<!-- #flashcards/devops/linux/sudo -->

<!-- Why must command paths in sudoers be absolute, and what attack does a relative path like `NOPASSWD: systemctl` enable? -->
<!-- ? -->
<!-- sudo resolves non-absolute names via the invoking user's `$PATH`. An attacker who can write to an early PATH directory can place a **malicious `systemctl`** there, and sudo executes it as root — full privilege escalation. Always use the absolute path (`/usr/bin/systemctl`), and specify arguments too to limit scope. -->
<!-- #flashcards/devops/linux/sudo -->

<!-- Why is `NOPASSWD: ALL` considered a security red flag, even though it still goes through sudo? -->
<!-- ? -->
<!-- It grants passwordless, unrestricted root execution. That is **effectively root with no audit gate at the moment of action** — any process or person who can reach the account has instant root without friction or contemporaneous logging intent. It collapses the three advantages of sudo (delegation, audit, friction) into nothing. -->
<!-- #flashcards/devops/linux/sudo -->

## Sudo shell modes and diagnostics

<!-- What is the difference between `sudo -i` and `sudo -s`, and when does the distinction matter? -->
<!-- ? -->
<!-- `sudo -i` starts a **login shell** — loads root's `.profile`, sets `$HOME` to `/root`, changes to root's home dir. `sudo -s` starts a root shell but **keeps your environment** (`$HOME` stays yours). The distinction matters for tools that read `$HOME` (shell rc files, `~/.config`, `~/.ssh`): `sudo -i` behaves like a real root login; `sudo -s` does not. -->
<!-- #flashcards/devops/linux/sudo -->

<!-- You run `sudo -l`. What does it tell you, and why is it the first diagnostic for sudo problems? -->
<!-- ? -->
<!-- It lists **the exact commands you are allowed to run via sudo**, including `NOPASSWD` entries and the users/groups you may run as. It is the first diagnostic because it shows the **effective policy for your user** — revealing whether a rule is missing, mis-scoped, or shadowed. If `sudo -l` doesn't show what you expect, the sudoers config is wrong. -->
<!-- #flashcards/devops/linux/sudo -->

<!-- What does `sudo -k` do, and when would you use it? -->
<!-- ? -->
<!-- It **resets the sudo timestamp**, forcing a password prompt on the next `sudo` invocation (otherwise cached for ~15 min). Use it before handing the machine to someone else, in scripts that should not reuse cached auth, or when you want to confirm a rule still prompts as expected. -->
<!-- #flashcards/devops/linux/sudo -->

## Debug-the-AI — find the planted bug

<!-- What's wrong with this sudoers rule: `bob ALL=NOPASSWD: systemctl restart nginx`? -->
<!-- ? -->
<!-- **Non-absolute command path.** sudo resolves `systemctl` via `$PATH`, enabling a **shadowing attack** — a malicious script named `systemctl` in an early PATH dir runs as root. Fix: `bob ALL=NOPASSWD: /usr/bin/systemctl restart nginx`. Specifying the absolute path (and the exact arguments) closes the hole. -->
<!-- #flashcards/devops/linux/debug -->

<!-- What's wrong with this sequence: `sudo usermod -G docker alice` then `sudo usermod -G ssh alice`? -->
<!-- ? -->
<!-- The second command **overwrites** the first — without `-a`, each `-G` **replaces** the full supplementary group list. Alice ends up in only `ssh`, having been silently removed from `docker` (and any other group, including `sudo`). The fix is `-aG` (append) in both commands. -->
<!-- #flashcards/devops/linux/debug -->

<!-- What's wrong with `sudo nano /etc/sudoers` to add a new privilege rule? -->
<!-- ? -->
<!-- **Editing sudoers without `visudo`.** A syntax error breaks `sudo` entirely and can **lock you out of root** (you need sudo to fix sudo). Use `sudo visudo` (validates on save), or better, a drop-in file: `sudo visudo -f /etc/sudoers.d/deploy`. -->
<!-- #flashcards/devops/linux/debug -->

<!-- What's wrong with `sudo userdel bob` on a departing employee's account, if bob had important work files? -->
<!-- ? -->
<!-- **No `-r` flag.** The account is removed but bob's home dir and files remain, now owned by an **orphaned UID**. When the UID is recycled for a new user, they inherit bob's files — a privacy/security breach. Fix: archive the home dir first, then `sudo userdel -r bob`. -->
<!-- #flashcards/devops/linux/debug -->

## Meta — what AI changes about this topic

<!-- In the AI era, which users-and-groups competences still matter, and which does AI safely absorb? -->
<!-- ? -->
<!-- **AI absorbs:** exact flags (`useradd`/`usermod`/`chage`), field positions in `/etc/passwd`, default config values — all lookups. **Still matters:** the *security model* (why shadow exists, why paths must be absolute, why `-aG` not `-G`), recognizing destructive operations before running them, and **auditing** AI-generated sudoers rules for over-granting. You can't approve a privilege rule you can't read critically. -->
<!-- #flashcards/devops/linux/meta -->

<!-- Why is "let the AI write the sudoers rule" risky without you understanding sudoers syntax? -->
<!-- ? -->
<!-- Because sudoers mistakes are **security holes and lockouts**: a typo'd `NOPASSWD: ALL` is a full privilege escalation; a non-absolute path enables shadowing attacks; a syntax error locks you out of root. You must be able to **read the rule critically** before installing it — AI generates plausible-but-dangerous configs confidently. Critical reading is the layer AI cannot give you. -->
<!-- #flashcards/devops/linux/meta -->
