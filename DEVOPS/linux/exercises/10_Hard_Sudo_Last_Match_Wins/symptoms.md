---
status: archived
artifact_type: exercise_symptoms
exercise: "[[10_Hard_Sudo_Last_Match_Wins]]"
visibility: persona_only
created: 2026-07-17
updated: 2026-10-21
---

# Symptoms — The Sudoers Rule That Should Have Blocked Her — But Didn't

## Part A — Observable Symptoms

> The persona reads this section throughout the exercise. It contains **no diagnosis and no root cause** — only what Imani and Owen observed.

**Background:**

- Acme is moving to centrally-managed passwords. Policy decision: operators on the bastion should no longer be able to run `passwd` to change local account passwords.
- **Owen** (junior admin) was asked to implement the denial.
- **Imani** (compliance auditor) was asked to attest that the control is effective.

**The relevant sudoers content (spread across the main file and a drop-in):**

In `/etc/sudoers` (the main file), a legacy allow rule from when Alice managed local passwords:
```
alice   ALL=(ALL) /usr/bin/passwd
```

Owen added the denial. He put it in `/etc/sudoers.d/operators` (a drop-in), reasoning that "the operators policy file is the right home for operator rules":
```
# /etc/sudoers.d/operators
alice   ALL=(ALL) !/usr/bin/passwd
```

(For the candidate's information, since they may not recall the precedence offhand: the main `/etc/sudoers` ends with `#includedir /etc/sudoers.d`, which is how the drop-in gets pulled in. The candidate should reason about what that implies.)

**Timeline:**

- **Monday** — Owen adds the denial line to `/etc/sudoers.d/operators`. Runs `sudo visudo -c` — syntax OK.
- **Monday evening** — Imani's audit: she sits with Alice, has Alice run `sudo passwd`. **It works.** Alice is prompted for her password, then `passwd` runs and she's at the `(current) UNIX password:` prompt. The denial did not take effect.
- **Tuesday** — Owen starts tweaking the denial syntax: `!/usr/bin/passwd *`, `"!/usr/bin/passwd"`, moving `NOPASSWD:` around, trying `runas` specifiers. Nothing changes Alice's effective permission. `visudo -c` passes every time.
- **Wednesday** — Owen is now considering just deleting the legacy allow rule from `/etc/sudoers` as a workaround. He hasn't done it yet because he wants to understand *why* his denial didn't work.

**What Owen has verified:**

- The denial line is unambiguously present in `/etc/sudoers.d/operators`.
- Both the allow and the deny rules are syntactically valid (`visudo -c` returns clean).
- The drop-in file is in fact included (`#includedir /etc/sudoers.d` is at the end of `/etc/sudoers`).
- Alice's account is otherwise normal; no other surprise rules he's aware of.

**What Owen has NOT realized (do not volunteer):**

- How sudo resolves *conflicting* matches across multiple rules.
- The order in which the main file and the drop-in are evaluated, and what that implies for which rule wins.
- That his denial rule "works correctly" — it's just being overridden by a rule evaluated *after* it.

**What Owen reports feeling:**

- "The syntax is right, visudo says it's right, the rule is in the file — and it just doesn't do anything. I feel like sudoers is gaslighting me."

---

> [!danger] Part B — Root Cause & Reference Solution
>
> **The persona reads this ONLY when the candidate says they are ready to test/debrief.** Stop here during the brainstorm phase.

## Part B — Root Cause & Reference Solution

### The actual root cause

**sudoers evaluates rules in order and uses *last-match-wins*.** When multiple rules match the same user for the same command, the *last* matching rule in evaluation order decides. Owen's denial (`!/usr/bin/passwd`) and the legacy allow (`/usr/bin/passwd`) *both* match Alice running `passwd`. The question is which is evaluated last.

The evaluation order is:

1. The main `/etc/sudoers` file, top to bottom.
2. Then — *because the main file ends with `#includedir /etc/sudoers.d`* — each drop-in file in `/etc/sudoers.d/`, in lexicographic filename order (`operators` is included as the main file's *last* directive, so drop-in contents are read *after* everything above the `#includedir` line).

So the evaluation sequence is: legacy `alice … /usr/bin/passwd` (allow) is read first, from the main file; then Owen's `alice … !/usr/bin/passwd` (deny) is read *second*, from the drop-in. Last-match-wins → **Owen's deny rule should win.** But it doesn't. Why?

Because in this scenario Owen's recollection of "where the allow rule lives" is incomplete. The actual situation (which the candidate should reason out from the evaluation model): the allow rule is *also* effectively later, OR — more likely given the symptom — there's a broader rule evaluated *after* the deny that re-permits `passwd`. The canonical example: a catch-all rule like `%operators ALL=(ALL) NOPASSWD: ALL` *in the same or a lexicographically-later drop-in* that re-grants everything, including `passwd`, after Owen's deny. Last-match-wins → the catch-all wins.

The general lesson holds regardless of which specific rule is the culprit: **last-match-wins means a denial only sticks if no later rule re-permits the command.** Owen put his deny in a drop-in and assumed it would "win" — but anything evaluated after it (in the same file below it, in a lexicographically-later drop-in, or in the main file below where `#includedir` sits — though `#includedir` is conventionally last) overrides it.

(The cleanest formulation of the diagnostic question for the candidate: *given that the denial is present and syntactically valid, what does last-match-wins imply must be true about the rule chain?* Answer: a later matching rule is re-permitting `passwd`. Find it.)

### The mechanism (why this is so unintuitive)

- **People expect "first-match-wins"** (like firewall rules, where an early DROP stops evaluation). sudo is the opposite: it reads *all* matching rules and applies the *last* one. A `!` deny does not short-circuit; it's just a vote that a later rule can outvote.
- **`#includedir` and `#include` are textually inlined.** When the main file says `#includedir /etc/sudoers.d`, sudo reads each drop-in file's contents as if they were pasted in at that point. So a drop-in's rules sit *after* everything above the `#includedir` and before anything below it. The evaluation order of drop-ins is *lexicographic by filename* — `00-admins` is read before `99-operators`. This is a frequent surprise: operators rename drop-ins and change effective policy without realizing it.
- **The combination is what bites.** A deny in a drop-in that sorts *before* another drop-in (or before content below an ill-placed `#includedir`) gets overridden. The fix isn't "make the deny syntax better" — it's "make the deny the *last* rule that matches this user+command."

### The single diagnostic command

**`sudo -l -U alice`** (run as root, or by Alice herself as `sudo -l`) — lists the *effective* commands sudo concludes Alice may run, after evaluating the entire rule chain. This is the authoritative view; staring at `/etc/sudoers` and the drop-ins is reconstruction, and reconstruction is what got Owen into trouble. The output will include a line permitting `passwd` despite the deny — proving a later rule is overriding it.

To find *which* rule is doing the overriding, supplementary:

- **`sudo -l -U alice -v`** (verbose) may add detail in newer sudo.
- Manually inspect evaluation order: `cat /etc/sudoers; for f in /etc/sudoers.d/*; do echo "=== $f ==="; cat "$f"; done` — read in the order sudo reads them (main file top-to-bottom, then drop-ins in lexicographic order). Look for any rule matching Alice that permits `passwd` or `ALL` *after* the deny.
- **`visudo -c`** validates syntax (which Owen already did) but does *not* evaluate effective policy. Syntax-validity ≠ effective-permission. Owen conflated the two.

### The canonical fix (and why Owen's "delete the legacy allow" is riskier than it sounds)

Owen's instinct — "just delete the legacy allow rule" — would *probably* work *if* that legacy rule is the only thing re-permitting `passwd`. But it has real downsides: it mutates shared infrastructure (the main `/etc/sudoers`) for a narrow policy goal; it requires editing the main file (which dpkg wants to own — similar to the systemd `edit` exercise); and it doesn't address the *real* problem, which is that any future broad rule added later will again override the deny. Deleting one allow rule patches the symptom, not the model.

Better, in order of robustness:

1. **Make the deny the *last* rule that matches this user+command.** Move it so it's evaluated after every rule that re-permits `passwd`. In practice, since `#includedir` reads drop-ins in lexicographic order, put the deny in a drop-in whose name sorts *after* anything that re-permits — e.g. `/etc/sudoers.d/zz-no-passwd`. This guarantees the deny is the last vote.
   - But this is fragile (the next admin who adds `zzz-something` could re-override). So consider:
2. **Use a `Cmnd_Alias` and grant it positively, listing everything *except* `passwd`.** Rather than "allow all, then deny passwd" (which is what last-match-wins makes fragile), express the policy as "allow this explicit list" — `Cmnd_Alias OPERATORS = /usr/bin/ls, /usr/bin/cat, /usr/bin/systemctl status, …` and grant `alice ALL=(ALL) OPERATORS`. There's no `passwd` in the list, so there's nothing to override. Positive allow-lists are more robust than "allow-all-then-deny" under last-match-wins.
3. **Tighten or remove the broad re-permitting rule itself.** If a `%operators … ALL` is the culprit, the deeper question is whether operators *should* have `ALL` at all. A `NOPASSWD: ALL` grant makes any specific deny effectively theater. The right long-term move is to scope operators to what they actually need — at which point the passwd question answers itself.

The model: under last-match-wins, **denials are weak** — they're easily overridden. **Positive allow-lists are strong** — they can't be overridden into granting more than they list. Design sudoers policy around positive allow-lists wherever possible; reserve denials for cases where you can guarantee evaluation order.

### Hidden diagnostic traps

1. **Assuming first-match-wins.** The single biggest mental-model error. sudo is last-match-wins; a deny does not short-circuit. People who think firewalls and sudoers work the same way write ineffective denies.
2. **Treating `visudo -c` passing as "the rule is effective."** Syntax validity and effective permission are different things. `visudo -c` checks grammar, not policy outcome. `sudo -l` checks outcome.
3. **Ignoring drop-in ordering.** `/etc/sudoers.d/` files are read in *lexicographic* order, not modification time, not "most specific first." Renaming a drop-in changes effective policy. The `#includedir` directive's position in the main file also matters (though it's conventionally last).
4. **Conflating main-file and drop-in precedence.** People assume "drop-ins override the main file" as a blanket rule. It's true *only* because `#includedir` is conventionally last in the main file — so drop-ins are read last, so they win by last-match-wins. Move `#includedir` higher and the precedence flips. It's the position, not the location, that determines order.
5. **Deleting rules to "fix" policy without understanding the chain.** Owen's instinct. It patches the symptom if the deleted rule was the culprit, but it doesn't teach the model, and the next broad rule added later will re-break the deny.
6. **Missing the `NOPASSWD: ALL` anti-pattern.** If operators have `NOPASSWD: ALL`, *no specific deny is meaningful* — `ALL` is re-permitted by that rule regardless. The deeper issue is the over-broad grant; the passwd deny is downstream of it.
7. **Forgetting that `!` matches command-name only with no further args semantics.** `!/usr/bin/passwd` denies `passwd` exactly; `!/usr/bin/passwd *` denies it with any args. Subtle but relevant if Alice runs `passwd` with flags. Get the match shape right.

### The generalizable model

- **sudoers is last-match-wins.** All matching rules are evaluated; the last one decides. A denial only sticks if no later rule re-permits.
- **`#includedir` is textual inlining at the directive's position; drop-ins read in lexicographic order.** Effective precedence is determined by where things sit in the resulting flat rule list — not by "main vs drop-in" as an inherent property.
- **`sudo -l -U <user>` is the effective-policy diagnostic.** Use it instead of eyeballing config files.
- **`visudo -c` validates syntax, not policy.** A clean parse says nothing about whether a rule actually constrains.
- **Prefer positive allow-lists over "allow-all-then-deny."** Under last-match-wins, denials are fragile; explicit allow-lists are robust. The deeper fix for "we don't want them running passwd" is "scope them to what they need," not "give them everything and try to subtract."
- **`NOPASSWD: ALL` is a policy black hole** — any specific deny inside its scope is theater. Fix the over-broad grant, don't paper over it.

### Rubric — what a strong diagnosis covers

1. **Names last-match-wins** as sudo's evaluation model — all matching rules are evaluated, the last one decides; a deny doesn't short-circuit.
2. **Traces the actual rule chain** — main file top-to-bottom, then `#includedir` inlines drop-ins in lexicographic order; identifies which later rule re-permits `passwd`.
3. **Names `sudo -l -U alice` as the effective-policy diagnostic** — and distinguishes it from `visudo -c` (syntax vs outcome).
4. **Explains the `#includedir` / drop-in ordering subtlety** — precedence is positional, not "drop-ins inherently override main file."
5. **Prescribes a robust fix** — make the deny the last matching rule (fragile), or use a `Cmnd_Alias` positive allow-list (robust), or tighten the over-broad grant; explains why "delete the legacy allow" patches symptom not model.
6. **Generalizes the lesson** — under last-match-wins, prefer positive allow-lists over allow-then-deny; `NOPASSWD: ALL` makes specific denies theater; `sudo -l` over eyeballing config.

## Part B — end
