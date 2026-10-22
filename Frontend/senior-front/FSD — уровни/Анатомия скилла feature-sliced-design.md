---
created: '2026-09-06'
difficulty: advanced
mastery: growing
source: 'Официальный скилл feature-sliced-design v2.1 — github.com/feature-sliced/skills
  (локальная копия: [[скилл — исходник]]; README репозитория + содержимое SKILL.md
  и 9 references). Назначение заметки — доскональное знание скилла и точки его
  адаптации под специфические задачи.'
status: analyzing
tags:
- frontend
- senior-front
- code-organization
- fsd
- agent-skills
updated: '2026-09-06'
---

# Анатомия скилла feature-sliced-design

> Официальный скилл FSD-команды ([github.com/feature-sliced/skills](https://github.com/feature-sliced/skills)) обучает агента применять FSD v2.1. Цель этой заметки — знать его устройство настолько, чтобы уверенно адаптировать под свои задачи (своя студия, свой стек, свои строгости).

## 1. Как скилл устроен: progressive disclosure

Два уровня чтения:

1. **`SKILL.md` (508 строк) — входная точка.** Агент читает только его. Внутри: философия и слои (§1), decision framework (§2), таблица размещения (§3), правила MUST (§4), рекомендации SHOULD (§5), анти-паттерны AVOID (§6), разрешение кросс-импортов (§7), сегменты (§8), гайд по shared (§9) и — ключ к экономии контекста — §10: условные ссылки.
2. **`references/` — 9 файлов, ленивая загрузка.** «Read the following reference files **only when the specific situation applies. Do not preload all references.**» (Читай reference только когда применила ситуация. Не подгружай всё заранее.)

| Файл | Строк | Триггер (§10) |
|---|---|---|
| `layer-structure.md` | 571 | реорганизация существующих папок, layout страниц, группы слайсов |
| `growth-walkthrough.md` | 194 | новый проект с нуля, «заработал ли слой», показать рост структуры |
| `cross-import-patterns.md` | 405 | кросс-импорты, оценка @x, выбор стратегии A–D |
| `excessive-entities.md` | 242 | создать/удалить entity, слой entities нужен ли вообще, CRUD |
| `asset-handling.md` | 181 | размещение статики: картинки, иконки, шрифты, PDF |
| `migration-guide.md` | 351 | v2.0 → v2.1, custom → FSD, вывод widgets/processes |
| `framework-integration.md` | 527 | Next.js / React Router / Nuxt / Vite / Astro, алиасы, env-API |
| `auth-and-api.md` | 427 | токены, login flow, DTO, где живут request-функции |
| `state-management.md` | 386 | Redux, TanStack Query (factories, infinite, Suspense) |

Это и есть архитектура скилла: **короткое всегда-в-контексте ядро + справочники по ситуации**. Экономия контекста — конструктивная особенность, не мелочь.

## 2. Дисциплина норм: MUST / SHOULD / AVOID

- **§4 Architectural rules (MUST)** — 5 правил: импорт только вниз; публичный API через index (для shared — по сегментам; папка-как-API при бандлинге); запрет кросс-импорта слайсов одного слоя; доменные имена файлов (не `types.ts`/`utils.ts`); никакого бизнеса в shared. Нарушение — только осознанно и с комментарием/ADR.
- **§5 Recommendations (SHOULD)** — pages first; консервативность с entities; минимальные слои; линтер Steiger.
- **§6 Anti-patterns (AVOID)** — преждевременные entities; CRUD в entities; user-entity ради auth; абьюз @x; извлечение single-use; god slices; верхнеуровневый assets.

## 3. Философия (что скилл считает главным)

> «**Start simple, extract when needed.**» + extraction rule (3 условия) + Golden Rule «When in doubt, keep it in `pages/`.» + «A layer is earned by a stable responsibility that needs one home, not by a count of how many places use something.»

Заголовок SKILL.md фиксирует источник и регулятор строгости:

> «**Source**: [fsd.how](https://fsd.how) | Strictness can be adjusted based on project scale and team context.»

Отношение к официальным докам — не копия: «It follows the official FSD v2.1 documentation but is not a verbatim copy» — при конфликте гайдов скилл выбирает один вариант и говорит какой; если интеграционный гайд отстал от документации фреймворка — следует фреймворку; отклонения помечаются инлайн (пример — примечание про `publicDir` в Astro внутри `asset-handling.md`).

## 4. Инженерка репозитория

- **`evals/cases.json`** — регрессионные кейсы размещения: сценарий → ожидаемое место. Скилл не просто текст, а тестируемый артефакт.
- **Валидатор** (`.github/scripts/validate-skills.mjs`, CI на каждый PR): тело SKILL.md ≤ 500 строк, `description` ≤ 1024 символов — ограничения заставляют держать ядро сжатым, детали — в references.
- **Steiger** — официальный линтер FSD, рекомендован §5-4: `insignificant-slice` (слайс без ссылок/с одной — слить выше), `excessive-slicing` (слишком много слайсов — слить/сгруппировать).

## 5. Точки адаптации под специфические задачи

1. **Сузить/расширить description-триггеры.** Frontmatter `description` решает, когда агент вообще возьмёт скилл. Внутри команды можно добавить свои триггеры («монорепо X», «наш дизайн-системный слой»).
2. **Добавить свой reference.** Формат тот же: один файл — одна ситуация; зарегистрировать в §10 условной ссылкой. Кандидаты из курса: работа с дизайн-токенами (глава 4.1), харнесс (модуль 2), i18n-процессы.
3. **Настроить строгость.** Скилл сам легитимизирует это («Strictness can be adjusted…»; §7: ранний продукт — допускаем кросс-импорты, fintech — строже). Адаптация = явная правка §4/§7 с комментарием, а не молчаливое послабление.
4. **Сменить биасы — осторожно.** pages-first и extraction rule — несущая конструкция; для thick-client продукта можно сместить порог entities, но решение фиксировать письменно (у нас это ADR-практика курса).
5. **Обрезка для лёгких агентов:** только SKILL.md без references — валидный режим (сам скилл так и работает до первой ситуации).
6. **Синхронизация с upstream:** официальный репозиторий живёт (evals, CI); при адаптации хранить форк-пометки, чтобы обновлять ядро без конфликтов.

## 6. Скилл и курс

- Уровни [[Оглавление|FSD — уровни]] — это шаги decision framework скилла, развёрнутые в порядок роста проекта; каждый уровень статьи цитирует и главу 1.2, и SKILL.md.
- Визуализатор (`tools/fsd-visualizer/`) проигрывает те же решения на Conduit.
- Отдельно: скилл и глава расходятся в деталях (shared/auth, 6 слоёв с widgets, консервативность к entity-UI) — оба взгляда зафиксированы в блоках «Глава ↔ скилл» каждого уровня.

## Связи

- [[FSD — кейсы размещения (шпаргалка)]] · [[FSD — слои и правило импортов]] · [[FSD — кросс-импорты и стратегии]]
- Локальная копия: [[скилл — исходник]] (папка с SKILL.md + README + 9 references).

🔗 **Якорь:** официальный репозиторий скилла — https://github.com/feature-sliced/skills (README; feature-sliced-design/SKILL.md и references/ — локальные копии процитированы).
