# Week 2: Scaffolding — Designing the Static Foundation

## Day-by-Day Plan

### Day 1 — Theory & Mental Models
**Focus:** Understand *why* scaffolding matters for AI agents before touching any code.

- [x] Read: "Architectural Constraints for AI Agents" (Bitloops / H. Villalobos)
- [x] Read: "AI-First UIs: Why shadcn/ui's Model is Leading the Pack" (Refine.dev)
- [x] Read: "Same Lint, Same Result: A Stylelint Toolchain for Humans and AI Agents" (Zetsubo, Dev.to)
- [x] Write down 3-5 key takeaways in `journal.md`

**Artifact:** None. Pure input day.

---

### Day 2 — Repository Architecture & Documentation (FSD)
**Focus:** Create the repo from scratch using Feature-Sliced Design. Every doc and folder must encode Day 1 principles as concrete constraints.

**Project:** GreatFrontEnd challenge — Shopping Cart component (feature scope for Day 6 agent test).

**Day 1 → Day 2 mapping:**
- *Minimize hallucination surface* → flat public APIs per slice, no barrel re-exports deeper than 1 level
- *Invariants not conventions* → CONVENTIONS.md lists only rules enforceable by lint/CI
- *Convergent feedback loops* → CLAUDE.md describes the write→lint→fix loop, not a list of "please do X"

**Tasks:**
- [ ] `git init` + Next.js/Vite init (separate from this vault)
- [ ] Design FSD folder structure with explicit layer boundaries
- [ ] Create `README.md` — project purpose, tech stack, setup, contribution rules
- [ ] Create `ARCHITECTURE.md` — FSD layers, dependency direction (top→bottom only), slice isolation rules, public API contract
- [ ] Create `CONVENTIONS.md` — ONLY machine-enforceable rules (naming, imports, file structure). Each rule tagged: `[eslint]`, `[stylelint]`, or `[ci-custom]`
- [ ] Create `CLAUDE.md` — agent constraints: what it CAN import, how to add a new feature slice, the feedback loop protocol
- [ ] Initialize `package.json` with project metadata
- [ ] Create empty slice scaffolding for `features/shopping-cart` (index.ts public API only)

**Tooling policy:**
- Agent (spec-kit / Claude) MAY generate boilerplate files from your specs
- Architectural decisions (what goes where, layer rules, public API shape) — YOU decide
- No feature code yet — only empty scaffolding with index.ts stubs

**Artifacts:**
```
fsd-shopping-cart/
├── README.md
├── ARCHITECTURE.md
├── CONVENTIONS.md
├── CLAUDE.md
├── package.json
└── src/
    ├── app/                  ← App layer (providers, routing, global styles)
    │   └── styles/
    ├── pages/                ← Page compositions
    ├── widgets/              ← Independent UI blocks
    ├── features/             ← User interactions (shopping-cart lives here)
    │   └── shopping-cart/
    │       └── index.ts      ← Public API (empty scaffold)
    ├── entities/             ← Business objects (Product, CartItem)
    │   └── product/
    │       └── index.ts
    ├── shared/               ← Reusable kit (ui, lib, api, config)
    │   ├── ui/
    │   ├── lib/
    │   └── config/
    └── ...
```

---

### Day 3 — Linting & Custom FSD Rules
**Focus:** Set up linting stack AND write custom ESLint rules that enforce FSD invariants. This is where theory becomes enforcement.

- [ ] Watch: "Tailwind CSS v4 Full Course" (JS Mastery) — first 30-40 min for fundamentals
- [ ] Install & configure ESLint (flat config, TypeScript, React)
- [ ] Install & configure Prettier
- [ ] Install & configure Stylelint (Tailwind plugin)
- [ ] **Write custom ESLint rule: `fsd/no-cross-slice-imports`** — features/A cannot import from features/B
- [ ] **Write custom ESLint rule: `fsd/no-layer-violation`** — enforce dependency direction (features → entities → shared, never reverse)
- [ ] **Write custom ESLint rule: `fsd/public-api-only`** — forbid deep imports like `features/cart/model/store`, only `features/cart`
- [ ] Add npm scripts: `lint`, `lint:fix`, `format`, `format:check`
- [ ] Write test cases: create intentional violations, verify rules catch them

**Artifacts:**
```
├── eslint.config.js
├── .prettierrc
├── .stylelintrc.json
├── .editorconfig
├── eslint-rules/               ← Custom FSD rules
│   ├── no-cross-slice-imports.js
│   ├── no-layer-violation.js
│   └── public-api-only.js
```

---

### Day 4 — Frontend Scaffolding: Tailwind + shadcn/ui
**Focus:** Set up the styling and component layer. shadcn components go into `shared/ui/` per FSD.

- [ ] Watch: "Why AI is Quietly Killing Tailwind CSS" (Huxn WebDev) — for perspective
- [ ] Install & configure Tailwind CSS v4
- [ ] Set up shadcn/ui — init into `shared/ui/`, add 3-5 base components (Button, Card, Input, Dialog, Badge)
- [ ] Verify components render and ALL lint rules pass (including custom FSD rules)
- [ ] Update `ARCHITECTURE.md` with component inventory and import rules

**Artifacts:**
```
├── tailwind.config.ts
├── src/
│   ├── shared/
│   │   ├── ui/              ← shadcn components live here (per FSD)
│   │   ├── lib/utils.ts     ← cn() helper
│   │   └── config/
│   └── app/
│       └── styles/globals.css
```

---

### Day 5 — CI/CD Pipeline + Architectural Validator
**Focus:** GitHub Actions as the final enforcement layer. Custom validator script as the architectural safety net.

- [ ] Set up GitHub Actions workflow: lint (including custom FSD rules) → type-check → build
- [ ] **Write `scripts/validate-architecture.ts`** — a standalone validator that checks:
  - Every slice has an `index.ts` public API
  - No orphan files outside the FSD structure
  - Dependency graph has no cycles between slices
- [ ] Add `validate` npm script, wire into CI
- [ ] Set up Storybook — init, write stories for shadcn components from Day 4
- [ ] Push repo to GitHub, verify CI passes green

**Artifacts:**
```
├── scripts/
│   └── validate-architecture.ts   ← Custom FSD validator
├── .github/
│   └── workflows/
│       └── ci.yml                 ← lint → validate → type-check → build
├── .storybook/
│   ├── main.ts
│   └── preview.ts
```

---

### Day 6 — Agent Test Drive + Retrospective
**Focus:** Let the AI agent build the Shopping Cart feature. Observe what the harness catches and what slips through.

- [ ] Write a clear prompt: "Build a shopping cart feature using FSD structure with add/remove/quantity"
- [ ] Run the agent — do NOT touch the code yourself
- [ ] Let the agent hit lint errors and fix them (this IS the feedback loop from Day 1)
- [ ] Document: Where did the agent succeed? Where did it misunderstand? Which custom rules saved us?
- [ ] Write `RETROSPECTIVE.md`: what scaffolding helped, what was missing, which rules need tuning
- [ ] Update `journal.md` with final reflections

**Artifact:** `RETROSPECTIVE.md` in the repo — the deliverable for Week 2.

---

## Study Materials Checklist

| Done | Format | Resource | Day |
|------|--------|----------|-----|
| [x] | Article | "Architectural Constraints for AI Agents" — Bitloops | 1 |
| [x] | Article | "AI-First UIs: Why shadcn/ui's Model is Leading the Pack" — Refine.dev | 1 |
| [x] | Article | "Same Lint, Same Result" — Zetsubo (Dev.to) | 1 |
| [ ] | Video | "Tailwind CSS v4 Full Course" — JS Mastery | 3 |
| [ ] | Video | "Why AI is Quietly Killing Tailwind CSS" — Huxn WebDev | 4 |
