---
tags: [flashcards, system-design, networking, proxy, forward-proxy, reverse-proxy]
created: 2026-06-30
updated: 2026-06-30
---

# Forward and Reverse Proxy — Flashcards

## Basic Concepts

What is a proxy server, and why is it used?
?
A proxy server is an intermediary that receives requests from one party and forwards them to another. It is used for anonymity, security filtering, caching, access control, logging, traffic optimization, and policy enforcement.
#flashcards/system-design

Explain the key difference between a forward proxy and a reverse proxy.
?
A forward proxy sits in front of clients and makes requests to external servers on their behalf. A reverse proxy sits in front of backend servers and receives incoming client requests on their behalf. The forward proxy protects the client; the reverse proxy protects the server.
#flashcards/system-design

## Technical Deep Dive

How does a forward proxy improve security and privacy?
?
It hides the client's real IP address from destinations, filters outbound traffic to block malicious or unwanted sites, inspects traffic for threats, and logs activity for compliance and auditing.
#flashcards/system-design

How does a reverse proxy help in load balancing and caching?
?
It distributes incoming requests across multiple backend servers using algorithms like round-robin or least connections, and it caches responses — especially static content — so repeat requests are served without hitting the origin server.
#flashcards/system-design

What are some real-world examples of forward and reverse proxies?
?
Forward proxies: Squid, Zscaler, corporate web proxies, SOCKS5 proxies, browser VPN extensions. Reverse proxies: Nginx, HAProxy, Cloudflare, AWS Application Load Balancer, Kubernetes Ingress controllers.
#flashcards/system-design

## Implementation and Use Cases

When should you use a forward proxy vs a reverse proxy in system architecture?
?
Use a forward proxy when you need to control, filter, or anonymize outbound client traffic. Use a reverse proxy when you need to load balance, cache, terminate SSL, hide backend topology, or protect servers from inbound attacks.
#flashcards/system-design

What are some common tools and technologies used for each type of proxy?
?
Forward proxy: Squid, Zscaler, Blue Coat, SOCKS5, Privoxy, corporate proxies. Reverse proxy: Nginx, HAProxy, Apache HTTP Server, Traefik, Caddy, Cloudflare, AWS ALB/ELB, Envoy.
#flashcards/system-design

How does a reverse proxy protect backend servers from DDoS attacks?
?
It absorbs traffic at the edge, applies rate limiting and connection limits, challenges suspicious clients with CAPTCHA or JavaScript challenges, filters malicious requests, and prevents attackers from directly reaching backend servers.
#flashcards/system-design

## Advanced Topics

How does SSL termination work in a reverse proxy?
?
The reverse proxy handles the TLS handshake and decrypts HTTPS traffic from clients. It then forwards the request as plain HTTP (or re-encrypted HTTPS) to backend servers, offloading cryptographic work and centralizing certificate management.
#flashcards/system-design

What are the advantages of using Cloudflare, Nginx, or HAProxy as a reverse proxy?
?
Cloudflare provides global CDN, DDoS protection, WAF, and DNS in one service. Nginx is lightweight, widely used, and excellent for static serving and routing. HAProxy is known for high performance, advanced load balancing, and detailed health checks.
#flashcards/system-design

How does a transparent proxy differ from a forward or reverse proxy?
?
A transparent proxy intercepts traffic without requiring client-side configuration or awareness. It can be placed on client or server networks for filtering or caching, whereas a forward proxy is explicitly configured by the client and a reverse proxy is explicitly placed in front of servers.
#flashcards/system-design

---

*Last updated: 2026-06-30*
