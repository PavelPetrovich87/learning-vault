# FSD Shopping Cart — Agent Instructions

## What This Project Is

This is a **learning project** for studying **Harness Engineering** — the discipline of designing environments where AI coding agents produce consistent, architecturally correct code.

The owner is **Pavel**, a Senior Software Engineer. He is building this repo as part of a 6-day curriculum (Week 2: "Scaffolding — Designing the Static Foundation"). The goal is NOT to build a shopping cart. The goal is to build a **self-enforcing repository** that forces any agent (including you) to write correct code through tooling, not through memorization.

**Tech stack:** Vite + React + TypeScript.
**Architecture:** Feature-Sliced Design (FSD).
**Target feature (Day 6):** Shopping cart from GreatFrontEnd challenge.

---

## The Three Pillars (Theory Behind This Repo)

These principles were derived from Day 1 research and govern EVERY decision in this repo:

### Pillar 1: Minimize Hallucination Surface
- Fewer abstractions = fewer places for the agent to guess wrong.
- Declarative tools over imperative (Tailwind classes > CSS-in-JS).
- Code ownership over black-box dependencies (shadcn/ui model: components live in `shared/ui/`, not in `node_modules`).
- Flat public APIs: each FSD slice exports through `index.ts`, no deep barrel re-exports.

### Pillar 2: Define Invariants, Not Conventions
- A **convention** is a recommendation ("keep files small"). It requires interpretation. It drifts.
- An **invariant** is a binary rule ("features/A cannot import from features/B"). It's either violated or not.
- Every rule in this repo must satisfy three criteria:
  1. **Binary evaluation** — no gray zones
  2. **Syntax-verifiable** — detectable from source code alone (no runtime)
  3. **Locally verifiable** — checkable by looking at the target file alone
- If a rule can't be enforced by a tool, it doesn't belong in CONVENTIONS.md.

### Pillar 3: Build Convergent Feedback Loops
- The agent workflow is: `write code → run lint → get error with fix instructions → fix → re-lint → zero errors`.
- The agent does NOT need to memorize rules. It needs to reach **exit code 0**.
- Error messages must include: (1) what's wrong, (2) why it's wrong, (3) how to fix it.
- "Same lint, same result" — same exit condition converges all authors (human or AI) to the same standard.

---

## FSD Architecture Rules

### Layer Hierarchy (top to bottom, NEVER reverse)

```
app → pages → widgets → features → entities → shared
```

Each layer can ONLY import from layers BELOW it. Never sideways within the same layer (except `shared`).

### Folder Structure

```
src/
├── app/              # App-level: providers, routing, global styles
│   └── styles/
├── pages/            # Page compositions (assemble widgets/features)
├── widgets/          # Self-contained UI blocks (combine features/entities)
├── features/         # User interactions with business logic
│   └── shopping-cart/
│       ├── index.ts  # PUBLIC API — the ONLY file other layers may import
│       ├── ui/       # React components
│       ├── model/    # State, hooks, business logic
│       └── api/      # Data fetching
├── entities/         # Business objects (Product, CartItem)
│   └── product/
│       ├── index.ts  # PUBLIC API
│       ├── ui/
│       └── model/
└── shared/           # Reusable infrastructure
    ├── ui/           # shadcn/ui components live here
    ├── lib/          # Utilities (cn(), formatPrice(), etc.)
    ├── api/          # API client, request helpers
    └── config/       # Constants, env, theme tokens
```

### Import Rules (Invariants)

These rules are enforced by custom ESLint rules. Do NOT violate them — the linter will catch you.

| Rule | What it means | ESLint rule |
|------|--------------|-------------|
| **No cross-slice imports** | `features/cart` cannot import from `features/wishlist` | `fsd/no-cross-slice-imports` |
| **No layer violation** | `entities/` cannot import from `features/`. `shared/` cannot import from `entities/`. | `fsd/no-layer-violation` |
| **Public API only** | `import { X } from '@/features/cart'` is OK. `import { X } from '@/features/cart/model/store'` is FORBIDDEN. | `fsd/public-api-only` |

### Adding a New Slice

When creating a new slice (e.g., `features/wishlist`):

1. Create folder: `src/features/wishlist/`
2. Create `index.ts` — this is the public API. Export ONLY what other layers need.
3. Create internal folders as needed: `ui/`, `model/`, `api/`
4. All internal imports stay internal. Only `index.ts` exports cross the boundary.
5. Run `npm run lint` — fix any errors before committing.

---

## Agent Workflow Protocol

When working on this codebase, follow this loop:

```
1. Read ARCHITECTURE.md and CONVENTIONS.md FIRST
2. Write code
3. Run: npm run lint
4. If errors → read error messages (they contain fix instructions) → fix → goto 3
5. Run: npm run validate (architectural validator)
6. If errors → fix structure → goto 5
7. Run: npm run typecheck
8. Run: npm run build
9. Done only when ALL commands exit with 0
```

Do NOT skip steps. Do NOT suppress warnings. The linter is your guide, not your enemy.

---

## Day-by-Day Build Plan

This repo is being constructed incrementally. Here's where we are:

### Day 1 — Theory (COMPLETE ✅)
Read three articles, extracted principles. Created concept notes and flashcards in the learning vault.
Key output: "The Self-Enforcing Repository" — a meta-concept unifying all three sources into the Three Pillars above.

### Day 2 — Repository Architecture & Documentation (CURRENT 🟡)
**Goal:** Create the skeleton. No feature code — only structure, docs, and empty scaffolding.

Tasks:
- [x] `git init` + Vite init (React + TypeScript)
- [ ] Design FSD folder structure with explicit layer boundaries
- [ ] Create `README.md` — project purpose, tech stack, setup, contribution rules
- [ ] Create `ARCHITECTURE.md` — FSD layers, dependency direction, slice isolation, public API contract
- [ ] Create `CONVENTIONS.md` — ONLY machine-enforceable rules, each tagged: `[eslint]`, `[stylelint]`, or `[ci-custom]`
- [ ] Create `CLAUDE.md` (THIS FILE — will be refined as project evolves)
- [ ] Create empty slice scaffolding: `features/shopping-cart/index.ts`, `entities/product/index.ts`

### Day 3 — Linting & Custom FSD Rules
**Goal:** The linting stack IS the harness. Standard rules + 3 custom ESLint rules for FSD.

Tasks:
- [ ] ESLint (flat config, TypeScript, React)
- [ ] Prettier
- [ ] Stylelint (Tailwind plugin)
- [ ] Custom rule: `fsd/no-cross-slice-imports`
- [ ] Custom rule: `fsd/no-layer-violation`
- [ ] Custom rule: `fsd/public-api-only`
- [ ] Test cases: intentional violations to verify rules catch them

### Day 4 — Tailwind + shadcn/ui
**Goal:** Styling layer. shadcn components go into `shared/ui/` (FSD-compliant placement).

Tasks:
- [ ] Tailwind CSS v4
- [ ] shadcn/ui init into `shared/ui/`
- [ ] Base components: Button, Card, Input, Dialog, Badge
- [ ] Verify components render AND pass all lint rules (including custom FSD rules)

### Day 5 — CI/CD + Architectural Validator
**Goal:** GitHub Actions as the final enforcement layer. Custom architecture validator script.

Tasks:
- [ ] GitHub Actions: lint → validate → type-check → build
- [ ] `scripts/validate-architecture.ts` — checks:
  - Every slice has `index.ts`
  - No orphan files outside FSD structure
  - No dependency cycles between slices
- [ ] Storybook for component stories
- [ ] Push to GitHub, CI green

### Day 6 — Agent Test Drive + Retrospective
**Goal:** Agent (Claude) builds the shopping cart feature. Pavel does NOT touch the code. The harness either works or it doesn't.

Tasks:
- [ ] Prompt: "Build a shopping cart feature using FSD structure with add/remove/quantity"
- [ ] Agent hits lint errors, reads fix instructions, fixes them (this IS the feedback loop)
- [ ] Write `RETROSPECTIVE.md`: what scaffolding helped, what was missing, which rules need tuning

---

## Communication Preferences

- **Language:** Speak in **Russian**. All code, docs, comments, and file content in **English**.
- **Tone:** Informal, concise. No corporate speak ("Certainly!", "Great question!"). Just get to the point.
- **Analogies:** Napoleonic/military analogies are welcome when they clarify a concept.
- **Proactivity:** Complete the task first, then briefly mention if you spot a knowledge gap or missing invariant. Don't overwhelm with unsolicited roadmaps.

---

## Related Learning Materials

The full study notes, concept notes, and flashcards for this project live in a separate Obsidian vault:
`/Users/user/Documents/my-learning-vault/harness_engineering/`

Key concept notes created during Day 1:
- `Self-Enforcing-Repository.md` — meta-article synthesizing all three Day 1 sources
- `Feedback-Systems-Over-Conventions.md` — why invariants beat conventions
- `Architectural-Constraints-AI-Flashcards.md` — SR flashcards on architectural enforcement
