---
created: '2026-08-26'
difficulty: beginner
mastery: growing
source: 'Глава 1.2 курса «Фронтенд senior+ в агентную эру», подглава 1. Якорь [FSD]
  — feature-sliced.design, About → Understanding → About architecture: https://feature-sliced.design/docs/about/understanding/architecture'
status: analyzing
tags:
- frontend
- senior-front
- code-organization
- fsd
- architecture
updated: '2026-08-26'
---

# FSD — зачем архитектура (проблемы и требования)

> Подглава 1 главы 1.2 курса «Фронтенд senior+ в агентную эру». Якорь [FSD] — feature-sliced.design, About → Understanding → «About architecture». Вводная подглава: не «как раскладывать папки», а **зачем вообще** методология организации кода.

## Executive Summary (ELI5)

FSD начинает разговор об архитектуре не со структуры, а с боли: *«usually, the conversation about architecture is raised when the development stops due to certain problems in the project»* (разговор об архитектуре поднимается, когда разработка останавливается из-за проблем). Это зеркало главы 1.1: там был выбор «где и когда исполняется рендеринг», здесь — вторая половина архитектуры: «как организовать код, чтобы проект оставался понятным при росте».

## Три боли (проблемы без архитектуры)

1. **Bus-factor и онбординг** — проект понимает ограниченное число людей:
   - «It's difficult to add a person to the development»
   - «I don't understand what is happening in this big piece of monolith»
2. **Неявные и неконтролируемые последствия** («всё зависит от всего»):
   - «The feature imports the feature»
   - «I updated the store of one page, and the functionality fell off on the other»
   - «The logic is smeared all over the application, and it is impossible to track where the beginning is, where the end is»
3. **Неконтролируемое переиспользование** — ровно **две крайности**:
   - логика пишется полностью с нуля для каждого модуля → дубли по кодовой базе («I have **N** implementations of the same business logic»);
   - всё переносится в `shared` → свалка модулей, большинство используется в одном месте («6 different components of the button», «Dump of helpers»).

⚠️ Крайности описаны про **модули и логику вообще**, не только про UI-компоненты.

## Три группы требований к «идеальной архитектуре»

| Группа | Суть | Ключевые пункты |
|---|---|---|
| **Явность (Explicitness)** | легко освоить и объяснить | структура отражает бизнес-ценности; явные связи/побочные эффекты; легко ловить дубли; логика не распылена; мало разнородных правил |
| **Контроль (Controlfulness)** | архитектура ускоряет разработку | декомпозиция и изоляция; каждый компонент заменяем и **удаляем**; «better optimize for deletion» |
| **Адаптивность (Adaptability)** | применима почти везде | любой этап проекта; без зависимости от фреймворка; масштабируется с командой; параллельная работа |

**«Better optimize for deletion» (оптимизируй под удаление, не под изменение):** будущее предсказать нельзя — *«No need to optimize for changes — we can't predict the future»*; а удаление опирается на контекст, который уже есть — *«Better optimize for removal — based on the context that already exists»*. Принадлежит группе **Control**.

## Связи с другими заметками

- [[Static-Rendering]] / [[React Server Components]] — глава 1.1 отвечала «где/когда исполняется»; эта глава — «как структурировать». Оси ортогональны (см. упр.3 главы 1.2).
- Следующие подглавы главы 1.2: обзор FSD → слои/слайсы/сегменты → public API → кросс-импорты. Каждое правило там — ответ на одну из болей выше.

## Self-check

1. Три группы проблем? → онбординг/bus-factor; неявные последствия; неконтролируемое переиспользование.
2. Две крайности переиспользования? → копировать с нуля vs свалка всего в shared.
3. Три группы требований? → явность, контроль, адаптивность.
4. «Optimize for deletion» — что и к какой группе? → писать код так, чтобы легко удалялся без каскада проблем; группа **Control** (не «неконтролируемые последствия» — это из списка проблем!).
