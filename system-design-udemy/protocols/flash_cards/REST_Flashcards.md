---
status: researching
tags: [system-design, udemy, flashcards, rest, api]
created: 2026-07-02
updated: 2026-07-02
---

# REST and RESTfulness Flashcards

## Basic Questions

What is REST and how does it differ from SOAP?
?
REST is an architectural style for resource-based systems using HTTP, JSON/XML, and standard methods. SOAP is a strict protocol relying on XML envelopes, WSDL contracts, and often proprietary standards.
#flashcards/system-design

What are the six constraints of REST architecture?
?
Client-server, stateless, cacheable, uniform interface, layered system, and code on demand (optional).
#flashcards/system-design

What is the difference between a REST API and a RESTful API?
?
A REST API uses HTTP and resources loosely. A RESTful API strictly follows REST constraints: statelessness, uniform interface, resource-based URLs, and proper HTTP methods.
#flashcards/system-design

What is a resource in REST and how is it represented?
?
A resource is a domain entity (e.g., user, order) identified by a URI. It is represented by a format such as JSON or XML exchanged between client and server.
#flashcards/system-design

What are endpoints in a REST API?
?
Endpoints are URIs that identify resources or collections, e.g., `/users`, `/users/42`, `/users/42/orders`.
#flashcards/system-design

## HTTP Methods and Status Codes

Explain the difference between GET, POST, PUT, PATCH, and DELETE.
?
GET retrieves a resource; POST creates a new resource; PUT replaces a resource entirely; PATCH applies a partial update; DELETE removes a resource.
#flashcards/system-design

When would you use PUT vs PATCH?
?
Use PUT when replacing the whole resource with a complete payload; use PATCH when updating only a subset of fields.
#flashcards/system-design

What are commonly used HTTP status codes in REST APIs?
?
200 OK, 201 Created, 204 No Content, 301 Moved Permanently, 302 Found, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error, 503 Service Unavailable.
#flashcards/system-design

## RESTful API Design and Best Practices

What are the best practices for designing RESTful APIs?
?
Use nouns for resources, proper HTTP methods, plural names, consistent casing, versioning, stateless authentication, correct status codes, pagination, filtering, and caching headers.
#flashcards/system-design

How do you design a RESTful API for a blogging platform?
?
Resources: `/posts`, `/posts/:id`, `/posts/:id/comments`, `/users`, `/users/:id/posts`. Use GET/POST/PATCH/DELETE for CRUD and query params for filtering, sorting, and pagination.
#flashcards/system-design

What is HATEOAS in REST?
?
HATEOAS means responses include hypermedia links to related actions, allowing clients to navigate the API dynamically without hard-coding URLs.
#flashcards/system-design

How do you handle authentication and authorization in REST APIs?
?
Use API keys, JWTs, or OAuth 2.0 tokens passed in the `Authorization` header; verify identity and enforce permissions on each stateless request.
#flashcards/system-design

What are API rate limiting and throttling and why are they important?
?
Rate limiting caps the number of requests per client/time window; throttling slows or blocks excess traffic. They prevent abuse, ensure fairness, and protect backend stability.
#flashcards/system-design

How do you ensure backward compatibility when updating a REST API?
?
Version the API, add new fields without removing old ones, keep old endpoints working during deprecation, avoid changing existing semantics, and use optional query parameters.
#flashcards/system-design

## Advanced and Real-World Questions

How does caching work in RESTful APIs?
?
Responses can be cached by clients and proxies using headers like `Cache-Control`, `ETag`, `Last-Modified`, `If-None-Match`, and `If-Modified-Since` to avoid redundant requests.
#flashcards/system-design

What is the difference between JSON and XML in REST APIs?
?
JSON is lightweight, human-readable, and natively supported; XML is verbose, schema-driven, and supports namespaces but adds overhead. JSON is preferred for modern APIs.
#flashcards/system-design

How do you implement pagination in REST APIs?
?
Common methods: offset/limit (`?offset=0&limit=20`), page-based (`?page=2&size=20`), and cursor-based (`?cursor=...`) for large or dynamic datasets.
#flashcards/system-design

How does versioning work in REST APIs?
?
Versioning can be done via URL path (`/v1/users`), query parameter (`?version=1`), or header (`Accept-Version: v1`).
#flashcards/system-design

Explain the difference between REST, GraphQL, and gRPC.
?
REST is resource-based with multiple endpoints and fixed responses. GraphQL lets clients request exactly the fields they need via a single endpoint. gRPC is a high-performance RPC framework using Protocol Buffers and typically HTTP/2.
#flashcards/system-design

How would you improve the performance of a REST API?
?
Use caching, compression, pagination, efficient serialization, database indexing, connection pooling, rate limiting, and CDN for static assets; consider HTTP/2 or batching.
#flashcards/system-design

Can a REST API be stateful? Why and why not?
?
Technically an API can be stateful, but it is not RESTful. REST requires statelessness so that any server can handle any request, improving scalability, reliability, and caching.
#flashcards/system-design
