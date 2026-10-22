# System Design Udemy Course — Agent Guidelines

This folder contains concepts, notes, and flashcards derived from the Udemy System Design course.

## Folder Purpose

Store structured learning material tied to the course: key concepts, architectural patterns, trade-offs, case studies, and extracted flashcards.

## Subfolder Structure

Course material is grouped by topic into subfolders. Each subfolder contains research notes and links out to stabilized concept notes.

Current subfolders:

- `networking/` — networking fundamentals covered by the course (protocols, DNS, TCP/UDP, load balancing, CDNs, etc.).
  - `networking/ip/` — IP addresses: IPv4, IPv6, public/private IPs, NAT, CIDR.
  - `networking/dns/` — DNS concepts, resolution process, caching, security, and large-scale DNS strategies.
  - `networking/client_server_model/` — client-server architecture, request-response, stateless vs stateful, communication patterns.
- `protocols/` — communication protocols covered by the course (TCP, UDP, HTTP, WebSocket, gRPC, etc.).
- `web_concepts/` — web fundamentals covered by the course (HTTP statelessness, sessions, cookies, tokens, authentication, security, scalability, CORS).
  - `web_concepts/web_sessions/` — web sessions: why state is needed despite stateless HTTP, state-management techniques (cookies/sessions/tokens), session-based vs token-based authentication, session security (hijacking, CSRF, cookie attributes), and scaling session management (sticky vs distributed, Redis/Memcached, stateless JWT).
  - `web_concepts/serialization/` — serialization: why it matters, common formats (JSON, XML, Protocol Buffers, Avro, BSON), trade-offs (readability vs efficiency vs compatibility), serialization in action (REST/gRPC/SOAP), serialization in caching and data storage (Redis/Memcached, MongoDB/BSON, Avro on Hadoop/Kafka), and deserialization security (RCE, DoS, tampering).
  - `web_concepts/cors/` — CORS and cross-origin web security: why CORS matters (Same-Origin Policy as the browser's default boundary), how CORS works (request/response headers, browser-enforced), preflight requests and CORS headers (when required, the `OPTIONS` flow, server config in Express/Nginx), security risks and common misconfigurations (`*` + credentials, origin reflection, `null` origin, regex bypasses), handling CORS in APIs (REST and GraphQL), and alternatives to CORS (reverse proxy, API Gateway, deprecated JSONP).
- `scalability/` — scalability fundamentals covered by the course: what scalability is, why systems need to scale, types of scalability (vertical vs horizontal), and common scaling challenges (latency, bottlenecks, downtime, cost).
  - `scalability/concept_notes/` — stabilized concept notes for scalability topics (Introduction to Scalability, Scaling Strategies, Load Balancers).
  - `scalability/flash_cards/` — spaced repetition flashcards framed as system-design interview questions.
- `software_architecture_patterns_and_styles/` — architectural patterns and styles (layered, hexagonal/onion, microservices, event-driven, CQRS, monolithic vs distributed, etc.).
  - `software_architecture_patterns_and_styles/concept_notes/` — stabilized concept notes for each pattern/style (Software Architecture, Layered, Microservices, Event-Driven, Factors Influencing Selection).
  - `software_architecture_patterns_and_styles/flash_cards/` — spaced repetition flashcards for each pattern/style.
  - `software_architecture_patterns_and_styles/exercises/` — **interactive course** on selecting between architectures (3 exercises, Easy→Advanced). Each exercise folder has a `description.md` (the scenario) and an `AGENTS.md` (a persona to role-play when opened in a CLI agent). Open the exercise folder as the workspace root to pick up the persona.
  - `software_architecture_patterns_and_styles/monolithic_architecture/` — deep dives on monolith: concept note, flashcards, and 3 interactive exercises (Easy→Advanced).
  - `software_architecture_patterns_and_styles/multi_layered_architecture/` — deep dives on multi-tier variants: two-tier, three-tier, and n-tier, with performance/scalability impacts and mitigations.
  - `software_architecture_patterns_and_styles/microservices_architecture/` — deep dives on microservices: overview, identifying services, structuring, communication, challenges, scaling — plus 3 interactive exercises (Easy→Advanced).
  - `software_architecture_patterns_and_styles/event_driven_architecture/` — deep dives on event-driven architecture: overview, sync vs async, pub/sub vs streaming, key components, challenges, best practices, use cases — plus 3 interactive exercises (Easy→Advanced).

### Interactive exercises convention
Each pattern's `exercises/` folder contains 3 numbered subfolders (`01_Easy_…`, `02_Intermediate_…`, `03_Advanced_…`). Inside each:
- `description.md` — the immutable scenario (type, difficulty, situation, stakeholders, constraints, the ask, hidden traps for the persona, reference answer). The learner does not edit this.
- `AGENTS.md` — the persona the CLI agent adopts when this folder is opened (role, behavior, rubric, opening line). Read automatically when the folder is the workspace root.

To run an exercise: open the specific exercise folder (e.g. `…/monolithic_architecture/exercises/01_Easy_Startup_MVP_Decision/`) in a fresh agent chat. The agent greets you in character and runs the exercise using `description.md` as its hidden reference.

Each topic subfolder contains:
- Research notes at the root of the subfolder
- `concept_notes/` — stabilized concept notes for that topic
- `flash_cards/` — spaced repetition flashcards for that topic

Add new topic subfolders here as new course sections are introduced.

## Note Lifecycle

### Research Notes
- **When to create:** While watching a course section, taking notes on a lecture, or analyzing a course case study.
- **Template:** `Templates/Research-to-Project Template.md`
- **Status:** `researching | analyzing | experimenting | implemented | archived`
- **Rule:** One document per course section or case-study thread. Do not split a single lecture thread into multiple files.

### Concept Notes
- **When to create:** When a course concept stabilizes into a reusable, referenceable idea (e.g., "Consistent Hashing", "Rate Limiting", "CQRS").
- **Template:** `Templates/Technical-Deep-Dive.md`
- **Location:** Create the concept note in the relevant topic subfolder under `system-design-udemy/<topic>/concept_notes/`.
- **Rule:** Link back to the originating research note here.

### Flashcards
- **When to create:** After completing a research note or concept note, extract key facts into flashcards.
- **Location:** `system-design-udemy/<topic>/flash_cards/`
- **Syntax:** Obsidian Spaced Repetition multi-line format:
  ```
  Question text?
  ?
  Answer text.
  #flashcards/system-design
  ```
- **Interview orientation:** This material is preparation for **system-design interviews**. Frame every card as an interview question, not rote recall. Prefer "How would you…", "When would you choose X over Y?", "What's the trade-off?", "What goes wrong if…?" over "What is the definition of…?". Answers should read like how you'd speak them aloud: concise, structured (lead with the trade-off), and concrete (name a real-world example where it helps).
- **Rule:** One card = one concept. Frame questions around problem-solving, architecture, trade-offs, and gotchas.

## Naming Conventions

- Research notes: `{SectionNumber}_{Topic_Name}.md` or `{CourseName}_{Topic_Name}.md`
  - Examples: `01_Introduction_To_System_Design.md`, `Consistent_Hashing_Deep_Dive.md`
- Prefer `Title_Case_Snake_Case.md` for file names.

## Frontmatter

All notes MUST include:

```yaml
---
status: researching | analyzing | brainstorming | implemented | archived
tags: [system-design, udemy, research]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Update `updated:` on every edit.

## Linking

- Link to related concept notes with `[[WikiLinks]]`.
- Link to the source lecture/section if applicable.
- Maintain a "Related Notes" section at the bottom of research notes.

## Anti-Patterns

- Do NOT use this folder for raw brainstorming; use `Заметки на будущее/` instead.
- Do NOT create flashcards in the root of this folder; use the `flash_cards/` subfolder within the relevant topic directory.
- Do NOT move notes out of this folder to "ripen" them. Create new concept notes elsewhere and link back.

---

*Last updated: 2026-07-23*
