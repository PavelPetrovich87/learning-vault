---
status: archived
artifact_type: exercise_symptoms
exercise: "[[07_Medium_Users_UsermodG_Locked_Out_Of_Sudo]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — One Character in `usermod` Locked Alice Out of `sudo`

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Alice experienced.

**Timeline:**

- **02:14** — Alice SSH'd into `web-prod-03` to debug a container issue. She's in the middle of an incident.
- **02:31** — She wants to run `docker` commands without prefixing `sudo` every time. She knows that's a `docker` group membership. She runs:
  ```
  sudo usermod -G docker alice
  ```
  (`sudo` worked fine *before* this command — she used it to run the `usermod` itself.)
- **02:31:20** — Next time she prefixes `sudo` for anything:
  ```
  $ sudo whoami
  [sudo] password for alice:
  Sorry, user alice is not allowed to execute '/usr/bin/whoami' as root on web-prod-03.
  ```
  Every subsequent `sudo` attempt gets the same rejection.
- **02:33** — Alice tries `sudo -l` to see what she's allowed to do: same rejection.

**What Alice has verified:**

- She is *definitely* `alice` (`whoami` without sudo confirms).
- Her current SSH session is otherwise healthy — she can run ordinary commands, edit files in her home directory, `ls`, `cat`, etc. The session itself isn't broken.
- `sudo` worked for the `usermod` command itself — that was the last successful `sudo`.
- She has **not** logged out. She's now sitting very still in her terminal because she has a vague understanding that group membership is read at login, and she's afraid that if she disconnects she won't be able to get back in with sudo.

**What Alice has NOT yet realized (do not volunteer):**

- That `usermod -G` *replaces* the supplementary group list rather than appending to it.
- That her command just removed her from `sudo` (or `wheel`) by replacing her supplementary groups with only `docker`.
- That her current session still has the *old* group membership cached and that's why ordinary commands still work.

**What Alice reports feeling:**

- "I did one tiny thing to make docker easier and I think I just bricked my production access. I'm not logging out of this session for anything."

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

**`usermod -G docker alice` *replaced* Alice's entire supplementary group list with `docker` only.** The `-G` flag (capital, no `a`) sets the supplementary groups to the list provided — it does **not** append. So Alice went from (for example) `alice : alice sudo docker www-data` to `alice : alice docker`. In one command, she removed herself from `sudo` (or `wheel`, on RHEL-family), and from any other supplementary group she was in.

The one missing character is **`a`**: `usermod -aG docker alice` (or `--append --groups`) *adds* `docker` to her existing supplementary groups, preserving everything else. Without `a`, the command is a wholesale replacement. This is the canonical "most destructive one-character mistake in Linux administration" — it's silent (no warning, no confirmation), immediate, and the natural reflex to "test if it worked" (run `sudo`) is the thing that reveals the damage.

### The mechanism (why current session still works)

- Linux reads your **group membership at login time** and caches it in the process's credentials. Every process you spawn from that login inherits the cached groups. The kernel does **not** re-read `/etc/group` on every operation.
- When Alice ran the `usermod`, it modified `/etc/group` on disk. But Alice's *current shell* still holds the credentials she authenticated with at 02:14 — including `sudo`. That's why her session is still functional for ordinary commands, and why `sudo` *almost* works (the `sudo` binary re-checks group membership against `/etc/group` at invocation — and now sees she's not in `sudo` — which is why it rejects her, even mid-session).
- The moment Alice starts a fresh login (new SSH, `su - alice`, even a new shell that re-reads credentials), she'll have only `docker` and her primary group. Her ordinary commands will still work, but sudo is gone for good until it's fixed.
- This is exactly why "don't log out yet" is the correct instinct: the live session is her lifeline, even though `sudo` itself is already broken within it.

### The confirmation command

- **`id alice`** (or just `id` for the current process's cached groups, vs `id alice` for what's on disk for that user). The difference is itself diagnostic: `id` (no arg) shows the *cached* groups of the current process; `id alice` shows what `/etc/passwd` and `/etc/group` currently say about alice.
  - Expected: `id alice` shows `groups=…(alice)…(docker)` — and **no `sudo`/`wheel`**. That's the proof.
  - Contrast with what it was before: would have included `…(sudo)`.
- `getent group sudo` (or `wheel`) — shows current members of the sudo group. Alice will not be listed.
- `/etc/group` directly: `grep -E '^(sudo|wheel):' /etc/group` — Alice's absence confirms the removal.

### The recovery — safely, in priority order

Alice has a live session and broken sudo. The recovery options, least-risky first:

1. **Fix it from the live session without sudo — if any non-sudo path exists.** Alice's session works, but editing `/etc/group` requires root. Without sudo, this path is closed *unless* she already has an authenticated root shell open somewhere, or a teammate with active sudo can SSH in. (Right now, neither.)
2. **Get a teammate with sudo to fix it.** The cleanest fix. A teammate SSHes in and runs `sudo usermod -aG sudo alice` (or `-aG wheel` on RHEL). This is the lowest-risk path *if* such a teammate is reachable. The downside: the teammate is off-shift, so this may take time.
3. **Use the system's recovery console / ILO / cloud-provider serial console.** Most servers have an out-of-band path:
   - **Physical / ILO / iDRAC / IPMI:** boot to single-user / rescue mode, get a root shell without sudo (single-user mode mounts root rw and drops to a root prompt), edit `/etc/group` to re-add alice to `sudo`, reboot normally.
   - **Cloud (AWS EC2 Serial Console, GCP Serial Port, Azure Serial Console):** same idea — boot to GRUB, edit the kernel line to add `init=/bin/bash` or `single`, boot to a root shell, fix, reboot.
   - **Or: detach the root volume, attach to another instance, edit `/etc/group` from there, reattach.** (The "break-glass" cloud option when serial console isn't available.)
   - This path is reliable but slow — minutes to tens of minutes — and Alice is mid-incident.
4. **What NOT to do:** Do **not** close the SSH session hoping to "log back in fresh and try sudo again." The fresh login will have the *new* (broken) groups; sudo will still reject her; and now she's lost the live session that, while not sudo-capable, is at least a working shell on the box in case she needs it for the recovery.
5. **What NOT to do:** Don't try to "fix" `/etc/group` by hand-editing it from Alice's non-sudo session — she doesn't have write permission to `/etc/group` as a non-root user.

The correct sequence: (a) Don't panic, don't log out. (b) Page the off-shift teammate if at all reachable — five minutes of their time fixes it cleanly. (c) In parallel, prepare the ILO/serial-console path so it's ready if the teammate can't be reached. (d) Once root is available (via teammate or console), `sudo usermod -aG sudo alice` — note the `-aG` this time. (e) Alice logs out and back in; verifies `sudo -l` works.

### Hidden diagnostic traps

1. **Conflating `-G` (replace) with `-aG` (append).** The single most important distinction. `-G` without `a` is a wholesale replacement; with `a`, an append. This is the whole incident in one flag.
2. **Closing the session prematurely.** The live session is the lifeline. Logging out doesn't fix anything — it just gives Alice a fresh login with the *same broken* groups, minus the safety of the existing shell. The instinct to "stay logged in" is correct.
3. **Assuming `sudo` will "come back" after a fresh login.** It won't. The damage is on disk in `/etc/group`; a fresh login reads the damaged file. Re-login changes nothing until the file is fixed.
4. **Forgetting that `sudo` re-checks groups at invocation.** This is why `sudo` *already* fails inside Alice's live session even though other commands work — `sudo` queries `/etc/group` fresh; the rest of her shell uses cached credentials. That asymmetry is itself a clue.
5. **Hand-editing `/etc/group` without `vipw -g` / `vigr`.** Even from a root shell, direct editing risks syntax corruption (a malformed group file breaks logins wholesale). Use `vigr` (or `vipw -g`) which locks and validates.
6. **Picking the riskiest recovery first.** Rebooting into single-user is reliable but slow and disruptive mid-incident. The teammate path is faster if reachable. Order recovery by least-disruptive-first.

### The preventive lesson

- **`-aG`, not `-G`.** Append, don't replace. The `a` stands for append and it is the difference between "add me to a group" and "delete me from every other group." This is the one rule that would have prevented the incident.
- **`gpasswd -a alice docker` is a safer alternative** for adding a single user to a single group — it only does that one thing and cannot accidentally wipe other memberships. Consider it the default tool for "add one user to one group."
- **Supplementary groups are read at login.** Changes don't affect existing sessions. This is both a safety property (you can recover from a live session) and a gotcha (the change "doesn't take effect" until you re-login, which misleads people into re-logging-in at exactly the wrong moment).
- **Never run `usermod -G …` on your own account while logged in as that account on a production box** unless a second sudo-capable human is reachable. The blast radius of a missing `a` is total lockout.

### Rubric — what a strong diagnosis covers

1. **Identifies `-G` vs `-aG` as the cause** — `usermod -G` replaces the entire supplementary group list; without `-a`, Alice was removed from `sudo`/`wheel`.
2. **Names the confirmation command** — `id alice` (and contrast with `id` for cached groups); `getent group sudo` to confirm absence.
3. **Explains why the live session still works** — group membership cached at login; `sudo` re-checks at invocation, which is why it fails while other commands work.
4. **Orders recovery by least-risk** — (a) don't log out, (b) page a sudo-capable teammate, (c) ILO/serial-console single-user or volume-attach rescue; warns against premature logout.
5. **Calls out the wrong moves** — re-logging-in doesn't fix it; hand-editing `/etc/group` risks corruption (use `vigr`).
6. **Names the preventive lesson** — `-aG` not `-G`; prefer `gpasswd -a user group` for single-add; never `usermod -G` your own account without a reachable second sudoer.

## Part B — end
