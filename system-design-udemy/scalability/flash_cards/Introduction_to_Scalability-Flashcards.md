---
tags: [flashcards, system-design, udemy, scalability, horizontal-scaling, vertical-scaling, performance]
created: 2026-07-23
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Introduction to Scalability — Flashcards

<!-- What does scalability mean in the context of system design? -->
<!-- ? -->
<!-- The ability of a system to handle increased load — more users, more data, or more traffic — without compromising performance or reliability. A scalable system grows linearly or elastically with demand, typically via horizontal scaling (adding machines) or vertical scaling (upgrading hardware), while keeping responsiveness, throughput, and availability intact. -->
<!-- #flashcards/system-design -->

<!-- Walk me through a real-world example where scalability was critical to success or failure. -->
<!-- ? -->
<!-- Twitter is the classic failure-to-success story: its early monolith couldn't handle rapid growth, causing frequent "fail whale" outages, so it migrated to a distributed microservices architecture for horizontal scaling. Zoom is the success story — it scaled from 10M to 300M+ daily participants during COVID-19 using cloud-native infrastructure and autoscaling. -->
<!-- #flashcards/system-design -->

<!-- What are the main challenges systems face as they scale? -->
<!-- ? -->
<!-- Four dominate: **latency** (more components and network hops introduce delays), **bottlenecks** (one overloaded component degrades the whole system), **downtime** (more nodes and dependencies increase failure risk), and **cost** (cloud infrastructure gets expensive fast). Handling them requires proactive architecture, monitoring, and scaling policies. -->
<!-- #flashcards/system-design -->

<!-- How would you identify a bottleneck in a scalable architecture? -->
<!-- ? -->
<!-- 1. Monitor metrics with observability tools (Prometheus, Grafana, Datadog). 2. Watch for spikes in CPU, memory, or latency. 3. Trace requests end-to-end with OpenTelemetry or Jaeger. 4. Load-test with JMeter or k6 to find where degradation begins. 5. Look for services where queue lengths or response times grow disproportionately. -->
<!-- #flashcards/system-design -->

<!-- Why does latency increase with scale, and how would you mitigate it? -->
<!-- ? -->
<!-- As systems scale: more services mean more network calls, data is sharded/partitioned so aggregation is needed, and dependency chains grow longer. Mitigate by caching hot data (Redis), moving non-critical work to async queues, reducing network hops via CDNs/edge computing, and optimizing DB queries to cut cross-service chatter. -->
<!-- #flashcards/system-design -->

<!-- How do you balance scalability with cost in cloud-based systems? -->
<!-- ? -->
<!-- Use autoscaling with upper/lower bounds to avoid overprovisioning, choose serverless/FaaS (AWS Lambda) for spiky event-driven workloads, implement tiered caching to reduce DB load, leverage spot or reserved instances, and regularly right-size infrastructure based on real usage patterns. -->
<!-- #flashcards/system-design -->

<!-- What is the difference between vertical and horizontal scaling? -->
<!-- ? -->
<!-- Vertical scaling (scale up) adds more power to a single machine — CPU, RAM, faster storage. Horizontal scaling (scale out) adds more machines and distributes the workload across them, typically behind a load balancer. -->
<!-- #flashcards/system-design -->

<!-- Why is horizontal scaling usually preferred over vertical scaling at large scale? -->
<!-- ? -->
<!-- Vertical scaling hits hard physical and economic limits on a single machine and leaves a single point of failure. Horizontal scaling uses cheaper commodity hardware, offers near-unlimited growth, improves fault tolerance (other nodes survive a failure), and enables elastic auto-scaling. -->
<!-- #flashcards/system-design -->

<!-- What is the trade-off you accept when you move to horizontal scaling? -->
<!-- ? -->
<!-- You gain growth headroom and fault tolerance, but take on distribution complexity: you need a load balancer, must solve data consistency and session management across nodes, and add network hops/latency. More machines also means more to monitor, deploy, and secure. -->
<!-- #flashcards/system-design -->

<!-- A common interview trap: what's the difference between scalability and performance? -->
<!-- ? -->
<!-- Performance is about how fast a single request completes. Scalability is about how the system behaves as load grows. A fast system that dies at 100 users is not scalable; a slower system that smoothly handles 10M users is. Don't conflate them. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-23* -->
