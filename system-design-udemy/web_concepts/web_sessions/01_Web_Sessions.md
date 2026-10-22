---
status: archived
tags: [system-design, udemy, web-concepts, sessions, research]
created: 2026-07-18
updated: 2026-10-21
---

# 01_Web_Sessions

## Source
Udemy System Design Course — Web Concepts / Sessions section.
Companion source: `[[../Web_Sessions_Important_Interview_Questions.pdf]]` (12 interview questions covering statelessness, session management, security, and scaling).

## Key Concepts
- **Why sessions exist:** HTTP is stateless — every request is independent — so web apps need an additional layer to remember who a user is across requests. See [[HTTP_Statelessness]].
- **Statelessness in HTTP:** Each request is treated as an independent transaction; the server keeps no memory of previous interactions. Simple to scale, but pushes state onto the application.
- **Techniques for maintaining state:** cookies, server-side session storage, JWTs, and browser `localStorage` / `sessionStorage`. See [[State_Management_Techniques]].
- **Session-based authentication:** server stores session data keyed by a random session ID delivered via cookie. See [[Session_Based_Authentication]].
- **Token-based authentication (JWT):** self-contained token carries the claims; verifiable without server-side storage. See [[Token_Based_Authentication]].
- **Security concerns:** session hijacking, CSRF, XSS, and the role of the `Secure`, `HttpOnly`, and `SameSite` cookie attributes. See [[Session_Security]].
- **Scaling session management:** sticky sessions vs distributed session stores (Redis, Memcached, DynamoDB) vs stateless JWT; trade-offs in load-balanced and microservice systems. See [[Scaling_Session_Management]].

## Concept Notes
- [[HTTP_Statelessness]]
- [[State_Management_Techniques]]
- [[Session_Based_Authentication]]
- [[Token_Based_Authentication]]
- [[Session_Security]]
- [[Scaling_Session_Management]]

## Flashcards
See flashcards: [[Web_Sessions-Flashcards]]

## Related Notes
- [[Client_Server_Model]]
- [[Load_Balancing_Introduction]]
- [[Microservices_Overview]]
- [[Microservices_Communication]]

---

*Last updated: 2026-07-18*
