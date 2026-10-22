---
created: '2026-08-05'
difficulty: intermediate
mastery: growing
source: Глава 1.1 курса «Фронтенд senior+ в агентную эру», подглава 6. Якорь [Pat]
  patterns.dev — React Server Components
status: analyzing
tags:
- frontend
- rendering-strategies
- rsc
- react
- nextjs
- server-components
updated: '2026-08-05'
---

# ⚛️ Topic: React Server Components (RSC)

> [!abstract] Executive Summary (ELI5)
> RSC — это компоненты React, которые выполняются **только на сервере** и **никогда не отправляют свой код в браузер**. В браузер приходит готовый результат (сериализованное дерево → HTML), а не исходник и не JS-bundle. Главная инверсия: раньше все компоненты по умолчанию были клиентскими, теперь — наоборот, **по умолчанию серверный**, а `'use client'` нужно добавлять явно.

> Источник: глава 1.1 курса, подглава 6. Якорь: [Pat] patterns.dev, «React Server Components».

---

## 🎯 The "Why"

- **Problem Solved:** Тяжёлые библиотеки (highlight.js, moment, Markdown-рендеры) попадают в клиентский bundle и замедляют загрузку. Секреты (API keys) нельзя использовать в клиентских компонентах. Данные требуют API-слоя: клиент → fetch → сервер → БД.
- **Alternatives:**
    - [[SSR-Server-Side-Rendering]] — рендерит всё дерево в HTML на каждый запрос.
    - [[Islands-Architecture]] — много независимых деревьев, статический HTML + острова JS.
    - Qwik (Resumability) — вообще без гидратации, события в HTML.
- **Trade-offs:** RSC убирает JS из bundle для статичных частей, но платит: серверные вычисления стоят денег («serverless or container minutes»), RSC-payload добавляет байты при навигации.

---

## 💡 Mental Model & Analogy

- **Аналогия:** CSR — ты покупаешь готовый набор конструктора и сам его собираешь (код + bundle на твоём устройстве). RSC — ты получаешь уже собранную фигуру (готовый результат), а инструкцию и детали оставляют на заводе (код не отгружается).
- **Ключевая инверсия:** раньше по умолчанию все компоненты запускались в браузере. Теперь — наоборот: **по умолчанию компонент серверный**, чтобы он стал клиентским, нужно явно добавить `'use client'`.

```
Старый мир (CSR):  клиент ← код всех компонентов + bundle
Новый мир (RSC):   клиент ← только результат серверных + JS только клиентских
```

---

## 🛠️ Core Principles

### Принцип RSC (определение)

> «never ship their code to the browser» — код серверных компонентов никогда не отправляется в браузер. Клиент получает **результат**, который чередуется («interleave») с обычными клиентскими компонентами.

### Инверсия по умолчанию

| До RSC | С RSC |
|---|---|
| Все компоненты по умолчанию клиентские | Все компоненты по умолчанию **серверные** |
| Чтобы стать серверным — нужно явно включить | Чтобы стать клиентским — нужно явно включить `'use client'` |

```tsx
'use client'   // ← явное включение; без неё — серверный

import { useState } from 'react'

export function Counter() {
  const [n, setN] = useState(0)
  return <button onClick={() => setN(n + 1)}>{n}</button>
}
```

---

## 🆚 RSC vs SSR (критическая таблица)

Это самый коварный момент — обе технологии работают на сервере, но закрывают **разные задачи**.

| Аспект | SSR | RSC |
|---|---|---|
| Что выполняется на сервере | При SSR рендерится **всё дерево** при каждом запросе | Только компоненты, помеченные как серверные |
| Что отправляется клиенту | HTML + JS для **всего дерева** | HTML + JS **только для клиентских компонентов** |
| Затраты на гидратацию | Всё дерево целиком | **Только клиентские компоненты** |
| Формат вывода | HTML | **Сериализованное дерево (RSC payload)**, затем опционально HTML |

**Ключевая строка — последняя.** У SSR формат вывода — HTML. У RSC — RSC-payload (специализированный сериализованный формат, описывающий дерево), из которого **затем** опционально генерируется HTML.

### Гидратация: nuance

- Серверный компонент → нет JS → **не гидрируется** (нечего).
- Клиентский компонент → есть JS → **гидрируется как обычно**.
- Страница целиком = микс → гидрируются **только клиентские куски**.

> [!warning] Ловушка формулировки
> «Не гидрируется» применимо к **серверной части**, а не к странице целиком. Страница как целое гидрируется, но только в клиентских кусках.

### «Первоначальная отрисовка» в определении SSR

«Первоначальная отрисовка» противопоставляется не «верхней части страницы», а **«обновлению дерева без перезагрузки»** (RSC-payload). Стримящиеся части в Streaming SSR — это всё «первоначальная отрисовка», просто разбитая по времени доставки. Граница: **первый визит vs обновления на живой странице**.

---

## ✅ Преимущества RSC

### 1. Прямые запросы к БД — без API-слоя

Серверным компонентам не нужен слой API. Они могут напрямую обращаться к БД:

```tsx
// Серверный компонент (по умолчанию)
export default async function UserProfile({ id }: { id: string }) {
  const user = await db.users.findUnique({ where: { id } })
  return <h1>Привет, {user.name}</h1>
}
```

### 2. Безопасность секретов

Переменные окружения, ключи API, URL баз данных — можно использовать безопасно, потому что они «never be bundled for the browser».

```tsx
// ✅ Безопасно: серверный компонент
const apiKey = process.env.STRIPE_SECRET_KEY   // в bundle не попадёт
```
```tsx
// ❌ ОПАСНО: если бы это был клиентский — ключ утёк бы в браузер
'use client'
const apiKey = process.env.STRIPE_SECRET_KEY   // ← в bundle клиента!
```

### 3. Оптимизация bundle (20–40% меньше JS)

Тяжёлые библиотеки (highlight.js, moment, Markdown-рендеры) в серверных компонентах **не добавляют ни байта** в клиентский код.

> «20–40 % reduction in JS shipped to the client» — сокращение JS, отправляемого клиенту, на 20–40%.

---

## ⚠️ Недостатки RSC

1. **Серверные вычисления стоят денег.** Раньше клиент делал это бесплатно у себя; теперь это крутится на ваших серверах → растут траты на «serverless or container minutes».
2. **RSC-payload добавляет байты при навигации.** Но в большинстве случаев объём сэкономленного JS значительно превышает размер добавленного payload. Чистый баланс — в плюс.

---

## 🔒 Ограничения серверных компонентов

Серверным компонентам **недоступны**:

| Что запрещено | Почему |
|---|---|
| React-хуки жизненного цикла (`useState`, `useEffect`) | На сервере нет жизненного цикла рендеринга клиента |
| Обработчики событий (`onClick`, `onChange`) | Серверный код не обрабатывает клики пользователя |
| Браузерные API (`window`, `localStorage`, `IntersectionObserver`) | На сервере нет браузера |

---

## 🧬 Правила композиции (важнейший раздел)

### Правило 1. Серверный → может рендерить клиентский (через пропсы)

```tsx
// ServerComponent.tsx (серверный, по умолчанию)
import ClientCounter from './ClientCounter'

export default async function ServerComponent() {
  const data = await db.query('...')
  return <ClientCounter initial={data.count} />   // ✅ передаём пропсы
}
```

### Правило 2. Клиентский ← может рендерить серверный ТОЛЬКО через children

```tsx
// ✅ РАЗРЕШЕНО: серверный передан как children
// Layout.tsx (серверный)
import ClientWrapper from './ClientWrapper'
import ServerInside from './ServerInside'

export default function Layout() {
  return (
    <ClientWrapper>
      <ServerInside />     {/* ← передаётся как children */}
    </ClientWrapper>
  )
}
```
```tsx
// ❌ ЗАПРЕЩЕНО: прямой импорт серверного в клиентском
'use client'
import ServerInside from './ServerInside'   // ← всё импортируемое
                                            //   становится клиентским
```

> **Логика правила:** всё, что импортируется внутри клиентского дерева, также становится клиентским кодом. Поэтому прямой импорт невозможен — фреймворк просто «утянет» серверный код в клиентский bundle.

### Правило 3. Данные через границу сервер→клиент должны быть сериализуемыми

| ✅ Можно передавать | ❌ Нельзя передавать |
|---|---|
| Числа, строки, массивы, объекты | Классовые инстансы (экземпляры классов) |
| Промиссы (Promise) | Функции |
| Элементы JSX | |

```tsx
// ✅ ОК: число, строка, массив, промис
<ClientComp count={42} name="Pavel" tags={['a', 'b']} promise={db.query('...')} />

// ❌ ОШИБКА: инстанс класса или функция
<ClientComp user={new User('Pavel')} onClick={handleClick} />
```

---

## 🔁 Data Fetching и асинхронные компоненты

Серверные компоненты — это **асинхронные функции**, позволяющие `await` прямо в теле:

> «await directly in the function body».

Сравнение «до и после»:

**Старый мир (CSR):**
```tsx
'use client'
function ProductList() {
  const [products, setProducts] = useState([])
  useEffect(() => {                 // ← «сложные танцы с useEffect»
    fetch('/api/products')
      .then(r => r.json())
      .then(setProducts)
  }, [])
  if (!products.length) return <Spinner />
  return products.map(p => <Product key={p.id} {...p} />)
}
```

**Новый мир (RSC):**
```tsx
export default async function ProductList() {
  const products = await db.products.findMany()   // ← await прямо в теле
  return products.map(p => <Product key={p.id} {...p} />)
}
```

- Устраняет необходимость `useEffect` для первоначальных данных.
- Получение данных — **в том же процессе**, что и рендеринг (а не «клиент скачал JS → пошёл в сеть»).

---

## 🌊 Streaming SSR + Suspense в Next.js App Router

Файловая система тесно интегрирована с RSC:

| Файл | Назначение |
|---|---|
| `layout.tsx` | каркас страницы (шапка, навигация, футер) |
| `page.tsx` | основной контент страницы |
| `loading.tsx` | **`<Suspense>` fallback для первоначального рендера `page.tsx`** |

Механика стриминга:

```
Каркас (layout.tsx) готов → отправляется НЕМЕДЛЕННО
                ↓
Медленные / асинхронные участки дерева
  автоматически оборачиваются в <Suspense>
                ↓
Фреймворк отправляет fallback'и сразу,
а данные для остальных частей — ПО МЕРЕ ГОТОВНОСТИ
                ↓
Формируется «single streaming response» — единый потоковый ответ
```

> RSC + Suspense = современная реализация Streaming SSR. Серверный компонент сам по себе асинхронный (`await db...`), Suspense-граница вокруг него позволяет стримить именно ту часть, которая «ждёт».

---

## 🧠 Главный паттерн: «серверный как обёртка над клиентским»

Самый распространённый паттерн в реальной жизни:

```tsx
// ProductPage.tsx — СЕРВЕРНЫЙ (по умолчанию, fetch'ит данные)
import AddToCartButton from './AddToCartButton'   // клиентский

export default async function ProductPage({ id }) {
  const product = await db.products.findUnique({ where: { id } })

  return (
    <article>
      <h1>{product.name}</h1>
      <p>{product.description}</p>          {/* статичный контент — сервер */}
      <AddToCartButton productId={product.id} initial={product.inCart} />
      {/* ↑ интерактивность — клиентский остров */}
    </article>
  )
}
```

> [!important] Главный инсайт
> Серверный компонент **не обязан** быть обёрткой. Если интерактивности нет — клиентский компонент не нужен вовсе. Правильный вопрос: **«нужна ли этому компоненту интерактивность?»** Если нет — серверный сам по себе.

**Пример — статьи блога с highlight.js (100 КБ):**

```tsx
// Article.tsx — серверный, по умолчанию
import hljs from 'highlight.js'   // 100 КБ, но НЕ попадёт в клиентский bundle

export default async function Article({ slug }: { slug: string }) {
  const article = await db.articles.findUnique({ where: { slug } })
  const html = hljs.highlightAuto(article.body).value

  return (
    <article>
      <h1>{article.title}</h1>
      <pre><code dangerouslySetInnerHTML={{ __html: html }} /></pre>
    </article>
  )
}
```

Без клиентского компонента вообще — `highlight.js` остался на сервере, данные из БД напрямую, HTML готов к отображению.

---

## 🔗 Связанные заметки

- [[SSR-Server-Side-Rendering]] — RSC работает вместе с SSR (а не заменяет).
- [[Static-Rendering]] — статика как базовый случай серверного рендеринга.
- [[CSR-Client-Side-Rendering]] — «до RSC».
- [[Islands-Architecture]] — Server Island как аналог RSC, но HTML-фрагмент, не дерево.
- [[Core-Web-Vitals]] — TTFB, FCP, LCP, INP, CLS.

## 📚 Источники

- Глава 1.1 курса «Фронтенд senior+ в агентную эру», подглава 6.
- Якорь [Pat] — patterns.dev, «React Server Components»: https://www.patterns.dev/react/react-server-components/