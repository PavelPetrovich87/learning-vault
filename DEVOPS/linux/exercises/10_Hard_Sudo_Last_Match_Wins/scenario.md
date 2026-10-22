---
status: archived
exercise_type: incident_diagnosis
difficulty: advanced
pattern: sudo
source_concepts:
  - "[[Sudo]]"
  - "[[Root User vs Regular User]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The Sudoers Rule That Should Have Blocked Her — But Didn't

## Type
**Incident diagnosis + policy debugging** — a security auditor found that Alice, despite an explicit `!/usr/bin/passwd` denial in sudoers, can still run `passwd`. The team assumed the rule was "broken"; they've been poking at the syntax for a day. You are the senior who recognizes that the rule *works exactly as written* — they just don't know how sudo evaluates rules. Hard difficulty: the symptom is subtle, the cause is sudoers' least-intuitive evaluation model, and a wrong remediation (moving rules around blindly) makes it worse.

## Difficulty
**Hard.** Single domain (sudoers policy) but the cause is sudo's **last-match-wins** evaluation order — the rule people *think* is being ignored is, in fact, being correctly overridden by a later, more-permissive rule they forgot was there. The remediation requires understanding the evaluation model, the drop-in include order, and the right way to express a denial that actually sticks. A strong answer names the model, traces the rule chain, distinguishes the *main file* from *drop-ins*, and prescribes the canonical fix (deny comes *last*, or use a Cmnd_Alias, or use the override-via-drop-in pattern).

## The Situation

**Where:** Acme WebOps — a hardened bastion host. You are the senior security engineer. **Imani** is the compliance auditor; **Owen** is the junior admin who wrote the rule.

**The system:** A jump host that several operators sudo through. The sudoers policy is the primary control surface. Owen recently added a rule to forbid operators from running `passwd` (the team is moving to centrally-managed passwords and doesn't want operators changing local ones). The rule looks correct. It doesn't work.

**The stakes:** Real. This is a *compliance control*: Imani needs to attest that operators cannot run `passwd`. She just demonstrated that Alice (a representative operator) *can*. Either the rule is broken (bad) or the team doesn't understand their own policy (also bad, and arguably worse — it means other rules may be ineffective too). The wrong fix — shuffling rules without understanding the evaluation model — could silently break *other* rules.

**The misdirection:** Owen has been debugging the **syntax** of the denial line for a day. The syntax is fine. The problem is *where the rule sits in the evaluation order* relative to another rule Owen forgot about — and how sudo's drop-in include model layers the whole thing.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- The sudoers file contains a rule like `alice ALL=(ALL) /usr/bin/passwd` *somewhere* (legacy — Alice used to be allowed to manage local passwords).
- Owen added `alice ALL=(ALL) !/usr/bin/passwd` to deny it.
- Alice can still run `sudo passwd`. Imani verified it.
- Owen has spent a day tweaking the syntax of the denial (quotes, `NOPASSWD:` ordering, `runas` spec) — nothing changes.
- The system uses `/etc/sudoers.d/` drop-ins (via `#includedir` in the main sudoers file).
- Both rules are syntactically valid (`visudo -c` passes).
- Owen is now considering removing the legacy allow rule entirely as a workaround.

## The Ask

Walk Imani and Owen through producing:

1. **The evaluation model.** How does sudo decide whether to allow or deny a given command when *multiple* rules match the same user? Name it precisely — this is the heart of the exercise.
2. **The exact reason the denial doesn't work**, traced through the actual rule chain. Which rule wins, and *why*? (Order matters — and there's a subtlety about main-file vs drop-in ordering.)
3. **The diagnostic command** that shows the *effective* policy for Alice — i.e. what sudo itself concludes she may run, after all rules are evaluated. (It is not "stare at `/etc/sudoers`.")
4. **The canonical fix** — and why Owen's instinct (delete the legacy allow rule) is riskier than the idiomatic alternative. Cover at least two correct approaches.
5. **The generalizable model** — last-match-wins, drop-in evaluation order, the right way to express "allow most things, deny these specific things" so the denial actually sticks.

## Related Concept Notes
- [[Sudo]] — last-match-wins evaluation, drop-in include order (`#includedir` reads *after* the main file → drop-ins win), aliases (`Cmnd_Alias`), why `sudo -l` is the effective-policy diagnostic.
- [[Root User vs Regular User]] — the privilege split sudo bridges; why a policy gap on `passwd` is a real security concern.

---

*Last updated: 2026-07-17*
