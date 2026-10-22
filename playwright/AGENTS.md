# Playwright: Agent Guidelines

This folder is dedicated to **Playwright end-to-end testing** — test architecture, patterns, CI/CD integration, debugging, and tooling.

## 📁 Directory Structure

| Directory | Purpose | Agent Action |
|-----------|---------|--------------|
| `concept_notes/` | Evergreen Playwright concepts: selectors, fixtures, page object model, parallelization, retries, tracing, sharding, CI integration. | Create concept notes using `Templates/Technical-Deep-Dive.md`. |
| `flashcards/` | Spaced repetition cards for Playwright facts and gotchas. | Generate from concept notes using Obsidian SR syntax. |
| `research_in_progress/` | Active investigations, article deep dives, experiments. | Use `Templates/Research-to-Project Template.md`. |

## 📝 Conventions

- Content is in **English**.
- Use `[[WikiLinks]]` to connect to related backend/frontend testing notes.
- Link related System Design notes where relevant (e.g., CI/CD, visual regression).
- Tag Playwright notes with `[playwright, testing, e2e]` plus topic-specific tags.

## 🎯 Learning Philosophy (AI-era focus)

AI writes Playwright boilerplate, syntax, and happy-path tests well. Spending practice time on those is low-value. **What AI cannot do well is where the author's effort belongs.**

When generating exercises, research prompts, or concept notes in this folder, bias toward the following skills — these are the author's real area of value:

| Skill (learn deep) | Why AI can't replace it | Exercise shape |
|--------------------|------------------------|----------------|
| **What to test vs. what not to test** (test pyramid judgment) | AI over-tests implementation details; can't tell e2e from unit territory | Given a feature spec, classify each potential check as unit / integration / e2e and justify. Spot the over-tested and under-tested gaps. |
| **Suite architecture** (POM/Screenplay, fixtures, test data, isolation) | AI emits fragments, not a coherent scalable system | Take a flaky, duplicated 20-test file and restructure it: extract page objects, share fixtures, remove order dependencies. |
| **Flakiness diagnosis** (race conditions, auto-wait gaps, shared state, `networkidle` traps, test-order coupling) | AI "fixes" flakiness with `waitForTimeout` — which masks it | Given a flaky test + trace/log, find the root cause and fix it *without* arbitrary waits. Review a "fixed" test and identify whether the fix is real or a sleep. |
| **Selector & assertion quality** (robust locators, meaningful assertions, testing intent not implementation) | AI picks brittle selectors and writes tests that pass "for the wrong reason" | Review AI-generated tests: flag brittle selectors (`nth-child`, CSS classes), weak/no-op assertions, and tests that would stay green after a regression. |
| **CI/CD integration** (sharding, parallelization, retries strategy, what runs on push vs. PR vs. nightly, cost trade-offs) | This is architectural judgment, not generation | Given a repo + budget constraints, propose a CI matrix: what to shard, what to cache, which suites gate merge, retry policy. Justify. |
| **Reading & auditing AI output critically** | The killer skill — green-but-meaningless tests are the silent failure mode | Here is an AI-written test suite. Audit it for: false greens, missing edge cases, hidden order deps, anti-patterns. Rewrite the worst offender. |

### Anti-skill (do NOT over-invest practice here)
- Memorizing Playwright API surface (`page.click`, `expect`, etc.) — reference it, don't drill it.
- Writing happy-path boilerplate from scratch by hand.
- Config syntax memorization — let the tool/AI scaffold, then verify.

### When the user asks for an "exercise" in this folder
Default to a **diagnosis / review / design** task (left column) over a "write a test for X" task. Prefer exercises that:
1. Present existing (possibly AI-generated or deliberately broken) code for critique, **or**
2. Pose an architecture/tradeoff decision with constraints, **or**
3. Give a failing/flaky scenario and ask for root-cause analysis.

Avoid exercises that reduce to "produce a passing test for this feature" — that is exactly what AI already does, and low learning value.

## 🏷️ Frontmatter

Same as the root `AGENTS.md`:

```yaml
---
status: researching | analyzing | brainstorming | implemented | archived
tags: [playwright, testing, e2e]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

---

*Last updated: 2026-07-17*
