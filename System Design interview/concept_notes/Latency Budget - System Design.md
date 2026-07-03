---
status: analyzing
mastery: beginner
tags: [system-design, latency, performance, sre, architecture, optimization, interview, observability]
source: Google SRE Book, "Designing Data-Intensive Applications" by Martin Kleppmann, AWS Well-Architected Framework, various SRE blogs
created: 2026-06-05
updated: 2026-07-03
difficulty: advanced
---

# 🧠 Topic: Latency Budget - System Design

> [!abstract] Executive Summary (ELI5)
> Imagine you have **100 dollars** to spend on a shopping trip. You can't spend more, or you'll be in debt. A **latency budget** is the same idea, but instead of dollars, you have **milliseconds** — say, 200ms — to answer a user's request. Every hop your request makes (database, cache, another service) "spends" some of those milliseconds. Your job as an engineer is to allocate those milliseconds wisely so you never go over budget, and to know exactly which "purchase" is breaking the bank.

---

## 🎯 The "Why"

### Problem Solved
You're designing a system, and someone says: "The API must respond in 200ms at p99."

Cool. But what does that actually MEAN for your architecture?

Without a latency budget, engineers make locally optimal but globally disastrous decisions:
- The frontend team builds a beautiful dashboard that makes **47 API calls** on load
- The backend team adds a "quick" feature that queries **3 databases** and **2 caches**
- The ML team adds a real-time recommendation call that takes **500ms**
- Individually, each decision seems fine. Together, the page loads in **8 seconds**.

**A latency budget forces you to:**
1. **Decompose** the 200ms goal into sub-budgets per component
2. **Trace** where time is actually spent
3. **Decide** trade-offs BEFORE building (not after users complain)
4. **Negotiate** between teams with DATA, not opinions

### The Business Impact of Latency

| Latency Increase | Impact |
|------------------|--------|
| +100ms | Amazon reported 1% revenue drop per 100ms delay |
| +500ms | Google reported 20% traffic loss |
| +1 second | User satisfaction drops 16% (Akamai study) |
| +3 seconds | 53% of mobile users abandon the page |
| +10 seconds | Users perceive the task as "impossible" |

> [!tip] Senior Insight
> Latency is not just a technical metric — it is a **business metric**. Every millisecond you shave off is directly correlated with user retention and revenue. The latency budget is the bridge between "make it fast" and "here is exactly HOW fast each part must be."

### Alternatives
- **"Just make it fast"** — vague, unactionable, leads to premature optimization
- **Load testing after building** — too late, expensive to refactor
- **SLA/SLO definitions without decomposition** — "API < 200ms" without saying WHICH part of the API
- **Gut feeling engineering** — "I think the DB query is fast enough" (spoiler: it's not)

### Trade-offs
| Approach | Pros | Cons |
|----------|------|------|
| Strict latency budget | Clear constraints, early trade-offs | May limit feature scope |
| No budget, just optimize later | Flexible, fast iteration | Technical debt, user churn, expensive rewrites |
| Budget but no enforcement | Documentation exists | Teams ignore it |
| Budget with automatic enforcement | Guaranteed compliance | Complex infrastructure (circuit breakers, timeouts) |

---

## 📜 History

| Year | Milestone |
|------|-----------|
| **1990s** | Web is born. Latency doesn't matter because everything is slow. Users wait 30 seconds for a GIF. |
| **2000s** | Google and Amazon discover latency directly correlates with revenue. Performance becomes a first-class concern. |
| **2004** | Jeff Dean's famous talk "Software Engineering Advice from Building Large-Scale Distributed Systems" introduces back-of-the-envelope latency numbers. |
| **2008** | Amazon reports the 1% = 100ms finding. The industry wakes up. |
| **2012** | Google publishes the SRE Book concepts. Latency SLOs become standard. |
| **2015** | The term "Latency Budget" enters mainstream system design discourse. |
| **2017** | Martin Kleppmann's "Designing Data-Intensive Applications" formalizes latency analysis as a core design skill. |
| **2020+** | Edge computing, 5G, and sub-100ms requirements make latency budgets essential, not optional. |

---

## 🏗️ What Is a Latency Budget?

### The Math

```
┌─────────────────────────────────────────────────────────────┐
│              Total Latency Budget: 200ms (p99)               │
├─────────────────────────────────────────────────────────────┤
│  DNS Lookup              │  ~5-50ms     │ Network baseline   │
│  TLS Handshake           │  ~20-100ms   │ (or 0ms with TLSv1.3 + 0-RTT)
│  TCP Connection          │  ~10-30ms    │ (or 0ms with HTTP/2 keepalive)
│  ────────────────────────┼──────────────┼────────────────────│
│  CDN / Edge              │  ~5-20ms     │ Static assets      │
│  Load Balancer           │  ~1-5ms      │ Routing            │
│  API Gateway             │  ~5-10ms     │ Auth, rate limit   │
│  ────────────────────────┼──────────────┼────────────────────│
│  Service A (business)    │  ~30-50ms    │ Core logic         │
│  Service B (auth)        │  ~10-20ms    │ Token validation   │
│  Service C (recommend)   │  ~20-40ms    │ ML inference       │
│  ────────────────────────┼──────────────┼────────────────────│
│  Cache (Redis)           │  ~0.5-2ms    │ Hot data           │
│  Database (PostgreSQL)   │  ~5-20ms     │ Indexed query      │
│  Database (scan)         │  ~50-500ms   │ ❌ BUDGET EXCEEDED │
│  ────────────────────────┼──────────────┼────────────────────│
│  Serialization           │  ~1-5ms      │ JSON/protobuf      │
│  Network RTT (internal)  │  ~0.1-1ms    │ Same datacenter    │
│  Network RTT (cross-AZ)  │  ~1-3ms      │ AWS/Azure/GCP      │
│  Network RTT (cross-region)│  ~50-150ms │ ❌ BUDGET KILLER   │
├─────────────────────────────────────────────────────────────┤
│  Remaining margin:       │  ~20-40ms    │ For the unexpected │
└─────────────────────────────────────────────────────────────┘
```

### Jeff Dean's Numbers (Back-of-the-Envelope)

These are the legendary numbers every system designer should know:

| Operation | Latency |
|-----------|---------|
| L1 cache reference | 0.5 ns |
| Branch mispredict | 5 ns |
| L2 cache reference | 7 ns |
| Mutex lock/unlock | 25 ns |
| Main memory reference | 100 ns |
| Compress 1KB with Zippy | 3,000 ns (3 µs) |
| Send 2KB over 1 Gbps network | 20,000 ns (20 µs) |
| Read 1MB sequentially from memory | 250,000 ns (250 µs) |
| Round trip within same datacenter | 500,000 ns (500 µs) |
| Read 1MB sequentially from SSD | 1,000,000 ns (1 ms) |
| Disk seek | 10,000,000 ns (10 ms) |
| Read 1MB sequentially from disk | 20,000,000 ns (20 ms) |
| Round trip CA to Netherlands | 150,000,000 ns (150 ms) |

**Key insight:** There are **300,000x** difference between L1 cache and cross-Atlantic network. Every time your code makes a network call, it is doing the equivalent of 300,000 cache lookups. Choose wisely.

---

## 💡 Mental Model & Analogy

### Analogy: A Restaurant Kitchen During Rush Hour

You are the head chef. A customer orders a dish, and the manager says: **"This dish must reach the table in 10 minutes, or the customer leaves."**

**Your latency budget for the dish: 10 minutes**

| Step | Budget | Actual | Status |
|------|--------|--------|--------|
| Take order | 30 sec | 45 sec | ⚠️ Over budget |
| Prep vegetables | 2 min | 1.5 min | ✅ Under budget |
| Cook protein | 4 min | 5 min | ❌ CRITICAL |
| Make sauce | 2 min | 3 min | ❌ CRITICAL |
| Plate and serve | 1.5 min | 1 min | ✅ Under budget |
| **Total** | **10 min** | **11.5 min** | ❌ **CUSTOMER LEFT** |

**The system design insight:** The problem isn't that EVERY step is slow. The problem is that you didn't know WHICH steps were over budget until the customer left. A latency budget tells you:
1. "Cook protein MUST be under 4 minutes"
2. If the protein takes 5 minutes, you either speed it up (better grill), cut something else (simpler sauce), or renegotiate the SLA (tell the manager 10 minutes is unrealistic)

**In distributed systems:**
- The "customer" is the user
- The "dish" is the API response
- Each "step" is a service call, DB query, or cache hit
- The "manager" is your SLO/SLA

---

## 🛠️ Core Principles & How to Build a Latency Budget

### Step 1: Define the Goal (SLO)

```
"The /api/v1/dashboard endpoint must respond in 200ms at p99"
```

**Key terms:**
- **p50 (median):** Half of requests are faster, half slower. Easy to meet, hides outliers.
- **p95:** 95% of requests meet the budget. Captures most user experience.
- **p99:** 99% of requests meet the budget. Captures worst-case normal operation. **This is the standard.**
- **p99.9:** 99.9% meet budget. For financial systems where every millisecond costs money.
- **p100 (max):** Theoretical, never use. One cold start on a Lambda and you're "infinite."

> [!warning] Don't Use Average!
> Average latency is MEANingless. One slow request (e.g., cache miss + DB scan) can be 10 seconds, but the average of 100 requests still looks fine at 300ms. Always use percentiles.

### Step 2: Decompose the Request Path

Draw the request flow. Every arrow is a potential latency sink:

```
User → CDN → WAF → LB → API Gateway → Auth Service → Load Balancer
                                                      ↓
User ←  CDN  ←  Response  ←  API Gateway  ←  Main Service ← Cache
                                                  ↑           ↓
                                              Database ← Fallback DB
```

**For each arrow, ask:**
1. What is the typical latency?
2. What is the p99 latency?
3. What is the failure mode latency (timeout, fallback, retry)?

### Step 3: Allocate Budgets Per Component

```
Total Budget: 200ms (p99)

Network Stack (TCP/TLS):     30ms  (15%)
CDN / Edge:                   15ms  (7.5%)
API Gateway + Auth:           25ms  (12.5%)
Main Service Logic:           50ms  (25%)
  ├─ Cache hit:               2ms
  ├─ Cache miss + DB query:   20ms
  └─ External API call:       30ms
Database Queries:             40ms  (20%)
Serialization + Network RTT:  20ms  (10%)
─────────────────────────────────────────
Contingency Buffer:           20ms  (10%)
```

**The 10% contingency buffer is non-negotiable.** Things go wrong. Cache invalidates. GC pauses. Network hiccups. Without a buffer, your p99 is guaranteed to fail.

### Step 4: Instrument and Measure

You can't manage what you don't measure.

```typescript
// Manual instrumentation (don't do this in production)
const start = Date.now();
await db.query('SELECT ...');
metrics.timing('db.query.duration', Date.now() - start);

// Better: distributed tracing
import { trace } from '@opentelemetry/api';

const span = trace.getTracer('my-service').startSpan('db-query');
try {
  const result = await db.query('SELECT ...');
  span.setAttribute('db.rows_returned', result.rows.length);
} finally {
  span.end(); // Automatically records duration
}
```

**Tools for latency tracing:**
- **OpenTelemetry + Jaeger/Zipkin** — distributed tracing
- **Prometheus + Grafana** — metrics and dashboards
- **AWS X-Ray / GCP Cloud Trace / Azure App Insights** — cloud-native tracing
- **DataDog / NewRelic / Honeycomb** — commercial APM

### Step 5: Enforce the Budget

```typescript
// Circuit breaker: fail fast instead of waiting forever
import CircuitBreaker from 'opossum';

const breaker = new CircuitBreaker(externalApiCall, {
  timeout: 50,        // 50ms max — if slower, fail
  errorThresholdPercentage: 50,
  resetTimeout: 30000
});

// Timeout wrapper
const result = await Promise.race([
  fetchExternalData(),
  new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Timeout')), 50)
  )
]);

// Async with fallback
const data = await Promise.race([
  fetchFromPrimary(),
  sleep(30).then(() => fetchFromCache())
]);
```

---

## 🔧 Techniques to Meet Your Latency Budget

### 1. Caching (The Biggest Lever)

```
Before:  User → API → Database (20ms) → API → User
After:   User → API → Cache (0.5ms) → API → User

Speedup: 40x
```

| Cache Type | Latency | Use Case |
|------------|---------|----------|
| In-process (LRU) | < 1 µs | Hot objects, tiny dataset |
| Local Redis | 0.5-2 ms | Session data, rate limits |
| Remote Redis | 1-5 ms | Shared cache across instances |
| CDN (CloudFront/Cloudflare) | 5-50 ms | Static assets, edge caching |

**Cache invalidation strategies:**
- **TTL (Time To Live):** Simple, but stale data risk
- **Write-through:** Cache updated synchronously on write (consistent but slower writes)
- **Write-behind:** Cache updated asynchronously (fast writes, eventual consistency)
- **Cache-aside:** App manages cache manually (most common, most bug-prone)

### 2. Connection Pooling

```
Without pooling:  50ms per request (TCP + TLS handshake)
With pooling:     1ms per request (reused connection)
```

```typescript
// PostgreSQL connection pool
const pool = new Pool({
  host: 'db.internal',
  max: 20,           // Max connections in pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000, // Fail fast if DB is down
});

// HTTP keep-alive (Node.js)
const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 50,
  maxFreeSockets: 10,
  timeout: 30000,
  freeSocketTimeout: 30000
});
```

### 3. Database Query Optimization

```sql
-- ❌ Full table scan — 500ms+ on large tables
SELECT * FROM orders WHERE created_at > '2024-01-01';

-- ✅ Indexed query — 5ms
CREATE INDEX idx_orders_created_at ON orders(created_at);
SELECT id, total FROM orders WHERE created_at > '2024-01-01' LIMIT 100;

-- ❌ N+1 query — 100 queries × 5ms = 500ms
for (const user of users) {
  await db.query('SELECT * FROM orders WHERE user_id = ?', [user.id]);
}

-- ✅ JOIN query — 1 query × 15ms = 15ms
SELECT u.*, o.id as order_id, o.total 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id 
WHERE u.id IN (...);
```

### 4. Asynchronous Processing

```
Synchronous:  User → API → Send Email (2s) → API → Response
Asynchronous: User → API → Queue (5ms) → API → Response
                      ↓
                     Worker → Send Email (2s, who cares)
```

```typescript
// Don't make the user wait for slow operations
app.post('/order', async (req, res) => {
  // Critical path — must be fast
  const order = await createOrder(req.body);
  
  // Non-critical — offload to queue
  await messageQueue.publish('send-confirmation-email', {
    orderId: order.id,
    email: req.body.email
  });
  
  res.json({ orderId: order.id }); // User gets response in 50ms
});
```

### 5. Parallelization

```typescript
// ❌ Sequential — adds latencies
const user = await getUser(userId);        // 20ms
const orders = await getOrders(userId);    // 30ms
const preferences = await getPrefs(userId); // 10ms
// Total: 60ms

// ✅ Parallel — max latency wins
const [user, orders, preferences] = await Promise.all([
  getUser(userId),        // 20ms
  getOrders(userId),      // 30ms
  getPrefs(userId)        // 10ms
]);
// Total: 30ms (the slowest one)
```

### 6. Pagination and Limits

```typescript
// ❌ Return everything — 10,000 items = 500ms
app.get('/orders', async (req, res) => {
  const orders = await db.query('SELECT * FROM orders');
  res.json(orders);
});

// ✅ Paginated — 50 items = 15ms
app.get('/orders', async (req, res) => {
  const limit = Math.min(parseInt(req.query.limit) || 20, 100);
  const offset = parseInt(req.query.offset) || 0;
  
  const [orders, count] = await Promise.all([
    db.query('SELECT * FROM orders LIMIT ? OFFSET ?', [limit, offset]),
    db.query('SELECT COUNT(*) FROM orders')
  ]);
  
  res.json({
    data: orders,
    pagination: { limit, offset, total: count[0].total }
  });
});
```

### 7. CDN and Edge Computing

```
User in Tokyo → Origin in Virginia:  250ms
User in Tokyo → CDN in Tokyo:        20ms
```

Move computation to the edge:
- **Cloudflare Workers** — run code at 300+ edge locations
- **AWS Lambda@Edge / CloudFront Functions** — edge compute
- **Vercel Edge Network** — SSR at the edge

### 8. Read Replicas

```
Write → Primary DB (synchronous, durable)
Read  → Replica 1 / Replica 2 / Replica 3 (load balanced)
```

```typescript
// Route reads to replicas, writes to primary
const readPool = new Pool({ host: 'replica.internal', ... });
const writePool = new Pool({ host: 'primary.internal', ... });

async function getUser(id: string) {
  return readPool.query('SELECT * FROM users WHERE id = ?', [id]);
}

async function updateUser(id: string, data: UserData) {
  return writePool.query('UPDATE users SET ... WHERE id = ?', [id, ...]);
}
```

### 9. Graceful Degradation

```typescript
// If recommendations are slow, show trending instead
async function getRecommendations(userId: string) {
  try {
    return await Promise.race([
      mlService.getPersonalized(userId),
      new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Timeout')), 30)
      )
    ]);
  } catch {
    // Fallback: show trending items (cached, always fast)
    return cache.get('trending-items');
  }
}
```

---

## 🕵️ How to Detect Latency Problems During Design

### The Design Interview Checklist

When someone presents a system design, ask these questions to uncover latency issues:

#### Question 1: "What is the p99 latency requirement?"
- If they say "fast" — they haven't thought about it.
- If they say "200ms" — good, now decompose it.
- If they say "< 100ms" — challenge accepted, but it will be hard.

#### Question 2: "Walk me through the critical path. How many network hops?"
```
❌ Bad answer: "The API calls the database."
✅ Good answer: "API Gateway (5ms) → Auth Service (10ms) → 
   Load Balancer (1ms) → Service A (20ms) → Cache (0.5ms, 95% hit rate) → 
   Database (15ms on miss) → back through the stack. Total: ~52ms best case, ~150ms worst case."
```

#### Question 3: "What happens on cache miss?"
- Cache miss should not be 10x slower than cache hit
- If cache miss = DB query + external API + serialization = 500ms, your budget is blown

#### Question 4: "Are there any synchronous cross-region calls?"
- Cross-region RTT: 50-150ms
- Two cross-region calls = 100-300ms = your entire budget

#### Question 5: "How many database queries per request?"
- 1 query × 10ms = fine
- 10 queries × 10ms = 100ms = half your budget
- N+1 query pattern = instant red flag

#### Question 6: "What is the payload size?"
- 1KB JSON = ~1ms to serialize + transfer
- 1MB JSON = ~100ms+ — you've blown the budget before any logic

#### Question 7: "What happens when a downstream service is slow?"
- No timeout = your service waits forever
- No circuit breaker = cascading failure
- No fallback = guaranteed p99 violation

#### Question 8: "Is there a cold start penalty?"
- Serverless functions: 100ms-5s cold start
- JVM applications: 5-30s startup
- Container startup: 1-10s

#### Question 9: "Are you using pagination?"
- `SELECT * FROM large_table` = latency death
- Every list endpoint MUST have pagination

#### Question 10: "What is the GC/memory pressure?"
- Node.js: GC pause can be 10-100ms on large heaps
- JVM: Full GC can be seconds
- Go: GC is concurrent but still adds latency

### Red Flags in System Design (Latency Edition)

| Red Flag | Why It's Bad | Better Alternative |
|----------|-------------|-------------------|
| No p99 target defined | No way to know if design is good | Define SLO before architecture |
| More than 3 synchronous service calls | Compounding latency | Parallelize, use async, consolidate |
| Cross-region sync calls | 50-150ms per call | Async replication, edge compute |
| No caching strategy | Every request hits DB | Multi-tier caching |
| No timeouts configured | One slow service kills everything | Aggressive timeouts + circuit breakers |
| N+1 queries | Linear latency growth | JOINs, batch queries, DataLoader |
| Full table scans | O(n) latency | Indexes, query optimization |
| Unbounded payload sizes | Serialization + network blowup | Pagination, streaming |
| Synchronous email/SMS/push | 100ms-5s latency | Async queue |
| No read replicas | All reads compete with writes | Read replicas, CQRS |

---

## ⚠️ Pitfalls & Anti-patterns

### 1. Optimizing Average Instead of p99

```typescript
// ❌ "Our average is 50ms, we're fine!"
// 99 requests: 10ms each
// 1 request:  5,000ms (cache miss + DB scan + GC)
// Average: (99×10 + 5000) / 100 = 59.9ms
// p99: 5,000ms ❌ USER EXPERIENCE IS BROKEN
```

### 2. Ignoring the Tail Latency Amplification

```
If Service A calls Service B and Service C in parallel:
  Service B p99 = 50ms
  Service C p99 = 50ms
  
  Parallel p99 ≈ max(50, 50) = 50ms  ✅

If Service A calls Service B and Service C sequentially:
  Service B p99 = 50ms
  Service C p99 = 50ms
  
  Sequential p99 ≈ 50 + 50 = 100ms  ❌
  
  AND if A calls 10 services sequentially:
  p99 ≈ 50 × 10 = 500ms  💀
```

### 3. Setting the Same Timeout for Everything

```typescript
// ❌ One timeout to rule them all
const TIMEOUT = 5000; // 5 seconds

// ✅ Different timeouts per dependency
const TIMEOUTS = {
  cache: 10,      // Cache should be FAST
  database: 50,   // DB should be reasonably fast
  externalApi: 100, // External APIs are unpredictable
  mlService: 200   // ML inference is heavy
};
```

### 4. Not Accounting for Retry Storms

```
Service A calls Service B. B is slow (500ms).
A retries 3 times with 100ms timeout.

Result: A makes 3 requests to B, all fail, A gives up.
But B is now handling 3x the load, making it SLOWER.
Other services start retrying B.
→ Cascading failure. Total system death.
```

**Solutions:**
- Exponential backoff with jitter
- Circuit breakers
- Bulkheads (isolate failing services)

### 5. Forgetting Client-Side Latency

```
Server response time: 100ms
But:
  DNS lookup:     50ms
  TCP handshake:  30ms
  TLS handshake:  100ms (or 0 with TLSv1.3 + 0-RTT)
  Download 2MB:   500ms
  Parse JSON:     50ms
  Render DOM:     100ms
  
  Total user experience: 930ms ❌
```

The server being fast doesn't matter if the client experience is slow.

---

## 🔗 Deep Connections

- **Parent Topic:** [[System Design]], [[Performance Engineering]], [[SRE]]
- **Related Concepts:**
  - [[Caching Strategies]] — the #1 latency optimization tool
  - [[Circuit Breaker Pattern]] — fail fast to protect the budget
  - [[Load Balancing]] — distribute load, reduce per-instance latency
  - [[Database Indexing]] — query optimization fundamentals
  - [[Connection Pooling]] — eliminate connection setup overhead
  - [[CQRS]] — separate read and write paths for optimization
  - [[Event-Driven Architecture]] — async processing for non-critical paths
  - [[CDN]] — edge caching and compute
  - [[Distributed Tracing]] — observe where time goes
  - [[Rate Limiting]] — prevent overload that causes latency spikes
  - [[Graceful Degradation]] — maintain SLAs when dependencies fail
  - [[Tail Latency]] — understanding and mitigating worst-case scenarios
- **Prerequisites:**
  - Understanding of HTTP, TCP, TLS
  - Database query basics
  - Basic statistics (percentiles, distributions)
- **Next Steps:**
  - [[Capacity Planning]] — how much infrastructure do you need?
  - [[Load Testing]] — verify your latency budget under real load
  - [[Chaos Engineering]] — break things to find latency surprises
  - [[Edge Computing]] — push computation closer to users

---

## ❓ Active Recall Questions

> [!faq]- What is the difference between average latency and p99 latency, and why does it matter?
> Average hides outliers. One 5-second request and ninety-nine 10ms requests average to 60ms — looking "fine" while 1% of users have a terrible experience. p99 captures the worst-case normal experience and is the standard for SLOs.

> [!faq]- Why is there a 300,000x difference between L1 cache and cross-Atlantic network latency?
> L1 cache is on the CPU die — electrical signals travel nanometers. Cross-Atlantic network involves fiber optics, routers, switches, and the speed of light through glass. Every network call is astronomically more expensive than local computation.

> [!faq]- What is "tail latency amplification" and how does it affect sequential vs parallel service calls?
> Tail latency amplification is when combining multiple services makes the overall p99 much worse than individual p99s. In parallel, p99 ≈ max(individual p99s). In sequential, p99 ≈ sum(individual p99s). Calling 10 services sequentially with 50ms p99 each gives ~500ms total p99.

> [!faq]- Why should you always include a contingency buffer in your latency budget?
> Because things go wrong: cache invalidates, GC pauses, network hiccups, deployment glitches, thundering herds. A 10% buffer absorbs normal variance. Without it, your p99 is guaranteed to fail under real-world conditions.

> [!faq]- What is the N+1 query problem and why is it a latency budget killer?
> The N+1 problem is fetching a list of N items, then making N additional queries for each item's details. If each query is 5ms, 100 items = 500ms. It turns an O(1) operation into O(n), linearly destroying your latency budget as data grows.

> [!faq]- How does caching improve latency, and what are the trade-offs?
> Caching replaces slow operations (DB queries, network calls) with fast memory lookups. Trade-offs: stale data risk, cache invalidation complexity, memory cost, cold start penalty on cache miss. A cache hit might be 0.5ms vs 20ms for a DB query — a 40x improvement.

> [!faq]- What is graceful degradation in the context of latency budgets?
> When a dependency is slow or failing, returning a degraded but fast response instead of waiting or failing entirely. Example: if personalized recommendations take >30ms, show trending items from cache instead.

> [!faq]- Why are cross-region synchronous calls dangerous for latency?
> Cross-region RTT is typically 50-150ms. Two such calls can consume your entire 200ms budget before any application logic runs. They should be async, or you should use regional replication.

> [!faq]- What is a circuit breaker and how does it protect latency?
> A circuit breaker monitors failures and, after a threshold, "opens" to fail fast for a period instead of waiting for timeouts. This prevents one slow service from consuming all your budget and causing cascading failures.

> [!faq]- What questions should you ask during a system design interview to uncover latency problems?
> 1) What is the p99 target? 2) How many network hops in the critical path? 3) What happens on cache miss? 4) Any cross-region sync calls? 5) How many DB queries per request? 6) What is the payload size? 7) What happens when downstream is slow? 8) Is there cold start penalty? 9) Is there pagination? 10) What is the GC/memory pressure?

> [!faq]- Why should different dependencies have different timeout values?
> Because a cache should respond in <10ms, a database in <50ms, and an external API might legitimately take 100ms. Using one timeout for everything means either your cache waits too long (wasting budget) or your external API gets cut off prematurely.

> [!faq]- What is the difference between connection pooling and creating new connections per request?
> New connections per request pay the TCP + TLS handshake cost every time (10-100ms). Connection pooling reuses established connections, reducing per-request overhead to ~1ms. Without pooling, your latency budget is consumed by connection setup before any real work happens.

---

## 📝 References & Further Reading

- [Google SRE Book — Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)
- ["Designing Data-Intensive Applications"](https://dataintensive.net/) — Martin Kleppmann, Chapter 1 (Reliable, Scalable, Maintainable)
- [Jeff Dean's "Numbers Everyone Should Know"](https://colin-scott.github.io/personal_website/research/interactive_latency.html)
- [AWS Well-Architected Framework — Performance Efficiency](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/welcome.html)
- ["Latency Numbers Every Programmer Should Know"](https://gist.github.com/jboner/2841832)
- ["The Tail at Scale"](https://research.google/pubs/pub40801/) — Jeff Dean, Luiz André Barroso (Google)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- ["Why Products Built for the Enterprise Must Be Reinvented for the End User"](https://a16z.com/2012/06/15/why-consumer-products-always-win/) — A16Z (latency as competitive advantage)
- [[Caching Strategies]] — next topic to master
- [[Circuit Breaker Pattern]] — essential for latency protection
- [[Distributed Tracing]] — observe where your milliseconds go
