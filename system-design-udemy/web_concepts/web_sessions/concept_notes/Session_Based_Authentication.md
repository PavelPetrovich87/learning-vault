---
status: archived
tags: [system-design, web-concepts, sessions, authentication]
source: Udemy System Design Course
created: 2026-07-18
difficulty: intermediate
updated: 2026-10-21
---

# 🧠 Topic: Session-Based Authentication

> [!abstract] Executive Summary (ELI5)
> When you log in, the server creates a private "file" for you (the session), puts your user info inside it, and hands you back just a **random ticket number** (the session ID) stored in a cookie. On every later request, the browser hands back the ticket, and the server looks up your file. The sensitive data never leaves the server — which is both the main strength (secure, revocable) and the main cost (you must store and look up sessions).

---

## 🎯 The "Why"
- **Problem Solved:** Authenticate a user once, then recognize them on subsequent requests without them re-sending their password.
- **Alternatives:** Token-based / JWT auth (see [[Token_Based_Authentication]]) keeps the credentials inside a client-held signed token instead of on the server.
- **Trade-offs:** Server-side sessions are more secure and trivially revocable, but require server-side storage and lookups — harder to scale across many nodes than stateless tokens.

## 💡 Mental Model & Analogy
- **Analogy:** "Session auth is a hotel. At check-in (login) the desk gives you a room key with a room number on it — the key itself knows nothing about you. Every time you return, you wave the key (cookie); the desk looks up the room (session) and knows it's you. Lose the key and the desk won't let you in; the front desk can also invalidate a key instantly if it's reported stolen."
- **Visual/Diagram:**
```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    participant Store as Session Store
    C->>S: POST /login (credentials)
    S->>Store: create session { userId, expires }
    Store-->>S: sid=7f3a9b
    S-->>C: 200 Set-Cookie: sid=7f3a9b; HttpOnly; Secure; SameSite=Lax
    Note over C: Browser stores cookie
    C->>S: GET /profile  Cookie: sid=7f3a9b
    S->>Store: lookup sid=7f3a9b
    Store-->>S: { userId: 42, ... }
    S-->>C: 200 Profile for Alice
```

## 🛠️ Core Principles

### The Full Login → Request Lifecycle
1. **Login:** The user submits credentials; the server verifies them (e.g. password hash check).
2. **Session creation:** The server creates a session record (user ID, roles, issued-at, expiry) in memory, a database, or a cache like Redis.
3. **Session ID delivery:** The server generates a **random, unguessable session ID** and sends it to the client via a `Set-Cookie` header.
4. **Subsequent requests:** The browser automatically attaches the cookie to matching requests. The server reads the ID, looks up the session, and re-derives the user context on every request.
5. **Logout / expiry:** The session is deleted server-side (or expires), so the cookie — even if still present client-side — points to nothing and authentication fails.

### The Two Halves Must Both Be Right
- **The cookie** is just a *pointer*. It carries only the session ID — never the session data.
- **The session store** holds the *actual data*. If the store is down, nobody can log in.
- Security lives in both: a stolen cookie is useless if the session was rotated/revoked; an unindexed session store makes every request slow.

### Server-Side Sessions vs Client-Side Tokens *(PDF Q3, first column)*

| Feature | Server-Side Sessions | Client-Side Tokens (JWT) |
|---|---|---|
| **Storage** | Stored on the server | Stored on the client |
| **Scalability** | Harder to scale (requires session replication or shared store) | Easier to scale (stateless authentication) |
| **Security** | More secure (session data never leaves the server) | Less secure (token can be stolen if not properly protected) |
| **Performance** | Extra overhead for session lookup on each request | Faster — no DB lookup, just signature verification |
| **Use case** | Best for traditional server-rendered web apps | Best for microservices and APIs |

### How Cookies and Sessions Cooperate *(PDF Q5)*
- On login, the server creates a session and stores session data (e.g. user ID) in memory or a database.
- The server generates a session ID and sends it to the client **via a cookie**.
- On subsequent requests, the client sends this cookie back, allowing the server to retrieve the session data.
- If the session is expired or deleted, authentication fails, requiring a new login.

The cookie is the *envelope*; the session ID is the *address*; the session store is the *mailbox*. Authentication works only when all three line up.

### Properties That Make Sessions Attractive
- **Revocability:** delete the server record → instant logout, even if the cookie was stolen.
- **No secret material on the client:** the cookie holds an opaque ID, not a signed credential.
- **Server-side control:** you can change roles, force re-auth, or kill all sessions at will.
- **Small cookie size:** only the ID travels (~20–40 bytes).

## ⚠️ Pitfalls & Anti-patterns
- **In-memory session store on a single server:** breaks under load balancing. Use a shared store (Redis/Memcached) or sticky sessions — see [[Scaling_Session_Management]].
- **Predictable or sequential session IDs:** attackers can guess them. Use a CSPRNG with ≥128 bits of entropy.
- **Missing cookie security flags:** without `HttpOnly`, `Secure`, `SameSite`, sessions are wide open to XSS, network sniffing, and CSRF — see [[Session_Security]].
- **No session expiry:** abandoned sessions accumulate forever (storage and security cost). Always set idle + absolute timeouts.
- **Not rotating the session ID after login:** exposes you to session-fixation attacks. Always issue a fresh ID on privilege change.

## 🔗 Deep Connections
- **Parent Topic:** [[01_Web_Sessions]]
- **Related Concepts:** [[HTTP_Statelessness]], [[State_Management_Techniques]], [[Token_Based_Authentication]], [[Session_Security]], [[Scaling_Session_Management]]
- **Prerequisites:** HTTP cookies, basic auth concepts.

## ❓ Active Recall Questions

> [!faq]- What are the differences between server-side sessions and client-side tokens? *(PDF Q3)*
> **Storage:** sessions on the server, tokens on the client. **Scalability:** sessions are harder to scale (need replication/shared store), tokens are stateless and scale easily. **Security:** sessions are more secure (data never leaves the server), tokens can be stolen if not protected. **Performance:** sessions have lookup overhead per request, tokens only need signature verification. **Use case:** sessions for traditional web apps, tokens for microservices/APIs.

> [!faq]- Walk through how cookies and sessions work together in session-based auth. *(PDF Q5)*
> 1) On login the server creates a session and stores session data (e.g. user ID) in memory or a DB. 2) It generates a session ID and sends it to the client via a cookie. 3) On subsequent requests the client sends the cookie back, letting the server retrieve the session data. 4) If the session is expired or deleted, auth fails and the user must log in again.

> [!faq]- What's the single biggest security advantage of session-based auth over JWT?
> Instant, cheap revocation. Because the session lives on the server, deleting it immediately invalidates the corresponding cookie. A JWT is valid until it expires (unless you build a server-side revocation list — which negates the statelessness).

> [!faq]- What breaks the moment you load-balance an in-memory session server?
> Subsequent requests may hit a node that doesn't have the session in memory → users get logged out randomly. Fixes: sticky sessions (route the user to the same node) or a shared session store (Redis/Memcached/DB). See [[Scaling_Session_Management]].

---

*Last updated: 2026-07-18*
