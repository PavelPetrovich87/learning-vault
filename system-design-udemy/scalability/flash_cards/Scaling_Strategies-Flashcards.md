---
tags: [flashcards, system-design, udemy, scalability, horizontal-scaling, vertical-scaling, diagonal-scaling, trade-offs]
created: 2026-07-23
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Scaling Strategies — Flashcards

<!-- What are the trade-offs between horizontal and vertical scaling in terms of performance and complexity? -->
<!-- ? -->
<!-- **Performance:** horizontal can scale almost linearly (with effort); vertical is limited by single-machine capacity. **Complexity:** horizontal needs distributed design; vertical is easier to manage initially. **Fault tolerance:** horizontal is resilient (failover); vertical is a single point of failure. **Cost:** horizontal has higher opex/complexity; vertical uses expensive high-capacity hardware. **Deployment:** horizontal needs orchestration + LB; vertical is simple with few moving parts. Summary: horizontal = scalable/resilient/complex; vertical = quick wins/limited long-term. -->
<!-- #flashcards/system-design -->

<!-- Can you describe a scenario where horizontal scaling wouldn't help? -->
<!-- ? -->
<!-- When the workload isn't parallelizable — e.g., a legacy monolith with shared state or global locks; when there's a non-distributable bottleneck like a single-threaded operation or a relational DB that doesn't shard/replicate reads; or a stateful service without sticky sessions/external session store. Real example: adding nodes to a PostgreSQL analytics DB does nothing unless you implement sharding or read replicas. -->
<!-- #flashcards/system-design -->

<!-- When would you choose vertical scaling over horizontal? -->
<!-- ? -->
<!-- When you need a quick fix without refactoring code, the system is monolithic/tightly coupled, the team lacks distributed-systems experience, you're in early startup stages, or concurrency needs are low and don't warrant added complexity. Ideal when you don't need high elasticity or can't justify the cost/complexity of a distributed architecture. -->
<!-- #flashcards/system-design -->

<!-- What challenges arise in horizontal scaling and how would you solve them? -->
<!-- ? -->
<!-- 1. **State management** → external stores like Redis/Memcached. 2. **Data consistency** → distributed transactions, eventual consistency, or CQRS. 3. **Load distribution** → smart load balancers (least-connections, IP hashing). 4. **Service discovery** → Consul, Eureka, or DNS-based discovery. 5. **Deployment & orchestration** → Docker + Kubernetes. 6. **Increased latency** → minimize inter-service calls, caching, optimize communication paths. -->
<!-- #flashcards/system-design -->

<!-- What is diagonal scaling and why do most production teams land there? -->
<!-- ? -->
<!-- Diagonal = scale up each node to a sensible size, then scale out by adding more of those upgraded nodes. It balances per-node performance with horizontal headroom, keeps node count manageable (fewer things to monitor than tiny commodity boxes), and pushes the architectural ceiling out. It's the pragmatic middle path most systems converge on. -->
<!-- #flashcards/system-design -->

<!-- What's the cost trade-off between vertical and horizontal scaling? -->
<!-- ? -->
<!-- Vertical uses disproportionately expensive high-end hardware (a 64-core box costs far more than eight 8-core boxes) but has low operational overhead. Horizontal uses cheaper commodity hardware but adds operational complexity — orchestration, monitoring, load balancing, and more nodes to manage. Horizontal wins on per-unit cost; vertical wins on simplicity. -->
<!-- #flashcards/system-design -->

<!-- Why is vertical scaling described as "a tactic, not a destination"? -->
<!-- ? -->
<!-- Vertical scaling buys you time with minimal effort, but a single box eventually maxes out physically (you can't buy an infinitely large server) and remains a single point of failure. The eventual migration to horizontal only gets harder over time, so plan for it rather than treating vertical as the long-term plan. -->
<!-- #flashcards/system-design -->

<!-- Walk me through the three scaling strategies with a real-world example of each. -->
<!-- ? -->
<!-- **Vertical:** upgrade an AWS RDS PostgreSQL primary from `db.r5.xlarge` to `db.r5.16xlarge` for more write throughput. **Horizontal:** Netflix runs thousands of EC2 instances behind load balancers with autoscaling groups reacting to demand. **Diagonal:** an AWS autoscaling group of reasonably-sized `c5.2xlarge` instances that scales from 2 → 50 nodes on demand. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-23* -->
