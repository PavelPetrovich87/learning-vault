---
status: archived
tags: [devops, linux, ubuntu, debian, commands, apache2, udemy]
created: 2026-07-15
updated: 2026-10-21
source: Udemy DevOps Course — Linux Fundamentals
difficulty: intermediate
---

# 🧠 Topic: Ubuntu / Debian Admin Commands

> [!abstract] Executive Summary (ELI5)
> **Ubuntu** is a pre-furnished Linux "apartment" — the kernel plus sensible defaults, a curated app store, and helper commands that make common admin tasks friendly. This note is a **hub**: it introduces the six commands the course groups together as "Ubuntu essentials," links the four that already have dedicated concept notes, and gives `tree` and `apache2` their full treatment here. The through-line is the **Debian/Ubuntu signature pattern** — *low-level tool + friendly wrapper* (`useradd`/`adduser`, `dpkg`/`apt`) and the *available/enabled symlink model* (`a2ensite`, `a2enmod`) — which shows up across package, user, and web-server management.

> [!note] Why these six are one note (and how to use it)
> The course bundles these as "Ubuntu commands," but they span four domains: **user accounts** (`useradd`/`adduser`), **privilege** (`visudo`), **packages** (`dpkg`/`apt`), **visualization** (`tree`), and **web serving** (`apache2`). Rather than duplicate the four that already have deep notes, this is a **hub**: each section gives you the one mental model you need, then links out for depth. Only `tree` and `apache2` — which have no dedicated note yet — are covered in full here. Jump to the section you need; the links are the point.

---

## 🎯 The "Why"
- **Problem Solved:** A Linux distribution is more than a kernel — it's a set of conventions, default configs, and helper tools that make the machine *inhabitable*. Ubuntu/Debian adds four things raw Linux lacks:
  1. **Friendly wrappers over low-level tools** — `adduser` wraps `useradd`, `apt` wraps `dpkg`, so common tasks are safe and ergonomic without losing power-user access to the raw tool underneath.
  2. **A curated, signed package repository** — one `apt install` fetches verified software and resolves dependencies (see [[Linux_Package_Management]]).
  3. **A structured web-server config layout** — `/etc/apache2/` with `sites-available`/`sites-enabled` and `mods-available`/`mods-enabled`, managed by `a2en*`/`a2dis*` tools. This is *the* Debian signature, copied by nginx.
  4. **A controlled privilege model** — `sudo` + `visudo` instead of a free-for-all root login.
- **Alternatives:** Other families do the same jobs differently — Red Hat/Fedora use `rpm`/`dnf`, `httpd` (single `httpd.conf` + `conf.d/`), and add users via `useradd` with `CREATE_HOME yes` by default. The *concepts* are portable; the *command names and config layouts* are distro-specific.
- **Trade-offs:** Ubuntu's friendliness (wrappers, interactive prompts, sensible defaults) is great for humans and fast setup, but can hide what's really happening and bite you in scripts where you need deterministic, non-interactive behavior (always use `useradd` with explicit flags, `apt-get` with `-y`, `DEBIAN_FRONTEND=noninteractive`).

## 💡 Mental Model & Analogy
- **Analogy:** Ubuntu is a **managed apartment building**. `dpkg`/`useradd` are the **raw building services** — plumbing and wiring that work but expect you to know what you're doing. `apt`/`adduser` are the **concierge** — you say "get me a desk" or "set up a new tenant" and it handles the details (delivery, assembly, passwords, skeleton files). `visudo` is the **building manager who checks your keycard config before it goes live**. `a2ensite`/`a2enmod` are the **light switches** in a breaker panel labeled available/enabled — the wiring exists in `*-available/`, you flip a symlink to turn it on in `*-enabled/`. `tree` is the **floor plan** that shows the whole layout at a glance. `apache2` is the **building's reception desk** — it greets every visitor (HTTP request) and routes them to the right office (virtual host / document root).
- **Visual/Diagram — the Debian/Ubuntu signature: wrapper-over-core + available/enabled:**

```mermaid
graph TD
    subgraph WRAP["Wrapper-over-core (human-friendly layer)"]
        AU["adduser (interactive)"] -->|calls| UA[useradd -m ...]
        APT["apt (deps + repos)"] -->|hands .deb to| DPKG[dpkg -i]
    end
    subgraph AVAIL["available/enabled symlink model (apache2, nginx)"]
        SA["sites-available/<br/>000-default.conf"] -->|a2ensite: symlink| SE["sites-enabled/<br/>000-default.conf"]
        MA["mods-available/<br/>rewrite.load"] -->|a2enmod: symlink| ME["mods-enabled/<br/>rewrite.load"]
        SE --> READ[apache2 reads sites-enabled/*<br/>at startup]
        ME --> READ
    end
    VIS[visudo] -->|validates syntax| SUD[/etc/sudoers<br/>rejects bad edits]
```

> [!tip] The single mental model that ties Ubuntu together
> **Debian/Ubuntu reaches for two patterns again and again:** (1) a *low-level core* (`dpkg`, `useradd`) with a *friendly wrapper* (`apt`, `adduser`) on top — learn the wrapper for daily use, the core for scripting and debugging; (2) the *available/enabled symlink split* — config ships in `*-available/` and is activated by symlinking into `*-enabled/`, managed by `a2en*`/`a2dis*` helpers. Once you internalize these two, the apache2, nginx, and apt config worlds all look familiar.

---

## 🛠️ The Six Commands

### 1. `useradd` vs `adduser` — low-level core vs interactive wrapper
```bash
sudo useradd -m -s /bin/bash -c "Bob Jones" bob    # low-level: explicit flags, scriptable
sudo passwd bob                                     # useradd does NOT set a password — you must
sudo adduser bob                                    # wrapper: prompts for password, makes home, copies skeleton
```
- **`useradd`** is the portable, low-level core — minimal by default. On Debian/Ubuntu it does **not** create a home dir unless you pass `-m`, and it never sets a password (you run `passwd` separately).
- **`adduser`** is a Debian/Ubuntu-only interactive Perl wrapper — it prompts for the full name, password, creates `~bob`, copies `/etc/skel/` skeleton files, and is friendlier for one-off manual account creation. Not present on RHEL/Fedora.
- **In scripts:** always `useradd` with explicit `-m -s` — predictable, non-interactive, portable.
- 👉 **Full treatment (defaults, `-m`, `/etc/skel`, all flags):** [[Users_and_Groups]] §3.

### 2. `visudo` — the only safe editor for `/etc/sudoers`
```bash
sudo visudo                       # edit /etc/sudoers with syntax validation
sudo visudo -f /etc/sudoers.d/deploy   # edit a drop-in file safely
```
- `visudo` is **not** "vi for sudo" — it's a *validator*. It parses the file after you save and **rejects syntax errors**, restoring the previous version if the new one wouldn't parse. A broken `/etc/sudoers` disables `sudo` entirely, and since you need sudo to fix it, you can lock yourself out of root.
- Always use it instead of editing the file with `vim`/`nano` directly.
- 👉 **Full treatment (rule syntax, NOPASSWD, `sudoers.d/` drop-ins, precedence):** [[Sudo_and_Sudoers]] §3.

### 3. `dpkg` — the low-level package tool (unpacks `.deb`)
```bash
dpkg -i package.deb         # install a local .deb (will FAIL on missing deps)
dpkg -r nginx               # remove
dpkg -l                     # list all installed packages
dpkg -L nginx               # what FILES did this package install?
dpkg -S /usr/bin/curl       # which package owns this file?
```
- `dpkg` does the **mechanical** work — unpack, place files, record in the database (`/var/lib/dpkg/`). It has **no idea** about the internet, repositories, or dependencies. Install a `.deb` whose deps aren't met and `dpkg -i` errors out; you then run `apt-get -f install` to resolve.
- This is the *low-level core* under `apt`. You rarely invoke it — but `dpkg -S` / `dpkg -L` are irreplaceable for "which package owns/installed this file?"
- 👉 **Full treatment (the two-layer model, dpkg vs rpm, the database):** [[Linux_Package_Management]] §1–2.

### 4. `apt` — the high-level front-end (deps + repositories)
```bash
sudo apt update             # refresh package lists from repos (NOT an upgrade!)
sudo apt install nginx      # install + resolve dependencies
sudo apt upgrade            # upgrade installed packages
sudo apt remove nginx       # remove (keeps config); add --purge for config too
apt search nginx            # search; apt show nginx for details
```
- `apt` is the **concierge**: it talks to repositories, downloads signed packages, resolves the dependency graph, then hands each one to `dpkg` to install.
- ⚠️ The #1 confusion: **`apt update` does not install anything** — it only refreshes the local "what's available" lists. `apt upgrade` does the actual upgrading. Always `update` before `upgrade`.
- In scripts, prefer **`apt-get`** (stable, scriptable output) over `apt` (interactive formatting).
- 👉 **Full treatment (repos, the update/upgrade split, autoremove, pinning):** [[Linux_Package_Management]] §2.

### 5. `tree` — visualize a directory as a nested ASCII tree *(full treatment)*
```bash
sudo apt install tree          # not installed by default on minimal systems
tree                           # draw the tree of the current directory
tree /etc/apache2              # draw from a specific path
tree -L 2                      # limit depth to 2 levels ⭐
tree -d                        # directories only (skip files)
tree -a                        # include hidden files (dotfiles)
tree -f                        # print full path prefix for each entry
tree -pug                      # permissions + owner + group per entry
tree -h                        # human-readable file sizes
tree --dirsfirst -L 3          # dirs first, capped depth — great for docs
```
- **What it is:** a recursive directory-listing tool that renders the filesystem as an **indented ASCII tree** with `├──`/`└──` connectors — instantly readable structure that `ls -R` (flat, repeated headers) cannot match.
- **Why it matters:** it's the fastest way to *understand* or *document* a layout — a project's folder structure, a config tree like `/etc/apache2/`, a deployment directory. The visual hierarchy is the whole point; you read it at a glance.
- **`tree` vs `ls -R`:** `ls -R` dumps a flat, repetitive listing (reprints the path header before each subdir's contents); `tree` shows the *shape* — nesting depth, sibling relationships, and a summary count at the end (`X directories, Y files`). For anything deeper than two levels, `tree` wins decisively.
- **Composes with output:** `tree -J` emits JSON, `tree -X` emits XML, `tree -H .` emits an HTML directory listing — handy for generating docs or feeding other tools.
- **Depth control is the key habit:** always pass **`-L N`** on big trees, or you'll drown in output. `tree -L 2 /etc` is a two-line mental map of the whole config layout.

> [!tip] `tree` is a reading tool, not just a listing tool
> Reach for `tree` when you want to *understand* or *communicate* structure (onboarding a teammate, documenting a layout, eyeballing what a package installed under `/etc/`). Reach for `ls`/`find` when you want to *do* something with the matches (move them, grep them, chmod them). `tree` is optimized for human eyes; it has no `-exec`. There's a reason every "project structure" section of a README looks like `tree` output — half the time it literally is.

### 6. `apache2` — the Debian/Ubuntu web server *(full treatment)*
```bash
sudo apt install apache2                    # installs + enables + starts the service
systemctl status apache2                    # is it running?
ls /etc/apache2/                            # the Debian config layout (see below)
sudo a2enmod rewrite && systemctl reload apache2   # enable a module, reload config
sudo a2ensite mysite && systemctl reload apache2   # enable a virtual host
```
- **What it is:** the **Apache HTTP Server** — the classic open-source web server. On Debian/Ubuntu the package and service are named **`apache2`**; on RHEL/Fedora the same software is **`httpd`**. Same server, different packaging name and config layout. Apache's job: listen on a port (80/443), receive HTTP requests, and serve content (files from a *document root*, or output from scripts/modules) in response.
- **The Debian/Ubuntu config layout** — this is the part worth memorizing, and it's the canonical example of the available/enabled symlink model:

| Path | Purpose |
|------|---------|
| `/etc/apache2/apache2.conf` | Main config — global directives, includes everything else |
| `/etc/apache2/ports.conf` | What ports to listen on (`Listen 80`, `Listen 443`) |
| `/etc/apache2/sites-available/` | **All** virtual host definitions (ship here, off by default) |
| `/etc/apache2/sites-enabled/` | Symlinks to the *active* sites — apache reads only these |
| `/etc/apache2/mods-available/` | **All** modules (each is a `.load` + optional `.conf`) |
| `/etc/apache2/mods-enabled/` | Symlinks to the *active* modules |
| `/etc/apache2/conf-available/` `/conf-enabled/` | Same pattern for miscellaneous config snippets |
| `/var/www/html/` | Default **document root** (where your web files live) |
| `/var/log/apache2/` | `access.log` and `error.log` |

- **The `a2en*` / `a2dis*` helpers create and remove those symlinks** — they are *literate* wrappers over `ln -s` so you don't fat-finger paths:
  - `a2ensite mysite` / `a2dissite mysite` — enable/disable a virtual host
  - `a2enmod rewrite` / `a2dismod rewrite` — enable/disable a module
  - `a2enconf security` / `a2disconf security` — enable/disable a config snippet
  - **Every one of these requires a `sudo systemctl reload apache2` afterward** to take effect — they only move the symlink; they don't notify the running server.
- **Virtual hosts** — hosting multiple websites on one server. You drop a config in `sites-available/`, `a2ensite` it, reload. Apache picks which site to serve based on the `Host:` HTTP header (name-based) or the IP/port. This is how one box serves `site-a.com` *and* `site-b.com`.
- **Config validation before reload** — always test before reloading a live server:
  ```bash
  apachectl configtest        # or: apache2ctl configtest
  # → "Syntax OK" means safe to reload
  sudo systemctl reload apache2   # graceful: re-reads config without dropping connections
  sudo systemctl restart apache2  # hard: stops + starts (brief interruption)
  ```
  `reload` vs `restart` matters here exactly as it does for any service ([[Linux_Services_and_Systemd]]): reload is graceful, restart is a brief blip.
- **Apache runs as a service under systemd** — so `systemctl start/stop/restart/reload/status/enable apache2` all work, and `systemctl status apache2` shows the Main PID and the cgroup of worker processes (see [[Linux_Processes]]). It runs as the **`www-data`** user (least privilege — see [[Users_and_Groups]]), which is *why* your web files must be readable by `www-data`.
- **MPMs (Multi-Processing Modules)** — *how* Apache handles many concurrent requests: `prefork` (one process per request, safe for old non-thread-safe code), `worker`/`event` (threads, far more efficient, the modern default). Selected the same available/enabled way (`mpm_event.*` in `mods-enabled/`). You rarely change this, but it's why `systemctl status apache2` shows a master process plus many worker processes/threads.

> [!warning] The two apache2 gotchas
> 1. **`a2ensite`/`a2enmod` do NOT reload apache.** They only create the symlink. You must run `sudo systemctl reload apache2` (or `restart`) for the change to take effect. Forgetting this is the #1 "I enabled the module but nothing changed" mistake.
> 2. **Permissions:** apache runs as **`www-data`**. If your files under `/var/www/html/` aren't readable by `www-data` (wrong owner, no read bit), apache returns **403 Forbidden**. The fix is file permissions ([[Linux_Permissions]]), not an apache config change — `chown -R www-data:www-data` or ensure `o+rx` on the files and `o+x` on every parent directory.

## ⚠️ Pitfalls & Anti-patterns
- **Common Mistakes:**
  - Using **`adduser` in a script** — it's interactive (prompts for password/details), so it blocks or fails unattended. Use `useradd` with explicit `-m -s` flags + `chpasswd` for non-interactive account creation.
  - **Editing `/etc/sudoers` with a normal editor** instead of `visudo` — one syntax error and `sudo` is dead, and you need sudo to fix it. Always `visudo`.
  - Confusing **`apt update`** (refresh lists, installs nothing) with **`apt upgrade`** (actually upgrades). Always run `update` first.
  - **Running `dpkg -i` on a package with unmet deps** and being surprised it fails — `dpkg` has no dependency resolver. Use `apt install ./file.deb` instead (apt resolves the deps), or follow with `apt-get -f install`.
  - **`a2ensite`/`a2enmod` without `systemctl reload apache2`** — the symlink is created but the running server doesn't know. Always reload.
  - **`tree` with no `-L` on a huge tree** (`tree /usr`) — thousands of lines of output. Always cap depth with `-L N`.
  - **403 Forbidden from apache** and assuming it's an apache config bug — check that `www-data` can actually *read* the files (permissions/ownership on the document root and every parent dir).
  - **`restart` instead of `reload`** on a busy apache — restart drops in-flight requests; reload is graceful. Use reload for config changes.
- **When NOT to use apache2:**
  - For **modern high-concurrency reverse-proxy / static-content** workloads, **nginx** is often lighter and faster (event-driven by design, vs apache's MPM model). Apache still excels for dynamic content per-request via `.htaccess` and its huge module ecosystem.
  - For **app serving in containers** — package your app; don't run a full apache in the container unless it's specifically the web tier. The container runtime supervises it.
- **Security considerations:**
  - **Never run apache as root** — it drops to `www-data` after binding port 80/443 (which require root to bind). Verify with `ps -u www-data`.
  - **`/var/www/html/` should not be writable by `www-data`** unless uploads require it — readable is enough. A compromised web process shouldn't be able to rewrite the site it serves.
  - **Disable directory listing** (`Options -Indexes`) so an empty dir doesn't dump its file list to the web.
  - **Keep it patched:** `sudo apt update && sudo apt upgrade apache2` — web servers are the most attacked service on the internet.

## 🔗 Deep Connections
- **Parent Topic:** [[Linux_Commands]] — this is a hub in the command family.
- **User management:** [[Users_and_Groups]] (the `useradd`/`adduser` deep dive), [[Root_User_vs_Regular_User]].
- **Privilege:** [[Sudo_and_Sudoers]] and [[Sudo]] (the `visudo` deep dive).
- **Packages:** [[Linux_Package_Management]] (the `apt`/`dpkg` two-layer model).
- **Services & processes:** [[Linux_Services_and_Systemd]] (`systemctl reload apache2`, `enable`/`start`), [[Linux_Processes]] (apache's master + workers, signaling).
- **Permissions & files:** [[Linux_Permissions]] (the 403/`www-data` fix), [[Linux_Important_Directories]] (`/var/www/`, `/etc/apache2/`, `/var/log/apache2/`).

## ❓ Active Recall Questions
> [!faq]- What are the two recurring patterns in Debian/Ubuntu command design? Give one example of each.
> (1) **Low-level core + friendly wrapper:** `dpkg` (unpacks .deb, no dep resolution) wrapped by `apt` (repos + deps); `useradd` (minimal, explicit flags) wrapped by `adduser` (interactive). Learn the wrapper for daily use, the core for scripting and debugging. (2) **Available/enabled symlink split:** config ships in `*-available/` (sites, mods, confs) and is activated by symlinking into `*-enabled/`, managed by `a2ensite`/`a2enmod`/`a2enconf` helpers. Apache reads only `*-enabled/` at startup. Both patterns appear across package, user, and web-server management on Debian/Ubuntu.

> [!faq]- You run `a2enmod rewrite` and reload apache, but URL rewriting still doesn't work. You realize you ran `a2enmod` but forgot a step. What did you forget, and why does order matter?
> You forgot **`sudo systemctl reload apache2`** (or restart). `a2enmod` only **creates the symlink** from `mods-available/rewrite.load` into `mods-enabled/` — it does not notify the running server. Apache reads `mods-enabled/` only at **startup or on reload**, so until you reload, the running process is unaware the module is active. Always pair every `a2en*`/`a2dis*` with a `reload` (graceful) or `restart` (brief blip).

> [!faq]- Why should you use `useradd` (not `adduser`) in a shell script, and what two flags must you almost always pass?
> `adduser` is **interactive** — it prompts for a password and GECOS details, which blocks or fails in a non-interactive script. `useradd` is non-interactive and scriptable. Always pass **`-m`** (create home dir — Debian's `useradd` does *not* by default, unlike RHEL) and **`-s /bin/bash`** (set login shell — the default may be `/bin/sh` or invalid). Then set the password non-interactively with `echo 'bob:secret' | chpasswd`. Result: deterministic, portable, unattended account creation.

> [!faq]- Why does editing `/etc/sudoers` with `nano` directly risk locking you out, and how does `visudo` prevent it?
> `sudo` reads `/etc/sudoers` on every invocation. A **syntax error** in the file (a stray character, malformed rule) makes sudo reject the *entire* file — so `sudo` stops working for everyone. And since you need sudo to fix the file, you're locked out of root, requiring reboot-to-recovery-mode to repair. `visudo` **parses the file after you save** and **rejects the change if it wouldn't parse**, restoring the previous working version. It also locks the file against concurrent edits and enforces mode `0440`. That validation is the entire reason it exists — it's a validator, not "vi for sudo."

> [!faq]- A `dpkg -i newpkg.deb` fails with dependency errors. Why couldn't `dpkg` handle this itself, and what's the right fix?
> `dpkg` is the **low-level** tool — it unpacks a `.deb` and records files, but it has **no concept of repositories or dependency resolution**. It can detect that a dep is missing, but it can't fetch it. The fix: use **`apt install ./newpkg.deb`** (apt resolves and downloads the missing deps from repos, then hands everything to dpkg), or run `dpkg -i` then **`sudo apt-get -f install`** ("fix broken install") which uses apt's resolver to satisfy the unmet deps. This is the two-layer model: dpkg does the mechanics, apt does the dependency graph.

> [!faq]- You visit `http://server/` and get **403 Forbidden**, but apache is running and the vhost points at `/var/www/html/`. What's the most likely cause, and how do you confirm it?
> **File permissions.** Apache runs as the **`www-data`** user; if your files aren't readable by `www-data` (wrong owner, missing read bit), or if any parent directory lacks execute (`o+x`) permission, apache can't access them and returns 403 — not an apache config issue. Confirm: `ps -u www-data` (verify apache's user), then check `ls -la /var/www/html/` and `namei -m /var/www/html/index.html` (shows perms on *every* path component). Fix: `chown -R www-data:www-data` or ensure `o+rx` on files and `o+x` on dirs. The lesson: web 403s are usually a filesystem-permissions problem, not a web-server problem.

> [!faq]- When would you reach for `tree` instead of `ls -R` or `find`? What's the key flag you should almost always pass?
> Use `tree` when you want to **understand or communicate structure** — onboarding, documenting a layout, eyeballing what got installed under a directory. It renders the *shape* (nesting, siblings, a directory/file count summary) that flat `ls -R` can't. Use `ls`/`find` when you want to **act on matches** (move/grep/chmod them) — `tree` is a reading tool with no `-exec`. Always pass **`-L N`** to cap depth, or a big tree (`tree /usr`) drowns you in output. `tree -L 2 /etc/apache2` is a two-line mental map of the whole config layout.

> [!faq]- `systemctl reload apache2` vs `systemctl restart apache2` — what's the difference, and when does each matter?
> **`reload`** sends a signal (SIGHUP) telling apache to **re-read its config without stopping** — in-flight requests finish, no dropped connections; graceful. **`restart`** stops the service entirely and starts it again — there's a brief window where no requests are served; a blip. For routine config changes (new vhost, new module), use **`reload`** so a live site isn't interrupted. Use **`restart`** only when reload is insufficient (e.g., changing the MPM, or a stuck worker that won't honor reload). Same distinction as for any systemd service.

---
## 📝 How to use this template effectively
1. **Never Copy-Paste:** If you find yourself copying documentation, stop. Read, close the tab, and write it from memory.
2. **Iterate:** Fill the 'ELI5' and 'Analogy' first. If you can't explain it simply, you don't understand it yet.
3. **The "Why" is King:** Seniority is defined by knowing *when* and *why* to use a tool, not just *how*.
4. **Update the Status:** Move from `analyzing` -> `implemented` -> `mastered` as you apply this in real projects.
