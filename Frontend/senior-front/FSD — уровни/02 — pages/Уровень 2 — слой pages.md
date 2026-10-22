---
created: '2026-09-06'
difficulty: intermediate
mastery: growing
source: 'Глава 1.2 курса «Фронтенд senior+ в агентную эру» (якорь [FSD] — Reference
  → Layers, FAQ) + официальный скилл feature-sliced-design v2.1 ([[скилл — исходник]],
  SKILL.md §1–2, §5-1; references/growth-walkthrough.md). 🟡 — course-кейсы.'
status: analyzing
tags:
- frontend
- senior-front
- code-organization
- fsd
- layers
- pages
updated: '2026-09-06'
---

# Уровень 2 — слой pages

> Второй этап роста: появляются экраны. Правило уровня простое и почти контринтуитивное: **всё, что нужно одной странице, живёт в её слайсе** — включая формы, валидацию, запросы и «кажущийся переиспользуемым» код. pages — не витрина для гостей, а дом по умолчанию.

## Семантика слоя (по главе)

> «Одна страница обычно соответствует одному слайсу; несколько очень похожих страниц можно группировать в один слайс (например, формы регистрации и логина). Нет ограничения на количество кода в слайсе страницы, пока команде легко навигировать. **Если UI-блок страницы не переиспользуется — совершенно нормально держать его внутри слайса страницы.** Обычно в слайсе страницы: UI страницы вместе с состояниями загрузки и error-границами (`ui`), запросы данных и мутаций (`api`). Отдельная модель данных для страницы нехарактерна, крошечные куски состояния живут в самих компонентах.» — глава 1.2, §3.2 [FSD]

Слайсы одного слоя не знают друг о друге:

> «**zero coupling** — независим от других слайсов своего слоя» — глава 1.2, §4.1 [FSD]

И ограничитель снизу — «не всё — фича»:

> «Хороший индикатор того, что что-то должно стать фичей — то, что она **переиспользуется на нескольких страницах**.» — глава 1.2, §3.2 [FSD]. Комментарии только на странице статьи — не фича, они остаются в слайсе страницы.

## В официальном скилле

Весь уровень — это **Шаг 1** decision framework:

> «**Step 1: Where is this code used?** Used in only one page → keep it that `pages/` slice. Used in 2+ pages but duplication is manageable → **keeping separate copies in each page is also valid.** An entity or feature with a single consumer → keep it there.» (Шаг 1: где код используется? Одна страница — держи в её слайсе. 2+ страницы с терпимым дублированием — отдельные копии в каждой странице тоже валидны.) — SKILL.md §2

Рекомендация уровня (SHOULD):

> «**5-1. Pages first: place code where it is used.** Place code in `pages/` first. Extract to lower layers only when truly needed. Extraction is a design decision that affects the whole project, so the threshold should be high. **What stays in pages:** Large UI blocks used only in one page; Page-specific forms, validation, data fetching, state management; Page-specific business logic and API integrations; Code that looks reusable but is simpler to keep local.» — SKILL.md §5-1

> «**Golden Rule: When in doubt, keep it in `pages/`.**» (Сомневаешься — оставляй в pages.) — SKILL.md §2

Reference этого уровня — `growth-walkthrough.md` («when setting up a new project from scratch … or to show how a structure earns each layer over time»). Его вводный тезис — ключ ко всем следующим уровням:

> «Not every product change earns a layer: **a layer is earned by a stable responsibility that needs one home, not by a count of how many places use something.**» (Не каждое изменение продукта заслуживает слой: слой зарабатывается стабильной ответственностью, которой нужен один дом, а не количеством мест использования.) — references/growth-walkthrough.md

Snapshot 0 walkthrough — стартовое состояние магазина: у каждого запроса один потребитель, поэтому он сидит в `api` своей страницы (`pages/home/api/fetch-products.ts`), а `ProductDTO` читают обе страницы из `shared/api`.

Snapshot 1 — самое важное для уровня: третья страница переиспользует данные, **и никакого слоя не появляется**:

> «**The second card is a copy, not an extraction.** Two `ProductCard` files look like a signal for `entities/product/ui`. They are not, yet. … Step 1 covers this case directly: used in two pages but the duplication is manageable, so separate copies are valid. **Extracting now would force two cards that want to differ into one component that has to serve both.**» (Вторая карточка — копия, а не извлечение. … Извлечь сейчас — значит заставить две карточки, которые хотят отличаться, жить в одном компоненте, обслуживающем обе.) — references/growth-walkthrough.md, Snapshot 1

### Глава ↔ скилл

- **Согласованы:** один экран — один слайс; нерелюзнутый UI-блок — нормально внутри слайса; «не всё — фича» ⇔ «код с одним потребителем остаётся на месте».
- **Скилл добавляет язык для дублирования:** глава говорит «переиспользуется на нескольких страницах» как индикатор; скилл уточняет — второй потребитель **сам по себе** не сигнал, важна extraction rule (уровень 3): копии, которые дрейфят по своим причинам, остаются копиями.
- **Скилл называет анти-паттерн:** «Do not extract single-use code» — фича или сущность с одним потребителем возвращается в страницу.

## Границы уровня

- Инфраструктура без бизнес-логики, которую зовут несколько слайсов → уровень 1 (`shared`).
- Общее с бизнес-контекстом, где копии перестали быть управляемыми → уровни 3–4 (extraction rule).
- Роуты, провайдеры → `app` (уровень 1).

## Алгоритм

![[2-алгоритм-pages.drawio]]

Один вопрос уровня — «где используется?» — и правило сегментов внутри слайса.

## Примеры

### Пример 1 (простой): запрос баннеров главной

![[2-пример-баннеры.drawio]]

Один потребитель → `pages/home/api`. Snapshot 0 walkthrough в точности про это.

### Пример 2 (средний): фильтры каталога

![[2-пример-фильтры-каталога.drawio]]

Вид + состояние экрана (категория, цена, сортировка) → `ui` и `model` слайса страницы.

### Пример 3 (сложный): две карточки видео — копия, не извлечение

![[2-пример-карточки-копии.drawio]]

Карточка на главной и в поиске отличаются и дрейфят → копии валидны (Snapshot 1).

## Итог уровня

- pages — дом по умолчанию; порог извлечения высокий (Golden Rule).
- Внутри слайса — сегменты по назначению: `ui` (включая loading/error), `api`, редко `model`.
- Дублирование между страницами — не преступление, а сигнал «пока не трогай».

## Связи

- Дальше: [[Уровень 3 — слой entities]] — момент, когда копия перестаёт быть копией (extraction rule).
- Теория: [[FSD — слои и правило импортов]], [[FSD — слайсы и группы слайсов]], [[FSD — сегменты]].
- Практика: визуализатор, этапы 1–2 (каркас и сегменты Conduit).

🔗 **Якоря:** [FSD] — feature-sliced.design, Reference → Layers / FAQ · Официальный скилл: https://github.com/feature-sliced/skills (SKILL.md §2, §5; references/growth-walkthrough.md)
