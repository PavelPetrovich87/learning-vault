<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/devops/linux -->

# 🎴 Filter Commands & I/O Redirection — Flashcards

<!-- Source: [[Filter_Commands]] · [[IO_Redirection]] · [[Linux_Commands]] -->

<!-- These cards target **mental models, gotchas, and composition** — the layer AI cannot install in your head. They deliberately avoid flag-table recall; look flags up when you need them. -->

## Foundations — the stream model

<!-- What are the three standard streams, their file descriptor numbers, and their default destinations? -->
<!-- ? -->
<!-- **stdin (FD 0)** → keyboard. **stdout (FD 1)** → terminal. **stderr (FD 2)** → terminal. Every process inherits these three open descriptors from its parent shell. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

<!-- Why is "filter commands" a *family* and not just a list of tools? What do they all share? -->
<!-- ? -->
<!-- They all read from **stdin**, transform text, and write to **stdout** — making them chainable through pipes. Their value is compositional, not individual. Knowing `awk` exists matters less than knowing how to thread it between `grep` and `sort`. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- In the age of AI, which layer of command knowledge actually compounds — and why can't AI give it to you? -->
<!-- ? -->
<!-- **Mental models + composition.** AI can generate one pipeline for you, but reading it, debugging it, and adapting it requires *you* to understand streams, data-shape transformations per stage, and exit codes. Recognition without a model makes you a dependent operator. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/meta -->

<!-- What is the single test for whether you actually have the model (vs. AI papering over the gap)? -->
<!-- ? -->
<!-- Can you **predict the output of a 4-stage pipeline before running it?** If not, you don't yet have the model — you're trusting AI on faith. Mispredictions reveal exactly where your understanding is fuzzy. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/meta -->

## Composition — pipelines

<!-- Walking through `grep "404" access.log | cut -d' ' -f1 | sort | uniq -c | sort -rn | head`, what does each stage do to the data shape? -->
<!-- ? -->
<!-- 1. `grep 404` → keeps only 404 lines. 2. `cut -f1` → reduces each line to just the IP. 3. `sort` → groups identical IPs adjacently (required by `uniq`). 4. `uniq -c` → collapses runs, prefixing each with a count. 5. `sort -rn` → re-orders by count, descending. 6. `head` → top 10. Each stage reshapes the data for the next. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/pipelines -->

<!-- Why is the ORDER of stages in a pipeline a design decision, not arbitrary? -->
<!-- ? -->
<!-- Each stage transforms the data shape for the next. `sort` must precede `uniq` (adjacency). Filtering early (`grep`) shrinks the data before expensive stages (`sort`), improving performance. Wrong order breaks correctness; suboptimal order wastes work. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/pipelines -->

<!-- What is "pipeline golf," and what skill does it build? -->
<!-- ? -->
<!-- Solving one task multiple ways (pure `awk`, then `grep | cut`, then `sed`) and comparing for readability and speed. It builds flexibility and teaches each tool's *character* — when one tool is cleaner than another for a given job. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/pipelines -->

## The `uniq` family — adjacency gotchas

<!-- Why MUST `uniq` come after `sort`? -->
<!-- ? -->
<!-- `uniq` only removes lines identical to their **immediate predecessor**. If duplicates are scattered, they never become adjacent and survive. Sorting first groups identical lines so `uniq` can collapse them. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- `sort names | uniq` vs `sort -u names` — same result? Is there a reason to prefer one? -->
<!-- ? -->
<!-- Same deduplicated output. `sort -u` does both in one process — slightly more efficient. But `sort | uniq -c` is irreplaceable when you need the **counts**, because `-u` has no count option. Reach for `uniq -c` for frequency analysis. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- How do you get a frequency-ranked list (most common first) in one pipeline? -->
<!-- ? -->
<!-- `... | sort | uniq -c | sort -rn`. `sort` enables `uniq`; `uniq -c` prefixes counts; `sort -rn` re-sorts numerically descending by that count. This is the canonical "top N" pattern. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/pipelines -->

## `grep` — matching behavior

<!-- Why does `grep error` match "errors", "errorLog", and "terror", and how do you stop that? -->
<!-- ? -->
<!-- `grep` matches **substrings anywhere** by default. To anchor: `^error` (start of line), `error$` (end), `^error$` (exact line), or `-w` for whole-word boundaries. Forgetting this is a top source of false matches. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- What is the difference between `grep -E` and `grep -F`, and when does `-F` matter for safety? -->
<!-- ? -->
<!-- `-E` = extended regex (alternation, `+`, `?` without backslashes). `-F` = **fixed string**, treated literally. `-F` matters when searching for text containing regex metacharacters (`.`, `*`, `[`) — like IPs (`1.2.3.4` matches `1A2B3C4` under regex!) or version strings. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- Why is `cat file | grep pattern` an anti-pattern (UUOC), and what's the fix? -->
<!-- ? -->
<!-- `grep` accepts filenames directly: `grep pattern file`. Prepending `cat` spawns an extra process and adds nothing — "Useless Use of Cat." Same logic applies to `sort < file` over `cat file | sort`. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

## `cut` vs `awk` — choosing the splitter

<!-- What breaks when you use `cut` on a CSV with quoted fields containing commas? -->
<!-- ? -->
<!-- `cut` splits naively on **every** delimiter character — it has no concept of quoting. `"smith, jr",30` becomes two fields at the embedded comma. For real CSV parsing use `awk`, `csvkit`, `mlr`, or a library that understands quotes. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- When should you reach for `awk` instead of `cut`? -->
<!-- ? -->
<!-- When you need **conditional logic** (`$3 > 100`), **arithmetic** (`sum += $2`), a **multi-character or regex** delimiter, or **output formatting**. `cut` is a dumb single-char splitter; `awk` is a mini programming language with `NR`, `NF`, `FS`/`OFS`. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

## `sed` — the destructive tool

<!-- What is the danger of `sed -i`, and how do you protect yourself? -->
<!-- ? -->
<!-- `-i` edits the file **in place with no undo**. A bad regex silently mangles it. Protection: (1) test without `-i` first and inspect stdout; (2) use `sed -i.bak` to keep a backup; (3) never run `-i *.conf` across many files untested. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- What's the difference between `sed 's/old/new/'` and `sed 's/old/new/g'`? -->
<!-- ? -->
<!-- Without `g`: replaces only the **first** match per line. With `g`: replaces **every** match on each line. Forgetting `g` when you meant "replace all" is a classic silent under-replacement bug. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

## `head` / `tail` / `wc` — edges and counts

<!-- Why does `tail -f` stop tracking a log after rotation, and what's the fix? -->
<!-- ? -->
<!-- `tail -f` follows by **file descriptor** — when the file is renamed/recreated, the old fd points at the renamed inode. Use `tail -F` (capital), which **reopens by path** and survives rotation. Default to `-F` for live logs. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- Why does `wc -l` sometimes report one fewer line than the file appears to have? -->
<!-- ? -->
<!-- `wc -l` counts **newlines**, not lines. A file whose last line has no trailing newline undercounts by one. Ensure files end with `\n` for exact counts. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/filters -->

<!-- `tr` has no file argument. What's the idiomatic way to feed it a file? -->
<!-- ? -->
<!-- Use **redirection**, not a pipe: `tr 'a-z' 'A-Z' < file` instead of `cat file | tr ...`. `tr` reads stdin only by design — the `<` form avoids spawning `cat`. See [[Filter_Commands]] and [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/filters -->

## Redirection operators — the basics

<!-- What is the difference between `>` and `>>`? -->
<!-- ? -->
<!-- `>` redirects stdout to a file and **truncates/overwrites** existing content. `>>` **appends** to the end, preserving what was there. Using `>` when you meant `>>` silently destroys a log — back up before redirecting onto important files. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

<!-- The shell runs `command > file.txt`. When is `file.txt` truncated — before or after the command finishes? -->
<!-- ? -->
<!-- **Before.** The shell opens and truncates the destination when setting up redirections, *then* starts the command. So if the command fails, the file is already empty. Never redirect onto an input file or a file you can't afford to lose. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

<!-- What do `<<EOF` (here-document) and `<<<"text"` (here-string) do, and when is each used? -->
<!-- ? -->
<!-- `<<EOF` feeds **multiple lines** of inline text into stdin until a line containing only `EOF`. `<<<` (bash) feeds a **single string** as stdin. Here-docs are for blocks (config generation); here-strings for one-liners (`grep x <<< "$VAR"`). See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

## The redirect-ordering trap

<!-- `command 2>&1 > file` — where do stdout and stderr end up? Why is this a bug? -->
<!-- ? -->
<!-- **stdout → file; stderr → terminal.** Redirections apply **left to right**: `2>&1` first points stderr wherever stdout currently goes (the terminal), THEN `> file` redirects stdout to the file. Stderr is left at the terminal. The classic silent failure. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

<!-- What is the correct order to send both stdout and stderr to the same file, and why? -->
<!-- ? -->
<!-- `command > file 2>&1` — redirect **stdout to the file first**, THEN point stderr at wherever stdout now goes (the file). Order is the whole game. Bash shorthand: `command &> file`. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

## Pipes and stderr

<!-- Does a pipe (`|`) carry stderr from the left command to the right command? -->
<!-- ? -->
<!-- **No.** A pipe connects only **stdout → stdin**. Stderr still goes to the terminal. To carry errors too, merge first: `command 2>&1 | grep pattern`. Forgetting this is why `grep` "doesn't see" error messages. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

<!-- Why does `command | grep error` not filter a command's error messages, and how do you fix it? -->
<!-- ? -->
<!-- Because the pipe only carries **stdout**; stderr bypasses `grep` straight to the terminal. Fix: merge stderr into stdout before the pipe — `command 2>&1 | grep error`. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

## `tee` and `/dev/null`

<!-- What does `tee` do that plain redirection cannot? -->
<!-- ? -->
<!-- `tee` writes stdin to **both stdout AND a named file** simultaneously. Redirection sends output *only* to the file (you lose the live view). `tee` lets you watch output in the terminal while saving it — essential for debugging pipelines and observing long-running builds. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

<!-- What is `/dev/null`, and why is `2>/dev/null` so common in pipelines? -->
<!-- ? -->
<!-- `/dev/null` is a special device that **discards everything written to it** and returns EOF on read. `2>/dev/null` silently drops a command's error messages — the standard way to suppress "permission denied" noise in `find`/`grep`. Caution: it can mask real failures; remove it while debugging. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

<!-- What's the idiomatic way to silence a command completely (both streams)? -->
<!-- ? -->
<!-- `command > /dev/null 2>&1` (or `command &> /dev/null` in bash). Sends stdout to the void, then merges stderr into stdout. Use when you care only whether the command *succeeds* (exit code), not its chatter. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

## Destructive traps

<!-- Why does `sort file.txt > file.txt` produce an empty file, and what's the fix? -->
<!-- ? -->
<!-- The shell **truncates `file.txt` (via `>`) before `sort` reads it** — so `sort` reads an empty input and writes nothing. Fix: write to a temp file then move it, or use a tool with in-place support: `sort -o file.txt file.txt`, or `sponge` (moreutils). See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

<!-- What does `set -o noclobber` protect against, and how do you override it intentionally? -->
<!-- ? -->
<!-- It makes `>` **refuse to overwrite** an existing file, preventing accidental truncation of logs/data. Override with `>|` when you really mean to overwrite. A cheap safety net for interactive shells. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/redirection -->

## Active-recall drills (predict before you run)

<!-- Predict the output: `printf '3\n1\n2\n1\n' | sort | uniq -c` -->
<!-- ? -->
<!-- ``` -->
<!--       1 2 -->
<!--       1 3 -->
<!--       2 1 -->
<!-- ``` -->
<!-- `sort` orders numerically/lexically → `1,1,2,3`; `uniq -c` collapses the run of two `1`s with count 2. Note `1` comes first (sort), and count appears left-padded. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/drills -->

<!-- Predict the output: `echo "a:b:c" | cut -d: -f2` -->
<!-- ? -->
<!-- `b`. Splits on `:`, returns field 2. Fields are 1-indexed. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/drills -->

<!-- Predict the output: `ls /nonexistent 2>&1 | grep -c "cannot"` -->
<!-- ? -->
<!-- Depends on locale, but typically `1` — `ls` writes the "No such file or directory" message to stderr, `2>&1` merges it into stdout so the pipe carries it, and `grep -c` counts matching lines. Without `2>&1` the answer would be `0`. See [[IO_Redirection]]. -->
<!-- #flashcards/devops/linux/drills -->

<!-- Predict the output: `echo "Hello World" | tr 'A-Z' 'a-z' | cut -d' ' -f1` -->
<!-- ? -->
<!-- `hello`. `tr` lowercases to `hello world`; `cut -d' ' -f1` takes the first space-delimited field → `hello`. Two-stage transformation. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/drills -->

## Debug-the-AI — find the planted bug

<!-- What's wrong with `sort names.txt | uniq`? -->
<!-- ? -->
<!-- Nothing structurally — it's correct. But if the goal is just deduplication, `sort -u names.txt` does it in one process. And if you need counts, you'd want `uniq -c`. The "bug" here is inefficiency, not correctness — a good AI answer would flag the simplification. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/debug -->

<!-- What's wrong with `cat file.txt | grep "error" | sort | uniq`? -->
<!-- ? -->
<!-- Two issues: (1) **UUOC** — `grep "error" file.txt` avoids the extra process; (2) `uniq` is redundant if you only want unique errors — `sort -u` combines the last two stages. Cleaner: `grep "error" file.txt | sort -u`. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/debug -->

<!-- What's wrong with `find / -name "*.conf" 2>/dev/null | grep pass`? -->
<!-- ? -->
<!-- Likely correct, but two risks: (1) `2>/dev/null` **hides real errors** — fine for a quick search, dangerous in a script where you need to know `find` failed; (2) grepping for "pass" matches substrings (e.g., "passenger", "bypass") — use `-w` or anchor for precision. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/debug -->

<!-- What's wrong with `sed 's/localhost/127.0.0.1/g' -i *.conf`? -->
<!-- ? -->
<!-- `-i` with **no backup across many files** is reckless: a typo'd regex silently edits every `.conf` at once with no undo. Fix: test on one file without `-i` first, and use `sed -i.bak` to retain backups. See [[Filter_Commands]]. -->
<!-- #flashcards/devops/linux/debug -->
