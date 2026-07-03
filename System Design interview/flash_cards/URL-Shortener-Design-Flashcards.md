#flashcards/system-design

# 🎴 URL Shortener System Design Flashcards (Senior Engineering)

In a **URL Shortener**, what is the primary Trade-off of using a **301 (Permanent)** vs a **302 (Found/Temporary)** redirect for performance and analytics?
?
- **301 (Performance):** Browser caches the result, skipping your server for subsequent visits. Reduces load but loses almost all click analytics after the first hit.
- **302 (Analytics):** Browser always hits your server first. Guarantees 100% accurate click tracking at the cost of higher latency (one extra hop each time).
<!--SR:!2026-04-29,1,210-->

---

In **URL Shortening**, how many characters are required for a Base-62 "short URL" slug to safely accommodate **365 billion** records?
?
**7 characters.**
$62^7 \approx 3.5$ trillion possibilities. This provides a safe buffer (nearly 10x) for 365 billion records over a 10-year lifespan.

---

In **URL Shortening**, what is the core architectural advantage of **Base-62 Conversion** over **Hash-based Generation**?
?
**Collision Management.**
- **Base-62:** Maps a unique ID from a distributed generator (e.g., Snowflake) to a string. No collision checks needed.
- **Hash:** Maps high-entropy content to a truncated string. Requires a database lookup on every "write" to detect collisions and recursive re-hashing if one occurs.
<!--SR:!2026-04-24,1,230-->

---

In a **URL Shortener**, why is using ==incremental IDs== (e.g., `1001`, `1002`, `1003`) as the source for Base-62 conversion considered a "Security Anti-pattern"?
?
**Predictable Enumeration.**
Competitors or attackers can easily crawl your entire database of short links by iterating through the ID sequence (e.g., `tinyurl.com/a`, `tinyurl.com/b`, etc.), exposing sensitive or private hidden links.

---

In a **URL Shortener**, a ==Bloom Filter== can be used to optimize write operations by quickly checking if a "long URL" has already been shortened without querying the database disk.
?
True. While it may have false positives (indicating it's in the DB when it's not), its **zero false negatives** ensure every *new* link bypasses the DB lookup for a hit, saving expensive I/O.
<!--SR:!2026-04-26,3,250-->

---

In a **URL Shortener**, what is the "High-Read / Low-Write" ratio and how does it influence data modeling?
?
Typically **10:1 to 100:1**. Since redirects happen far more often than link creations, the system must be optimized for **read-heavy performance** using distributed caching (Redis/Memcached) and read-replicas in the database layer.
<!--SR:!2026-04-29,1,210-->

---

In a **URL Shortener**, when using Hash-based generation (e.g., SHA-256 truncated to 7 chars), what is the "recursive collision resolution" flow?
?
1. Hash total `longURL`.
2. Take first 7 chars.
3. Check `shortURL` in DB.
4. **If exists:** Append a predefined salt/suffix to the `longURL` and **repeat step 1**.
5. **If unique:** Save to DB and return.
<!--SR:!2026-04-24,1,230-->

---

How can ==Snowflake IDs== or distributed ID generators improve a URL shortener using Base-62 transformation?
?
They provide **globally unique, non-colliding, and roughly time-ordered IDs** in a distributed environment without needing a central bottleneck (like a single SQL auto-increment), allowing the shortener to scale horizontally.
<!--SR:!2026-04-30,2,230-->

---

In a **URL Shortener**, why is an ==LRU (Least Recently Used) cache== particularly effective on the edge nodes or application servers?
?
**Because of the "80/20 Rule."**
Usually, a small fraction of links (viral social media posts) drive the vast majority of traffic. Caching these "hot" links in memory dramatically reduces the load on the DB for over 90% of requests.
<!--SR:!2026-04-24,1,230-->

---

In a **URL Shortener**, what is the "quirky" benefit of ==shuffling the Base-62 alphabet== (e.g., `[A-Z, a-z, 0-9]` instead of standard order) before mapping IDs?
?
**Obfuscation.**
Even with sequential IDs, a shuffled alphabet makes the resulting short URLs (e.g., `a7X1`, `b2Y9`) appear random to the end-user. This prevents competitors from gauging system growth by looking at sequential slugs.
<!--SR:!2026-04-24,1,230-->

---

In a **URL Shortener**, how should the system handle ==Vanity/Custom URLs== (e.g., `bit.ly/hire-me`) compared to auto-generated ones?
?
Custom URLs must be stored in the same DB but with a flag. They require a separate **uniqueness check** during creation and should be validated against a **reserved keywords list** (e.g., `admin`, `login`, `api`) to prevent phishing or system spoofing.
<!--SR:!2026-04-24,1,230-->

---

What is a robust strategy for ==cleaning up expired links== in a large-scale URL Shortener?
?
Instead of a per-row TTL (which might be hard in some DBs), use a **lazy-deletion or batch-cleanup** process during off-peak hours. Alternatively, check expiration during lookup (lazy-delete) and return a 410 (Gone).
<!--SR:!2026-04-24,1,230-->

---

In a **URL Shortener**, what specific security measure prevents a single user from exhausting the entire Base-62 ID space with a script?
?
**Rate Limiting.**
Implement an IP-based or UserID-based rate limiter (e.g., Token Bucket) on the `POST /api/v1/data/shorten` endpoint to prevent automated "link bombing."
<!--SR:!2026-04-24,1,230-->
