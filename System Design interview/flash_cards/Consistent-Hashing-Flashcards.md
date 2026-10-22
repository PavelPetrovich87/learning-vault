<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/system-design -->

# 🎴 Consistent Hashing Flashcards (Senior Engineering)

<!-- What is the fundamental architectural flaw of "Simple Modulo Hashing" (`key % n`) for data partitioning? -->
<!-- ? -->
<!-- **Scalability Friction (The Rehash Problem).** When `n` (number of nodes) changes, nearly every key maps to a new location, causing catastrophic data migration or cache misses. -->
<!--SR:!2026-05-17,25,230-->

---

<!-- In Consistent Hashing, how does a key "find" its target node on the hash ring? -->
<!-- ? -->
<!-- The key is hashed onto the ring. We then traverse the ring **clockwise** from the key's position; the first server (or virtual node) encountered becomes the storage target. -->
<!--SR:!2026-06-17,55,250-->

---

<!-- In **Consistent Hashing**, if a system uses physical nodes on a hash ring without Virtual Nodes, what is the risk when a physical node fails? -->
<!-- ? -->
<!-- **Cascading Failure.** The entire load of the failed node falls onto exactly one neighbor. If that neighbor cannot handle the sudden spike, it may fail too, causing a domino effect. -->
<!--SR:!2026-04-26,3,210-->

---

<!-- In **Consistent Hashing**, how do ==Virtual Nodes (vnodes)== solve the problem of heterogeneous hardware in a cluster? -->
<!-- ? -->
<!-- We assign more virtual points on the ring to powerful servers (proportionally), allowing them to capture a larger segment of the hash space and thus store more data. -->
<!--SR:!2026-05-09,11,210-->

---

<!-- In **Consistent Hashing**, what percentage of keys typically require remapping when a node is added or removed from a cluster of $n$ nodes? :: Only $1/n$ of the total keys. -->
<!--SR:!2026-05-23,30,230-->

---

<!-- When using Consistent Hashing, a ==hotspot/skewed distribution== occurs when the hash function is poorly distributed or a single key (e.g., a "Celebrity" ID) is accessed disproportionately. -->
<!--SR:!2026-04-26,24,250-->
