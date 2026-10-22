#flashcards/frontend/rendering-strategies/nextjs

# 🧩 Next.js Server and Client Components — Flashcards

Source: [[Next.js Server and Client Components]]. Глава 1.1 курса, подглава 7. Якорь [NEXT] Next.js 16.

## Поведение по умолчанию

Что является поведением по умолчанию для layouts и pages в Next.js App Router?
?
**Server Components (серверные компоненты).** Это позволяет fetching'ать данные, рендерить UI на сервере, опционально кэшировать результат и стримить его клиенту. Когда нужна интерактивность или браузерные API — используется `'use client'`.
#flashcards/frontend/rendering-strategies/nextjs/foundations

Назови 4 случая, когда нужен Client Component (`'use client'`).
?
**1.** State и event handlers (`onClick`, `onChange`). **2.** Lifecycle logic (`useEffect`). **3.** Browser-only APIs (`localStorage`, `window`, `Navigator.geolocation`). **4.** Custom hooks.
#flashcards/frontend/rendering-strategies/nextjs/foundations

Назови 4 случая, когда подходит Server Component.
?
**1.** fetch'ить данные из БД или API близко к источнику. **2.** использовать API keys, токены, секреты (не раскрывая клиенту). **3.** уменьшить объём JavaScript в браузере. **4.** улучшить FCP (First Contentful Paint) и стримить контент прогрессивно.
#flashcards/frontend/rendering-strategies/nextjs/foundations

## RSC Payload

Что такое RSC Payload?
?
**Компактное бинарное представление** отрендеренного дерева React Server Components. Используется React на клиенте для обновления DOM браузера. Это **НЕ HTML** — это бинарный формат, из которого HTML генерируется опционально.
#flashcards/frontend/rendering-strategies/nextjs/rsc-payload

Назови 3 содержимого RSC Payload.
?
**1. Результат рендеринга Server Components** (их вывод, в бинарном виде). **2. Плейсхолдеры** для мест, где должны рендериться Client Components, и **ссылки на их JS-файлы** (не сами файлы). **3. Любые пропсы**, переданные из Server Component в Client Component.
#flashcards/frontend/rendering-strategies/nextjs/rsc-payload

Содержит ли RSC Payload HTML клиентских компонентов?
?
**НЕТ.** RSC Payload содержит только плейсхолдеры и ссылки на JS-файлы для Client Components. HTML клиентских компонентов живёт **в HTML-странице** (отдельный канал доставки) при первом запросе. RSC Payload и HTML-страница — разные каналы.
#flashcards/frontend/rendering-strategies/nextjs/rsc-payload

## Три канала доставки

Назови три канала доставки Next.js и что каждый содержит для Client Components.
?
**1. HTML-страница** — ✅ prerender'ный HTML Client Components (сервер их отрендерил). Только при первом запросе. **2. RSC Payload** — ❌ плейсхолдеры + ссылки на JS-файлы (HTML не лежит). И при первом, и при последующих. **3. JS-бандлы** — код Client Components. По мере необходимости.
#flashcards/frontend/rendering-strategies/nextjs/channels

## Первая загрузка vs последующие навигации

Назови 3 шага на клиенте при первой загрузке (строгий порядок).
?
**1. HTML** — для немедленного показа быстрого **неинтерактивного preview** (пользователь видит страницу, клики ещё не работают — uncanny valley). **2. RSC Payload** — для reconcile (согласования) деревьев Client и Server Components. **3. JavaScript** — для гидратации Client Components, делает приложение интерактивным.
#flashcards/frontend/rendering-strategies/nextjs/lifecycle

Чем отличается первая загрузка от последующей навигации?
?
**Первая загрузка:** HTML + RSC Payload + JS (гидратация). **Последующие навигации:** HTML с сервера **не приходит**; RSC Payload prefetch'ится и кэшируется → навигация мгновенная; Client Components рендерятся **целиком на клиенте**, без серверного HTML.
#flashcards/frontend/rendering-strategies/nextjs/lifecycle

Что такое гидратация (определение из источника)?
?
**Hydration** — процесс React по **прикреплению event handlers (обработчиков событий)** к DOM, чтобы сделать статический HTML интерактивным. HTML уже есть (из шага 1), но он «мёртвый» — клики не работают. Гидратация его «оживляет».
#flashcards/frontend/rendering-strategies/nextjs/lifecycle

Почему при последующих навигациях HTML больше не нужен?
?
Потому что к этому моменту: (1) JS-рантайм React уже загружен и работает на клиенте; (2) RSC Payload prefetch'ится → React сразу строит дерево; (3) Client Components рендерятся **на клиенте** (JS уже есть) → React сам нарисует. Первая загрузка платит «налог на старт», потом — уже нет.
#flashcards/frontend/rendering-strategies/nextjs/lifecycle

## Module graph и граница `'use client'`

Что объявляет директива `"use client"`?
?
**Границу** между серверным и клиентским **графами модулей (module graphs)**. Как только файл помечен `"use client"`, **все его импорты и компоненты, которые он напрямую рендерит, попадают в клиентский bundle**. Не нужно добавлять директиву в каждый компонент — достаточно пометить корневой.
#flashcards/frontend/rendering-strategies/nextjs/module-graph

Применяется ли правило module graph к Server Components, переданным как `children`?
?
**НЕТ.** Server Components, переданные как `children` или другие пропсы, **не импортируются** в module graph клиентского компонента. Они рендерятся на сервере и передаются клиентскому компоненту как уже отрендеренный вывод. Поэтому не утягиваются в клиентский bundle.
#flashcards/frontend/rendering-strategies/nextjs/module-graph

Нужно ли добавлять `'use client'` в каждый клиентский компонент?
?
**НЕТ.** Достаточно пометить «корневой» клиентский компонент — и всё его поддерево (импорты и прямо рендеримые компоненты) автоматически станет клиентским. `'use client'` объявляет границу графа модулей.
#flashcards/frontend/rendering-strategies/nextjs/module-graph

## Уменьшение bundle

Как уменьшить размер клиентского JS-bundle в Next.js?
?
Добавлять `'use client'` к **конкретным интерактивным компонентам**, а не помечать большие части UI как Client Components. **Принцип:** `'use client'` — точечно, не блоками. Например, Layout остаётся серверным; `'use client'` получает только интерактивный `<Search />` внутри.
#flashcards/frontend/rendering-strategies/nextjs/bundle

В чём проблема этого кода и как исправить? `// layout.tsx` `'use client'` `import { useState } from 'react'` + Layout с Header/Nav/SearchBar/Footer.
?
Проблема: весь Layout (включая статичные Header/Footer) уехал в клиентский bundle. **Решение:** Layout остаётся серверным; логика клика переносится **внутрь** Nav (который и так клиентский); SearchBar общается с Nav через стейт/Context. Корень клиентского поддерева должен быть минимальным.
#flashcards/frontend/rendering-strategies/nextjs/bundle

## Interleaving (чередование)

Можно ли передать Server Component как пропс в Client Component? Как?
?
**ДА.** Server Component можно передать как `children` (или другой пропс) в Client Component. Это позволяет визуально вкладывать серверный UI внутрь клиентских. Например: `<Modal><Cart /></Modal>` где Modal — клиентский, Cart — серверный.
#flashcards/frontend/rendering-strategies/nextjs/interleaving

Можно ли прямо импортировать Server Component в Client Component? Почему?
?
**НЕТ.** Всё, что импортируется внутри клиентского дерева, становится клиентским кодом — фреймворк «утянет» серверный код в клиентский bundle. Серверный код не должен стать клиентским — иначе смысл RSC теряется.
#flashcards/frontend/rendering-strategies/nextjs/interleaving

Напиши рабочий паттерн interleaving (Cart серверный внутри Modal клиентского).
?
```tsx
// page.tsx — СЕРВЕРНЫЙ
import Modal from './ui/modal'
import Cart from './ui/cart'

export default function Page() {
  return (
    <Modal>
      <Cart />           {/* ← передаётся как children */}
    </Modal>
  )
}
```
```tsx
// modal.tsx
'use client'
export default function Modal({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>
}
```
#flashcards/frontend/rendering-strategies/nextjs/interleaving

## Механика пропсов в children

Что происходит с пропсами серверного компонента, переданного как `children` в клиентский?
?
**Пропсы «умирают» на сервере.** Сервер **выполняет** серверный компонент с пропсами (например, `productId={42}`), получает готовый JSX, сериализует его в RSC Payload. К моменту пересечения границы Server → Client пропсов уже нет — есть только **готовый отрендеренный вывод**.
#flashcards/frontend/rendering-strategies/nextjs/props-mechanics

Как клиент «понимает» пропсы серверного компонента из children, если HTML не понимает пропсы?
?
**Клиенту не нужно понимать пропсы.** Сервер обрабатывает пропсы заранее: выполняет компонент (`db.products.findUnique(...)`), получает результат, сериализует готовый JSX в RSC Payload. На клиенте React просто **вставляет готовое поддерево** в слот `children` — он не вызывает серверный компонент (кода нет) и не получает пропсы.
#flashcards/frontend/rendering-strategies/nextjs/props-mechanics

Аналогия «заморозки» для механики пропсов в children — поясни.
?
- Серверный компонент с пропсами = повар готовит блюдо по заказу.
- Выполнение на сервере = повар готовит (запросы к БД).
- RSC Payload = замороженное готовое блюдо.
- Клиент = микроволновка разогревает (не готовит заново, не знает рецепт).
«Микроволновка» (React на клиенте) не знает рецепт и не получает ингредиенты (пропсы). Она получает готовое блюдо и разогревает.
#flashcards/frontend/rendering-strategies/nextjs/props-mechanics

## Расхождение с картой

Какое мажорное расхождение с картой зафиксировано в подглаве 7?
?
Классический раздел Next.js «Rendering» (`/docs/app/building-your-application/rendering`) в Next.js **16.2.12** больше не существует как самостоятельная страница — он **редиректит** на **Caching (Cache Components)**. Материал расслоился на *Server and Client Components* и *Caching / Partial Prerendering*. Старые `getServerSideProps`/`getStaticProps` ушли. Покрытие 🟢, но по другой структуре документа.
#flashcards/frontend/rendering-strategies/nextjs/map-divergence
