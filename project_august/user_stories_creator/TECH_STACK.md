# StoryCraft AI — Технический стек

> Дата выбора: 2026-06-05
> Статус: Утверждено

---

## Обзор

| Слой | Технология | Роль в проекте |
|------|-----------|----------------|
| **Frontend** | React 19 + React Flow v12 + Zustand + Tailwind CSS | Интерактивная доска, drag-and-drop, zoom |
| **Canvas Performance** | Viewport culling + LOD | 500+ карточек без лагов |
| **Backend Runtime** | Bun 1.2+ | Быстрый runtime, native TypeScript |
| **Backend Framework** | NestJS 10+ | Модульная архитектура, DI, WebSocket Gateway |
| **Real-time** | Socket.io (msgpack) + Yjs + IndexedDB | Коллаборация без конфликтов, offline-first |
| **Database** | Supabase PostgreSQL | Данные + Auth + Storage + RLS |
| **Queue / Jobs** | pg-boss | Background jobs на PostgreSQL, без Redis |
| **AI** | OpenRouter + Vercel AI SDK | LLM через единый API, structured outputs |
| **Auth** | Supabase Auth | OAuth, RLS, workspace isolation |
| **File Storage** | Supabase Storage | Загрузка файлов с RLS |
| **Export** | Client-side (html-to-image, jsPDF, papaparse) | PNG/PDF/CSV без нагрузки на сервер |
| **Hosting** | Fly.io (app) + Supabase (data) | Без сна, managed DB |

---

## 1. Frontend

### React 19 + React Flow v12

React Flow — библиотека для построения node-based интерфейсов. Для StoryCraft AI используется только layout engine (drag, zoom, pan, selection) — сами карточки — обычные div'ы.

```
Activity (Backbone) → горизонтальная линия node'ов
  └── Task Cards → div'ы, position absolute под Activity
```

**Почему не tldraw / Excalidraw / raw Canvas:**
- Не нужен inline editing текста внутри карточки (клик → сайдбар)
- Не нужна freehand drawing
- React Flow даёт drag/zoom/minimap/controls из коробки
- Простые div'ы = accessibility (screen reader, keyboard nav) без танцев с canvas

### Zustand

Минималистичный state manager. Один store для canvas state (nodes, edges, viewport). Нет boilerplate Redux, нет реактивных proxy проблем MobX.

```typescript
// stores/canvas.store.ts
interface CanvasState {
  nodes: Node<StoryCardData>[];
  edges: Edge[];
  selectedNodeId: string | null;
  viewport: Viewport;
  // actions
  moveNode: (id: string, position: XYPosition) => void;
  updateNodeData: (id: string, data: Partial<StoryCardData>) => void;
}
```

### Tailwind CSS

Утилитарный CSS — идеален для разработки агентами (нет CSS-файлов, которые агент может поломать).

---

## 2. Canvas Performance Strategy

### Viewport Culling

Рендерим только карточки, попадающие во viewport (+ 20% buffer). React Flow v12 имеет встроенный `onlyRenderVisibleElements`.

### Level-of-Detail (LOD)

| Zoom Level | Отображение |
|-----------|-------------|
| 100%+ | Полная карточка: текст, статус, аватар, цвет персоны |
| 40-99% | Только текст заголовка, цветная полоска персоны |
| < 40% | Цветной прямоугольник (heatmap по плотности задач) |

**500+ div'ов на современном десктопе** — без проблем. bottleneck не в DOM, а в layout thrashing. Решение: `transform` вместо `top/left`, `will-change: transform` на перетаскиваемых элементах.

---

## 3. Backend

### Bun 1.2+

- **3x скорее Node.js** на I/O-bound задачах
- **Native TypeScript** — нет ts-node, нет transpile step в dev
- **Встроенный bundler** — можно собирать CLI/tools тем же runtime
- **Встроенный test runner** — не нужен Jest/Vitest (хотя Vitest тоже ок)
- **SQLite встроен** — для local development без Docker

**Ограничения:** некоторые нативные модули (node-sass, старые bcrypt) могут не собираться. В проекте их нет.

### NestJS 10+

Модульная структура:

```
src/
├── modules/
│   ├── auth/           # Supabase auth integration, guards
│   ├── board/          # CRUD story maps, nodes, edges
│   ├── ai/             # LLM calls, discovery dialog, gap analysis
│   ├── collaboration/  # Socket.io gateway, Yjs document sync
│   ├── export/         # PRD generation, export jobs
│   ├── integration/    # JIRA, Linear, GitHub webhooks
│   ├── file/           # Supabase Storage uploads
│   └── workspace/      # Teams, invites, permissions
├── common/             # Filters, pipes, interceptors, decorators
├── config/             # env validation (zod)
└── main.ts
```

**WebSocket Gateway** для Socket.io:

```typescript
@WebSocketGateway({
  cors: { origin: '*' },
  namespace: 'board',
  transports: ['websocket'],
})
export class BoardGateway implements OnGatewayConnection {
  // room per board: `board:${boardId}`
  // broadcast Yjs updates to room members
}
```

---

## 4. Real-Time Collaboration

### Архитектура: Socket.io + Yjs

```
Client A ──WebSocket──┐
                      ├──→ Yjs Doc (server) ──→ PostgreSQL (persist)
Client B ──WebSocket──┘
```

**Почему Yjs + ручной Socket.io, а не Liveblocks / PartyKit:**
- **Полный контроль**: видим каждый broadcast, можем логировать, rate-limit, авторизовывать
- **Бесплатно**: нет per-user pricing (важно для учебного/стартап проекта)
- **CRDT из коробки**: нет merge conflicts при одновременном редактировании
- **IndexedDB provider**: offline-first — изменения сохраняются локально, синхронизируются при reconnect

### Data Flow

1. Пользователь открывает доску
2. Frontend: Yjs Doc загружается из IndexedDB (если есть) или с сервера
3. Socket.io: клиент подключается к room `board:${boardId}`
4. Каждое изменение → Yjs update → Socket.io broadcast (msgpack для бинарности)
5. Сервер: получает update, применяет к in-memory Yjs Doc, сохраняет snapshot в PostgreSQL каждые 30 сек
6. При отключении: IndexedDB держит локальную копию, sync при reconnect

### Msgpack

Бинарный протокол вместо JSON для WebSocket messages. Yjs updates и так бинарные — msgpack убирает base64 overhead.

---

## 5. Database

### Supabase PostgreSQL

**Почему Supabase, а не self-hosted Postgres:**
- Free tier: 500MB DB + Auth + Storage + Edge Functions
- Row Level Security (RLS) — авторизация на уровне БД
- Realtime API (можно использовать для некоторых фич, хотя основной real-time — Socket.io)
- Автоматические backups
- PostgREST API (опционально, если нужен REST без кода)

### Schema Overview

```sql
-- Workspaces (isolation boundary)
create table workspaces (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  owner_id uuid references auth.users(id),
  created_at timestamptz default now()
);

-- Workspace members (RBAC)
create table workspace_members (
  workspace_id uuid references workspaces(id),
  user_id uuid references auth.users(id),
  role text check (role in ('admin', 'editor', 'viewer')),
  primary key (workspace_id, user_id)
);

-- Story Maps
create table story_maps (
  id uuid primary key default gen_random_uuid(),
  workspace_id uuid references workspaces(id),
  name text not null,
  yjs_state bytea,           -- Yjs document snapshot
  version integer default 1,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- Versions (for history & revert)
create table map_versions (
  id uuid primary key default gen_random_uuid(),
  map_id uuid references story_maps(id),
  yjs_state bytea,
  snapshot_name text,
  created_by uuid references auth.users(id),
  created_at timestamptz default now()
);

-- Jobs (pg-boss)
create table job_queue (
  -- managed by pg-boss
);
```

### RLS Policies

```sql
-- Users see only their workspaces
alter table workspaces enable row level security;
create policy workspace_isolation on workspaces
  for all using (owner_id = auth.uid() or 
    exists (select 1 from workspace_members where workspace_id = id and user_id = auth.uid()));
```

---

## 6. Queue / Background Jobs

### pg-boss

Job queue поверх PostgreSQL. Не нужен отдельный Redis.

**Use cases:**
- AI batch calls (discovery dialog, gap analysis — не должны блокировать HTTP request)
- Export generation (PDF/PNG/CSV)
- JIRA/Linear sync
- Email notifications

```typescript
// ai.module.ts
this.pgBoss.work('ai-discovery', async (job) => {
  const { mapId, userAnswers } = job.data;
  const result = await this.aiService.runDiscovery(userAnswers);
  await this.boardService.applyDiscoveryResults(mapId, result);
});
```

---

## 7. AI Layer

### OpenRouter + Vercel AI SDK

**OpenRouter:**
- Единый API для 100+ моделей (OpenAI, Anthropic, Google, open-source)
- **Бесплатные модели** для MVP (например, Meta-Llama-3 via OpenRouter free tier)
- OpenAI-compatible API — простая миграция

**Vercel AI SDK:**
- `generateObject()` — строгий typed output (Zod schema) для:
  - Opportunity Canvas (9 секций)
  - Acceptance Criteria (Given-When-Then)
  - Effort Estimation (S/M/L)
  - User Personas
- `streamText()` — streaming для discovery dialog

```typescript
import { generateObject } from 'ai';
import { openrouter } from '@openrouter/ai-sdk-provider';
import { z } from 'zod';

const { object } = await generateObject({
  model: openrouter('meta-llama/llama-3.1-70b-instruct'),
  schema: z.object({
    activities: z.array(z.object({
      title: z.string(),
      tasks: z.array(z.object({
        title: z.string(),
        persona: z.string(),
        acceptanceCriteria: z.array(z.string()),
      })),
    })),
  }),
  prompt: `Generate a story map for: ${productIdea}`,
});
```

### Voice Input

**Whisper API (OpenAI)** или **браузерный Web Speech API** для MVP:
- Web Speech API — бесплатно, offline, но качество хуже
- Whisper API — $0.006/min, отличное качество, поддержка EN/ES/DE/FR/RU

**Решение:** Web Speech API как fallback, Whisper API для точности.

### Document Processing

- **PDF**: `pdf-parse` (Node.js) или `pdf2json`
- **DOCX**: `mammoth` — извлекает текст без стилей
- **Markdown**: нативный парсинг

Извлечённый текст → OpenRouter для извлечения инсайтов (needs, pain points, feature requests).

---

## 8. Authentication

### Supabase Auth

**Из коробки:**
- Email/password (с confirmation)
- Google OAuth (1 строка конфигурации)
- JWT tokens
- RLS integration

**Enterprise (V3):** SAML/OIDC/SCIM можно добавить позже через:
- Supabase Enterprise plan
- Или переход на Auth0/Clerk/Keycloak при необходимости

### Workspace Isolation

```
User → JWT → Supabase Auth → RLS Policy → Only workspace data
```

Каждый API call включает `Authorization: Bearer <jwt>`.

---

## 9. File Storage

### Supabase Storage

- 2GB free tier
- Integrated RLS (тот же JWT, тот же policies)
- `sharp` (Node.js) для resize аватаров / preview изображений
- Upload через presigned URLs (без проксирования через backend)

```typescript
// Client-side upload
const { data, error } = await supabase.storage
  .from('attachments')
  .upload(`boards/${boardId}/${fileName}`, file, {
    upsert: false,
    contentType: file.type,
  });
```

---

## 10. Export Architecture

### Client-Side Generation

| Формат | Библиотека | Почему client-side |
|--------|-----------|-------------------|
| PNG | `html-to-image` | Screenshot DOM элемента, нет server load |
| PDF | `jsPDF` + `html2canvas` | Генерация в worker thread, offline |
| CSV | `papaparse` | Тривиально, мгновенно |
| PRD (MD) | Template literals | Нет генерации, просто string concat |
| PRD (DOCX) | `docx` (npm) | Client-side generation в Blob |

**Почему не Puppeteer/Playwright:**
- Нет server load (важно для cheap hosting)
- Работает offline
- Нет headless browser overhead (500MB RAM + 3-5 сек на screenshot)
- Мгновенный feedback для пользователя

### JIRA/Linear/GitHub Export

OAuth → API calls из frontend или backend job (pg-boss):

```
User clicks "Export to JIRA" → OAuth popup → Token stored encrypted → 
pg-boss job: create Epic → create Stories → set Fix Version → notify user
```

---

## 11. Hosting & Deployment

### Fly.io

- **Не спит** (в отличие от Render free tier)
- 3GB persistent volume (для временных файлов, кэшей)
- Multi-region deployment (при необходимости)
- `fly.toml` — простая конфигурация
- Docker-based deployment

```toml
# fly.toml
app = 'storycraft-ai'
primary_region = 'fra'

[build]
  dockerfile = 'Dockerfile'

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false  # не спим
  auto_start_machines = true
  min_machines_running = 1
```

### Supabase

- Managed PostgreSQL (zero-config)
- Автоматические backups
- Point-in-time recovery
- Connection pooling (PgBouncer) — важно для serverless/lambda workloads

---

## 12. Development Workflow

### Local Development

```bash
# 1. Start Supabase locally
npx supabase start

# 2. Start backend (with hot reload)
bun run --watch src/main.ts

# 3. Start frontend (Vite)
cd frontend && bun run dev

# 4. Socket.io работает на том же порту (NestJS Gateway)
```

### Environment Variables

```bash
# .env
DATABASE_URL=postgresql://postgres:postgres@localhost:54322/postgres
SUPABASE_URL=http://localhost:54321
SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...
OPENROUTER_API_KEY=...
```

### Type Safety

- **Zod** для валидации env, API inputs, AI outputs
- **Prisma** или **Drizzle** для ORM (type-safe queries)
- **Shared types** между frontend и backend (monorepo или package)

---

## 13. Cost Estimate (MVP: 1000 активных пользователей)

| Сервис | Стоимость/мес | Примечание |
|--------|--------------|------------|
| Fly.io | $5-10 | 1-2 shared CPU, 512MB RAM |
| Supabase | $0 (free tier) | 500MB DB, 2GB storage, 2GB bandwidth |
| OpenRouter | $0-20 | Бесплатные модели для MVP |
| Whisper API | $0-10 | Только если активно используется voice input |
| **Итого** | **$5-40/мес** | |

### Scale (10 000 пользователей)

| Сервис | Стоимость/мес |
|--------|--------------|
| Fly.io | $50-100 |
| Supabase Pro | $25 |
| OpenRouter | $100-500 |
| **Итого** | **$175-625/мес** |

---

## 14. Риски и Митигация

| Риск | Вероятность | Митигация |
|------|------------|-----------|
| Bun несовместим с какой-то библиотекой | Низкая | NestJS полностью поддерживает Bun. Fallback: Node.js (один флаг в package.json) |
| Yjs + Socket.io сложнее в отладке, чем Liveblocks | Средняя | Логирование всех broadcast, Yjs DevTools, тестовые сценарии conflict resolution |
| React Flow плохо масштабируется за 1000+ карточек | Низкая | Viewport culling + LOD + виртуализация (если нужно — миграция на Canvas-based renderer) |
| Supabase vendor lock-in | Низкая | PostgreSQL — стандарт, миграция на self-hosted/AWS/GCP — pg_dump |
| OpenRouter free tier нестабилен | Средняя | OpenRouter поддерживает paid models с тем же API. Миграция на OpenAI/Anthropic — 1 строка |

---

## 15. Миграционные Пути

| От | К | Сложность | Когда |
|----|---|-----------|-------|
| Supabase Auth | Auth0 / Clerk | Низкая | Enterprise SAML/SCIM требования |
| pg-boss | BullMQ + Redis | Низкая | Если нужны сложные job patterns (job dependencies) |
| Socket.io + Yjs | Liveblocks | Средняя | Если хочется меньше кода поддержки real-time |
| Fly.io | AWS / GCP | Средняя | PCI/SOC2 compliance, dedicated infrastructure |
| OpenRouter | OpenAI Direct | Низкая | Одна строка (baseURL + apiKey) |
| React Flow | Custom Canvas | Высокая | Если нужен Figma-level performance на 10K+ элементов |

---

## Appendix A: API Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT (Browser)                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ React 19     │  │ Zustand      │  │ Yjs + Socket.io      │  │
│  │ React Flow   │  │ (local state)│  │ (sync)               │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTPS / WSS
┌──────────────────────────▼──────────────────────────────────────┐
│                        FLY.IO                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    NestJS (Bun)                           │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ │   │
│  │  │ Auth   │ │ Board  │ │ AI     │ │ Export │ │ Integr.│ │   │
│  │  │ Module │ │ Module │ │ Module │ │ Module │ │ Module │ │   │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘ │   │
│  │  ┌──────────────────────────────────────────────────────┐ │   │
│  │  │ Socket.io Gateway (msgpack, rooms: board:${id})      │ │   │
│  │  └──────────────────────────────────────────────────────┘ │   │
│  └──────────────────────────┬───────────────────────────────┘   │
│                             │ HTTP / TCP                        │
│  ┌──────────────────────────▼───────────────────────────────┐   │
│  │              SUPABASE (PostgreSQL + Auth + Storage)       │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │   │
│  │  │ RLS      │  │ Yjs Snap │  │ Versions │  │ pg-boss  │  │   │
│  │  │ Policies │  │ shots    │  │ History  │  │ Queue    │  │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                           │ HTTPS
┌──────────────────────────▼──────────────────────────────────────┐
│              EXTERNAL SERVICES                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ OpenRouter   │  │ JIRA API     │  │ Whisper (optional)   │  │
│  │ (LLM)        │  │ Linear API   │  │                      │  │
│  │              │  │ GitHub API   │  │                      │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Appendix B: WebSocket Message Protocol

```typescript
// Client → Server
interface ClientMessage {
  type: 'yjs-update' | 'cursor-move' | 'user-join' | 'user-leave';
  boardId: string;
  payload: Uint8Array | CursorPosition | null;
}

// Server → Client
interface ServerMessage {
  type: 'yjs-update' | 'cursor-update' | 'user-list' | 'error';
  boardId: string;
  userId: string;
  payload: Uint8Array | CursorPosition | string[] | ErrorInfo;
}
```

- `yjs-update`: Бинарный Yjs update (msgpack encoded)
- `cursor-move`: `{ x: number, y: number, color: string, name: string }`
- `user-join/leave`: Broadcast в room

---

## Appendix C: AI Prompting Patterns

### Discovery Dialog

```
System: You are a product discovery assistant. Guide the user through 
structured discovery with adaptive questions.

State Machine:
1. Parse product idea → extract domain, problem, implied users
2. Generate question 1 (about target users)
3. Based on answer → generate question 2 (about current solutions)
4. Continue for 5-7 questions
5. Generate Opportunity Canvas + Personas + Journey Map

Rules:
- Max 7 questions
- Each question has "skip" option
- Maintain full conversation context
- Questions in user's language
```

### Gap Analysis

```
Input: Story map (activities + tasks + personas)
Output: List of gaps with severity (Critical/Warning/Suggestion)

Checks:
- Missing auth flow steps
- Missing error handling
- Missing admin functions
- Missing onboarding
- Orphan tasks (no activity)
- Overloaded activities (>10 tasks)

Constraints:
- Max 20 suggestions
- Explain why each gap matters
- Link to relevant persona
```

---

*Generated for StoryCraft AI project.*
*Methodology: User Story Mapping by Jeff Patton*
