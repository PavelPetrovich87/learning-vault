<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/system-design -->

# 🎴 CAP & PACELC Flashcards (Senior Interview Edition)

<!-- A client can reach Node A and Node B. Node A cannot reach Node B. According to CAP, is this a network partition? How should a CP system behave? -->
<!-- ? -->
<!-- **Yes, this is an asymmetrical network partition.** -->
<!-- In a CP system (like Zookeeper), nodes attempt to elect a leader. If they cannot reach a majority (Quorum), they must stop accepting writes to prevent data divergence. -->
<!--SR:!2026-04-30,7,210-->

---

<!-- "I have a single-instance Postgres DB. It's CA, right?" How do you answer this in a Senior interview? -->
<!-- ? -->
<!-- **No.** CAP only applies to **distributed systems**. -->
<!-- - A single-node DB is not "distributed." -->
<!-- - Once you add a read-replica: -->
<!--   - **Synchronous Replication** = CP-leaning (Consistency over Availability if the replica fails). -->
<!--   - **Asynchronous Replication** = AP-leaning. -->
<!--SR:!2026-04-25,2,190-->

---

<!-- Why would a system ever choose "Latency" (EL) over "Consistency" (EC) when there is **NO** network failure? -->
<!-- ? -->
<!-- **User Experience & Scalability.** -->
<!-- Example: **Amazon's Shopping Cart**. Waiting for 5 global replicas to confirm an item is in the cart (EC) adds significant lag. Amazon chooses **EL (Latency)** to show success immediately, accepting that the cart might be slightly inconsistent for a second to prioritize sales. -->
<!--SR:!2026-05-15,22,230-->

---

<!-- In a 5-node CP cluster, 2 nodes are partitioned away. The remaining 3 nodes are still up. Can they still provide Availability? -->
<!-- ? -->
<!-- **Yes, but only for the majority side.** -->
<!-- A CP system is available as long as a quorum (e.g., 3/5) exists. The 2 isolated nodes become unavailable. The system prioritizes consistency by disabling the minority side of the split. -->
<!--SR:!2026-06-13,51,250-->

---

<!-- You are building a **High-Frequency Trading** platform. Which PACELC category do you pick, and why? -->
<!-- ? -->
<!-- **PC/EC (Consistent/Consistent).** -->
<!-- - **PC:** During a network split, you must stop trading to prevent double-spending or ghost trades. -->
<!-- - **EC:** Normally, you cannot trade off consistency for speed. Even microsecond differences in price between nodes (arbitrage) could lead to massive losses. -->
<!--SR:!2026-04-24,22,250-->

---

<!-- Cassandra is often called an "AP" database. Is it possible to make it "CP"? -->
<!-- ? -->
<!-- **Mostly, via Quorum configurations.** -->
<!-- By setting `ConsistencyLevel.QUORUM` for both Reads and Writes (where $R + W > N$), you achieve strong consistency. If a partition makes a quorum unreachable, the request fails, prioritizing C over A. -->
<!--SR:!2026-05-07,9,190-->
