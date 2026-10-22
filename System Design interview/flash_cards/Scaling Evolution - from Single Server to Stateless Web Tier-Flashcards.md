#flashcards/system-design

# Scaling Evolution - from Single Server to Stateless Web Tier-Flashcards

---

Walk Su's four-stage scaling evolution: at which stage does the system first survive a component failure?
?
Stage 3 (load balancer + second web server) — and only a web-server failure; the DB is still single and fatal. Stages: 1) single server — anything kills it; 2) web/DB split — independent scaling, still no fault tolerance; 3) LB + 2nd web server — first survivable failure; 4) stateless web + DB replication — web death loses no sessions, DB-node death survived by replicas.

---

What does splitting web and DB onto separate machines actually buy — and what does it NOT buy?
?
Buys: independent scaling of web tier and data tier (they peak differently: reads vs writes). Does NOT buy: fault tolerance — either machine failing still takes the whole site down. Fault tolerance appears only where there is more than one of a component (stage 3).

---

Why is autoscaling a consequence of statelessness rather than a separate technology?
?
Because it makes servers replaceable: adding one = boot + register in the LB; removing one = drain traffic + kill. Any server can serve any client, so add/remove is routine — and routine is automatable. In a stateful tier, add/remove means reshuffling sticky sessions: slow and risky, the opposite of what you want an automation to do.

---

Name the two hard constraints of vertical scaling (the anchor answer, not the cost consequence).
?
(1) A hard hardware limit — one machine's resources can't grow indefinitely. (2) No fault tolerance — the single server is a single point of failure; its death takes the site down. "Expensive" is a consequence, not one of the two anchor constraints (classic miss: naming "physical and financial").
