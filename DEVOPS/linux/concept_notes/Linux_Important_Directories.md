---
status: analyzing
tags: [devops, linux, filesystem, directories, fhs, udemy]
created: 2026-07-01
updated: 2026-07-01
source: Udemy DevOps Course — Linux Fundamentals
difficulty: beginner
---

# 🧠 Topic: Linux Important Directories

> [!abstract] Executive Summary (ELI5)
> Linux organizes files into a single tree where each folder has a job. Knowing the purpose of the main directories helps you find config files, binaries, logs, and user data without guessing.

---

## 🎯 The "Why"
- **Problem Solved:** Without a predictable layout, every distribution would store configs, programs, and logs in different places. The Filesystem Hierarchy Standard (FHS) defines conventions so admins and tools can locate resources quickly.
- **Alternatives:** Windows uses drive letters and a separate registry; macOS inherits a BSD layout with extra Apple-specific directories; NixOS and Flatpak use unconventional paths for isolation.
- **Trade-offs:** FHS is a convention, not a strict law. Some distros bend the rules (e.g., `/usr` merge, `/var` vs `/run`), but the core ideas remain stable enough to rely on.

## 💡 Mental Model & Analogy
- **Analogy:** A Linux filesystem is like a city with zoning laws. `/home` is residential, `/usr/bin` is downtown retail, `/etc` is city hall paperwork, `/tmp` is a construction dumpster, `/var/log` is the newspaper archive, and `/proc` is a live city dashboard.
- **Visual/Diagram:**

```mermaid
graph TD
    A[/] --> B[/home]
    A --> C[/usr]
    A --> D[/bin]
    A --> E[/sbin]
    A --> F[/etc]
    A --> G[/tmp]
    A --> H[/var]
    A --> I[/run]
    A --> J[/proc]
    A --> K[/sys]
    A --> L[/lib]
    C --> C1[/usr/bin]
    C --> C2[/usr/sbin]
    C --> C3[/usr/lib]
    H --> H1[/var/log]
    H --> H2[/var/spool]
    H --> H3[/var/lib]
```

## 🛠️ Core Principles & Code Snippets

### 1. Home directories: `/home`
- One subdirectory per regular user.
- User-owned configs, documents, and local data live here.
- The root user's home is `/root`, not `/home/root`.

```bash
ls -la /home
# Typical output: alice/ bob/
```

### 2. User executables: `/bin` and `/usr/bin`
- Commands any user can run: `ls`, `cat`, `grep`, `docker`.
- `/bin` is for programs needed early in boot; `/usr/bin` is for the bulk of installed packages.
- On most modern distros `/bin` is a symlink to `/usr/bin`.

```bash
which ls
# /usr/bin/ls
```

### 3. System executables: `/sbin` and `/usr/sbin`
- Commands intended for the root user or system administration: `fdisk`, `mkfs`, `systemctl`, `ip`.
- Normal users can often read them but lack privileges to use them effectively.

```bash
ls /sbin | head -n 5
```

### 4. Configuration: `/etc`
- System-wide config files, not user-specific ones.
- Examples: `/etc/ssh/sshd_config`, `/etc/nginx/nginx.conf`, `/etc/passwd`, `/etc/fstab`.
- Backup this directory before major changes.

```bash
ls /etc | head -n 10
```

### 5. Temporary files: `/tmp` and `/var/tmp`
- `/tmp` is for short-lived files; many distros clean it on reboot or with `tmpwatch`/`systemd-tmpfiles`.
- `/var/tmp` persists across reboots and is meant for longer-lived temporary data.
- Both are world-writable, so set appropriate permissions.

```bash
mktemp -d /tmp/demo-XXXXXX
```

### 6. Kernel and server data: `/var`, `/run`, `/dev`, `/sys`
- `/var` holds variable data: logs, mail queues, databases, caches (`/var/log`, `/var/spool`, `/var/lib`).
- `/run` is a tmpfs for runtime state since early boot: PID files, sockets, mount info.
- `/dev` contains device files created by `udev` or `devtmpfs`.
- `/sys` exposes kernel and driver information as a virtual filesystem (`sysfs`).

```bash
ls /run | head -n 5
ls /sys/class/net
```

### 7. System information: `/proc`
- A virtual filesystem that shows live process and kernel state.
- Each process has a directory named by its PID: `/proc/<pid>/`.
- Files like `/proc/cpuinfo`, `/proc/meminfo`, `/proc/loadavg` are read-only views into the kernel.

```bash
cat /proc/loadavg
cat /proc/cpuinfo | grep 'model name' | head -n 1
```

### 8. Shared libraries: `/lib`, `/lib64`, `/usr/lib`, `/usr/lib64`
- Essential shared libraries needed by programs in `/bin` and `/sbin` live in `/lib` (or `/lib64`).
- `/usr/lib` holds libraries for `/usr/bin` programs and package data.
- `/usr/local/lib` is for libraries compiled manually or installed outside the package manager.

```bash
ldd /bin/ls
ldconfig -p | head -n 5
```

> [!tip] Implementation Note
> When you install software from a package manager, it usually lands in `/usr`. When you install from source or custom scripts, use `/usr/local` or `/opt` to keep the system layout clean and avoid conflicts with the package manager.

## ⚠️ Pitfalls & Anti-patterns
- **Common Mistakes:**
  - Editing files directly in `/usr` that belong to packages; the next update will overwrite them.
  - Putting sensitive data in `/tmp` and assuming it will survive a reboot.
  - Treating `/proc` and `/sys` as regular files; they are virtual and can change or disappear.
- **When NOT to use:** `/tmp` is not a place for long-term storage. `/etc` is not for application runtime data — use `/var/lib` or `/run` instead.
- **Performance/Security considerations:**
  - `/tmp` is often mounted with `nodev`, `nosuid`, `noexec` flags to limit attacks.
  - `/run` lives in RAM, so don't store large files there.
  - World-writable directories like `/tmp` require careful permissions and avoiding predictable filenames.

## 🔗 Deep Connections
- **Parent Topic:** [[Linux Operating System]]
- **Related Concepts:** [[Unix_Like_Kernel]], [[Linux_Shell]], [[Linux File System]], [[Process Management]], [[Environment Variables]]
- **Prerequisites:** Basic command line navigation, understanding of files and permissions.

## ❓ Active Recall Questions
> [!faq]- Why are `/bin` and `/usr/bin` separate, and why do modern distros often merge them?
> Historically `/bin` held programs needed before `/usr` was mounted. With initramfs and unified `/usr`, that separation is mostly obsolete, so many distros symlink `/bin` to `/usr/bin`.

> [!faq]- What is the difference between `/tmp` and `/var/tmp`?
> `/tmp` is cleaned frequently or on reboot. `/var/tmp` is expected to persist across reboots and is for longer-lived temporary files.

> [!faq]- Why should application runtime data not go into `/etc`?
> `/etc` is for static configuration. Runtime state is variable, so it belongs in `/var` (persistent) or `/run` (volatile). This separation makes backups, restores, and configuration management easier.

---

## 📝 How to use this template effectively
1. **Never Copy-Paste:** If you find yourself copying documentation, stop. Read, close the tab, and write it from memory.
2. **Iterate:** Fill the 'ELI5' and 'Analogy' first. If you can't explain it simply, you don't understand it yet.
3. **The "Why" is King:** Seniority is defined by knowing *when* and *why* to use a tool, not just *how*.
4. **Update the Status:** Move from `analyzing` -> `implemented` -> `mastered` as you apply this in real projects.
