# Архитектор AI — User Stories & DDD Bounded Contexts

## 1. User Stories (MVP)

| ID | Роль | Действие | Ценность | Приоритет | Критерии приёмки | Блок |
|---|---|---|---|---|---|---|
| **US-01** | Архитектор | выбрать архитектурный паттерн из галереи | быстро начать проектирование с проверенной структуры | **Must** | 1. **Given** галерея паттернов открыта, **when** пользователь выбирает "Clean Architecture", **then** редактор создаёт схему с предустановленными нодами (Entities, Use Cases, Controllers, DB, External Services). <br>2. Для выбранного паттерна отображается описание, ограничения и правила. <br>3. Ноды паттерна соединены связями согласно канонической структуре паттерна. | Pattern Catalog + Schema Design |
| **US-02** | Архитектор | создавать семантические ноды и связи в редакторе | визуализировать архитектуру приложения с корректной семантикой | **Must** | 1. **Given** редактор открыт, **when** пользователь добавляет ноду типа "Domain Service", **then** нода отображается на холсте с корректным семантическим типом. <br>2. Связь "depends on" создаётся между двумя нодами с валидацией типов. <br>3. Система запрещает создание семантически некорректных связей (например, Entity → Bounded Context как "implements"). | Schema Design |
| **US-03** | Архитектор | сгенерировать схему из текстового описания требований | ускорить начальный этап проектирования и снизить порог входа | **Must** | 1. **Given** пользователь ввёл текст "E-commerce с Clean Architecture, микросервисы на заказы и платежи", **when** нажимает "Сгенерировать", **then** AI создаёт схему с нодами Bounded Context (Order, Payment), Aggregates, Use Cases, Infrastructure. <br>2. Сгенерированная схема отображается в редакторе в режиме предпросмотра. <br>3. Пользователь может принять, отредактировать или отклонить результат. | AI Assistant + Schema Design |
| **US-04** | Архитектор | запросить рефакторинг существующей схемы через AI | улучшить архитектуру на основе AI-рекомендаций без ручного перестроения | **Should** | 1. **Given** схема содержит 15+ нод в одном слое без разделения, **when** пользователь запрашивает "Разделить на Bounded Contexts", **then** AI предлагает новую структуру с выделенными контекстами и связями между ними. <br>2. Пользователь может предпросмотреть diff изменений перед применением. <br>3. Рефакторинг создаёт snapshot текущего состояния перед изменениями. | AI Assistant + Schema Design |
| **US-05** | Архитектор | задать вопрос AI о конкретной ноде в контексте схемы | получить целевую помощь по части системы без потери контекста | **Should** | 1. **Given** пользователь выбрал ноду "Payment Aggregate", **when** отправляет вопрос "Почему это Aggregate, а не Entity?", **then** AI отвечает с учётом текущего состояния схемы и типа ноды. <br>2. Ответ содержит ссылки на релевантные паттерны из каталога. <br>3. История чата сохраняется в привязке к ноде и доступна при повторном выборе. | AI Assistant |
| **US-06** | Архитектор | получать предупреждение о нарушении архитектурных правил в реальном времени | избежать фундаментальных ошибок проектирования на раннем этапе | **Must** | 1. **Given** схема содержит связь от Infrastructure к Domain, **when** валидация выполняется, **then** система выделяет нарушение Dependency Rule с пояснением. <br>2. Циклические зависимости между нодами обнаруживаются и визуально помечаются. <br>3. Валидация работает инкрементально при каждом изменении схемы. | Schema Design |
| **US-07** | Архитектор | использовать undo/redo и создавать именованные snapshots | экспериментировать с архитектурой без риска потери работы | **Must** | 1. **Given** пользователь удалил ноду и изменил 3 связи, **when** нажимает Ctrl+Z, **then** все изменения откатываются в обратном порядке. <br>2. **Given** схема в стабильном состоянии, **when** пользователь создаёт snapshot "После разделения контекстов", **then** полное состояние схемы сохраняется с меткой времени. <br>3. Возможность восстановления из любого snapshot с предпросмотром. | Schema Design |
| **US-08** | Архитектор | создать проект и организовать в нём схемы | структурировать работу по клиентам, доменам или версиям системы | **Must** | 1. **Given** пользователь авторизован, **when** создаёт проект "Bank App v2", **then** проект отображается в списке с метаданными. <br>2. В проект можно добавить несколько схем (например, Context Map и детальная схема модуля). <br>3. Проект поддерживает описание, теги и дату последнего изменения. | Project Management |
| **US-09** | Архитектор | просматривать историю изменений проекта и схем | отслеживать эволюцию архитектурных решений | **Should** | 1. **Given** проект содержит 3 схемы с несколькими редакциями, **when** пользователь открывает историю, **then** отображается хронология изменений с автором и типом действия. <br>2. Можно сравнить две версии одной схемы (side-by-side diff). <br>3. Возможность отката схемы к предыдущей версии с созданием snapshot текущей. | Project Management + Schema Design |
| **US-10** | Архитектор | экспортировать схему в JSON, PNG/SVG и boilerplate структуру | интегрировать схему в документацию и генерировать начальный код проекта | **Must** | 1. **Given** схема открыта, **when** пользователь выбирает "Export → PNG", **then** генерируется изображение схемы в высоком разрешении. <br>2. JSON-экспорт содержит все ноды, связи, позиции и метаданные, пригодные для импорта. <br>3. Экспорт boilerplate создаёт zip со структурой папок и stub-файлами, соответствующими нодам схемы. | Collaboration & Export |
| **US-11** | Архитектор | получить shareable link на схему для коллег | поделиться архитектурным решением с командой или заказчиком | **Should** | 1. **Given** схема сохранена в проекте, **when** пользователь нажимает "Share", **then** генерируется публичная ссылка с readonly-доступом. <br>2. По ссылке открывается полноценный просмотрщик схемы без возможности редактирования. <br>3. Владелец может отозвать ссылку или изменить уровень доступа. | Collaboration & Export |
| **US-12** | Архитектор | сравнить два архитектурных паттерна через AI с учётом контекста проекта | выбрать оптимальный паттерн обоснованно, а не интуитивно | **Could** | 1. **Given** пользователь выбрал сравнение "Hexagonal vs Onion" для проекта с высокой частотой изменений, **when** запрашивает анализ, **then** AI выводит структурированное сравнение по критериям. <br>2. Сравнение содержит: сложность внедрения, тестируемость, гибкость к изменениям, накладные расходы. <br>3. AI даёт рекомендацию с обоснованием на основе характеристик текущего проекта. | AI Assistant + Pattern Catalog |

---

## 2. Bounded Contexts (DDD)

Система «Архитектор AI» моделируется как собственное DDD-приложение. Ниже выделены bounded contexts с их ответственностью, моделями данных и интеграциями.

| Контекст | Назначение | Entities / Aggregates | Публичный API | Интеграционные события |
|---|---|---|---|---|
| **Pattern Catalog** *(Supporting Subdomain)* | Хранение и предоставление архитектурных паттернов (Clean, Hexagonal, Onion, Layered, VSA, Modular Monolith, CQRS, Event Sourcing, BFF, Strangler Fig). Каждый паттерн содержит каноническую структуру (шаблон нод и связей), описание, ограничения и trade-offs. | **Entity**: `Pattern` (id, slug, name, category, description, constraints[], complexityLevel), `PatternStructure` (patternId, templateNodes[], templateEdges[], validationRules[]), `PatternComparison` (patternIds[], criteria[], result) | `GET /patterns` — список паттернов с фильтрацией по категории <br>`GET /patterns/{id}` — детали паттерна с ограничениями <br>`GET /patterns/{id}/template` — шаблон схемы для импорта в редактор <br>`GET /patterns/compare?ids={id1},{id2}` — сравнение по критериям | `PatternSelected` (patternId, projectId, userId) → Schema Design |
| **Schema Design** *(Core Domain)* | **Ядро продукта.** Создание, редактирование и валидация визуальных схем архитектуры. Управление семантическими нодами (Bounded Context, Aggregate, Entity, Service и др.), типизированными связями (depends on, uses, implements, sends events to и др.), позиционированием. Валидация архитектурных правил (циклы, Dependency Rule, семантическая корректность). Undo/redo, именованные snapshots. | **Aggregate Root**: `Schema` (id, projectId, name, version, lastModifiedAt) <br><br>**Entities**: <br>• `Node` (id, schemaId, type: NodeType, position: Position, metadata: JSON, createdAt) <br>• `Edge` (id, schemaId, sourceNodeId, targetNodeId, type: EdgeType, label?) <br><br>**Value Objects**: <br>• `NodeType` (DOMAIN, APPLICATION, INFRASTRUCTURE, PRESENTATION, DB, EXTERNAL_SERVICE, API_GATEWAY, MESSAGE_BUS, BOUNDED_CONTEXT, AGGREGATE, ENTITY, VALUE_OBJECT, REPOSITORY, SERVICE, CONTROLLER, DTO) <br>• `EdgeType` (DEPENDS_ON, USES, IMPLEMENTS, SENDS_EVENTS_TO, READS_FROM, WRITES_TO) <br>• `Position` (x, y, zIndex) <br>• `ValidationResult` (isValid, violations[]) <br>• `Snapshot` (id, schemaId, label, schemaState: JSON, createdAt) | `POST /schemas` — создать схему <br>`GET \| PUT \| DELETE /schemas/{id}` — CRUD <br>`POST /schemas/{id}/nodes` — добавить ноду <br>`PUT \| DELETE /schemas/{id}/nodes/{nodeId}` — изменить/удалить <br>`POST /schemas/{id}/edges` — добавить связь <br>`POST /schemas/{id}/validate` — запустить валидацию <br>`POST /schemas/{id}/snapshots` — создать snapshot <br>`POST /schemas/{id}/undo` / `/redo` <br>`GET /schemas/{id}/snapshots` — список snapshots <br>`POST /schemas/{id}/restore` — восстановить из snapshot | `SchemaCreated` (schemaId, projectId) → Project Management <br>`SchemaUpdated` (schemaId, changeType: node_added \| edge_removed \| refactored) → AI Assistant, Project Management <br>`ValidationFailed` (schemaId, violations[]) → AI Assistant (для контекстных подсказок) <br>`SnapshotCreated` (schemaId, snapshotId) → Project Management (history) |
| **AI Assistant** *(Core Domain)* | Генерация схем из текстовых описаний, рефакторинг по запросу, объяснение архитектурных решений (trade-offs), контекстный чат привязанный к ноде или схеме. AI-агент работает с состоянием схемы и каталогом паттернов для обоснованных рекомендаций. | **Entity**: `AIRequest` (id, type: generate \| refactor \| explain \| chat, context: PromptContext, payload: JSON, createdAt) <br><br>`AIResponse` (id, requestId, content: Markdown, suggestedSchemaChanges?: NodeChange[], confidenceScore?) <br><br>`ChatSession` (id, schemaId, nodeId?, userId, messages[], createdAt) <br><br>**Value Object**: `PromptContext` (schemaState: SchemaSnapshot, selectedNodeId?, patternCatalog: Pattern[], projectContext?) | `POST /ai/generate` — генерация схемы из текста (payload: prompt, projectContext?) <br>`POST /ai/refactor` — рефакторинг (payload: schemaId, instruction, preview: true) <br>`POST /ai/explain` — объяснение trade-offs (payload: schemaId, nodeId?, question) <br>`POST /ai/chat` — сообщение в контекстный чат (payload: schemaId, nodeId?, message) <br>`GET /ai/chat/{sessionId}` — история чата <br>`POST /ai/compare-patterns` — сравнение паттернов | `AISchemaGenerated` (schemaId, proposedSchema: JSON, reasoning) → Schema Design <br>`AIRefactoringSuggested` (schemaId, changeset: NodeChange[], description) → Schema Design <br>`NodeExplained` (schemaId, nodeId, explanation) → Schema Design (annotation) <br>`AIChatMessage` (sessionId, message) → Schema Design (UI layer) |
| **Project Management** *(Supporting Subdomain)* | Управление проектами как контейнерами для схем. Сохранение, список, метаданные, история изменений на уровне проекта. Связь между пользователем и его архитектурными артефактами. | **Aggregate Root**: `Project` (id, ownerId, name, description, tags[], createdAt, updatedAt, settings?) <br><br>**Entities**: <br>• `ProjectMember` (userId, projectId, role: Role) <br>• `ChangeLogEntry` (id, projectId, action: created \| updated \| deleted, entityType: schema \| snapshot, entityId, actorId, timestamp, diff?) <br><br>**Value Objects**: <br>• `Role` (OWNER, EDITOR, VIEWER) <br>• `ProjectSettings` (defaultPattern?, validationStrictness) | `POST /projects` — создать проект <br>`GET /projects` — список проектов пользователя <br>`GET \| PUT \| DELETE /projects/{id}` — CRUD <br>`GET /projects/{id}/schemas` — схемы проекта <br>`GET /projects/{id}/history` — история изменений <br>`POST /projects/{id}/members` — добавить участника <br>`PUT /projects/{id}/members/{userId}` — изменить роль | `ProjectCreated` (projectId, ownerId) → User & Access Management <br>`SchemaAddedToProject` (schemaId, projectId) → Schema Design <br>`SchemaRemovedFromProject` (schemaId, projectId) → Schema Design <br>`ProjectHistoryUpdated` (projectId, entry: ChangeLogEntry) → Collaboration & Export |
| **Collaboration & Export** *(Generic Subdomain)* | Экспорт схем во внешние форматы (JSON, PNG, SVG), генерация boilerplate-структуры проекта, импорт схем, создание и управление shareable links. | **Entity**: `ExportJob` (id, schemaId, format: ExportFormat, status: pending \| processing \| done \| failed, resultUrl?, createdAt) <br><br>`SharedLink` (id, schemaId, token, expiresAt?, accessLevel: AccessLevel, createdBy, revokedAt?) <br><br>**Value Objects**: <br>• `ExportFormat` (JSON, PNG, SVG, BOILERPLATE_ZIP) <br>• `AccessLevel` (READ, COMMENT) | `POST /export` — запуск асинхронного экспорта (payload: schemaId, format, options?) <br>`GET /export/{jobId}` — статус задачи <br>`GET /export/{jobId}/download` — скачать результат <br>`POST /import` — импорт схемы из JSON (payload: file, projectId?) <br>`POST /share` — создать shareable link (payload: schemaId, accessLevel, expiresIn?) <br>`GET /share/{token}` — доступ к схеме по ссылке <br>`DELETE /share/{token}` — отозвать ссылку | `SchemaExported` (schemaId, format, jobId) → Project Management (history) <br>`SchemaImported` (schemaId, projectId, source: upload \| link) → Project Management, Schema Design <br>`LinkShared` (schemaId, token) — audit <br>`LinkRevoked` (token) — audit |
| **User & Access Management** *(Generic Subdomain)* | Аутентификация, авторизация, управление пользователями, рабочими пространствами (workspaces) и правами доступа к проектам. Единая точка контроля доступа для всех контекстов. | **Aggregate Root**: `User` (id, email, displayName, avatarUrl, createdAt) <br><br>**Entity**: `Workspace` (id, ownerId, name, members[], settings?) <br><br>`Permission` (id, userId, projectId, role: Role, grantedAt, grantedBy) <br><br>**Value Objects**: <br>• `Role` (OWNER, ADMIN, EDITOR, VIEWER) <br>• `WorkspaceSettings` (defaultProjectVisibility) | `POST /auth/register` / `/login` / `/refresh` <br>`POST /auth/oauth/{provider}` <br>`GET /users/me` — текущий пользователь <br>`GET \| PUT /users/me` — профиль <br>`GET /workspaces` — список workspace'ов <br>`POST /workspaces` — создать workspace <br>`PUT /permissions/{projectId}` — назначить права <br>`GET /permissions/{projectId}` — проверить права текущего пользователя | `UserRegistered` (userId) <br>`UserLoggedIn` (userId, sessionId) <br>`PermissionGranted` (userId, projectId, role) → Project Management <br>`PermissionRevoked` (userId, projectId) → Project Management |

---

## 3. Схема взаимодействия контекстов

### 3.1. Текстовое описание потоков

**Pattern Catalog** выступает источником истины об архитектурных паттернах. При выборе паттерна пользователем контекст публикует событие `PatternSelected` с идентификаторами паттерна и проекта. **Schema Design** принимает это событие и инициализирует новую схему на основе шаблона структуры паттерна (получаемого через синхронный API-вызов `GET /patterns/{id}/template`).

**Schema Design** — центральный контекст ядра. Он управляет жизненным циклом схем, нод, связей, валидацией и версионированием (undo/redo, snapshots). Каждое значимое изменение схемы порождает событие `SchemaUpdated`, которое потребляется **AI Assistant** (для поддержания актуального контекста в чатах) и **Project Management** (для обновления истории и времени последнего изменения).

**AI Assistant** получает текущее состояние схемы через синхронный запрос к `Schema Design` (`GET /schemas/{id}` с полным графом) и использует его для формирования `PromptContext`. Результаты работы AI (сгенерированная схема, changeset рефакторинга, объяснение) возвращаются как события (`AISchemaGenerated`, `AIRefactoringSuggested`), которые **Schema Design** применяет как предложенные изменения. AI Assistant также обращается к **Pattern Catalog** для сравнений паттернов и контекстных подсказок.

**Project Management** агрегирует схемы в проекты. Он получает события `SchemaCreated` и `SchemaUpdated` для ведения `ChangeLogEntry`. Через свой API предоставляет список схем проекта, который **Schema Design** использует для навигации. При создании snapshot **Schema Design** публикует `SnapshotCreated`, которое **Project Management** фиксирует в истории проекта.

**Collaboration & Export** принимает полное состояние схемы от **Schema Design** (через API) для генерации экспортных файлов. Экспорт выполняется асинхронно (`ExportJob`). Импорт JSON-схемы порождает событие `SchemaImported`, которое инициирует создание новой схемы в **Schema Design** и привязку к проекту в **Project Management**. Shareable links предоставляют readonly-доступ к схемам через публичный endpoint, который внутренне делегирует запросы к **Schema Design**.

**User & Access Management** обеспечивает аутентификацию и авторизацию для всех контекстов. События `PermissionGranted` / `PermissionRevoked` синхронизируют права доступа с **Project Management**. Все публичные API остальных контекстов проверяют токены и права через middleware, обращающуюся к **User & Access Management**.

### 3.2. ASCII-диаграмма взаимодействий

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        «Архитектор AI» — Bounded Contexts                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐                                                        │
│  │  Pattern Catalog│         template (sync API)                             │
│  │  (Supporting)   │──────────────────────────────┐                         │
│  │                 │                              │                         │
│  │  [Pattern]      │    PatternSelected (async)   │                         │
│  │  [PatternStruct]│──────────────────────────────┼────┐                    │
│  └────────┬────────┘                              │    │                    │
│           ^                                       ▼    │                    │
│           │    compare/query                 ┌─────────┴────────┐           │
│           │    (sync API)                    │   Schema Design  │           │
│           └──────────────────────────────────│   (Core Domain)  │           │
│                                              │                  │           │
│  ┌─────────────────┐    schema state (sync)   │  [Schema] (AR)  │           │
│  │   AI Assistant  │<─────────────────────────│  [Node]         │           │
│  │   (Core Domain) │                          │  [Edge]         │           │
│  │                 │    changeset (async)     │  [Snapshot]     │           │
│  │  [AIRequest]    │─────────────────────────>│  [Validation]   │           │
│  │  [AIResponse]   │  AISchemaGenerated       │                 │           │
│  │  [ChatSession]  │  AIRefactoringSuggested  │  undo/redo      │           │
│  └─────────────────┘                          └────────┬────────┘           │
│                                                        │                    │
│                              SchemaCreated             │                    │
│                              SchemaUpdated             │                    │
│                              SnapshotCreated           ▼                    │
│  ┌─────────────────┐                          ┌─────────────────┐          │
│  │ Project Mgmt    │<─────────────────────────│ Collaboration   │          │
│  │ (Supporting)    │  ProjectHistoryUpdated   │ & Export        │          │
│  │                 │─────────────────────────>│ (Generic)       │          │
│  │  [Project] (AR) │  SchemaAdded/Removed     │                 │          │
│  │  [Member]       │                          │  [ExportJob]    │          │
│  │  [ChangeLog]    │<─────────────────────────│  [SharedLink]   │          │
│  └─────────────────┘   SchemaExported         └─────────────────┘          │
│           ^           SchemaImported                                        │
│           │                                                                 │
│           │  PermissionGranted / PermissionRevoked                          │
│           │                                                                 │
│  ┌────────┴─────────────────────────────────────────────────────────┐      │
│  │              User & Access Management (Generic)                  │      │
│  │                                                                  │      │
│  │  [User] (AR)  •  [Workspace]  •  [Permission]  •  [Role]       │      │
│  │                                                                  │      │
│  │  authN/authZ middleware ──────────────────────────> все контексты│      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.3. Ключевые интеграционные потоки данных

| Отправитель | Получатель | Данные / Событие | Тип коммуникации |
|---|---|---|---|
| Pattern Catalog | Schema Design | `PatternSelected` + шаблон структуры | Async событие + Sync API (`GET /template`) |
| Schema Design | AI Assistant | Полное состояние схемы (`SchemaSnapshot`) | Sync API (`GET /schemas/{id}`) |
| AI Assistant | Schema Design | `AISchemaGenerated`, `AIRefactoringSuggested` — proposed changeset | Async события |
| Schema Design | Project Management | `SchemaCreated`, `SchemaUpdated`, `SnapshotCreated` | Async события |
| Schema Design | Collaboration & Export | Полный граф схемы + метаданные | Sync API (экспорт), Async (Job-очередь) |
| Collaboration & Export | Schema Design | `SchemaImported` — создание схемы из JSON | Async событие |
| Project Management | Schema Design | Список схем проекта, история изменений | Sync API |
| User & Access Management | Все контексты | JWT-токены, проверка прав (`GET /permissions/{projectId}`) | Sync API (middleware) |
| User & Access Management | Project Management | `PermissionGranted`, `PermissionRevoked` | Async события |

### 3.4. Примечания по проектированию

1. **Schema Design — единственный Core Domain**, поскольку визуальное проектирование архитектуры является конкурентным преимуществом продукта. AI Assistant также отнесён к Core, так как AI-возможности дифференцируют продукт на рынке.

2. **Anti-Corruption Layer**: AI Assistant взаимодействует с Pattern Catalog через ACL, транслируя паттерны в свой формат `PromptContext`, чтобы избежать утечки доменной модели каталога в AI-домен.

3. **Согласованность**: Создание схемы в Schema Design и привязка к проекту требует межконтекстной саги — `SchemaCreated` обрабатывается Project Management, который подтверждает добавление через `SchemaAddedToProject`.

4. **Валидация** в Schema Design выполняется синхронно (в рамках транзакции изменения), поскольку это критичное для домена правило. Результаты `ValidationFailed` могут асинхронно отправляться в AI Assistant для генерации контекстных подсказок.
