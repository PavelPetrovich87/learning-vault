<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
# Feedback Systems Over Conventions — Flashcards

<!-- #flashcards/harness-engineering -->

<!-- What is **"drift"** in software architecture, and why do AI agents make it worse? -->
<!-- ? -->
<!-- Drift is the silent accumulation of small decision inconsistencies — boundary ambiguity, responsibility confusion, naming variance — until architectural coherence is lost. AI agents amplify drift because they **optimize for the prompt, not the system**. They have no mental model of your architecture; they process tokens. A human might shortcut once; an agent does it consistently, at scale. -->
<!--SR:!2026-05-15,23,250-->

---

<!-- In **harness engineering**, what is the core argument for replacing conventions with **feedback systems**? -->
<!-- ? -->
<!-- Conventions fail for three structural reasons: **(1)** memorization burden grows with guideline length, **(2)** subjective decisions create gray zones, **(3)** compliance can't be verified mechanically. Feedback systems flip the model: instead of "learn rules, then write," the author writes first and the system responds with "what's wrong + how to fix it." Code converges to compliance through the loop — no upfront knowledge required. Think TypeScript: you don't memorize type rules, the compiler tells you. -->
<!--SR:!2026-05-01,1,170-->

---

<!-- What three criteria must a rule satisfy to qualify as an **architectural invariant** (not just a convention)? -->
<!-- ? -->
<!-- 1. **Binary evaluation** — unambiguous pass/fail, no "it depends" -->
<!-- 2. **Syntax-verifiable** — determined from source code analysis alone, no runtime needed -->
<!-- 3. **Locally verifiable** — determined by examining the target file alone, no project-wide scanning -->
<!-- If a rule meets all three, a lint tool can enforce it automatically. If it doesn't, it's a convention — and conventions drift. -->
<!--SR:!2026-05-02,2,170-->

---

<!-- Why must architecture lint error messages **include fix instructions**, not just report violations? -->
<!-- ? -->
<!-- Traditional lint says "wrong" — fine for formatting where the fix is obvious. But for structural/architectural rules, "wrong" alone causes the AI agent to **guess the fix — and guess wrong**. Effective error messages must be: **(1)** specific (not "invalid" but "X must be Y"), **(2)** include fix instructions, **(3)** self-contained (no external docs needed), **(4)** machine-readable (AI can parse into executable actions). This is the difference between a lint that blocks bad code and a lint that **converges** code to good architecture. -->
<!--SR:!2026-05-04,20,250-->

---

<!-- What does **"Same lint, same result"** mean as an architectural principle? -->
<!-- ? -->
<!-- If the feedback loop has the same exit condition (zero lint errors) and the same rules, then the output converges to the same structural standard **regardless of who wrote it** — junior dev, senior dev, or AI agent. The architecture becomes an emergent property of the toolchain, not of the author's skill or memory. This is why investing in validator tooling pays off more than investing in longer guideline documents. -->
<!--SR:!2026-05-23,25,230-->
