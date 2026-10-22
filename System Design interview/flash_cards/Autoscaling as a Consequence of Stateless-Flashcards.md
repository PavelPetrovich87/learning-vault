#flashcards/system-design

# Autoscaling as a Consequence of Stateless-Flashcards

---

Why does autoscaling require stateless? Give BOTH directions.
?
Scale-out: a new server only helps if any server can serve any client — in stateful, old clients stay pinned and the new node idles while "their" server dies. Scale-in: removing a stateful node kills its live sessions — automated scale-in would log users out (the "night of the failed scale-in"); stateless removal is safe after connection drain. Automatable = routine in both directions — that's what stateless earns.

---

Vertical scaling "works" for a hot server. Why is it still not a valid autoscaling strategy?
?
Not elastic in time: you can't rent extra power for three peak hours. Vertical steps in whole machines, usually with replacement downtime (the swap itself loses in-flight requests), the price grows superlinearly, and there's a hardware ceiling. Elasticity means capacity matches the bill in time — horizontal per-hour instances do, resized iron doesn't.

---

Manual vs automatic horizontal scaling — what does the automatic kind close that manual can't?
?
Unpredictable spikes (×100). A human reacts in minutes and not at 03:00; an autoscaler watches load and adds/removes by thresholds with cooldowns in seconds. Static margin ×2 is bounded — the chain is: static margin bounded → need autoscaling → autoscaling needs stateless. Manual scaling handles forecastable growth; only automatic handles spikes you didn't forecast.

---

What does a real autoscaler do before removing an instance (what naive on/off simulation skips)?
?
Connection drain: stop routing new traffic to it, let in-flight requests finish, then kill. Skipping drain turns every scale-in into a small outage burst. Plus cooldowns/hysteresis around thresholds so the pool doesn't flap nodes in and out at the boundary.
