---
name: review-diagram
description: Review a software architecture (or any technical) diagram for visual quality and legibility, and produce a structured, actionable feedback report. Use this skill whenever the user shares a diagram — as an image file (PNG/JPG/screenshot), a Mermaid/PlantUML/C4-PlantUML source block, a `.drawio`/`.xml` file, or a live URL — and asks to review, critique, check, audit, improve, or give feedback on it, OR when the user is about to draw a diagram and wants to know what visual-quality rules to follow. Also use it proactively when a user shares a diagram and asks "is this any good?" or "what's wrong with this diagram?". This skill ONLY checks HOW the diagram is drawn (visual quality), never WHAT it shows (content/audience/abstraction-level decisions are a separate concern).
---

# Review Diagram

A skill that reviews any technical diagram against **7 visual-quality checks** and produces a structured, actionable feedback report. The checks are deliberately about *how* the diagram is drawn — not *what* it shows — so they apply to any diagram regardless of its subject matter, audience, or abstraction level.

## Scope — what this skill does and does not check

This boundary is critical. Confusing the two produces useless feedback.

| ✅ This skill checks (HOW it's drawn) | ❌ This skill does NOT check (WHAT it shows) |
|---|---|
| Legend present? Colors meaningful? Layout readable? Lines tangled? Labels short? Consistent? Accessible? | Is the audience right? Are abstraction levels mixed? Is the notation appropriate? Is the content complete? |

The "what we draw" checks (audience, abstraction-level mixing, notation choice, content scope) are a **separate skill**. Do not comment on them here. If the diagram has a content problem, note it in the report's "Out of scope" section and stop — do not fold it into the visual review.

## The two input modes

The skill must first detect which of two modes the diagram arrived in, then apply the same 7 checks adapted to that mode.

### Mode A — Image input

The diagram is a raster/vector image: a `.png`/`.jpg`/`.jpeg` file, a screenshot pasted inline, a photo of a whiteboard, or an exported diagram.

- **How to analyze:** use vision. Look at the rendered diagram directly.
- **Strengths:** catches visual problems a text parser cannot — color usage, actual line crossings, spatial spacing, real decoration (3D/shadows/gradients), perceived reading order.
- **Limitations:** cannot see whether a legend's symbols *match* the diagram's actual symbols if they're far apart, and cannot parse the underlying connector semantics (e.g., cannot verify two "async" arrows are truly dashed vs. just thin).
- **Action:** Read the image, then run the 7 checks against what you observe visually.

### Mode B — Text/source input

The diagram is a text source: a Mermaid block (`graph TD; A-->B`), PlantUML/C4-PlantUML, Structurizr DSL, or a `.drawio`/`.xml` file (which is XML text).

- **How to analyze:** read the source structurally. Parse node styles, edge declarations, `style` directives, `classDef` blocks, label text, and layout direction.
- **Strengths:** catches precise inconsistencies (e.g., two connectors declared as different styles but meaning the same thing), missing `classDef`/legend definitions, label text length (measurable), and explicit layout direction.
- **Limitations:** cannot see how it actually *renders* — a Mermaid `LR` direction might still produce a confusing layout, and color choices are only names/hex codes without visual context.
- **Action:** Read the source, extract the relevant structural facts (styles, labels, edges, direction), then run the 7 checks against those facts.

### Auto-detection

Decide the mode from what the user provided:

- File path ending in `.png`/`.jpg`/`.jpeg`/`.svg` → **Mode A (image)**. Read it as an image.
- File path ending in `.mmd`/`.puml`/`.c4`/`.dsl`/`.drawio`/`.xml`, OR a fenced code block containing `graph`/`flowchart`/`sequenceDiagram`/`@startuml`/`<mxGraphModel>`/`C4Context` → **Mode B (text)**. Read it as text.
- Pasted image / screenshot → **Mode A**.
- Unclear? Ask the user briefly, or default to whichever mode the content matches.

## The 7 checks

Each check has a **positive form** (what good looks like) and a **negative form** (the anti-pattern). When reviewing, look for the negative form; when the diagram passes, affirm the positive. Every check produces a verdict: `pass` / `fail` / `warning` / `n/a`.

The checks are grouped into three blocks. Walk them in order.

---

### Block B — Notation & Standards

#### Check 1 — Legend present and adequate

- **Look for:** Is there a legend or key that decodes every symbol shape, color, line style (solid/dashed/dotted), and non-obvious acronym that carries meaning in this diagram?
- **Fail (anti-pattern):** *Assuming people know what symbols and acronyms mean.* Custom shapes with no key; TLAs ("IdP", "MQ") never expanded; colors/styles whose meaning is "obvious" only to the author.
- **Pass (practice):** A legend is present in an unobtrusive corner, or the diagram uses a fully standard notation (UML, BPMN, C4) whose vocabulary needs no legend. Every meaning-carrying visual element is decodable by a first-time reader.
- **Verdict `n/a`:** the diagram uses only generic boxes + plain-language labels with no color/line-style semantics, so there is nothing to decode.
- **Mode A note:** look at the rendered diagram for a legend box and for any symbols/colors/acronyms in the body that aren't defined there.
- **Mode B note:** look for a legend node (`subgraph Legend` / a styled key block) or explicit comments; flag any `classDef`/color/line-style that has no documented meaning.

#### Check 2 — Consistency of symbols, styles, and naming

- **Look for:** Does the same kind of thing always use the same shape, color, and line style? Is naming consistent (same entity spelled the same way throughout)? Are the conventions the same across multiple diagrams if a set was provided?
- **Fail (anti-pattern):** *Inconsistent naming, symbols, or styles.* `OrderService` on one node and `order-svc` on another; a rounded blue box meaning "container" here but "external system" there; solid and dashed arrows used interchangeably for the same relationship.
- **Pass (practice):** One shape = one meaning; one color = one semantic; one naming style throughout. Consistency holds within the diagram and, if a set was given, across the set.
- **Mode B note:** this is where text parsing is strongest — you can mechanically list every node's style and every label's casing and find mismatches.

---

### Block C — Visual Clarity & Accessibility

#### Check 3 — Colors carry meaning; no purposeless decoration

- **Look for:** Does every color encode a meaning, or are colors used decoratively? Is there ornament that doesn't communicate — 3D effects, gradient fills, drop shadows, excessive borders, boxes around trivial things?
- **Fail (anti-pattern):** *Using too many colors with no purpose* — a rainbow palette where every box is a different hue "to make it pretty," plus heavy decoration (gradients, 3D, shadows) that adds no information.
- **Pass (practice):** Every color earns its place by encoding meaning (e.g., green = external system). Decoration is stripped. Maximizing the data-ink ratio: ink encodes meaning, not ornament.
- **Test:** *If I removed this color/decoration, would the reader understand less?* If no, it should go.
- **Mode A note:** this is the primary mode for this check — decoration is a visual phenomenon.
- **Mode B note:** look for many distinct `fill`/`stroke` colors assigned without a `classDef` grouping them by meaning; flag gradients/shadows in style strings.

#### Check 4 — Differentiation is not color-only (accessibility)

- **Look for:** Whenever color distinguishes things (red vs green status, blue vs red internal/external), is there a **second channel** — a label, a shape difference, a pattern (dashed/solid), or an icon — conveying the same distinction?
- **Fail (anti-pattern):** *Relying only on color to differentiate elements.* Invisible to ~8% of men (color blindness) and lost in grayscale printouts/low-contrast screens.
- **Pass (practice):** Color is always paired with a second channel. `color + icon` is accessible; `color alone` is not.
- **Mode A note:** look at color-coded distinctions and verify each has a non-color backup.
- **Mode B note:** where two `classDef`s differ only by `fill`/`stroke` color and the nodes carry no distinguishing label/icon/pattern, flag it.

#### Check 5 — Labels are short and well-placed

- **Look for:** Are element labels short nouns (or noun phrases) and connector labels short verb-phrases? Or are there full sentences and paragraphs stuffed into boxes and onto arrows? Are labels placed close to the element they describe (not floating midway across the canvas)?
- **Fail (anti-pattern):** *Long sentences and labels.* Boxes that grow to hold a wall of text; arrows with full clauses ("when the user submits the form and validation passes, the service then calls…"); labels floating far from their element.
- **Pass (practice):** Nouns for elements ("Order Service"), short verb-phrases for connectors ("publishes order.created", "HTTPS/JSON"). Long explanations move to side notes, legends, or companion text. A diagram complements prose; it doesn't replace it.
- **Mode B note:** label length is measurable — flag any label over ~6 words or containing a full clause.

---

### Block D — Layout & Connections

#### Check 6 — Connections are clear; no spiderweb

- **Look for:** Can you trace any single path from source to target without losing the thread? Are there many crossing/overlapping lines? Are parallel edges spread apart, or stacked on top of each other?
- **Fail (anti-pattern):** *Crisscrossing lines everywhere (the spiderweb effect)* — a rat's nest where relationships are unreadable. Often a smell: either too much on one canvas, or the topology is genuinely over-coupled.
- **Pass (practice):** Orthogonal (right-angle) routing for structural diagrams; parallel edges spread; routing through hubs/gateways rather than every dependency drawn across the canvas; minimal crossings.
- **Mode A note:** visually trace a few representative paths; count crossings.
- **Mode B note:** a high edge-to-node ratio or many nodes with degree ≥ 4 is a structural warning sign of a potential spiderweb; note it and let the rendered output confirm.

#### Check 7 — Layout follows natural reading patterns

- **Look for:** Does the primary flow follow the reading direction (left-to-right, top-to-bottom for Western readers)? Are elements aligned on a grid, with even spacing? Are related elements grouped spatially (proximity)? Or does the eye zig-zag randomly and is the canvas lopsided (crammed corner, empty elsewhere)?
- **Fail (anti-pattern):** *Poor layout and uneven spacing* — boxes crammed in one corner, misaligned, arbitrary gaps, no visual grouping, primary flow zig-zagging.
- **Pass (practice):** Primary flow left-to-right or top-to-bottom; aligned to a grid; even spacing; related elements clustered (Gestalt proximity).
- **Mode A note:** this is primarily a visual check — assess alignment, spacing, and flow direction from the rendered output.
- **Mode B note:** check the declared direction (`graph LR`/`TD` etc.) matches the natural flow of the described process; note that text mode cannot verify actual rendered spacing, so mark spacing concerns as "verify in render."

---

## Output format — the review report

Produce the report in this exact structure. Be concrete and specific — point to *which* element has the problem, not just that a problem exists.

```
# Diagram Review

**Input mode:** <Image | Text source>
**Source:** <filename, "pasted image", "Mermaid block", etc.>
**Overall:** <one-line summary — e.g., "Mostly clear; two accessibility issues and a tangled lower-right corner.">

## Verdict summary
| # | Check | Verdict |
|---|-------|---------|
| 1 | Legend present and adequate | pass / fail / warning / n/a |
| 2 | Consistency of symbols, styles, naming | … |
| 3 | Colors carry meaning; no purposeless decoration | … |
| 4 | Differentiation not color-only (accessibility) | … |
| 5 | Labels short and well-placed | … |
| 6 | Connections clear; no spiderweb | … |
| 7 | Layout follows reading patterns | … |

## Findings

### Check N — <name> — <verdict>
**What I see:** <concrete observation, pointing at specific elements>
**Why it matters:** <one sentence on the harm>
**Fix:** <specific, actionable suggestion>

(repeat for each non-pass check; for passes, a one-line affirmation is enough)

## Out of scope (content-level, not checked here)
<If you noticed a content problem — wrong audience, mixed abstraction levels, wrong notation choice — note it here as a pointer to the separate "what we draw" review, then stop. Do not mix it into the visual findings above.>
```

### Verdict definitions

- **pass** — the check is satisfied; the diagram does this well.
- **fail** — the anti-pattern is clearly present; this needs fixing.
- **warning** — a borderline case or a minor instance; worth attention but not blocking.
- **n/a** — the check doesn't apply to this diagram (e.g., no color semantics → legend check is n/a).

### Tone and principles

- **Be specific.** "The legend is missing" is weak; "the four box colors (red/blue/green/yellow) have no legend explaining what each represents" is useful.
- **Explain the why.** Don't just list rules — say what harm the anti-pattern causes (accessibility exclusion, misreading, cognitive load) so the author understands the reason.
- **Give the fix, not just the diagnosis.** Every non-pass finding should end with a concrete suggestion.
- **Affirm what works.** Don't only report failures; a one-line affirmation on passing checks reinforces good practice.
- **Stay in scope.** Resist the urge to comment on content/audience/notation-choice — that's the other skill's job. Park those observations in "Out of scope."
- **Adapt to mode.** For text input, be honest about what you can and can't see (e.g., "I can't verify rendered spacing from the source — confirm Check 7 visually").

## Proactive use — "about to be created" diagrams

When the user is *about to draw* a diagram and wants guidance (not a review of an existing one), don't run the checks as a verdict — instead, present the 7 checks as a **pre-flight checklist** the author should satisfy while drawing, phrased in the positive form. The same 7 rules apply; only the framing changes from "review" to "guide."

## Source of these checks

The 7 checks are distilled from two concept notes in this vault:
- `Modeling Software Architecture/concept_notes/best-practices/Best Practices for Diagrams.md`
- `Modeling Software Architecture/concept_notes/antipatterns/Anti-Patterns for Diagrams.md`

Each check is the positive/negative pair from those notes. The "what we draw" checks (audience tailoring, abstraction-level separation, notation choice) that were intentionally excluded from this skill live in those same notes and belong to a separate content-review skill.
