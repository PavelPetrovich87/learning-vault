---
status: analyzing
tags: [agent-harness, test-corpus, evaluation, specification]
created: 2026-06-09
updated: 2026-06-09
---

# Test Corpus Specification + Evaluation Rubric

> Specification for 6 synthetic test repositories + criteria for validating the `repo-classifier` and maturity engine against them.
>
> **For:** Separate agent that will physically create these repos in a dedicated folder.
> **Purpose:** Provide ground truth to validate the agent-readiness ladder (L0→L3) and the classifier's detection accuracy.

---

## 📋 Overview

**What the corpus tests:**
1. **L0 detection** — does the classifier recognize a repo with zero agent affordances?
2. **L1 detection** — does it identify context quality (AGENTS.md, repo map) vs. navigability?
3. **L2 detection** — does it find feedback loops (lint/test/type) and architectural boundaries?
4. **L3 detection** — does it spot drift risks and maintenance discipline?
5. **Greenfield protection** — does it recommend L1, not L3, for an empty project?
6. **Legacy handling** — does it not crash on old/heterogeneous code?

**Repository count:** 6
**Expected effort to create:** Low — all are intentionally small (≤50 files each, except monorepo which needs package structure).

---

## 🏗️ Repository 1: `greenfield-nextjs` (L0 — The Walking Skeleton)

**Purpose:** P2 truth-detector. If the skill dumps L3 advice on an empty project, the ladder is broken.

**Target maturity:** L0 Blind

**Directory structure:**
```
greenfield-nextjs/
├── package.json          # create-next-app default
├── next.config.js        # default
├── tsconfig.json         # default
├── tailwind.config.ts    # default
├── postcss.config.mjs    # default
├── app/
│   ├── layout.tsx        # default
│   ├── page.tsx          # default
│   └── globals.css       # default
├── public/
│   └── (empty)
└── .gitignore            # default
```

**Intentionally missing (these are the gaps the skill should find):**
- No `AGENTS.md`
- No `README.md` (or default create-next-app README — counts as "minimal boilerplate, not agent-oriented")
- No repo map, no conventions doc
- No lint/test scripts in package.json (or default ones if any — but no custom config)
- No `.cursorrules`, `.aider.conf.yml`, `.claude/`
- No architecture boundaries defined

**Expected classifier output per axis:**

| Axis | Expected value | Why |
|------|----------------|-----|
| `project_class` | `web-frontend` | Next.js app |
| `language` | `typescript` | tsconfig.json, .tsx files |
| `framework` | `nextjs` | package.json dependencies |
| `build_system` | `next` | next.config.js, scripts |
| `test_framework` | `none` | No test deps, no test scripts |
| `package_manager` | `npm` | package-lock.json (or whatever create-next-app makes) |
| `monorepo` | `none` | Single package root |
| `agents_md_quality` | `none` | No AGENTS.md |
| `doc_richness` | `minimal` | Only auto-generated README |
| `conventions` | `none` | No style guide, no naming conventions doc |

**Expected gaps (what the skill should recommend):**
1. **Create root `AGENTS.md`** — project identity, build/test commands, architecture summary.
2. **Add test framework** — even minimal (Vitest or Jest).
3. **Add lint script** — ESLint config if missing.

**What it should NOT recommend:**
- Nested `AGENTS.md` (no monorepo)
- Drift detection (no context to drift from)
- Complex boundary rules (no architecture to protect yet)
- CI/CD (out of scope)

---

## 🏗️ Repository 2: `express-api` (L0→L1 — Small Backend)

**Purpose:** Tests that the classifier doesn't over-engineer a simple API. Medium-sized backend, some structure, but zero agent context.

**Target maturity:** L0→L1 (has structure, but no agent affordances)

**Directory structure:**
```
express-api/
├── package.json              # express, dotenv, cors
├── tsconfig.json
├── src/
│   ├── index.ts              # Express app bootstrap
│   ├── routes/
│   │   ├── users.ts          # CRUD routes
│   │   └── orders.ts
│   ├── models/
│   │   ├── User.ts           # Simple interfaces
│   │   └── Order.ts
│   └── middleware/
│       ├── auth.ts
│       └── errorHandler.ts
├── tests/
│   └── (empty folder — intention: tests exist as concept but no files)
├── .env.example
└── .gitignore
```

**Intentionally present:**
- Basic folder structure (routes, models, middleware)
- Express framework clearly identifiable
- TypeScript
- Empty tests/ folder (signal: "tests were planned but never written")

**Intentionally missing:**
- No `AGENTS.md`
- No `README` with architecture overview
- No test files (empty folder is a signal!)
- No lint config
- No API documentation
- No validation schema documented

**Expected classifier output per axis:**

| Axis | Expected value | Signals |
|------|----------------|---------|
| `project_class` | `api-backend` | Express, routes/ folder |
| `language` | `typescript` | tsconfig.json, .ts files |
| `framework` | `express` | package.json |
| `build_system` | `tsc` | tsconfig.json, no bundler |
| `test_framework` | `none` (or `planned`) | Empty tests/ folder |
| `package_manager` | `npm` | package.json |
| `monorepo` | `none` | |
| `agents_md_quality` | `none` | No AGENTS.md |
| `doc_richness` | `minimal` | .env.example only |
| `conventions` | `emergent` | Folder structure implies conventions but none documented |

**Expected gaps:**
1. **Root `AGENTS.md`** — document API structure: routes, models, middleware pattern.
2. **Tests** — empty tests/ folder is a smell; recommend test framework + sample tests.
3. **Linting** — no ESLint/Prettier config.
4. **API docs** — no OpenAPI/typed routes.

**What it should NOT recommend:**
- Monorepo structure (single package)
- Complex boundaries (simple API)
- Drift detection (no generated context yet)

---

## 🏗️ Repository 3: `nx-monorepo` (L1→L2 — Complex Structure, Partial Tooling)

**Purpose:** Tests monorepo detection, nested context, and the L1→L2 boundary. Has good structure and some tooling, but missing feedback loops and boundaries.

**Target maturity:** L1 Oriented (has structure + some context, but no self-correcting feedback loop)

**Directory structure:**
```
nx-monorepo/
├── nx.json
├── package.json              # Nx workspace deps
├── tsconfig.base.json
├── README.md                 # Generic Nx README, NOT agent-oriented
├── apps/
│   ├── web/
│   │   ├── project.json
│   │   ├── src/
│   │   │   ├── main.tsx
│   │   │   └── app/
│   │   │       └── app.tsx
│   │   └── tsconfig.json
│   └── api/
│       ├── project.json
│       ├── src/
│       │   ├── main.ts
│       │   └── routes/
│       │       └── health.ts
│       └── tsconfig.json
├── libs/
│   ├── shared-ui/
│   │   ├── project.json
│   │   ├── src/
│   │   │   └── index.ts
│   │   └── tsconfig.json
│   └── shared-types/
│       ├── project.json
│       ├── src/
│       │   └── index.ts
│       └── tsconfig.json
└── tools/
    └── scripts/
        └── generate-api-client.ts
```

**Intentionally present:**
- Nx workspace structure (apps + libs)
- README.md (but generic — doesn't explain app/lib boundaries to an agent)
- TypeScript configs per project
- Tools/scripts folder (custom scripts exist)

**Intentionally missing:**
- No `AGENTS.md` at root or in apps/
- No lint config (no .eslintrc, no lint targets in project.json)
- No test targets in project.json
- No `.cursorrules` or agent config
- No architecture decision records (ADRs)
- No API client generation automated (script exists but not wired to build)

**Expected classifier output per axis:**

| Axis | Expected value | Signals |
|------|----------------|---------|
| `project_class` | `monorepo-fullstack` | Nx, apps/web + apps/api |
| `language` | `typescript` | tsconfig.base.json |
| `framework` | `nextjs` (web), `express` (api) | App structures |
| `build_system` | `nx` | nx.json, project.json |
| `test_framework` | `none` | No test configs in project.json |
| `package_manager` | `npm` | package.json |
| `monorepo` | `nx` | nx.json, apps/, libs/ |
| `agents_md_quality` | `none` | No AGENTS.md anywhere |
| `doc_richness` | `partial` | README exists but generic |
| `conventions` | `partial` | Nx conventions implicit, but not documented for agents |

**Expected gaps:**
1. **Root `AGENTS.md`** — explain monorepo topology: apps vs libs, dependency rules, Nx commands.
2. **Nested `AGENTS.md`** — one per app (web, api) documenting entry points, env vars, API surface.
3. **Lint + test targets** — add to project.json files.
4. **Boundary rules** — document which libs can depend on which (enforced by Nx, but not documented).
5. **Command registry** — document custom scripts (generate-api-client.ts).

**What it should NOT recommend:**
- L3 drift detection (no context to drift yet — context must exist first)
- CI/CD setup (out of scope)

---

## 🏗️ Repository 4: `microservices-partial` (L2→L3 — Good Feedback Loop, Missing Maintenance)

**Purpose:** Tests L2 detection and L3 gap identification. Has CI, tests, linting, but no drift detection and stale context.

**Target maturity:** L2 Self-Correcting (has feedback loop, but not self-maintaining)

**Directory structure:**
```
microservices-partial/
├── README.md                 # Exists but outdated (mentions old service names)
├── AGENTS.md                 # EXISTS but stale — mentions removed "billing-service"
├── docker-compose.yml
├── .github/
│   └── workflows/
│       └── ci.yml            # Runs tests + lint on PR
├── services/
│   ├── user-service/
│   │   ├── package.json
│   │   ├── Dockerfile
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   └── routes/
│   │   │       └── userRoutes.ts
│   │   ├── tests/
│   │   │   └── user.test.ts
│   │   └── .eslintrc.json
│   ├── order-service/
│   │   ├── package.json
│   │   ├── Dockerfile
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   └── routes/
│   │   │       └── orderRoutes.ts
│   │   ├── tests/
│   │   │   └── order.test.ts
│   │   └── .eslintrc.json
│   └── notification-service/  # NEW — not mentioned in AGENTS.md or README
       ├── package.json
       ├── Dockerfile
       ├── src/
       │   ├── index.ts
       │   └── routes/
       │       └── notifyRoutes.ts
       ├── tests/
       │   └── notify.test.ts
       └── .eslintrc.json
└── scripts/
    └── run-tests.sh          # Runs all service tests
```

**Intentionally present (L2 signals):**
- `AGENTS.md` exists! (but stale — mentions removed service)
- CI workflow (GitHub Actions)
- ESLint config per service
- Tests exist per service
- Docker compose
- Run-tests.sh script

**Intentionally missing/stale (L3 gaps):**
- AGENTS.md mentions "billing-service" which doesn't exist (stale)
- AGENTS.md doesn't mention "notification-service" (new service, missing from context)
- README.md also outdated
- No drift detection mechanism
- No automated context regeneration
- No service dependency map

**Expected classifier output per axis:**

| Axis | Expected value | Signals |
|------|----------------|---------|
| `project_class` | `microservices` | services/ folder, docker-compose |
| `language` | `typescript` | Multiple tsconfig |
| `framework` | `express` | Service routes |
| `build_system` | `docker` + `tsc` | Dockerfiles, compose |
| `test_framework` | `present` | tests/ per service, run-tests.sh |
| `package_manager` | `npm` | Multiple package.json |
| `monorepo` | `polyrepo-in-monorepo` | Multiple services, shared scripts |
| `agents_md_quality` | `stale` | AGENTS.md exists but mentions removed service, misses new one |
| `doc_richness` | `partial` | README + AGENTS.md but both stale |
| `conventions` | `present` | ESLint, consistent structure |

**Expected gaps:**
1. **Resync `AGENTS.md`** — remove billing-service, add notification-service.
2. **Resync `README.md`** — same.
3. **Service dependency map** — document which service calls which.
4. **Drift detection** — mechanism to catch when new services are added but context isn't updated.
5. **Per-service `AGENTS.md`** — each service should document its API surface, env vars, dependencies.

**What it should NOT recommend:**
- Adding tests (already present)
- Adding linting (already present)
- Major architectural changes (L2 is achieved — gap is maintenance, not capability)

---

## 🏗️ Repository 5: `legacy-monolith` (L0/L1 — Old, Confusing)

**Purpose:** Tests legacy handling. The classifier must not crash, must not over-engineer, must identify the repo is old and agent-unfriendly.

**Target maturity:** L0 Blind (legacy code with minimal modern affordances)

**Directory structure:**
```
legacy-monolith/
├── package.json              # Very old deps (express@3, lodash@3)
├── server.js                 # Main entry — plain JS, no modules
├── config/
│   └── database.js           # Hardcoded credentials (bad practice signal)
├── routes/
│   ├── index.js
│   ├── users.js              # Mixed business logic + DB queries
│   └── admin.js
├── public/
│   ├── css/
│   ├── js/
│   └── images/
├── views/
│   └── (EJS or Pug templates)
├── models/
│   └── (mixed — some in routes/, some here)
├── migrations/
│   └── (empty or one file)
├── README.md                 # One line: "Run node server.js"
└── .gitignore                # Minimal
```

**Intentionally present:**
- Old Express v3
- Mixed concerns (DB queries in routes)
- Minimal README
- No TypeScript
- No tests
- No lint
- Inconsistent folder structure (models scattered)
- Hardcoded config

**Intentionally missing:**
- No `AGENTS.md`
- No modern tooling
- No test framework
- No linting
- No env-based config
- No module boundaries

**Expected classifier output per axis:**

| Axis | Expected value | Signals |
|------|----------------|---------|
| `project_class` | `legacy-web` | Old Express, mixed MVC |
| `language` | `javascript` | .js files, no tsconfig |
| `framework` | `express-v3` | package.json shows old version |
| `build_system` | `none` | No build scripts |
| `test_framework` | `none` | No test files, no test deps |
| `package_manager` | `npm` | package.json |
| `monorepo` | `none` | |
| `agents_md_quality` | `none` | No AGENTS.md |
| `doc_richness` | `minimal` | One-line README |
| `conventions` | `none` | Inconsistent structure |

**Expected gaps:**
1. **Root `AGENTS.md`** — document the legacy structure honestly: "This is a legacy Express 3 monolith. Entry point: server.js. Routes are mixed with DB logic. Caution:..."
2. **Modernization roadmap** — suggest upgrade path (Express 4, TypeScript, tests) but frame as long-term, not immediate agent requirement.
3. **Boundary warnings** — flag hardcoded credentials, mixed concerns as "dangerous for agent modification."

**What it should NOT recommend:**
- L2 feedback loop (impossible without massive modernization — must be honest)
- L3 drift detection (no context to drift)
- Complex monorepo structure
- The skill should be **honest** — "this repo needs L1 context first, then gradual modernization."

---

## 🏗️ Repository 6: `mature-reference` (L3 — The North Star)

**Purpose:** The "gold standard" reference. Every axis should score high. Tests that the classifier recognizes a mature repo and recommends minimal gaps.

**Target maturity:** L3 Self-Maintaining

**Directory structure:**
```
mature-reference/
├── AGENTS.md                 # Comprehensive, current, agent-oriented
├── README.md                 # Architecture overview + quickstart
├── .cursorrules              # Cursor-specific agent rules
├── .github/
│   ├── workflows/
│   │   ├── ci.yml            # Test + lint + type-check
│   │   └── context-check.yml # Custom: validates AGENTS.md freshness
│   └── PULL_REQUEST_TEMPLATE.md
├── docs/
│   ├── architecture/
│   │   ├── overview.md
│   │   └── decisions/        # ADRs
│   ├── api/
│   │   └── openapi.yaml
│   └── conventions/
│       ├── naming.md
│       └── testing.md
├── packages/
│   ├── core/
│   │   ├── package.json
│   │   ├── AGENTS.md         # Package-level context
│   │   ├── src/
│   │   ├── tests/
│   │   └── tsconfig.json
│   ├── ui/
│   │   ├── package.json
│   │   ├── AGENTS.md
│   │   ├── src/
│   │   ├── tests/
│   │   └── tsconfig.json
│   └── utils/
│       ├── package.json
│       ├── AGENTS.md
│       ├── src/
│       ├── tests/
│       └── tsconfig.json
├── scripts/
│   ├── validate-context.ts   # Checks AGENTS.md vs repo structure
│   └── generate-api-docs.ts
├── package.json              # Root scripts: test, lint, typecheck, validate-context
├── tsconfig.json
├── eslint.config.mjs
├── vitest.config.ts
└── .gitignore
```

**Intentionally present (all L3 signals):**
- Root `AGENTS.md` — comprehensive, current
- Per-package `AGENTS.md` — nested context
- `.cursorrules` — agent tool config
- CI with context-check workflow
- `validate-context.ts` script — drift detection
- ADRs — architecture decisions documented
- OpenAPI spec — API surface documented
- Conventions docs — naming, testing
- ESLint, Vitest, TypeScript — full feedback loop
- Consistent structure across packages

**Intentionally minimal (should have tiny gaps):**
- Maybe one package lacks `AGENTS.md` (test: does classifier spot the ONE missing nested context?)
- Maybe one script is not documented in root AGENTS.md

**Expected classifier output per axis:**

| Axis | Expected value | Signals |
|------|----------------|---------|
| `project_class` | `monorepo-library` | packages/, shared structure |
| `language` | `typescript` | tsconfig.json |
| `framework` | `node-library` | Package structure |
| `build_system` | `tsc` + `vitest` | Configs present |
| `test_framework` | `vitest` | vitest.config.ts |
| `package_manager` | `npm` | package.json |
| `monorepo` | `workspace` | packages/ |
| `agents_md_quality` | `comprehensive` | Root + per-package AGENTS.md |
| `doc_richness` | `rich` | README, docs/, ADRs, OpenAPI |
| `conventions` | `documented` | docs/conventions/ |

**Expected gaps (minimal):**
1. **One missing nested `AGENTS.md`** — if we intentionally omit one.
2. **Minor README update** — if something is slightly stale.

**What it should NOT recommend:**
- Major architectural changes
- Adding tests (already present)
- Adding linting (already present)
- L3 jump for other repos (this is the reference, not the recommendation)

---

## 🎯 Evaluation Rubric

### For each repository, judge the classifier output on:

#### 1. Profile Completeness (binary per axis)
- **Pass:** All 10 axes have a value (including `undetermined`).
- **Fail:** Any axis is missing or `null` without `undetermined`.

#### 2. Evidence Quality (graded)
- **A:** Every non-obvious axis has ≥1 recorded signal. Obvious axes (language from file extension) don't need evidence.
- **B:** Most axes have signals; some obvious ones skip evidence.
- **C:** Several axes lack signal justification.
- **F:** Multiple axes are guesses without recorded signals.

#### 3. Axis Accuracy (binary per axis)
- **Pass:** Value matches the expected value in this spec.
- **Fail:** Value is wrong (e.g., classifies `legacy-monolith` as `modern-api`).

**Tolerance:** `framework` and `project_class` can have synonyms (e.g., `api-backend` vs `backend-api`).

#### 4. Gap Identification (graded per repo)
- **A:** All expected gaps identified, no false positives.
- **B:** All expected gaps identified, 1–2 minor false positives.
- **C:** Missing 1 expected gap, or 3+ false positives.
- **F:** Missing >1 expected gap, or major false positives (e.g., recommending L3 for greenfield).

#### 5. Level Recommendation (graded)
- **A:** Recommends exactly the next level from the repo's target. Greenfield → L1. Partial → next level up. Mature → minimal gaps.
- **B:** Recommends reasonable level, slightly aggressive or conservative.
- **C:** Recommends level off by >1 (e.g., L0→L2 or L2→L0).
- **F:** Recommends L3 for L0, or says "already perfect" for legacy.

#### 6. Greenfield Protection (specific to `greenfield-nextjs`)
- **Pass:** Advice is minimal, focused, actionable. Does not mention drift, nested context, or complex boundaries.
- **Fail:** Dumps L2/L3 recommendations on empty project.

#### 7. Legacy Honesty (specific to `legacy-monolith`)
- **Pass:** Acknowledges legacy state. Recommends L1 context first, then gradual modernization. Warns about dangerous patterns (hardcoded creds).
- **Fail:** Pretends legacy is modern. Recommends L2/L3 without acknowledging modernization cost.

#### 8. Drift Detection (specific to `microservices-partial`)
- **Pass:** Identifies AGENTS.md is stale (mentions removed service, misses new service).
- **Fail:** Treats stale AGENTS.md as current, or doesn't detect the mismatch.

---

## 📊 Scoring Summary

| Criterion | Weight | How measured |
|-----------|--------|--------------|
| Profile completeness | 10% | Binary count across all repos |
| Evidence quality | 15% | Graded per repo, averaged |
| Axis accuracy | 25% | Binary count across all axes × repos |
| Gap identification | 25% | Graded per repo, averaged |
| Level recommendation | 15% | Graded per repo, averaged |
| Special cases (greenfield + legacy + drift) | 10% | Binary pass/fail |

**Overall pass threshold:** ≥75% weighted score, with zero F grades on gap identification or level recommendation.

---

## 🚀 For the Agent Creating These Repos

**Folder structure to create:**
```
agent-harness-test-corpus/
├── greenfield-nextjs/        # L0
├── express-api/              # L0→L1
├── nx-monorepo/              # L1→L2
├── microservices-partial/    # L2→L3
├── legacy-monolith/          # L0/L1
└── mature-reference/         # L3
```

**Requirements:**
1. Each repo must be a valid, runnable (or at least parseable) project.
2. File contents should be minimal but realistic (don't need full implementations, just believable structure).
3. `package.json` files should have real dependency names (even if versions are fictional).
4. Intentionally missing items (like tests) should be truly missing, not just empty files.
5. Stale context (`microservices-partial` AGENTS.md) should be believably outdated (mention real service names that don't exist).

**What NOT to include:**
- `node_modules/` (add to .gitignore)
- Real secrets (use placeholder values)
- Full implementations (stubs are fine)

---

## 🔗 Related Documents

- [[Agent Harness Skill — Implementation Plan]] — build phases P1–P2
- [[P0 — Foundations Decisions]] — contracts and Investigation Playbook
- [[Agent Harness Skill — RFC]] — classifier axes definition

---

*Created: 2026-06-09*