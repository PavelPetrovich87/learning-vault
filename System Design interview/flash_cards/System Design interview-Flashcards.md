#flashcards/system-design

# System Design interview-Flashcards

---

On a system design interview (or in any product brief), how do I systematically find clarifying questions instead of memorizing question lists?
?
Two passes. Pass 1 (read the text): explicit holes — a label without a number ("fast", "like the big messengers") → ask "how much?"; a number without an owner → ask "who set it, how accurate?". The question form is mechanical here: the marker in the text dictates the probe. Pass 2 (probe the silence): what the brief never mentions is invisible — questions come first there, and the hole is discovered by the missing answer (see the probe × target card).

---

DAU and actions-per-day feel like "not enough" for requirements questions. What is the complete universal layer of clarifying questions?
?
They cover only load parameters (frame field 4). Every frame field generates question-holes: goals & horizon → "how much growth in a year?"; FT two signs → "what do we consciously NOT build?"; NFT → a number for every label; constraints → budget / team / deadline; scenarios → peak profile. A question = a hole in a frame field.

---

What are the two layers of clarifying questions, and what marks a weak question list?
?
Universal layer (holes in the frame fields — identical for any domain) + domain layer (list the domain entities and ask "what happens to it when…?" for each: message, session, presence status, a smart water dispenser). Weak list marker: universal only ("accounts, payments, sessions"), zero domain questions.

---

What is the test for the correct form of a clarifying question?
?
Behavior question, not solution question: "what happens if two requests compete?" — not "how will we handle competing requests?" Marker of a solution question: a named component or technology inside the question.

---

Every label in a brief ("fast", "reliable", "like the big messengers") must end up as exactly one of two things. Which?
?
Exactly one of two things: an NFR with a number ("p95 ≤ 300 ms") or an out-of-scope line ("not in v1, because…"). A label that stays a label is an unpriced promise.

---

What is the probe formula that generates clarifying questions where the brief is silent?
?
question = probe × target. Probes: "how much?" (quantify an entity) · "how often? when is it worst?" (frequency, peak profile) · "what happens if it fails / goes offline / doubles?" (behavior) · "who set this number?" (owner) · "for how long?" (retention, horizon) · "where?" (geography) · "what do we NOT build?" (boundary). Targets: frame fields (universal layer) and domain entities (domain layer). If the target has no answer — that is the hole.

---

The interviewer says: "Design Uber." What are the very first questions to ask?
?
The brief does not exist yet — first job is to extract it. Walk the frame as a questionnaire, out loud: (1) what exactly is "Uber" — rides, Eats, freight? MVP or mature? platform? (2) who are the users and the core scenario in one sentence? (3) scale — DAU, actions/day, one city or the world? (4) what is in v1 and what is out? (5) quality expectations — a number for every hope ("matching in how many seconds? availability vs correctness on failure?"); (6) constraints (team, deadline). On "assume whatever" → state an assumption with a number aloud and move on. Close with an echo: "So we build X for Y at scale Z, without W — correct?" After that, detectors and probes run against your own notes.
