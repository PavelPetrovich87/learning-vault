<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/system-design -->

# 🎴 Quorum Consensus & Consistency Flashcards

<!-- What specific mathematical relationship between $W$ (write quorum), $R$ (read quorum), and $N$ (replicas) guarantees **Strong Consistency**? :: **$W + R > N$**. -->
<!--SR:!2026-06-24,57,250-->

---

<!-- In a cluster where $N=5$, you set $W=1$ and $R=1$. What is the primary benefit and the primary risk? -->
<!-- ? -->
<!-- **Benefit:** Maximum speed (latency) for both reads and writes. -->
<!-- **Risk:** Extreme data inconsistency. You could read a value that is dozens of versions old because the reader and writer have zero overlap. -->
<!--SR:!2026-06-08,46,250-->

---

<!-- A system configured with $W=Q$ (Quorum) and $R=Q$ is unavailable for writes if more than ==half ($N/2$)== nodes fail. -->
<!--SR:!2026-05-01,8,210-->

---

<!-- Why is "Strong Consistency" problematic for high-availability systems according to the CAP theorem? -->
<!-- ? -->
<!-- During a network partition, a strongly consistent system must refuse writes if it cannot reach its quorum. This sacrifices **Availability** to ensure that no stale data is ever read. -->
<!--SR:!2026-05-11,18,230-->

---

<!-- What is ==Eventual Consistency==? -->
<!-- ? -->
<!-- A model where replicas can temporarily diverge (stale reads are possible), but the system guarantees that all replicas will converge to the same state if no more updates are made. -->
<!--SR:!2026-06-14,52,250-->

---

<!-- In an eventually consistent system, if two clients update the same key simultaneously, how is the conflict resolved? :: Through conflict resolution mechanisms such as **Vector Clocks**, **Last-Write-Wins (LWW)**, or application-level merges. -->
<!--SR:!2026-06-12,50,250-->

---

<!-- How can you configure a system to optimize for **Fast Reads** while still maintaining Strong Consistency? :: Set $R=1$ and $W=N$. (Note: This makes write availability extremely fragile). -->
<!--SR:!2026-06-10,49,250-->
