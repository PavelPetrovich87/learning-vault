#flashcards/system-design

# Stateless vs Stateful - the SAM Lens-Flashcards

---

Give the one-sentence test for whether a web tier is stateless.
?
Kill any server at any moment — any in-flight request can be replayed on another server and the user keeps their session. If true, the tier is stateless. Equivalent view: nothing user-critical lives only in a server's RAM.

---

Name the three SAM axes and what breaks in each for a stateful tier.
?
S — Scalability: new servers only take new clients; the LB routes by ownership, not load. A — Availability: server death kills its in-RAM sessions → that share of users re-logins. M — Modifiability: deploy = restart = RAM wiped → rolling logouts or maintenance window. Stateless frees all three with one move: the session dictionary moves to a shared store.

---

Why is deploying a new version painful for a stateful tier? Walk through the mechanics.
?
A deploy changes code, not sessions — but a restart clears RAM, and in a stateful tier the session dictionary lives exactly there: code and state die together. Options: rolling (portioned logouts), maintenance window (everyone at once), or session drain/migration (complex). In stateless they're separated — restarts don't touch sessions, users don't notice.

---

Where do sessions live in the naive (stateful) approach, and what exactly does the cookie carry?
?
Sessions live in a dictionary in the server's process RAM: sessions["abc123"] = {user, cart}. The cookie carries ONLY the key (session_id) — it's a coat-check ticket, not the coat. Confusing the two leads to thinking cookies "hold the session" and missing why a second server can't recognize the user.

---

Contrast the mechanics of losing one server in a stateless pool vs a stateful tier.
?
Stateless: a one-time loss of that node's in-flight requests (slots + travel), traffic reflows instantly; you lose capacity, the service stays healthy. Stateful: you lose in-flight INCLUDING the whole queue the pinned overload accumulated, plus the sessions of pinned clients (a re-login window where ALL their requests fail, ≈ their rps × window), and then re-sticking overloads the survivor. Stateless loses capacity; stateful loses capacity + state + user experience.

---

When a sticky load balancer assigns a client to a server, what is the assignment based on — and what does that imply for one very heavy user?
?
The FIRST request (or a hash of the client id) — never current load. One heavy user pins one server; adding servers doesn't relieve it; balancing by load is impossible by construction because routing is by ownership. It's a web-tier-only phenomenon: the shared DB doesn't need stickiness, only servers holding in-RAM sessions do.
