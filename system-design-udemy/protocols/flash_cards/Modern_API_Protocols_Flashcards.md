---
status: researching
tags: [system-design, udemy, flashcards, grpc, graphql, rest, api]
created: 2026-07-02
updated: 2026-07-02
---

# Modern API Protocols Flashcards

## Comparisons and Trade-Offs

How would you compare REST, gRPC and GraphQL?
?
REST is resource-based, simple, and widely supported. gRPC is a high-performance binary RPC framework over HTTP/2 with streaming and protobuf. GraphQL is a query language with a single endpoint that lets clients request exactly the fields they need.
#flashcards/system-design

When would you use gRPC over REST?
?
Use gRPC for low-latency, high-throughput internal microservices, real-time streaming, polyglot systems, and IoT where binary efficiency and strong typing matter.
#flashcards/system-design

What are the trade-offs of using GraphQL in large-scale systems?
?
GraphQL adds caching complexity, N+1 query risk, rate-limiting challenges, schema governance overhead, and server complexity from resolvers and federation.
#flashcards/system-design

## Security and Scaling

How does gRPC handle authentication and security?
?
gRPC uses TLS for transport encryption, token-based auth via metadata (e.g., JWT), mutual TLS for client-server authentication, and interceptors for validation and enforcement.
#flashcards/system-design

How do you scale GraphQL APIs efficiently?
?
Use query complexity analysis, depth limits, persisted queries, DataLoader batching, response caching, schema federation, and pub/sub-backed subscriptions.
#flashcards/system-design

