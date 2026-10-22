---
status: archived
exercise_type: incident_diagnosis
difficulty: intermediate
pattern: processes
source_concepts:
  - "[[Linux Processes]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: `kill -9` Did Nothing — The Process Won't Die

## Type
**Incident diagnosis** — an operator hit a wedged process with `kill -9` and the process is still sitting there. They've now hit it with `kill -9` *five more times*, escalating in frustration. You are the senior who explains why "more SIGKILL" is the wrong response and what the actual diagnostic fork is. Medium ambiguity: the symptom ("process ignores kill -9") has two distinct causes that demand opposite fixes.

## Difficulty
**Medium.** Single concept (process states and signal delivery), but the symptom is genuinely ambiguous: a process that ignores `kill -9` is in one of *two* states — zombie or uninterruptible sleep (D-state) — and they require completely different responses. Reaching for `kill -9` harder fixes neither. A strong answer identifies *which* state by reading the right column, explains why SIGKILL can't touch either, and names the actual fix for each.

## The Situation

**Where:** Acme WebOps — a shared build server. You are the senior. **Ravi** is a developer who was trying to clean up a stuck `git` operation that had wedged during a large clone.

**The system:** A build box several developers SSH into. Ravi ran a `git clone` of a huge repo over NFS; it hung; he tried to kill it; the kill "didn't work"; he's now hammered it with `kill -9` six times and it's still in `ps`.

**The stakes:** Low — a stuck process on a shared dev box, no customer impact. But Ravi is escalating (he's about to ask for a reboot "because nothing else works"), and the reboot would kick off three other developers' work. The cheap fix is a diagnosis, not a reboot.

**The contradiction:** `kill -9` is supposed to be the unstoppable signal — the "nuclear option." Ravi used it six times. The process is still there. To Ravi, this means "kill is broken." It isn't. It means the process is in a state where SIGKILL is *defined* to have no effect, and the next move is diagnostic, not force.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- The stuck process is still listed by `ps`.
- Ravi sent `kill -9 <pid>` six times. No error from `kill` itself; the signal was delivered (or queued). The process is unchanged.
- Ravi is about to request a reboot of the shared box.
- Three other developers are actively using the box.
- The original trigger was a large `git clone` over the network/NFS that hung.

## The Ask

Walk Ravi through producing:

1. **Why `kill -9` "did nothing" — and why doing it six more times won't help.** What does SIGKILL actually guarantee, and what does it *not* guarantee? Name the states in which SIGKILL is defined to have no effect.
2. **The single diagnostic** that tells you which of those states you're in. What column of what tool, and what letter are you looking for? (There are two candidate letters; each means a different thing and demands a different fix.)
3. **The fix for each state**, and why they're opposite — one is "kill something *else*," the other is "diagnose hardware/network, don't reach for kill at all."
4. **The generalizable lesson** about `kill -9` — when it's the right tool, when it's useless, and why "more SIGKILL" is never the answer.

## Related Concept Notes
- [[Linux Processes]] — process states (`R`, `S`, `Z`, `D`), signal delivery semantics, why zombies and D-state ignore SIGKILL, the fork/exec distinction, and parent/child relationships.

---

*Last updated: 2026-07-17*
