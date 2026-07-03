# Deep Research Prompt v3 (Master): The Future of Fullstack Interviews in the AI Era

> **Назначение:** Единый мастер-промпт, объединяющий все направления предыдущих исследований (Phase 1 — общий ландшафт; Phase 2 — мутация классических вопросов) в одно forward-looking исследование. Установленные факты Phase 1 вшиты как **baseline** — агент НЕ переоткрывает их, а строит поверх и экстраполирует в будущее.
>
> **Связанные заметки:** [[AI-Era Fullstack Interview Landscape]], [[ai-era-fullstack-interview-research-prompt]] (Phase 1), [[ai-era-classic-questions-research-prompt-v2]] (Phase 2), отчёт v1 в `ai-era-full-stack-result/report/report.md`.
> **Как использовать:** Скопировать "Часть 2" целиком в Deep Research агента. Baseline уже встроен — отдельно прикреплять report.md не обязательно (но можно для глубины).

---

## Часть 1. Контекст (для меня, не для агента)

**Где мы:** Phase 1 дал снимок текущего состояния (структурный сдвиг: coding ↓, System Design ↑, AI-блоки, разрешение AI на интервью). Phase 2 (промпт) был нацелен закрыть пробел по классике. Теперь я хочу **не два разрозненных отчёта, а один синтез**, который смотрит вперёд.

**Что я хочу от этого промпта:**
1. Объединить оба трека (AI-native + классика) в одну картину.
2. Использовать уже добытые данные как фундамент, а не пересобирать их.
3. **Сделать упор на будущее** — куда дрейфует вектор на 12–24 месяца, к чему готовиться, чтобы быть впереди кривой, а не догонять вчерашний день.

**Зачем:** Я строю learning path и portfolio на 6–12 месяцев. Готовиться к собеседованию 2026 года, которое к моменту моего выхода на рынок уже устареет, — стратегическая ошибка. Мне нужен прогноз, а не только снимок.

---

## Часть 2. Deep Research Prompt (копировать в агента)

> **Роль:** Ты — стратегический аналитик технического найма и forecasting-исследователь. Ты прогнозируешь, как будут выглядеть собеседования fullstack-инженеров в горизонте 12–24 месяцев, опираясь на текущие данные и векторы изменений. Ты не пересказываешь настоящее — ты экстраполируешь его в будущее с обоснованием.
>
> **Задача:** На основе встроенного baseline (текущее состояние, добытое в Phase 1) построить **forward-looking карту** собеседований fullstack-разработчика: куда движутся и AI-native темы, и классический канон; что станет table stakes завтра; к чему готовиться сегодня, чтобы быть релевантным через год. Главный выход — **future-proof стратегия подготовки** (learning path + portfolio), приоритизированная по горизонту времени.

> [!important] SCOPE (жёсткая рамка — не выходи за неё)
> Всё исследование строго в рамках двух связанных доменов:
> 1. **Full Stack Web Development** — React/Next.js, TypeScript, Node.js, фронтенд + бэкенд + БД.
> 2. **Cross-Platform Mobile Development — React Native** (включая Expo, нативные мосты, мобильная специфика).
>
> Это **единый стек одного инженера** (React/React Native — общий ядровый навык). НЕ уходи в data science, ML research, нативный iOS/Android (Swift/Kotlin) отдельно от RN, DevOps-only или embedded. Каждый блок анализа и каждая рекомендация должны быть применимы к fullstack-web + RN-mobile инженеру. Где web и mobile расходятся (например, mobile system design, offline-first, app store, нативные модули) — разбирай отдельно.

### 2.1. BASELINE — установленные факты (НЕ переоткрывай, прими как данность)

> [!info] Это результаты Phase 1. Считай доказанным. Опирайся, не пересобирай. Твоя ценность — в том, что **дальше**.

**Anatomy 2026 (текущие веса):** Coding ~20% (↓), System Design ~25% (↑), AI-Related ~20% (↑↑), Behavioral ~20% (→), Architecture ~10% (→), Take-home ~15% (эволюционирует).

**Подтверждённые тренды:**
- Coding/LeetCode теряет сигнал (AI решает мгновенно); System Design растёт.
- «AI evaluation» (ревью AI-кода) встраивается в раунды; у Google — отдельный code-comprehension round.
- RAG/agents/MCP спрашивают даже на не-AI fullstack-ролях.
- Планка senior: «cheap implementation, expensive understanding» — оценка по specs/review/architecture.
- Junior-найм −13% (Harvard, 62M резюме); «lean senior teams».
- Компании раскололись на 3 лагеря: embrace AI (Google/Meta/Shopify/Canva), запрет + AI-resistant evals (Anthropic), в процессе адаптации (enterprise).
- Растущие роли: Forward Deployed Engineer (+800% вакансий), AI Engineer. Девальвация: «traditional coder».
- «Vibe coding trap»: портфолио легко сгенерировать → ценится evidence of judgment (commit history, debug loops, reverts).

**Текущие differentiators:** GenAI System Design (RAG), AI code review, eval frameworks, agent orchestration, MCP, LLM gateway/cost optimization, prompt engineering.

**Пробел Phase 1 (закрой по ходу):** не показано, как **классические** вопросы (DSA, классический System Design, frontend/backend deep-dives, CS fundamentals) мутировали — только декларация «они трансформировались».

### 2.2. Центральный вопрос исследования

**Куда дрейфует собеседование инженера с единым стеком Full Stack Web + React Native (cross-platform mobile) в горизонте 12–24 месяцев (2027–2028), и что нужно начать осваивать СЕЙЧАС, чтобы в момент выхода на рынок быть впереди кривой — по обоим трекам (AI-native и классика) и обоим доменам (web и mobile)?**

### 2.3. Направления анализа (разбери каждое С УКЛОНОМ В БУДУЩЕЕ)

#### D-1. Траектория AI-native тем (что дальше после RAG/agents/MCP)
- RAG/agents/MCP сейчас — differentiators. Когда они станут **table stakes**, и что займёт их место как новый differentiator?
- Какие AI-темы только зарождаются в JD/собеседованиях и взлетят к 2027? (например: AI evals as a discipline, multi-agent reliability, context engineering, AI security/red-teaming, on-device/edge inference, agent observability, cost governance).
- Будет ли отдельный «AI evaluation round» стандартом к 2027 (Phase 1: пока только Google формализовал)?

#### D-2. Будущее классического канона (мутация + прогноз)
Для каждого классического блока: **как мутировал к 2026** → **куда движется к 2027–2028**.
- **DSA/Coding:** вернётся ли «честный» live-coding через прокторинг/on-site как анти-AI мера, или окончательно сместится в «explain/review/debug AI-output»?
- **Классический System Design** (не-GenAI: Twitter/Uber/rate limiter/cache): останется ядром? Будет ли сливаться с GenAI System Design в гибрид?
- **Frontend deep-dives** (event loop, reconciliation, rendering, a11y, perf): вырастут ли в значимости («архитектура фронта — человеческая задача»)?
- **Backend/API/DB** (schema, indexing, transactions, concurrency, caching): станут ли важнее как фундамент, который AI не закрывает?
- **CS fundamentals:** усилятся ли как фильтр «понимание vs vibe-coding» и основа AI-resistant evals?

#### D-2b. Mobile / React Native трек (отдельно от web)
- **RN deep-dives:** bridge / JSI / new architecture (Fabric, TurboModules), Hermes, рендеринг и перформанс на устройстве, навигация, нативные модули. Как это спрашивают сейчас и куда движется?
- **Mobile System Design:** offline-first, sync/conflict resolution, локальное хранилище, push-нотификации, фоновые задачи, app lifecycle, energy/battery, app size. Появляются ли AI-гибриды (on-device inference, AI-фичи в мобайле)?
- **Cross-platform специфика:** RN vs Expo vs нативка — что спрашивают; shared code между web и mobile (monorepo, React Native Web); как AI меняет mobile-разработку и mobile-собеседования.
- **Where web ≠ mobile:** что в классике/AI-темах для mobile принципиально иначе, чем для web, и как это отражается в вопросах.

#### D-3. Инварианты vs Переменные (фундамент сквозь время)
- Что НЕ изменится ни в 2026, ни в 2028 — вечный фундамент, в который безопасно инвестировать сейчас.
- Что меняется быстро (формат/оценка/инструменты) — где нужно держать руку на пульсе.

#### D-4. Дрейф индустрии → дрейф найма (causal forecast)
- Как сдвиги в самой разработке (agentic workflows, spec-driven development, AI-native архитектуры) изменят требования найма к 2027?
- Какие роли появятся/исчезнут дальше? Что будет с уровнями (junior/mid/senior/staff) при дальнейшем удешевлении кода?
- Куда движется граница «что делает человек vs что делает AI» в инженерной работе, и как это отразится в вопросах?

#### D-5. Форматы и анти-AI гонка вооружений
- Эволюция форматов: исчезнет ли take-home? вырастет ли on-site/live? появятся ли «build with AI, defend without AI» гибриды?
- Как будет развиваться «AI-resistant evaluation» (Anthropic-подход) — станет ли это мейнстримом или нишей?
- Появятся ли стандартизированные AI-fluency сертификации/бенчмарки в найме?

### 2.4. Гипотезы о будущем (валидируй с обоснованием прогноза)

Для каждой: **Likelihood (High/Med/Low)**, **Time Horizon (6/12/24 мес)**, **Leading Indicators (что уже видно сейчас)**, **Confidence (1-5)**.

| # | Гипотеза о будущем |
|---|---------------------|
| F1 | RAG/agents/MCP к 2027 станут table stakes; новым differentiator станет надёжность/оценка AI-систем (evals, multi-agent reliability, observability). |
| F2 | Классический System Design не умрёт, а сольётся с GenAI в гибридные задачи («design X, и добавь AI-feature с учётом cost/latency»). |
| F3 | Frontend/backend deep-dives и CS fundamentals вырастут в весе как «human moat» — то, что AI не закрывает и что отличает понимание от vibe-coding. |
| F4 | Отдельный «AI evaluation / code comprehension» раунд станет стандартом loop к 2027 (сейчас только Google формализовал). |
| F5 | Гонка вооружений: рост on-site/proctored/AI-resistant форматов параллельно с ростом «build-with-AI» раундов — рынок поляризуется, а не унифицируется. |
| F6 | Появится новый класс вопросов про «инженерию контекста/спецификаций» (spec-driven, context engineering) как ядро senior-оценки. |
| F7 | Portfolio-сигнал сместится от «готового продукта» к «прозрачному процессу» (traces, evals, decision logs) — потому что продукт легко сгенерировать. |
| F8 | Cross-platform владение (web + React Native из одного ядра) станет растущим differentiator: компании ценят инженера, закрывающего web и mobile, сильнее, чем двух узких. AI усиливает этот эффект (один человек + AI покрывает оба домена). |
| F9 | Mobile-собеседования отстают от web в adoption AI-форматов, но повторят траекторию web с лагом 6–12 мес; on-device AI и offline-first станут новыми mobile-differentiators. |

### 2.5. Требования к выходу

Структурируй строго:

```markdown
# Отчёт v3: The Future of Fullstack Interviews (Master Synthesis)

## 1. Executive Summary (4-6 абзацев)
- Главный forward-looking вывод: куда всё придёт к 2027–2028.
- Топ-3 сдвига, к которым готовиться СЕЙЧАС.
- Что безопасно инвестировать (инварианты) vs что мониторить (переменные).

## 2. Unified Interview Map: Now → Next (ГЛАВНАЯ ТАБЛИЦА)
| Блок/Тема | Домен (Web/Mobile/Shared) | Сейчас (2026, из baseline) | Вектор | Прогноз 2027–2028 | Статус-сдвиг (differentiator→table stakes?) | Что делать сейчас |
|-----------|---------------------------|----------------------------|--------|-------------------|----------------------------------------------|-------------------|
(покрой ОБА трека: AI-native и классику; и ОБА домена: web и mobile/RN)

## 3. AI-Native Track: Trajectory
- Что после RAG/agents/MCP; зарождающиеся темы и их leading indicators.

## 4. Classic Track: Mutation → Forecast
- По каждому классическому блоку: как мутировал → куда движется. Before/after примеры.
- Отдельный подраздел **4.x Mobile / React Native**: RN deep-dives, mobile system design, cross-platform специфика — как мутируют и куда движутся.

## 5. Invariants vs Variables (инвестиционная карта)
- Вечный фундамент (инвестировать глубоко сейчас).
- Быстро меняющееся (держать руку на пульсе).

## 6. Future Hypotheses Validation (F1–F9)
| # | Likelihood | Horizon | Leading Indicators | Confidence |

## 7. Industry Drift → Hiring Forecast
- Causal-цепочка: изменения в разработке → изменения в вопросах и ролях.

## 8. Future-Proof Preparation Strategy (ГЛАВНЫЙ ВЫХОД)
Приоритизация по горизонту времени:
### 8.1 Learning Path
| Горизонт | Тема | Трек (AI/Classic/Fundamental) | Домен (Web/Mobile/Shared) | Зачем (связь с прогнозом) | Усилие |
| Now (0–1 мес) | ... | ... | ... | ... | ... |
| Next (1–3 мес) | ... | ... | ... | ... | ... |
| Later (3–12 мес) | ... | ... | ... | ... | ... |
### 8.2 Portfolio Projects (process-transparent)
| Проект | Домен (Web/Mobile/Shared) | Что демонстрирует (с учётом F7) | Покрываемые тренды | Scope |
- Приоритет проектам с **shared-skill leverage** (один проект демонстрирует и web, и RN — например, monorepo с общим ядром).
### 8.3 Time Allocation
- Рекомендованное соотношение времени по двум осям: (а) AI Track / Classic Track / Fundamentals; (б) Web / Mobile / Shared. Обоснуй.

## 9. Risk & Uncertainty Register
| Прогноз | Риск ошибки | Что мониторить (signal) | Хедж |

## 10. Sources & References (с датами, приоритет 2025–2026 + forward-looking аналитика)
```

### 2.6. Инструкции по выполнению

1. **Не пересобирай baseline.** Раздел 2.1 — данность. Твоя ценность в экстраполяции и синтезе, а не в повторении.
2. **Каждый прогноз = вектор + leading indicators.** Не «думаю, будет X», а «уже видно A, B, C → значит к горизонту T вероятно X». Прогноз без индикаторов сейчас — не принимается.
3. **Объединяй треки.** Не делай два изолированных раздела «AI» и «классика» без связи — в разделе 2 и 8 они должны быть в одной картине с распределением времени между ними.
4. **Разделяй уверенность по горизонтам.** 6 мес — высокая уверенность; 24 мес — сценарии. Помечай уровень.
5. **Before → After для классики обязателен** (закрываем пробел Phase 1): конкретные примеры, как вопрос звучал и как звучит/будет звучать.
6. **Контекст кандидата:** senior инженер с **единым стеком web + mobile** — React/Next.js/TypeScript/Node.js (fullstack web) + React Native/Expo (cross-platform mobile). Активно использует AI. Горизонт выхода на рынок 6–12 мес. Frontend/RN-уклон классики — подробнее. Везде, где возможно, показывай **shared-skill leverage** (что качает оба домена сразу) vs domain-specific (что только web или только mobile).
7. **Главный выход — разделы 2, 5 и 8.** Карта Now→Next, инвестиционная карта инвариантов и future-proof стратегия с распределением времени. Всё сходится к плану действий на будущее.
8. **Свежесть + forecasting.** Источники 2025–2026 для фактов; для прогнозов опирайся на векторы (action крупных компаний, JD-тренды, заявления лидеров), честно помечая, где экстраполяция.
9. **Учитывай неопределённость.** Где сценарии расходятся — давай 2-3 сценария (base/bull/bear) вместо ложной точности.

---

## Приложение: Глоссарий

| Термин | Определение |
|--------|-------------|
| **Baseline (Phase 1)** | Установленные факты о текущем состоянии собеседований; принимаются как данность. |
| **AI-Native Track** | Темы, рождённые AI-эрой: RAG, agents, MCP, evals, LLM gateway, prompt/context engineering. |
| **Classic Track** | Классический канон: DSA, классический System Design, frontend/backend deep-dives, CS fundamentals. |
| **Инвариант** | Элемент, не меняющийся во времени — вечный фундамент для инвестиций сейчас. |
| **Переменная** | Быстро меняющийся элемент (формат/оценка/инструменты) — требует мониторинга. |
| **Leading indicator** | Уже наблюдаемый сигнал, на основе которого строится прогноз. |
| **Status-сдвиг** | Переход навыка из differentiator в table stakes (или устаревание). |
| **Process-transparent portfolio** | Портфолио, показывающее процесс мышления (traces, evals, decision logs), а не только готовый продукт. |
| **Horizon (Now/Next/Later)** | Временные окна подготовки: 0–1 / 1–3 / 3–12 мес. |
| **Shared-skill leverage** | Навык/проект, прокачивающий оба домена сразу (web + React Native из общего React-ядра). |
| **Domain (Web/Mobile/Shared)** | Принадлежность темы: только web, только mobile (RN), или общее ядро для обоих. |
| **Base/Bull/Bear сценарии** | Три варианта прогноза при высокой неопределённости. |
