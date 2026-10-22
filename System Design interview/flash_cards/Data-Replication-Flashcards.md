<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/system-design -->

# 🎴 Data Replication Flashcards (Senior Reliability Focus)

<!-- A developer chooses the first 3 nodes on a consistent hashing ring for data replication. What critical check are they potentially missing? -->
<!-- ? -->
<!-- **Unique Physical Server Check.** On a ring with virtual nodes, multiple consecutive virtual points can map to the same physical machine. Replicating to them provides zero fault tolerance if that server fails. -->
<!--SR:!2026-05-06,13,210-->

---

<!-- Why is it essential to replicate data across different ==Data Centers (DCs)== in a high-availability system? -->
<!-- ? -->
<!-- To survive **Correlated Failures.** All servers within a single DC share the same power grid, cooling, and often the same backbone networking, making the entire facility a single point of failure. -->
<!--SR:!2026-05-02,30,270-->

---

<!-- What is the trade-off of using a higher value for ==N (number of replicas)==? :: Increased **Durability/Reliability** at the cost of higher **Storage/Bandwidth** costs and potentially higher **Write Latency** (if using synchronous writes). -->
<!--SR:!2026-04-25,23,250-->

---

<!-- A system replicates asynchronously to 3 nodes. A write is acknowledged by the leader, but the leader crashes before the replicas sync. What is the impact? -->
<!-- ? -->
<!-- **Data Loss.** This is a trade-off of asynchronous replication (Availability over Durability). To prevent this, one must use **Synchronous Replication** or wait for a Quorum before acknowledging the write. -->
<!--SR:!2026-04-25,23,250-->

---

<!-- Which mechanism is used to maintain synchronization and consistency across replicas in a distributed key-value store? :: ==Quorum Consensus==. -->
<!--SR:!2026-04-23,1,150-->

---

<!-- In the context of replication, what is an ==Anti-affinity rule==? -->
<!-- ? -->
<!-- A constraint that prevents replicas of the same data from being placed on the same physical host, rack, or availability zone, ensuring that a single infrastructure failure doesn't lose all copies of the data. -->
<!--SR:!2026-04-24,2,190-->
