<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/system-design -->

# 🎴 Failure Handling & Anti-Entropy Flashcards

<!-- How does the ==Gossip Protocol== avoid the "all-to-all" $O(N^2)$ messaging bottleneck in failure detection? -->
<!-- ? -->
<!-- Each node periodically sends its heartbeat counter only to a small, **random** subset of nodes. Those nodes then propagate the info to others, spreading the "rumor" of health or failure across the cluster with low overhead. -->
<!--SR:!2026-05-19,27,230-->

---

<!-- Define ==Sloppy Quorum==. -->
<!-- ? -->
<!-- A mechanism that allows a system to satisfy its $W$ (write) or $R$ (read) quorum by using the first $W/R$ **healthy** nodes on the hash ring, even if they aren't the primary assigned replicas. This prioritizes availability over strict consistency during node outages. -->
<!--SR:!2026-04-29,6,210-->

---

<!-- What is the role of ==Hinted Handoff== in distributed systems? -->
<!-- ? -->
<!-- When a node is temporarily down, its write operations are diverted to a "proxy" node. This proxy stores a "hint" (metadata) and re-plays those writes back to the original node once it recovers, ensuring eventual consistency. -->
<!--SR:!2026-04-26,24,250-->

---

<!-- A system uses ==Merkle Trees== to sync two replicas. Both nodes compare their root hashes and find a mismatch. What is the next step in the algorithm? -->
<!-- ? -->
<!-- They compare the hashes of the root's children. If the left child's hash matches but the right child's doesn't, they ignore the left subtree and recursively descend only into the right subtree to find the specific data divergence. -->
<!--SR:!2026-06-06,44,250-->

---

<!-- What is ==Anti-entropy== in distributed systems? -->
<!-- ? -->
<!-- A process of comparing replicas and synchronizing any differences to ensure they reach a consistent state (e.g., using Merkle Trees for full-sync during permanent failures). -->
<!--SR:!2026-05-01,8,190-->

---

<!-- In failure detection, why is a single server's signal usually insufficient to mark another server as "down"? :: To avoid **False Positives** due to localized network congestion or server-specific issues. Most systems require confirmation from multiple independent nodes. -->
<!--SR:!2026-05-13,13,210-->

---

<!-- What is the primary trade-off of using ==Sloppy Quorum==? :: Improved **Availability** at the cost of potential **Stale Reads**, as the "temporary" nodes may not have the absolute latest data until a handoff or anti-entropy sync occurs. -->
<!--SR:!2026-04-25,2,190-->
