---
tags: [flashcards, system-design, networking, api-gateway, microservices, security]
created: 2026-06-30
updated: 2026-06-30
---

# API Gateways — Flashcards

## Basic Questions

What is an API Gateway and why is it used?
?
An API gateway is a server that sits between clients and backend services. It provides a single entry point, routes requests, and centralizes cross-cutting concerns such as security, rate limiting, caching, logging, and request transformation.
#flashcards/system-design

How does an API gateway differ from a load balancer?
?
A load balancer mainly distributes traffic across backend servers. An API gateway does that and also handles authentication, rate limiting, caching, protocol translation, request/response transformation, and API management.
#flashcards/system-design

What are the key benefits of using an API Gateway?
?
Centralized security, authentication, rate limiting, load balancing, caching, request transformation, logging, monitoring, and hiding internal service topology from clients.
#flashcards/system-design

## Technical Questions

How does an API Gateway handle authentication and authorization?
?
It validates tokens, API keys, or session credentials at the edge, often using OAuth2, OpenID Connect, or LDAP. Unauthorized requests are rejected before they reach backend services.
#flashcards/system-design

Explain rate limiting and throttling in API gateways.
?
Rate limiting sets a maximum number of requests a client can make in a time window. Throttling enforces that limit by slowing down, queuing, or rejecting excess requests to protect backend services.
#flashcards/system-design

What caching strategies can be implemented in an API Gateway?
?
Cache read-heavy or static responses, use TTL-based expiration, vary cache keys by path/query/header, and invalidate or refresh cache when underlying data changes.
#flashcards/system-design

How does an API Gateway improve security against DDoS attacks?
?
It terminates TLS, applies rate limiting and connection limits, blocks malicious traffic patterns, integrates with WAFs, and prevents direct access to backend services.
#flashcards/system-design

## Scenario-Based Questions

When should you use an API gateway in a microservice architecture?
?
Use it when multiple services are exposed to clients and you need a single entry point for routing, authentication, rate limiting, caching, logging, and hiding internal service details.
#flashcards/system-design

How would you design an API Gateway for a large-scale system with millions of users?
?
Deploy gateways in multiple regions close to users, use autoscaling, terminate TLS at the edge, apply caching and rate limiting, integrate with identity providers, use health checks and failover, log and monitor traffic centrally, and keep business logic out of the gateway.
#flashcards/system-design

What challenges might arise when implementing an API gateway and how would you address them?
?
Single point of failure — deploy in clusters with failover. Latency — cache, use edge locations, and keep logic lightweight. Bottleneck — autoscale and use layered gateways. Over-aggregation — keep business logic in services. Security misconfiguration — centralize and audit policies.
#flashcards/system-design

---

*Last updated: 2026-06-30*
