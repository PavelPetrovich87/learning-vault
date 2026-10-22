---
status: drafting
tags: []
created: 2026-07-09
updated: 2026-10-21
---

# System Design Sandbox — 2-Месячный Интенсивный План

> **Платформа:** [systemdesignsandbox.com](https://www.systemdesignsandbox.com/) (100% Free)  
> **Формат:** Теория (Playbook) + Практика (Sandbox) + Review  
> **Цель:** Пройти 12 сценариев на 100%, уметь объяснять sizing за 2 мин, рисовать HLD за 10 мин, говорить вслух 45 мин.

---

## Правила плана

1. **Каждый сценарий минимум 2 раза** — первый проход (понять), второй (100% + bonus).
2. **System Design Journal** — после каждого сценария фиксировать паттерны, trade-offs, подсказки AI-интервьюера.
3. **Теория : Практика = 1 : 2** — на каждую статью Playbook 2 практических прохода.
4. **Воскресенье = Review** — таблицы, сравнения, связи, не новый материал.
5. **Говорить вслух** — на втором проходе каждого сценария объяснять решение голосом (можно записывать себя).

---

## Неделя 1: Фундамент — Sizing, APIs, Caching

| День  | Теория (Playbook)         | Практика (Sandbox)                | Фокус                                         | ✅   |
| ----- | ------------------------- | --------------------------------- | --------------------------------------------- | --- |
| **1** | Sizing Calculation        | —                                 | `2^10 ≈ 10^3`, KB/MB/GB/TB, секунды в сутках  | ☐   |
| **2** | API Design Best Practices | **URL Shortener** — шаги 1-3      | Требования, custom aliases, expiration        | ☐   |
| **3** | Caching Fundamentals      | **URL Shortener** — шаги 4-5      | Drag-and-drop, Redis, CDN, cache invalidation | ☐   |
| **4** | Load Balancing Basics     | **Pastebin** — шаги 1-3           | Read/write ratio, хранилище текстов           | ☐   |
| **5** | Database Sharding Intro   | **Pastebin** — шаги 4-5           | Шардирование по ID, CDN для статики           | ☐   |
| **6** | Consistency Patterns      | **Rate Limiter** — полный проход  | Token bucket, sliding window, Redis           | ☐   |
| **7** | Review                    | **Notifications** — полный проход | Pub/sub, message queues, fan-out              | ☐   |

### Journal Week 1
- [ ] Зафиксировать: 3 паттерна, которые повторяются во всех 4 сценариях
- [ ] Зафиксировать: мои слабые места (где AI давал подсказки?)
- [ ] Сводная таблица: сравнить URL Shortener vs Pastebin (что общее, что разное)

---

## Неделя 2: Фундамент — Polish + Queues

| День   | Теория (Playbook)  | Практика (Sandbox)           | Фокус                                          | ✅   |
| ------ | ------------------ | ---------------------------- | ---------------------------------------------- | --- |
| **8**  | Queues & Messaging | —                            | Kafka vs RabbitMQ, at-least-once, exactly-once | ☐   |
| **9**  | —                  | **URL Shortener** — повторно | Цель: 100% на всех шагах + bonus               | ☐   |
| **10** | —                  | **Pastebin** — повторно      | Цель: 100% + bonus                             | ☐   |
| **11** | —                  | **Rate Limiter** — повторно  | Отработать weak spots, говорить вслух          | ☐   |
| **12** | —                  | **Notifications** — повторно | Отработать weak spots, говорить вслух          | ☐   |
| **13** | Review Week 1-2    | Сводная таблица ошибок       | Паттерны, которые даются тяжело                | ☐   |
| **14** | REST vs WebSocket  | —                            | Подготовка к real-time модулю                  | ☐   |

### Journal Week 2
- [ ] Список «моих 5 любимых паттернов» с примерами систем
- [ ] Сравнительная таблица: 4 foundation-системы по latency/throughput/storage
- [ ] Записать себя на 5 мин: объясняю URL Shortener с нуля

---

## Неделя 3: Real-time + State

| День | Теория (Playbook) | Практика (Sandbox) | Фокус | ✅ |
|------|-------------------|-------------------|-------|----|
| **15** | Real-time Systems | **WhatsApp** — шаги 1-3 | WebSockets, presence, message delivery | ☐ |
| **16** | Event-Driven Architecture | **WhatsApp** — шаги 4-5 | Шардирование по user_id, E2E encryption trade-offs | ☐ |
| **17** | Streaming Data | **Leaderboard** — шаги 1-3 | Redis Sorted Sets, real-time updates, Top-K | ☐ |
| **18** | Time-Series Databases | **Leaderboard** — шаги 4-5 | Batch vs real-time, aggregation | ☐ |
| **19** | Job Scheduling | **Job Scheduler** — шаги 1-3 | Cron, distributed queues, at-least-once | ☐ |
| **20** | Distributed Transactions | **Job Scheduler** — шаги 4-5 | Saga pattern, retries, idempotency | ☐ |
| **21** | Review real-time | **WhatsApp** — повторно | Цель: 100% + bonus | ☐ |

### Journal Week 3
- [ ] Сравнить: WhatsApp vs Notifications (когда WebSocket, когда push?)
- [ ] Зафиксировать: 3 способа шардирования stateful-систем
- [ ] Таблица trade-offs: strong consistency vs eventual consistency

---

## Неделя 4: Real-time — Polish + CAP

| День | Теория (Playbook) | Практика (Sandbox) | Фокус | ✅ |
|------|-------------------|-------------------|-------|----|
| **22** | Review state | **Leaderboard** — повторно | Цель: 100% + bonus | ☐ |
| **23** | — | **Job Scheduler** — повторно | Цель: 100% + bonus | ☐ |
| **24** | — | **Notifications** → переделать с real-time lens | Как бы изменилась архитектура с WebSockets? | ☐ |
| **25** | CAP Theorem Deep Dive | — | Систематизировать trade-offs: CP vs AP | ☐ |
| **26** | Consistency Models | — | Strong vs Eventual, CRDTs, read-your-writes | ☐ |
| **27** | Review Week 3-4 | Сравнительная таблица | 3 real-time системы: общее vs разное | ☐ |
| **28** | — | **Rate Limiter** + **Leaderboard** combo | Как связать две системы? | ☐ |

### Journal Week 4
- [ ] Сравнительная матрица: 7 пройденных систем × используемые паттерны
- [ ] Записать себя: объясняю CAP theorem на примере 2 систем
- [ ] Список вопросов, которые AI-интервьюер задаёт чаще всего

---

## Неделя 5: Data + Money (сложные домены)

| День | Теория (Playbook) | Практика (Sandbox) | Фокус | ✅ |
|------|-------------------|-------------------|-------|----|
| **29** | Ledger & Double-Entry | **Payments** — шаги 1-3 | Idempotency keys, payment state machine | ☐ |
| **30** | ACID in Distributed Systems | **Payments** — шаги 4-5 | 2PC vs Saga, compensating transactions | ☐ |
| **31** | Financial Ledger Design | **Ledger** — шаги 1-3 | Immutability, append-only logs, audit | ☐ |
| **32** | Audit & Compliance | **Ledger** — шаги 4-5 | Reconciliation, event sourcing | ☐ |
| **33** | Metrics & Monitoring | **Metrics** — шаги 1-3 | Time-series, aggregation, retention policies | ☐ |
| **34** | Data Pipeline Patterns | **Metrics** — шаги 4-5 | Kafka → ClickHouse, rollups, downsampling | ☐ |
| **35** | Ad Tech Architecture | **Ad Clicks** — шаги 1-3 | Fraud detection, bidding, real-time constraints | ☐ |

### Journal Week 5
- [ ] Сравнить: Payments vs Ledger (где какой паттерн применяется?)
- [ ] Зафиксировать: 3 способа обеспечить idempotency
- [ ] Таблица: ACID vs BASE — где какое применять

---

## Неделя 6: Data + Money — Polish + CDNs

| День | Теория (Playbook) | Практика (Sandbox) | Фокус | ✅ |
|------|-------------------|-------------------|-------|----|
| **36** | Stream Processing | **Ad Clicks** — шаги 4-5 | Windowing, deduplication, exactly-once | ☐ |
| **37** | Review money | **Payments** — повторно | Цель: 100% + bonus | ☐ |
| **38** | Review data | **Metrics** — повторно | Цель: 100% + bonus | ☐ |
| **39** | — | **Ledger** — повторно | Отработать double-entry, говорить вслух | ☐ |
| **40** | — | **Ad Clicks** — повторно | Отработать stream processing, говорить вслух | ☐ |
| **41** | CDNs & Edge Caching | — | Теория для media-систем: edge, origin, cache-hit | ☐ |
| **42** | Review Week 5-6 | Сводная таблица: Money vs Data | Какие паттерны повторяются? | ☐ |

### Journal Week 6
- [ ] Сравнительная матрица: 11 пройденных систем × паттерны
- [ ] Записать себя: объясняю Saga pattern на примере Payments
- [ ] Список «мои 3 самых сильных домена» и «3 самых слабых"

---

## Неделя 7: Media + Интеграция

| День | Теория (Playbook) | Практика (Sandbox) | Фокус | ✅ |
|------|-------------------|-------------------|-------|----|
| **43** | Distributed File Systems | **Dropbox** — шаги 1-3 | Chunking, deduplication, sync, conflict detection | ☐ |
| **44** | Conflict Resolution | **Dropbox** — шаги 4-5 | OT/CRDT, last-write-wins, vector clocks | ☐ |
| **45** | Web Crawler Architecture | **Web Crawler** — шаги 1-3 | Politeness, Bloom filters, frontier, robots.txt | ☐ |
| **46** | Distributed Storage | **Web Crawler** — шаги 4-5 | Storage backend, dedup at scale, indexing | ☐ |
| **47** | Video Streaming | **YouTube** — шаги 1-3 | Adaptive bitrate, transcoding, CDN, storage | ☐ |
| **48** | Content Delivery | **YouTube** — шаги 4-5 | Video storage, recommendation infra, playback | ☐ |
| **49** | Review media | **Dropbox** — повторно | Цель: 100% + bonus | ☐ |

### Journal Week 7
- [ ] Сравнить: Dropbox vs Web Crawler (deduplication strategies)
- [ ] Сравнить: YouTube vs URL Shortener (как CDN используется по-разному)
- [ ] Таблица: 12 систем × latency requirements (какие системы самые чувствительные?)

---

## Неделя 8: Final Polish + Mock Interviews

| День | Теория (Playbook) | Практика (Sandbox) | Фокус | ✅ |
|------|-------------------|-------------------|-------|----|
| **50** | Review crawler | **Web Crawler** — повторно | Цель: 100% + bonus | ☐ |
| **51** | Review video | **YouTube** — повторно | Цель: 100% + bonus | ☐ |
| **52** | — | **URL Shortener** → переделать с media lens | Как бы изменилась архитектура для видео? | ☐ |
| **53** | — | **Payments** + **YouTube** combo | Monetization: как связать платежи и стриминг? | ☐ |
| **54** | Final Review | Все 12 сценариев | Список «мои 10 любимых паттернов» с примерами | ☐ |
| **55** | Mock Interview 1 | **WhatsApp** с таймером 45 мин | Говорить вслух, записывать себя | ☐ |
| **56** | Mock Interview 2 | **YouTube** с таймером 45 мин | Говорить вслух, записывать себя | ☐ |
| **57** | Weak spots drill | 2-3 сценария с худшими оценками | Довести до 100% | ☐ |
| **58** | Weak spots drill | Ещё 2-3 сценария | Довести до 100% | ☐ |
| **59** | Final Review | Сравнительная матрица всех 12 систем | Какой паттерн где применяется | ☐ |
| **60** | — | **Свободный день** или повторить любимый сценарий | Ментальная подготовка | ☐ |

### Journal Week 8 (Final)
- [ ] Сравнительная матрица: 12 систем × 15 паттернов (check/uncheck)
- [ ] Записать финальное видео: 45-минутный mock на случайную систему
- [ ] Список «вопросы, которые я хочу задать настоящему интервьюеру"

---

## Playbook Index (49 статей)

### Foundations
- [ ] Sizing Calculation
- [ ] API Design
- [ ] Caching Fundamentals
- [ ] Load Balancing Basics
- [ ] Database Sharding Intro
- [ ] Consistency Patterns
- [ ] Queues & Messaging

### Real-time & Streaming
- [ ] Real-time Systems
- [ ] Event-Driven Architecture
- [ ] Streaming Data
- [ ] Time-Series Databases
- [ ] Job Scheduling
- [ ] Distributed Transactions
- [ ] CAP Theorem Deep Dive
- [ ] Consistency Models
- [ ] REST vs WebSocket

### Data & Money
- [ ] Ledger & Double-Entry
- [ ] ACID in Distributed Systems
- [ ] Financial Ledger Design
- [ ] Audit & Compliance
- [ ] Metrics & Monitoring
- [ ] Data Pipeline Patterns
- [ ] Ad Tech Architecture
- [ ] Stream Processing
- [ ] CDNs & Edge Caching

### Media & Storage
- [ ] Distributed File Systems
- [ ] Conflict Resolution
- [ ] Web Crawler Architecture
- [ ] Distributed Storage
- [ ] Video Streaming
- [ ] Content Delivery

*(Остальные 21 статья — дополнительное чтение по мере необходимости)*

---

## Sandbox Scenarios Checklist

### Foundations
- [ ] URL Shortener (пройден 2+ раза, 100%+)
- [ ] Pastebin (пройден 2+ раза, 100%+)
- [ ] Rate Limiter (пройден 2+ раза, 100%+)
- [ ] Notifications (пройден 2+ раза, 100%+)

### Real-time
- [ ] WhatsApp (пройден 2+ раза, 100%+)
- [ ] Leaderboard (пройден 2+ раза, 100%+)
- [ ] Job Scheduler (пройден 2+ раза, 100%+)

### Money & Data
- [ ] Payments (пройден 2+ раза, 100%+)
- [ ] Ledger (пройден 2+ раза, 100%+)
- [ ] Metrics (пройден 2+ раза, 100%+)
- [ ] Ad Clicks (пройден 2+ раза, 100%+)

### Media
- [ ] Dropbox (пройден 2+ раза, 100%+)
- [ ] Web Crawler (пройден 2+ раза, 100%+)
- [ ] YouTube (пройден 2+ раза, 100%+)

---

## Итоговые метрики успеха (Day 60)

- [ ] **Sizing:** Объяснить «на салфетке» за 2 минуты (storage + throughput)
- [ ] **HLD:** Нарисовать high-level design любой системы за 10 минут
- [ ] **Подача:** Говорить вслух 45 минут без пауз и «э-э-э»
- [ ] **Паттерны:** Назвать 15+ паттернов и привести пример системы для каждого
- [ ] **Trade-offs:** Аргументировать каждый выбор (Redis vs Memcached, Kafka vs RabbitMQ, SQL vs NoSQL)
- [ ] **Bonus:** Пройти любой из 12 сценариев на 100% за 1 попытку

---

## System Design Journal Template

После каждого сценария заполнять:

```
Дата: ___________
Сценарий: ___________
Проход: 1-й / 2-й / 3-й
Оценка: ___%

Паттерны, которые использовал:
1. ___________
2. ___________
3. ___________

Trade-offs, которые выбрал:
- ___________ vs ___________ (почему первое?)

Что AI-интервьюер подсказал, а я не догадался:
- ___________

Что было самым сложным:
- ___________

Что повторить в следующий раз:
- ___________
```

---

*План составлен на основе материалов System Design Sandbox (systemdesignsandbox.com). 60 дней. 12 сценариев. 49 статей. 100% free.*
