---
tags: [flashcards, system-design, networking, client-server-model]
created: 2026-06-27
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Client-Server Model — Flashcards

## Fundamental Questions

<!-- What is the client-server model, and how does it work? -->
<!-- ? -->
<!-- The client-server model is a distributed architecture where clients (requesters) initiate communication by sending requests to servers (providers), which process the requests and return responses. It separates presentation logic (client) from business logic and data (server). -->
<!-- #flashcards/system-design -->

<!-- How does a client communicate with a server? -->
<!-- ? -->
<!-- 1. The client opens a connection to the server's IP and port (e.g., TCP handshake). 2. The client sends a formatted request (e.g., HTTP GET /users). 3. The server parses, validates, and processes the request. 4. The server sends a response back. 5. The connection may be closed or kept alive for subsequent requests. -->
<!-- #flashcards/system-design -->

<!-- What are some real-world examples of the client-server model? -->
<!-- ? -->
<!-- Web browsing (browser + web server), email (Outlook + SMTP/IMAP server), file sharing (FTP client + FTP server), streaming (Netflix app + CDN), gaming (game client + game server), databases (SQL client + PostgreSQL), and chat (Slack + messaging server). -->
<!-- #flashcards/system-design -->

<!-- What is the difference between a client and a server? -->
<!-- ? -->
<!-- A client initiates requests, is usually resource-constrained, and runs user-facing applications. A server listens for requests on a well-known port, has abundant resources, runs 24/7, and manages shared resources like databases and files. -->
<!-- #flashcards/system-design -->

## Request-Response Cycle Questions

<!-- Explain the HTTP request-response cycle with an example. -->
<!-- ? -->
<!-- A browser (client) sends an HTTP GET request to `example.com` via TCP port 443. The web server receives it, routes it to the appropriate handler, queries a database if needed, and returns an HTTP 200 OK response with HTML/JSON. The browser then renders the response. -->
<!-- #flashcards/system-design -->

<!-- What are the key differences between synchronous and asynchronous communication? -->
<!-- ? -->
<!-- In synchronous communication, the client waits (blocks) until the server responds. In asynchronous communication, the client sends a request and continues processing, receiving the response later via a callback, promise, or message queue. Async is better for high-latency or high-throughput scenarios. -->
<!-- #flashcards/system-design -->

<!-- How does a browser load a webpage? -->
<!-- ? -->
<!-- 1. DNS resolution to get the server's IP. 2. TCP connection (and TLS handshake for HTTPS). 3. HTTP GET request for the HTML document. 4. Server responds with HTML. 5. Browser parses HTML and sends additional requests for CSS, JS, images, and fonts. 6. Browser renders the page as resources arrive. -->
<!-- #flashcards/system-design -->

## Architecture and Design Questions

<!-- What is the difference between stateless and stateful servers? -->
<!-- ? -->
<!-- Stateless servers don't store client session data between requests — each request is self-contained (e.g., REST API with JWT). Stateful servers maintain session state (e.g., WebSocket chat, shopping cart in memory). Stateless is easier to scale horizontally; stateful is needed for real-time interactions. -->
<!-- #flashcards/system-design -->

<!-- How does caching improve performance in a client-server model? -->
<!-- ? -->
<!-- Caching stores frequently accessed data closer to the client (browser cache, CDN, reverse proxy) so that repeated requests don't reach the origin server. This reduces latency, server load, and bandwidth usage. Common layers: browser cache, CDN, application cache (Redis), and database query cache. -->
<!-- #flashcards/system-design -->

<!-- How do load balancers work in a client-server architecture? -->
<!-- ? -->
<!-- A load balancer sits between clients and servers, distributing incoming requests across multiple backend servers. For stateless servers, any server can handle any request (round-robin). For stateful servers, sticky sessions or a shared session store (e.g., Redis) is needed. Health checks ensure traffic only goes to healthy servers. -->
<!-- #flashcards/system-design -->

## Advanced Questions

<!-- What are some security challenges in the client-server model? -->
<!-- ? -->
<!-- Man-in-the-middle attacks (mitigate with TLS), client-side code exposure (don't store secrets in client code), injection attacks (SQL, XSS, CSRF), session hijacking (use secure, httpOnly cookies and short-lived tokens), and DDoS attacks (rate limiting, WAFs, CDN absorption). -->
<!-- #flashcards/system-design -->

<!-- How does WebSockets differ from traditional request-response communication? -->
<!-- ? -->
<!-- Traditional HTTP request-response is half-duplex: the client sends a request and the server responds. WebSocket is full-duplex and persistent — after an initial handshake, both client and server can send messages at any time without polling. Ideal for real-time apps like chat, gaming, and live dashboards. -->
<!-- #flashcards/system-design -->

<!-- What are some limitations of the client-server model, and how can they be addressed? -->
<!-- ? -->
<!-- Limitations: single point of failure (mitigate with redundancy and failover), bottleneck under high load (scale horizontally with load balancers), network latency (use caching, CDNs, and persistent connections), and security risks (TLS, WAFs, rate limiting). The server is a central authority — if it fails, service degrades. -->
<!-- #flashcards/system-design -->

<!-- How would you design a scalable client-server system for a high-traffic application? -->
<!-- ? -->
<!-- Use stateless application servers behind a load balancer with auto-scaling. Cache at multiple layers: browser, CDN (CloudFront/Cloudflare), reverse proxy (Varnish/Nginx), and application cache (Redis). Use a database cluster with read replicas. Employ asynchronous processing (queues like SQS/Kafka) for heavy tasks. Store session data in a shared store (Redis) if stateful behavior is needed. Monitor health and scale based on metrics. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-06-27* -->
