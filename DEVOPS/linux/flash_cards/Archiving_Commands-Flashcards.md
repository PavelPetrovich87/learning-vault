<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/devops/linux -->

# 🎴 Archiving & Compression — Flashcards

<!-- Source: [[Archiving_Commands]] -->

<!-- **Design note:** Per the AI-era flashcard principle in `AGENTS.md`, these cards target the **mental model** layer — the archive-vs-compression distinction, *why* `.tar.gz` is two stages, *compress-the-whole vs compress-each-entry* and its random-access consequence, fidelity/portability trade-offs, and the tarbomb/path-stripping safety reasoning. They deliberately **avoid flag-table recall** (the `-z`/`-j`/`-J` letters, `zip` options) — those are lookups that live in the concept note. Most cards are **predict-the-outcome** and **debug-the-behavior** drills, because the competence they build — *reasoning about the two-stage tar pipeline and the single-stage zip model* — is the layer AI cannot install in your head. -->

## Archive vs compression — the core distinction

<!-- Is "archiving" the same as "compression"? Name a tool that does only one, and a tool that does both. -->
<!-- ? -->
<!-- **No.** **Archiving** bundles many files into one container (no shrinking required). **Compression** reduces size by encoding redundancy — and on its own does **not** bundle. **`gzip`/`bzip2`/`xz`** do *only compression* (one file/stream each). **`tar`** does *only archiving* (no compression unless you add `-z`/`-j`/`-J` to pipe through a compressor). **`zip`** does *both* in one operation. The Unix separation of concerns is exactly why `.tar.gz` is a two-step artifact: bundle with tar, then squeeze the tar stream with gzip. -->
<!-- #flashcards/devops/linux/archiving -->

<!-- Why does `gzip file.txt` turn your file into `file.txt.gz` without bundling the directory it sits in? What would you have to use instead? -->
<!-- ? -->
<!-- Because **`gzip` only compresses a single file/stream** — it has no concept of "many files" or "directory." It replaces `file.txt` with the squeezed `file.txt.gz`. To bundle the whole directory you must first **archive** it with **`tar`** (which records the tree, permissions, ownership), and *then* compress that tar stream — yielding `.tar.gz`. This is the entire reason `tar` exists alongside `gzip`: gzip can't bundle, tar can't squeeze. -->
<!-- #flashcards/devops/linux/archiving -->

<!-- `.tar.gz` has two extensions. What does each represent, and how many tools produced the file? -->
<!-- ? -->
<!-- `.tar.gz` = a **tar archive** (`.tar` — the bundled container with full Unix metadata, uncompressed) that was **then gzip-compressed** (`.gz`). **Two tools**, two operations: `tar` made the container, `gzip` squeezed the tar stream. (The tar `-z` flag just automates the pipe to gzip — it isn't tar doing the compression.) Contrast `.zip`, where one tool fuses bundling and per-entry compression into a single operation and a single extension. -->
<!-- #flashcards/devops/linux/archiving -->

## The two philosophies — compress-the-whole vs compress-each-entry

<!-- Why can you extract a single file from a `.zip` cheaply, but extracting one file from a `.tar.gz` requires walking the whole compressed stream? -->
<!-- ? -->
<!-- **zip compresses each entry independently** as it enters the archive and keeps a central directory indexing each entry's offset — so extracting one file seeks to its offset and decompresses just that entry. True **random access**. **tar+gzip compresses the whole tar stream as one block** — gzip sees the archive as a single byte sequence, so reaching a file near the end means decompressing everything before it. This single design difference explains the trade-off: zip gains random-access portability; tar.gz gains better cross-file compression and full Unix fidelity. -->
<!-- #flashcards/devops/linux/archiving -->

<!-- Which archive type gives a better compression ratio for a directory of many similar text files — `.tar.gz` or `.zip` — and why? -->
<!-- ? -->
<!-- **`.tar.gz`, usually.** Because gzip compresses the *whole tar stream* as one block, its dictionary can exploit redundancy **across files** — repeated headers, similar log lines, duplicated boilerplate all compress together. **zip compresses each entry independently**, so each file starts with a cold dictionary and can't borrow redundancy from its neighbors. The cross-file compression gain is the main ratio advantage of tar+gzip (and even more so for xz/bzip2). zip pays this cost in exchange for random per-entry access. -->
<!-- #flashcards/devops/linux/archiving -->

## tar flag grammar — predict the outcome

<!-- You run `tar -cvfz out.tgz dir/`. What file actually gets created, and what's wrong? -->
<!-- ? -->
<!-- You get an archive literally named **`z`**, *uncompressed*, containing `dir/` (and `out.tgz` may be treated as a second input). Reason: `-f` takes the **next argument as the archive filename**, and in the cluster `-cvfz` the next token is `z` itself — so `f` grabs `z`. The lesson: keep **`f` last** in the cluster (`-cvzf out.tgz dir/`) so the real filename immediately follows it. (GNU tar is sometimes forgiving, but don't rely on it.) -->
<!-- #flashcards/devops/linux/archiving -->

<!-- A teammate claims `tar -czf arch.tgz dir/` is the same as `tar -c --gzip -f arch.tgz dir/`. Are they right, and what does that tell you about tar's design? -->
<!-- ? -->
<!-- **Right.** `-z` is just the short form of `--gzip`, and `-c`/`-f` are short forms of `--create`/`--file`. Tar's flags are **modular letters that compose** — three verb flags (`-c` create / `-x` extract / `-t` list) plus optional modifiers (`-f` file, `-v` verbose, `-z`/`-j`/`-J` for gzip/bzip2/xz, `-C` dir). Learn the grammar (which letter means what) and you can read any tar command, instead of memorizing whole incantations. The compression is a *swappable stage*, not part of tar itself. -->
<!-- #flashcards/devops/linux/archiving -->

<!-- What's the difference between `tar -czf` and `tar -xzf`? What stays identical between them, and what changes? -->
<!-- ? -->
<!-- **`-c`** = **create** a new archive; **`-x`** = e**x**tract from one. They are the two opposing *verbs* — you use exactly one per command. What stays identical: `-z` (same gzip stage), `-f` (same archive file), `-v` (same verbosity). What changes: direction of data flow (in vs out) and whether files appear on disk or disappear into the archive. The same compression flag pair appears on both, because gzip is symmetric — it decompresses as readily as it compresses. -->
<!-- #flashcards/devops/linux/archiving -->

## Fidelity vs portability — when to use which

<!-- You back up `/etc` (with symlinks, setuid binaries, root ownership) as a `.zip` and restore on a fresh box. What goes wrong, and what should you have used? -->
<!-- ? -->
<!-- `.zip` was designed cross-platform, so its classic format stores only **partial Unix mode bits** and historically mangles **symlinks, ownership (UID/GID), device nodes, and hard links**. Restoring `/etc` via `.zip` can produce broken symlinks, wrong owners, missing/dropped setuid bits — a broken system config. You should have used **`tar`** (e.g. `tar -czf etc.tgz /etc`), which records all that Unix metadata and reproduces it faithfully on extract as root. Rule: **Linux system backups → tar; cross-platform distribution to humans → zip.** -->
<!-- #flashcards/devops/linux/archiving -->

<!-- You need to send a folder of PDFs to a Windows-using client. Do you reach for `tar -czf` or `zip -r`? Why? -->
<!-- ? -->
<!-- **`zip -r`.** The decisive factor here is **portability**, not ratio or fidelity: Windows opens `.zip` natively (double-click), but `.tar.gz` needs 7-Zip, WinRAR, or WSL. PDFs are already compressed, so the ratio difference between tar+gzip and zip is negligible anyway. `zip -r folder.zip folder/` is the right call. Note zip's argument order: **archive name first, then inputs** — the opposite habit from tar. -->
<!-- #flashcards/devops/linux/archiving -->

## Compression algorithm trade-offs

<!-- Rank `gzip`, `bzip2`, `xz`, `zstd` by compression ratio (best→worst) and by speed (fastest→slowest). Then pick one for: (a) quick network transfer, (b) smallest cold-storage archive, (c) a modern default where available. -->
<!-- ? -->
<!-- **Ratio:** xz ≈ zstd (high) > bzip2 > gzip. **Speed:** zstd >> gzip > bzip2 > xz (xz is slowest and RAM-heavy on compress). **Picks:** (a) quick transfer → **gzip** (fast, universal) or **zstd** if both ends have it; (b) smallest cold archive → **xz** (best ratio; compression cost paid once); (c) modern default → **zstd** (near-xz ratio at multiples of gzip's speed — kernels, packages, containers migrating to it). In doubt: **gzip**, the universal lingua franca. -->
<!-- #flashcards/devops/linux/archiving -->

<!-- You compress a 5 GB log directory with `xz -9` on a small 2 GB-RAM VM and it gets killed by the OOM killer. Why, and what's the fix? -->
<!-- ? -->
<!-- **xz (LZMA) at high compression levels is memory-hungry on *both* compress and decompress** — level 9 can need well over 1 GB of working memory. On a 2 GB VM the kernel OOM-kills it. Fix: drop the level (xz defaults to **6**, which is usually plenty and far lighter) or switch to **gzip/zstd**, which have tiny memory footprints. The general lesson: high-ratio algorithms (xz, and to a lesser degree bzip2) trade *memory and CPU time* for ratio — they're for cold data on machines that can afford it, not for constrained VMs. -->
<!-- #flashcards/devops/linux/archiving -->

## Safety — tarbombs, absolute paths, untrusted archives

<!-- What is a **tarbomb**, and what one habit prevents the damage on archives you didn't create? -->
<!-- ? -->
<!-- A tarbomb is an archive whose top level is *files* (not a wrapper directory) — usually because the maker ran `tar -czf x.tgz *` from *inside* the folder. Extracting it dumps those files directly into your **cwd**, littering or overwriting it. The preventive habit: **always `tar -tvzf` (list) before `tar -xvzf`** on anything untrusted — if the listing shows no common top-level directory, extract into a fresh empty dir (`mkdir x && tar -xzf a.tgz -C x`). For archives *you* make, always tar a **directory** (`tar -czf app.tgz app/`), never a glob from inside. -->
<!-- #flashcards/devops/linux/archiving -->

<!-- `tar -czf etc.tgz /etc/passwd` prints `Removing leading '/' from member names`. Did something fail? Why does tar do this, and what's the danger of disabling it? -->
<!-- ? -->
<!-- **No, it's normal and expected.** Tar stores **relative paths** by stripping the leading `/`, so the entry is `etc/passwd`, not `/etc/passwd`. This is a **safety feature**: if tar stored absolute paths, extracting an archive of `/etc` or `/usr` could **overwrite your live system files** destructively and silently. Stripping the `/` forces extraction to be relative to your *current* directory — you choose where things land. Disabling it with `--absolute-names` on untrusted input is a classic foot-gun. Don't try to "fix" the warning. -->
<!-- #flashcards/devops/linux/archiving -->

<!-- Why should you extract an archive from an untrusted source as a **non-root user into a throwaway directory**, rather than as root in your home dir? -->
<!-- ? -->
<!-- Two compounding risks: (1) **Malicious metadata** — a crafted archive can set **setuid bits**, weird ownership, or world-writable perms that only root would honor; extracting as non-root neutralizes them (you can't give files away or set setuid you don't own). (2) **Path traversal / symlink escape** — entries like `../../../etc/passwd`, or a symlink pointing outside the extract dir that a later write follows. Modern GNU tar guards against leading `/` and `..` by default, but a throwaway sandbox dir limits the blast radius if a guard is bypassed. Never auto-extract user-supplied archives as root on a server. -->
<!-- #flashcards/devops/linux/archiving -->

## Composition — tar as a stream

<!-- What does the `-` mean in `tar -czf - dir/ | ssh host 'tar -xzf -'`, and why is this idiomatic for copying a directory tree between hosts? -->
<!-- ? -->
<!-- **`-` means "the archive is the standard stream, not a file."** On the create side, `tar -czf -` writes the compressed tar stream to **stdout**; on the extract side, `tar -xzf -` reads it from **stdin**. The pipe (carried over SSH) ferries bytes from one to the other. It's idiomatic because it copies a tree **with no temp file and full metadata fidelity** — tar bundles perms/ownership/symlinks, gzip squeezes it for the network, ssh transports it, and the remote tar unpacks it. This is the payoff of tar's archive/compression split: each stage does one job and snaps together with `|`. -->
<!-- #flashcards/devops/linux/archiving -->

<!-- You run `gzip photo.jpg` to "save space." The result is barely smaller and you've wasted CPU. What did you miss? -->
<!-- ? -->
<!-- **`photo.jpg` is already compressed** — JPEG's own compression has already squeezed out nearly all the redundancy a general-purpose compressor like gzip could find. Re-compressing an already-compressed file yields **near-zero gain** (sometimes even slight growth from overhead) and wastes CPU. The same applies to `.mp4`, `.png`, `.mp3`, `.zip` entries, `.gz` files themselves, etc. General-purpose compressors only help on **incompressible-not-yet** data: text, logs, source code, raw databases, BMPs/WAVs. Recognize already-compressed formats and skip them. -->
<!-- #flashcards/devops/linux/archiving -->
