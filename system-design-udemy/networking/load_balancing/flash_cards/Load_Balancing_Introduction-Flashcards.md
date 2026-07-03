---
tags: [flashcards, system-design, networking, load-balancing, scalability, availability]
created: 2026-06-30
updated: 2026-06-30
---

# Load Balancing Introduction — Flashcards

What problem does load balancing solve?
?
Load balancing solves the single-server bottleneck by distributing incoming traffic across multiple backend servers, improving performance, scalability, and fault tolerance.
#flashcards/system-design

Why does a single server eventually become a bottleneck?
?
A single server has finite CPU, memory, disk I/O, and network bandwidth. When traffic exceeds these limits, response times increase, errors rise, and the server becomes unreliable.
#flashcards/system-design

What is the difference between vertical scaling and horizontal scaling?
?
Vertical scaling adds more power to a single machine. Horizontal scaling adds more machines and distributes the workload across them.
#flashcards/system-design

Why is horizontal scaling often preferred over vertical scaling?
?
It is more cost-effective, scales farther using commodity hardware, improves fault tolerance, and avoids the physical and economic limits of a single powerful machine.
#flashcards/system-design

What challenge arises when an application is deployed across multiple servers?
?
You need a way to distribute traffic, share or route session state, detect failed servers, and keep data consistent across the cluster.
#flashcards/system-design

What role does a load balancer play in a distributed system?
?
A load balancer sits between clients and backend servers, routing requests, performing health checks, and hiding the backend topology from clients.
#flashcards/system-design

How does load balancing improve scalability?
?
Additional servers can be added behind the load balancer as traffic grows, and the balancer automatically distributes requests across the expanded pool.
#flashcards/system-design

How does load balancing improve availability?
?
The load balancer monitors server health and stops sending traffic to failed or unhealthy servers, so requests are handled by the remaining healthy nodes.
#flashcards/system-design

What happens if one of the servers fails in a load-balanced architecture?
?
The load balancer detects the failure through health checks, removes the failed server from the pool, and continues routing traffic to healthy servers.
#flashcards/system-design

Why is load balancing considered a foundational component of modern system design?
?
It enables horizontal scaling, high availability, fault tolerance, and elasticity, which are essential for building systems that handle large and variable internet traffic.
#flashcards/system-design

When should you introduce a load balancer into a system?
?
Introduce a load balancer when a single server can no longer handle the load, when you need high availability, or when you want to scale horizontally by adding more servers.
#flashcards/system-design

Walk me through the evolution from a single server application to a load-balanced architecture.
?
1. A single server runs the application and database. 2. The server is upgraded vertically until it hits limits. 3. The database is moved to a separate server. 4. Multiple application servers are deployed. 5. A load balancer is placed in front to distribute traffic. 6. Additional layers like caching, CDN, auto-scaling, and health checks are added for resilience and scale.
#flashcards/system-design

---

*Last updated: 2026-06-30*
