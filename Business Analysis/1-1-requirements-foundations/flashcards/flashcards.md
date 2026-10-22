#flashcards/business-analysis

# 1-1 Requirements Foundations — Flashcards

---

Name the three requirement levels top-down. Which question does each level answer?
?
Business requirement → User requirement → Functional requirement.
Level questions: **WHY** (why the organisation needs it) → **WHY THE USER** (what they must be able to do) → **WHAT THE SYSTEM DOES** (behaviour, "shall…").
Mnemonic: Why → Why-the-user → What-the-system-does. This is a causal chain, not three parallel boxes: the business goal generates user tasks, those generate FRs.
Source: [[1-1 Requirements Foundations — Concept Note]]

---

Which three facets of "requirement" do Wiegers/Beatty distinguish (after Sommerville/Sawyer)?
?
After Sommerville/Sawyer — three facets: **behaviour** (what the system does), **properties** (what characteristics it has), **constraints** (within what bounds it is built).
These are three different places to go wrong: you can implement behaviour flawlessly and still miss the constraint (forget performance) — the system "works, but is unfit".
Rule: action verb → functional; "what is it" → property; "within what bounds" → constraint.
Source: [[1-1 Requirements Foundations — Concept Note]]

---

Which facet does each of these belong to: (a) "runs on PostgreSQL 15", (b) "sends an email on status change", (c) "available in 5 languages"?
?
(a) constraint — a frame within which the system is built;
(b) behaviour (a functional requirement) — a system action;
(c) property (a quality attribute) — a characteristic, "what it is". Multilingual support is a grey zone: a property with a tinge of constraint (if the customer dictates "mandatory for our market", it feels like a constraint).
Source: [[1-1 Requirements Foundations — Concept Note]]

---

How does a feature differ from a functional requirement and from a quality attribute?
?
**Feature** is a **granularity** notion: a named product capability = **a group** of related FRs, valuable to the user. It sits between a user requirement and an individual FR.
"Bookmarks" is a feature; "the system shall allow renaming a bookmark" is an FR; "bookmarks load in 200 ms" is a quality attribute.
The book translates feature as "характеристика", but that conflicts with quality attributes — the course fixes "function (feature)".
Source: [[1-1 Requirements Foundations — Concept Note]]

---

How does a business rule differ from a functional requirement? Why does this distinction matter?
?
A **business rule** exists **outside** the software system (company policy, law, standard) — it can be enforced manually, without software. The system merely *obeys* it, spawning FRs.
An **FR** is system behaviour; it requires the system to be implemented.
Example: "the coworking charges a surcharge for the 4-hour limit" — a business rule; "the system shall block bookings longer than 4 hours" — an FR.
Chain: **business rule → feature → many FRs** (but a rule can also yield a single FR, a quality attribute, or nothing — see the refined distinction).
Why it matters: the rule can change (4→6 hours); if kept separate you change one constant, not a pile of FRs across the SRS.
Source: [[1-1 Requirements Foundations — Concept Note]]

---

Which requirement types "permeate" all three levels (do not belong to one)?
?
These are **NOT** levels — they are types applicable to all three levels:
- **Quality attributes (NFRs)** — how well it works (performance, availability).
- **External interfaces** — what it interacts with (other systems, devices, UI).
- **Constraints** — bounds (technology, budget, regulations).
- **Business rules** — policies outside the system that spawn FRs.
Common mistake: "all non-functional = quality attributes". No — constraints and external interfaces are also non-functional.
Permeation ≠ membership: knowing the type does not tell you the level.
Source: [[1-1 Requirements Foundations — Concept Note]]

---

What is "gold plating" and how do you spot it?
?
Gold plating is a feature added without a user-requirement parent, "because it seemed useful".
Marker: ask "which user requirement does this FR serve?". If there's no answer, or "well, it's generally useful" — red flag.
Example: "the system shall send a daily booking-summary email every morning" with no user requirement it serves. A candidate for deletion / priority lowering.
Source: [[1-1 Requirements Foundations — Concept Note]]

---

State the requirements economics numbers (Wiegers).
?
- **40–50%** of all product defects are seeded at the requirements stage (Davis, 2005).
- Rework of "finished" work consumes **30–50%** of the dev budget; requirements errors = **70–85%** of rework cost.
- Defect-cost curve: **$1** while working on it → **$2–3** at design → **$100+** in production (Wiegers's case: $200 via internal review vs $4200 from a user, ×21).
Takeaway: requirements are the biggest leverage point on quality; defects there are the most expensive.
Source: [[1-1 Requirements Foundations — Concept Note]]

---

Name the six symptoms of poor requirements (Wiegers/Beatty, Ch. 1). Which symptom describes "one text, different readings by readers"? Which — "every week the PM throws in a new feature"?
?
Six symptoms: (1) insufficient user involvement; (2) careless planning; (3) scope creep; (4) ambiguous requirements; (5) gold plating; (6) missing user classes.

"One text, different readings by different readers" → #4 (ambiguous). Marker: informal reading does *not* reveal ambiguity — everyone "understood their own way, and it made sense to each". Remedy: official review with different stakeholders + prototype/tests.

"Every week the PM throws in a new feature" → #3 (scope creep). Remedy: clarify business goals/vision, assess the impact of each new feature, planning buffers; in agile — fix iteration budget, adjust scope.

Scope creep vs gold plating: scope creep is growth "from outside" (initiated by the customer), gold plating is "from inside" (the developer adds unasked).
Source: [[1-1 Requirements Foundations — Concept Note]]

---

What is the difference between an "under-specified" requirement and an "ambiguous" requirement? Which symptom does each belong to?
?
These are **different symptoms**:

- **Under-specified** = not detailed enough → symptom #2 (careless planning). Example: "the form should be convenient" — nobody asked for details, it wasn't fleshed out.

- **Ambiguous** = admits multiple interpretations → symptom #4 (ambiguous requirements). Example: "the form should be convenient" — the designer read it as cards, the developer as a long form, the tester as a wizard. All three are sure they understood correctly.

**Distinguishing marker:** if one text yields different *sensible* readings by different readers → #4 (ambiguous), not #2. Ambiguity is insidious because it is invisible on a casual read: every reader is sure they got it. It surfaces only when two readers compare understandings — hence "official review", not "I reread and it's fine".
Source: [[1-1 Requirements Foundations — Concept Note]]

---

What is the difference between an elicitation failure and a specification failure? Marker: "the requirement was only verbal in a call, not recorded" — which failure is it?
?
These are **different primary failures** of requirements-development subprocesses:

- **Elicitation failure** — the requirement *was never known*, nobody knew about it. The right people weren't talked to, the work wasn't observed, the task wasn't collected.

- **Specification failure** — the requirement *was elicited but never persisted*: only verbal in a call, in chat, "on the fly". Everyone understood it their own way, and the customer "meant something else".

**Marker:** "verbal but not recorded in a persistent container" (SRS, backlog user story, document) = specification failure, **not** elicitation. The requirement was known, but not nailed down.

**Remedy:** after any call/interview — record the user requirement as a user story in the backlog and ask the customer to explicitly accept it before coding starts. Then "meant" becomes "written and agreed".

Practical takeaway: different failures — different remedies. Not collecting requirements → improve elicitation (interviews, observation). Collecting but losing them → improve specification (containers, acceptance).
Source: [[1-1 Requirements Foundations — Concept Note]]

---

An item in your task list says "calculate what breaks in tariffs and reports if we raise the booking window from 7 to 30 days". Is this requirements *analysis* (development) or *impact assessment* (management)? One-line marker?
?
**Impact assessment → management.**
Marker: "calculate what breaks if we change X, where X is already agreed" → management. The booking window is an existing, agreed rule; the work produces a *consequence estimate* for a change decision, not refined requirement content.
Analysis works on the content of requirements that are still being born (separate tasks from FRs, decompose, prioritise); its product is structured requirements. Requirements management owns agreed baselines and their changes.
Keyword trap: "calculate/посчитать" smells like analysis — classify by the object of work, not by the verb.
Source: [[1-1 Requirements Foundations — Concept Note]]

---

"Sort the week's collected wishes: separate user tasks from proposed solutions and policies." Axis A (is it a requirement: product / project / neither)? Axis B (object of work: development subprocess / management / —)?
?
**A: neither** (it is an action, not a property that must be). **B: development → analysis.**
The X-rule: *an action over X is not X* — sorting requirements is work WITH requirements, but the sorting itself is not a requirement. Yet the object of the work IS requirements (raw wishes being structured), so axis B is filled.
Key law: the axes are independent. "Neither" on A does not force «—» on B. Watching users work → A neither, B elicitation. Fixing a baseline → A neither, B management.
Source: [[1-1 Requirements Foundations — Concept Note]]

---

"Record a short training video for duty admins" and "prepare a one-off export script from 1C". For each: axis A (product / project / neither) and axis B (work with requirements?).
?
Both: **A — project requirement** (user docs/training; transition requirement for the script — one-off, not part of the final product, never belongs in the SRS). **B — «—»** for both actions.
Axis B marker: the object of the work is a video / code — NOT requirements. "Specification" in the requirements sense means writing requirement texts, not producing any document whatsoever.
Contrast: "formulate WHICH data must migrate from 1C" would be specification (development) — the product is a requirement formulation, not code.
Source: [[1-1 Requirements Foundations — Concept Note]]
