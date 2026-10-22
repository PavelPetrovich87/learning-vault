#flashcards/system-design

# Headroom - Practical Capacity Ceilings-Flashcards

---

The math says the limit is 40 rps, but queues appear at 38. Why is the practical ceiling below the theoretical one?
?
Arrival jitter. Little's Law sizes the average, but arrivals come in bursts: near saturation (ρ → 1) random bursts find all slots busy and queue, and wait time grows explosively as utilization approaches 1. Measured: hard limit 50 rps, micro-queues from pure jitter at 48 (96%). Hence the rule: plan for 70-80% utilization.
<!--SR:!2026-09-10,3,250-->

---

Why plan capacity at 70-80% utilization instead of 100%?
?
Because queues appear from pure jitter well before the hard limit — the last 20% of utilization is bought with exploding tail latency. Headroom is also the failure budget: lose one node and survivors run hotter; with headroom they absorb it, without it the pool collapses into queues. Plus it covers the autoscaler's reaction gap until a new node is warm.

---

Capacity is now ample after a spike, but the deep queue isn't vanishing. What sets the drain speed?
?
A queue drains at the SURPLUS rate (new capacity − input), not at the new total capacity — and the surplus is small. It grew at the deficit rate (fast), drains at (C′ − λ) (slow): a 150-deep queue with +12 rps spare takes ~12+ seconds. Recovery is much slower than collapse — the price of running near the threshold lingers after the spike.
