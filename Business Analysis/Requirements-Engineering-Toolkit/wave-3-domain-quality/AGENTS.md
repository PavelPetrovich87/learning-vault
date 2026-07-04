# Wave 3 — Domain & Quality

> Canonical detailed description for Wave 3 of the [[Requirements-Engineering-Roadmap|Requirements Engineering Toolkit]].

**Status:** Not started
**Window:** Week 5-6 (Aug 1 – Aug 14, 2026)
**Budget:** ~12h (5h study + 6h cases + ~2h distillation)

---

## Goal
On exit you can:
- Extract bounded contexts and their relationships from a storm/model.
- Turn vague quality claims into measurable SLOs.
- Reason about consistency / availability / latency trade-offs without hand-waving.

---

## Topics to master
1. DDD strategic: domain vs subdomain vs bounded context — the differences that matter.
2. Subdomain types: core / supporting / generic.
3. Ubiquitous Language: why it matters, how to form it.
4. **Context Mapping**: 9 relationship types (customer-supplier, conformist, ACL, shared kernel, published language, open-host service, partners, big ball of mud, separated ways).
5. Domain Storytelling (Eber/Widmann) — elicitation technique complementing Event Storming.
6. **ISO/IEC 25010** quality model: 8 characteristics.
7. SMART-ifying NFRs: "fast" → p99 < 200ms under 1000 RPS.
8. SLO/SLI/SLA: engineering language for NFRs.
9. Trade-off thinking: consistency / availability / latency / cost.

---

## Study materials (~5h core)

| Source | What to read | Time | Mark |
|---|---|---|---|
| Vaughn Vernon, *Implementing DDD* | Ch. 2 "Domains, Subdomains, and Bounded Contexts" + Ch. 3 "Context Mapping" | 2.5h | CANON |
| Eric Evans, *DDD* | Ch. 14 "Maintaining Model Integrity" (the context mapping origin) | 1.5h | CANON |
| Google SRE Book, Ch. 4 "SLO/SLI" | full chapter, free online | 1h | CANON |

**Reference:**
- ISO/IEC 25010 specification (official quality model) — the 8 characteristics.
- Nick Tune / Scott Wlaschin — Context Mapping patterns blog posts.

**Optional:**
- Brandolini, *Introducing Event Storming* — Design Level storming part.
- domainstorytelling.org — Domain Storytelling primer.

---

## Preparation — Subagent `The Tech Lead` (builds BEFORE cases)

The subagent is **infrastructure** for the learning process, not a learning product. Build it first, improve it iteratively as cases expose weaknesses.

**Inputs:**
- NFR set + architecture sketch.
- Context map.
- Project constraints (team size, timeline, budget).

**Behavior contract (hard mode):**
- Challenges NFR targets: "p99 < 200ms — at what load? at what cost?"
- Questions architecture choices: "Why microservices here? What's the team size?"
- Pushes back on missing trade-offs.
- Demands measurement strategy, not just claims.
- Cites realistic constraints: team size, time-to-market, operational burden.

**Acceptance criteria — pass when:**
- [ ] Challenges ≥ 3 NFR targets per review.
- [ ] Surfaces ≥ 1 cost / operational concern per architecture.
- [ ] Asks "how will you measure this?" for every claim.

**File output:**
```
work/ba-ai-toolkit/subagents/the-tech-lead/
  persona.md
  checklists/
    nfr-challenge-questions.md
    architecture-fit-questions.md
  examples/
    review-smart-home.md
    review-monolith-migration.md
    review-food-delivery.md
  README.md
```

---

## Cases (3 × ~2h = 6h)
1. **Smart home / IoT — many contexts, few people.** Device management, user, scene automation, billing, telemetry. Trains: identifying contexts at scale; shared kernel vs ACL decisions.
2. **Legacy monolith migration — brownfield DDD.** Extract contexts from an existing e-commerce monolith. Trains: recognizing "big ball of mud" and seams for extraction.
3. **High-load real-time (food delivery tracking).** Trains: NFR elicitation under stress; latency / throughput / consistency trade-offs.

---

## Distillation contract — Wave 3

### Skill `context-mapper`

**Inputs:**
- Domain events from `event-storming-facilitator`.
- Project brief.
- Organization structure (teams, existing systems).

**Outputs:**
- Bounded context inventory with names + ubiquitous language hints.
- Context map (visual + table) with relationship types.
- Risk flags: where ACL needed, where shared kernel is tempting but dangerous.

**Acceptance criteria — pass when:**
- [ ] Identifies ≥ 4 bounded contexts for non-trivial domain.
- [ ] Each relationship labeled with one of the 9 patterns + rationale.
- [ ] Surfaces ≥ 1 "big ball of mud" risk area.
- [ ] Names contexts using ubiquitous language, not technical jargon ("Ordering" not "OrderService").

**File output:**
```
work/ba-ai-toolkit/skills/context-mapper/
  SKILL.md
  assets/
    context-map-templates.md     # the 9 patterns with decision guide
    ubiq-language-template.md
  examples/
    smart-home-context-map.md
    ecommerce-monolith-map.md
    food-delivery-context-map.md
  README.md
```

### Skill `nfr-elicitor`

**Inputs:**
- Stakeholder statements ("the system should be fast", "secure", "always available").
- Architecture summary or constraints.
- Domain context (real-time? batch? regulated?).

**Outputs:**
- NFR table classified by ISO 25010 category.
- Each NFR as: attribute / stimulus / response measure / target / trade-off note.
- SLO/SLI candidates for engineering teams.

**Acceptance criteria — pass when:**
- [ ] Maps every vague quality claim to ≥ 1 ISO 25010 category.
- [ ] Every NFR has a measurable target (no "fast" without a latency number).
- [ ] Flags ≥ 2 trade-offs per non-trivial architecture (e.g., consistency vs availability).
- [ ] Distinguishes hard constraints (regulatory) from soft targets (UX).

**File output:**
```
work/ba-ai-toolkit/skills/nfr-elicitor/
  SKILL.md
  assets/
    iso-25010-cheatsheet.md
    slo-template.md
  examples/
    food-delivery-nfrs.md
    iot-nfrs.md
    ecommerce-nfrs.md
  README.md
```

---

## Wave 3 progress
- [ ] Study materials read
- [ ] **Subagent: The Tech Lead (preparation, before cases)**
- [ ] Case 7: Smart home contexts
- [ ] Case 8: Legacy monolith migration
- [ ] Case 9: High-load real-time NFRs
- [ ] Skill: `context-mapper`
- [ ] Skill: `nfr-elicitor`
