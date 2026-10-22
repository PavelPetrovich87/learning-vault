#flashcards/system-design

# Percentiles and Tail Latency-Flashcards

---

Your dashboard shows average latency 80ms. Is the service fast?
?
Can't tell — the mean hides the tail. 99×10ms + 1×10s gives a "mean 110ms" while one user in a hundred sits on a 10s hang. Report the trio p50/p95/p99: p50 = typical experience, p95/p99 = the tail — and the tail is where the most valuable users live (Amazon: slowest requests = most data in account).

---

Response time vs latency — what's the difference and why does it matter?
?
Response time = what the client sees: service time + network + queue wait. Latency = only the wait, excluding the work. Response time ≥ latency, always. Diagnostic value: slow-from-queue ≠ slow-from-processing — before optimizing, find which one you have.
<!--SR:!2026-09-08,1,230-->

---

Why measure response time on the client side, and why must load tests fire requests independently of responses?
?
The server doesn't see its own queue from the client's vantage point — tail latency is mostly queue waits (head-of-line blocking: slow requests occupy all slots, fast ones queue behind them). A test client that waits for each response never builds a queue, so it tests a system that doesn't exist in production.

---

Why stop at p99 instead of chasing p99.9+?
?
Diminishing returns: above p99.9 the causes are often random external factors you don't control, and the price of optimization doesn't pay back (Amazon explicitly stopped there). Name the percentile the business needs and stop.
