---
status: analyzing
mastery: growing
tags: [frontend, rendering-strategies, ssr, react, nextjs]
source: "Глава 1.1 курса «Фронтенд senior+ в агентную эру»; якорь [Pat] patterns.dev — Server-side Rendering"
created: 2026-08-02
updated: 2026-08-02
difficulty: intermediate
---

# 🧠 Topic: Server-Side Rendering (SSR)

> [!abstract] Executive Summary (ELI5)
> SSR — это когда сервер сам «рисует» HTML-страницу под твой конкретный визит и отправляет её готовую. В отличие от статики (один HTML на всех, готов заранее), при SSR сервер работает на **каждый запрос**, поэтому может подставить в HTML твои персональные данные — корзину, баланс, валюту. Браузер сразу видит контент, а потом React «оживляет» страницу (гидратация).

> Источник: глава 1.1 курса, подглава 3. Якорь: [Pat] patterns.dev, «Server-side Rendering».

---

## 🎯 The "Why"

- **Problem Solved:** SSR нужен, когда контент страницы **зависит от конкретного запроса** — персональные данные (cookie, авторизация, A/B-тест, гео). Статика тут бессильна: её HTML один на всех, в него нельзя вшить персональные данные.
- **Alternatives:**
    - [[Static-Rendering]] (SSG) — если контент одинаков для всех (блог, документация);
    - [[CSR-Client-Side-Rendering]] — если высокоинтерактивное приложение (app shell, Figma-тип);
    - [[ISR-Incremental-Static-Regeneration]] — если данные терпят устаревание (Tolerable Staleness).
- **Trade-offs:** SSR решает проблемы статики (персонализация) и CSR (LCP/SEO), но платит: TTFB медленнее (работа на каждый запрос), server costs, hydration expense.

---

## 💡 Mental Model & Analogy

- **Аналогия:** Статика — это напечатанная книга в библиотеке: лежит готовая, берёшь и читаешь, но одна на всех. SSR — это ксерокопия, сделанная под твой запрос: библиотекарь находит нужное, копирует, отдаёт именно тебе. Можно попросить «мою страницу с моими закладками».
- **Ключевое различие со статикой — КОГДА, а не ГДЕ:** и статика, и SSR рендерят на сервере. Разница в моменте. Статика — один раз при сборке. SSR — на каждый визит.

```
Static:  npm run build → HTML готов → лежит на CDN → всем одинаковый
SSR:     визит → сервер рендерит под запрос → HTML уникальный → отправлен
```

---

## 🛠️ Core Principles & Mechanics

### Lifecycle SSR (два этапа)

```
ЭТАП 1 (сервер): React-компоненты → HTML → отправлен пользователю
ЭТАП 2 (клиент): HTML → пользователь видит контент → React «оживляет» (гидратация)
```

> **Жёсткое правило:** рендеринг = сервер (или сборка). Гидратация = клиент. Никогда не наоборот.

### Триггеры SSR (когда нужен)

Источник: «the signed-in user, a cookie, an A/B-test cohort, a geolocation header.»

| Триггер | Пример |
|---|---|
| signed-in user | дашборд (для каждого свои данные) |
| cookie | корзина без авторизации (cookie session) |
| A/B-test cohort | группа A видит новый дизайн |
| geolocation header | цены в валюте страны |

**Принцип:** контент персонален → SSR. Контент один на всех → статика.

### Преимущества и недостатки

| Преимущества                                               | Недостатки                                                   |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| **LCP** — главный контент в initial HTML                   | **TTFB** медленнее (work happens per request)                |
| **SEO** — краулеры видят HTML без JS                       | **Server Costs** — every request runs the render             |
| **Personalization** — cookies/headers доступны при рендере | **Hydration Expense** — клиенту нужен JS для интерактивности |

---

## 🌊 Streaming SSR (React 18+)

React 18 заменил синхронный `renderToString` на потоковые API:

```tsx
// Node-рантайм
import { renderToPipeableStream } from 'react-dom/server'
const { pipe } = renderToPipeableStream(<App />, {
  onShellReady() { /* shell готов → отправляем немедленно */ },
  onAllReady()   { /* всё готово → для краулеров */ },
  onError(error) { logError(error) },
})

// Edge/Web-рантайм (Cloudflare Workers, Vercel Edge, Deno)
import { renderToReadableStream } from 'react-dom/server'
```

> **Почему два метода:** Node работает с потоками через `pipe` (Node Streams). Edge-среды — через `ReadableStream` (Web Streams API). Это разные среды, у Edge нет `pipe`.

### Shell vs Suspense fallback

| | Shell | Suspense fallback |
|---|---|---|
| Что | статический layout: шапка, навигация, футер | плейсхолдер конкретного блока |
| Где | вне Suspense-границ | внутри Suspense |
| Когда отправляется | первым (через `onShellReady`) | потом подменяется на чанк |

---

## 👻 Uncanny Valley («зловещая долина»)

Гидратация дорогая (~равна первому клиентскому рендеру). Создаёт разрыв:

```
HTML получен → пользователь ВИДИТ контент (LCP пройден)
       ↓
   РАЗРЫВ (uncanny valley): страница выглядит готовой, но:
   - JS-бандл ещё не распарсен
   - React ещё не завершил гидратацию
   - клики ТИХО ТЕРЯЮТСЯ
       ↓
Гидратация завершена → клики работают
```

> **Жёсткая связка:** uncanny valley ↔ **INP** (Interaction to Next Paint).

### Три паттерна снижения uncanny valley

| Паттерн | Как снижает |
|---|---|
| **1. Streaming SSR + `<Suspense>`** | гидрирует по частям, не ждёт весь бандл — **фундамент** |
| **2. Selective Hydration** (React 18+) | приоритизирует гидратацию кликнутого компонента |
| **3. [[React-Server-Components]]** | убирает гидратацию совсем (нет JS → не из чего гидрировать) |

> **Важно:** Selective Hydration **не существует без Suspense-границ**. Suspense создаёт «куски», по которым можно резать и стриминг, и гидратацию. Streaming+Suspense = фундамент; два других — надстройки над ним.

---

## ⚛️ SSR в Next.js App Router

- Серверный рендеринг — **поведение по умолчанию**.
- `getServerSideProps` **удалён** (данные — в async-компоненте прямо в теле).
- Маршрут становится динамическим **автоматически**, если использовать «динамические функции»:

| API | Тип | Что даёт |
|---|---|---|
| `cookies()` | функция (`next/headers`) | куки пользователя |
| `headers()` | функция (`next/headers`) | заголовки запроса |
| `searchParams` | пропс страницы | query-параметры URL |

```tsx
import { cookies } from 'next/headers'

export default async function Page() {
  const cookieStore = await cookies()                    // ← await (Next.js 15+)
  const currency = cookieStore.get('currency')?.value    // ← триггер SSR
  const products = await db.query('SELECT * FROM products')
  return <ProductList items={products} currency={currency} />
}
```

### Изменения Next.js 15

- `fetch()` **больше не кэшируется по умолчанию** → прозрачность (явное кэширование через `'use cache'`).
- `cookies()`, `headers()`, `draftMode()` — **асинхронные** (нужен `await`).
- `async`-API создают **границу `await`**, по которой фреймворк «режет» рендер на статическую часть (при сборке) и динамическую (при запросе) — основа **Partial Prerendering (PPR)**.

---

## 🌍 Edge SSR

Edge-рантайм (Cloudflare Workers, Vercel Edge, Deno Deploy) приближает рендеринг к пользователю.

| Метрика | Origin (центр. сервер) | Edge |
|---|---|---|
| TTFB | 200–600 мс | **30–80 мс** |
| Cold start | ощутимый | стремится к нулю |

### Trade-off Edge

- **Подмножество Node:** нет нативных модулей (C/C++), нет файловой системы. Нужен `renderToReadableStream`.
- **Жёсткие лимиты:** память **50–128 МБ**, CPU **десятки миллисекунд на запрос**.
- **Ловушка данных:** БД всё равно на origin → если далеко от Edge-точки, сетевая задержка съедает выигрыш. **Решение:** edge-реплицированные данные (Redis/Turso/D1 рядом с Edge).

> **TTFB ≠ LCP:** Edge + Streaming дают быстрый TTFB (shell уходит сразу), но LCP (главный контент) всё равно ждёт данных. Без edge-репликации выигрыш половинчатый.

---

## ⛔ Когда SSR НЕ нужен

| Сценарий | Почему | Альтернатива |
|---|---|---|
| **Truly Static Content** (блог, доки, лендинг) | контент одинаков для всех → SSR избыточен | Static (SSG) |
| **Highly Interactive Apps** (Figma, app shell) | «hydration cost outweighs SSR's benefit» | CSR с app shell |
| **Tolerable Staleness** (цены меняются, но терпят 5 мин) | нет смысла платить за per-request свежесть | ISR с revalidation |

---

## ✅ Production-чеклист

- [ ] `renderToPipeableStream` / `renderToReadableStream`, не устаревший `renderToString`;
- [ ] клиент использует `hydrateRoot`, не старый `ReactDOM.hydrate`;
- [ ] `onError`-обработчик + Error Boundary ловят сбои;
- [ ] медленные данные в `<Suspense>` → shell стримится первым;
- [ ] request-scoped кэш через React `cache()` для дедупликации выборок **внутри одного рендера**;
- [ ] JS-бандл разделён по маршрутам → гидратации нужен только JS текущей страницы.

```tsx
// request-scoped кэш: дедупликация ВНУТРИ одного рендера
import { cache } from 'react'
const getUser = cache(async () => db.query('SELECT * FROM users WHERE id=...'))
// ↑ повторные вызовы в рамках одного рендера берут результат из кэша
```

---

## 🔗 Связанные заметки

- [[Static-Rendering]] — SSG и 4 вариации (включая ISR);
- [[CSR-Client-Side-Rendering]] — рендеринг в браузере;
- [[React-Server-Components]] — устранение гидратации;
- [[Islands-Architecture]] — частичная гидратация;
- [[Core-Web-Vitals]] — TTFB, FCP, LCP, INP, CLS.

## 📚 Источники

- Глава 1.1 курса «Фронтенд senior+ в агентную эру», подглава 3.
- Якорь [Pat] — patterns.dev, «Server-side Rendering»: https://www.patterns.dev/react/server-side-rendering
- Якорь [NEXT] — Next.js Docs (Server/Client Components, Caching).
