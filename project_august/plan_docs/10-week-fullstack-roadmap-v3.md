# 10-Week Fullstack System Design Roadmap v3
## Frontend-focused + Backend coverage + Полное покрытие frontend-тем 2026 + Архитектурные подходы

---

## Часть 1. Frontend-темы, которые спрашивают на собеседованиях (2026)

На основе анализа реальных интервью (Meta, Google, Netflix, Amazon, Яндекс, VK) выделяем **топ-15 frontend-тем** по частоте:

| # | Тема | Middle | Senior | Staff | Частота |
|---|------|--------|--------|-------|---------|
| 1 | **Core Web Vitals** (LCP, INP, CLS) | ★★★ | ★★★ | ★★★ | Очень высокая |
| 2 | **Rendering Strategies** (SSR/CSR/SSG/ISR/RSC/Islands) | ★★ | ★★★ | ★★★ | Очень высокая |
| 3 | **State Management** (server vs client, normalization, optimistic UI) | ★★★ | ★★★ | ★★★ | Очень высокая |
| 4 | **Micro-frontends** (Module Federation, composition patterns) | — | ★★ | ★★★ | Высокая |
| 5 | **Design Systems** (component library, compound components, Storybook) | ★★ | ★★★ | ★★★ | Высокая |
| 6 | **Real-time** (WebSocket, SSE, optimistic updates) | ★★★ | ★★★ | ★★★ | Высокая |
| 7 | **Performance** (virtualization, code splitting, bundle optimization) | ★★★ | ★★★ | ★★★ | Высокая |
| 8 | **Offline-first / PWA** (Service Workers, IndexedDB, Background Sync) | ★★ | ★★★ | ★★ | Средняя |
| 9 | **Security** (XSS, CSP, Trusted Types, CSRF) | ★★ | ★★★ | ★★★ | Средняя |
| 10 | **Accessibility** (keyboard nav, ARIA, focus management) | ★★★ | ★★★ | ★★ | Средняя |
| 11 | **Zero-downtime deployments** (feature flags, atomic deploys) | — | ★★ | ★★★ | Средняя |
| 12 | **Frontend monitoring** (RUM, error tracking, perf metrics) | ★ | ★★ | ★★★ | Средняя |
| 13 | **Resumability** (Qwik vs Hydration vs React Server Components) | — | ★★ | ★★★ | Низкая-средняя |
| 14 | **Image/Media optimization** (WebP/AVIF, srcset, lazy loading) | ★★★ | ★★★ | ★★ | Средняя |
| 15 | **API Design** (GraphQL vs REST, pagination, caching strategies) | ★★★ | ★★★ | ★★★ | Высокая |

---

## Часть 2. Топ-10 архитектурных подходов Senior Frontend 2026

| Ранг | Архитектурный подход | Частота на интервью | Сложность (1-10) | Лучший фреймворк |
|:---:|---|:---:|:---:|:---:|
| 1 | **Multi-Layered Rendering** (SSR/SSG/ISR/RSC/PPR) | **High** | 6 | Next.js 15 (App Router) |
| 2 | **State Management Architecture** (server vs client, optimistic UI) | **High** | 5 | React + TanStack Query + Zustand |
| 3 | **Micro-Frontends & Module Federation 2.0** | **Medium** | 8 | React + Webpack/Rspack |
| 4 | **Real-time & Event-Driven Architecture** | **High** | 9 | React + Socket.io / SSE |
| 5 | **Offline-First & PWA Architecture** | **Medium** | 7 | React/Vue + Workbox |
| 6 | **Component-Driven Design System** | **High** | 4 | React + Storybook + monorepo |
| 7 | **Performance-First Architecture** | **High** | 6 | React + react-window + Web Workers |
| 8 | **Security-First Frontend** (CSP, Trusted Types) | **Medium** | 5 | React/Next.js + headers |
| 9 | **Server-Driven UI (SDUI)** | **Medium** | 7 | React Native / Next.js + GraphQL |
| 10 | **Resumability & Zero-Hydration** (Qwik-style) | **Low-Medium** | 9 | Qwik / Astro Islands |

---

## Часть 3. Матрица покрытия: проект × frontend-тема

| Frontend Тема | W1 Social Feed | W2 E-com + Search | W3 Chat | W4 Collab Editor | W5 Video | W6 Dashboard | W7 Calendar | W8 File Manager | W9 Maps | W10 Notifications |
|---|---|---|---|---|---|---|---|---|---|---|
| **Core Web Vitals** | ★★★ | ★★★ | ★★ | ★★ | ★★★ | ★★★ | ★★ | ★★ | ★★★ | ★★ |
| **Rendering Strategies** | ★★★ | ★★★ | ★ | ★ | ★★★ | ★★ | ★ | ★ | ★★ | ★ |
| **State Management** | ★★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ |
| **Micro-frontends** | — | — | ★★★ | — | — | ★★ | — | ★★ | ★★ | ★★★ |
| **Design Systems** | ★★ | ★★★ | ★★ | ★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ |
| **Real-time** | ★★★ | ★ | ★★★ | ★★★ | ★ | ★★★ | ★ | ★ | ★★ | ★★★ |
| **Performance** | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ |
| **Offline-first / PWA** | ★ | ★ | ★★★ | ★★★ | ★★ | ★ | ★★ | ★★★ | ★★★ | ★ |
| **Security** | ★ | ★★★ | ★★ | ★★★ | ★★★ | ★ | ★ | ★★★ | ★ | ★ |
| **Accessibility** | ★ | ★★★ | ★★ | ★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ | ★★★ |
| **Zero-downtime deploys** | — | ★★ | — | ★★★ | — | ★★ | ★ | ★★ | ★ | ★★ |
| **Frontend monitoring** | ★ | ★★ | ★ | ★ | ★★★ | ★★★ | ★ | ★ | ★★ | ★★★ |
| **Resumability** | — | ★ | — | — | ★★ | — | — | — | ★★★ | — |
| **Image/Media optimization** | ★★★ | ★★★ | ★★ | ★ | ★★★ | ★ | ★ | ★ | ★ | ★ |
| **API Design** | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ |

---

## Часть 4. Матрица архитектурных подходов по проектам

| Архитектурный подход | W1 | W2 | W3 | W4 | W5 | W6 | W7 | W8 | W9 | W10 |
|---|---|---|---|---|---|---|---|---|---|---|
| **#1 Multi-Layered Rendering** | ★★★ | ★★★ | ★ | ★ | ★★★ | ★★ | ★ | ★ | ★★ | ★ |
| **#2 State Management** | ★★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ |
| **#3 Micro-Frontends** | — | — | ★★★ | — | — | ★★ | — | ★★ | ★★ | ★★★ |
| **#4 Real-time / Event-Driven** | ★★★ | ★ | ★★★ | ★★★ | ★ | ★★★ | ★ | ★ | ★★ | ★★★ |
| **#5 Offline-First / PWA** | ★ | ★ | ★★★ | ★★★ | ★★ | ★ | ★★ | ★★★ | ★★★ | ★ |
| **#6 Design System** | ★★ | ★★★ | ★★ | ★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ |
| **#7 Performance-First** | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★ |
| **#8 Security-First** | ★ | ★★★ | ★★ | ★★★ | ★★★ | ★ | ★ | ★★★ | ★ | ★ |
| **#9 Server-Driven UI** | ★ | ★★ | — | — | ★★ | — | — | — | — | ★★ |
| **#10 Resumability** | — | ★ | — | — | ★★ | — | — | — | ★★★ | — |

---

## Часть 5. Детальные карточки проектов (с архитектурными подходами)

---

### Неделя 1: Social Feed with Media (Instagram-like)
**Frontend-фокус:** Infinite scroll, virtualization, optimistic UI, **SSR для SEO**, **Core Web Vitals**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Virtualized list (10K+ items) | react-window + dynamic heights + intersection observer | Performance |
| 2 | Image pipeline | Blur-hash → WebP/AVIF progressive → full (srcset: 200w, 800w, 1200w) | Image/Media optimization |
| 3 | Optimistic like/comment | useOptimistic hook → local state → server sync → rollback на ошибку | State Management |
| 4 | Real-time new posts | SSE / WebSocket: prepend animation + unread counter + badge | Real-time |
| 5 | Stories bar | Auto-advance 5s timer, preload next story, tap navigation, progress indicator | Performance |
| 6 | **SSR для профилей** | **Next.js SSR/SSG для страниц профилей (/user/[id]) — SEO + fast LCP** | **Rendering Strategies** ★NEW |
| 7 | **ISR для ленты** | **ISR (revalidate 60s) для popular feeds — stale-while-revalidate** | **Rendering Strategies** ★NEW |
| 8 | **LCP optimization** | **Preload LCP image, critical CSS inline, font-display: swap, eliminate render-blocking JS** | **Core Web Vitals** ★NEW |
| 9 | **INP для взаимодействий** | **scheduler.yield() для long tasks, useTransition для некритичных обновлений** | **Core Web Vitals** ★NEW |

#### Архитектурные подходы в этом проекте

**#1 Multi-Layered Rendering (SSR/SSG/ISR)** — профили пользователей (/user/[id]) используют **Next.js SSR** для SEO и быстрого первого рендера. Публичная лента применяет **ISR с revalidate 60s** — статическая генерация с инкрементальным обновлением даёт CDN-скорость при сохранении свежести контента. Личная лента (авторизованная) — **SSR с Streaming** через Suspense-границы: shell загружается мгновенно, а feed-данные стримятся по мере готовности. Trade-off: SSR требует Node.js-сервера (стоимость), но даёт LCP < 1.5s против 3-4s у чистого CSR.

**#2 State Management Architecture (optimistic UI)** — лайки и комментарии реализованы через **TanStack Query с optimistic updates**: UI обновляется мгновенно (useOptimistic hook), фоном идёт POST-запрос, при ошибке — автоматический rollback. Feed-данные — **server state** через TanStack Query (cache, stale-while-revalidate, background refetch). Stories — **client state** через Zustand (таймеры, preload очередь). Это разделение даёт мгновенную обратную связь при взаимодействиях без потери консистентности данных.

**#4 Real-time & Event-Driven (SSE)** — новые посты доставляются через **Server-Sent Events** (SSE): сервер держит HTTP-соединение открытым и пушит события (newPost, likeUpdate). SSE выбран вместо WebSocket, потому что поток однонаправленный (server→client) и проще масштабируется — не требует sticky sessions. Unread counter — **derived state**, вычисляемый из messages + readReceipts через селекторы, а не хранимый отдельно.

**#7 Performance-First Architecture (virtualization)** — лента 10K+ постов рендерится через **react-window с динамическими высотами** — только viewport + overscan (~15 items) находятся в DOM. Это даёт 60fps scroll при любом объёме данных. Image pipeline: blur-hash placeholder → WebP progressive → полное изображение с srcset (200w, 800w, 1200w). Intersection Observer запускает загрузку за 200px до входа в viewport.

#### Backend
| Задача | Технологии |
|--------|-----------|
| Feed generation | Fan-out on write (push) vs fan-out on read (pull) |
| Hot shard | Consistent hashing по userId |
| Cache layers | Redis hot feeds (TTL 5min) + CDN media + Edge static cache |
| Message queue | Kafka: new post → propagate to followers' feed chunks |
| Image processing | Async Lambda pipeline: оригинал → thumbnails 200/800/full WebP/AVIF |
| Storage | S3 originals + CloudFront CDN, PostgreSQL для metadata |
| API | GraphQL: один query для feed + author + counts + like status |
| Rate limit | Token bucket per-user для создания постов |

**Ключевой trade-off:** SSR (SEO, fast LCP, но server cost) vs CSR (интерактивность, но медленный first paint). Где провести границу?

---

### Неделя 2: E-commerce with Smart Search (Amazon-like)
**Frontend-фокус:** Autocomplete, product page, checkout, **Core Web Vitals**, **Design System**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Autocomplete | Debounce 150ms + AbortController + LRU cache + keyboard nav (↓/↑/Enter/Esc) | Performance |
| 2 | Product page | SSR/SSG для SEO, critical CSS inline, image lazy loading, skeleton screens | Rendering Strategies |
| 3 | Cart | Optimistic add-to-cart, offline cart (localStorage), merge on login, micro-animation | State Management |
| 4 | Checkout wizard | State machine (idle → filling → validating → submitting → success/error) | State Management |
| 5 | Flash sale UI | Server-synced countdown, queue position, sold-out transition, disabled кнопка | Real-time |
| 6 | **Design System** | **Shared component library (Button, Card, Input, Modal) в monorepo (Turborepo), Storybook для документации** | **Design Systems** ★NEW |
| 7 | **Compound components** | **`<ProductCard image title price>` — composition pattern, flexible API** | **Design Systems** ★NEW |
| 8 | **LCP для product page** | **Preload hero image, responsive images (srcset + sizes), modern formats (WebP/AVIF)** | **Core Web Vitals** ★NEW |
| 9 | **INP для поиска** | **Бюджет 200ms на interaction: debounce + cancel + fast render результатов** | **Core Web Vitals** ★NEW |
| 10 | **CLS prevention** | **Фиксированные размеры для images/skeletons, font-display: optional, резерв для ads** | **Core Web Vitals** ★NEW |
| 11 | **Bundle splitting** | **Route-based code splitting (React.lazy), vendor chunks, critical vs deferred CSS** | **Performance** ★NEW |

#### Архитектурные подходы в этом проекте

**#1 Multi-Layered Rendering (PPR)** — страница товара использует **Partial Prerendering (Next.js 15)**: статическая оболочка (описание, изображения, отзывы — SSG) + динамические "отверстия" (наличие на складе, актуальная цена — SSR через Suspense). Это даёт LCP < 1.5s для основного контента и свежие данные для динамических элементов. Каталог — **ISR с revalidate 1h** для SEO и скорости. Чек-аут — **CSR**, так как нет SEO-требований и нужна максимальная интерактивность.

**#2 State Management Architecture (cart + checkout state machine)** — корзина — **hybrid state**: TanStack Query для серверной синхронизации (merge on login, multi-device), Zustand для локального UI-state (открыта/закрыта, анимации). Checkout — **state machine** через XState или useReducer: `idle → filling → validating → submitting → success/error`. Каждое состояние имеет чёткие допустимые переходы, что предотвращает неправильные действия (например, double-submit).

**#6 Component-Driven Design System (monorepo + Storybook)** — вся UI-библиотека живёт в **Turborepo monorepo**: `@shop/ui` (Button, Card, Input, Modal), `@shop/icons`, `@shop/utils`. Компоненты построены как **compound components**: `<ProductCard.Image>`, `<ProductCard.Title>`, `<ProductCard.Price>` — гибкий API без prop drilling. **Storybook** служит документацией, визуальным тестированием и playground для дизайнеров. Design tokens — CSS variables (color-primary, spacing-md), поддерживают light/dark/high-contrast темы.

**#7 Performance-First Architecture (code splitting + bundle budgets)** — route-based code splitting через React.lazy + Suspense: каждый маршрут (product, cart, checkout) — отдельный chunk. Vendor chunks (React, TanStack Query) — кешируются между деплоями. **Bundle budget в CI**: initial JS < 200KB, LCP < 2.5s, INP < 200ms — проверка через Lighthouse CI на каждый PR.

**#8 Security-First Frontend (checkout protection)** — checkout flow защищён **CSP с nonce-based scripts**: только явно разрешённые inline-скрипты выполняются. **Trusted Types** блокируют DOM XSS в полях ввода (адрес, имя). **SameSite=Lax** cookies + **Fetch Metadata headers** (Sec-Fetch-Site) защищают от CSRF на API-эндпоинтах оплаты.

**#9 Server-Driven UI (personalized layouts)** — homepage layout персонализируется сервером на основе ML/user behavior: "часто покупают вместе", "рекомендации для вас", "персональные скидки". Сервер отдаёт JSON с описанием секций (`{ type: "ProductCarousel", props: {...} }`), клиент рендерит через component registry. Новые layout-варианты деплоятся на сервере без обновления клиентского приложения — идеально для A/B testing.

#### Backend
| Задача | Технологии |
|--------|-----------|
| Search index | Elasticsearch: fuzzy matching, typo tolerance, facets |
| Autocomplete API | In-memory Trie для hot prefixes + Redis cache топ-100 + CDN edge cache |
| Inventory | Redis DECR (atomic stock check) → async commit в PostgreSQL |
| Distributed transactions | Saga pattern: order → payment → inventory → shipping |
| Flash sale | Kafka request queue + token bucket rate limit + bot detection |
| Caching | Redis product details (TTL 1h), CDN images, cache invalidation |
| API | REST для catalog, GraphQL для product page |
| Rate limit | Sliding window per-IP для поиска, per-user для checkout |

**Ключевой trade-off:** Большой rich design system (consistency, speed of development) vs lean components (меньше bundle, проще поддержка). Когда что?

---

### Неделя 3: Real-time Chat (WhatsApp / Slack-like)
**Frontend-фокус:** Virtual scroll, optimistic send, **Micro-frontends**, **State normalization**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Message list | Virtual scroll (react-virtuoso) с динамическими высотами, scroll anchoring | Performance |
| 2 | Optimistic send | Message мгновенно → sending → sent → delivered → read | State Management |
| 3 | Media handling | Client-side image compression (canvas), chunked upload, inline preview | Image/Media optimization |
| 4 | Presence / typing | Typing indicator (throttle 3s), online/offline/last seen, heartbeat | Real-time |
| 5 | Notifications | In-app toast stack (auto-dismiss, priority), unread badge, browser push | Real-time |
| 6 | **Micro-frontend chat widget** | **Chat как federated module: встраивается в любое приложение (CRM, Support, E-com) через Module Federation 2.0** | **Micro-frontends** ★NEW |
| 7 | **State normalization** | **Нормализованный store: `{ messages: { id: {...} }, users: { id: {...} } }` — не nested JSON** | **State Management** ★NEW |
| 8 | **Derived state** | **Unread count = derived from messages + readReceipts (селекторы, не дублирование)** | **State Management** ★NEW |
| 9 | **Server state vs client state** | **TanStack Query для server state (cache, refetch, stale-while-revalidate), Zustand для UI state** | **State Management** ★NEW |
| 10 | **Keyboard accessibility** | **↑/↓ для навигации по сообщениям, Enter для отправки, Cmd+K для поиска, ARIA live regions** | **Accessibility** ★NEW |

#### Архитектурные подходы в этом проекте

**#2 State Management Architecture (normalization + server/client split)** — чат требует **строгой нормализации**: `{ messages: { byId: {} }, users: { byId: {} }, conversations: { byId: {} } }` вместо вложенного JSON. Это даёт O(1) доступ к любому сообщению, избегает дублирования (user данные хранятся один раз), позволяет эффективные обновления. **TanStack Query** управляет server state (fetch messages, send message, mark as read) с кешированием по conversationId. **Zustand** — для client-only state (typing indicator, selected conversation, UI theme). Unread count — **derived state** через селектор: `unread = messages.filter(m => m.timestamp > readReceipt[convId]).length`, не хранится отдельно.

**#3 Micro-Frontends & Module Federation 2.0 (chat widget)** — чат реализован как **federated module** через Module Federation 2.0: независимо деплоимый widget, который встраивается в shell-приложения (CRM, Support Portal, E-commerce). **Контракты**: shared types (npm package `@acme/chat-types`), event schemas (shell ↔ chat communication), routing conventions. Module Federation 2.0 даёт **type safety** между shell и chat через `.d.ts` файлы, **runtime plugins** для загрузки, и **manifest-протокол** вместо hardcoded URLs. Shared dependencies (React, ReactDOM) — singleton с version negotiation.

**#4 Real-time & Event-Driven Architecture (WebSocket)** — двунаправленная коммуникация через **WebSocket (Socket.io)**: message send/receive, typing events, delivery receipts (sent → delivered → read), presence (online/offline). Сервер масштабируется через **sticky sessions** (userId → server) + **Redis Pub/Sub** для cross-server broadcast. Optimistic send: сообщение появляется в UI мгновенно ("sending"), статус обновляется по WebSocket-confirmation.

**#5 Offline-First Architecture (sync queue)** — сообщения, отправленные offline, попадают в **outbound queue** (IndexedDB): `{ operation: 'sendMessage', data: {...}, timestamp, status: 'pending' }`. Background Sync API автоматически отправляет queue при восстановлении соединения. Conflict resolution: **Last-Write-Wins** по timestamp для простых случаев, полная CRDT-синхронизация — stretch goal.

**#7 Performance-First Architecture (virtualization)** — список сообщений рендерится через **react-virtuoso с динамическими высотами** (текстовые, медиа-сообщения, стикеры — разные высоты). Scroll anchoring: при получении нового сообщения scroll position сохраняется относительно viewport. Media: client-side compression через canvas (resize до 1080px max), WebP конвертация перед upload.

#### Backend
| Задача | Технологии |
|--------|-----------|
| Delivery guarantee | At-least-once delivery, idempotent send (dedup по UUID) |
| Message storage | Cassandra / ScyllaDB — write-heavy, time-series по conversationId |
| WebSocket cluster | Sticky sessions + Redis pub/sub для cross-server broadcast |
| Message queue | Kafka: reliable ingestion, consumer groups |
| File upload | Chunked resumable upload, S3 multipart, virus scan async |
| Push notifications | FCM (Android) / APNs (iOS) для offline users |
| Rate limit | Per-conversation message rate, per-user connection limit |

**Ключевой trade-off:** Micro-frontend (независимость команд, но latency загрузки) vs monolith (быстрее, но coupling). Как шарить state между shell и chat widget?

---

### Неделя 4: Collaborative Editor (Google Docs-like)
**Frontend-фокус:** CRDT, offline-first, cursor presence, **Security**, **Zero-downtime deployments**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Real-time collaboration | CRDT (Yjs) vs OT — trade-offs, eventual consistency | Real-time |
| 2 | Offline-first | Service Worker + IndexedDB для local ops, sync queue | Offline-first / PWA |
| 3 | Cursor presence | Ephemeral awareness (y-protocols), throttle 50ms, cursor color per user | Real-time |
| 4 | Conflict UI | «User X edited this section» banner, diff view, manual merge | State Management |
| 5 | Undo/redo | Per-user undo stack, global vs local undo | State Management |
| 6 | **XSS prevention** | **Trusted Types API — санитизация user-generated content, никакого innerHTML** | **Security** ★NEW |
| 7 | **CSP policy** | **Strict CSP: default-src 'self', script-src с nonce, style-src 'self' + hashes** | **Security** ★NEW |
| 8 | **Zero-downtime deployments** | **Feature flags для новых editing features, gradual rollout (1% → 10% → 100%)** | **Zero-downtime deploys** ★NEW |
| 9 | **Immutable builds** | **Content-hash в filenames (main.abc123.js), atomic deploys, rollback capability** | **Zero-downtime deploys** ★NEW |
| 10 | **State separation** | **Server state (document content) — Yjs sync. Client state (UI panels) — local store. Никогда не mix!** | **State Management** ★NEW |

#### Архитектурные подходы в этом проекте

**#4 Real-time & Event-Driven Architecture (CRDT)** — документ синхронизируется через **Yjs CRDT** (Conflict-free Replicated Data Types): каждая операция (вставка, удаление, форматирование) — это CRDT-операция, которая может примениться в любом порядке без конфликтов. **y-protocols** обеспечивает presence awareness: позиция курсора, выделение, online status каждого пользователя — throttle 50ms для cursor updates. Транспорт: **WebSocket** для sync + **WebRTC** (y-webrtc) как fallback для p2p sync в том же офисе. CRDT выбран вместо OT, потому что не требует центрального ordering-сервера и поддерживает offline editing natively.

**#5 Offline-First & PWA Architecture** — полноценное offline editing: **Service Worker** перехватывает все операции, **IndexedDB** (via y-indexeddb provider) хранит локальную копию документа + очередь операций. При восстановлении соединения Yjs автоматически мержит offline-операции с серверной версией через CRDT-merge — никаких конфликтов. Background Sync для гарантированной доставки очереди даже при закрытии таба.

**#2 State Management Architecture (strict separation)** — **жёсткое разделение**: server state (содержимое документа) — только через Yjs sync, client state (панели инструментов, выбранный текст, UI-режимы) — только через локальный store (Zustand). Никакого смешивания: компоненты редактора не знают про Zustand, UI-панели не знают про Yjs. Это предотвращает race conditions и делает код предсказуемым.

**#7 Performance-First Architecture (memory + CPU)** — CRDT-документы растут из-за tombstones (soft delete). Решение: **V2 encoder** (compact binary format) + периодическая компактация (snapshot каждые 1000 операций). Per-user undo stack ограничен 100 операциями (LRU eviction). Editor rendering: contenteditable → virtual DOM diff минимизирует реальные DOM-операции.

**#8 Security-First Frontend (Trusted Types + CSP)** — редактор содержит **user-generated content** (текст, ссылки, вставки из интернета) — главная поверхность атаки XSS. **Trusted Types API** блокирует все DOM XSS sinks: `element.innerHTML = string` выбрасывает TypeError. Политика `dompurify` санитизирует вставки: `trustedTypes.createPolicy('dompurify', { createHTML: (s) => DOMPurify.sanitize(s) })`. **Strict CSP**: `default-src 'self'`, `script-src 'nonce-{nonce}'`, `require-trusted-types-for 'script'`.

#### Backend
| Задача | Технологии |
|--------|-----------|
| WebSocket scaling | Sticky sessions по documentId, horizontal scaling с Redis pub/sub |
| Message ordering | Kafka partitions по documentId — гарантия ordering |
| Persistence | Event Sourcing: append-only log операций, snapshot каждые 100 ops |
| CAP trade-off | Availability при partition (редактируй локально) + eventual sync |
| Auth / Access | JWT + document-level ACL (owner/editor/viewer), share links с expiry |
| Storage | PostgreSQL для metadata + S3 для snapshot-ов + Kafka для operation log |

**Ключевой trade-off:** Trusted Types (zero XSS, но сложная миграция) vs традиционная санитизация (проще, но bypass-ы возможны). Как внедрить Trusted Types в legacy редактор?

---

### Неделя 5: Video Streaming (Netflix-like)
**Frontend-фокус:** Adaptive bitrate, startup latency, **Frontend monitoring**, **Performance budgets**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Adaptive bitrate | HLS/DASH, bandwidth estimation, seamless quality switch | Performance |
| 2 | Startup latency | Preload first segment, DNS prefetch CDN, poster thumbnail | Performance |
| 3 | Player UI | Custom controls, keyboard shortcuts, PiP, AirPlay, playback speed | Accessibility |
| 4 | Watch history | Beacon API для progress tracking (5s interval), resume playback | Performance |
| 5 | Offline viewing | Background fetch для скачивания, encrypted local storage, DRM license | Offline-first / PWA |
| 6 | **Frontend monitoring (RUM)** | **web-vitals library для LCP/INP/CLS, custom metrics (time-to-first-frame, buffer-ratio), Datadog RUM** | **Frontend monitoring** ★NEW |
| 7 | **Performance budgets** | **Budget: <200KB initial JS, <1.5MB total, LCP <2.5s, INP <200ms — CI проверка через Lighthouse** | **Frontend monitoring** ★NEW |
| 8 | **Bundle analysis** | **webpack-bundle-analyzer, tree shaking, dead code elimination, vendor splitting** | **Performance** ★NEW |
| 9 | **Streaming SSR для metadata** | **RSC (React Server Components) для страницы title/episodes — zero client JS для static части** | **Rendering Strategies** ★NEW |
| 10 | **Error tracking** | **Sentry для JS exceptions, source maps, component-level error boundaries** | **Frontend monitoring** ★NEW |

#### Архитектурные подходы в этом проекте

**#1 Multi-Layered Rendering (RSC)** — страница видео (/watch/[id]) использует **React Server Components** для метаданных (title, description, список эпизодов, cast, recommendations) — **zero client JS** для этой части. Player UI — Client Component (`'use client'`) с интерактивностью. Это даёт LCP < 1.5s для текста и постера при минимальном клиентском бандле. Streaming SSR: метаданные стримятся мгновенно, player инициализируется параллельно.

**#7 Performance-First Architecture (adaptive bitrate + bundle budgets)** — **HLS/DASH** для adaptive streaming: bandwidth estimation через `navigator.connection` + buffer health monitoring, seamless quality switch без rebuffering. **Performance budgets в CI**: initial JS < 200KB (через aggressive code splitting — player в отдельном chunk, загружается по требованию), total < 1.5MB, LCP < 2.5s. Custom metrics: time-to-first-frame (TTFF), buffer-ratio, quality switches per minute. **Bundle analysis**: webpack-bundle-analyzer на каждый PR, tree shaking, vendor splitting (React, video library — отдельные кешируемые chunks).

**#5 Offline-First (download manager)** — **Background Fetch API** для скачивания видео: пользователь выбирает quality (480p/720p/1080p), download идёт в фоне даже при закрытии таба. **Encrypted IndexedDB** для хранения: Content Decryption Module (CDM) + локальные DRM-лицензии. Offline player: тот же HLS player, но с локальными segments вместо сетевых.

**#10 Resumability & Zero-Hydration (Islands)** — страница видео — идеальный кейс для **Islands Architecture**: 80% контента статично (описание, cast, recommendations), только player интерактивен. В Next.js это реализуется через **Server Components** (статика, zero JS) + **Client Component** (player, интерактивность). Stretch goal: эксперимент с Qwik или Astro для полной resumability — сериализовать player state в HTML и resume без hydration replay.

#### Backend
| Задача | Технологии |
|--------|-----------|
| Transcoding | FFmpeg pipeline: raw → multiple resolutions (240p to 4K), async job queue |
| Multi-CDN | Akamai + CloudFront, origin shield, geo-routing |
| Storage | S3 для raw + transcoded segments, lifecycle policies |
| DRM | Widevine / FairPlay / PlayReady, signed URLs с expiry |
| API | GraphQL для metadata, REST для signed stream URLs |
| Load balancing | GeoDNS + anycast для низкой latency |

**Ключевой trade-off:** RSC (меньше JS, но сложность инфраструктуры) vs традиционный SSR (проще, но больше hydration cost). Как measure impact на реальных устройствах?

---

### Неделя 6: Analytics Dashboard (10K+ data points)
**Frontend-фокус:** Charts, real-time updates, **Web Workers**, **Frontend monitoring as product**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Chart rendering | Web Workers для data processing, Canvas 2D / WebGL (deck.gl), LTTB downsampling | Performance |
| 2 | Real-time updates | SSE для streaming metrics, throttled re-render (requestAnimationFrame) | Real-time |
| 3 | Filters panel | Multi-select с debounce, URL-sync для shareable state | State Management |
| 4 | Export | Client-side CSV generation, streaming XLSX download | Performance |
| 5 | Large tables | Virtual scroll, column resize, frozen columns, cursor pagination | Performance |
| 6 | **Web Workers архитектура** | **Worker thread для data aggregation (filter, sort, compute) — main thread только для render** | **Performance** ★NEW |
| 7 | **Worker → Main thread communication** | **Transferable Objects для zero-copy передачи large arrays, structured clone для metadata** | **Performance** ★NEW |
| 8 | **Monitoring the monitor** | **Self-monitoring: dashboard измеряет собственный INP, memory usage, render time — dogfooding** | **Frontend monitoring** ★NEW |
| 9 | **CLS для charts** | **Фиксированные контейнеры для chart-ов (aspect-ratio), skeleton matching real dimensions** | **Core Web Vitals** ★NEW |
| 10 | **Accessibility для data viz** | **ARIA labels для chart elements, keyboard navigation по data points, screen reader summaries** | **Accessibility** ★NEW |

#### Архитектурные подходы в этом проекте

**#7 Performance-First Architecture (Web Workers + virtualization)** — **Web Workers** offloading: основной поток (main thread) отвечает только за render, вся data processing (aggregation, filter, sort, compute 10K+ точек) — в **dedicated Worker**. Communication через **Transferable Objects** (ArrayBuffer, ImageBitmap) — zero-copy передача large arrays без structured clone overhead. **LTTB downsampling** (Largest Triangle Three Buckets) в Worker: сокращение 100K точек до 1K для визуализации без потери формы графика. Таблицы: **virtual scroll + frozen columns + cursor pagination** для бесконечных списков.

**#2 State Management Architecture (URL-sync + filters)** — dashboard state разделено: **TanStack Query** для server state (метрики из ClickHouse, агрегаты), **Zustand** для client state (выбранные фильтры, UI layout). Ключевой паттерн: **URL-sync** — все фильтры (date range, dimensions, metrics) синхронизируются с URL (`?from=2024-01&to=2024-03&metrics=sessions,revenue`), что делает любое состояние shareable (copy-paste URL → тот же dashboard).

**#4 Real-time & Event-Driven Architecture (SSE)** — real-time метрики через **Server-Sent Events**: Kafka → Flink / Spark Streaming → SSE endpoint → клиент. **Throttled re-render**: данные буферизуются 1 секунду, рендер происходит через requestAnimationFrame — не чаще 60fps. Это предотвращает jank при burst-обновлениях (например, Black Friday traffic spike).

**#3 Micro-Frontends (dashboard widgets)** — dashboard состоит из **независимых widget-модулей** (traffic chart, revenue table, conversion funnel, geo map), каждый — federated module через Module Federation. Команды владеют своими widget'ами и деплоят независимо. Shell-приложение: layout grid, фильтры (shared как cross-cutting concern), routing.

**#6 Component-Driven Design System** — data viz компоненты: Chart, Table, KPI-card, FilterPanel — reusable, themeable, документированы в Storybook. Каждый chart type (line, bar, pie, heatmap) — compound component с consistent API (`<Chart.Data>`, `<Chart.Axis>`, `<Chart.Tooltip>`).

#### Backend
| Задача | Технологии |
|--------|-----------|
| Time-series DB | ClickHouse / TimescaleDB для метрик, partitioning по time range |
| Aggregation | Pre-aggregated rollups: 1min → 5min → 1h → 1d, materialized views |
| Streaming | Kafka → Flink / Spark Streaming → real-time aggregates в Redis |
| API | GraphQL с columnar selection, cursor pagination |
| Cache | Redis для pre-computed dashboards, TTL по freshness |
| Observability | OpenTelemetry tracing, Prometheus metrics, Grafana dashboards |

**Ключевой trade-off:** Web Workers (не блокируем main thread, но overhead сериализации) vs main thread (проще, но jank при больших данных). Где threshold для offloading?

---

### Неделя 7: Calendar & Booking (Google Calendar-like)
**Frontend-фокус:** Timeline, drag-and-drop, recurrence, **Accessibility**, **Feature flags**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Timeline rendering | Custom time-based layout, virtual scroll по времени, week/month/day views | Performance |
| 2 | Drag-and-drop | @dnd-kit, snap-to-grid 15min, resize handles, touch support | Performance |
| 3 | Recurrence | RRULE (RFC 5545) parser, expand в клиенте, exception instances | State Management |
| 4 | Timezone | Intl.DateTimeFormat, moment-timezone, UTC storage, local display | State Management |
| 5 | Conflict preview | Визуальная подсветка overlap при drag, atomic validation | State Management |
| 6 | **Accessibility (WCAG 2.1)** | **`role="grid"` для calendar, `aria-selected` для events, keyboard: arrows для навигации по дням** | **Accessibility** ★NEW |
| 7 | **Screen reader support** | **`aria-label="Meeting with John, 2:00 PM to 3:00 PM"` для каждого event, live region для notifications** | **Accessibility** ★NEW |
| 8 | **Focus management** | **Focus trap в modal для создания event, restore focus после закрытия, skip link для main content** | **Accessibility** ★NEW |
| 9 | **Feature flags** | **LaunchDarkly / Unleash: новый view (year view) за флагом, gradual rollout, A/B testing** | **Zero-downtime deploys** ★NEW |
| 10 | **Reduced motion** | **`prefers-reduced-motion: reduce` — отключаем анимации drag, instant snap** | **Accessibility** ★NEW |

#### Архитектурные подходы в этом проекте

**#2 State Management Architecture (recurrence + timezone)** — календарь требует сложного state management: **RRULE parser** (RFC 5545) для recurrence — правила хранятся компактно (`FREQ=WEEKLY;BYDAY=MO,WE,FR`), клиент "раскрывает" в instances при отображении. Timezone: все события хранятся в UTC, отображаются в локальном часовом поясе через `Intl.DateTimeFormat`. **Zustand** для client state (selected date, view mode, visible range). **TanStack Query** для server state (events, bookings, availability).

**#6 Component-Driven Design System** — calendar UI построен как набор compound components: `<Calendar.Grid>`, `<Calendar.Event>`, `<Calendar.Day>`, `<Calendar.Toolbar>`. Drag-and-drop через @dnd-kit с snap-to-grid 15 минут. Accessibility: `role="grid"` для calendar container, каждый event — `aria-label` с полным описанием ("Meeting with John, 2:00 PM to 3:00 PM"). Focus management: focus trap в modal создания события, restore focus после закрытия.

**#7 Performance-First Architecture (virtual scroll + DnD)** — timeline рендерится через **custom virtual scroll**: видим только viewport (7 дней × 24 часа), остальное — виртуально. Drag-and-drop: @dnd-kit с оптимизациями (throttled collision detection 16ms, GPU-accelerated transforms). Conflict detection: при drag вычисляем overlap с существующими events в real-time — derived state через селекторы.

**#9 Server-Driven UI (feature flags)** — новые view (year view, timeline view) и experiment'ы (smart scheduling suggestions) за **feature flags** (LaunchDarkly / Unleash). Сервер определяет, какие features доступны пользователю на основе cohort + rollout percentage. Это zero-downtime deployment: новая feature включается для 1% → 10% → 100% пользователей без redeploy frontend.

#### Backend
| Задача | Технологии |
|--------|-----------|
| Double-booking prevention | Optimistic locking (version field) / pessimistic (SELECT FOR UPDATE) |
| Recurrence storage | Store RRULE + exceptions, expand on read |
| Availability queries | Time-slot indexing (GiST в PostgreSQL), exclusion ranges |
| Distributed lock | Redis Redlock для concurrent booking |
| Notifications | Cron + message queue для reminder emails/push |
| API | REST + iCal export, timezone parameter |

**Ключевой trade-off:** Feature flags (гибкость, но tech debt от stale flags) vs direct deployment (проще, но риск). Как управлять lifecycle feature flags?

---

### Неделя 8: File Manager (Dropbox-like)
**Frontend-фокус:** Tree view, chunked upload, **Design System на уровне OS**, **Security**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Tree navigation | Virtualized tree, breadcrumbs, keyboard nav (arrows, expand/collapse) | Performance |
| 2 | Chunked upload | Resumable upload (tus protocol), concurrent chunks, pause/resume | Performance |
| 3 | Sync status | Icon indicators (synced/syncing/error/offline), bandwidth throttle | Offline-first / PWA |
| 4 | Conflict resolution | «Your version vs Server version» side-by-side diff, manual merge | State Management |
| 5 | Offline editing | Download → edit → queue upload → sync on reconnect | Offline-first / PWA |
| 6 | **Design System (OS-level)** | **File manager как часть design system: FileIcon, FolderTree, ContextMenu, Toolbar — compound components** | **Design Systems** ★NEW |
| 7 | **Theming** | **CSS variables для light/dark/high-contrast themes, design tokens (color, spacing, typography)** | **Design Systems** ★NEW |
| 8 | **Content Security Policy** | **Strict CSP: `default-src 'self'`, `connect-src` для API endpoints, `img-src` для previews** | **Security** ★NEW |
| 9 | **CSRF protection** | **Double-submit cookie pattern для upload/delete операций, SameSite=Lax для cookies** | **Security** ★NEW |
| 10 | **File type validation** | **Client-side mime-type check + magic numbers, не доверять extension, sandboxed preview (iframe sandbox)** | **Security** ★NEW |

#### Архитектурные подходы в этом проекте

**#5 Offline-First & PWA Architecture (core feature)** — file manager без offline — не file manager. **Service Worker** кеширует файловую структуру (metadata tree в IndexedDB), **Background Sync** отправляет queued uploads при reconnect. **tus protocol** для resumable upload: каждый файл разбивается на chunks (5MB), каждый chunk upload'ится independently с retry. Offline editing: файл скачивается → редактируется локально → queue upload → sync on reconnect. **Conflict resolution UI**: side-by-side diff ("Your version vs Server version") с manual merge.

**#6 Component-Driven Design System (OS-level)** — file manager UI — это **design system на уровне операционной системы**: FileIcon (200+ mime-type иконок), FolderTree (virtualized, expandable), ContextMenu (right-click actions), Toolbar (bulk operations), Breadcrumbs, PreviewPanel. Все компоненты — **compound components** с consistent API. Theming: CSS variables для light/dark/high-contrast, design tokens для spacing, typography, color. Monorepo: `@acme/file-ui`, `@acme/icons`, `@acme/theme`.

**#8 Security-First Frontend (file upload protection)** — file manager — главная поверхность атаки (upload malicious files, XSS через preview). **CSP**: `default-src 'self'`, `connect-src` ограничен API endpoints, `img-src` для previews. **File validation**: client-side mime-type check + magic numbers (не доверяем extension!), sandboxed preview через `<iframe sandbox>`. **CSRF protection**: double-submit cookie для upload/delete, SameSite=Lax. **Trusted Types** для preview panel (санитизация HTML previews).

**#7 Performance-First Architecture (virtual tree + chunked upload)** — дерево файлов виртуализировано (react-window для tree): рендерим только видимые узлы + 2 levels вглубь. Chunked upload через tus: concurrent chunks (3-5 parallel), pause/resume, progress tracking. Bandwidth throttle: автоматическое снижение concurrency при медленном соединении.

**#3 Micro-Frontends (file manager widget)** — file manager как **federated module** для встраивания в другие приложения: CRM (attach files to deals), Support (attach to tickets), Chat (share files). Module Federation 2.0 обеспечивает type safety и runtime loading.

#### Backend
| Задача | Технологии |
|--------|-----------|
| Content-addressable storage | Hash (SHA-256) = filename, automatic deduplication |
| Delta sync | Rsync algorithm — передаём только diff |
| Metadata | PostgreSQL: nested set model / LTREE для folder tree |
| Versioning | Append-only storage, retain N версий, cleanup policy |
| Share / Auth | JWT + ACL per file/folder, public share links с expiry |
| Large files | Multipart S3 upload, presigned URLs, parallel chunk upload |

**Ключевой trade-off:** Strict CSP (безопасность, но ограничения на inline scripts/styles) vs permissive (гибкость, но XSS риск). Как handle third-party widgets?

---

### Неделя 9: Maps Service (Google Maps-like)
**Frontend-фокус:** Tile rendering, gestures, clustering, **Resumability**, **Bundle optimization**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Tile rendering | Mapbox GL JS (WebGL), viewport-based tile fetching, zoom levels 0-20 | Performance |
| 2 | Gestures | Pan, pinch-zoom, rotate, momentum, double-tap-to-zoom | Performance |
| 3 | Marker clustering | Supercluster algorithm, dynamic по zoom, spiderify при клике | Performance |
| 4 | Geocoding search | Autocomplete адресов, reverse geocoding, place details | API Design |
| 5 | Offline maps | Service Worker cache vector tiles, MBTiles offline packs | Offline-first / PWA |
| 6 | **Resumability (Qwik-style)** | **Статические элементы UI (header, search bar) — resumable, zero hydration. Только map interactive.** | **Resumability** ★NEW |
| 7 | **Hydration vs Resumability trade-off** | **Next.js hydration для map container, но Qwik-style для остального — гибридный подход** | **Resumability** ★NEW |
| 8 | **Lazy load map library** | **Mapbox GL JS — dynamic import только когда user кликает "Open map" или scroll в viewport** | **Performance** ★NEW |
| 9 | **Bundle budget enforcement** | **Map library в отдельном chunk (~500KB), не в critical bundle. Lighthouse CI gate.** | **Performance** ★NEW |
| 10 | **Accessibility для карты** | **ARIA labels для markers, keyboard shortcuts (+/- для zoom, arrows для pan), альтернативный list view** | **Accessibility** ★NEW |

#### Архитектурные подходы в этом проекте

**#10 Resumability & Zero-Hydration (Islands/Qwik)** — maps — идеальный кейс для resumability: **90% UI статично** (header, search bar, filters, sidebar), только карта интерактивна. В Next.js: Server Components для всего static UI (zero client JS), Client Component только для map container. Stretch goal: эксперимент с **Qwik** — полная resumability: сериализовать состояние карты (zoom, center, selected marker) в HTML, resume без hydration replay. Это даёт **instant interactivity** даже на low-end устройствах.

**#7 Performance-First Architecture (lazy load + clustering)** — **Mapbox GL JS** (~500KB) — **dynamic import** только при взаимодействии: либо по клику "Open map", либо при scroll в viewport через Intersection Observer. Не попадает в critical bundle. **Supercluster algorithm** для marker clustering: точки группируются динамически по zoom level (алгоритм на основе KD-tree). При клике на cluster — "spiderify" (разлет маркеров вокруг точки). **Bundle budget**: map library в отдельном chunk, Lighthouse CI gate на каждый PR.

**#5 Offline-First (vector tiles cache)** — **Service Worker** кеширует vector tiles (MVT format) для просмотренных регионов. **MBTiles offline packs**: пользователь может скачать регион (город/область) для offline navigation. IndexedDB для хранения tiles с LRU eviction (ограничение 500MB).

**#2 State Management Architecture (URL-sync map state)** — состояние карты (zoom, center, selected marker, filters) **синхронизировано с URL** (`?z=14&lat=55.75&lng=37.62&filter=restaurants`) — делает любое состояние shareable. Zustand для client state, TanStack Query для POI data (places, reviews).

#### Backend
| Задача | Технологии |
|--------|-----------|
| Geospatial index | PostGIS: GiST index, geo_distance queries, bounding box search |
| Tile server | Tippecanoe → pre-generated vector tiles (MVT), CDN cache |
| Routing engine | OSRM / Valhalla: contraction hierarchies, real-time traffic overlay |
| Geocoding | Nominatim / Pelias: address parsing, fuzzy matching |
| Real-time | WebSocket / SSE для live location sharing, traffic updates |
| Storage | PostgreSQL + PostGIS для POI, S3 для tile-ов, Redis для routing cache |

**Ключевой trade-off:** Resumability (мгновенный first paint, но новая парадигма) vs hydration (знакомо, но delay до interactive). Где граница для гибридного подхода?

---

### Неделя 10: Cross-Channel Notification System
**Frontend-фокус:** Notification center, preferences, **Micro-frontends**, **State sync**

#### Frontend (с дополнениями)

| # | Задача | Технологии | Frontend-тема |
|---|--------|-----------|---------------|
| 1 | Notification center | Slide-out panel, infinite scroll, mark-all-read, фильтры | Performance |
| 2 | Preference management | Per-channel toggles (push/email/sms), per-category settings | State Management |
| 3 | Real-time badge | SSE для unread count, BroadcastChannel API для sync между табами | Real-time |
| 4 | Toast stack | Priority queue (error > warning > info), auto-dismiss, action buttons | Accessibility |
| 5 | Push subscription | Service Worker push event, FCM token management, resubscribe | Real-time |
| 6 | **Micro-frontend notification center** | **Notification panel как federated module: встраивается в любое приложение (dashboard, chat, e-com)** | **Micro-frontends** ★NEW |
| 7 | **Module Federation contracts** | **Shared types, event schemas, routing conventions — defined upfront как npm package** | **Micro-frontends** ★NEW |
| 8 | **Cross-tab state sync** | **BroadcastChannel API для unread count, Service Worker для background sync** | **State Management** ★NEW |
| 9 | **SSR для notification emails** | **RSC для email template preview (server-side render MJML → HTML preview)** | **Rendering Strategies** ★NEW |
| 10 | **Design system для notifications** | **Toast, Badge, BellIcon, PreferencePanel — reusable, themeable, documented в Storybook** | **Design Systems** ★NEW |

#### Архитектурные подходы в этом проекте

**#3 Micro-Frontends & Module Federation 2.0 (notification widget)** — notification center — **federated module** через Module Federation 2.0: независимо деплоимый widget, встраиваемый в любое shell-приложение (dashboard, chat, e-commerce). **Контракты**: `@acme/notifications-contracts` — shared TypeScript types, event schemas (notificationReceived, markAsRead, preferenceChanged), routing conventions. Type safety через `.d.ts` файлы, runtime loading через manifest-протокол. Shared design system: Toast, Badge, BellIcon, PreferencePanel — themeable через CSS variables.

**#4 Real-time & Event-Driven Architecture (SSE + BroadcastChannel)** — **SSE** для real-time delivery: сервер пушит события (newNotification, unreadCountUpdate) по HTTP-соединению. **BroadcastChannel API** для cross-tab sync: unread count синхронизируется между всеми открытыми табами одного origin. **Service Worker** для background processing: push notifications (FCM/APNs), background sync для offline actions.

**#2 State Management Architecture (preferences + cross-tab sync)** — preferences — **нормализованный store**: `{ channels: { push: true, email: false }, categories: { marketing: false, security: true } }`. **BroadcastChannel** для cross-tab sync: изменение preferences в одном табе мгновенно отражается в других. **Zustand** для client state, **TanStack Query** для server state (notifications list, preferences).

**#6 Component-Driven Design System** — notification UI — reusable компоненты: Toast (priority queue, auto-dismiss, action buttons), Badge (count, pulse animation), BellIcon (animated on new notification), PreferencePanel (per-channel toggles). Все компоненты в Storybook с accessibility tests.

**#9 Server-Driven UI (notification templates)** — email и push notification templates управляются сервером: **RSC для email preview** — server-side render MJML template → HTML preview в клиенте. A/B testing templates: сервер выбирает template variant на основе user cohort. Новые templates деплоятся без обновления клиентского приложения.

#### Backend
| Задача | Технологии |
|--------|-----------|
| Event ingestion | Kafka: все сервисы публикуют события |
| Routing engine | Notification service → определяет канал по preferences + priority |
| Rate limiting | Per-user daily limit, per-channel throttle, cooldown periods |
| Retry + DLQ | Exponential backoff, dead letter queue после 5 попыток |
| Template engine | Handlebars / MJML для email, i18n, A/B testing |
| Delivery workers | FCM (push), SendGrid/SES (email), Twilio (SMS) — изолированные workers |
| Deduplication | Idempotency key: один event → одно notification |

**Ключевой trade-off:** Module Federation (runtime integration, независимые деплои) vs build-time integration (проще, но coupled releases). Как version shared design system между микрофронтендами?

---

## Часть 6. Итоговое покрытие: все 15 frontend-тем

| Frontend Тема | W1 | W2 | W3 | W4 | W5 | W6 | W7 | W8 | W9 | W10 | Всего ★★★ |
|--------------|----|----|----|----|----|----|----|----|----|-----|-----------|
| Core Web Vitals | ★★★ | ★★★ | ★★ | ★★ | ★★★ | ★★★ | ★★ | ★★ | ★★★ | ★★ | 4 |
| Rendering Strategies | ★★★ | ★★★ | ★ | ★ | ★★★ | ★★ | ★ | ★ | ★★ | ★ | 2 |
| State Management | ★★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | 8 |
| Micro-frontends | — | — | ★★★ | — | — | ★★ | — | ★★ | ★★ | ★★★ | 2 |
| Design Systems | ★★ | ★★★ | ★★ | ★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ | 6 |
| Real-time | ★★★ | ★ | ★★★ | ★★★ | ★ | ★★★ | ★ | ★ | ★★ | ★★★ | 5 |
| Performance | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | 10 |
| Offline-first / PWA | ★ | ★ | ★★★ | ★★★ | ★★ | ★ | ★★ | ★★★ | ★★★ | ★ | 4 |
| Security | ★ | ★★★ | ★★ | ★★★ | ★★★ | ★ | ★ | ★★★ | ★ | ★ | 3 |
| Accessibility | ★ | ★★★ | ★★ | ★★ | ★★★ | ★★★ | ★★★ | ★★ | ★★★ | ★★ | 5 |
| Zero-downtime deploys | — | ★★ | — | ★★★ | — | ★★ | ★ | ★★ | ★ | ★★ | 1 |
| Frontend monitoring | ★ | ★★ | ★ | ★ | ★★★ | ★★★ | ★ | ★ | ★★ | ★★★ | 3 |
| Resumability | — | ★ | — | — | ★★ | — | — | — | ★★★ | — | 1 |
| Image/Media optimization | ★★★ | ★★★ | ★★ | ★ | ★★★ | ★ | ★ | ★ | ★ | ★ | 3 |
| API Design | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | 10 |

**Все 15 тем покрыты минимум 1 раз, большинство — 3-10 раз в разных контекстах.**

---

## Часть 7. Тематические «глубокие погружения» по грейдам

### Middle (3-5 лет) — фундамент
Сфокусируйтесь на этих темах в проектах:
- **Core Web Vitals** — знать thresholds (2.5s / 200ms / 0.1), уметь объяснить rendering pipeline
- **State Management** — server vs client state, normalization, optimistic UI
- **Performance** — virtualization, code splitting, lazy loading, debounce
- **Accessibility** — keyboard nav, ARIA basics, focus management
- **Image optimization** — WebP/AVIF, srcset, lazy loading, blur-hash

### Senior (6-10 лет) — архитектурные решения
- **Rendering Strategies** — SSR vs CSR vs SSG vs ISR vs RSC vs Islands. Trade-offs для каждого
- **Micro-frontends** — Module Federation, composition patterns, когда оправданы
- **Design Systems** — component library в monorepo, compound components, versioning
- **Security** — CSP, Trusted Types, XSS prevention, CSRF protection
- **Frontend monitoring** — RUM vs lab data, performance budgets, CI gates

### Staff / Principal (10+ лет) — платформенное мышление
- **Zero-downtime deployments** — feature flags, atomic deploys, rollback strategy
- **Resumability** — Qwik-style architecture, hydration cost analysis, hybrid approaches
- **Frontend as platform** — cross-org design system, shared infrastructure, governance
- **Performance at scale** — establish organization-wide performance budgets, CI enforcement
- **Security governance** — CSP rollout strategy, Trusted Types adoption in legacy codebase

---

## Часть 8. Еженедельный формат работы

| День | Активность | Время |
|------|-----------|-------|
| **Пн** | Прочитать ресурсы, изучить теорию | 2-3ч |
| **Вт** | Нарисовать high-level архитектуру (Excalidraw/draw.io) | 2ч |
| **Ср** | Описать API контракты + компонентную архитектуру | 2ч |
| **Чт** | Написать proof-of-concept (ключевые компоненты) | 3-4ч |
| **Пт** | Пройти чек-лист, провести mock interview (вслух) | 2ч |
| **Сб** | Ревью: trade-offs, что забыл, дополнительные вопросы | 2ч |
| **Вс** | Отдых / дочитать ресурсы | — |

---

## Часть 9. Полезные ресурсы по frontend-темам

### Core Web Vitals
- [web.dev Core Web Vitals](https://web.dev/articles/vitals) — официальная документация Google
- [INP Optimization Guide](https://web.dev/articles/optimize-inp) — конкретные техники
- [LCP Optimization](https://web.dev/articles/optimize-lcp) — пошаговое улучшение

### Rendering Strategies
- [Next.js Rendering](https://nextjs.org/docs/app/building-application/rendering) — SSR/SSG/ISR/RSC
- [Astro Islands Architecture](https://docs.astro.build/en/concepts/islands/) — Islands pattern
- [Qwik Resumability](https://qwik.dev/docs/concepts/resumable/) — resumable vs hydration

### Micro-frontends
- [Module Federation 2.0](https://module-federation.io/) — официальная документация
- [Micro-frontends in 2026](https://www.codercops.com/blog/micro-frontends-module-federation-architecture-2026) — обзор паттернов

### Design Systems
- [Compound Components Pattern](https://kentcdodds.com/blog/compound-components-with-react-hooks) — Kent C. Dodds
- [Storybook Documentation](https://storybook.js.org/docs) — component-driven development

### State Management
- [TanStack Query](https://tanstack.com/query/latest) — server state management
- [Zustand](https://docs.pmnd.rs/zustand/getting-started/introduction) — client state
- [State Normalization](https://redux.js.org/usage/structuring-reducers/normalizing-state-shape) — Redux docs (применимо везде)

### Security
- [Trusted Types](https://web.dev/articles/trusted-types) — XSS prevention
- [CSP Guide](https://web.dev/articles/csp) — Content Security Policy
- [OWASP Frontend Security](https://owasp.org/www-project-web-security-testing-guide/) — полный гайд

### Frontend Monitoring
- [web-vitals JS library](https://github.com/GoogleChrome/web-vitals) — RUM measurement
- [Sentry Frontend Monitoring](https://docs.sentry.io/platforms/javascript/) — error tracking + performance
- [Datadog RUM](https://docs.datadoghq.com/real_user_monitoring/) — полный monitoring stack
