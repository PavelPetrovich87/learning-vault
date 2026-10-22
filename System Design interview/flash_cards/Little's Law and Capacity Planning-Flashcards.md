#flashcards/system-design

# Little's Law and Capacity Planning-Flashcards

---

State Little's Law with units and give a worked example.
?
L = λ·W, where L = average requests in flight (dimensionless counter), λ = arrival rate in req/s (peak QPS from the funnel), W = response time in seconds. Example: λ=1000 rps, W=50ms → 50 requests in flight on average — roughly the slots/connections to hold for steady state.

---

Why is λ·W ≤ N a threshold and not a target? Why won't a ×2 capacity margin always save you?
?
Near the threshold the queue grows non-linearly and p99 explodes — head-of-line blocking. Systems fail at peak, not average: a ×2 margin dies at an overlapping ×3–4 peak because λ doubles → L doubles → threshold crossed. You can't pre-buy ×100: static margin is bounded, so the way out is autoscaling — which requires a stateless tier.

---

What are the dimensions of λ, W, L in Little's Law — and which pair do people confuse?
?
λ = req/s, W = s/req, L = requests in flight (no units — a counter). λ and L are related by the formula but never interchangeable; calling L=400 "the peak" on an interview is an instant red flag.
