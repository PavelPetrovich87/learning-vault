---
status: archived
exercise_type: incident_diagnosis
difficulty: intermediate
pattern: shell
source_concepts:
  - "[[Linux Shell]]"
created: 2026-07-17
updated: 2026-10-21
---

# Exercise: The One-Liner That Works at the Prompt But Dies in Cron

## Type
**Incident diagnosis** — an engineer built a reporting one-liner, tested it at the prompt, it worked perfectly, then scheduled it in cron — where it fails silently every night. You are the senior who recognizes the category immediately. Medium ambiguity: "works interactively, fails automated" has several possible causes, and the engineer has been debugging the *command* instead of the *environment*.

## Difficulty
**Medium.** Single concept (the non-interactive shell vs the interactive shell), but the symptom class is broad and the engineer has anchored on the wrong thing (the script's contents) instead of the right thing (the environment the script runs in). A strong answer names the specific environment differences between interactive and non-interactive shells and pinpoints which one bit this script.

## The Situation

**Where:** Acme WebOps — the analytics product again. You are the senior SRE. **Dana** is a data analyst who writes her own reporting scripts.

**The system:** A reporting box that produces nightly summaries. Dana wrote a one-liner that pulls data, formats it, and mails a report. She iterated on it for an afternoon, got it perfect, dropped it into crontab, and went home. The report has not arrived once.

**The stakes:** Low-stakes operationally (a missing internal report), but high-stakes for Dana's credibility — she's presented this report to leadership as "automated as of this week," and it's been silently failing for five nights.

**The contradiction:** The command, run by hand at the prompt in Dana's terminal, produces the correct report every time. Run from cron, it fails — and the failure mode is "the report doesn't arrive," which is ambiguous. The script's *contents* are identical in both cases.

## What You Know

Neutral facts available before the persona speaks. Confirm via the persona:

- Dana runs the one-liner by hand: it works, mail arrives, report is correct.
- Dana pasted the *exact same command line* into her crontab, scheduled for 02:00 nightly.
- Five mornings in a row, the report has not arrived. No partial output, nothing.
- Dana has re-tested the command by hand each morning after noticing the failure: it works by hand, every time.
- The crontab entry appears syntactically fine (right schedule, right user).
- Dana has been tweaking the *command itself* — quoting, flags, escaping — trying to find the bug.

## The Ask

Walk Dana through producing:

1. **The category of problem.** "Works interactively, fails in cron" is a well-known class. Name it, and explain *why* the same bytes behave differently in the two contexts.
2. **The specific environment differences** between the interactive shell Dana tested in and the non-interactive shell cron spawns. List them — at minimum three.
3. **How to actually find which one bit her script**, without guessing. What's the disciplined way to capture what cron's shell *sees* vs what her interactive shell sees?
4. **The generalizable habit** — how Dana should test a cron-bound command *in an environment that matches cron's environment*, not her interactive shell, before ever scheduling it.

## Related Concept Notes
- [[Linux Shell]] — interactive vs non-interactive shells, what startup files each reads, why `PATH`, aliases, functions, and environment variables differ between the prompt and cron.

---

*Last updated: 2026-07-17*
