---
status: researching
tags: [system-design, udemy, flashcards, real-time, websockets, long-polling]
created: 2026-07-02
updated: 2026-07-02
---

# Real-time Communication Protocols Flashcards

## Basic Questions

What is real-time communication and why is it important?
?
Real-time communication is the near-instantaneous exchange of data between clients and servers. It is important for chat, online gaming, stock trading, IoT telemetry, live notifications, and collaborative editing.
#flashcards/system-design

How do WebSockets work and how do they differ from traditional HTTP?
?
WebSockets start as an HTTP upgrade request and then become a persistent, full-duplex TCP connection. Unlike HTTP, they allow both client and server to send messages at any time with low latency and no repeated headers.
#flashcards/system-design

Explain the WebSocket handshake process.
?
The client sends an HTTP GET request with `Upgrade: websocket`, `Connection: Upgrade`, and `Sec-WebSocket-Key`. The server responds with `101 Switching Protocols` and a `Sec-WebSocket-Accept` header derived from the key. After that, data is exchanged as WebSocket frames.
#flashcards/system-design

What is long polling and how does it work?
?
Long polling is a technique where the client sends a request and the server holds it open until it has data to send. Once the server responds, the client immediately sends another request to wait for the next event.
#flashcards/system-design

## Intermediate Questions

What are the advantages of WebSockets over long polling?
?
WebSockets offer lower latency, full-duplex bidirectional messaging, no repeated headers, fewer reconnections, and better support for high-frequency updates.
#flashcards/system-design

In what scenarios would you prefer long polling over WebSockets?
?
Prefer long polling when WebSockets are blocked by firewalls/proxies, infrastructure is limited, or when you need a quick fallback solution for low-frequency updates.
#flashcards/system-design

How do WebSockets handle connection failures or network interruptions?
?
Applications implement reconnection logic with exponential backoff, heartbeat/ping-pong frames to detect dead connections, and replay buffers to recover missed messages after reconnection.
#flashcards/system-design

Can you use WebSockets with a load balancer? If yes, how?
?
Yes, but you need sticky sessions (session affinity) so the client reconnects to the same server, or a shared pub/sub bus so any server can push messages to the correct socket.
#flashcards/system-design

What are some challenges of scaling WebSockets in a distributed system?
?
Challenges include connection affinity, broadcasting across servers, process-bound socket state, memory/file descriptor limits, backpressure from slow clients, and handling reconnections.
#flashcards/system-design
