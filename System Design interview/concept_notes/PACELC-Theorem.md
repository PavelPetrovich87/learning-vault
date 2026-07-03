---
status: analyzing
mastery: growing
tags: [system-design, distributed-systems, theory]
source: "Daniel Abadi, 2010"
created: 2026-03-09
updated: 2026-07-03
difficulty: advanced
---

# 🧠 Topic: PACELC Theorem

> [!abstract] Executive Summary (ELI5)
> Imagine the notebook friends again. PACELC says: **P** (If there's a partition/split), you must choose between **A** (Availability) or **C** (Consistency). **E** (Else/Normally), you must choose between **L** (Latency - speed) or **C** (Consistency - accuracy). Essentially: "How do you act when things break, and how do you act when things are fine?"

---

## 🎯 The "Why"
- **Problem Solved:** Addresses a major limitation of the [[CAP Theorem]]—it only talks about what happens during a network partition. PACELC explains the tradeoffs even during normal operations (latency vs. consistency).
- **Alternatives:** [[CAP Theorem]] (subset of PACELC).
- **Trade-offs:** 
    - **PA/EL:** During a partition, stay available. Normally, prioritize latency (low latency over consistency). Example: **DynamoDB**, **Cassandra**.
    - **PC/EC:** During a partition, stay consistent. Normally, prioritize consistency (even if it's slower). Example: **VoltDB**, **HBase**.
    - **PC/EL:** During a partition, stay consistent. Normally, prioritize latency. Example: **MongoDB** (in some configurations).

## 💡 Mental Model & Analogy
*Connect this technical concept to a real-world object or process.*
- **Analogy:** Ordering a coffee at a busy cafe. 
    - **P (Partition):** The barista at the register can't talk to the barista at the machine. They can keep taking orders (AP) but might make mistakes, or stop taking orders (CP) until the connection is fixed.
    - **E (Else/Normal):** Everything is fine. Do they give you the coffee immediately (Latency) even if they're not sure it's the right one? Or do they double-check with the head barista (Consistency) making you wait longer?

## 🛠️ Core Principles & Code Snippets
*Focus on "The Right Way" (Best Practices).*

```typescript
// Configuring a PACELC-aware system like Cassandra
// (Writes can trade off Latency vs Consistency normally)

const cassandraWrite = async (query, params) => {
  // If we prioritize 'Latency' (EL):
  // We write to 1 node and return immediately.
  const consistencyLevel = 'ONE'; 
  
  // If we prioritize 'Consistency' (EC):
  // We wait for a majority of nodes to acknowledge.
  // const consistencyLevel = 'QUORUM'; 

  await client.execute(query, params, { prepare: true, consistency: consistencyLevel });
}
```

> [!tip] Implementation Note
> Most modern databases allow you to configure PACELC behaviors per-request (e.g., a "read-your-own-writes" requirement uses higher consistency, while a "background log" uses lower).

## ⚠️ Pitfalls & Anti-patterns
- **Common Mistakes:** Forgetting the "E" (Else). Many developers think they only have to worry about failures, but the Latency vs. Consistency trade-off affects 99.9% of normal traffic.
- **When NOT to use:** If you have a strictly single-node database, PACELC (like CAP) is overkill, though the "E" part still somewhat applies to memory vs. disk durability.

## 🔗 Deep Connections
- **Parent Topic:** [[Distributed Systems]]
- **Related Concepts:** [[Quorum]], [[Vector Clocks]], [[Anti-entropy]], [[Hinted Handoff]]
- **Prerequisites:** [[CAP Theorem]]

## ❓ Active Recall Questions
> [!faq]- What does the 'E' in PACELC stand for, and why is it important?
> It stands for "Else" (i.e., when no partition is present). It's important because it highlights that the consistency vs. latency trade-off is constant, not just during failures.

> [!faq]- Name a database that is PC/EC.
> **VoltDB** or **Megastore**. These systems choose consistency during partitions (PC) and prioritize consistency over latency during normal operations (EC).

> [!faq]- If a system is PA/EL, what is its primary goal?
> Maximum speed and uptime. It stays available during partitions (PA) and minimizes latency during normal operation (EL) by not waiting for all replicas to be consistent.
