---
created: '2026-08-05'
difficulty: intermediate
mastery: growing
source: Глава 1.1 курса «Фронтенд senior+ в агентную эру», подглава 7. Якорь [NEXT]
  Next.js 16 — Server and Client Components
status: analyzing
tags:
- frontend
- rendering-strategies
- nextjs
- rsc
- server-components
- client-components
- app-router
updated: '2026-08-05'
---

# 🧩 Topic: Next.js Server and Client Components

> [!abstract] Executive Summary (ELI5)
> В Next.js App Router по умолчанию все layouts и pages — **Server Components** (серверные). Чтобы компонент стал **Client Component** (клиентским), нужно явно добавить `'use client'`. Next.js оркестрирует их вместе: Server Components рендерятся в **RSC Payload** (бинарное дерево), а Client Components вместе с payload'ом используются для **prerender HTML**. При первой загрузке клиент получает HTML (preview) + RSC Payload + JS (для гидратации). При последующих навигациях HTML больше не приходит — только prefetch'нутый RSC Payload, и Client Components рендерятся на клиенте.

> Источник: глава 1.1 курса, подглава 7. Якорь: [NEXT] Next.js 16, «Server and Client Components».

---

## ⚠️ Расхождение с картой (мажорное)

Классический раздел Next.js «Rendering» (`/docs/app/building-your-application/rendering`) в Next.js **16.2.12** больше не существует как самостоятельная страница — он **редиректит** на **Caching (Cache Components)**. Материал расслоился на две страницы: *Server and Client Components* (эта заметка) и *Caching / Partial Prerendering* (см. [[Partial-Prerendering]]). Старые `getServerSideProps` / `getStaticProps` ушли. Покрытие 🟢, но по другой структуре документа.

---

## 🎯 The "Why"

- **Problem Solved:** В чистом React RSC (подглава 6) — это концепция. Next.js App Router — это **конкретная реализация**: как именно фреймворк разбивает рендер на чанки, что отправляется на первой загрузке vs при навигации, как работает граница `'use client'` через module graph.
- **Alternatives:** Pages Router (устарел), чистый React без фреймворка (нужно самому собирать RSC-инфраструктуру).
- **Trade-offs:** App Router даёт Server Components по умолчанию (малый bundle, прямой доступ к БД), но требует понимания module graph, prerender mechanics и трёх каналов доставки (HTML / RSC Payload / JS).

---

## 🛠️ Когда что использовать (чек-листы из источника)

### Client Components — когда нужны:

| Потребность | Пример API |
|---|---|
| State и event handlers | `onClick`, `onChange` |
| Lifecycle logic | `useEffect` |
| Browser-only APIs | `localStorage`, `window`, `Navigator.geolocation` |
| Custom hooks | любая логика, инкапсулирующая хуки |

### Server Components — когда нужно:

| Потребность | Зачем |
|---|---|
| fetch'ить данные из БД или API близко к источнику | без API-слоя |
| использовать API keys, токены, секреты | безопасность (не попадают в bundle) |
| уменьшить объём JavaScript в браузере | bundle |
| улучшить FCP (First Contentful Paint) и стримить контент | метрики |

---

## 📦 Три канала доставки (критическая модель)

Next.js доставляет контент **тремя разными каналами**. Их нужно различать:

| Канал | Что содержит для Client Components | Когда приходит |
|---|---|---|
| **HTML-страница** | ✅ **prerender'ный HTML** Client Components (сервер их отрендерил в HTML) | только при первом запросе |
| **RSC Payload** | ❌ **плейсхолдеры + ссылки на JS-файлы** (HTML НЕ лежит) | и при первом, и при последующих |
| **JS-бандлы** | код Client Components | по мере необходимости |

> [!important] Частая путаница
> RSC Payload **НЕ содержит** HTML клиентских компонентов. HTML клиентских компонентов живёт в HTML-странице (отдельный канал). RSC Payload содержит только плейсхолдеры и ссылки на JS-файлы.

### Что такое RSC Payload (по источнику)?

> RSC Payload — это **компактное бинарное представление** отрендеренного дерева React Server Components. Используется React на клиенте для обновления DOM браузера.

**Три содержимого RSC Payload:**
1. **Результат рендеринга Server Components** (их вывод, в бинарном виде).
2. **Плейсхолдеры** для мест, где должны рендериться Client Components, и **ссылки на их JS-файлы** (не сами файлы).
3. **Любые пропсы**, переданные из Server Component в Client Component.

---

## ⚙️ Lifecycle: первая загрузка vs последующие навигации

### Первая загрузка — три шага на клиенте (строгий порядок)

| Шаг | Что используется | Зачем |
|---|---|---|
| **1. HTML** | для **немедленного показа** быстрого **неинтерактивного preview** | пользователь ВИДИТ страницу сразу (uncanny valley — клики ещё не работают) |
| **2. RSC Payload** | для **reconcile (согласования)** деревьев Client и Server Components | синхронизация |
| **3. JavaScript** | для **гидратации Client Components** и делает приложение интерактивным | интерактивность |

> [!note] Что такое гидратация
> **Hydration** — процесс React по **прикреплению event handlers (обработчиков событий)** к DOM, чтобы сделать статический HTML интерактивным.

### Последующие навигации — другая механика

| | Первая загрузка | Последующие навигации |
|---|---|---|
| HTML с сервера | ✅ есть (prerender) | ❌ нет |
| RSC Payload | ✅ есть | ✅ **prefetch'ится и кэшируется** |
| Client Components | гидратируются | **рендерятся целиком на клиенте** |
| Скорость | зависит от данных | **мгновенно** |

---

## 🚧 Граница `'use client'` и module graph (граф модулей)

`"use client"` объявляет **границу** между серверным и клиентским **графами модулей (module graphs)**.

> Как только файл помечен `"use client"`, **все его импорты и компоненты, которые он напрямую рендерит, попадают в клиентский bundle**.

**Важное исключение — Server Components как `children`:**

> Это поведение **не** применяется к Server Components, переданным как `children` или другие пропсы. Такие компоненты **не импортируются** в module graph клиентского компонента. Они рендерятся на сервере и передаются клиентскому компоненту как уже отрендеренный вывод.

```tsx
'use client'
import Child from './Child'      // ← попадает в клиентский bundle
import Lib from 'heavy-lib'      // ← попадает в клиентский bundle

function ClientComp({ children }) {
  return (
    <div>
      <Child />                  // ← клиентский, в bundle
      {children}                 // ← может быть серверным! НЕ в bundle
    </div>
  )
}
```

### Практика — уменьшение bundle

Чтобы уменьшить размер клиентских JS-bundle, добавляйте `'use client'` к **конкретным интерактивным компонентам**, а не помечайте большие части UI как Client Components.

> **Принцип:** `'use client'` — точечно, не блоками. Layout остаётся серверным; `'use client'` получает только интерактивный `<Search />` внутри.

**Архитектурно лучший подход** (от пользователя курса): перенести логику интерактивности **внутрь** самого интерактивного компонента (например, Nav), чтобы корень клиентского поддерева был минимальным.

---

## 🔁 Interleaving — чередование Server и Client Components

Server Component можно передать как **пропс** (часто `children`) в Client Component. Это позволяет визуально вкладывать серверный UI внутрь клиентских.

### Паттерн — `children` как «слот»

```tsx
// app/ui/modal.tsx
'use client'

export default function Modal({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>
}
```
```tsx
// app/page.tsx
import Modal from './ui/modal'
import Cart from './ui/cart'        // ← серверный, fetch'ит данные

export default function Page() {
  return (
    <Modal>
      <Cart />                      // ← серверный передан как children
    </Modal>
  )
}
```

### Ключевое утверждение источника

> Server Components рендерятся на сервере **заранее, даже когда передаются как пропсы** в Client Components.

### Прямой импорт запрещён

```tsx
// ❌ ЗАПРЕЩЕНО: прямой импорт серверного в клиентском
'use client'
import Cart from './Cart'   // ← фреймворк откажется; всё импортируемое
                             //   становится клиентским кодом
```

---

## 🧠 Механика: что происходит с пропсами серверного компонента в children

> [!important] Пропсы «умирают» на сервере
> К моменту, как что-либо пересекает границу Server → Client, пропсов уже нет — есть только **готовый отрендеренный вывод**.

Когда сервер встречает `<Cart productId={42} />` (серверный, передан как children в клиентский `Modal`):

```
Шаг 1: сервер ВЫПОЛНЯЕТ Cart с пропсом productId={42}
        → db.products.findUnique({ where: { id: 42 } })
        → результат: <div><h2>Футболка</h2><p>999₽</p></div>

Шаг 2: результат сериализуется в RSC Payload
        (никаких «пропсов» уже нет — есть готовый JSX)

Шаг 3: HTML prerender — готовая разметка в HTML-странице

Шаг 4: на клиенте React гидрирует Modal
        → видит: children = готовое поддерево из Payload
        → НЕ вызывает Cart (его кода на клиенте нет)
        → вставляет готовый вывод в DOM
```

**Аналогия «заморозки»:**
- Серверный компонент с пропсами = повар готовит блюдо по заказу.
- Выполнение на сервере = повар готовит.
- RSC Payload = замороженное готовое блюдо.
- Клиент = микроволновка разогревает (не готовит заново, не знает рецепт).

«Микроволновка» (React на клиенте) не знает рецепт и не получает ингредиенты (пропсы). Она получает готовое блюдо и просто разогревает.

---

## 🔗 Связанные заметки

- [[React Server Components]] — общая концепция RSC (подглава 6).
- [[SSR-Server-Side-Rendering]] — Next.js App Router использует SSR + RSC вместе.
- [[Static-Rendering]] — статика как один из режимов Next.js.
- [[CSR-Client-Side-Rendering]] — «до RSC» мир.
- [[Islands-Architecture]] — альтернативный подход (Server Islands).
- [[Partial-Prerendering]] — Cache Components / PPR (раздел 8).
- [[Core-Web-Vitals]] — TTFB, FCP, LCP, INP, CLS.

## 📚 Источники

- Глава 1.1 курса «Фронтенд senior+ в агентную эру», подглава 7.
- Якорь [NEXT] — Next.js 16, «Server and Client Components»: https://nextjs.org/docs/app/getting-started/server-and-client-components