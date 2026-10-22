---
created: '2026-08-05'
difficulty: advanced
mastery: growing
source: Глава 1.1 курса «Фронтенд senior+ в агентную эру», подглава 8. Якорь [NEXT]
  Next.js 16 — Caching
status: analyzing
tags:
- frontend
- rendering-strategies
- nextjs
- ppr
- partial-prerendering
- cache-components
- suspense
- use-cache
updated: '2026-08-05'
---

# 🧩 Topic: Cache Components и Partial Prerendering (PPR)

> [!abstract] Executive Summary (ELI5)
> **PPR (Partial Prerendering = частичный предрендеринг)** — режим Next.js (включается `cacheComponents: true`), который позволяет **одной и той же странице** быть одновременно **частично статической** (предрендерится при сборке) **и частично динамической** (стримится при запросе). До PPR каждый маршрут был «целиком статический» или «целиком динамический». С PPR страница режется: **shell** (большая часть) рендерится при сборке и кэшируется, а динамические куски в `<Suspense>` стримятся при запросе.

> Источник: глава 1.1 курса, подглава 8. Якорь: [NEXT] Next.js 16, «Caching». Это та страница, на которую редиректит классический якорь «Rendering».

---

## 🎯 The "Why"

- **Problem Solved (главная проблема):** До PPR у Next.js была модель «или/или». Каждый маршрут был **целиком** либо статическим, либо динамическим. Страница `/dashboard`, где 90% контента статично, а 10% — персональная корзина (`cookies()`), **целиком** становилась динамической. Сервер работал на каждый запрос, хотя мог бы отдать 90% из кэша.
- **Solution:** PPR ломает «или/или». Страница режется на **shell** (при сборке) и **Suspense-чанки** (при запросе).
- **Trade-offs:** PPR — это не новая стратегия рендеринга, а **способ склеить уже известные** (SSG + Streaming SSR + RSC Payload + кэш). Главное новое — механизм склейки через `'use cache'` и Suspense-границы.

---

## 💡 Mental Model & Analogy

**Ресторан с шведским столом:**

| | Без PPR | С PPR |
|---|---|---|
| **Статический маршрут** (блог) | все блюда готовы заранее | то же |
| **Динамический маршрут** (дашборд) | повар готовит всё под твой визит, ты ждёшь | большая часть готова заранее (shell), повар готовит только персональный десерт (Suspense-чанк), пока ты ешь остальное |

PPR = «шведский стол с персональной доработкой».

---

## 🛠️ Как включить

Опция `cacheComponents` в `next.config.ts`:

```ts
import type { NextConfig } from 'next'
const nextConfig: NextConfig = { cacheComponents: true }
export default nextConfig
```

> При включённых Cache Components `GET` Route Handlers следуют той же модели prerendering'а, что и страницы.

---

## 🏷️ Директива `'use cache'` (два уровня)

`'use cache'` кэширует возвращаемое значение async-функций и компонентов. Два уровня:

| Уровень | Что кэширует | Пример |
|---|---|---|
| **Data-level** | функцию, которая fetch'ит данные | `getProducts()`, `getUser(id)` |
| **UI-level** | целиком компонент или страницу | `async function BlogPosts()` |

### Cache key (ключ кэша) — критичная деталь

> Аргументы и любые замкнутые (closed-over) значения из родительских скоупов автоматически становятся частью **cache key**. **Разные входы дают разные записи кэша**. Это позволяет кэшировать персонализированный или параметризованный контент.

`getUser(id)` с `id=1` и `id=2` — две разные записи в кэше.

### Пример — Data-level

```tsx
import { cacheLife } from 'next/cache'

export async function getUsers() {
  'use cache'
  cacheLife('hours')
  return db.query('SELECT * FROM users')
}
```

### Пример — UI-level

```tsx
export default async function Page() {
  'use cache'
  cacheLife('hours')
  const users = await db.query('SELECT * FROM users')
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>
}
```

> Если `"use cache"` в начале файла — все экспортируемые функции кэшируются.

---

## 🆚 `'use cache'` vs `cache()` из React (частая путаница)

| | `cache()` из React (подглава 3) | `'use cache'` из Next.js (подглава 8) |
|---|---|---|
| Что кэширует | результат функции **внутри одного рендера** | результат функции **между запросами** |
| Время жизни | один запрос (request-scoped) | настраивается (`cacheLife('hours')`) |
| Задача | дедупликация (не дёргать БД 5 раз за рендер) | одинаковые запросы не пересчитываются |

---

## 🌊 Стриминг некэшированных данных (Suspense)

Для компонентов, требующих **свежих данных на каждый запрос**, **не** используйте `'use cache'`. Оберните в `<Suspense>` с fallback:

```tsx
async function LatestPosts() {
  const data = await fetch('https://api.example.com/posts')
  const posts = await data.json()
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>
}

export default function Page() {
  return (
    <>
      <h1>My Blog</h1>
      <Suspense fallback={<p>Loading posts...</p>}>
        <LatestPosts />           {/* ← НЕ 'use cache', стримится */}
      </Suspense>
    </>
  )
}
```

- Fallback (`<p>Loading posts...</p>`) попадает в **статический shell**.
- Контент компонента **стримится в момент запроса**.

> [!warning] Частое заблуждение
> `<Suspense>` **сам по себе** не переводит компонент в динамический рендеринг. Если компонент выполняет только синхронную работу — он завершится во время prerendering'а независимо от Suspense. Источник динамичности — **`await` (асинхронная работа)**, а Suspense лишь **позволяет** оформить её как стримящийся чанк.

---

## 🚧 Runtime APIs — что переводит в запрос-время

**Runtime APIs** требуют информации, доступной только при запросе пользователя:

| API | Что даёт |
|---|---|
| `cookies()` | данные cookie пользователя |
| `headers()` | заголовки запроса |
| `searchParams` | параметры URL-запроса |
| `params` (без `generateStaticParams`) | параметры динамического маршрута |

> Компоненты, обращающиеся к runtime APIs, следует **обязательно** оборачивать в `<Suspense>`.

---

## ⚙️ Механика: статический shell + PPR (ключевой раздел)

При сборке Next.js обходит дерево. Поведение каждого компонента зависит от API:

| Маркер | Что означает | Куда попадает |
|---|---|---|
| `'use cache'` | «закэшируй, положи в shell» | **статический shell** |
| `cookies()` / `headers()` / `searchParams` внутри | «мне нужны данные запроса» | **динамика (требует Suspense)** |
| `await fetch()` / `await db.query()` **без** `'use cache'` | «асинхронная работа, не кэширую» | **динамика (требует Suspense)** |
| ничего (чистая синхронная работа) | «нет ничего от запроса» | **автоматически в shell** |

### Что такое статический shell

Так генерируется **статический shell**, состоящий из:
- **HTML** для первоначальных загрузок страницы,
- **сериализованного RSC Payload** для клиентской навигации.

> Этот подход к рендерингу называется **Partial Prerendering (PPR)**, и это **поведение по умолчанию** при включённых Cache Components.

---

## ✂️ Почему именно Suspense — граница разреза

> [!important] Suspense = граница, по которой фреймворк «режет» страницу

При сборке Next.js обходит дерево и пытается всё отрендерить. Когда он доходит до компонента с `cookies()` внутри (а запроса нет!), он **застревает**: не может выполнить, но и пропустить не может.

**Suspense — это разрешение фреймворку сказать «не могу сейчас, давай позже».**

```
С Suspense:
  сборка:
    статичные куски → в shell
    <Suspense>
      <Recommendations />  → НЕ МОГУ (cookies), кладу FALLBACK в shell
    </Suspense>            →标记 «этот кусок достримлю при запросе»

  запрос пользователя:
    shell отдаётся мгновенно из кэша (включая fallback)
    параллельно сервер считает динамический кусок
    когда готово → достримливает чанк, заменяя fallback
```

**Без Suspense** вокруг динамического куска → фреймворк не может отложить кусок → **весь маршрут становится динамическим** → теряется всё преимущество PPR.

---

## 📋 Пример страницы с PPR

```tsx
// app/products/page.tsx
import { Suspense } from 'react'
import { cacheLife } from 'next/cache'
import { cookies } from 'next/headers'

// --- КУСОК 1: список товаров (СТАТИЧЕСКИЙ SHELL) ---
async function ProductList() {
  'use cache'                              // ← в shell
  cacheLife('hours')
  const products = await db.query('SELECT * FROM products')
  return products.map(p => <ProductCard key={p.id} {...p} />)
}

// --- КУСОК 2: рекомендации (ДИНАМИЧЕСКИЙ ЧАНК) ---
async function Recommendations() {
  const cookieStore = await cookies()      // ← нужен запрос → динамика
  const userId = cookieStore.get('uid')?.value
  const recs = await db.query(`SELECT * FROM recs WHERE user=${userId}`)
  return recs.map(r => <RecCard key={r.id} {...r} />)
}

// --- СТРАНИЦА ---
export default function ProductsPage() {
  return (
    <div>
      <h1>Товары</h1>
      <ProductList />                      {/* без Suspense, в shell */}
      <Suspense fallback={<p>Реком. загружаются...</p>}>
        <Recommendations />                {/* в Suspense, стримится */}
      </Suspense>
    </div>
  )
}
```

### Временная шкала запроса

```
t=0    пользователь запрашивает /products
t=0    сервер мгновенно отдаёт shell из кэша:
         <h1>Товары</h1>
         <ProductList />         (из кэша)
         <p>Реком. загружаются...</p>   (fallback)
       пользователь ВИДИТ страницу
t=0    параллельно сервер считает Recommendations
t=200мс Recommendations готовы → достримливаются → заменяют fallback
```

---

## 🚫 Отказ от статического shell (Opting out)

Полностью отказаться от shell — граница `<Suspense>` с **пустым fallback** над `<body>` в Root Layout:

```tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <Suspense fallback={null}>           {/* ← пустой fallback */}
        <body>{children}</body>
      </Suspense>
    </html>
  )
}
```

Fallback пуст → немедленно отправлять нечего → каждый запрос блокируется, пока страница полностью не отрендерится. Это по сути **отключает PPR** для всего приложения. Чтобы ограничить конкретными маршрутами — **multiple root layouts**.

---

## 🔗 Связанные заметки

- [[Next.js Server and Client Components]] — три канала доставки, RSC Payload.
- [[SSR-Server-Side-Rendering]] — Streaming SSR + Suspense (фундамент PPR).
- [[Static-Rendering]] — статический shell опирается на SSG.
- [[React Server Components]] — серверные компоненты в shell.
- [[Core-Web-Vitals]] — PPR улучшает TTFB и LCP через мгновенный shell.

## 📚 Источники

- Глава 1.1 курса «Фронтенд senior+ в агентную эру», подглава 8.
- Якорь [NEXT] — Next.js 16, «Caching»: https://nextjs.org/docs/app/getting-started/caching