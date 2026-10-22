---
artifact_type: exercise_convention
topic: linux_administration
created: 2026-07-17
updated: 2026-07-17
---

# Linux Administration Exercises — Agent-Driven Incident Diagnosis

> [!abstract] What this folder is
> A set of hands-on Linux administration exercises built for the **agent era**. The goal is *not* to memorize flags or syntax — an agent can look those up. The goal is to train the judgment that AI cannot install in your head: **recognizing a problem from its symptoms, forming a hypothesis, choosing a diagnostic path, and defending the decision.**

## The pedagogical bet

Recall is cheap and on-demand now. What compounds is the **mental model**: knowing *why* `chmod -R 644` bricks a directory tree, *why* `kill -9` does nothing to a zombie, *why* your shell one-liner works at the prompt but dies in cron. These exercises stage realistic incidents and force you to *diagnose*, not to recite. Every exercise is built around a documented gotcha from a concept note in `../concept_notes/`.

## How to run an exercise

1. Open the exercise folder as the workspace root in a **fresh agent chat**.
2. The agent auto-reads `AGENTS.md`, greets you **in character**, and delivers the incident (it roleplays a colleague reporting a problem).
3. You brainstorm together. The agent **guides Socratically** — it never hands you the answer, never thinks for you. It asks probing questions, surfaces facts only when you ask for them, and pushes back on weak reasoning.
4. Once you converge on a hypothesis and an approach, tell the agent you're ready to **test**.
5. The agent reads `symptoms.md` Part B (the root cause + reference), validates your approach against it, and runs the **debrief**: what you nailed, what you missed, what to study next.
6. The agent **proposes the next exercise** based on your gaps.

## The 3-file structure

Every exercise folder contains exactly three files:

```
NN_Difficulty_Concept_Incident/
├── scenario.md     # candidate-visible — the situation, what you know, the ask
├── symptoms.md     # persona-only — Part A (symptoms, read throughout) + Part B (root cause, read at debrief only)
└── AGENTS.md       # the persona — role, behavior, diagnostic flow, rubric, opening line
```

### `scenario.md` — what the candidate sees

The immutable, spoiler-free framing. Frontmatter fields:

```yaml
---
exercise_type: incident_diagnosis
difficulty: easy | medium | hard        # lowercased value
pattern: permissions | processes | systemd | sudo | …   # domain tag
source_concepts:                         # wikilinks to concept_notes/
  - "[[Linux Permissions]]"
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Sections: `# Exercise: {Title}` → `## Type` → `## Difficulty` → `## The Situation` → `## What You Know` → `## The Ask` → `## Related Concept Notes` → `*Last updated*`.

### `symptoms.md` — persona-only, two deliberately delimited zones

```yaml
---
artifact_type: exercise_symptoms
exercise: "[[01_Easy_Permissions_Chmod_Bricked_Tree]]"
visibility: persona_only
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

- **Part A — Observable Symptoms.** The incident as the persona experiences it: error messages, timeline, what the junior already tried, monitoring clues. The persona reads this throughout and uses it as the roleplay script. **Contains no diagnosis, no root cause.**
- A hard divider, then:
- `> [!danger] Part B — Root Cause & Reference Solution. The persona reads this ONLY at the test/debrief phase.`
- **Part B** — actual root cause · the mechanism (why it happened) · the correct approach/fix · hidden diagnostic traps · the rubric ("what a strong diagnosis covers").

The two-zone split is the core mechanic. It lets a single persona agent roleplay authentically *without knowing the answer* during the brainstorm — then become the examiner once you're ready to test.

### `AGENTS.md` — the persona

No frontmatter (it is an agent-instruction file, not an Obsidian note). Sections: `# Agent Persona — Exercise: {Title}` → `## Your Role` → `## The Files` (the two-phase reading protocol) → `## How You Behave` → `## The Diagnostic Flow` → `## Question Bank` → `## Pacing & Closing` → `## The Rubric` → `## Debrief & Next-Exercise Proposal` → `## Your Opening Line`.

## Difficulty scales on two compounding axes

| Axis | Easy | Medium | Hard |
|------|------|--------|------|
| **Concepts in play** | 1 | 1–2 | 2–3+ |
| **Symptom clarity** | Points clearly at the concept | Could be 2–3 things; must rule out | Someone already misdiagnosed + worsened it |

The hardest exercises are **second-line diagnoses**: a junior applied a plausible-but-wrong fix, and you must untangle the original fault *and* the damage. That is the work AI struggles to do for you, because it requires holding the system's state in your head.

## Naming convention

- Per-concept: `NN_Difficulty_Concept_Incident/` (e.g. `01_Easy_Permissions_Chmod_Bricked_Tree/`). `NN` is sequence of creation, **not** difficulty rank.
- Multi-concept (integration tier): `M{N}_Difficulty_…/` (e.g. `M1_Easy_Redis_Auto_Started/`). The `M` prefix marks exercises that sit "one level up" — they weave together multiple concept notes.

## The exercise catalog

**Per-concept (10):**

| # | Folder | Source concept | Difficulty |
|---|--------|----------------|------------|
| 01 | `01_Easy_Permissions_Chmod_Bricked_Tree` | [[Linux Permissions]] | Easy |
| 02 | `02_Easy_Searching_Which_Finds_Nothing` | [[Searching Commands]] | Easy |
| 03 | `03_Medium_Kernel_Mystery_Exit_137` | [[Unix Like Kernel]] | Medium |
| 04 | `04_Medium_Shell_Works_In_Terminal_Breaks_In_Cron` | [[Linux Shell]] | Medium |
| 05 | `05_Medium_Processes_Kill9_Did_Nothing` | [[Linux Processes]] | Medium |
| 06 | `06_Medium_Packages_UsrLocal_Orphan` | [[Linux Package Management]] | Medium |
| 07 | `07_Medium_Users_UsermodG_Locked_Out_Of_Sudo` | [[Users and Groups]] | Medium |
| 08 | `08_Medium_Systemd_Edited_Unit_No_Effect` | [[Linux Services and Systemd]] | Medium |
| 09 | `09_Medium_Archiving_Tarbomb_In_Home` | [[Archiving Commands]] | Medium |
| 10 | `10_Hard_Sudo_Last_Match_Wins` | [[Sudo]] · [[Root User vs Regular User]] | Hard |

**Multi-concept (3):**

| # | Folder | Source concepts | Difficulty |
|---|--------|-----------------|------------|
| M1 | `M1_Easy_Redis_Auto_Started` | [[Linux Package Management]] · [[Linux Services and Systemd]] | Easy |
| M2 | `M2_Medium_Slow_Load_High_Cpu_Idle` | [[Linux Processes]] · [[Linux Services and Systemd]] | Medium |
| M3 | `M3_Hard_Sudo_Path_Escalation` | [[Sudo]] · [[Linux Permissions]] · [[Linux Services and Systemd]] | Hard |

## Suggested ordering for a learner

1. Start with the Easy pair (01, 02) to learn the exercise format.
2. Work the Medium per-concept set (03–09) in any order — each is self-contained.
3. Tackle the Hard per-concept exercise (10) once sudoers internals feel familiar.
4. Move to the integration tier (M1 → M2 → M3) to practice weaving concepts under pressure.

---

*Last updated: 2026-07-17*
