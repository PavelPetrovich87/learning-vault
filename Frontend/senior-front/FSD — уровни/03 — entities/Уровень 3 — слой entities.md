---
created: '2026-09-06'
difficulty: intermediate
mastery: growing
source: 'Глава 1.2 курса «Фронтенд senior+ в агентную эру» (якорь [FSD] — Reference
  → Layers, FAQ) + официальный скилл feature-sliced-design v2.1 ([[скилл — исходник]],
  SKILL.md §1–2, §5-2, §6; references/growth-walkthrough.md, references/excessive-entities.md).
  🟡 — course-кейсы.'
status: analyzing
tags:
- frontend
- senior-front
- code-organization
- fsd
- layers
- entities
updated: '2026-09-06'
---

# Уровень 3 — слой entities

> Момент, когда копия перестаёт быть копией. Появляется общее с бизнес-контекстом — «понятие реального мира», которому нужен **один дом**. Это самый тонкий уровень: и недо-извлечение (дублирование расползается), и пере-извлечение (кладбище пустых сущностей) — ошибки.

## Семантика слоя (по главе)

> «Слайсы этого слоя представляют **понятия реального мира**, с которыми работает проект. Обычно это термины, которыми бизнес описывает продукт: соцсеть работает с сущностями User (пользователь), Post (пост), Group (группа). Слайс сущности может содержать хранилище данных (`model`), схемы валидации данных (`model`), функции API-запросов, относящиеся к сущности (`api`), и визуальное представление сущности в интерфейсе (`ui`). Визуальное представление не обязано быть законченным UI-блоком — его главная цель: **переиспользовать один и тот же вид в нескольких страницах**, а разная бизнес-логика может подключаться через пропсы или слоты.» — глава 1.2, §3.2 [FSD]

Почему не `shared` — определение shared этого не позволяет:

> «Не содержит слайсов (**бизнес-доменов здесь не существует**)» — глава 1.2, §3.2 [FSD]. Даже если перенести бизнес-тип вниз «чтобы стрелки не краснели», самый нижний слой узнаёт о бизнесе — правило импортов этого не ловит, критерий семантический.

Критерий «существительное» и «сложность — не критерий»: корзина — термин бизнеса → `entities/cart`; действия над ней (`add-to-cart`) — фичи (уровень 4).

## В официальном скилле

**Extraction rule** — сердце уровня (уже цитировалась в [[Уровень 1 — слои shared и app]]):

> «Extract only when all three conditions hold: 1. The same code is used in multiple places right now, not hypothetically. 2. It has a reason to change that is independent of any one consumer. 3. The boundary has a focused responsibility.» (Извлекай, только когда выполняются все три условия: код реально используется в нескольких местах; у него своя причина меняться; у границы сфокусированная ответственность.) — SKILL.md §1

Сам уровень — **Шаг 4**:

> «**Step 4: Is it a business domain model that several consumers share, with a focused responsibility and a reason to change of its own?** Yes → `entities/`. Uncertain, single use, or speculative reuse → keep in the page.» (Шаг 4: это бизнес-доменная модель, которой делятся несколько потребителей…? Да → entities. Не уверен, один потребитель, спекулятивное переиспользование → оставь в странице.) — SKILL.md §2

И рекомендация осторожности (SHOULD):

> «**5-2. Be conservative with entities.** The entities layer is highly accessible (almost every other layer can import from it), so changes propagate widely. 1. **Start without entities.** `shared/` + `pages/` + `app/` is valid FSD. Thin-client apps rarely need entities. … 3. Business logic does not automatically require an entity. Keeping types in `shared/api` and logic in the current slice's `model/` segment may be sufficient. 4. **CRUD is infrastructure, not entities.** … 5. Place auth data in `shared/auth/` or `shared/api/`.» — SKILL.md §5-2

`references/excessive-entities.md` закрывает уровень («when deciding whether to create or remove an entity … evaluating whether the entities layer is needed at all»): тонкие клиенты (бэкенд держит обработку) — обычно без entities; толстые — кандидаты; DTO живёт в `shared/api` и не едет в сущность вслед за логикой; `user`-сущность только ради обёртки login-ответа — анти-паттерн (токены → `shared/auth`); несколько сущностей, завязанных `@x`-цепочками, — сигнал слить границы.

**Момент появления слоя** — Snapshot 2 walkthrough: маркетинг меняет смысл «on sale» (цена ниже прейскурантной **и** товар в наличии), одну копию правила обновили, вторую забыли — страницы противоречат друг другу:

> «This is the signal. The two copies are the same rule, they must agree, and they no longer do. Check the extraction rule … All three hold, so the rule gets one home (Step 4).» (Это сигнал. Две копии — одно правило, они обязаны соглашаться и больше не соглашаются… Все три условия выполняются — правило получает один дом.) — references/growth-walkthrough.md, Snapshot 2

> «**What did not move.** The transport type `ProductDTO` stays in `shared/api`. … The badge and the cards stay in their pages: they are UI, and Section 6 warns against adding UI to entities until there is a reason. **The entity is one file and an index. That is enough.**» (Транспортный тип ProductDTO остаётся в shared/api… Бейдж и карточки остаются в своих страницах… Сущность — один файл и index. Этого достаточно.) — references/growth-walkthrough.md, Snapshot 2

### Глава ↔ скилл

- **Согласованы:** сущность = «понятие реального мира» ⇔ «business domain model»; пропсы/слоты для разной бизнес-логики; осторожность против преждевременных сущностей (глава: «извлекается по ходу», принцип туториала уровня shared).
- **Скилл консервативнее к UI сущностей:** глава называет переиспользование вида «главной целью» `ui`-сегмента; скилл добавляет предостережение — «Be cautious adding UI to entities», UI сущности импортируется только сверху, никогда из других сущностей (иначе кросс-импорты, уровень 5). Практический порядок скилла: сначала модель заслуживает дом (`model` + `index`), UI переезжает, когда для него есть причина.
- **Скилл добавляет:** тонкие/толстые клиенты; «CRUD is infrastructure, not entities»; DTO не следует за логикой; entities для auth-данных — анти-паттерн.
- **Порядок шагов:** скилл проверяет features (Шаг 3) **раньше** entities (Шаг 4) — глагол спрашивается первым, существительное — уточнением. 🟡

## Границы уровня

- Код нужен одному потребителю → страница (уровень 2), как бы «сущностно» он ни выглядел.
- Инфраструктура без бизнес-логики (CRUD, DTO, токены) → уровень 1 (`shared`), не entities.
- Действие пользователя у нескольких потребителей → уровень 4 (`features`), даже если оно «про сущность».

## Алгоритм

![[3-алгоритм-entities.drawio]]

Шаг 4 + чек-лист extraction rule + фильтры из excessive-entities (CRUD? auth? только тип?).

## Примеры

### Пример 1 (простой): правило «на скидке» разошлось → entities/product/model

![[3-пример-правило-скидки.drawio]]

Snapshot 2 walkthrough целиком: две копии `is-on-sale` перестали соглашаться → один дом.

### Пример 2 (средний): одинаковая карточка товара в трёх местах → entities/product/ui

![[3-пример-карточка-товара.drawio]]

Вид действительно один и меняется синхронно (дизайн-система) → `ui` сущности; пропсы для различий.

### Пример 3 (сложный): корзина — когда сущность, когда нет

![[3-пример-корзина.drawio]]

Состояние внутри `features/add-to-cart` — не сущность; `entities/cart` зарабатывается, когда чекаут и сводка заказа обязаны соглашаться. Рядом анти-кейс: `user`-сущность ради токенов.

## Итог уровня

- entities зарабатывается extraction rule, не количеством потребителей и не «архитектурной красотой».
- Минимальная сущность — `model` + `index`; UI переезжает с осторожностью и импортируется только сверху.
- Транспортные типы, CRUD и auth-данные в entities не живут.

## Связи

- Назад: [[Уровень 2 — слой pages]] (копия vs извлечение). Дальше: [[Уровень 4 — слой features]] (глаголы).
- Теория: [[FSD — слои и правило импортов]] (раздел «куда общее для нескольких слайсов»), [[FSD — кросс-импорты и стратегии]] (@x).
- Практика: визуализатор, этап 5 (драма дублирования → entities/article).

🔗 **Якоря:** [FSD] — feature-sliced.design, Reference → Layers / FAQ · Официальный скилл: https://github.com/feature-sliced/skills (SKILL.md §1–2, §5-2, §6; references/growth-walkthrough.md, references/excessive-entities.md)
