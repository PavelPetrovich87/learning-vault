# IT Business Analysis: Free Bootcamp

Собственный учебный курс по IT Business Analysis — бесплатно, с упором на техническую сторону профессии.

---

## 1. Существующие бесплатные курсы (проверенные)

### 🇷🇺 Русскоязычные

| Курс | Платформа | Что внутри | Ссылка |
|------|-----------|------------|--------|
| **Бизнес-аналитик: первые шаги** | Нетология | Введение в профессию, жизненный цикл ПО, сбор требований | [netology.ru](https://netology.ru) |
| **Базовый курс бизнес-анализа** | SF Education | Методологии, типы требований, введение в BPMN | [sf.education](https://sf.education) |
| **Введение в аналитику** | Skillbox | 11 направлений аналитики, тест на профориентацию, SQL-введение | [skillbox.ru](https://skillbox.ru) |

### 🌍 Англоязычные (бесплатно / audit)

| Ресурс | Что внутри | Как получить бесплатно |
|--------|------------|------------------------|
| **Coursera — Business Analysis Fundamentals** (UC Irvine) | Требования, stakeholder analysis, документация | "Audit the course" — без сертификата, но контент бесплатно |
| **edX — Business Analysis** (University of British Columbia) | BABOK-aligned, требования, моделирование | Audit mode |
| **IIBA — Entry Certificate in Business Analysis (ECBA)** | Официальные материалы IIBA | Free PDF handbook при регистрации на сайте IIBA |
| **YouTube — "Business Analysis in Agile"** (разные каналы) | User stories, acceptance criteria, refinement | Поиск: "agile business analyst tutorial" |

---

## 2. «IT Business Analysis: Free Bootcamp»

Учитывая бэкграунд в разработке, курс построен с упором на **техническую сторону BA** — то, что отличает IT-аналитика от обычного.

### 📋 Структура курса (8 недель, ~5-7 часов в неделю)

---

#### **Модуль 1. Роль IT Business Analyst**
**Темы:**
- Чем IT BA отличается от обычного BA
- BA в Waterfall vs BA в Agile/Scrum
- Границы ответственности: BA, Product Owner, System Analyst, UX
- Стейкхолдеры в IT-проектах: заказчик, разработка, тестирование, DevOps

**Практика:** Составить RACI-матрицу для вымышленного проекта (e-commerce сайт)

**Источники:** 
- BABOK Guide v3 (free chapters) — iiba.org
- Статьи: "The Role of BA in Scrum" (Scrum.org)

---

#### **Модуль 2. Требования в IT**
**Темы:**
- Типы требований: бизнес, пользовательские, функциональные, нефункциональные (NFR)
- FURPS+ модель для NFR
- Техники сбора: интервью, observation, prototyping, document analysis
- MoSCoW приоритизация

**Практика:** 
- Провести "интервью" с другом (он — заказчик, ты — BA)
- Записать 10 требований и классифицировать их по типам

**Источники:**
- Книга: "Software Requirements" (Karl Wiegers) — есть free PDF в сети
- Шаблон SRS (IEEE 830) — бесплатно на ieee.org

---

#### **Модуль 3. User Stories & Acceptance Criteria**
**Темы:**
- Формат user story: "As a [role], I want [goal], so that [reason]"
- INVEST критерии качества
- Acceptance Criteria: Given-When-Then (Gherkin)
- Definition of Ready vs Definition of Done
- Story splitting techniques

**Практика:**
- Написать 15 user stories для мобильного банка
- Написать AC в Gherkin для 5 из них
- Провести "refinement" с самим собой: декомпозировать 3 большие истории

**Источники:**
- YouTube: "User Story Masterclass" (Mike Cohn)
- Статья: "Gherkin for Business Analysts" (cucumber.io)

---

#### **Модуль 4. UML и моделирование для BA**
**Темы:**
- Use Case Diagrams (actors, boundaries, relationships)
- Activity Diagrams (swimlanes, decisions)
- Sequence Diagrams (взаимодействие систем)
- BPMN 2.0 basics (для бизнес-процессов)
- Data Flow Diagrams (DFD)

**Практика:**
- Нарисовать Use Case Diagram для системы онлайн-записи к врачу
- Нарисовать Sequence Diagram для процесса "Пользователь оплачивает заказ"
- BPMN-схема процесса "Обработка возврата товара"

**Инструменты (бесплатно):**
- draw.io (diagrams.net) — полностью бесплатно
- PlantUML — текстовый DSL для диаграмм
- Lucidchart — free tier

---

#### **Модуль 5. SQL и работа с данными**
**Темы:**
- SELECT, WHERE, JOIN (INNER, LEFT, RIGHT)
- GROUP BY, HAVING, агрегатные функции
- Подзапросы
- Data profiling: проверка качества данных
- Базовое понимание ER-диаграмм

**Практика:**
- SQLZoo.net — бесплатные интерактивные упражнения
- Mode Analytics SQL Tutorial — бесплатно
- Написать 10 запросов к вымышленной БД интернет-магазина

---

#### **Модуль 6. API и интеграции**
**Темы:**
- Что такое REST API (GET, POST, PUT, DELETE)
- JSON структура
- Swagger / OpenAPI Specification
- Как BA анализирует API: endpoint, request/response, status codes
- Postman для BA: тестирование API без кода

**Практика:**
- Найти публичный API (JSONPlaceholder, OpenWeather)
- Составить "API Requirements Document" для 3 endpoints
- Протестировать в Postman

**Источники:**
- Postman Learning Center (бесплатно)
- Swagger Editor (online, бесплатно)

---

#### **Модуль 7. Документация в IT**
**Темы:**
- SRS (Software Requirements Specification) — структура
- BRD (Business Requirements Document)
- User Story Map
- Traceability Matrix (требования → код → тесты)
- Confluence / Notion для документации
- Jira: эпики, стории, задачи, связи

**Практика:**
- Создать SRS на 10-15 страниц для вымышленного проекта
- Создать Traceability Matrix в Excel/Google Sheets
- Настроить Jira-проект (free tier до 10 пользователей)

---

#### **Модуль 8. Подготовка к собеседованию**
**Темы:**
- Типовые вопросы на собеседовании BA в IT
- Кейс-задачи: "Как ты соберёшь требования для..." 
- Портфолио: как показать проекты без опыта работы
- Сертификации: ECBA (IIBA), PSPO/PSM (Scrum.org)

**Практика:**
- Разбор 20 типовых вопросов
- Составить своё резюме с IT-акцентом
- Подготовить 3 кейса для портфолио

---

## 3. Практические проекты для портфолио (делай в процессе)

| Проект | Что делаешь | Инструменты |
|--------|-------------|-------------|
| **Мобильный банк** | 20 user stories, AC, Use Case Diagram, SRS | Jira, draw.io, Google Docs |
| **Система онлайн-записи** | BPMN процессы, SQL-запросы, API документация | draw.io, SQLZoo, Postman |
| **E-commerce возвраты** | As-Is / To-Be анализ, stakeholder map, requirements | Miro (free), Notion |

---

## 4. Чек-лист: что ты должен уметь после курса

- [ ] Провести интервью с заказчиком и выявить 10+ требований
- [ ] Написать user story с INVEST-критериями
- [ ] Написать acceptance criteria в Gherkin
- [ ] Нарисовать Use Case Diagram в UML
- [ ] Нарисовать BPMN-схему процесса
- [ ] Написать SQL-запрос с JOIN и GROUP BY
- [ ] Прочитать Swagger-документацию и найти ошибку
- [ ] Составить SRS документ
- [ ] Создать Traceability Matrix
- [ ] Настроить Jira-проект с эпиками и связями

---

## 5. Где всё это взять бесплатно — сводная таблица ресурсов

| Ресурс | Что там | Ссылка |
|--------|---------|--------|
| **BABOK Guide v3** | Библия BA, 500+ страниц | iiba.org (free PDF для членов, $14/год) |
| **SQLZoo** | Интерактивный SQL | sqlzoo.net |
| **draw.io** | UML/BPMN редактор | diagrams.net |
| **Postman** | API тестирование | postman.com |
| **Jira** | Управление требованиями | atlassian.com (free до 10 users) |
| **Miro** | Воркшопы, схемы | miro.com (free tier) |
| **Coursera Audit** | Курсы UC Irvine, UBC | coursera.org |
| **edX Audit** | Университетские курсы | edx.org |
| **YouTube** | Agile BA, User Stories | поиск по каналам: "KnowledgeHut", "Simplilearn" |
