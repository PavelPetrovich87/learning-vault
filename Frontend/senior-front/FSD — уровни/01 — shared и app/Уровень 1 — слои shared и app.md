---
created: '2026-09-06'
difficulty: intermediate
mastery: growing
source: 'Глава 1.2 курса «Фронтенд senior+ в агентную эра» (якорь [FSD] — Reference
  → Layers) + официальный скилл feature-sliced-design v2.1 ([[скилл — исходник]],
  SKILL.md §1–2, §5, §9; references/asset-handling.md, references/auth-and-api.md).
  🟡 — course-кейсы.'
status: analyzing
tags:
- frontend
- senior-front
- code-organization
- fsd
- layers
- shared
- app
updated: '2026-09-06'
---

# Уровень 1 — слои shared и app

> Первый этап роста проекта. Фундамент: код **без бизнес-контекста** (`shared`) и дела **всего приложения** (`app`). С этих двух слоёв FSD начинается, и ими же часто заканчивается: «большинству проектов хватает Shared + Pages + App».

## Зачем этот уровень первый

FSD разрешает начинать с минимума — три слоя, и это уже валидный FSD:

> «Не обязательно использовать все слои; минимум большинства проектов — **Shared + Pages + App**.» — глава 1.2, §3 ([FSD] Reference → Layers)

> «Most projects can start with only `shared/`, `pages/`, and `app/`. … Do not create empty layer folders "just in case."» (Большинство проектов может начать с shared, pages и app. Не создавай пустые папки слоёв «на всякий случай».) — официальный скилл, SKILL.md §1

Ядро всего уровня — принцип скилла:

> «**Start simple, extract when needed.**» (Начинай просто, извлекай, когда понадобится.) — SKILL.md §1

## Семантика слоёв (по главе)

**Shared** — фундамент остального приложения:

> «Место для связей с внешним миром — бэкенды, сторонние библиотеки, окружение. Также место для собственных высоко самодостаточных библиотек. **Не содержит слайсов (бизнес-доменов здесь не существует)**, все файлы могут ссылаться друг на друга.» — глава 1.2, §3.2 [FSD]

Сегменты shared: `api` (клиент, запросы, транспортные типы), `ui` (UI-кит), `lib` (библиотеки одной области ответственности — «не helpers и не utilities»), `config`, `routes`, `i18n`.

Про `ui` — важная граница:

> «Компоненты этого слоя не должны содержать бизнес-логику, но им **допустимо** быть „бизнес-тематическими“ (например, логотип компании или каркас страницы). Компоненты с UI-логикой разрешены (автокомплит, строка поиска).» — глава 1.2, §3.2 [FSD]

**App** — дела всего приложения:

> «Всевозможные дела уровня всего приложения — и в техническом смысле (context-провайдеры), и в бизнес- (аналитика). Обычно не содержит слайсов — сразу сегменты: `routes`, `store`, `styles`, `entrypoint`.» — глава 1.2, §3.2 [FSD]

И критерий для «большого» UI:

> «UI уровня всего приложения (глобальные лейауты) — в `app`.» — глава 1.2, §3.2 (callout про widgets) [FSD]

## В официальном скилле

Скилл формулирует уровень как **Шаг 2** и **Шаг 5** decision framework:

> «**Step 2: Is it reusable infrastructure with no business logic?** The official layer reference draws the line for Shared like this: no business logic, but business-themed is fine (a company logo, a page layout), and so is UI logic (autocomplete, a search bar). Exchanging data with the backend and CRUD boilerplate are not business logic either. **Business logic is a rule the product enforces on its own data, such as applying a discount to an order.**» (Шаг 2: это переиспользуемая инфраструктура без бизнес-логики? … Бизнес-логика — правило, которое продукт налагает на собственные данные, например применение скидки к заказу.) — SKILL.md §2

Разветвление шага — по типу инфраструктуры:

> «UI components → `shared/ui/` · Utility functions → `shared/lib/` · API client, route constants → `shared/api/` or `shared/config/` · Auth tokens, session management → `shared/auth/` · CRUD once several slices call it → `shared/api/`» — SKILL.md §2, Step 2

> «**Step 5: Is it app-wide configuration?** Global providers, router, theme → `app/`» (Шаг 5: это конфигурация всего приложения? Глобальные провайдеры, роутер, тема → app.) — SKILL.md §2

Жёсткое правило уровня (MUST):

> «Shared contains only infrastructure: UI kit, utilities, API client setup, route constants, assets. Business calculations, domain rules, and workflows belong in `entities/` or higher layers.» (Shared содержит только инфраструктуру… Бизнес-вычисления, доменные правила и workflows — в entities или выше.) — SKILL.md §4-5

**Reference-файлы этого уровня** (скилл грузит их по ситуации, SKILL.md §10):
- `asset-handling.md` — «when deciding where to place static assets»: ассеты следуют тем же правилам владения, что и код — рядом с модулем, разделяющим их жизненный цикл; глобальные стили/шрифты → `app/styles`, `app/fonts`; верхнеуровневый `assets/`-сегмент не рекомендуется.
- `auth-and-api.md` — «when implementing authentication, type definitions, or API request handling»: токены, login flow, DTO, request-функции.

### Глава ↔ скилл

- **Согласованы:** нет бизнес-логики в shared; «бизнес-тематическое» допустимо; сегменты ui/lib/api/config; app-сегменты.
- **Скилл строже в терминах:** даёт определение бизнес-логики («правило продукта о своих данных») и явно относит CRUD-boilerplate и обмен данными с бэкендом к «не бизнес-логике» — это снимает споры о пограничных случаях.
- **Скилл добавляет сегмент `shared/auth`** (токены, сессия, login-утилиты): в справочнике слоёв главы такого сегмента нет (там api/ui/lib/config/routes/i18n); скилл опирается на официальный Auth guide. Транспортные типы (`ProductDTO`) скилл спокойно держит в `shared/api` — «a type that exists because of an API response» не заслуживает слоя (это раскроется на уровне 3).
- **Именование файлов** (правило 4-4 скилла = «сегменты не по сущности» из главы §5): имена по домену (`format-price.ts`), не по технической роли (`utils.ts`).

## Границы уровня

- Код нужен **одной странице** → слайс страницы (уровень 2). Один потребитель — не повод для shared.
- Код несёт **бизнес-контекст** (правило продукта) → уровни 3–4. Даже если «общий».
- Ассет принадлежит одному слайсу → рядом с его потребителем внутри слайса, не в shared «на всякий случай».

## Алгоритм

![[1-алгоритм-shared-app.drawio]]

Порядок вопросов — из decision framework скилла (Шаги 1 → 2 → 5). Ромб «бизнес-логика?» отсекает главное: правила продукта не живут на фундаменте.

## Примеры

### Пример 1 (простой): кнопка магазина

![[1-пример-button.drawio]]

Кнопка на трёх страницах, без бизнес-логики → Шаг 2 → `shared/ui`. Наружу — только через `index.ts` сегмента.

### Пример 2 (средний): formatPrice и plural

![[1-пример-форматтер-цен.drawio]]

«12 990 ₽» и «3 товара» — утилиты → `shared/lib`, файлы с доменными именами. Анти-пример: `utils.ts`-ящик.

### Пример 3 (сложный): шапка видеоплатформы

![[1-пример-шапка.drawio]]

Простая шапка (логотип + навигация) — «каркас страницы», допустимый в `shared/ui`. Шапка-композиция с аватаром пользователя — «app-wide layout» → `app`.

## Итог уровня

- `shared` — инфраструктура без бизнес-логики; сегменты по назначению; публичный API на сегмент.
- `app` — провайдеры, роутер, тема, глобальные стили/шрифты, аналитика.
- Критерий «общности» — не «сколько мест используют», а «нет бизнес-контекста» (второй потребитель сам по себе слоя не даёт — это extraction rule, уровень 3).

## Связи

- Дальше: [[Уровень 2 — слой pages]] — всё, что нужно одному экрану.
- Теория: [[FSD — слои и правило импортов]] (включая «куда общее для нескольких слайсов»), [[FSD — сегменты]].
- Скилл: [[скилл — исходник]] (SKILL.md §1–2, §4–5, §9; references/asset-handling.md, references/auth-and-api.md).

🔗 **Якоря:** [FSD] — feature-sliced.design, Reference → Layers: https://feature-sliced.design/docs/reference/layers · Официальный скилл: https://github.com/feature-sliced/skills (SKILL.md, FSD v2.1)
