---
tags: [flashcards, system-design, networking, ip-addresses]
created: 2026-06-27
updated: 2026-06-27
---

# IP Addresses — Flashcards

What is the primary difference between IPv4 and IPv6 in terms of address space?
?
IPv4 uses 32-bit addresses (~4.3 billion total), while IPv6 uses 128-bit addresses (~3.4 × 10³⁸), effectively eliminating address exhaustion.
#flashcards/system-design

Why do we use private IP addresses instead of assigning every device a public IP?
?
Private IPs conserve the limited IPv4 address space and provide a security boundary by keeping internal devices non-routable from the public internet.
#flashcards/system-design

How does NAT help address the IPv4 shortage?
?
NAT allows many devices with private IP addresses to share a single public IP by translating the source address on outbound traffic and mapping return traffic back to the correct internal device.
#flashcards/system-design

How does a load balancer distribute traffic using IP addresses?
?
A load balancer exposes a single Virtual IP (VIP) to clients, then distributes incoming connections across multiple backend servers based on their individual IP addresses using algorithms like round-robin, least-connections, or IP-hash.
#flashcards/system-design

How does DNS resolve IP addresses in a large-scale system?
?
DNS uses hierarchical caching (browser → OS → local resolver → root → TLD → authoritative), plus techniques like anycast, geo-DNS, and load-balanced DNS servers to resolve domain names to IP addresses efficiently and reliably at scale.
#flashcards/system-design

---

*Last updated: 2026-06-27*
