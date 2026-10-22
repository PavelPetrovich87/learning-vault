---
status: stable
tags: [vault, audit, changelog]
created: 2026-10-21
updated: 2026-10-21
---

# Vault Audit — 2026-10-21

Ченджлог уборки 2026-10-21 + остаточные задачи. Полный аудит (возраста, статусы, сироты) сделан в этой же сессии; здесь — только результаты и что осталось.

## Что сделано (2026-10-21)

| # | Операция | Объём |
|---|---|---|
| 1 | **Удалён `project_august/`** (проект закрыт) | 28 файлов; восстановление: `git checkout HEAD -- project_august` |
| 2 | 4 ссылки на удалённые файлы отвязаны в `AI-Era Fullstack Interview Landscape.md` | ссылки → простой текст |
| 3 | **Нормализация enum'ов**: `status: draft→drafting (6), partial→analyzing (2), fixed→implemented (1), growing→stable (1, словарь)`; `difficulty: easy→beginner (11), medium→intermediate (8), hard→advanced (2)`; `mastery: intermediate→growing (7)` | 37 заметок |
| 4 | **Статусы замороженных зон**: `system-design-udemy`, `Modeling Software Architecture`, `DEVOPS`, `playwright` → `archived` (225); `Backend`, `agentic_ai` → `stable` (70); заметкам с FM без `status` статус дописан | 295 заметок |
| 5 | Frontmatter добавлен 6 заметкам без него (реальный хвост; остальные 120+ «без FM» — flashcards-исключения и служебные файлы) | 6 заметок |
| 6 | **AGENTS.md обновлён**: enum `status` + `drafting`/`stable` с определениями, опциональное поле `reviewed` (дата последней ревизии evergreen-заметки), описана `notes/`, `project_august` помечен удалённым, footer исправлен | — |
| 7 | **MOC**: [[System Design — Map of Content]] — карта 4 зон system design + мосты | 1 заметка |
| 8 | Мосты живое↔замороженное: `Dual-Write…` → репликация/отказоустойчивость; `REST-API-URL-Design` → API Gateway | 2 заметки |
| 9 | Починены битые ссылки: `Harness-Engineering-Overview` → `1-Harness-Engineering-Overview`; висячий `Daily Plan 2026-06-08` | 1 заметка |

Все изменённые заметки получили `updated: 2026-10-21`.

## Сироты: что осталось (154, из них реально стоящих линковки — единицы)

Не сироты, несмотря на ноль wikilinks:
- **`sleight-of-mouth/knowledge/` (53)** — связан через frontmatter-ID (`related_patterns: [som-chunk-down]`), это штатный механизм этой зоны.
- **Flashcards вне `flash_cards/`** (Frontend, Business Analysis, harness) — SR-карточки, ссылка ведётся через `Source: [[note]]`.
- `daily-plans/`, `Templates/`, `CLAUDE.md`/`GEMINI.md`/`DDD_CONTEXT.md` — служебные, линковать не нужно.

Реальный остаток (по убыванию пользы):
1. `System Design interview/concept_notes/DDD/Frontend/Optimistic-Locking-Hexagonal.md` — единственный содержательный сирота evergreen-зоны; просится в [[System Design — Map of Content]].
2. `notes/Skill-Design-From-Competency-Maps.md` — связать с `Skill Creation/`.
3. `Self-hypnosis-app-DDD/` (2 файла) — остатки проекта; решить: переименовать `Untitled.md` и дооформить, или удалить по образцу project_august.
4. `harness_engineering/learning/week-2/` (journal, plan) — журнал; достаточно сослаться из верхнеуровневой заметки зоны.
5. `system-design-course/system_design_sandbox_2month_plan.md` — сослаться из `system-design-course/INDEX.md`.
6. `research_in_progress/MacBook-Pro-ANS2-PCIe-Panic-2026-07-05.md` — законченный кейс-дебаг; кандидат в `stable`.
7. `Frontend/senior-front/FSD — уровни/скилл — исходник/` (11) — исходник скилла; линковка опциональна.

## Дашборд протухания

⚠️ Плагин **Dataview не установлен** (есть только drawio и obsidian-spaced-repetition). Ниже — запросы на будущее; пока их текст можно просто грепать.

```dataview
TABLE status, updated, reviewed
WHERE updated < date(2026-10-21) - dur(60 days) AND !contains(status, "archived") AND !contains(file.folder, "_reference")
SORT updated ASC
```

```dataview
LIST WHERE length(file.inlinks) = 0 AND length(file.outlinks) = 0 AND file.ext != "mdex"
```

Без dataview то же самое одной командой из терминала (протухшие не-archived):

```bash
cd ~/learning/learning-vault && grep -rl "^updated:" --include="*.md" . | xargs grep -L "^status: archived" | xargs grep -l "^updated: 2026-0[6-8]"
```

## Правило на будущее (внесено в AGENTS.md)

`reviewed: YYYY-MM-DD` — ставится при перечитывании/подтверждении evergreen-заметки. Интервальное повторение для конспектов, как SR для карточек.
