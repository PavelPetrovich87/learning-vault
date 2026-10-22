---
tags: [flashcards, system-design, udemy, scalability, load-balancing, layer-4, layer-7, high-availability]
created: 2026-07-23
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Load Balancers — Flashcards

<!-- What is load balancing, and why is it important? -->
<!-- ? -->
<!-- Distributing incoming network traffic across multiple backend servers to ensure efficient utilization, prevent overload, and improve availability. Five reasons: **high availability** (redirects on failure), **resource utilization** (even spread), **performance** (routes to the best server), **scalability** (supports horizontal scaling), and **fault tolerance** (redirects if a server fails). -->
<!-- #flashcards/system-design -->

<!-- Explain the difference between Layer 4 and Layer 7 load balancing. -->
<!-- ? -->
<!-- **Layer 4** (transport): distributes traffic based on IP and port without inspecting content — faster but less flexible (AWS NLB, HAProxy L4). **Layer 7** (application): routes on HTTP headers, cookies, URL paths with SSL termination, caching, auth — intelligent but adds overhead (AWS ALB, Nginx, Traefik, Envoy). Key: L4 is faster/blunt; L7 is smart/slower. Many systems use both — L7 at the edge, L4 internally. -->
<!-- #flashcards/system-design -->

<!-- How does a load balancer handle high availability and failover? -->
<!-- ? -->
<!-- **Health Checks** (ping/HTTP/TCP) continuously monitor servers. **Automatic Failover** redirects traffic from unresponsive servers to healthy ones. **Redundancy** is achieved via active-active or active-passive LB pairs (so the LB itself isn't a SPOF). **Session Persistence** maintains user sessions across failover. **Global Load Balancing** uses GeoDNS or Anycast to distribute across data centers. -->
<!-- #flashcards/system-design -->

<!-- Compare Round Robin and Least Connections strategies. -->
<!-- ? -->
<!-- **Round Robin** sends requests in circular order — best for uniform workloads on equal-capacity servers, but can overload weaker nodes since it assumes everyone is identical. **Least Connections** routes to the server with the fewest active connections — best when requests vary in length (e.g., slow DB queries), but requires tracking connections (overhead). Key: Round Robin is simpler/assumes equal capacity; Least Connections adapts to load. -->
<!-- #flashcards/system-design -->

<!-- What are the advantages of Weighted Load Balancing? -->
<!-- ? -->
<!-- It assigns different priorities to servers based on capacity: better resource utilization (high-performance servers get more traffic), custom traffic distribution (fine-tuned routing control), and support for heterogeneous environments (servers with different processing power). Examples: Weighted Round Robin, Weighted Least Connections. -->
<!-- #flashcards/system-design -->

<!-- When would you use a software load balancer over a hardware one? -->
<!-- ? -->
<!-- **Software LBs** (Nginx, HAProxy, Envoy) run on standard hardware — cost-effective, flexible, easily scalable via containers/VMs, but consume server resources and can add latency under heavy load. **Hardware LBs** are dedicated appliances — high performance with hardware acceleration and built-in security (DDoS), but expensive, inflexible, and hard to scale. Use software for cloud-native apps; hardware for enterprise/high-traffic on-prem. -->
<!-- #flashcards/system-design -->

<!-- How would you design a scalable load balancing solution for a large e-commerce site? -->
<!-- ? -->
<!-- 1. **Multiple LBs** — primary/secondary for redundancy, plus DNS-based global distribution. 2. **Right LB type** — Layer 7 for dynamic content, Layer 4 for DB connections. 3. **Strategies** — Round Robin for static content, Least Connections for dynamic requests. 4. **High availability** — auto-scaling groups for spikes, health checks to bypass failures. 5. **Performance** — CDN caching, Gzip compression, minification to shrink responses. -->
<!-- #flashcards/system-design -->

<!-- What factors should be considered when choosing a load balancing strategy? -->
<!-- ? -->
<!-- 1. **Traffic pattern** — even → Round Robin; varying complexity → Least Connections. 2. **Server capacity** — heterogeneous → Weighted. 3. **Session persistence** — sticky sessions if needed. 4. **Performance vs complexity** — L4 faster/blunter; L7 slower/intelligent. 5. **Scalability needs** — cloud-native → cloud LBs (AWS ELB); on-prem → software or hardware. -->
<!-- #flashcards/system-design -->

<!-- How does a load balancer improve security? -->
<!-- ? -->
<!-- 1. **DDoS Protection** — detects/blocks malicious traffic spikes (hardware LBs often have built-in mitigation). 2. **SSL Termination** — offloads SSL decryption from backends, centralizing cert management. 3. **Access Control** — firewalls and IP whitelisting. 4. **WAF Integration** — prevents SQL injection, XSS, and other attacks. 5. **Rate Limiting** — caps requests per second to prevent abuse. -->
<!-- #flashcards/system-design -->

<!-- Why do many production systems run both Layer 4 and Layer 7 load balancers? -->
<!-- ? -->
<!-- Because they solve different problems. L7 at the edge handles HTTP routing, SSL termination, path-based routing to microservices, and WAF/security. L4 is used internally for service-to-service TCP traffic (e.g., to a database or between microservices) where raw throughput matters more than content inspection. Running both gives intelligence at the edge and efficiency in the core. -->
<!-- #flashcards/system-design -->

<!-- What's the danger of using Round Robin on a heterogeneous server fleet? -->
<!-- ? -->
<!-- Round Robin assumes all servers are equal, so it sends the same number of requests to a weak `c5.large` as to a beefy `c5.4xlarge`. The weaker server becomes a bottleneck and falls behind, while the stronger one sits underutilized. Fix: use Weighted Round Robin to give capacity-proportional shares. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-23* -->
