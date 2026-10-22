---
status: archived
tags: [system-design, udemy, web-concepts, cors, research]
created: 2026-07-18
updated: 2026-10-21
---

# 01_CORS

## Source
Udemy System Design Course — Web Concepts / CORS section.
Companion source: `[[../CORS+&+Web+Security+–+Interview+Questions+&+Answers.pdf]]` (7 interview questions covering Same-Origin Policy, CORS mechanics, preflight, server config, security risks, and CORS alternatives).

## Key Concepts
- **Why CORS matters:** the browser's **Same-Origin Policy (SOP)** blocks scripts from reading resources served by a different origin (protocol + domain + port). SOP protects users but breaks legitimate cross-origin APIs — CORS is the controlled, opt-in relaxation of SOP. See [[Same_Origin_Policy]].
- **How CORS works:** the browser attaches an `Origin` header to cross-origin requests and inspects the response for `Access-Control-Allow-Origin`. If origins don't match (or `*` is used with credentials), the browser blocks the JS from reading the response. Enforcement is **by the browser, not the server**. See [[How_CORS_Works]].
- **Preflight requests and CORS headers:** for "non-simple" requests (PUT/DELETE, custom headers, `application/json`), the browser sends an `OPTIONS` preflight first; the server must approve methods, headers, and origin before the actual request is sent. Key headers: `Access-Control-Allow-Origin`, `-Methods`, `-Headers`, `-Max-Age`, `-Allow-Credentials`. See [[Preflight_Requests]].
- **Security risks and common misconfigurations:** `Access-Control-Allow-Origin: *` on credentialed endpoints, reflecting the `Origin` header without allowlisting, pairing `*` with `Allow-Credentials: true` (rejected by browsers), and forgetting that CORS is not authentication. See [[CORS_Security_Risks]].
- **Handling CORS in APIs:** REST frameworks expose middleware/config (Express `cors`, Spring `@CrossOrigin`, Django `CORS_ALLOWED_ORIGINS`, FastAPI `CORSMiddleware`); GraphQL has a single POST endpoint so CORS rules are simpler but must still account for preflight on `application/json`. See [[CORS_In_APIs]].
- **Alternatives to CORS for cross-origin requests:** reverse proxy (Nginx `proxy_pass`, the client and API share one origin — CORS never triggers) and API Gateway handling CORS centrally (AWS API Gateway, Kong, Apigee) across many backends. See [[CORS_Alternatives]].

## Concept Notes
- [[Same_Origin_Policy]]
- [[How_CORS_Works]]
- [[Preflight_Requests]]
- [[CORS_Security_Risks]]
- [[CORS_In_APIs]]
- [[CORS_Alternatives]]

## Flashcards
See flashcards: [[CORS-Flashcards]]

## Related Notes
- [[HTTP_Statelessness]]
- [[Session_Security]]
- [[State_Management_Techniques]]
- [[Client_Server_Model]]

---

*Last updated: 2026-07-18*
