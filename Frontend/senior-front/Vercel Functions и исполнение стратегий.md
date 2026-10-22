---
created: '2026-08-05'
difficulty: intermediate
mastery: growing
source: Глава 1.1 курса «Фронтенд senior+ в агентную эру», подглава 9. Якорь [DEP]
  Vercel — https://vercel.com/docs/functions ; связка с [Pat] §3.6 Edge SSR
status: analyzing
tags:
- frontend
- senior-front
- rendering-strategies
- vercel
- serverless
- edge
- data-locality
- deployment
updated: '2026-08-05'
---

# Vercel Functions и исполнение стратегий

> Подглава 9 курса «Фронтенд senior+ в агентную эру». Якорь [DEP] — Vercel Functions. Здесь «сервер» из прошлых подглав превращается в конкретную инфраструктуру: serverless-функции, регионы, рантаймы.

## Executive Summary (ELI5)

До сих пор «сервер» был абстракцией. На Vercel «сервер» = набор **serverless-функций**, которые:
- запускаются под каждый запрос,
- живут в одном регионе (по умолчанию `iad1` — Вашингтон),
- scale-to-zero (выключаются без трафика),
- переиспользуются между запросами, когда тёплые (warm).

Самое важное правило всей подглавы: **функция должна стоять рядом с БД**. Межконтинентальный лаг 150–200 мс убивает любой выигрыш от SSR/PPR, как бы умно они ни были настроены.

## The Why

Зачем вообще думать про инфраструктуру, если выбрал правильную стратегию рендеринга?

Потому что **стратегия рендеринга описывает «что»**, а **деплой описывает «где и как»**. SSR рядом с БД = быстрая страница. SSR на Edge в Берлине + БД в США = 150–200 мс RTT (Round Trip Time) до БД, и быстрая доставка HTML пользователю не спасает.

## Mental Model: «функция» — это не «сервер»

| Уровень | Что это | Сколько живёт |
|---|---|---|
| **Функция (function)** | Развёрнутый кусок кода (route handler, page, server action) | Пока задеплоена |
| **Экземпляр (instance)** | Запущенный процесс этой функции | Минуты-часы, пока есть трафик |
| **Инвокация (invocation)** | Один запуск под один запрос | Доли секунды |

N запросов → N инвокаций, но **M экземпляров (где M ≤ N)** — благодаря **fluid compute**.

## Core Concepts

### 1. Инвокация = один прогон «запрос → ответ»

Инвокация — это **не рендеринг** и **не запрос к БД**. Это весь цикл:

```
HTTP request
   ↓
[инвокация начинается]
   ├─ parse cookies/headers/searchParams
   ├─ await db.query(...)        ← idle: fluid compute крутит чужую работу
   ├─ render React tree → HTML + RSC Payload
   └─ return HTTP response
[инвокация закончилась]
```

Рендеринг и запрос к БД — **шаги внутри** инвокации.

### 2. Тёплый (warm) vs холодный (cold) экземпляр

- **Warm** — Vercel переиспользует тот же процесс → module-level `let cache = {}` сохраняется между инвокациями.
- **Cold start** — Vercel поднимает новый экземпляр → модуль переинициализируется → кэш пуст.

→ In-memory кэш в serverless **ненадёжен**. Правильный кэш — внешний (Redis, Vercel KV) или фреймворковый (`'use cache'` в Next.js 16).

### 3. Fluid compute

> «За счёт параллельного выполнения внутри одного экземпляра (и использования idle-времени для вычислений) fluid compute снижает cold starts, уменьшает задержку и экономит на вычислениях.» — [DEP]

Серверный код почти всё время **ждёт** (БД, внешние API). Fluid compute в эти idle-паузы запускает обработку других инвокаций в том же экземпляре → меньше экземпляров = меньше cold starts.

### 4. Data locality — главное правило

> «Функции всегда должны выполняться близко к источнику данных, чтобы снизить задержку.» — [DEP]

По умолчанию Node-функции — в `iad1` (Вашингтон), потому что большинство управляемых БД (Supabase, PlanetScale, Neon) по умолчанию живут в `aws-east-1` (физически тот же регион).

Если БД во Франкфурте (`fra1`) → переносим **функции** в `fra1`. Двигать надо функцию к БД, а не наоборот (БД тяжело двигать, она обычно уже стоит в оптимальном регионе провайдера).

Связка с [Pat] §3.6 Edge SSR:

> «Задержка коннекта между Токио и Сан-Франциско может убить выигрыш от SSR.»

## Сводная таблица: где что исполняется (по всей главе 1.1)

| Стратегия | Где исполняется | Когда |
|---|---|---|
| **Static / SSG (Static Site Generation)** | Сборка → статическое хранилище → CDN / Edge-кэш | один раз при сборке (ISR — фоновая регенерация) |
| **CSR (Client-Side Rendering)** | Браузер клиента | каждый визит |
| **SSR (Node)** | Serverless-функция (Node-рантайм), один регион | каждый запрос |
| **SSR (Edge)** | Edge-функция (Web-рантайм, близко к пользователю) | каждый запрос; TTFB ~30–80 мс; лимиты CPU/памяти |
| **RSC (React Server Components)** | Сервер (Node или Edge), результат → RSC Payload + HTML | рендер на сервере, код не отправляется клиенту |
| **PPR (Partial Prerendering, Cache Components)** | Статический shell при сборке + динамические чанки в `<Suspense>` стримятся при запросе | гибрид: сборка + запрос |
| **Islands** | Сервер (статический HTML) + отдельные JS-острова на клиенте | сборка + точечная гидратация |

## 🔴 Edge vs Node: соединения к БД (СИНТЕЗ — нет якоря в главе)

> **Внимание:** этот раздел — **инженерный синтез**, в якорях [DEP] Vercel Functions его нет. Помечен 🔴 по правилу инвариантов курса. Кандидат на дополнение `content.md` в будущих ревизиях.

Это самый недооценённый аспект выбора Edge vs Node, и он напрямую бьёт по PPR (Partial Prerendering) из подглавы 8.

### Проблема

PPR-страница с `'use cache'` на Data-level часто делает `await db.query(...)`. Вопрос: **как именно устанавливается соединение с БД** на разных рантаймах?

### Node-рантайм: пулы соединений (connection pools)

Node-функция — это полноценный процесс. В нём можно держать **долгоживущий пул соединений** к БД (например, `pg.Pool` для Postgres):

```ts
// lib/db.ts
import { Pool } from 'pg'

// Pool создаётся один раз на экземпляр функции (warm).
// Между инвокациями соединения переиспользуются.
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10, // до 10 одновременных коннектов
})

export const query = (text: string, params?: unknown[]) => pool.query(text, params)
```

Тёплый экземпляр Node-функции:
1. получает инвокацию,
2. берёт свободный коннект из пула (это микросекунды, коннект уже установлен),
3. делает запрос,
4. возвращает коннект в пул,
5. ждёт следующую инвокацию.

TCP/TLS-рукопожатие (handshake) с БД (а это может быть 30–100 мс само по себе) происходит **редко**: только при инициализации пула и при его расширении.

### Edge-рантайм: нет пулов, каждый запрос = новый коннект

Edge-функция — это **Web-стандарт**, а не Node. Она:
- запускается в изоляции V8 (как Service Worker),
- не имеет долгоживущей памяти между инвокациями в том же смысле,
- не поддерживает классические TCP-пулы к БД.

Если ты используешь классический драйвер `pg` (или `mysql2`) на Edge, каждый запрос к БД = **полное рукопожатие TCP + TLS + auth**:
1. TCP handshake (1 RTT ≈ если БД в другом регионе — это десятки мс),
2. TLS handshake (ещё 1–2 RTT),
3. auth (ещё 1 RTT),
4. сам запрос,
5. закрытие.

Это может добавить **сотни миллисекунд** к каждому обращению к БД — и полностью съесть выигрыш от близости Edge к пользователю.

### Решение: HTTP-based драйверы на Edge

На Edge используют **драйверы поверх HTTP**, а не поверх TCP-пула. Каждый запрос = один HTTP-вызов, без установки долгого соединения. Примеры:

- **Neon Postgres** — serverless-драйвер `@neondatabase/serverless` ходит к БД по HTTP (через WebSocket для streaming-вариантов).
- **PlanetScale** — драйвер на основе `fetch()`.
- **Vercel Postgres / Edge-совместимые клиенты** — то же.

```ts
// lib/db.ts — Edge-совместимо
import { neon } from '@neondatabase/serverless'

// Каждый запрос = один HTTP-fetch к БД. Не держит соединений.
const sql = neon(process.env.DATABASE_URL!)

export const getUsers = () => sql`SELECT * FROM users`
```

HTTP-драйвер жертвует эффективностью одного соединения (нет переиспользования TCP-коннекта), но **на Edge это единственный путь**, и в реальных числах это часто **быстрее**, чем «классический» пул на Edge (потому что рукопожатий просто нет — каждый fetch = готовый HTTP-запрос).

### Что это значит для PPR (БД-коннекты)

PPR (Cache Components) из подглавы 8 предполагает, что:
- **Статический shell** строится при сборке → на рантайме не зависит от выбора Node/Edge.
- **Динамические чанки в `<Suspense>`** стримятся при запросе → вот тут и проявляется ограничение:
  - если динамический чанк ходит в БД через **пул** → это Node, не Edge;
  - если хочешь Edge (близко к пользователю) → нужен **HTTP-драйвер** БД.

### Решающее правило (БД-коннекты)

| Если динамическая часть страницы... | Рантайм | Драйвер БД |
|---|---|---|
| Лёгкая (auth-чек, чтение cookie) | Edge | любой |
| Тяжёлые npm-библиотеки (Markdown, syntax highlight) | Node | любой |
| Пул соединений к БД | Node | `pg.Pool`, `mysql2/pool` |
| Близость к пользователю + обращения к БД | Edge | **HTTP-драйвер** (Neon, PlanetScale) |

## 🟢 Где хранится кэш `'use cache'` (по Next.js 16.3.0 — первичный источник)

> Источник: [Next.js 16 — Caching, раздел «Where cached content is stored»](https://nextjs.org/docs/app/getting-started#caching). Это **первичный якорь**, не синтез. ⚠️ В якорях главы 1.1 ([DEP] Vercel Functions) этого материала нет — но он есть в якоре [NEXT] (Next.js 16 Caching), который уже задействован в подглаве 8. **Это не синтез, а неоткрытый фрагмент уже покрытого якоря.**

### Ошибка, которую нужно исправить

Раньше считалось: «Node = диск/shared memory, Edge = только внешний сервис». Это **неправильная дихотомия**. По документации Next.js 16.3.0 дефолтное поведение `'use cache'` — **in-memory per-instance, ephemeral на serverless, в обоих рантаймах**. Различие Node vs Edge для кэша **вообще не водораздел**.

### Три уровня хранения (цитата из источника)

> «A cached function's output is serialized into an **RSC payload**, at build time or at runtime. This payload is what everything else works from. Next.js renders it into HTML, keeps it in a server or remote store, or sends it to the browser, and `cacheLife` sets how long each copy stays fresh.»

| Уровень | Где | durable? | зависит от Node/Edge? |
|---|---|---|---|
| **Prerendered HTML** (static shell) | durable storage платформы за CDN | ✅ durable | ❌ не зависит |
| **`'use cache'` (дефолт)** | in-memory per-instance | ❌ **ephemeral на serverless** | ❌ не зависит |
| **`'use cache: remote'`** | внешний cacheHandler (Vercel KV, Redis) | ✅ durable | ❌ не зависит |
| **`'use cache: private'`** | браузер пользователя | session-scoped | ❌ не зависит |

### Ключевые цитаты

**Про дефолт:**
> «By default the result stays in a **per-instance, in-memory store** that is **ephemeral on serverless**.»

**Про `'use cache: remote'`:**
> «`use cache: remote` moves it to a durable cache handler shared across instances, a network roundtrip that pays off only at a **high hit rate**.»

**Про `'use cache: private'`:**
> «The payload is included in the RSC sent for a client navigation or prefetch, where the browser keeps it fresh for its `stale` window. `use cache: private` results live only here.»

**Про поведение на serverless явно:**
> «At runtime it's cached **in-memory** by default, which **doesn't persist across serverless requests**, so it may re-evaluate on each request. Reach for `'use cache: remote'` for durable, shared caching.»

### Что это значит для PPR

PPR-страница из подглавы 8:

```tsx
async function ProductList() {
  'use cache'        // ← ДЕФОЛТ: in-memory ephemeral
  cacheLife('hours')
  ...
}
```

На serverless-деплое **этот кэш ephemeral**: каждый инстанс функции (см. подглаву 9, warm/cold) держит свою копию, и при cold start / scale-to-zero она теряется. На практике `'use cache'` может **пере-вычисляться на каждом запросе**, если инстансы часто умирают.

Для настоящей долговременной кэш-пользы между запросами → `'use cache: remote'` + настроенный `cacheHandlers` (множественное число) в `next.config.ts`:

```ts
// next.config.ts
const nextConfig: NextConfig = {
  cacheComponents: true,
  cacheHandlers: {
    remote: require('./my-redis-handler'),
  },
}
```

### Что НЕ зависит от Node/Edge

- Дефолтное in-memory ephemeral поведение — **одинаково в обоих рантаймах**.
- `'use cache: remote'` — работает **в обоих рантаймах**.
- `'use cache: private'` — работает **в обоих рантаймах**.
- `cacheHandlers` (множественное число) — конфиг Next.js 16, **единый для обоих рантаймов**.

### Итог

Главное различие для кэша — **не Node vs Edge**, а **дефолтный in-memory (ephemeral) vs `'use cache: remote'` (durable)**. Именно эту дихотомию упустили в первоначальном разборе.

## Checklist: выбор региона и рантайма

- [ ] Где физически стоит БД? (узнать у провайдера БД — регион)
- [ ] В каком регионе Vercel-функции по умолчанию? (`iad1` — обычно надо двигать, если БД не в США)
- [ ] Перенести функции в регион БД через настройки проекта Vercel
- [ ] Для Edge: выбран HTTP-драйвер БД, а не классический пул
- [ ] `'use cache'` (дефолт) = ephemeral → для durable нужен `'use cache: remote'` + `cacheHandlers`
- [ ] Для дашборда/чекаута: рассмотрен PPR (static shell + `<Suspense>`), а не чистый SSR/CSR

## Источники

- Якорь [DEP]: https://vercel.com/docs/functions (разделы «Vercel Functions», «How the Vercel Functions lifecycle works», «Why functions should run near your data source»)
- Связка с [Pat] §3.6 Edge SSR: https://www.patterns.dev/react/server-side-rendering (раздел про edge)
- 🔴 Раздел «Edge vs Node: соединения к БД» — **инженерный синтез**, в якорях главы материала нет. Кандидат на дополнение `content.md`.
- 🟢 Раздел «Где хранится кэш `'use cache'`» — [Next.js 16 — Caching](https://nextjs.org/docs/app/getting-started#caching), раздел «Where cached content is stored». Неоткрытый фрагмент уже покрытого якоря [NEXT]; нужно добавить в `content.md` подглавы 8.

## Связанные заметки

- [[React Server Components]]
- [[Next.js Server and Client Components]]
- [[Partial Prerendering]]