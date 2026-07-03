# AI Harness: Модули и Дорожная Карта (W1–W10)

## Архитектура Harness

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AI HARNESS — 3 СЛОЯ + 7 МОДУЛЕЙ                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  СЛОЙ 1: STRUCTURED INSTRUCTIONS                                    │   │
│  │  "Агент знает, что делать"                                           │   │
│  │                                                                     │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│  │  │  AGENTS.md   │  │ feature_list │  │  ADR-архив   │              │   │
│  │  │  (root)      │  │ .json        │  │ (decisions/) │              │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘              │   │
│  │                                                                     │   │
│  │  ┌──────────────┐  ┌──────────────┐                               │   │
│  │  │ Path-specific│  │   Skills     │                               │   │
│  │  │ rules (.md)  │  │ (SKILL.md)   │                               │   │
│  │  └──────────────┘  └──────────────┘                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  СЛОЙ 2: MECHANICAL ENFORCEMENT                                     │   │
│  │  "Агент не может сделать неправильно"                                │   │
│  │                                                                     │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│  │  │ PreToolUse   │  │  Lefthook    │  │  GitHub      │              │   │
│  │  │ hooks        │  │  (pre-commit)│  │  Actions CI  │              │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘              │   │
│  │                                                                     │   │
│  │  ┌──────────────┐  ┌──────────────┐                               │   │
│  │  │ Custom       │  │ Evaluator    │                               │   │
│  │  │ linters      │  │ (review bot) │                               │   │
│  │  └──────────────┘  └──────────────┘                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  СЛОЙ 3: SESSION MEMORY & CHECKPOINTING                             │   │
│  │  "Агент не теряет прогресс"                                          │   │
│  │                                                                     │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│  │  │ Git commits  │  │ Context reset│  │ Lessons      │              │   │
│  │  │ (per feature)│  │ protocol     │  │ (docs/)      │              │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘              │   │
│  │                                                                     │   │
│  │  ┌──────────────┐  ┌──────────────┐                               │   │
│  │  │ Known-good   │  │ Meta-lessons │                               │   │
│  │  │ git tags     │  │ (cross-proj) │                               │   │
│  │  └──────────────┘  └──────────────┘                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  ДОПОЛНИТЕЛЬНЫЕ МОДУЛИ (cross-cutting)                              │   │
│  │                                                                     │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │   │
│  │  │  MCP     │ │ Skills   │ │ Impr.    │ │ Obsrv.   │ │ Security │ │   │
│  │  │  tools   │ │ registry │ │  Loop    │ │  (RUM)   │ │  scan    │ │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Инвентарь модулей Harness

### 1.1. Слой 1: Structured Instructions — 4 модуля

| # | Модуль | Описание | Артефакт | MVP (W1) | Эволюция |
|---|--------|----------|----------|----------|----------|
| **1.1** | **AGENTS.md** | Корневой файл инструкций: stack, architecture, commands, anti-patterns. <50 строк [^4^][^41^]. | `AGENTS.md` (symlink: `CLAUDE.md`, `.openhands-instructions`) | 50 строк, базовый stack (Next.js, React, TS), 5 anti-patterns, 5 commands | W2: +Turborepo rules; W4: +Trusted Types; W6: +Web Workers; W10: +cross-project conventions |
| **1.2** | **Path-specific rules** | Domain-specific инструкции, загружаемые по globs. Progressive disclosure [^47^][^69^]. | `.claude/rules/*.md` (YAML frontmatter + globs) | 2 rules: react-components.md, api-endpoints.md | W2: +tests.md; W3: +websocket.md; W4: +security.md (CSP, Trusted Types); W5: +performance.md (budgets) |
| **1.3** | **Skills (SKILL.md)** | Reusable instruction bundles: create-component, write-test, deploy-docker. Progressive disclosure (3 уровня: catalog → activate → execute) [^60^][^67^][^72^]. | `.claude/skills/<name>/SKILL.md` + scripts/ + references/ | 3 skills: react-component, api-endpoint, e2e-test | W2: +compound-component; W3: +websocket-handler; W4: +crdt-setup; W6: +web-worker; W8: +docker-deploy |
| **1.4** | **feature_list.json** | Machine-readable якорь состояния: features, tech_debt, decisions, lessons. Агент читает при старте, обновляет при завершении [^44^][^55^]. | `feature_list.json` | 5 features W1, статус pending/completed, commit SHA | W3: +tech_debt массив; W5: +performance metrics; W7: +ADR ссылки; W10: +cross-project patterns |

### 1.2. Слой 2: Mechanical Enforcement — 4 модуля

| # | Модуль | Описание | Артефакт | MVP (W1) | Эволюция |
|---|--------|----------|----------|----------|----------|
| **2.1** | **PreToolUse hooks** | Безопасность в full-auto режиме: блокировка опасных операций (rm -rf, git push --force, .env edit). 12 lifecycle events [^39^][^4^]. | `.claude/config.json` | Block: rm -rf, git push --force, curl \| sh | W3: +block .env edits; W4: +block CSP config changes; W6: +auto-lint on Edit; W8: +block Docker secrets |
| **2.2** | **Lefthook (pre-commit)** | Локальный feedback loop: lint + format + typecheck перед коммитом. Go-based, быстрый. Для агентов — обязательно, для human — можно `--no-verify` [^4^]. | `lefthook.yml` | pnpm lint, pnpm typecheck | W2: +Biome format; W4: +security lint (Semgrep); W5: +Lighthouse CI; W7: +a11y checks (axe-core) |
| **2.3** | **Custom linters** | Project-specific правила с self-correcting messages. 4 категории: grep-ability, glob-ability, architecture boundaries, security [^68^][^4^]. | `eslint-plugin-local-rules/` или `ast-grep/` | 5 rules: no-any, no-console, no-jsx-logic, layer-check, no-inline-styles | W2: +bundle-budget, +compound-component-pattern; W3: +state-normalization; W4: +trusted-types-enforce; W6: +worker-boundary |
| **2.4** | **CI Quality Gates** | Hard gates в GitHub Actions: lint, test, typecheck, build. PR не мержится без прохождения [^46^][^52^]. | `.github/workflows/quality-gates.yml` | 4 jobs: lint, unit-tests, e2e-tests, build | W2: +Lighthouse budget gate; W4: +security scan (Snyk); W5: +bundle analyzer; W6: +coverage gate 80% |

### 1.3. Слой 3: Session Memory — 4 модуля

| # | Модуль | Описание | Артефакт | MVP (W1) | Эволюция |
|---|--------|----------|----------|----------|----------|
| **3.1** | **Git checkpoints** | Commit после каждой feature: «feat: F02 image pipeline». Feature — atomic unit работы [^44^]. | Git history | Commit per feature, conventional commits | W3: +task-level commits (sub-features); W5: +known-good git tags; W8: +rollback procedures |
| **3.2** | **Context reset protocol** | Полный сброс (/clear) между задачами с передачей структурированных артефактов [^44^][^55^]. | Protocol (documented in AGENTS.md) | Reset после каждой feature, чтение feature_list.json | W4: +reset после 20 сообщений; W6: +reset при переключении домена; W9: +auto-reset по токен-лимиту |
| **3.3** | **ADR-архив** | Architecture Decision Records: каждое архитектурное решение фиксируется (почему SSE а не WebSocket) [^38^][^55^]. | `docs/decisions/ADR-NNN-*.md` | 2–3 ADR (SSR vs CSR, SSE vs WebSocket) | W2: +5 ADR (design system); W4: +3 ADR (CRDT, Trusted Types); W6: +2 ADR (Web Workers); W10: +cross-project ADR consolidation |
| **3.4** | **Lessons log** | Ошибки и находки: «blur-hash требует canvas fallback для SSR». Агент пишет сам, читает при следующей сессии [^55^]. | `docs/lessons/lesson-NNN-*.md` | 3–5 lessons после W1 | W3: +10 lessons; W5: +category tags; W7: +severity levels; W10: +meta-lessons (cross-project) |

### 1.4. Дополнительные модули (cross-cutting) — 5 модулей

| # | Модуль | Описание | Артефакт | MVP (W1) | Эволюция |
|---|--------|----------|----------|----------|----------|
| **4.1** | **Skills Registry** | Общий репозиторий шаблонов: skills symlink'ятся в каждый проект. Cross-project reuse [^60^][^73^]. | `~/harness-templates/skills/` | 3 skills в shared repo | W3: +7 skills; W5: +skill versioning; W7: +skill marketplace (Agensi); W10: +15 skills, internal library |
| **4.2** | **Improvement Loop** | Traces → feedback → evals → optimization. После каждого проекта: что пошло не так → обновление harness [^62^]. | `harness-improvement.md` | Неактивен (собираем data) | W3: +первый review; W5: +eval prompts; W7: +HALO optimization; W9: +quantitative metrics (PRs/day, rework rate) |
| **4.3** | **Evaluator module** | Отдельный агент-оценщик: проверяет security, performance, correctness, a11y [^55^][^59^]. | `evaluator-skill/` (SKILL.md) | Неактивен | W4: +security evaluator; W6: +performance evaluator; W8: +full evaluator (5 criteria); W10: +auto-evaluation on every PR |
| **4.4** | **Observability (RUM)** | Real User Monitoring: web-vitals, error tracking, perf metrics. Dogfooding [^53^]. | Sentry + Lighthouse CI | Неактивен | W5: +web-vitals library; W6: +Datadog RUM; W8: +custom metrics; W10: +full monitoring dashboard |
| **4.5** | **Security Scanning** | Автоматический security scan: Snyk, Semgrep, secret detection [^68^][^53^]. | `.github/workflows/security.yml` | Неактивен | W2: +Semgrep rules; W4: +Snyk scan; W6: +secret detection; W8: +full security pipeline |

---

## 2. Дорожная Карта: W1 → W10

### 2.1. Матрица модулей × проекты

| Модуль | W1 Social Feed | W2 E-com | W3 Chat | W4 Collab | W5 Video | W6 Dashboard | W7 Calendar | W8 File Mgr | W9 Maps | W10 Notifications |
|--------|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| **AGENTS.md** | MVP (50стр) | +Turborepo | +MFE | +Security | +Budgets | +Workers | +A11y | +CSP strict | +Lazy load | +Cross-proj |
| **Path rules** | 2 rules | 4 rules | 6 rules | 8 rules | 10 rules | 12 rules | 14 rules | 16 rules | 18 rules | 20 rules |
| **Skills** | 3 skills | 5 skills | 7 skills | 10 skills | 12 skills | 14 skills | 16 skills | 18 skills | 20 skills | 22 skills |
| **feature_list.json** | basic | +tech_debt | +metrics | +ADR refs | +perf | +coverage | +a11y | +security | +bundle | +patterns |
| **PreToolUse hooks** | basic (3 блока) | +4 блока | +6 блоков | +8 блоков | +10 блоков | +12 блоков | +14 блоков | +16 блоков | +18 блоков | +20 блоков |
| **Lefthook** | lint+type | +format | +test | +security | +Lighthouse | +a11y | +i18n | +docker | +bundle | +full |
| **Custom linters** | 5 rules | 8 rules | 12 rules | 16 rules | 20 rules | 24 rules | 28 rules | 32 rules | 36 rules | 40 rules |
| **CI Gates** | 4 jobs | 5 jobs | 6 jobs | 7 jobs | 8 jobs | 9 jobs | 10 jobs | 11 jobs | 12 jobs | 13 jobs |
| **Git checkpoints** | per feature | per feature | per task | per task | +tags | +tags | +rollback | +rollback | +rollback | +rollback |
| **Context reset** | per feature | per feature | per 20 msg | per 20 msg | auto | auto | auto | auto | auto | auto |
| **ADR** | 2–3 | 5 | 8 | 11 | 14 | 17 | 20 | 23 | 26 | 30+ |
| **Lessons** | 3–5 | 10 | 15 | 20 | 25 | 30 | 35 | 40 | 45 | 50+ |
| **Skills Registry** | 3 shared | 5 shared | 7 shared | 10 shared | 12 shared | 14 shared | 16 shared | 18 shared | 20 shared | 22 shared |
| **Improvement Loop** | data collect | data collect | первый review | +evals | +metrics | +HALO | +optimization | +quantitative | +auto | +full loop |
| **Evaluator** | — | — | — | security eval | +perf eval | +full eval | +a11y eval | +all criteria | +auto-eval | +gate |
| **Observability** | — | — | — | — | web-vitals | Datadog RUM | +custom metrics | +dashboard | +alerts | +full |
| **Security Scan** | — | Semgrep | +Snyk | +secrets | +full pipeline | +auto-fix | +CSP check | +CSRF check | +XSS check | +complete |

### 2.2. Эволюция по фазам

#### Phase 0: Foundation (до W1) — «Строим скелет»

**Цель:** Полноценный MVP Harness со всеми 3 слоями, готовый к работе.

| Модуль | Что создаём |
|--------|------------|
| AGENTS.md | 50 строк, stack W1, 5 anti-patterns, 5 commands |
| Path rules | react-components.md, api-endpoints.md |
| Skills | react-component, api-endpoint, e2e-test |
| feature_list.json | Schema + 5 features W1 |
| PreToolUse hooks | Block: rm -rf, git push --force, curl \| sh |
| Lefthook | pnpm lint, pnpm typecheck |
| Custom linters | 5 rules: no-any, no-console, no-jsx-logic, layer-check, no-inline-styles |
| CI Gates | 4 jobs: lint, unit, e2e, build |
| Git protocol | Conventional commits, commit per feature |
| Context reset | Protocol в AGENTS.md, reset per feature |
| ADR | Template + 2–3 initial ADR |
| Lessons | Template + docs/lessons/ directory |
| Skills Registry | `~/harness-templates/skills/` с 3 skills |

#### Phase 1: W1–W3 — «Обкатка и первые улучшения»

**W1: Social Feed** — первый запуск harness. Собираем lessons, наблюдаем за поведением агента.

**W2: E-commerce** — harness тот же, контент обновлён:
- AGENTS.md: +Turborepo, +compound components, +design system rules
- Path rules: +tests.md, +component-library.md
- Skills: +compound-component, +checkout-state-machine
- Custom linters: +bundle-budget, +compound-component-pattern
- Lefthook: +Biome format
- Security: +Semgrep initial rules

**W3: Chat** — первый real-time проект, первый MFE:
- AGENTS.md: +WebSocket, +Module Federation 2.0, +state normalization
- Path rules: +websocket.md, +microfrontend.md
- Skills: +websocket-handler, +federated-module
- Custom linters: +state-normalization, +mf-boundary-check
- PreToolUse: +block .env edits
- feature_list.json: +tech_debt массив
- **Первый Improvement Loop review:** собираем feedback, обновляем harness

#### Phase 2: W4–W7 — «Углубление и специализация»

**W4: Collaborative Editor** — security-first проект:
- AGENTS.md: +Yjs CRDT, +Trusted Types, +CSP, +feature flags
- Path rules: +security.md (CSP, Trusted Types)
- Skills: +crdt-setup, +trusted-types-policy
- Custom linters: +trusted-types-enforce, +csp-check
- PreToolUse: +block CSP config changes
- Security: +Snyk scan, +secret detection
- Evaluator: +security evaluator (первый запуск)

**W5: Video Streaming** — performance-first проект:
- AGENTS.md: +HLS/DASH, +performance budgets, +lazy loading
- Path rules: +performance.md (budgets, metrics)
- Skills: +video-player-setup, +lazy-load-component
- Custom linters: +lazy-import-enforce, +performance-budget
- Lefthook: +Lighthouse CI
- CI: +Lighthouse budget gate, +bundle analyzer
- Observability: +web-vitals library (первый запуск)
- feature_list.json: +performance metrics

**W6: Analytics Dashboard** — data-intensive проект:
- AGENTS.md: +Web Workers, +Canvas/WebGL, +LTTB downsampling
- Path rules: +web-worker.md, +data-viz.md
- Skills: +web-worker-setup, +canvas-chart
- Custom linters: +worker-boundary, +no-main-thread-blocking
- PreToolUse: +auto-lint on Edit
- Evaluator: +performance evaluator
- Observability: +Datadog RUM
- CI: +coverage gate 80%

**W7: Calendar** — accessibility-first проект:
- AGENTS.md: +drag-and-drop, +ARIA, +feature flags (LaunchDarkly)
- Path rules: +a11y.md, +dnd.md
- Skills: +accessible-component, +dnd-setup
- Custom linters: +a11y-check (axe-core)
- Lefthook: +a11y checks
- Evaluator: +a11y evaluator

#### Phase 3: W8–W10 — «Полировка и масштабирование»

**W8: File Manager** — offline-first + security:
- AGENTS.md: +tus protocol, +Service Worker, +CSP strict
- Path rules: +offline.md, +file-upload.md
- Skills: +docker-deploy, +service-worker-setup
- Custom linters: +csp-strict-mode, +file-validation
- PreToolUse: +block Docker secrets exposure
- Security: +full pipeline (Semgrep + Snyk + secrets + CSP)
- Git: +rollback procedures

**W9: Maps** — lazy loading + resumability:
- AGENTS.md: +Mapbox lazy load, +Qwik islands, +bundle optimization
- Path rules: +lazy-load.md, +islands.md
- Skills: +map-integration, +island-component
- Custom linters: +dynamic-import-enforce, +bundle-budget-strict
- Context reset: +auto-reset по токен-лимиту
- Observability: +custom metrics, +alerts

**W10: Notifications** — MFE + cross-tab + capstone:
- AGENTS.md: +Module Federation contracts, +BroadcastChannel, +MJML
- Path rules: +mfe-contracts.md, +cross-tab-sync.md
- Skills: +mfe-contract, +notification-template
- Evaluator: +auto-evaluation on every PR
- Improvement Loop: +quantitative metrics (PRs/day, rework rate, review-comment rate)
- Observability: +full monitoring dashboard
- **Meta-lessons:** consolidate все lessons в cross-project patterns

---

## 3. Принцип эволюции: «Агент ошибается → Harness растёт»

Ключевой принцип OpenAI [^4^]:

> *«Every time the agent makes a mistake, add a test or a linter rule.»*

| Что произошло | Какой модуль обновляется | Пример |
|--------------|-------------------------|--------|
| Агент использовал `any` вместо типа | Custom linters: +no-any rule | W1: после первого `any` |
| Агент забыл про SSR fallback для blur-hash | Lessons: +lesson-001 | W1: docs/lessons/ |
| Агент создал inline styles вместо Tailwind | Custom linters: +no-inline-styles | W1: первый violation |
| Агент нарушил dependency direction | Custom linters: +layer-check | W1: Service → UI import |
| Агент забыл запустить тесты | PreToolUse: +auto-test on Edit | W2: PostToolUse hook |
| Агент сделал git push --force | PreToolUse: +block git push --force | W1: сразу |
| Агент сгенерировал уязвимый код | Security: +Semgrep rule | W2: после первого scan |
| Агент превысил bundle budget | Custom linters: +bundle-budget | W2: Lighthouse CI fail |
| Агент забыл про accessibility | Lefthook: +axe-core | W7: после review |
| Агент потерял контекст после 30 сообщений | Context reset: +auto-reset | W4: после наблюдения |

---

## 4. Группировка проектов по интенсивности Harness

| Группа | Проекты | Характеристика | Harness-интенсивность |
|--------|---------|---------------|----------------------|
| **A: Standard** | W1, W2, W7 | UI-фокус, стандартные паттерны | 12 модулей, базовые линтеры |
| **B: Intensive** | W3, W4, W6, W8, W10 | Real-time, security, data, offline, MFE | 14 модулей, custom линтеры, PreToolUse hooks |
| **C: Performance** | W5, W9 | Media, lazy loading, budgets | 15 модулей, Lighthouse CI, bundle gates, observability |

---

## 5. Итоговая метрика зрелости

| Метрика | W1 | W5 | W10 |
|---------|:--:|:--:|:---:|
| **Правил AGENTS.md** | 5 | 12 | 20+ |
| **Custom linter rules** | 5 | 20 | 40+ |
| **Skills** | 3 | 12 | 22+ |
| **ADR записей** | 2–3 | 14 | 30+ |
| **Lessons** | 3–5 | 25 | 50+ |
| **CI jobs** | 4 | 8 | 13 |
| **PreToolUse блоков** | 3 | 10 | 20 |
| **Security scan уровень** | — | medium | full |
| **Observability** | — | web-vitals | full RUM |
| **Improvement Loop** | data | metrics | auto |
| **Evaluator** | — | partial | full gate |

Harness начинается с **17 модулей** на W1 (MVP) и вырастает до **22+ модулей** на W10 (зрелая система). Каждый проект добавляет 1–3 новых модуля и 3–5 улучшений существующих. Ключевой принцип: **не добавлять всё сразу, а усиливать harness реакцией на реальные ошибки агента**.
