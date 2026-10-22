---
status: archived
exercise_type: incident_diagnosis
difficulty: beginner
pattern: searching
source_concepts:
  - "[[Searching Commands]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: "`which docker` Says It's Not Installed — But It Runs"

## Type
**Incident diagnosis** — a teammate has filed a confused ticket. They're sure a tool isn't installed, yet the tool clearly works. You need to figure out what's actually going on and correct the mental model that led to the false alarm. Friendly, single-concept, low-stakes.

## Difficulty
**Easy.** One concept, one observation, a clear contradiction that points at the domain. The trap is **tool selection**: the teammate reached for the wrong diagnostic and got a misleading answer. A strong answer names the *right* tool and explains *why* the wrong one lies.

## The Situation

**Where:** Acme WebOps — same shop as the permissions exercise. You are the senior. **Priya** is a mid-level developer who just joined the platform team.

**The system:** Priya's workstation. She was onboarded last week and is setting up her container workflow. Another dev told her "just type `docker run hello-world` to verify your install" — and it worked. But when Priya tried to "locate the binary" the way she's used to, she got nothing.

**The stakes:** Trivial — this is a workstation setup wrinkle, not an outage. But Priya has now escalated it as "docker isn't actually installed, something's faking it," and she's worried her machine is in a weird state. You'd like to (a) reassure her, (b) fix her mental model so she stops filing these tickets, and (c) leave her with the right tool for next time.

**The contradiction (high level):** `docker run hello-world` produces the expected "Hello from Docker!" output. But the command Priya used to "find where docker lives" returns empty, which she interprets as "not installed."

## What You Know

Neutral facts available before the persona speaks. Confirm or elaborate via the persona:

- `docker run hello-world` works and prints the canonical Docker greeting. Docker is functionally present.
- Priya ran a specific "find the binary" command and it returned **nothing** — no path, no output, just an empty line and back to the prompt.
- That empty result is the *entire* basis for Priya's "it's not installed" conclusion.
- Nothing on the machine is actually broken. No outage, no security issue. The only thing wrong is Priya's interpretation.

## The Ask

Walk Priya through producing:

1. **The actual situation** — if `docker run` works, then *something* is responding to the name `docker`. What are the candidates? Why might a "find the binary" command miss all of them?
2. **The one diagnostic command** that would tell Priya exactly what `docker` is on her machine — and an explanation of *why this tool works where the other didn't*.
3. **The mental-model correction** — what the tool Priya used is actually good for, and what it explicitly does *not* see. Make it a one-line rule she can remember.
4. **A preventive habit** — what should Priya reach for first next time she wants to know "what will actually run when I type this name?"

## Related Concept Notes
- [[Searching Commands]] — the difference between `which`/`whereis` (only find files on `PATH`) and `type` (the shell's authoritative answer, including aliases, functions, and builtins).

---

*Last updated: 2026-07-17*
