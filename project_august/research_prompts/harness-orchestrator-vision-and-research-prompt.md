# AI Harness Orchestrator: Vision Document & Deep Research Prompt

> **Назначение:** Этот документ содержит (1) архитектурное видение единой точки входа для управления AI-harness-системой и (2) промпт для глубокого исследования, который должен быть выполнен агентом-исследователем для валидации, поиска gap'ов и выработки улучшенной инфраструктуры.

---

## Часть 1. Vision: Единый Orchestrator `harness`

### 1.1. Проблема

Мы строим модульную AI-harness-систему из трёх слоёв:
- **Layer 1 — Structured Instructions:** `AGENTS.md`, path-specific rules, `SKILL.md`, `feature_list.json`.
- **Layer 2 — Mechanical Enforcement:** PreToolUse hooks, Lefthook, custom linters, CI quality gates.
- **Layer 3 — Session Memory & Checkpointing:** Git commits, ADR-архив, lessons log, context reset protocol.

Каждый слой использует **свой** способ переноса между проектами: CLI-скаффолдеры, template repos, Copier, NPM-пакеты, Git submodules, symlinks, MCP-серверы, Docker. Управлять этим зоопарком вручную при масштабировании от W1 до W10 невозможно — требуется единая точка входа.

### 1.2. Решение

**Harness Orchestrator** — CLI-инструмент `harness`, который абстрагирует все способы переноса через **плагиновую систему драйверов** и предоставляет единый жизненный цикл:

```
harness init   → bootstrap всех 3 слоёв в новом проекте
harness sync   → синхронизация живого проекта с upstream
harness upgrade --to w4 → атомарный переход между фазами
harness doctor → диагностика целостности
harness eval   → запуск cross-cutting модулей (Evaluator, Security)
```

### 1.3. Архитектура

```
┌─────────────────────────────────────────────┐
│              harness CLI (Node.js/Go)        │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │  init   │ │  sync   │ │ upgrade │       │
│  └────┬────┘ └────┬────┘ └────┬────┘       │
│       └───────────┴───────────┘              │
│                 │                            │
│       ┌─────────▼──────────┐                │
│       │   Driver Router    │                │
│       └─────────┬──────────┘                │
│  ┌──────────────┼──────────────┐            │
│  ▼              ▼              ▼            │
│ agent      copier/npm        git/fs         │
│ driver     drivers           drivers      │
│ (LLM)      (packages)        (submodules)   │
└─────────────────────────────────────────────┘
```

**Конфигурационный стек:**
- `~/harness/config.yaml` — глобальные defaults (registry URL, agent type).
- `./.harness/config.yaml` — проектный профиль: phase (W1..W10), agent, layer versions, overrides.
- `./.harness/lock.yaml` — lock-файл с точными версиями (npm semver, git SHA, copier revision), обеспечивающий воспроизводимость.

### 1.4. Принципы дизайна

1. **Атомарность.** `harness upgrade --to w4` либо применяет все изменения ко всем 3 слоям, либо откатывает всё. Lock-файл позволяет `harness rollback`.
2. **Версионирование всего.** Не только NPM-пакеты, но и шаблоны ADR, skills, base-rules — всё имеет версию и фиксируется в lock.
3. **Agent-native.** Orchestrator сам является вызываемым агентом через Tool Use. Агент не редактирует 5 конфигов — он вызывает `harness sync`.
4. **Progressive disclosure.** W1 получает 17 модулей, W10 — 22+. Orchestrator знает, какие модули включать на каждой фазе, и не перегружает проект раньше времени.
5. **Error-driven growth.** Каждый `harness lesson create` не только пишет markdown, но и может генерировать PR в `@harness/lint` для добавления нового правила — замыкая Improvement Loop.

### 1.5. Матрица драйверов × слои

| Слой | Компонент | Драйвер | Механизм |
|------|-----------|---------|----------|
| **L1** | `AGENTS.md` | `agent` | LLM-генерация по шаблону + project context |
| **L1** | Path-specific rules | `fs` | Symlink из `~/harness/registry/rules/` |
| **L1** | Skills | `git` | Git submodule / subtree |
| **L2** | Custom linters | `npm` | Пакет `@harness/lint` |
| **L2** | Lefthook preset | `npm` | Пакет `@harness/lefthook-config` |
| **L2** | CI workflows | `npm` | Пакет `@harness/ci` |
| **L2** | PreToolUse hooks | `fs` | Symlink `.claude/config.json` |
| **L3** | ADR-шаблоны | `copier` | Copier-шаблон + `copier update` |
| **L3** | Lessons | `copier` | Шаблон + директория `docs/lessons/` |
| **L3** | Git protocol | `agent` | Инструкции в `AGENTS.md` |
| **Cross** | Evaluator | `mcp` | MCP-сервер или Docker-контейнер |
| **Cross** | Security scan | `mcp` / `docker` | Внешний агент-оценщик |
| **Cross** | Improvement Loop | `mcp` | Отправка traces в центральный реестр |

### 1.6. Фазы жизненного цикла (W1→W10)

| Команда | W1 (Foundation) | W2 (E-com) | W4 (Collab) | W10 (Capstone) |
|---------|----------------|------------|-------------|----------------|
| `init` | Создаёт 17 модулей | — | — | — |
| `sync` | Подтягивает patch-версии | +Biome, +bundle-budget | +Security eval | +Auto-eval |
| `upgrade` | — | W1→W2 (Turborepo) | W2→W4 (CRDT, Trusted Types) | W8→W10 (MFE contracts) |
| `doctor` | Проверяет 17 модулей | +5 новых правил | +Security gate | +13 CI jobs |

---

## Часть 2. Deep Research Prompt

> **Роль:** Ты — исследовательский агент по DevTooling, Developer Experience (DX) и инфраструктуре масштабирования AI-агентов.
> **Задача:** Проверить архитектурное видение Harness Orchestrator (Часть 1) на реальных механизмах, инструментах и edge cases. Выявить gap'ы, риски и альтернативы. Предложить улучшенную инфраструктуру.

### 2.1. Контекст исследования

Мы строим систему, которая позволяет:
1. **Bootstrap'ить** AI-harness (3 слоя + cross-модули) в новом проекте одной командой.
2. **Синхронизировать** живой проект с центральным реестром harness (обновление линтеров, skills, CI).
3. **Атомарно апгрейдить** проект между фазами (W1→W2→...→W10), добавляя модули по мере роста сложности.
4. **Замыкать Improvement Loop:** ошибка агента → lesson → обновление harness → синхронизация во все проекты.

**Ключевой вопрос:** Существует ли готовое решение или комбинация инструментов, которая решает эти 4 задачи лучше, чем самописный Orchestrator? Если нет — какие компоненты стоит взять из экосистемы, а какие писать с нуля?

### 2.2. Use-cases для валидации (проверь каждый)

Проведи исследование по следующим сценариям. Для каждого укажи: **Feasibility (1-5)**, **Existing Tools**, **Gaps**, **Recommendation**.

#### UC-1: Bootstrap W1 from zero
**Сценарий:** Пустая директория. Нужно создать полноценный MVP harness (17 модулей) за < 2 минуты и 1 команду.
- Какие инструменты существуют для multi-layer scaffolding (Copier, Cookiecutter, Plop, Hygen, Turbo Gen, Nx Gen, degit)?
- Может ли Copier (с `copier update`) заменить custom CLI для bootstrap + последующих обновлений?
- Как генерация текстовых инструкций (`AGENTS.md`) сочетается со scaffolding — нужен ли LLM-шаг или достаточно шаблонов?
- **Edge case:** Проект уже существует и имеет свой `.github/workflows/` — как merge'ить harness-CI без потери существующих jobs?

#### UC-2: Continuous Sync живого проекта
**Сценарий:** Проект W3 работает 2 недели. Вышла новая версия `@harness/lint` (новое правило) и обновился shared skills-репозиторий (новый skill).
- Как существующие инструменты (npm, git submodules, git subtree, symlinks) решают проблему **конфликтного merge**?
- Что делать, если агент вручную отредактировал symlink'нутый файл (например, `.claude/config.json`)?
- Как обеспечить **идемпотентность** `harness sync` (повторный запуск не ломает проект)?
- **Edge case:** Проект форкнут и имеет собственные overrides — как не затереть их при sync?

#### UC-3: Atomic Upgrade W1 → W4
**Сценарий:** Проект перешёл из фазы "стандартный UI" в фазу "security-first collaborative editor". Нужно добавить 8+ новых модулей (Trusted Types, CSP, CRDT, Security Evaluator, Snyk, Semgrep, new skills, new rules).
- Какие инструменты поддерживают **атомарные миграции** с rollback (Flyway для кода? Terraform для инфры?)?
- Как ведёт себя Copier при `copier update` через 3 мажорные версии шаблона — насколько надёжны его merge-алгоритмы?
- Нужен ли **transaction log** (как в БД) для harness-изменений, чтобы откатить `upgrade`?
- **Edge case:** Upgrade ломает существующие тесты проекта (например, новый linter rule делает CI красным). Какой механизм "breaking change adoption"?

#### UC-4: Error → Lesson → Harness Update → Propagation
**Сценарий:** Агент в проекте W2 использовал `any` вместо типа. Это должно привести к:
1. Созданию lesson в `docs/lessons/`.
2. Добавлению правила `no-any` в `@harness/lint`.
3. Синхронизации нового правила во **все** проекты W2+.
- Как организовать **центральный registry** для `@harness/*` пакетов (GitHub Packages, npm private, Verdaccio, JSR)?
- Как триггерить обновление 10+ проектов при публикации новой версии пакета? Dependabot? Renovate? Custom webhook?
- **Edge case:** Новое правило ломает legacy-проект W1, который ещё не готов к нему. Как semver и phase-gating решают это?

#### UC-5: Cross-project Skill Sharing
**Сценарий:** 5 проектов (W1, W3, W6, W8, W10) должны использовать общий пул skills, но каждый проект — только подмножество.
- Git submodules vs git subtree vs npm-пакеты со скриптами — что лучше для **частичного reuse** (проект W1 не должен видеть skills для W10)?
- Как symlink'ить только нужные skills, а не весь репозиторий?
- **Edge case:** Skill имеет project-specific параметры (например, `basePath` в Next.js конфиге). Как inject'ить переменные в SKILL.md?

#### UC-6: CI-only / Headless Mode
**Сценарий:** Harness должен работать в CI (GitHub Actions) без LLM-агента. Например, `harness doctor` в CI проверяет, что структура harness не сломана после human-коммита.
- Какие компоненты Orchestrator **не требуют** LLM (lint, typecheck, structure validation)?
- Можно ли сделать `harness` CLI полностью deterministic в CI-режиме (без вызовов к Claude API)?
- **Edge case:** CI должен проверить, что `AGENTS.md` соответствует текущей фазе (например, <50 строк в W1). Как валидировать текстовые инструкции без LLM?

### 2.3. Вопросы для глубокого анализа

Ответь на каждый вопрос с привлечением реальных инструментов, статей, документации и best practices.

#### Q1. State of the Art в multi-repo scaffolding
- Сравни Copier, Cookiecutter, Plop, Hygen, Nx Generators, Turbo Gen, Projen, degit.
- Какой из них поддерживает **обновление существующего проекта** из шаблона (не только bootstrap)?
- Есть ли инструмент, который комбинирует **шаблоны + пакеты + скрипты** в одном CLI?

#### Q2. Lock-файлы для non-code assets
- `package-lock.json` фиксирует NPM. Как зафиксировать версии git-submodule, copier-template, symlink-target, LLM-prompt?
- Существуют ли форматы lock-файлов для таких гибридных систем (Bazel lock, Nix lock, Pnpm workspace)?
- Как обеспечить **cross-platform** lock (macOS vs Linux CI)?

#### Q3. Agent-CLI Interaction Patterns
- Как существующие AI-инструменты (Claude Code, Cursor, Aider, OpenHands, Devin) взаимодействуют с CLI?
- Поддерживают ли они вызов произвольных CLI-команд как Tool Use?
- Какие ограничения есть у Claude Code при работе с monorepo / symlink'ами / git submodules?

#### Q4. Versioning & Distribution for Markdown/Text Assets
- Как версионировать `AGENTS.md`, path-rules, ADR-шаблоны? Это не код, но требует semver.
- Подходит ли NPM для дистрибуции markdown-файлов? Или лучше Git-репозиторий + tags?
- Как организовать **канареечное развёртывание** новой версии правила (сначала W10, потом W1)?

#### Q5. Rollback & Disaster Recovery
- Что если `harness upgrade --to w4` сломал проект? Как откатить **все** слои одновременно?
- Git stash + npm uninstall + copier revert — достаточно ли этого? Или нужен harness-level snapshot?
- Как работает `harness rollback` в контексте git-истории (не должен ли он быть git-tag-based)?

#### Q6. Performance at Scale
- 10 проектов × 22 skills × 40 linter rules. Каково время `harness sync`?
- Какие оптимизации возможны: lazy sync, differential update, content-addressable storage?
- Нужен ли локальный кэш `~/harness/.cache/` для skills и шаблонов?

### 2.4. Требования к выходу исследования

Структурируй ответ строго по следующему плану:

```markdown
# Отчёт: Валидация Harness Orchestrator

## 1. Executive Summary (3-5 абзацев)
- Общая оценка видения: реализуемо / частично / требует пересмотра.
- Главный риск и главная возможность.

## 2. Use-case Matrix
| Use-case | Feasibility (1-5) | Existing Tools | Gaps | Recommendation |
|----------|-------------------|----------------|------|----------------|
| UC-1 ... | 4/5 | Copier, Plop | ... | ... |
| ... | ... | ... | ... | ... |

## 3. Comparative Analysis (таблица)
| Критерий | Custom CLI | Copier | Nx/Turbo | Projen | Recommendation |
|----------|------------|--------|----------|--------|----------------|
| Bootstrap | ... | ... | ... | ... | ... |
| Sync/Update | ... | ... | ... | ... | ... |
| Atomic upgrade | ... | ... | ... | ... | ... |
| Agent-native | ... | ... | ... | ... | ... |
| Cross-project reuse | ... | ... | ... | ... | ... |

## 4. Gap Analysis
- 5-7 конкретных gap'ов в текущем видении (с указанием, в каком UC/Q они проявляются).
- Для каждого: Severity (High/Med/Low), Why it matters, Possible fix.

## 5. Improved Architecture Proposal
- Если видение требует изменений — предложи улучшенную архитектуру.
- Если видение корректно — предложи roadmap реализации с конкретными инструментами.
- Укажи, какие компоненты брать из экосистемы (off-the-shelf), а какие писать с нуля.

## 6. Risk Register
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| ... | High | High | ... |

## 7. Implementation Roadmap
| Phase | Deliverable | Tools | Owner (human/agent) | ETA |
|-------|-------------|-------|---------------------|-----|
| 0. PoC | ... | ... | ... | 1 неделя |
| 1. MVP | ... | ... | ... | 2 недели |
| ... | ... | ... | ... | ... |

## 8. Sources & References
- Список источников (документация инструментов, статьи, GitHub-репозитории).
```

### 2.5. Инструкции по выполнению

1. **Проверяй гипотезы реальными инструментами.** Если ты утверждаешь, что Copier поддерживает update — проверь это в документации Copier. Если говоришь про Nx Generators — приведи пример команды.
2. **Ищи edge cases.** Не ограничивайся happy path. Что происходит при конфликте? При отсутствии сети? При работе в monorepo?
3. **Оценивай trade-offs.** Не просто "X лучше Y", а "X лучше Y для sync, но хуже для bootstrap из-за Z".
4. **Будь конкретен.** Вместо "используй CI" — "используй GitHub Actions reusable workflow с matrix strategy для 10 проектов".
5. **Учитывай контекст пользователя.** Пользователь работает на MacBook Pro, использует React/Next.js/TypeScript, pnpm, Claude Code, GitHub. Не предлагай Windows-specific или Maven-based решения без явной мотивации.
6. **Если информация не найдена — честно скажи.** "Не удалось найти документацию по X, предполагаю Y, требуется валидация."

---

## Приложение: Глоссарий для агента-исследователя

| Термин | Определение |
|--------|-------------|
| **Harness** | Система инструкций и ограничений, управляющая поведением AI-агента в проекте. |
| **Layer 1** | Structured Instructions: текстовые правила, skills, feature lists. |
| **Layer 2** | Mechanical Enforcement: линтеры, хуки, CI, которые агент не может обойти. |
| **Layer 3** | Session Memory: ADR, lessons, git-протоколы, сохраняющие знания между сессиями. |
| **Phase W1-W10** | Этапы эволюции проекта: W1 = простой UI, W10 = сложная MFE-система. |
| **Driver** | Плагин Orchestrator, реализующий один способ переноса (npm, git, copier, agent). |
| **Lock-файл** | Фиксированный снапшот версий всех harness-компонентов для воспроизводимости. |
| **Improvement Loop** | Цикл: ошибка агента → lesson → обновление harness → распространение в проекты. |
| **Skill** | Переиспользуемый bundle инструкций (SKILL.md + скрипты + references) для агента. |
| **PreToolUse hooks** | Хуки Claude Code, блокирующие опасные операции (rm -rf, git push --force). |
