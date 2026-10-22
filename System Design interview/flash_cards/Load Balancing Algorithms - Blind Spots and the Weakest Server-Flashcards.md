#flashcards/system-design

# Load Balancing Algorithms - Blind Spots and the Weakest Server-Flashcards

---

Why does round-robin drown a degraded server even when a stronger sibling has spare capacity?
?
Round-robin splits by request count, not by capacity: equal counts ≠ equal load. A degraded server receiving its "equal" share accumulates a queue growing at (share − capacity) per second while the strong sibling idles. The fix is not the algorithm — it's capacity or pulling the node from rotation.

---

What does least-connections actually minimize, and what does it NOT know?
?
It minimizes busy slots + queue depth + in-flight requests — load counters. It does NOT know server capacity: it keeps feeding a weak server until its counter rises to the others' level (the weak node ends up with a small permanent "equalizer" queue — honest physics). LC redistributes unevenness; it cannot fix a total capacity deficit: the algorithm divides the pie, it doesn't bake a new one.

---

Why must least-connections count in-flight requests (nginx least_conn semantics)?
?
Because a server's counters only move when a request ARRIVES — while it travels, the server looks free. Without in-flight, the LB bursts dozens of requests at the "least loaded" node during one network delay (600 ms at 40 rps ≈ 24 invisible requests), causing 0/N ↔ N/N+queue oscillations. nginx least_conn counts the connection from the moment of forwarding, closing that blind window.

---

A pool has ample total capacity, yet aggregate p95 breaks the SLO. Who sets your tail, and what's the fix?
?
The weakest/slowest backend, not the average. One degraded node carrying 25% of traffic with a ~1 s queue set pool p95 = p99 ≈ 1100 ms while p50 stayed 200 ms — the SLO broke with ample total capacity (60 rps capacity vs 40 rps load). Remedy: health checks + pull the degraded node from rotation. The LB won't heal a sick server; it will faithfully keep poisoning your percentiles.
