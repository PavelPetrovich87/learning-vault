<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/devops/linux -->

# 🎴 Ubuntu / Debian Admin Commands — Flashcards

<!-- Source: [[Ubuntu_Commands]] · [[Users_and_Groups]] · [[Sudo_and_Sudoers]] · [[Linux_Package_Management]] -->

<!-- **Design note:** Per the AI-era flashcard principle in `AGENTS.md`, these cards target the **mental model** layer — the two recurring Debian/Ubuntu patterns (wrapper-over-core, available/enabled symlinks), the *why* behind `adduser` vs `useradd` in scripts, why `visudo` exists as a validator (not an editor), the apt/dpkg two-layer split, and the apache2 reload-vs-restart + 403-permissions reasoning. They deliberately **avoid flag-table recall** (full `useradd`/`apt`/`a2enmod` option lists) — those are lookups that live in the concept notes. Most cards are **debug-the-symptom** and **predict-the-outcome** drills, because the competence they build — *reasoning about the Debian config model and the low-level/wrapper split* — is the layer AI cannot install in your head. -->

## The two Debian/Ubuntu patterns

<!-- What are the two recurring design patterns in Debian/Ubuntu command/config tooling? Give one example of each. -->
<!-- ? -->
<!-- (1) **Low-level core + friendly wrapper:** a powerful-but-terse core tool with a human-friendly layer on top. Examples: `dpkg` (unpacks .deb, no dep resolution) wrapped by `apt` (repos + deps); `useradd` (minimal, explicit flags) wrapped by `adduser` (interactive). Learn the wrapper for daily use, the core for scripting/debugging. (2) **Available/enabled symlink split:** config ships in `*-available/` and is activated by symlinking into `*-enabled/`, via `a2ensite`/`a2enmod`/`a2enconf` helpers. Apache reads only `*-enabled/` at startup. These two patterns recur across package, user, and web-server management on Debian/Ubuntu. -->
<!-- #flashcards/devops/linux/ubuntu -->

## useradd vs adduser — the wrapper-over-core split

<!-- Why should a shell script use `useradd` (not `adduser`) to create an account? What two flags must you almost always pass, and how do you set the password non-interactively? -->
<!-- ? -->
<!-- `adduser` is **interactive** — it prompts for a password and details, which blocks or fails unattended in a script. `useradd` is non-interactive and scriptable. Almost always pass **`-m`** (create home dir — Debian's `useradd` does *not* by default, unlike RHEL where `CREATE_HOME yes` is set) and **`-s /bin/bash`** (login shell — default may be `/bin/sh` or invalid). Set the password non-interactively with `echo 'bob:secret' | chpasswd`. Result: deterministic, portable, unattended account creation. -->
<!-- #flashcards/devops/linux/ubuntu -->

<!-- A teammate runs `sudo useradd charlie` on Ubuntu and then complains "the account has no home directory." Why did that happen, and what does the same command do on RHEL/Fedora? -->
<!-- ? -->
<!-- On **Debian/Ubuntu**, `useradd` does **not** create a home directory by default — you must pass **`-m`** explicitly. On **RHEL/Fedora**, `/etc/login.defs` has `CREATE_HOME yes`, so `useradd charlie` *does* create the home dir automatically. This is a distro default difference that trips people moving between families. The portable fix: **always pass `-m`** in scripts and manual use so behavior is identical regardless of distro. (And `useradd` never sets a password — run `passwd charlie` or `chpasswd` afterward.) -->
<!-- #flashcards/devops/linux/ubuntu -->

## visudo — the validator, not the editor

<!-- "visudo" sounds like "vi for sudo." What does it actually do that a normal editor doesn't, and what disaster does that prevent? -->
<!-- ? -->
<!-- `visudo` is a **validator**, not just an editor. After you save, it **parses `/etc/sudoers` and rejects the change if it wouldn't parse**, restoring the previous working version. It also locks against concurrent edits and enforces mode `0440`. The disaster it prevents: `sudo` reads `/etc/sudoers` on *every* invocation, so a **syntax error** (stray character, malformed rule) breaks `sudo` for *everyone* — and since you need sudo to fix the file, you're **locked out of root**, requiring a reboot to recovery mode. Editing with `nano`/`vim` directly has no such safety net. -->
<!-- #flashcards/devops/linux/ubuntu -->

<!-- You break `/etc/sudoers` by editing it with `nano` directly, and now `sudo` rejects every command with a parse error. Why are you locked out, and how do you recover? -->
<!-- ? -->
<!-- You're locked out because `sudo` needs to read `/etc/sudoers` to authorize *any* sudo command — including the one that would fix the file. With the file broken, there's no path to root via sudo. Recovery: reboot into **recovery mode / a live USB / single-user mode**, where you get a root shell *without* sudo, then repair the file (and run `visudo` to validate before rebooting). Prevention: **always use `visudo`**, which validates on save and would have rejected your bad edit. This is the canonical sudo disaster — avoid it entirely with visudo. -->
<!-- #flashcards/devops/linux/ubuntu -->

## dpkg vs apt — the two-layer model

<!-- You run `dpkg -i newpkg.deb` and it fails with "dependency problems." Why couldn't `dpkg` resolve them itself, and what are two correct fixes? -->
<!-- ? -->
<!-- `dpkg` is the **low-level** tool — it unpacks a `.deb` and records files, but it has **no concept of repositories or dependency resolution**. It can *detect* a missing dep but can't *fetch* it. Fixes: (1) **`sudo apt install ./newpkg.deb`** — apt resolves and downloads the missing deps from configured repos, then hands everything to dpkg to install. (2) Run `dpkg -i newpkg.deb` (fails), then **`sudo apt-get -f install`** ("fix broken install") — apt's resolver satisfies the unmet deps. This is the two-layer model: dpkg does mechanics, apt does the dependency graph. For a local .deb, prefer `apt install ./file.deb`. -->
<!-- #flashcards/devops/linux/ubuntu -->

<!-- Why does `dpkg -S /usr/bin/curl` work to find "which package owns this file?" when `apt` has no equivalent query? -->
<!-- ? -->
<!-- Because **`dpkg` owns the package database** (`/var/lib/dpkg/`) — the record of every file every installed package placed on the system. `apt` is the high-level front-end (repos, deps, downloads); once packages are installed, it hands off to dpkg, and the on-disk database is dpkg's. So ownership queries (`dpkg -S`), file-listing (`dpkg -L pkg`), and the full installed inventory (`dpkg -l`) all live at the dpkg layer. The split: `apt` knows about the *catalog* (what's available), `dpkg` knows about the *installation* (what's on disk and where). -->
<!-- #flashcards/devops/linux/ubuntu -->

## apt — update vs upgrade, the #1 confusion

<!-- What's the difference between `apt update` and `apt upgrade`? Which one installs nothing? -->
<!-- ? -->
<!-- **`apt update` installs nothing** — it only **refreshes the local package lists** (the "what's available" metadata cached in `/var/lib/apt/lists/`) by querying each configured repository. **`apt upgrade`** is what actually **installs newer versions** of already-installed packages. The required ritual: run `update` first so `upgrade` (and `install`) know the latest versions; skipping `update` means you may install a version that was current when the lists were *last fetched* — potentially days or weeks stale. In scripts, use **`apt-get`** (stable output) over **`apt`** (interactive formatting). -->
<!-- #flashcards/devops/linux/ubuntu -->

## tree — the structure-reading tool

<!-- When do you reach for `tree` vs `ls -R` vs `find`? What single flag should you almost always pass to `tree`? -->
<!-- ? -->
<!-- **`tree`** = understand/communicate *structure* — renders nesting, siblings, and a directory/file count summary at a glance (onboarding, docs, eyeballing a layout). **`ls -R`** = flat recursive listing with repeated path headers — readable but loses the *shape*. **`find`** = act on matches (move/grep/chmod via `-exec`) — `tree` has no `-exec`, it's a reading tool. Almost always pass **`-L N`** to cap depth, or a big tree like `tree /usr` drowns you in output. `tree -L 2 /etc/apache2` is a two-line mental map of the whole config layout. `tree -d` (dirs only), `-a` (hidden), `-pug` (perms/owner/group) are handy too. -->
<!-- #flashcards/devops/linux/ubuntu -->

## apache2 — the Debian config layout

<!-- Why does Debian/Ubuntu split apache config into `sites-available/` and `sites-enabled/` (and `mods-available`/`mods-enabled`) instead of one big config file? How does a site move between them? -->
<!-- ? -->
<!-- The **available/enabled split** lets you *stage* config without activating it: `*-available/` holds every definition (shipped, off by default); apache reads only `*-enabled/` at startup. Activating is a **symlink** from available into enabled, created by the **`a2ensite mysite`** helper (removed by `a2dissite mysite`; same pattern for `a2enmod`/`a2dismod`, `a2enconf`/`a2disconf`). Benefits: you can keep configs staged but disabled, see at a glance what's live, and toggle without deleting anything. It's literate `ln -s` management — and nginx copied this exact pattern. Remember: moving the symlink does **not** notify the running server — you must `systemctl reload apache2`. -->
<!-- #flashcards/devops/linux/ubuntu -->

<!-- You run `sudo a2enmod rewrite`. The command prints "To activate the new configuration, you need to run: systemctl restart apache2." You skip that and test your site — rewriting still doesn't work. Diagnose. -->
<!-- ? -->
<!-- `a2enmod` only **created the symlink** `mods-enabled/rewrite.load → ../mods-available/rewrite.load`. The **running apache process** still has its old config in memory — apache reads `mods-enabled/` only at **startup or on reload**. Until you run **`sudo systemctl reload apache2`** (graceful) or `restart`, the module isn't loaded into the live server. That's why `a2enmod` itself *prints* the reload instruction. The fix is literally what the command told you. Always pair every `a2en*`/`a2dis*` with a reload. -->
<!-- #flashcards/devops/linux/ubuntu -->

## apache2 — reload vs restart, and the 403 trap

<!-- `systemctl reload apache2` vs `systemctl restart apache2` — what's the difference, and which do you use for a routine config change on a live site? -->
<!-- ? -->
<!-- **`reload`** sends SIGHUP: apache **re-reads config without stopping** — in-flight requests finish, no dropped connections; graceful. **`restart`** stops the service entirely and starts it again — a brief window where no requests are served; a blip. For a routine config change (new vhost, enabled module) on a live site, use **`reload`** so visitors aren't interrupted. Reserve **`restart`** for changes reload can't honor (switching MPM, killing a wedged worker). Same reload-vs-restart distinction applies to every systemd service. Always `apachectl configtest` before either, to avoid reloading broken config. -->
<!-- #flashcards/devops/linux/ubuntu -->

<!-- You visit `http://server/` and get **403 Forbidden**. Apache is running, the vhost points at `/var/www/html/`, and the file exists. What's the most likely cause, and how do you confirm and fix it? -->
<!-- ? -->
<!-- **File permissions — apache can't read the file.** Apache runs as the **`www-data`** user; if files aren't readable by `www-data` (wrong owner, missing read bit) or any parent dir lacks execute (`o+x`), apache returns 403. This is a *filesystem* problem, not an apache config problem. Confirm: `ps -u www-data` (verify apache's user), `ls -la /var/www/html/`, and **`namei -m /var/www/html/index.html`** (prints perms on *every* path component — a missing `o+x` on a parent dir is the classic hidden culprit). Fix: `chown -R www-data:www-data /var/www/html` or ensure `o+rx` on files and `o+x` on dirs. Lesson: web 403s are usually permissions, not config. -->
<!-- #flashcards/devops/linux/ubuntu -->

<!-- Why does apache run as `www-data` instead of root, and what's the security implication for the files under `/var/www/html/`? -->
<!-- ? -->
<!-- Apache starts as **root only to bind ports 80/443** (privileged ports require root), then **drops privileges to `www-data`** for all request handling — least privilege, so a compromised web process isn't root. Implication: the files under `/var/www/html/` must be **readable by `www-data`** for apache to serve them, but should **not be writable by `www-data`** unless uploads require it — a compromised web process shouldn't be able to rewrite the site it serves or plant malicious files. Verify the running user with `ps -u www-data`; set ownership/permissions accordingly ([[Linux_Permissions]]). -->
<!-- #flashcards/devops/linux/ubuntu -->

## Predict-the-outcome drills

<!-- On a fresh Ubuntu box you run: `sudo useradd -m -s /bin/bash dave` (no `passwd`, no `chpasswd`). Predict what happens when dave tries to SSH in. -->
<!-- ? -->
<!-- The account is created with a home dir (`-m`) and bash shell (`-s`), but **no password is set** — `useradd` does not prompt for one. The account's password field in `/etc/shadow` is `!` (locked) or `*` (disabled), so **password login fails**. Whether SSH key login works depends on whether you placed a key in `~dave/.ssh/authorized_keys` (you didn't, here). The fix: run **`sudo passwd dave`** (interactive) or **`echo 'dave:secret' | sudo chpasswd`** (scriptable). Lesson: `useradd` alone is an incomplete account setup — pair it with a password step. -->
<!-- #flashcards/devops/linux/ubuntu -->

<!-- Trace the layers: you run `sudo apt install apache2`. Which tools actually do the work, and in what order? -->
<!-- ? -->
<!-- (1) **`apt`** queries the configured repositories, resolves the dependency graph (apache2 depends on apache2-utils, ssl libs, the chosen MPM, etc.), and downloads the signed `.deb` files. (2) For each package, apt hands it to **`dpkg`** (the low-level core), which **unpacks** the `.deb`, places files in the right paths (`/usr/sbin/apache2`, `/etc/apache2/`, `/var/www/html/`), runs maintainer scripts, and records them in **`/var/lib/dpkg/`**. (3) The apache package's postinst script **enables + starts** the service via systemd. So apt = catalog + deps + download; dpkg = mechanics + database; systemd = runtime supervision. The two-layer (apt over dpkg) plus the service model, all in one `apt install`. -->
<!-- #flashcards/devops/linux/ubuntu -->
