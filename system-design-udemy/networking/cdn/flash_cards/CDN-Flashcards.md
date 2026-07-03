---
tags: [flashcards, system-design, networking, cdn, caching, latency, availability]
created: 2026-06-30
updated: 2026-06-30
---

# CDN — Flashcards

## Basics of CDN

What is a CDN and how does it work?
?
A Content Delivery Network is a geographically distributed set of edge servers that cache content close to users. When a user requests content, DNS or anycast routes the request to the nearest edge server, which serves cached content or fetches it from the origin on a cache miss.
#flashcards/system-design

Why do we need CDNs in system design?
?
To reduce latency for geographically distributed users, improve availability, absorb traffic spikes, reduce origin server load, and add security at the edge.
#flashcards/system-design

What are the key benefits of using a CDN?
?
Lower latency, higher availability, better load handling, DDoS protection, SSL/TLS offloading, bandwidth savings, and global content distribution.
#flashcards/system-design

## Architecture and Components

Explain the difference between an origin server and an edge server in a CDN.
?
The origin server is the source of truth where original content is stored. Edge servers are distributed caches that store copies of content and serve it to nearby users.
#flashcards/system-design

What is a POP (Point of Presence) in a CDN?
?
A POP is a physical location where a CDN provider has deployed edge servers. Users are routed to the nearest POP to minimize latency.
#flashcards/system-design

How does request routing work in a CDN?
?
Request routing uses DNS resolution, anycast IP addresses, and real-time metrics such as geography, latency, server load, and availability to direct the user to the best edge server.
#flashcards/system-design

## Caching Strategies and Content Delivery

What is a cache hit versus a cache miss, and how does a CDN handle them?
?
A cache hit means the edge server has the content and serves it immediately. A cache miss means the edge server fetches the content from the origin, caches it, and then serves it.
#flashcards/system-design

Explain cache expiration and TTL in a CDN.
?
TTL (Time-to-Live) defines how long a cached item is considered fresh. After the TTL expires, the edge server must revalidate or refetch the content from the origin.
#flashcards/system-design

What are cache validation strategies and why are they important?
?
Validation strategies like Last-Modified/If-Modified-Since and ETag/If-None-Match let edge servers check whether cached content is still current without downloading the full resource. They save bandwidth and reduce origin load.
#flashcards/system-design

How does a CDN handle dynamic content caching?
?
It uses short TTLs, stale-while-revalidate, cache-key segmentation, and edge logic to cache dynamic responses safely without serving stale or user-specific data.
#flashcards/system-design

## Load Balancing and Failover Handling

How do CDNs use load balancing to improve reliability?
?
CDNs distribute requests across many edge servers and POPs, perform health checks, and route users away from failed or overloaded locations.
#flashcards/system-design

Explain different request routing strategies: geo-based, latency-based, load-aware.
?
Geo-based routing sends users to the closest POP by geography. Latency-based routing selects the POP with the lowest measured round-trip time. Load-aware routing avoids POPs that are currently congested.
#flashcards/system-design

What happens if a CDN POP fails? How does failover handling work?
?
The routing system detects the failure through health checks and redirects user requests to the next best POP. Cached content at other locations continues to be served.
#flashcards/system-design

## CDN Optimization Techniques

What compression and minification techniques do CDNs use?
?
Gzip and Brotli compression for text assets; minification of CSS, JavaScript, and HTML to remove unnecessary characters and reduce file size.
#flashcards/system-design

How does a CDN optimize image and video delivery?
?
It serves responsive images in optimal formats like WebP or AVIF, resizes images for the device, and uses adaptive bitrate streaming such as HLS or DASH for video.
#flashcards/system-design

How does API acceleration work in a CDN?
?
API acceleration caches API responses at the edge, terminates TLS closer to users, optimizes TCP connections, keeps persistent connections to origins, and reduces round trips.
#flashcards/system-design

## CDN Security and Challenges

How does a CDN protect against DDoS attacks?
?
It absorbs traffic across a globally distributed network, applies rate limiting and anomaly detection, challenges suspicious requests, and hides the origin server's IP address.
#flashcards/system-design

What is SSL/TLS offloading and why is it useful?
?
SSL/TLS offloading means the CDN handles encryption and decryption at the edge instead of the origin. It reduces CPU load on origin servers and centralizes certificate management.
#flashcards/system-design

What are some challenges in CDN implementation and how can they be mitigated?
?
Cache invalidation complexity — use versioned URLs and clear invalidation policies. Stale content — set appropriate TTLs and use validation. Origin exposure — restrict origin access to CDN IPs. Cost — monitor cache hit ratios and optimize cache rules.
#flashcards/system-design

## Advanced CDN Topics

How does a multi-CDN architecture work?
?
A multi-CDN setup uses two or more CDN providers. A traffic manager routes requests to the best-performing or healthiest CDN at any moment, improving resilience and avoiding vendor lock-in.
#flashcards/system-design

What is edge computing and how does it relate to CDNs?
?
Edge computing runs code or functions on CDN edge servers close to users. CDNs provide the global infrastructure that makes edge computing possible, enabling low-latency processing and personalization.
#flashcards/system-design

How would you design a CDN for a large-scale video streaming platform?
?
Use many POPs close to users, cache video segments at multiple bitrates, support adaptive bitrate streaming (HLS/DASH), use origin shield to reduce origin load, apply TCP and TLS optimizations, and use analytics to tune cache policies.
#flashcards/system-design

How do CDNs help in real-time applications like online gaming and stock trading?
?
They place game servers, matchmaking, and market data endpoints closer to users, reducing latency, jitter, and packet loss for time-sensitive interactions.
#flashcards/system-design

---

*Last updated: 2026-06-30*
