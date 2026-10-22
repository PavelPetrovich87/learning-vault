---
tags: [flashcards, system-design, web-concepts, sessions]
created: 2026-07-18
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Web Sessions — Flashcards

<!-- > One card per interview question (Q1–Q12) from `Web_Sessions_Important_Interview_Questions.pdf`, plus synthesis cards on the headline trade-offs. Source notes: [[HTTP_Statelessness]], [[State_Management_Techniques]], [[Session_Based_Authentication]], [[Token_Based_Authentication]], [[Session_Security]], [[Scaling_Session_Management]]. -->

<!-- Why is HTTP considered a stateless protocol? -->
<!-- ? -->
<!-- Each client request is treated as an independent transaction — the server retains no memory of previous interactions. This simplifies server design and makes horizontal scaling trivial, but forces the application to layer on cookies, sessions, or tokens to maintain any user state. -->
<!-- #flashcards/system-design -->

<!-- How do web applications maintain state despite HTTP being stateless? -->
<!-- ? -->
<!-- Four layered techniques: (1) cookies — small browser key/value pairs, often holding a session ID; (2) server-side session storage — the server stores session data linked to a session ID stored in the client's cookie; (3) JWT — a self-contained, signed token carrying claims, verifiable without server-side storage; (4) localStorage / sessionStorage — browser-side persistence for client-only data. -->
<!-- #flashcards/system-design -->

<!-- How would you compare server-side sessions vs client-side tokens as an auth strategy? -->
<!-- ? -->
<!-- Storage: sessions on the server, tokens on the client. Scalability: sessions harder (need shared/replicated store), tokens easier (stateless). Security: sessions more secure (data never leaves server), tokens can be stolen if not protected. Performance: sessions have per-request lookup overhead; tokens need only signature verification. Use case: sessions for traditional web apps, tokens for microservices and APIs. -->
<!-- #flashcards/system-design -->

<!-- When would you choose JWT-based auth over session-based auth — and when would you avoid it? -->
<!-- ? -->
<!-- Choose JWT when you need stateless auth for scalability (microservices, public APIs), want to avoid server-side session storage, or need a token reusable across services (OAuth, SSO). Avoid JWT when you need easy revocation — once issued it's valid until expiry unless you maintain a server-side blocklist, which negates the statelessness. -->
<!-- #flashcards/system-design -->

<!-- Walk through how cookies and sessions cooperate in session-based authentication. -->
<!-- ? -->
<!-- 1) On login the server creates a session and stores the session data (e.g. user ID) in memory or a DB. 2) It generates a session ID and sends it to the client via a Set-Cookie header. 3) On subsequent requests the browser sends the cookie back, letting the server retrieve the session data. 4) If the session is expired or deleted, auth fails and the user must log in again. -->
<!-- #flashcards/system-design -->

<!-- What is session hijacking and how would you prevent it? -->
<!-- ? -->
<!-- Session hijacking is when an attacker steals a user's session ID and impersonates them. Prevent with: Secure/HttpOnly/SameSite cookie attributes; SSL/TLS to encrypt the wire; rotating the session ID after login or privilege escalation; short session expiration with re-authentication; and anomaly detection by loosely binding sessions to client properties. -->
<!-- #flashcards/system-design -->

<!-- How does CSRF exploit session management, and how do you mitigate it? -->
<!-- ? -->
<!-- CSRF tricks an authenticated user's browser into sending an unintended request to a trusted site — the browser auto-attaches the session cookie, so the forged request looks legitimate. Mitigate with anti-CSRF tokens validated per request, SameSite=Lax/Strict cookies, re-authentication for sensitive operations, and strict CORS policies. -->
<!-- #flashcards/system-design -->

<!-- Why must cookies be set with Secure, HttpOnly, and SameSite attributes? -->
<!-- ? -->
<!-- Secure ensures the cookie is only sent over HTTPS (blocks sniffing). HttpOnly hides the cookie from JavaScript (blocks XSS-driven theft). SameSite restricts cross-site sends (blocks CSRF). Each attribute covers a different attack vector — together they form layered defense. -->
<!-- #flashcards/system-design -->

<!-- How would you scale session management in a distributed system? -->
<!-- ? -->
<!-- Three options: (1) Sticky sessions — load balancer pins each user to one node. (2) Distributed session store — Redis, Memcached, or a DB shared by all nodes. (3) Stateless JWT auth — remove server-side sessions entirely. Most production systems use option 2 or 3, reserving sticky sessions for specific cases like WebSocket connections. -->
<!-- #flashcards/system-design -->

<!-- A user keeps getting logged out after you added a second server behind a load balancer. What's happening and how do you fix it? -->
<!-- ? -->
<!-- The session lives in one node's memory, so any request landing on the other node looks unauthenticated. Fix with: sticky sessions to keep the user on the original node; a distributed session store (Redis/DynamoDB) shared by all nodes; or JWT-based stateless auth, which removes the per-node dependency entirely. -->
<!-- #flashcards/system-design -->

<!-- In a microservices architecture, how do you manage authentication across multiple services? -->
<!-- ? -->
<!-- Use a centralized auth service (OAuth 2.0 / OpenID Connect) that issues JWTs valid across services. Each service validates the token locally without calling the auth server on every request, and refresh tokens enable long-term auth without frequent re-login. An API gateway can verify once and propagate identity headers internally. -->
<!-- #flashcards/system-design -->

<!-- How do platforms like Facebook, Amazon, or Google handle user sessions at scale? -->
<!-- ? -->
<!-- They combine: token-based auth (JWT, OAuth) for cross-service scalability; distributed stores (Redis, Cassandra, DynamoDB) for any server-side state; single sign-on (SSO) for seamless auth across products; and short-lived access tokens paired with refresh tokens so a leaked token has limited blast radius. Regional edge session services keep latency low. -->
<!-- #flashcards/system-design -->

<!-- What goes wrong if you put JWTs in localStorage and your site has a stored XSS bug? -->
<!-- ? -->
<!-- The XSS payload can read localStorage and exfiltrate the token, fully impersonating the user until it expires. Mitigate by preferring an HttpOnly cookie (invisible to JS) for sensitive tokens, plus CSP and strict output escaping to prevent the XSS in the first place. -->
<!-- #flashcards/system-design -->

<!-- What's the difference between CSRF and XSS in one sentence? -->
<!-- ? -->
<!-- CSRF exploits the browser's automatic cookie-sending to forge a request the user never intended (attacker never sees the cookie); XSS injects attacker-controlled JavaScript into your page that *can* read non-HttpOnly cookies and forge requests inside the user's session. -->
<!-- #flashcards/system-design -->

<!-- Why pair a short-lived access token with a longer-lived refresh token? -->
<!-- ? -->
<!-- Short access tokens (5–15 min) cap the damage if one is stolen; the refresh token is used only to mint new access tokens, is stored more carefully (HttpOnly cookie), can be rotated, and gives you a revocation hook (revoke the refresh) without blocklisting every access token. -->
<!-- #flashcards/system-design -->

<!-- You need to revoke a user's access right now, but you're using JWTs. What are your options? -->
<!-- ? -->
<!-- Because JWTs are stateless, you can't trivially revoke before expiry. Options: keep access tokens very short so the window is small; maintain a server-side revocation list (blocklist) keyed by token ID (cost: reintroduces state); revoke the refresh token so no new access tokens can be minted; or rotate the signing key to invalidate a whole class of tokens at once. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-18* -->
