---
tags: [flashcards, system-design, web-concepts, cors]
created: 2026-07-18
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# CORS — Flashcards

<!-- > One card per interview question (Q1–Q7) from `CORS+&+Web+Security+–+Interview+Questions+&+Answers.pdf`, plus synthesis cards on the headline trade-offs and gotchas. Source notes: [[Same_Origin_Policy]], [[How_CORS_Works]], [[Preflight_Requests]], [[CORS_Security_Risks]], [[CORS_In_APIs]], [[CORS_Alternatives]]. -->

<!-- What is the Same-Origin Policy, and why does it exist? -->
<!-- ? -->
<!-- SOP is a browser-enforced security mechanism that prevents scripts on one origin (scheme + host + port) from reading resources from a different origin. It exists to stop malicious sites from making unauthorized requests on a user's behalf, protect sensitive data like auth tokens and session cookies, and form the foundation for mitigating cross-site attacks like CSRF. -->
<!-- #flashcards/system-design -->

<!-- What three components define an "origin"? When are two URLs considered same-origin? -->
<!-- ? -->
<!-- Scheme (protocol), host (domain), and port. Two URLs are same-origin only if all three match exactly — path, query, and fragment are ignored. `https://app.com/a` and `https://app.com/b?x=1` are same-origin; `https://app.com` vs `http://app.com` (scheme), `app.com` vs `api.app.com` (host), or `:443` vs `:8080` (port) are not. -->
<!-- #flashcards/system-design -->

<!-- Can a page send a cross-origin request even without CORS? Why is that a CSRF problem? -->
<!-- ? -->
<!-- Yes — SOP blocks *reading the response*, not *sending*. Forms, `<img>`, `<script>`, and `fetch` all fire cross-origin. The problem: state-changing requests (a money-transfer POST) still execute on the server even when the attacker's JS can't read the response. That's the whole CSRF threat — and why CSRF tokens / `SameSite` cookies are needed on top of SOP. -->
<!-- #flashcards/system-design -->

<!-- Is CORS the same as authentication? -->
<!-- ? -->
<!-- No. CORS only tells the browser whether to hand the response to JavaScript — it validates **origin**, not **user**. A request without CORS approval still hits your server and may execute. CORS must be combined with real authentication (sessions, JWTs) and authorization. Non-browser clients (curl, servers, mobile apps) ignore CORS entirely. -->
<!-- #flashcards/system-design -->

<!-- How does CORS enable cross-origin requests? -->
<!-- ? -->
<!-- CORS is a browser security feature that allows controlled cross-origin access via HTTP headers. The browser attaches an `Origin` header to cross-origin requests; the server returns `Access-Control-Allow-Origin` indicating which origins may read the response. The browser compares the two and only delivers the response to JS if they match (or the server returned `*`). Enforcement is on the browser side. -->
<!-- #flashcards/system-design -->

<!-- Who actually enforces CORS — the server or the browser? Why does this matter? -->
<!-- ? -->
<!-- The browser. The server just returns headers; the browser reads them and decides whether to expose the response to JS. This matters because (1) a CORS error means the request *still hit your server*, so CORS is not server protection, and (2) CORS is not authentication — it only protects JS reading responses, not your endpoints from being called. -->
<!-- #flashcards/system-design -->

<!-- Why does `Access-Control-Allow-Origin: *` break credentialed requests? -->
<!-- ? -->
<!-- The wildcard means "any origin," which combined with cookies/credentials would let any site make authenticated requests on a user's behalf. Browsers refuse the combination: when `Access-Control-Allow-Credentials: true`, you must return a specific origin, never `*`. -->
<!-- #flashcards/system-design -->

<!-- What's the difference between a "simple" and a "preflighted" CORS request? -->
<!-- ? -->
<!-- Simple requests (`GET`/`HEAD`/`POST` with only safelisted headers and `Content-Type` of `application/x-www-form-urlencoded` / `multipart/form-data` / `text/plain`) go straight to the server; response headers decide access. Non-simple requests (PUT/DELETE, custom headers like `Authorization`, `application/json` bodies) trigger an `OPTIONS` preflight first — the real request is only sent if the preflight approves. -->
<!-- #flashcards/system-design -->

<!-- What is a preflight request, and when is it required? -->
<!-- ? -->
<!-- A preflight is an `OPTIONS` request the browser sends *before* the real cross-origin request to check whether the server allows it. Required when: (1) the method is not `GET`/`HEAD`/`POST`, (2) custom headers like `Authorization` are set, or (3) `Content-Type` is not form-encoded / multipart / text-plain — so `application/json` always triggers one. -->
<!-- #flashcards/system-design -->

<!-- Walk through the preflight flow end to end. -->
<!-- ? -->
<!-- 1) Browser sends `OPTIONS` with `Origin`, `Access-Control-Request-Method`, `Access-Control-Request-Headers`. 2) Server responds with `Access-Control-Allow-Origin`, `-Allow-Methods`, `-Allow-Headers` (and `-Allow-Credentials` / `-Max-Age`). 3) If approved, the browser sends the real request. 4) The server's response on the real request must *also* carry `Access-Control-Allow-Origin` or JS still can't read it. -->
<!-- #flashcards/system-design -->

<!-- Name the CORS *request* headers and the CORS *response* headers. -->
<!-- ? -->
<!-- Request: `Origin` (always), `Access-Control-Request-Method` and `Access-Control-Request-Headers` (preflight only). Response: `Access-Control-Allow-Origin`, `-Allow-Methods`, `-Allow-Headers`, `-Expose-Headers`, `-Allow-Credentials`, `-Max-Age`. -->
<!-- #flashcards/system-design -->

<!-- How do you configure CORS headers on a server? -->
<!-- ? -->
<!-- The server returns the right headers in its response. In Express, set `Access-Control-Allow-Origin` (validated against an allowlist, never reflected blindly), `-Allow-Methods`, `-Allow-Headers`, optionally `-Allow-Credentials` and `-Max-Age`, and respond `204` to `OPTIONS`. In Nginx use `add_header … always` and return `204` for `OPTIONS`. The four key headers are `Allow-Origin` (which domains), `Allow-Methods` (which verbs), `Allow-Headers` (which custom request headers), and `Allow-Credentials` (whether cookies may be sent). -->
<!-- #flashcards/system-design -->

<!-- Why set `Access-Control-Max-Age`? What's a reasonable value? -->
<!-- ? -->
<!-- Each preflight is an extra round trip; `Max-Age` lets the browser cache the preflight answer for a given (origin, method, headers) tuple so subsequent requests skip it. ~600 seconds (10 minutes) is a common production default — long enough to amortize across a session, short enough to roll out policy changes within minutes. Browsers cap the effective value (Chrome ~2h, Firefox ~24h). -->
<!-- #flashcards/system-design -->

<!-- What are the common security risks associated with CORS? -->
<!-- ? -->
<!-- Three big ones: (1) Overly permissive `Access-Control-Allow-Origin: *` on authenticated resources → data leaks. (2) Allowing credentials (`Allow-Credentials: true`) without restricting origins → any site can make credentialed requests. (3) Exposing sensitive headers via `Allow-Headers` / `Expose-Headers`. Mitigate by allowlisting trusted origins explicitly, never pairing `*` with credentials, and pairing CORS with real authentication. -->
<!-- #flashcards/system-design -->

<!-- Why is reflecting the `Origin` header into `Access-Control-Allow-Origin` a vulnerability? -->
<!-- ? -->
<!-- Reflection is functionally identical to `*` — every origin is "approved" — but it bypasses the browser's `*` + credentials block. Combined with `Allow-Credentials: true`, any malicious site can call your API on behalf of a logged-in user and read the response. Correct pattern: validate `Origin` against a finite allowlist, then echo only if it matches, with `Vary: Origin`. -->
<!-- #flashcards/system-design -->

<!-- Why is `Origin: null` dangerous to allowlist? -->
<!-- ? -->
<!-- Sandboxed `<iframe>`s, `data:` URIs, and `file://` pages send `Origin: null`. If you admit `null`, any of those untrusted contexts can make credentialed calls against your API. Treat `null` as untrusted — never add it to an allowlist. -->
<!-- #flashcards/system-design -->

<!-- Why must you anchor regexes when matching subdomains in CORS? -->
<!-- ? -->
<!-- `endsWith('app.com')` admits `evil-app.com` and `app.com.evil.com`; `/app\.com/` admits `notapp.com`. Anchor your pattern (`/^https:\/\/([a-z0-9-]+\.)?app\.com$/`) or compare against exact origin strings — a sloppy regex is the same as `*`. -->
<!-- #flashcards/system-design -->

<!-- "I set everything to `*` to make CORS work." Why is this wrong, and what's the safe alternative? -->
<!-- ? -->
<!-- `*` on authenticated resources lets any site read user-specific responses, and is outright rejected by the browser when credentials are involved. Safe alternative: maintain a finite origin allowlist, validate the `Origin` header against it per request, echo only the matched origin with `Vary: Origin`, and use `Allow-Credentials: true` only when you genuinely need cookies cross-origin. -->
<!-- #flashcards/system-design -->

<!-- How do you configure CORS for a REST API? -->
<!-- ? -->
<!-- Register a global CORS middleware with an explicit origin allowlist, allowed methods (`GET`/`POST`/`PUT`/`DELETE`/`OPTIONS`), allowed headers (`Authorization`, `Content-Type`), `credentials: true` if using cookies, and a sensible `maxAge`. Express uses the `cors` package; Spring `WebMvcConfigurer.addCorsMappings`; Django `django-cors-headers` with `CORS_ALLOWED_ORIGINS`; FastAPI `CORSMiddleware`. Always handle `OPTIONS` automatically and emit `Vary: Origin` when origin is dynamic. -->
<!-- #flashcards/system-design -->

<!-- What's different about CORS for a GraphQL API? -->
<!-- ? -->
<!-- Two things. (1) GraphQL typically exposes a single POST endpoint (`/graphql`), so CORS rules collapse to one route. (2) Every client POSTs JSON, and `application/json` always triggers a preflight — so your server must correctly answer `OPTIONS /graphql` with `204` and the right headers or queries silently fail. You use the same CORS middleware as the underlying HTTP framework. -->
<!-- #flashcards/system-design -->

<!-- A GraphQL client's queries silently fail in the browser but work in curl. Most likely cause? -->
<!-- ? -->
<!-- The preflight isn't being answered. `application/json` POSTs trigger an `OPTIONS /graphql` preflight; if your server returns `405` or doesn't set the CORS headers on the `OPTIONS` response, the browser never sends the real POST. curl works because it ignores CORS. Fix: register CORS middleware properly and ensure `OPTIONS` returns `204` with `Allow-Origin`, `-Allow-Methods`, `-Allow-Headers`. -->
<!-- #flashcards/system-design -->

<!-- Does cookie-based auth vs Bearer-token auth change your CORS setup? -->
<!-- ? -->
<!-- Yes. Cookie/session auth needs `Access-Control-Allow-Credentials: true` plus a specific origin (never `*`), paired with `SameSite` cookies and CSRF tokens. Bearer tokens in `Authorization` don't need credentials mode — the client sets the header explicitly — but `Authorization` is a custom header so it always triggers a preflight. Either way, CORS only controls which origins may read responses; the API authenticates and authorizes independently. -->
<!-- #flashcards/system-design -->

<!-- What are the alternatives to CORS for handling cross-origin requests? -->
<!-- ? -->
<!-- Three: (1) **Reverse proxy** — Nginx/Caddy serves the SPA and proxies `/api/*` to backends, so the browser sees one origin and CORS never triggers. (2) **API Gateway handling CORS** — AWS API Gateway / Kong / Apigee centralize CORS policy across many backends. (3) **JSONP** — deprecated and unsafe (injects a `<script>` tag, GET-only, vulnerable to data theft). Modern systems use (1) or (2); JSONP should not be used in new code. -->
<!-- #flashcards/system-design -->

<!-- How does a reverse proxy eliminate the need for CORS? -->
<!-- ? -->
<!-- The proxy serves both the SPA (from `/`) and the API (by forwarding `/api/*` to backends) under the same origin (e.g. `app.com`). Because the browser sees same-origin requests, SOP is satisfied automatically — no `Origin` comparison, no preflight, no CORS headers needed. The backend stays browser-agnostic; the trade-off is an extra network hop through the proxy. -->
<!-- #flashcards/system-design -->

<!-- What does an API Gateway add over configuring CORS in each service? -->
<!-- ? -->
<!-- Centralized, consistent CORS policy across many backends; one place to manage, audit, and rotate origin allowlists; fine-grained per-route rules; built-in handling of `OPTIONS` preflight (often as a mock integration with no backend call); and decoupling of services from browser concerns. Trade-off: extra infra component, an additional hop, and a single CORS-config surface you must get right. -->
<!-- #flashcards/system-design -->

<!-- Why is JSONP deprecated and unsafe? -->
<!-- ? -->
<!-- It loads data via a `<script src>` tag whose response is executed as JS. Any site can include the tag, so if the endpoint is cookie-authenticated a malicious site can read the data. It's GET-only, has no proper error handling, and risks XSS via attacker-supplied callback names. CORS solves the same problem safely; JSONP should not be used. -->
<!-- #flashcards/system-design -->

<!-- You've configured CORS at both the API Gateway and inside the backend service. What goes wrong? -->
<!-- ? -->
<!-- Response headers get duplicated (e.g. two `Access-Control-Allow-Origin` values). Browsers reject responses with duplicated CORS headers, so requests fail with a confusing CORS error even though each layer is "correct" on its own. Pick one layer — usually the gateway — and remove the other. -->
<!-- #flashcards/system-design -->

<!-- If the API Gateway enforces CORS, can your backend services safely be exposed directly to the internet? -->
<!-- ? -->
<!-- No. CORS is browser-only; a non-browser client (curl, a script, another server) ignores it entirely. If backends are directly reachable, attackers bypass the gateway's CORS policy by hitting them straight. Network-isolate backends so only the gateway (and other internal services) can reach them, and require authentication on every endpoint regardless. -->
<!-- #flashcards/system-design -->

---

<!-- *Last updated: 2026-07-18* -->
