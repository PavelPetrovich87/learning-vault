#flashcards/system-design

# 🎴 Typeahead System Design Flashcards (Senior Engineering)

---

## 1. Capacity Estimation (DAU → QPS)

In **Typeahead Capacity Estimation**, why does 100M DAU transform into 230,000+ QPS?
?
**Session & keystroke multiplier:**
- 100M users × 1 session/day (conservative)
- ~10 searches/user/day = 1B searches/day
- ~6 chars/query + 1 debounce delay → ~7 requests per query (including prefix increments)
- 1B × 7 = 7B requests/day ÷ 86,400 ≈ **81,000 QPS base**
- Peak = 3× average = **~230,000 QPS peak**
<!--SR:!2026-04-24,1,230-->

---

What is the typical Peak-to-Average ratio used in capacity planning and why?
?
**3× rule.** Most traffic follows diurnal patterns — people search in bursts (morning, lunch, evening). Designing for 3× average ensures availability during viral events, news cycles, or product launches without cascading failures.
<!--SR:!2026-04-24,1,230-->

---

How do you estimate Trie memory footprint for 100M nodes with Top-K storage?
?
**Per-node calculation:**
- Children pointers: 26 × 8 bytes = **208 bytes**
- Top-K entries: 5 × (4 + 4) = **40 bytes** (query_id + score)
- Metadata: ~16 bytes
- **Total ≈ 264 bytes/node**

**100M nodes × 264 bytes ≈ 26 GB RAM** (single machine)
This is why we sharding by prefix — no single machine holds the full Trie.
<!--SR:!2026-04-24,1,230-->

---

## 2. Trie Optimization

Why is storing ==Top-K results in each Trie node== a classic Space vs Time trade-off?
?
- **Time benefit:** Instant return of popular suggestions — O(1) lookup at prefix match
- **Space cost:** Each node stores 5 extra query_ids + scores (~40 bytes)
- **Scale impact:** At 100M nodes, that's +4GB RAM just for Top-K alone
- **Trade-off:** If cache hit rate is high, you save Trie lookups. If queries are long-tail, you're wasting memory on results nobody clicks.
<!--SR:!2026-04-30,2,230-->

---

Why store ==Query_ID== instead of full strings in Trie nodes?
?
**Memory efficiency + decoupling:**
1. **String storage:** "how to bake a chocolate cake" = 28 bytes × 100M = 2.8GB just for one branch
2. **ID storage:** 4-byte integer = 4 bytes per node
3. **Hash Map separation:** Full strings live in a separate `query_id → full_string` dictionary
4. **Update bonus:** Updating a query string only requires updating the Hash Map, not rewriting Trie paths

This reduces Trie memory by **~70%**.
<!--SR:!2026-04-24,1,230-->

---

Why do we ==not update the main Trie in real-time== during writes?
?
**Blocking & consistency problems:**
1. **Write locks:** Trie is a tree structure — a write to one branch can block reads on another
2. **Re-balancing:** Frequent insertions cause structural reorganization
3. **Inconsistency window:** Between write completion and full tree propagation, different replicas see different data
4. **Solution:** Write to an append-only log + rebuild Trie offline periodically (MapReduce job)
<!--SR:!2026-04-24,1,230-->

---

## 3. Sharding & Scaling

Why is ==alphabetical sharding (first letter)== a disaster for hot-spot avoidance?
?
**Data skew breakdown:**
- 'S' prefix: "software", "sql", "stackoverflow", "spotify" — millions of queries
- 'Z' prefix: "zebra", "zip", "zoom" — thousands of queries

**Result:**
- Shard handling 'S' is 1000× hotter than one handling 'Z'
- Load balancer sees wildly different CPU/memory across shards
- Single "viral prefix" can overwhelm one shard while others idle

**Fix:** Use **Consistent Hashing on prefix hash**, not lexicographic order.
<!--SR:!2026-04-24,1,230-->

---

Why is ==n=3 prefix length== the "sweet spot" for Consistent Hashing sharding?
?
**n = prefix length to hash for shard selection:**

| n value | Problem |
|---------|---------|
| **n = 1** | Only 26 shards — too coarse, hot-spot still possible |
| **n = 2** | 676 shards — better, but still limited |
| **n = 3** | **17,576 shards** — good balance |
| **n = 4+** | Explodes state space, rebuild time grows exponentially |

**Too small:** Poor load distribution
**Too large:** Shard state becomes unwieldy, rebalancing breaks cache locality
<!--SR:!2026-04-24,1,230-->

---

Why does ==Eventual Consistency suffice== for Typeahead read replicas?
?
**User tolerance for stale suggestions:**
- Typeahead is a **soft real-time** feature — slight staleness (seconds/minutes) is imperceptible
- Users type slowly (~300ms between keystrokes)
- Typical prefix queries are dominated by **long-standing popular queries** (news, weather)
- **Trade-off:** Accept 1-2s staleness in exchange for **read scalability** — replicas offload 90% of QPS without synchronous coordination overhead
<!--SR:!2026-04-24,1,230-->

---

## 4. Lambda Architecture

How do ==Batch Layer== and ==Real-time (Streaming) Layer== work together in Typeahead ranking?
?
```
Query Score = α × Historical_Frequency + β × Freshness_Boost

Batch Layer: Computes "yesterday's truth" — slow, comprehensive, accurate
  → Heavy Trie with full history
  → Handles 99% of queries for known popular terms

Real-time Layer: Handles "today's trends" — fast, incremental, ephemeral
  → Redis cache of trending queries
  → Catches news, viral events, breaking topics

Merging: During lookup, union results from both, re-rank by composite score
```
<!--SR:!2026-04-26,3,250-->

---

What are the components of the ==Ranking Score== in Typeahead?
?
**Score = Historical_Frequency + Freshness_Boost**

| Component | Source | Weight |
|-----------|--------|--------|
| **Historical Frequency** | Batch Trie (weeks/months of data) | α (e.g., 0.7) |
| **Freshness Boost** | Real-time stream (last hours) | β (e.g., 0.3) |
| **Recency decay** | Time since last query | Bonus multiplier |
| **Click-through rate** | User engagement signal | Additional signal |

The α/β tuning depends on query type: "weather" needs freshness, "javascript array map" needs accuracy.
<!--SR:!2026-04-24,1,230-->

---

How do you safely deploy a ==giant Trie index update== without service interruption?
?
**Rolling deploy with double-Trie:**
1. Build new Trie on a background machine (offline)
2. When ready, send a **config flag** to switch load balancers
3. **Method A (Feature Flag):** API servers start routing traffic to new Trie
4. **Method B (Shadow Mode):** New Trie serves shadow traffic until validated
5. Old Trie stays in memory for 5-10 minutes as **rollback target**
6. Old Trie released after stability confirmed

**Critical:** Both Tries must share the same Hash Map for query strings.
<!--SR:!2026-04-24,1,230-->

---

## 5. Latency & Client Optimization

Break down the ==100ms latency budget== in a Typeahead request chain:
?
```
Browser (render + JS)          │  10-15ms
DNS resolution                 │   5-10ms
TLS handshake                  │  10-20ms
Load Balancer                  │   2-5ms
API Server (routing)           │   5-10ms
Trie lookup (in-memory)        │   1-3ms  ← sub-millisecond
Redis cache (hot data)        │   1-2ms
Database (cold miss)           │  10-50ms
Response + TCP round-trip      │  10-15ms
─────────────────────────────────────────
Total (cache hit)              │  ~50ms  ✓
Total (cache miss + DB)        │ ~100ms  ⚠
```
<!--SR:!2026-04-24,1,230-->

---

How does ==Debounce== reduce backend QPS compared to ==Throttle==?
?
| Technique | Behavior | QPS Impact |
|-----------|----------|------------|
| **Debounce** | Reset timer on each keystroke; fire **only after N ms of silence** | **~10-20× reduction** for fast typers |
| **Throttle** | Fire **at most once per N ms**, regardless of typing speed | ~3-5× reduction |

**Example:** User types "python" (6 keystrokes, 100ms apart)
- No debounce: 6 requests
- Debounce 300ms: **1 request** (after "n" silence)
- Throttle 150ms: 4 requests (fires at t=0, 150, 300, 450)
<!--SR:!2026-04-24,1,230-->

---

How should a client handle ==HTTP 429 (Rate Limited)== response with ==Retry-After==?
?
**Correct approach:**
```javascript
async function fetchWithRetry(url, retries = 3) {
  const response = await fetch(url);

  if (response.status === 429) {
    const retryAfter = parseInt(
      response.headers.get('Retry-After') || '1000'
    );
    // Respect the server's directive, add jitter
    await sleep(retryAfter + Math.random() * 200);
    return fetchWithRetry(url, retries - 1);
  }

  return response;
}
```

**Anti-patterns to avoid:**
- ❌ Immediate retry (makes load worse — "thundering herd")
- ❌ Fixed busy-wait loop
- ❌ Retrying without exponential backoff
<!--SR:!2026-04-24,1,230-->

---

What is the "Coat Check" mental model analogy for a ==URL Shortener==?
?
**Analogy:** The Coat Check at a theater.
- You give the attendant your **big, heavy coat** (long, messy URL)
- They give you a **tiny plastic token** (short URL slug)
- When you return the token, they look up the mapping and return your coat

**Key insight:** The token (short link) is just a reference — the real value (the coat/long URL) lives in storage until needed. The shortening service is purely a lookup and redirect mechanism.
<!--SR:!2026-04-24,1,230-->

---

## Edge Cases & Gotchas

Why does ==Protocol Normalization== matter in URL Shorteners?
?
**Problem:** `http://example.com` and `https://example.com` could be treated as different URLs.

**Solution:** Always normalize to one protocol (prefer HTTPS). Strip trailing slashes, `www.` prefix, and query parameter order before generating the hash/ID.

**Impact:** Without normalization, you get duplicate entries for the same destination, split click analytics, and wasted ID space.
<!--SR:!2026-04-24,1,230-->

---

In a Typeahead Trie, why might ==Top-5 results differ== between shards?
?
1. **Sharding splits the prefix space** — "java" and "javascript" might be on different shards with different local popularity rankings
2. **Async replication lag** — one shard has newer data than another
3. **Result:** Global Top-5 across all shards requires a **scatter-gather merge** at the aggregator, not per-shard Top-K

This is why distributed ranking is harder than single-machine ranking.
<!--SR:!2026-04-24,1,230-->

---

What is the "Memory/Accuracy Paradox" in Trie-based Typeahead?
?
**The paradox:**
- **More nodes** = better prefix coverage = higher recall = more memory
- **Fewer nodes** = faster lookups = lower memory = missed suggestions

**The solution:** Use a **sparse Trie** (only store nodes for popular prefixes) combined with a **fallback to trigram indexes** for rare queries. This balances memory (~26GB vs 100GB for full Trie) with accuracy (~95% vs 99%).

**YouTube/Search engines explicitly sacrifice recall for memory efficiency.**
<!--SR:!2026-04-24,1,230-->
