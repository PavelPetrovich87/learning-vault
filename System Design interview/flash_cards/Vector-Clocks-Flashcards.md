#flashcards/system-design

# 🎴 Vector Clocks & Versioning Flashcards

How do ==Vector Clocks== represent a version of a data item? :: As a list of `[Server, VersionNumber]` pairs, for example, `D([S1, 1], [S2, 2])`.
<!--SR:!2026-06-05,43,250-->

---

Two vector clocks are compared. Clock A: `[S1: 2, S2: 1]`. Clock B: `[S1: 1, S2: 1]`. Is there a conflict?
?
**No.** Clock A is a direct descendant of Clock B because all its version counters are greater than or equal to Clock B's.
<!--SR:!2026-05-20,28,230-->

---

Two vector clocks are compared. Clock A: `[S1: 2, S2: 1]`. Clock B: `[S1: 1, S2: 2]`. How does the system handle this?
?
**Conflict detected.** Since Clock A has a higher S1 value and Clock B has a higher S2 value, they have diverged. The system will typically return both "sibling" values to the client for manual resolution.
<!--SR:!2026-06-10,48,250-->

---

What is the main downside of a distributed system delegating conflict resolution via Vector Clocks? :: **Client-Side Complexity.** The application logic must now include code to merge conflicting data (e.g., merging two shopping cart versions).
<!--SR:!2026-04-30,8,230-->

---

What is ==Clock Pruning== in the context of Vector Clocks?
?
A mechanism to limit the size of the vector clock by removing the oldest `[Server, Version]` pairs when the list exceeds a pre-defined threshold, preventing metadata from consuming too much storage.
<!--SR:!2026-05-10,17,210-->

---

Why are Vector Clocks preferred over physical timestamps for conflict detection in multi-master systems?
?
**Physical Clock Drift.** Different servers can have unsynchronized clocks, leading to "Last-Write-Wins" deleting newer data. Vector Clocks track **Causality** (Logical time) rather than physical time.
<!--SR:!2026-06-08,47,250-->

---

A vector clock `D([S1, 10], [S2, 5])` receives an update from Server S2. What is the resulting clock? :: `D([S1, 10], [S2, 6])`.
<!--SR:!2026-04-24,24,250-->
