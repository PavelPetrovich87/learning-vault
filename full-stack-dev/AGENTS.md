# full-stack-dev — Agent Guidelines

Этот файл задаёт локальные конвенции для папки `full-stack-dev/` learning-vault.

## Назначение

Конспект курса **«AI-Native Full-Stack: бэкенд + агентский SDLC»** (8 недель × 18 ч, 144 ч). Курс ведётся локальным агентом в отдельном репозитории `fullstack-agentic-trainer/`. Эта папка — зеркало пройденного материала: тексты глав и флеш-карты для повторения.

> **Важно:** здесь лежат **только принятые** главы. Текущее производство (черновики, упражнения, протокол) — в репозитории курса. В vault попадает финальный `content.md` после того, как пользователь сказал «Глава принята».

## Структура

```
full-stack-dev/
├── AGENTS.md            # этот файл
├── concept_notes/       # тексты глав: <N.M>_<Slug>.md  (1.1_Event-Loop.md, 2.3_MVCC.md, ...)
├── flash_cards/         # <N.M>_<Slug>-Flashcards.md   (по файлу на главу)
└── diagrams/            # .drawio-файлы диаграмм (рендерятся плагином draw.io inline)
```

## Конвенции

| Поле | Значение |
|---|---|
| **deck_tag** | `#flashcards/full-stack-dev` |
| **extra_tags** | `[full-stack, agentic-sdlc, backend]` + slug темы (event-loop, mvcc, idempotency…) |
| **source_link** | `true` — каждая карта и заметка ссылается на главу: `Source: [[1.1 Event loop]]` |
| **filename** | `{N.M}_{Slug}.md` — номер главы курса + slug (например `1.1_Event-Loop.md`) |
| **status** | `analyzing` (читаю) → `experimenting` (делаю упражнения) → `implemented` (проектная глава сдана) → `archived` |

## Формат concept-note

Текст главы = **академический текст**, близкий к источнику (книге/докам), НЕ формат «ELI5 + analogy» из `Technical-Deep-Dive.md`. Поэтому при создании через MCP используется `body_override` — полный текст главы минует шаблон. Шапка frontmatter сохраняется:

```yaml
---
status: implemented
mastery: growing
tags: [full-stack, agentic-sdlc, backend, event-loop]
source: "[НД] Node.js Design Patterns, 4ed, ch.1 + ch.11"
created: 2026-08-02
updated: 2026-08-02
difficulty: intermediate
---
```

## Диаграммы

- Формат: `.drawio` (плагин `draw.io` v3.1.1 в Obsidian рендерит inline).
- Лежат в `diagrams/`, встраиваются в заметку: `![[reactor-pattern.drawio]]`.
- Производятся через drawio MCP (`open_drawio_xml` / `open_drawio_mermaid`) — автораскладка `postLayout: elk` для flowchart-ов.
- На сложных диаграммах (архитектура shop-api, sequence) — `routing: libavoid`.

## Флеш-карты

- Создаются автоматически после приёмки главы: 5–10 карт по ключевым концепциям.
- Доп. карты — по требованию пользователя или после провала skill-check.
- Формат: `multi` (`?`-разделитель, obsidian-spaced-repetition default).
- Каждая карта несёт `Source: [[<номер главы> <название>]]`.

## Правила (из корневого AGENTS.md vault)

- `wiki_links: filename_only` — `[[1.1 Event loop]]`, не `[[path/to/1.1 Event loop]]`.
- `never_create_empty_folders` — папки создаются под контент, не заранее.
- `flashcards_append_only` — существующие карты не правятся; `<!--SR:...-->` не трогается.
