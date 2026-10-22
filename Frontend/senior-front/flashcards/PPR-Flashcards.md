#flashcards/frontend/rendering-strategies/ppr

# 🧩 Cache Components / Partial Prerendering (PPR) — Flashcards

Source: [[Partial Prerendering]]. Глава 1.1 курса, подглава 8. Якорь [NEXT] Next.js 16.

## Главная идея

Что такое PPR (Partial Prerendering) одним предложением?
?
Режим Next.js (включается `cacheComponents: true`), который позволяет **одной и той же странице** быть одновременно **частично статической** (предрендерится при сборке) **и частично динамической** (стримится при запросе). До PPR каждый маршрут был «целиком статический» или «целиком динамический».
#flashcards/frontend/rendering-strategies/ppr/foundations

Какая проблема была в Next.js до PPR (модель «или/или»)?
?
Каждый маршрут был **целиком** либо статическим, либо динамическим. Страница `/dashboard`, где 90% контента статично, а 10% — персональная корзина (`cookies()`), **целиком** становилась динамической. Сервер работал на каждый запрос, хотя мог бы отдать 90% из кэша.
#flashcards/frontend/rendering-strategies/ppr/foundations

Как включить Cache Components / PPR?
?
Опция `cacheComponents: true` в `next.config.ts`:
```ts
import type { NextConfig } from 'next'
const nextConfig: NextConfig = { cacheComponents: true }
export default nextConfig
```
#flashcards/frontend/rendering-strategies/ppr/foundations

Является ли PPR новой стратегией рендеринга?
?
**НЕТ.** PPR — это **способ склеить уже известные стратегии** (SSG + Streaming SSR + RSC Payload + кэш) на одной странице. Главное новое — механизм склейки через `'use cache'` и Suspense-границы.
#flashcards/frontend/rendering-strategies/ppr/foundations

## Директива `'use cache'`

На каких двух уровнях применяется `'use cache'`?
?
**1. Data-level** — кэшировать функцию, которая fetch'ит данные (`getProducts()`, `getUser(id)`). **2. UI-level** — кэшировать целиком компонент или страницу (`async function BlogPosts()`). Если `"use cache"` в начале файла — все экспортируемые функции кэшируются.
#flashcards/frontend/rendering-strategies/ppr/use-cache

Что автоматически становится частью cache key (ключа кэша) при использовании `'use cache'`?
?
**Аргументы функции и любые замкнутые (closed-over) значения из родительских скоупов.** Разные входы → разные записи кэша. Это позволяет кэшировать персонализированный или параметризованный контент: `getUser(id)` с `id=1` и `id=2` — две разные записи.
#flashcards/frontend/rendering-strategies/ppr/use-cache

Чем отличается `'use cache'` (Next.js) от `cache()` (React)?
?
**`cache()` из React** — кэширует результат функции **внутри одного рендера** (request-scoped), для дедупликации (не дёргать БД 5 раз за рендер). **`'use cache'` из Next.js** — кэширует результат **между запросами**, время жизни настраивается (`cacheLife('hours')`), для того чтобы одинаковые запросы не пересчитывались.
#flashcards/frontend/rendering-strategies/ppr/use-cache

## Стриминг + Suspense

Какой паттерн для компонента, требующего свежих данных на каждый запрос (например, `LatestPosts`)?
?
**НЕ использовать `'use cache'`**, обернуть в `<Suspense>` с fallback:
```tsx
<Suspense fallback={<p>Loading posts...</p>}>
  <LatestPosts />
</Suspense>
```
Fallback попадает в статический shell, контент компонента **стримится в момент запроса**.
#flashcards/frontend/rendering-strategies/ppr/streaming

Верно ли: «Если обернуть синхронный компонент в `<Suspense>`, он станет стримиться при запросе»?
?
**НЕТ.** `<Suspense>` **сам по себе** не переводит компонент в динамический рендеринг. Если компонент выполняет только синхронную работу — он завершится во время prerendering'а независимо от Suspense. Источник динамичности — **`await` (асинхронная работа)**, а Suspense лишь **позволяет** оформить её как стримящийся чанк.
#flashcards/frontend/rendering-strategies/ppr/streaming

## Runtime APIs

Назови 4 runtime APIs, которые переводят компонент в запрос-время.
?
**1. `cookies()`** — данные cookie пользователя. **2. `headers()`** — заголовки запроса. **3. `searchParams`** — параметры URL-запроса. **4. `params`** (без `generateStaticParams`) — параметры динамического маршрута. Все требуют информации, доступной только при запросе.
#flashcards/frontend/rendering-strategies/ppr/runtime-apis

Что нужно сделать с компонентом, использующим `cookies()` в мире Cache Components?
?
**Обязательно обернуть в `<Suspense>`** с fallback. Без Suspense фреймворк не может отложить кусок → весь маршрут становится динамическим → теряется преимущество PPR.
#flashcards/frontend/rendering-strategies/ppr/runtime-apis

## Механика: что куда попадает

Заполни таблицу: какой маркер → куда попадает компонент при сборке.
?
- `'use cache'` → **статический shell** (результат кэшируется)
- `cookies()` / `headers()` / `searchParams` внутри → **динамика** (требует Suspense)
- `await fetch()` / `await db.query()` **без** `'use cache'` → **динамика** (требует Suspense)
- ничего (чистая синхронная работа) → **автоматически в shell**
#flashcards/frontend/rendering-strategies/ppr/mechanics

Что такое «статический shell»?
?
Часть страницы, которая рендерится при сборке и кэшируется. Состоит из **HTML** (для первоначальных загрузок страницы) и **сериализованного RSC Payload** (для клиентской навигации). Гарантирует, что браузер получает отрендеренный контент **мгновенно** — независимо от того, прямой заход или клиентская навигация.
#flashcards/frontend/rendering-strategies/ppr/mechanics

## Почему именно Suspense — граница разреза

Почему Suspense — это граница, по которой фреймворк «режет» страницу?
?
При сборке Next.js обходит дерево. Когда доходит до компонента с `cookies()` (а запроса нет!) — **застревает**: не может выполнить, но и пропустить не может. **Suspense — это разрешение фреймворку сказать «не могу сейчас, давай позже».** Без Suspense фреймворк не знает, где резать → весь маршрут становится динамическим.
#flashcards/frontend/rendering-strategies/ppr/suspense-boundary

Что произойдёт, если использовать `cookies()` в компоненте БЕЗ Suspense?
?
Фреймворк не сможет отложить этот кусок → **весь маршрут /products становится динамическим** → shell НЕ кэшируется → каждый запрос ждёт ВСЁ дерево. Отсутствие Suspense вокруг динамического куска **откатывает всю страницу к чистому SSR** — теряется всё преимущество PPR.
#flashcards/frontend/rendering-strategies/ppr/suspense-boundary

Опиши временную шкалу запроса с PPR (shell + Recommendations в Suspense).
?
**t=0**: пользователь запрашивает страницу → сервер **мгновенно отдаёт shell из кэша** (включая fallback «Загружается...») → пользователь ВИДИТ страницу. **Параллельно** сервер считает динамический кусок (`cookies()` + `db.query`). **t=200мс**: Recommendations готовы → достримливаются как чанк → **заменяют fallback**. Пользователь не ждёт персональный контент, чтобы увидеть страницу.
#flashcards/frontend/rendering-strategies/ppr/suspense-boundary

## Отказ от статического shell

Как полностью отказаться от статического shell (отключить PPR)?
?
Граница `<Suspense>` с **пустым fallback** над `<body>` в Root Layout:
```tsx
<Suspense fallback={null}><body>{children}</body></Suspense>
```
Fallback пуст → немедленно отправлять нечего → каждый запрос блокируется, пока страница полностью не отрендерится. Чтобы ограничить конкретными маршрутами — **multiple root layouts**.
#flashcards/frontend/rendering-strategies/ppr/opt-out

## Практика

Страница `/products`. Список товаров одинаковый для всех (обновляется раз в час), блок «рекомендации» персональный (`cookies()`). Как распределить в PPR?
?
**Список товаров** → `'use cache'` + `cacheLife('hours')` → попадает в **статический shell** (без Suspense). **Рекомендации** → без `'use cache'`, с `cookies()` внутри, **обязательно обернуть в `<Suspense>`** с fallback → fallback в shell, контент стримится при запросе. Shell отдаётся из кэша мгновенно, рекомендации достримливаются параллельно.
#flashcards/frontend/rendering-strategies/ppr/practice
