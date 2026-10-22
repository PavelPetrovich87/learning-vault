#flashcards/system-design

# Functional vs Non-Functional Requirements-Flashcards

---

How do you tell a functional requirement from a non-functional one — two one-line tests?
?
FR = "what can the user do?" — test: can you demonstrate it ("look, I publish a post → it appears in the feed"). NFR = "how well, under what conditions?" — test: measured by a number or shows up under load/failure. Stuck bullet like "search with high speed" = two requirements glued, neither testable.

---

The interviewer says the service should be "practically instant". Is that a requirement? What do you do with it?
?
Not a requirement — an NFR without a number doesn't exist. Ground it via the latency table: intercontinental (RTT ~150ms) → candidate p95 ≤ 1s; intra-DC (tens of ms) → p95 ≤ 100ms. Geography changes the number — ask which one. Otherwise mark TBD and dig via a behavior question.

---

At requirements-gathering, what marks a question as solution-shaped rather than behavior-shaped?
?
It names a specific component, technology, or mechanism. "How does the system behave during an outage?" is behavior-shaped (fine); "How will we use Kafka for delivery?" is a solution in disguise. The marker is the named component — not the presence of the word "how".
