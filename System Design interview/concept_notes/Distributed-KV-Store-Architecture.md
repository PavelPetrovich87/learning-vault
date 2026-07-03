---
status: analyzing
mastery: growing
tags: [system-design, distributed-systems, architecture, kv-store]
source: "System Design Interview – An Insider's Guide (Alex Xu)"
created: 2026-03-09
updated: 2026-07-03
difficulty: advanced
---

# 🏗️ Architecture: Distributed Key-Value Store

> [!abstract] Executive Summary (ELI5)
> Think of this system like a circular table (Hash Ring) where every person (Node) is exactly the same. There is no "boss." If you want to store a book, you give it to anyone at the table. That person (Coordinator) finds where it belongs on the ring, makes copies for their neighbors, and keeps an eye out to see if anyone else at the table leaves or joins.

---

## 🗺️ System Blueprint

The following diagram illustrates the decentralized nature of the store, the role of the coordinator, and the internal responsibilities of every node.

```mermaid
graph TB
    subgraph Client_Layer [Client Layer]
        C1[Client]
    end

    subgraph Hash_Ring [Consistent Hashing Ring]
        direction LR
        N1((Node 1)) --- N2((Node 2))
        N2 --- N3((Node 3))
        N3 --- N4((Node 4))
        N4 --- N5((Node 5))
        N5 --- N1
    end

    subgraph Node_Internals [Internal Node Architecture (All Nodes)]
        direction TB
        API[Client API: get/put]
        RE[Replication Engine]
        CR[Conflict Resolver: Vector Clocks]
        FD[Failure Detector: Gossip]
        FR[Recovery: Hinted Handoff / Merkle]
        ST[Local Storage Engine]
        
        API --> RE
        RE --> CR
        FD --> FR
        CR --> ST
    end

    %% Interactions
    C1 -- "put(k, v)" --> N1
    N1 -- "Coordinator Role" --> N2
    N1 -- "Coordinator Role" --> N3
    
    N1 -. "Gossip / Heartbeats" .-> N5
    N1 -. "Gossip / Heartbeats" .-> N2
    
    style N1 fill:#f96,stroke:#333,stroke-width:4px
    note left of N1: Coordinator for this request
```

---

## 🚀 Key Architectural Pillars

### 1. Simple Client API
The interface is intentionally minimal to reduce complexity:
- `get(key)`: Retrieves the value (or a list of conflicting "siblings") associated with the key.
- `put(key, value)`: Stores a value, triggering replication and versioning.

### 2. The Coordinator Node
- **No Master:** Any node can act as a coordinator. 
- **Responsibility:** When a node receives a request, it acts as a proxy, determines the correct storage nodes on the hash ring, and ensures the $W$ or $R$ quorum is met.

### 3. Complete Decentralization (Masterless)
- **Peer-to-Peer:** Every node runs the exact same software stack.
- **Dynamic Scaling:** Nodes can be added or removed without manual reconfiguration, thanks to **Consistent Hashing** and the **Gossip Protocol**.

### 4. Node Responsibilities (The "Full Stack" Node)
| Component | Function |
| :--- | :--- |
| **Client API** | Handles incoming traffic and enforces quorums. |
| **Replication** | Ensures data is copied to $N$ consecutive healthy nodes. |
| **Conflict Resolution** | Uses Vector Clocks to detect and merge divergent data versions. |
| **Failure Detection** | Uses Gossip to monitor the health of peers. |
| **Failure Recovery** | Uses Hinted Handoff (Temporary) and Merkle Trees (Permanent). |
| **Storage Engine** | Manages local disk I/O (often an LSM-tree for fast writes). |

---

## 🎯 The "Why" behind this Design
- **High Availability:** By replicating data and using "Sloppy Quorums," the system remains writable even during partial network partitions.
- **Scalability:** The decentralized nature allows the system to scale linearly by simply adding more nodes to the ring.
- **No Single Point of Failure (SPOF):** The loss of any single node (even a "coordinator") doesn't stop the system because any other node can step in.

## 🔗 Deep Connections
- [[Data-Partitioning-Consistent-Hashing]]
- [[Data-Replication-Distributed-Systems]]
- [[Quorum-Consensus-Consistency-Models]]
- [[Vector-Clocks-Versioning]]
- [[Failure-Handling-Distributed-Systems]]
