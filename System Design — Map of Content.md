---
status: stable
tags: [system-design, moc, navigation, architecture]
created: 2026-10-21
updated: 2026-10-21
---

# System Design — Map of Content

Навигационный хаб по всем зонам vault'а, где живёт system design и архитектура.
Четыре зоны не дублируют друг друга — у каждой своя роль:

| Зона | Роль | Статус (2026-10-21) |
|---|---|---|
| `System Design interview/` | Evergreen-конспекты распределённых систем (CAP, репликация, latency) | поддерживаемая |
| `system-design-udemy/` | Udemy-курс: networking, protocols, scalability, паттерны | archived (пройден 2026-07) |
| `Modeling Software Architecture/` | Моделирование: viewpoints, perspectives, C4, TOGAF | archived (пройден 2026-07) |
| `system-design-course/` | Русский курс-тренажёр: метод алгоритма, модули | активная |

## Тема → где лежит

| Тема | Основная заметка | Соседи |
|---|---|---|
| Networking-базы (DNS, CDN, proxy, client-server) | `system-design-udemy/networking/` | [[Latency Budget - System Design]] |
| API Gateway | `system-design-udemy/networking/api_gateways/concept_notes/API_Gateways` | [[REST-API-URL-Design]] |
| Load balancing | [[Load Balancing Algorithms - Blind Spots and the Weakest Server]] | udemy `networking/load_balancing` |
| Stateless/stateful, эволюция веб-тира | [[Scaling Evolution - from Single Server to Stateless Web Tier]] | [[Stateless vs Stateful - the SAM Lens]], курс `1.5 Масштабирование stateless` |
| CAP / PACELC | [[CAP-Theorem]] | [[PACELC-Theorem]] |
| Репликация | [[Data-Replication-Distributed-Systems]] | [[Quorum-Consensus-Consistency-Models]], [[Vector-Clocks-Versioning]] |
| Партиционирование, consistent hashing | [[Data-Partitioning-Consistent-Hashing]] | [[Distributed-KV-Store-Architecture]] |
| Отказоустойчивость | [[Failure-Handling-Distributed-Systems]] | [[Dual-Write-Problem-and-Transactional-Outbox]] |
| Capacity planning | [[Little's Law and Capacity Planning]] | [[Headroom - Practical Capacity Ceilings]], [[Percentiles and Tail Latency]], [[Back-of-the-Envelope Size Calculations]], курс `1.2 Математика нагрузки` |
| Требования FR/NFR | [[Functional vs Non-Functional Requirements]] | курс `1.3 Требования FR-NFR` |
| Моделирование архитектуры (C4, viewpoints) | `Modeling Software Architecture/concept_notes/` | [[c4 model]], [[viewpoint]], [[4+1 view model]] |
| Паттерны стилей (monolith, microservices, event-driven, multi-layer) | `system-design-udemy/software_architecture_patterns_and_styles/` | udemy `microservices_architecture` |
| DDD | `Backend/Microservices/Domain-Driven Design/` | `System Design interview/concept_notes/DDD/` |
| Классические кейсы | [[URL-Shortener-Design]] | курс `1.6 Проект Мессенджер v1` |

## Свежее ↔ пройденное (мосты)

Заметки из `notes/` (архитектура eloomi-next, 2026-10) опираются на июльскую теорию:

- [[Dual-Write-Problem-and-Transactional-Outbox]] ← [[Data-Replication-Distributed-Systems]], [[Failure-Handling-Distributed-Systems]]
- [[REST-API-URL-Design]] ← `API_Gateways` (udemy)
- [[Postgres-RLS-Multi-Tenancy]] — мультиарендность как альтернатива шардированию: [[Data-Partitioning-Consistent-Hashing]]

## Поддержка карты

- Новую system-design тему класть в зону по её роли (таблица выше), а строку — сюда.
- Если заметка про распределённые системы написана «на вырост» — это `System Design interview/concept_notes/`; конспект урока — `system-design-udemy/`; разбор метода — `system-design-course/`.
