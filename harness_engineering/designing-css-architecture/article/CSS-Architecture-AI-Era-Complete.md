# CSS Architecture in the AI Agent Era — Complete Series

> **Source:** Zetsubo (Dev.to), 5-part series on SpiraCSS
> **Core thesis:** CSS architecture should shift from "rules to memorize and follow" to "feedback systems" — machine-verifiable invariants + error messages that tell you *how* to fix it.

---

## Part 1: How Should CSS Architecture Evolve in the Age of AI Coding Agents?

CSS architecture has long been built around human-centered workflows. The naming conventions of BEM/SMACSS and the design philosophy of utility-first both assume authors who memorize rules, make judgment calls, and maintain consistency through review.

AI coding agents (Claude Code, Cursor, Windsurf) are changing that assumption. In the Tailwind ecosystem, `llms.txt` — a file that feeds project rules to AI — has emerged as one approach. But `llms.txt` represents a "teach the rules to AI" approach. The real question: **can we design an architecture that doesn't break down even without that teaching?**

### The Limits of Convention-Based Design

Utility-first is often said to pair well with AI — class names directly represent styles. But structural decisions remain: Should a card component's margin belong to the parent or the child? Should visually identical elements be shared or duplicated?

Convention-based designs (BEM, etc.) share the same weaknesses whether the author is human or AI:
- There's a cost to memorizing conventions (or teaching them to AI)
- Decisions are subjective, and review standards tend to drift
- Consistency erodes as teams grow
- AI accuracy drops as context grows — the same problem humans face in a different form

### From "Rules to Follow" to "Feedback Systems"

TypeScript's type system is a useful reference. You don't memorize type rules — just write code and the compiler tells you "this is wrong" and "fix it like this." Developers follow the feedback, and the code converges to type safety.

**CSS architecture needs the same structure.** Instead of teaching rules upfront, the system should respond to what's written with "what's wrong" and "how to fix it." Humans read error messages and fix issues; AI agents parse lint output and self-correct — a feedback loop where the design converges regardless of who writes the code.

---

## Part 2: CSS Drift — Why Conventions Break Down and How Machine-Verifiable Rules Fix It

### Drift — How Design Breaks Down

CSS design doesn't collapse all at once. Small decision inconsistencies accumulate, and by the time you notice, coherence is lost. This is **"drift"** — specifically:

1. **Component boundary ambiguity:** Where one component ends and another begins differs from person to person
2. **Layout responsibility confusion:** Whether margin belongs to parent or child varies file by file
3. **Naming and structural inconsistency:** Naming conventions and file organization vary across the project

Introducing AI agents doesn't change the structure. Human drift simply becomes AI drift — faster and at scale.

### Why Conventions Can't Prevent Drift

Convention-based designs share a structural weakness:
- **Memorization burden:** The longer the guidelines, the harder accurate application becomes
- **Subjective decisions:** "Where does one component end?" has no objective answer
- **Verification difficulty:** Whether something violates a convention can't be determined mechanically

When all three are present, drift is inevitable.

### Invariants — Eliminating Interpretive Ambiguity

Programming has "invariants" — conditions that must hold at every point in a program. The same idea applies to CSS architecture: eliminate rules whose answers change based on interpretation, adopt only conditions mechanically evaluated as true or false.

| Convention-based (requires interpretation) | Invariant (mechanically verifiable) |
|---|---|
| "Decompose components into appropriate granularity" | "A Block can only contain Blocks or Elements as direct children" |
| "Parents manage layout" | "margin-top can only be specified from the parent's `> .child` selector" |
| "Keep naming consistent" | "Blocks use two-word kebab-case; Elements use a single word" |
| "Distinguish between state and variation" | "Variants use `data-variant`; States use `data-state`" |

### Design Criteria for Invariants

For a rule to function as an invariant:
1. **Binary evaluation:** Violation or compliance is determined unambiguously (no gray zones)
2. **Syntax-verifiable:** Determined through source code syntax analysis alone (no runtime needed)
3. **Locally verifiable:** Determined by looking at the target file alone (no project-wide scanning needed)

> "Isn't this just freezing preferences rather than defining 'correctness'?" — Fair objection. But there's value in fixing design preferences to make all authors converge on the same output — especially when authors include both humans and AI.

---

## Part 3: Architecture Lint — TypeScript-Style Errors That Tell You How to Fix It

### Two Approaches: Upfront Teaching vs. Post-Hoc Feedback

| Approach | How it works | Weakness |
|---|---|---|
| **Upfront teaching** | Guidelines, education, reviews. For AI: instruction files (llms.txt, AGENTS.md) | Memorization cost, subjective decisions, accuracy drops as context grows |
| **Post-hoc feedback** | Verify what's written, return errors with fix instructions | Requires well-designed error messages |

Post-hoc feedback doesn't require the author to know the rules beforehand. Write, see the error, fix it. Starting from zero knowledge, the code converges on architecture compliance.

### Traditional Lint vs. Architecture Lint

- **Traditional CSS lint** checks *what you wrote* — naming patterns, property duplication (surface-level, local)
- **Architecture lint** checks *where you wrote it* — whether the structure follows design rules (structural, global concern)

### Error Message Design Principles

For the feedback loop to function, error messages must:
1. **Be specific:** Not "invalid naming" but "Blocks must use two-word kebab-case (e.g., `hero-container`)"
2. **Include fix instructions:** Not just what's wrong, but how to fix it
3. **Be self-contained:** Understandable without referencing external documentation
4. **Be machine-readable:** Structured so AI can parse and convert to actionable fixes

Example of a feedback-style lint error:
```
"margin-top" is an item property — it must be specified
from the parent Block using the `> .child-name` selector,
not inside the child Block itself.
```

### The Feedback Loop

Whether the author is human or AI, the loop structure is the same:

```
Write → Run lint → Detect errors → Fix → Run lint → ... (until zero errors)
```

The exit condition is identical: **zero errors = architecture-compliant code**, regardless of who wrote it.

---

## Part 4: Parent Owns Layout — A Drift-Resistant CSS Architecture

### Single Principle

> The parent decides the child's layout; the child only writes its own internals.

CSS layout is fundamentally a "parent arranges children" mechanism. All structural rules derive from this one principle.

### Block and Element — Two Structural Units

| Type | Naming Convention | Example | Role |
|---|---|---|---|
| Block | Two-word kebab-case | `.feature-card`, `.card-header` | Independent component unit |
| Element | Single-word | `.title`, `.body`, `.icon` | Constituent part within a Block |

Allowed parent-child relationships:

| Parent | Child | Allowed |
|---|---|---|
| Block | Block | ✅ |
| Block | Element | ✅ |
| Element | Block | ❌ |
| Element | Element | ✅ (depth limit) |

### Property Responsibility Separation

```scss
// Parent Block
.card-list {
  display: flex;           // Container property → on itself
  gap: 24px;               // Container property → on itself

  > .feature-card {
    margin-top: 16px;      // Item property → specified from parent
    flex: 1;               // Item property → specified from parent
  }
}

// Child Block (separate file)
.feature-card {
  padding: 16px;           // Internal property → on itself
  text-align: center;      // Internal property → on itself
}
```

Three property categories:
- **Container properties** (display, gap, justify-content) → Write on the Block itself
- **Item properties** (margin, flex, order, align-self) → Write in the parent Block's `> .child` selector
- **Internal properties** (padding, font-size, color) → Write on the selector itself

### 1 Block = 1 File

```
feature-card/
├── feature-card.scss      ← Root Block
└── scss/
    ├── card-header.scss   ← Child Block
    ├── card-body.scss     ← Child Block
    └── index.scss         ← @use aggregation
```

Advantages:
- Physical prevention of class name collisions
- Forced responsibility separation
- Navigation without searching (`@rel` comments)

### Variant and State Separation

| Type | Purpose | HTML | SCSS |
|---|---|---|---|
| Variant | Static variations | `data-variant="primary"` | `&[data-variant="primary"]` |
| State | Dynamic states | `data-state="active"` | `&[data-state="active"]` |

### SCSS Section Structure

A single Block file consists of up to three sections:
1. **Base structure:** Block layout, child placement, Variants
2. **--shared (optional):** Styles shared among descendants
3. **--interaction (optional):** States, hover, focus, ARIA, transitions, animations

### The Big Picture

```
Single principle (parent handles layout, child handles internals)
  ├── Naming: Block (two-word) / Element (single-word)
  ├── Structure: Element > Block forbidden, Element depth limit
  ├── Properties: Container / Item / Internal — 3 categories
  ├── Files: 1 Block = 1 file
  ├── State: Variant / State separation
  └── Sections: Base / --shared / --interaction
```

Every rule derives from the single principle, and every rule is verifiable by Stylelint.

---

## Part 5: Same Lint, Same Result — A Stylelint Toolchain for Humans and AI Agents

### Tool Overview

| Tool | Role | Users |
|---|---|---|
| Stylelint plugin | Verifies SCSS structure, naming, placement | Humans and AI agents |
| HTML CLI | Generates SCSS from HTML, validates HTML structure | Primarily AI agents |
| AI Agent Guide | Self-contained document: rule specs, decision flow, fix procedures | AI agents |
| VS Code: Comment Links | Click `@rel` comments to navigate between files | Humans |
| VS Code: HTML to SCSS | Generate SCSS via keyboard shortcuts | Humans |

Humans use VS Code; AI agents use the CLI. Both share the same generation logic and the same validation rules.

### Stylelint Plugin — Eight Rules

| Rule | Verification |
|---|---|
| `spiracss/class-structure` | Block/Element naming, parent-child structure, section composition |
| `spiracss/property-placement` | Container/Item/Internal property placement |
| `spiracss/page-layer` | Page layer boundaries and links |
| `spiracss/interaction-scope` | --interaction section position and structure |
| `spiracss/interaction-properties` | transition/animation placement |
| `spiracss/keyframes-naming` | @keyframes naming and placement |
| `spiracss/pseudo-nesting` | Pseudo-class and pseudo-element nesting |
| `spiracss/rel-comments` | @rel comment parent-child links |

### AI Agent Workflow

```
Load AI Agent Guide → Write HTML → Generate SCSS via CLI → Add styles → Validate with Stylelint
```

Lint alone is enough to guarantee convergence. The AI Agent Guide accelerates by reducing iterations, but is supplementary, not required.

### Series Summary

1. CSS architecture should shift from "rules to memorize" to "feedback systems"
2. Preventing design drift requires "invariants," not conventions
3. Leveraging invariants requires a feedback loop that returns "what to fix and how"
4. SpiraCSS derives everything from a single principle: "parent handles layout, child handles internals"
5. A Stylelint plugin, CLI, and VS Code extensions support this design in practice

> **The underlying idea:** A system where the design converges to the same result regardless of who writes it. Whether junior developer, senior developer, or AI agent — if it passes the same lint, the structure, separation of responsibilities, and placement align to the same standard.

**What matters is not creating good rules, but designing an environment that resists breaking down.**
