---
created: '2026-09-06'
difficulty: intermediate
mastery: growing
source: 'Глава 1.2 курса «Фронтенд senior+ в агентную эру» (якорь [FSD] — Reference
  → Layers, FAQ) + официальный скилл feature-sliced-design v2.1 ([[скилл — исходник]],
  SKILL.md §2–3, §5–6; references/growth-walkthrough.md). 🟡 — course-кейсы.'
status: analyzing
tags:
- frontend
- senior-front
- code-organization
- fsd
- layers
- features
updated: '2026-09-06'
---

# Уровень 4 — слой features

> Переиспользуемые **действия**: «в корзину», «в избранное», «подписаться». Глаголы, которые продукт обещает пользователю на нескольких страницах. Как и с entities, порог высокий: «не всё — фича».

## Семантика слоя (по главе)

> «Features — слой для главных взаимодействий в приложении — того, что пользователям **заботливо хочется делать**. Взаимодействия часто затрагивают бизнес-сущности — ведь именно об этом приложение. Ключевой принцип: **не всё должно быть фичей**. Хороший индикатор того, что что-то должно стать фичей — то, что она **переиспользуется на нескольких страницах**. Слайс фичи может содержать UI для выполнения взаимодействия, например форму (`ui`), API-вызовы (`api`), валидацию и внутреннее состояние (`model`), feature-флаги (`config`).» — глава 1.2, §3.2 [FSD]

Тест уровня — глагол при существительном: «пользователь хочет [добавить] [товар в корзину]». И общая цель слоёв:

> «Optimize for the experience of a newcomer to the project to quickly discover large important areas of code.» (Оптимизируй под опыт новичка — быстро находить большие важные области кода.) — глава 1.2, §3.2 [FSD]

Срез главы про юз-кейсы и домен (пример справочника slice groups): `add-to-cart`/`remove-from-cart` — юз-кейсы (фичи), а cart-DTO и мапперы — домен корзины (`entities/cart`).

## В официальном скилле

Уровень — **Шаг 3**, и скилл спрашивает его **раньше** entities:

> «**Step 3: Is it a complete user action that several consumers share, with a focused responsibility and a reason to change of its own?** Yes → `features/`. **Uncertain, single use, or speculative reuse → keep in the page.**» (Шаг 3: это полное пользовательское действие, которым делятся несколько потребителей…? Да → features. Не уверен, один потребитель, спекулятивное переиспользование → оставь в странице.) — SKILL.md §2

Таблица быстрого размещения (§3) проводит границу «одна страница vs подтверждённое переиспользование»:

> «User profile form: single use → `pages/profile/ui/ProfileForm.tsx`; confirmed multi-use → `features/profile-form/`. Auth login form: `pages/login/ui/LoginForm.tsx` … `features/auth/`.» — SKILL.md §3

Snapshot 3 walkthrough — момент появления слоя: между снапшотами кнопка «в корзину» (с запросом и оптимистичным обновлением) жила в `api`/`model` страницы товара; когда то же действие понадобилось поиску:

> «Step 3 asks whether this is a complete user action, used in multiple places, with a stable boundary. The button, the request, and the cart update form one action; two pages use it; adding to the cart means the same thing from either page. **Extract it.**» — references/growth-walkthrough.md, Snapshot 3

И важная тонкость про `api`-сегмент фичи:

> «**The request goes with the feature because it is the add-to-cart use case itself, not a generic cart CRUD wrapper.** A plain reusable cart request would have stayed in `shared/api` however many slices called it.» (Запрос едет с фичей, потому что он и есть юз-кейс «добавить в корзину», а не общий CRUD-обёртчик корзины. …) — references/growth-walkthrough.md, Snapshot 3

Анти-паттерны уровня (§6): «Do not extract single-use code» — фича с одним потребителем возвращается в страницу; «Do not create god slices» — `user-management/` режется на `auth/`, `profile-edit/`, `password-reset/`.

### Глава ↔ скилл

- **Согласованы:** «не всё — фича» ⇔ «uncertain → keep in the page»; индикатор — переиспользование на страницах; сегменты фичи (ui/api/model/config).
- **Скилл уточняет порог:** «переиспользуется» = extraction rule (несколько потребителей сейчас + своя причина меняться + сфокусированная ответственность), а не «второе место появилось».
- **Скилл фиксирует порядок:** features проверяется раньше entities — сначала глагол, потом существительное. 🟡

## Границы уровня

- Действие нужно на одной странице → слайс страницы (уровень 2).
- Это данные + вид, а не действие → entities (уровень 3).
- Фича разрослась в «всё про пользователя» → разбить (god slices, §6).

## Алгоритм

![[4-алгоритм-features.drawio]]

Шаг 3: три вопроса — действие? несколько потребителей? стабильная граница?

## Примеры

### Пример 1 (простой): «в избранное» на карточке и в каталоге → features/add-to-favorites

![[4-пример-избранное.drawio]]

Паттерн Snapshot 3 наизнанку: кнопка + запрос + оптимистичное состояние = одно действие.

### Пример 2 (средний): add-to-cart vs cart-DTO — юз-кейсы и домен

![[4-пример-юзкейсы-домен.drawio]]

Кто где живёт: действия — фичи, корзина с данными — сущность, транспорт — shared/api.

### Пример 3 (сложный): подписка на канал + контраст «просмотр — не фича»

![[4-пример-подписка.drawio]]

Видеоплатформа: subscribe на двух экранах — фича; watch — экран, а не действие.

## Итог уровня

- features — полные пользовательские действия с подтверждённым переиспользованием.
- Запрос едет с фичей, только если он и есть юз-кейс; общий CRUD — в `shared/api`.
- Сомнение — не фича: остаётся в странице (Golden Rule).

## Связи

- Назад: [[Уровень 3 — слой entities]]. Дальше: [[Уровень 5 — кросс-импорты]] (стратегии A–D для фич).
- Теория: [[FSD — слои и правило импортов]] (entity vs feature), [[FSD — слайсы и группы слайсов]].
- Практика: визуализатор, этап 6 (favorite-article, хедер, @x).

🔗 **Якоря:** [FSD] — feature-sliced.design, Reference → Layers / FAQ · Официальный скилл: https://github.com/feature-sliced/skills (SKILL.md §2–3, §6; references/growth-walkthrough.md)
