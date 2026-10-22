<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
# The Self-Enforcing Repository — Flashcards

<!-- #flashcards/harness-engineering -->

<!-- What are the **three pillars** of a self-enforcing repository for AI-era development, and how do they relate? -->
<!-- ? -->
<!-- 1. **Minimize hallucination surface** — visible code (ownership > dependency), declarative tools (finite vocabulary > complex API), minimal abstraction layers. This is **prevention** — fewer ways to go wrong. -->
<!-- 2. **Define invariants, not conventions** — rules that are binary, syntax-verifiable, and locally verifiable. This is **specification** — what "correct" means, mechanically. -->
<!-- 3. **Build convergent feedback loops** — validators with fix instructions, same exit condition for all authors. This is **enforcement** — how violations get corrected. -->
<!-- They work as defense in depth: Layer 1 reduces the space of possible errors, Layer 2 defines which errors are unacceptable, Layer 3 catches and corrects what slips through. -->
<!--SR:!2026-05-01,1,170-->

---

<!-- Why do conventions fail **identically** for humans and AI agents, despite failing for different underlying reasons? -->
<!-- ? -->
<!-- Three structural failure modes map across both: -->
<!-- | Mode | Human | AI | -->
<!-- |---|---|---| -->
<!-- | **Memorization** | Forgets the rule | Context window overflows, accuracy drops | -->
<!-- | **Subjectivity** | Interprets differently | Picks statistically likely pattern from training data | -->
<!-- | **Verification** | Reviewer misses it | No validator checks it | -->
<!-- Different mechanisms, same result: **drift** — slow, silent architectural decay until coherence is fiction. -->
<!--SR:!2026-05-03,19,250-->

---

<!-- In the context of **AI-assisted development**, what does "the repository must be a self-enforcing system, not a collection of suggestions" mean concretely? -->
<!-- ? -->
<!-- It means three things must be true: -->
<!-- 1. **Tooling choices** make the wrong thing hard to write (visible code, declarative primitives, low abstraction) -->
<!-- 2. **Architectural rules** are encoded as machine-checkable invariants, not as guidelines in docs -->
<!-- 3. **Validators** run automatically (pre-commit + CI), return errors with fix instructions, and converge all authors to the same standard -->
<!-- If any leg is missing — you have suggestions, not enforcement. And AI agents treat suggestions as optional. -->
<!--SR:!2026-05-04,6,210-->
