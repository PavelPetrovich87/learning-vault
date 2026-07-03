---
status: researching
tags: [system-design, udemy, flashcards, http, https]
created: 2026-07-02
updated: 2026-07-02
---

# HTTP Flashcards

## Basic Questions

What is HTTP and how does it work?
?
HTTP is a request-response protocol where a client sends a request (method, URL, headers, optional body) to a server, and the server returns a response (status code, headers, optional body).
#flashcards/system-design

Why is HTTP considered a stateless protocol?
?
Each HTTP request is independent; the server does not retain client context between requests. State is maintained using cookies, sessions, or tokens.
#flashcards/system-design

What are the key differences between HTTP and HTTPS?
?
HTTP runs on port 80 and sends plaintext; HTTPS runs on port 443 and encrypts data with TLS, providing encryption, authentication, and integrity.
#flashcards/system-design

## Technical and Deep-Dive

Explain the HTTP request-response cycle with an example.
?
A client opens a TCP connection to a server, sends a request such as `GET /index.html`, the server processes it and returns a response like `HTTP/1.1 200 OK` with the HTML body, and the connection is closed or reused.
#flashcards/system-design

What are HTTP methods?
?
HTTP methods define the action on a resource: GET (read), POST (create), PUT (replace), DELETE (remove), and PATCH (partial update).
#flashcards/system-design

HTTP Status Codes
?
1xx informational, 2xx success, 3xx redirection, 4xx client error, 5xx server error. Examples: 200 OK, 201 Created, 301 Moved Permanently, 302 Found, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error, 503 Service Unavailable.
#flashcards/system-design

How do cookies, sessions, and tokens help maintain state?
?
Cookies store small identifiers on the client; sessions store data on the server linked by a session ID; tokens (e.g., JWT) carry signed claims in each request, enabling stateless authentication.
#flashcards/system-design

What is the difference between a 301 and 302 redirection?
?
301 means the resource has moved permanently and clients should cache the new URL; 302 means the redirect is temporary and the original URL should still be used.
#flashcards/system-design

How does caching work in HTTP and which headers control it?
?
Caching stores responses to avoid repeated requests. Headers include `Cache-Control` (max-age, no-store, no-cache), `Expires`, `ETag`, `Last-Modified`, `If-None-Match`, and `If-Modified-Since`.
#flashcards/system-design

## Security and System Design

What security risks are associated with HTTP and how can they be mitigated?
?
HTTP sends plaintext, risking eavesdropping, MITM, and session hijacking. Mitigations include HTTPS/TLS, HSTS, secure and HttpOnly cookies, CSRF tokens, CORS policies, input validation, and proper authentication.
#flashcards/system-design
