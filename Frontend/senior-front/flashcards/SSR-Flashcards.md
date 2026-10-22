#flashcards/frontend/rendering-strategies/ssr

# 🎴 SSR (Server-Side Rendering) — Flashcards

Source: [[SSR-Server-Side-Rendering]]. Глава 1.1 курса, подглава 3. Якорь [Pat] patterns.dev.

## Базовая концепция

Чем SSR отличается от статического рендеринга (SSG) по оси «когда»?
?
И статика, и SSR рендерят на сервере — разница в моменте. **Статика** рендерит один раз при сборке (`npm run build`), HTML один на всех и лежит на CDN. **SSR** рендерит на каждый запрос пользователя → HTML уникален для каждого визита → возможна персонализация.
#flashcards/frontend/rendering-strategies/ssr/foundations

Назови 4 триггера, когда SSR — выбор по умолчанию (по источнику [Pat]).
?
1. **signed-in user** (залогиненный пользователь); 2. **cookie** (данные из cookie); 3. **A/B-test cohort** (группа A/B-теста); 4. **geolocation header** (геолокация). Общий принцип: контент зависит от конкретного запроса → нужен SSR. Если контент один на всех → статика.
#flashcards/frontend/rendering-strategies/ssr/foundations

Почему статика **архитектурно не может** показать баланс счёта пользователя?
?
Потому что статический HTML один на всех (рендерится раз при сборке). Баланс — персональные данные, их нельзя вшить в общий HTML. SSR может: сервер видит запрос → видит cookie → рендерит под этот визит → HTML уникален.
#flashcards/frontend/rendering-strategies/ssr/foundations

## Lifecycle и гидратация

Назови два этапа lifecycle SSR.
?
**Этап 1 (сервер):** React-компоненты → HTML → отправлен пользователю. **Этап 2 (клиент):** React «оживляет» HTML — гидратация (прикрепляет event listeners, связывает DOM с деревом компонентов, делает интерактивной).
#flashcards/frontend/rendering-strategies/ssr/hydration

Что такое гидратация (определение из источника)?
?
Это процесс, при котором React: (1) обходит серверный DOM; (2) сопоставляет узлы с деревом компонентов; (3) прикрепляет event listeners («attaching event listeners and reconnecting the DOM to the component tree so it becomes interactive»). Визуально ничего не меняется, но стоит ~как первый клиентский рендер.
#flashcards/frontend/rendering-strategies/ssr/hydration

Может ли гидратация происходить на сервере?
?
**Нет.** Гидратация — по определению клиентский процесс. Рендеринг = сервер (или сборка); гидратация = клиент. Никогда не наоборот.
#flashcards/frontend/rendering-strategies/ssr/hydration

## Преимущества и недостатки

Назови 3 преимущества SSR.
?
**1. Load Performance** — LCP улучшается, потому что «main content is in the initial HTML». **2. SEO** — краулеры видят «fully rendered markup, no JS execution required». **3. Personalization** — request headers, cookies, session data доступны в момент рендеринга.
#flashcards/frontend/rendering-strategies/ssr/tradeoffs

Назови 3 недостатка SSR.
?
**1. Slower TTFB** — «work happens per request» (сервер работает на каждый визит). **2. Server Costs** — «every request runs the render» (дорогой рантайм). **3. Hydration Expense** — клиенту «still needs the JS to make the UI interactive».
#flashcards/frontend/rendering-strategies/ssr/tradeoffs

Какая метрика ловит недостаток «Hydration Expense»?
?
**INP (Interaction to Next Paint).** В uncanny valley клики «тихо теряются» — INP измеряет задержку между действием пользователя и следующей отрисовкой.
#flashcards/frontend/rendering-strategies/ssr/tradeoffs

## Streaming SSR

Какой серверный метод для Node, а какой — для Edge/Web? Почему два разных?
?
**Node:** `renderToPipeableStream` (использует `pipe` из Node Streams). **Edge/Web:** `renderToReadableStream` (использует `ReadableStream` из Web Streams API). Edge-среды — не Node, у них нет `pipe`. Это разные среды с разными API работы с потоками.
#flashcards/frontend/rendering-strategies/ssr/streaming

Что такое «shell» в streaming SSR, и что делает колбэк `onShellReady`?
?
**Shell** — статическая часть страницы **вне** Suspense-границ (шапка, навигация, футер) — не ждёт данных. **`onShellReady`** срабатывает, как только shell готов, и сервер немедленно flush'ит (отправляет) его клиенту. Браузер отрисовывает shell, пока данные в Suspense продолжают резолвиться.
#flashcards/frontend/rendering-strategies/ssr/streaming

В чём разница между shell и Suspense fallback?
?
**Shell** — статический layout страницы (структура: шапка/навигация/футер), вне Suspense. **Suspense fallback** — плейсхолдер конкретного блока (скелетон), внутри Suspense. Shell отправляется первым; fallback'и потом подменяются на стримящиеся чанки.
#flashcards/frontend/rendering-strategies/ssr/streaming

Зачем краулерам нужен `onAllReady`, а не стрим?
?
Некоторые краулеры слишком простые и не выполняют стримящиеся скрипты. Если отдать им shell — они проиндексируют только шапку. `onAllReady` ждёт полного HTML за один раз → краулер видит весь контент и индексирует правильно.
#flashcards/frontend/rendering-strategies/ssr/streaming

## Uncanny Valley

Что такое «uncanny valley» в SSR?
?
Состояние, когда страница **визуально выглядит готовой** (HTML есть, LCP пройден), но **не реагирует** на действия пользователя: клики тихо теряются, пока JS-бандл не распарсен и гидратация не завершена. React на кнопке ещё не повесил обработчик → DOM-событие уходит «в никуда». Метрика: **INP**.
#flashcards/frontend/rendering-strategies/ssr/uncanny-valley

Назови 3 паттерна снижения стоимости гидратации (uncanny valley).
?
**1. Streaming SSR + `<Suspense>`** — гидрирует по частям, не ждёт весь бандл (**фундамент**). **2. Selective Hydration** (React 18+) — приоритизирует гидратацию кликнутого компонента. **3. React Server Components** — убирает гидратацию совсем (нет JS → не из чего гидрировать).
#flashcards/frontend/rendering-strategies/ssr/uncanny-valley

Будет ли работать Selective Hydration на странице без единого `<Suspense>`?
?
**Нет.** Selective Hydration опирается на границы Suspense — именно Suspense создаёт «куски», по которым можно приоритизировать гидратацию. Без Suspense нет границ → React не может «разрезать» работу → не к чему применять приоритизацию. Streaming+Suspense = фундамент.
#flashcards/frontend/rendering-strategies/ssr/uncanny-valley

Какой из трёх паттернов снижения uncanny valley — фундамент, без которого остальные не работают?
?
**Streaming SSR + `<Suspense>`.** Он создаёт независимые куски (и для доставки, и для гидратации). Selective Hydration и RSC — надстройки над этой структурой.
#flashcards/frontend/rendering-strategies/ssr/uncanny-valley

## Next.js App Router

Что является поведением по умолчанию в Next.js App Router?
?
**Серверный рендеринг (SSR).** Все компоненты в `app/` рендерятся на сервере по умолчанию, если не помечены `'use client'`. `getServerSideProps` удалён — данные запрашиваются в async-компоненте прямо в теле.
#flashcards/frontend/rendering-strategies/ssr/nextjs

Назови 3 «динамические функции», которые автоматически переводят маршрут в динамический рендеринг (SSR).
?
**1. `cookies()`** (функция из `next/headers`). **2. `headers()`** (функция из `next/headers`). **3. `searchParams`** (пропс страницы). Общее: все дают доступ к данным, зависящим от конкретного запроса → контент перестаёт быть одинаковым для всех → фреймворк переводит в SSR.
#flashcards/frontend/rendering-strategies/ssr/nextjs

Что изменилось с `fetch()` в Next.js 15 и почему?
?
`fetch()` **больше не кэшируется по умолчанию**. Раньше (Next 13-14) он кэшировался молча → данные были «несвежими», разработчик путался. Теперь поведение по умолчанию — всегда свежие данные; кэширование — **явное** через `'use cache'`. Это для **прозрачности**.
#flashcards/frontend/rendering-strategies/ssr/nextjs

Зачем в Next.js 15 `cookies()`, `headers()`, `searchParams` стали асинхронными?
?
`await` создаёт **явную границу**, которую видит фреймворк. До `await` — статическая часть (можно при сборке), после — динамическая (рендер при запросе). Это основа **Partial Prerendering (PPR)**: фреймворк «режет» рендер по точке `await`.
#flashcards/frontend/rendering-strategies/ssr/nextjs

## Edge SSR

Как Edge SSR меняет TTFB (числа из источника)?
?
TTFB падает «typically from 200–600 ms on a centralized origin to 30–80 ms at the edge». Cold start серверлесса стремится к нулю. Edge-точки физически ближе к пользователю → меньше сетевая задержка.
#flashcards/frontend/rendering-strategies/ssr/edge

Назови 3 ограничения edge-рантайма vs Node.
?
**1.** Это подмножество Node: нет нативных модулей (C/C++), нет файловой системы; нужен `renderToReadableStream`. **2.** Жёсткие лимиты CPU — **десятки миллисекунд на запрос**. **3.** Жёсткие лимиты памяти — **50–128 МБ**.
#flashcards/frontend/rendering-strategies/ssr/edge

Что произойдёт с TTFB и LCP, если Edge рендерит в Москве, а БД — на origin во Франкфурте?
?
**TTFB останется быстрым** — shell (вне Suspense) уходит сразу, не ждёт данных. **Но LCP пострадает** — главный контент ждёт данных из Франкфурта (~200 мс сетевой задержки). Edge купил TTFB, но не LCP. **Решение:** edge-реплицированные данные (Redis/Turso/D1 рядом с Edge).
#flashcards/frontend/rendering-strategies/ssr/edge

## Когда SSR НЕ нужен

Назови 3 сценария, когда SSR использовать НЕ надо.
?
**1. Truly Static Content** (блог, доки, лендинг) → статика (тот же HTML без затрат). **2. Highly Interactive Apps** (Figma, app shell) → «hydration cost outweighs SSR's benefit» → CSR. **3. Tolerable Staleness** (данные терпят устаревание) → ISR с revalidation.
#flashcards/frontend/rendering-strategies/ssr/when-not

Что такое «Tolerable Staleness» и какой паттерн вместо SSR?
?
«Допустимое устаревание» — данные меняются, но пользователю не критично видеть их абсолютно свежими (цены, счётчик лайков). Нет смысла платить за SSR per-request ради избыточной свежести. **Альтернатива:** ISR с revalidation — «почти свежие» данные по цене статики (CDN-edge-кэш).
#flashcards/frontend/rendering-strategies/ssr/when-not

Что такое «app shell» и почему высокоинтерактивным приложениям SSR вредит?
?
**App shell** — стабильная оболочка, загружается один раз, дальше всё взаимодействие внутри неё без перезагрузок (Figma, Maps). Для них «hydration cost outweighs SSR's benefit»: интерфейс — тысячи событий, гидратация всего дерева очень дорогая, а выгоды SSR (LCP/SEO) не нужны. Предпочтительнее CSR.
#flashcards/frontend/rendering-strategies/ssr/when-not

## Production-чеклист

Что такое request-scoped кэш (React `cache()`) и какую проблему решает?
?
`cache()` **дедуплицирует запросы внутри одного рендера**. Если `getUser()` вызывается из нескольких компонентов одной страницы, без `cache()` он пойдёт в БД многократно. С `cache()` — один раз, остальные вызовы берут из кэша. Это **request-scoped** (живёт один рендер), не путать с кэшем `fetch()` (между визитами).
#flashcards/frontend/rendering-strategies/ssr/checklist

Почему JS-бандл должен быть разделён по маршрутам (route splitting)?
?
Чтобы гидратации нужен был **только JS текущей страницы**. Если весь JS в одном бандле, пользователю на `/products` приходится качать и `/cart`, и `/blog` — гидратация ждёт весь бандл → uncanny valley дольше. Route splitting: качаю только свой чанк → гидратация быстрее.
#flashcards/frontend/rendering-strategies/ssr/checklist
