---
status: analyzing
mastery: beginner
tags: [github-actions, ci-cd, devops, automation, fsd-shopping-cart]
source: Обсуждение в рамках проекта fsd-shopping-cart + docs.github.com
created: 2026-05-14
updated: 2026-07-03
difficulty: intermediate
---

# 🧠 Topic: GitHub Actions — Workflow файлы и CI/CD архитектура

> [!abstract] Executive Summary (ELI5)
> GitHub Actions — это робот-дворник в вашем репозитории. Каждый раз, когда вы что-то меняете (пушите код, создаёте PR), робот просыпается, выполняет заранее написанный список задач (проверить, собрать, протестировать) и докладывает: всё чисто или где-то наведено. Список задач пишется в YAML-файле, а робот живёт на виртуальных машинах GitHub.

---

## 🎯 The "Why"

### Problem Solved
Ручное тестирование и деплой занимают часы, содержат человеческие ошибки и не масштабируются. GitHub Actions автоматизирует pipeline: lint → test → build → deploy — запускаясь автоматически на каждое событие в репозитории.

### Alternatives
| Инструмент | Что это | Почему не GitHub Actions |
|---|---|---|
| **GitLab CI** | Встроенный CI в GitLab | Если репозиторий на GitLab, а не GitHub |
| **CircleCI** | Облачный CI/CD | Больше контроля над ресурсами, но сложнее настройка |
| **Travis CI** | Один из первых облачных CI | Устаревший, дорогой, медленнее |
| **Jenkins** | Self-hosted CI | Полный контроль, но требует сервера и админа |
| **Drone CI** | Лёгкий self-hosted | Минималистичный, но меньше экосистемы |

### Trade-offs
- **За**: Встроен в GitHub, бесплатные минуты (2000/месяц для Free), огромный Marketplace actions, простой YAML.
- **Против**: Ограниченные ресурсы (2 vCPU, 7GB RAM на Free), vendor lock-in на GitHub, сложно debug'ить локально.

## 💡 Mental Model & Analogy

- **Analogy:** GitHub Actions — это конвейер на фабрике. Событие (push, PR) — это кнопка "старт". Workflow — это весь конвейер. Job — это отдельный участок конвейера (например, покраска или сборка). Step — это конкретное действие на участке (взять деталь, покрасить, проверить цвет). Action — это готовый робот-манипулятор, который можно взять с полки и поставить на конвейер (например, "робот для покраски красным" = `actions/setup-node`).

- **Visual/Diagram:**
```mermaid
graph TB
    subgraph "Event (Trigger)"
        E1[push to main]
        E2[pull request]
        E3[schedule: cron]
    end

    subgraph "Workflow (.github/workflows/*.yml)"
        W1[Job: Lint & Build]
        W2[Job: Test]
        W3[Job: Deploy]
    end

    subgraph "Runner (VM)"
        R1[ubuntu-latest<br/>2 vCPU / 7 GB RAM]
        R2[macos-latest]
    end

    E1 --> W1
    E2 --> W1
    W1 -->|needs: [lint]| W2
    W2 -->|needs: [test]| W3
    W1 --> R1
    W2 --> R1
    W3 --> R2
```

## 🛠️ Core Principles & Code Snippets

### 1. Анатомия workflow файла

Файл лежит в `.github/workflows/<name>.yml`. Вот полная структура с комментариями:

```yaml
# ───────────────────────────────────────────────
# 1. Имя workflow (видно в UI GitHub → Actions)
# ───────────────────────────────────────────────
name: CI

# ───────────────────────────────────────────────
# 2. Триггеры (когда запускать)
# ───────────────────────────────────────────────
on:
  push:
    branches: [main, develop]
    paths: ['src/**', '!src/**/*.test.ts']  # ← только при изменении src, игнор тестов
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]
  schedule:
    - cron: '0 0 * * 1'  # ← каждый понедельник в 00:00 UTC
  workflow_dispatch:  # ← ручной запуск из UI GitHub

# ───────────────────────────────────────────────
# 3. Глобальные переменные окружения
# ───────────────────────────────────────────────
env:
  NODE_VERSION: '22'
  CI: true

# ───────────────────────────────────────────────
# 4. Jobs (задачи, выполняемые параллельно или последовательно)
# ───────────────────────────────────────────────
jobs:
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - run: npm ci
      - run: npm run lint

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: lint  # ← ждёт завершения job 'lint'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - run: npm ci
      - run: npm run test:unit

  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: [lint, test]  # ← ждёт ОБЕ jobs
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - run: npm ci
      - run: npm run build

  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'  # ← только для main ветки
    environment: production  # ← защищённое окружение с approve
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: npm run deploy
```

### 2. Секция `on:` — триггеры подробно

```yaml
on:
  push:
    branches: [main, 'release/**']
    tags: ['v*']
    paths:
      - 'src/**'
      - 'package.json'
      - '!**.md'  # ← исключить markdown файлы
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened, ready_for_review]
```

| Тип триггера | Когда срабатывает | Типичное использование |
|---|---|---|
| `push` | Любой git push | Запуск CI на коммиты |
| `pull_request` | События PR | Проверка PR перед merge |
| `schedule` | По cron | Ночные сборки, проверка зависимостей |
| `workflow_dispatch` | Ручной запуск из UI | Деплой по кнопке, rerun конкретного workflow |
| `workflow_call` | Вызов из другого workflow | Реюзабельные workflow |
| `release` | Создание релиза | Публикация пакетов |

**Фильтры `paths` и `paths-ignore`:**

```yaml
on:
  push:
    paths:
      - 'src/**'        # ← только при изменении в src/
      - 'package.json'  # ← или package.json
    paths-ignore:
      - '**.md'         # ← игнорировать markdown
      - 'docs/**'       # ← игнорировать docs/
```

Это экономит минуты CI: если разработчик поправил README — незачем гонять весь test suite.

### 3. Jobs — изолированные задачи

```yaml
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    timeout-minutes: 10  # ← job упадёт, если выполняется дольше 10 минут
    strategy:
      matrix:
        node-version: [18, 20, 22]  # ← запустить job 3 раза с разными Node
        os: [ubuntu-latest, windows-latest]
    # ↑ Всего будет 3 × 2 = 6 job'ов (все комбинации)
    
    env:
      MY_VAR: "hello"  # ← переменная только для этой job
    
    outputs:
      build-id: ${{ steps.build.outputs.id }}
    # ↑ Передать данные из этой job в другие
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm run build
```

| Параметр job | Что делает | Пример |
|---|---|---|
| `runs-on` | Выбор ОС runner'а | `ubuntu-latest`, `macos-latest`, `windows-latest`, `self-hosted` |
| `needs` | Зависимость от других jobs | `needs: [lint, test]` |
| `if` | Условие выполнения | `if: github.ref == 'refs/heads/main'` |
| `strategy.matrix` | Матрица параметров (параллельный запуск) | `node: [18, 20, 22]` |
| `strategy.fail-fast` | Остановить все job'ы матрицы при первом падении | `fail-fast: false` |
| `timeout-minutes` | Максимальное время выполнения | `timeout-minutes: 15` |
| `env` | Переменные окружения job | `NODE_ENV: test` |
| `outputs` | Результаты для передачи другим job'ам | `build-id: ${{ steps.x.outputs.y }}` |
| `permissions` | Права доступа (token) | `permissions: contents: read` |

### 4. Steps — шаги внутри job

```yaml
steps:
  # ── Шаг 1: Имя + Action из Marketplace ──
  - name: Checkout code
    uses: actions/checkout@v4
    with:
      fetch-depth: 0
    # ↑ uses — подключить готовый Action (как npm-пакет, но для CI)

  # ── Шаг 2: Имя + произвольная команда ──
  - name: Install dependencies
    run: npm ci
    # ↑ run — выполнить shell-команду

  # ── Шаг 3: Несколько команд ──
  - name: Build and test
    run: |
      echo "Starting build..."
      npm run build
      npm run test
    # ↑ | — многострочный скрипт (YAML literal block)

  # ── Шаг 4: Условный шаг ──
  - name: Deploy (only on main)
    if: github.ref == 'refs/heads/main'
    run: npm run deploy
    # ↑ if — шаг выполнится только если условие истинно

  # ── Шаг 5: Использование output предыдущего шага ──
  - name: Generate build ID
    id: build  # ← ID шага для обращения к его outputs
    run: echo "id=$(date +%s)" >> $GITHUB_OUTPUT

  - name: Use build ID
    run: echo "Build ID is ${{ steps.build.outputs.id }}"
```

**Важно**: шаги выполняются **последовательно** внутри одного runner'а. Если шаг падает (exit code ≠ 0) — все последующие шаги по умолчанию **не выполняются**.

### 5. Contexts — данные о событии и окружении

GitHub предоставляет объекты с информацией. Самые полезные:

```yaml
${{ github.ref }}           # refs/heads/main или refs/pull/42/merge
${{ github.sha }}           # SHA коммита
${{ github.event_name }}    # push, pull_request, schedule...
${{ github.actor }}         # username того, кто triggered
${{ github.repository }}    # owner/repo
${{ github.run_id }}        # уникальный ID запуска workflow
${{ github.job }}           # имя текущей job
${{ runner.os }}            # Linux, macOS, Windows
```

**Пример условия:**
```yaml
if: github.event_name == 'pull_request' && github.event.pull_request.draft == false
```

### 6. Secrets и Variables

**Secrets** (шифрованные, не видны в логах):
```yaml
# Настройка: GitHub → Settings → Secrets and variables → Actions
# Использование:
${{ secrets.CHROMATIC_PROJECT_TOKEN }}
${{ secrets.DEPLOY_KEY }}
${{ secrets.NPM_TOKEN }}
```

**Variables** (нешифрованные, видны в логах):
```yaml
# Настройка: GitHub → Settings → Secrets and variables → Variables
# Использование:
${{ vars.API_URL }}
${{ vars.APP_NAME }}
```

**Env (переменные окружения в workflow):**
```yaml
env:
  NODE_VERSION: '22'

jobs:
  build:
    env:
      API_URL: ${{ vars.API_URL }}
    steps:
      - run: echo $NODE_VERSION  # ← 22
      - run: echo $API_URL       # ← значение из Variables
```

### 7. Actions Marketplace — готовые строительные блоки

Action — это переиспользуемый unit (как npm-пакет), который можно подключить через `uses:`.

```yaml
# Официальные Actions от GitHub
- uses: actions/checkout@v4              # Клонирование репозитория
- uses: actions/setup-node@v4            # Установка Node.js
- uses: actions/setup-python@v5          # Установка Python
- uses: actions/cache@v4                 # Кеширование
- uses: actions/upload-artifact@v4       # Сохранение файлов между job'ами
- uses: actions/download-artifact@v4     # Загрузка сохранённых файлов

# Популярные community Actions
- uses: chromaui/action@latest           # Chromatic visual regression
- uses: docker/login-action@v3           # Логин в Docker Hub
- uses: aws-actions/configure-aws-credentials@v4  # AWS credentials
```

**Синтаксис `uses`:**
```yaml
uses: owner/repo@ref
# ref может быть:
#   v4          — floating tag (обновляется)
#   v4.1.2      — pinned tag (стабильно)
#   abc1234     — commit SHA (максимально стабильно)
#   main        — ветка (опасно, может сломаться)
```

**Рекомендация для production**: pin'ить конкретную версию или SHA:
```yaml
uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1
```

### 8. Artifacts — передача файлов между job'ами

Job'ы изолированы. Чтобы передать файл из `build` в `deploy`:

```yaml
jobs:
  build:
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-files        # ← имя артефакта
          path: dist/              # ← что сохранить
          retention-days: 7        # ← хранить 7 дней

  deploy:
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-files        # ← имя артефакта
          path: dist/              # ← куда распаковать
      - run: ls dist/              # ← файлы доступны
```

### 9. Reusable Workflows — DRY для CI

Если несколько проектов используют одинаковый pipeline:

```yaml
# .github/workflows/reusable-build.yml
name: Reusable Build
on:
  workflow_call:  # ← вызывается из других workflow
    inputs:
      node-version:
        required: true
        type: string
    secrets:
      npm-token:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm run build
```

```yaml
# .github/workflows/app.yml
name: App CI
on: push
jobs:
  call-reusable:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '22'
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

### 10. Environments — защищённые окружения

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # ← защищённое окружение
    steps:
      - run: npm run deploy
```

**Что даёт Environment:**
- Required reviewers (нужен approve перед деплоем)
- Deployment protection rules (таймауты, branch restrictions)
- Secrets scoped to environment (разные токены для staging и production)
- Deployment history в UI GitHub

### 11. Concurrency — отмена устаревших запусков

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

**Эффект**: если разработчик пушит 3 коммита подряд, запускаются 3 workflow. С `cancel-in-progress: true` первые 2 автоматически отменяются. Экономит минуты CI.

### 12. Сравнение подходов: один workflow vs несколько

| Подход | Плюс | Минус |
|---|---|---|
| **Один `ci.yml`** | Просто, всё в одном месте | Длинный файл, Chromatic тормозит lint |
| **Разные workflow** (`ci.yml`, `chromatic.yml`, `deploy.yml`) | Параллельность, isolation | Много файлов |
| **Reusable workflow** | DRY, единый стандарт | Сложнее debug |

**Рекомендация**: разделять по доменам — `ci.yml` (lint/test/build), `chromatic.yml` (visual regression), `deploy.yml` (staging/prod).

## ⚠️ Pitfalls & Anti-patterns

### Common Mistakes
1. **Хардкод секретов в коде** — никогда не пишите `password: abc123` в workflow. Используйте `secrets.*`.
2. **`npm install` вместо `npm ci`** — `npm install` может обновить зависимости, нарушая воспроизводимость. В CI только `npm ci`.
3. **Забыть `fetch-depth: 0` для Chromatic** — TurboSnap отключается, сжигается квота.
4. **Не pin'ить версии Actions** — `uses: actions/checkout@main` может сломаться без предупреждения. Используйте теги или SHA.
5. **Job'ы без `needs` там, где нужна последовательность** — deploy может запуститься до окончания тестов.
6. **Не использовать `concurrency`** — старые workflow продолжают бежать, тратя минуты.
7. **Хранить artifacts вечно** — GitHub даёт 500 MB бесплатно. Без `retention-days` артефакты копятся.
8. **Матрица без `fail-fast: false`** — если одна версия Node упала, остальные отменяются. Для диагностики это мешает.

### When NOT to use
- **Самописный runner на старом железе** — проще использовать GitHub-hosted runners для open-source.
- **Workflow длиннее 30 минут** — возможно, стоит оптимизировать (кеширование, TurboSnap).
- **CI только ради CI** — если проект из 1 файла, хватит `npm test` локально.

## 🔗 Deep Connections

- **Parent Topic:** [[CI-CD Pipeline]], [[DevOps]], [[Automation]]
- **Related Concepts:**
  - [[Chromatic]] — визуальная регрессия через GitHub Actions
  - [[Storybook]] — компоненты для тестирования в CI
  - [[Docker]] — контейнеризация для self-hosted runners
  - [[Design Tokens]] — изменения токенов триггерят CI pipeline
  - [[Feature-Sliced Design]] — FSD требует lint:arch в CI (Steiger)
  - [[NPM Scripts]] — CI вызывает `npm run lint`, `npm run build`
  - [[Git Workflow]] — branch protection rules + required checks
- **Upstream:** [[GitHub Platform]] — хостинг кода и CI/CD в одном месте
- **Downstream:** [[Deployment Strategies]] — blue-green, canary, rolling deploy через CI

## ❓ Active Recall Questions

> [!faq]- Почему в CI используем `npm ci`, а не `npm install`?
> `npm ci` строго читает `package-lock.json` и не обновляет его. `npm install` может обновить версии зависимостей, нарушая воспроизводимость сборки на runner'е.

> [!faq]- Зачем `needs:` в job'ах?
> `needs` создаёт зависимость между job'ами. Job с `needs: [lint, test]` не запустится, пока `lint` и `test` не завершатся успешно. Это гарантирует, что deploy произойдёт только после прохождения проверок.

> [!faq]- В чём разница между `secrets` и `vars` в GitHub?
> `secrets` — шифрованные, не видны в логах (токены, пароли). `vars` — открытые переменные, видны в логах (URL, имена). `vars` безопаснее для нечувствительных данных.

> [!faq]- Зачем `concurrency` с `cancel-in-progress: true`?
> Если разработчик пушит 3 коммита подряд, запускаются 3 workflow. `cancel-in-progress: true` автоматически отменяет устаревшие запуски, экономя минуты CI.

> [!faq]- Что делает `strategy.matrix`?
> Запускает одну и ту же job несколько раз с разными параметрами (например, Node 18/20/22). Полезно для тестирования совместимости.

> [!faq]- Почему `actions/checkout` без `fetch-depth: 0` ломает TurboSnap?
> TurboSnap анализирует git history, чтобы понять, какие stories изменились. `fetch-depth: 1` клонирует только последний коммит — истории нет, анализ невозможен.

> [!faq]- Как передать данные из одной job в другую?
> Через `outputs` в job + `needs.<job>.outputs` в зависимой job. Для файлов используются `actions/upload-artifact` и `actions/download-artifact`.

> [!faq]- Какие триггеры существуют в GitHub Actions?
> `push`, `pull_request`, `schedule` (cron), `workflow_dispatch` (ручной), `workflow_call` (реюзабельный), `release`. Каждый запускает workflow на определённое событие.

---
## 📝 How to use this template effectively
1. **Never Copy-Paste:** If you find yourself copying documentation, stop. Read, close the tab, and write it from memory.
2. **Iterate:** Fill the 'ELI5' and 'Analogy' first. If you can't explain it simply, you don't understand it yet.
3. **The "Why" is King:** Seniority is defined by knowing *when* and *why* to use a tool, not just *how*.
