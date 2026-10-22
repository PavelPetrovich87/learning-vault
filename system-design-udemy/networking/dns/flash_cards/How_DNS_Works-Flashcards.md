---
tags: [flashcards, system-design, networking, dns]
created: 2026-06-27
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# How DNS Works — Flashcards

<!-- Explain the DNS resolution process step by step. -->
<!-- ? -->
<!-- 1. Browser checks cache. 2. OS checks cache. 3. Query sent to recursive resolver. 4. Resolver checks cache. 5. If miss, query root server for TLD. 6. Query TLD for authoritative server. 7. Query authoritative server for the IP. 8. Resolver caches and returns IP to client. 9. Client connects to the IP. -->
<!-- #flashcards/system-design -->

<!-- What is the difference between an authoritative DNS server and a recursive DNS server? -->
<!-- ? -->
<!-- A recursive resolver acts on behalf of the client, queries other servers iteratively, and caches results. An authoritative server holds the actual DNS records for a domain and is the source of truth for that domain's data. -->
<!-- #flashcards/system-design -->

<!-- How does DNS caching improve performance, and where does it occur? -->
<!-- ? -->
<!-- DNS caching reduces latency and load by storing resolved records at multiple layers: browser cache, OS cache, local/recursive resolver cache, and intermediate forwarding resolvers. Up to 80-90% of queries are served from cache without reaching authoritative servers. -->
<!-- #flashcards/system-design -->

<!-- What is TTL in DNS, and why is it important? -->
<!-- ? -->
<!-- TTL (Time To Live) is a value in seconds that tells resolvers how long they can cache a DNS record. It balances performance (high TTL = less querying) against agility (low TTL = faster propagation of changes during failover or migration). -->
<!-- #flashcards/system-design -->

<!-- How does DNS-based load balancing work in large-scale systems? -->
<!-- ? -->
<!-- DNS can return multiple A records (round-robin), use GeoDNS to route users to the nearest datacenter based on location, or use weighted records to control traffic distribution. Anycast DNS advertises the same IP from multiple locations, routing users to the nearest healthy node via BGP. -->
<!-- #flashcards/system-design -->

<!-- What are common DNS-related security threats, and how can they be mitigated? -->
<!-- ? -->
<!-- DNS spoofing/poisoning (false records injected), cache poisoning (corrupting resolver caches), and DDoS attacks (overwhelming DNS infrastructure). Mitigation: DNSSEC for cryptographic validation, DNS over HTTPS/TLS for encrypted queries, and Anycast routing to distribute and absorb DDoS traffic. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-06-27* -->
