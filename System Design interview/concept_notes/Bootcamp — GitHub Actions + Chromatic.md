---
date: 2026-05-14
type: course
created: 2026-05-14
tags: [github-actions, chromatic, ci-cd, storybook, course, fsd-shopping-cart]
source: На основе концепт-ноутов "Chromatic — Визуальная регрессия" и "GitHub Actions — Workflow файлы"
difficulty: intermediate
---

# 🎓 Bootcamp: GitHub Actions + Chromatic
## *Или как Наполеон бы автоматизировал свой деплой*

> [!quote] Предисловие от Императора
> *«Армия марширует на желудок — а хороший фронтендер марширует на CI/CD.»*
> — Наполеон Бонапарт (если бы он кодил на React)

Добро пожаловать в учебный курс, где мы превратим сухую документацию GitHub Actions и Chromatic в живую, дышащую стратегию. Без воды, без корпоративного буллшита, только мясо, аналогии и немного французской агрессии.

**Что вы вынесете:**
- Поймёте, почему CI/CD — это не «плюшка», а инфраструктура выживания.
- Научитесь читать workflow-файлы как военные карты.
- Узнаете, как Chromatic ловит баги раньше, чем ваш PM их заметит.
- Пройдёте квизы, которые отделят юнкеров от маршалов.

---

## 🏛️ Модуль 1: GitHub Actions — Великая Армия Автоматизации

### Теория: Что это и зачем

Представьте, что вы — Наполеон. У вас есть армия разработчиков (код), артиллерия (тесты), кавалерия (деплой) и обоз (сборка). Каждое утро вам нужно:
1. Проверить, что оружие не заржавело (`lint`).
2. Пострелять по мишеням (`test`).
3. Собрать обоз (`build`).
4. Отправить войска на позиции (`deploy`).

Если делать это руками — к вечеру вы устанете, а солдаты (коммиты) будут стоять без дела. **GitHub Actions — это ваш штабной автомат**, который выполняет всё по сигналу (push, PR, cron) и докладывает: *«Всё чисто, монсеньор»* или *«Артиллерия взорвалась на третьем шаге»*.

**Ключевая идея:** Workflow-файл (`.github/workflows/*.yml`) — это план битвы. Событие — это ревелль, который будит армию. Job — это отдельный корпус (артиллерийский, кавалерийский). Step — это приказ внутри корпуса. Action — это готовый батальон, который можно позвать с Marketplace (*«Пришлите мне швейцарскую гвардию `actions/setup-node`!»*).

```yaml
# Пример: План битвы при Маренго (упрощённо)
name: Grande Armée CI
on:
  push:
    branches: [main]  # Ревелль только для main
jobs:
  lint-corps:
    runs-on: ubuntu-latest  # Поле боя — равнина Linux
    steps:
      - uses: actions/checkout@v4  # Призвать картографа
      - uses: actions/setup-node@v4  # Выдать винтовки Node.js
      - run: npm ci  # Зарядить патроны
      - run: npm run lint  # Проверить, не ржавеют ли стволы
```

### Когда это нужно

| Ситуация | Без CI | С CI |
|---|---|---|
| Разработчик забыл запустить тесты локально | Баг уходит в main, ломает прод | Workflow ловит на PR, merge заблокирован |
| Дизайнер меняет токен, ломает 20 экранов | Через 3 дня крики в Slack | Chromatic показывает diff ещё на PR |
| Нужно деплоить staging в 3 часа ночи | Кто-то должен встать и нажать кнопку | `schedule: cron` делает это за вас |
| Команда из 10 человек, 5 PR в день | Хаос, конфликты, «у меня локально работает» | Параллельные checks, артефакты, порядок |

### Что будет, если не следовать правилам

> ⚠️ **Сценарий «Битва при Аустерлице без карты»**
>
> Вы хардкодите секреты прямо в workflow: `password: abc123`. GitHub Actions запускается, логи публичны, пароль виден всем. Через 2 часа вашу базу данных майнят крипту. Мораль: `secrets.*` — ваш лучший друг, а хардкод — измена родине.

> ⚠️ **Сценарий «Артиллерия стреляет раньше пехоты»**
>
> У вас нет `needs:` между job'ами. `deploy` запускается параллельно с `test`. Тесты ещё не прошли, а прод уже обновлён сломанным билдом. Мораль: `needs` — это цепь командования. Артиллерия ждёт сигнала пехоты.

> ⚠️ **Сценарий «Тридцать тысяч солдат и ни одного коня»**
>
> Вы используете `npm install` вместо `npm ci`. Зависимости обновились, локально билд собирается, на runner'е падает с криптической ошибкой. 2 часа дебага, оказывается — minor-апдейт ломает API. Мораль: `npm ci` читает `package-lock.json` как военный устав. Не спорьте с уставом.

### 🎯 Мини-квиз: «Юнкер или Маршал?»

> **Вопрос 1.** Вы видите в workflow: `uses: actions/checkout@main`. Что скажет маршал?
>
> <details>
> <summary>Показать ответ</summary>
>
> **Проблема:** `actions/checkout@main` означает «возьми Action из ветки `main` репозитория `actions/checkout`». Ветка `main` — это живой, движущийся таргет. Сегодня там стабильный код, завтра мейнтейнер влил PR с breaking change — и ваш workflow сломался, хотя вы ничего не меняли.
>
> **Что такое «пинить» (pin):** Вместо плавающей ветки указать конкретную версию, которая не изменится:
>
> ```yaml
> # ❌ Плохо — ветка живёт и меняется
> uses: actions/checkout@main
>
> # ✅ Хорошо — конкретный тег, обновляется только при major релизе
> uses: actions/checkout@v4
>
> # ✅ Ещё лучше — конкретный коммит SHA, никогда не изменится
> uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1
> ```
>
> **Почему SHA — золотой стандарт:** Даже тег `v4` теоретически можно переписать (force-push). SHA коммита — это криптографический хеш, его нельзя подменить. Для production-кода и security-critical проектов — только SHA.
>
> **Практический пример:** В начале 2023 года `tj-actions/changed-files` (популярный Action) был скомпрометирован — злоумышленник получил доступ и подменил ветку `main`. Все workflow с `@main` начали выполнять вредоносный код. Те, кто пинил SHA, остались в безопасности.
>
> **Аналогия (если уж нужна):** Вместо «берите последнее издание учебника из магазина» (где издатель мог перепечатать главу) — «используйте именно 4-е издание, страница за страницей».
> </details>

> **Вопрос 2.** Зачем `concurrency` с `cancel-in-progress: true`?
>
> <details>
> <summary>Показать ответ</summary>
> Разработчик пушит 3 коммита подряд — запускаются 3 армии. Зачем тратить ресурсы на первые две, если третья уже актуальна? `concurrency` отменяет устаревшие походы, экономя минуты CI (и нервы казначея).
> </details>

> **Вопрос 3.** В чём разница между `secrets` и `vars`?
>
> <details>
> <summary>Показать ответ</summary>
> `secrets` — шифрованные, скрыты в логах (пароли, токены). `vars` — открытые, видны в логах (URL, имена). `vars` для нечувствительного, `secrets` для всего, что нельзя показывать шпионам.
> </details>

---

## 🏛️ Модуль 2: Chromatic — Императорская Гвардия Визуальной Регрессии

### Теория: Что это и зачем

Представьте: вы — Наполеон, и вы только что сменили цвет мундира гвардейцев с тёмно-синего на тёмно-зелёный. Вы думаете: *«Ну чуть-чуть, кто заметит?»* А через неделю выясняется, что на всех портретах, медалях и баннерах цвет разный, и гвардейцы выглядят как болотные лягушки. Художники (дизайнеры) в ярости, народ смеётся, репутация подорвана.

**Chromatic — это ваш личный инспектор мундиров.** Он фотографирует каждый компонент UI в каждом состоянии и сравнивает с эталоном. Если пиксель сдвинулся — тревога. Если кнопка стала красной вместо синей — тревога. Если отступ уехал на 2px — тревога.

```
Developer push → GitHub Actions → Build Storybook → Chromatic Cloud
    → Capture screenshots → Compare with baseline
        → No changes: ✅ Pass
        → Changes: 🟡 Pending Review → Accept / Deny
```

**Ключевой момент:** Chromatic не заменяет Storybook — он использует его. Ваши `.stories.tsx` — это тест-кейсы. Каждый story — это одна фотография компонента в одном состоянии.

### Когда это нужно

| Ситуация | Без Chromatic | С Chromatic |
|---|---|---|
| Разработчик меняет padding в `Button`, ломает `Modal` | Никто не замечает до продакшена | Diff показан на PR, дизайнер видит side-by-side |
| Дизайн-система из 50 компонентов, обновление токенов | Ручная проверка 200+ состояний | Автоматические скриншоты всех состояний |
| Ночной релиз, все спят | Баг уходит в 3 часа ночи | Workflow блокирует merge до review |
| Дизайнер говорит «это не то, что я нарисовал» | Спор без доказательств | Chromatic UI — объективный арбитр |

### Что будет, если не следовать правилам

> ⚠️ **Сценарий «Бaseline из битвы при Ватерлоо»**
>
> Вы создали baseline с багнутого состояния. Теперь Chromatic считает баг нормой. Каждый PR проходит зелёным, пока клиент не пришлёт скриншот сломанной кнопки. Мораль: первый baseline — это конституция. Если она кривая, страна обречена. Пересоздавайте baseline вручную, если ошиблись.

> ⚠️ **Сценарий «Турбо-экономия без топлива»**
>
> Вы включили `onlyChanged: true`, но забыли `fetch-depth: 0` в checkout. Chromatic не видит git history — TurboSnap отключается, снимаются ВСЕ stories. Квота сгорит за один PR. Мораль: `fetch-depth: 0` — это топливо для TurboSnap. Без него вы толкаете танк ногами.

> ⚠️ **Сценарий «Стоящая армия без денег»**
>
> Вы запускаете Chromatic на каждый push в любую ветку. 10 разработчиков × 20 веток × 50 stories = вы исчерпали бесплатную квоту к обеду вторника. Мораль: запускайте на `pull_request` и `push: [main]`. Всё остальное — роскошь, которую вы не можете себе позволить.

> ⚠️ **Сценарий «Анимированный флаг на ветру»**
>
> В ваших stories есть анимации, таймеры, случайные данные. Каждый прогон даёт разный скриншот. Chromatic кричит «изменение!» на каждый PR. Мораль: stories должны быть детерминированными. Отключайте анимации, мокайте даты, фиксируйте случайность.

### 🎯 Мини-квиз: «Гвардеец или Ополченец?»

> **Вопрос 1.** Почему Chromatic запускается ДВАЖДЫ: на PR и на push в main?
>
> <details>
> <summary>Показать ответ</summary>
> На PR — сравнение с baseline для поиска регрессий. На push в main — обновление baseline (`autoAcceptChanges: true`), чтобы следующие PR сравнивались с актуальным эталоном. Это как обновлять карту после каждой битвы.
> </details>

> **Вопрос 2.** В чём разница между `exitZeroOnChanges: true` и strict mode?
>
> <details>
> <summary>Показать ответ</summary>
> `exitZeroOnChanges: true` (soft mode) — Chromatic показывает diff, но GitHub check зелёный. PR можно мержить без review. Strict mode (без флага) — GitHub check висит Pending до Accept в Chromatic UI. Soft mode для внедрения, strict mode для зрелой команды.
> </details>

> **Вопрос 3.** Chromatic скажет «border должен быть красным»?
>
> <details>
> <summary>Показать ответ</summary>
> Нет. Chromatic сравнивает пиксели, не стили. Он скажет «скриншот изменился», но не объяснит почему. Для проверки конкретных CSS-правил нужны Rule Based Assertions (computed styles через Playwright/Vitest).
> </details>

---

## 🏛️ Модуль 3: Кампания — Интеграция в Полевых Условиях

### Теория: Связываем всё в единый фронт

У Наполеона была мечта: **Великая Армия**, где каждый корпус действует согласованно. Артиллерия готовит, пехота штурмует, кавалерия преследует. Но если корпуса не синхронизированы — хаос.

Вот как выглядит **согласованный CI/CD pipeline** для фронтенд-проекта:

```yaml
# ci.yml — Главный штаб
name: CI
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run test:unit

  build:
    runs-on: ubuntu-latest
    needs: lint-and-test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/

# chromatic.yml — Императорская Гвардия
name: Chromatic
on:
  pull_request:
    branches: [main]
    paths:
      - 'src/**'
      - 'package.json'
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'package.json'

jobs:
  chromatic:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci

      - name: Chromatic (main)
        if: github.ref == 'refs/heads/main'
        uses: chromaui/action@latest
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          autoAcceptChanges: true
          onlyChanged: true

      - name: Chromatic (PR)
        if: github.event_name == 'pull_request'
        uses: chromaui/action@latest
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          exitZeroOnChanges: true
          onlyChanged: true
```

**Почему два файла, а не один?**

| Подход | Плюс | Минус |
|---|---|---|
| Один `ci.yml` | Просто, всё в одном месте | Chromatic тормозит lint, длинный файл |
| Разные workflow (`ci.yml`, `chromatic.yml`) | Параллельность, изоляция | Много файлов |

**Рекомендация:** разделять по доменам. `ci.yml` — lint/test/build. `chromatic.yml` — визуальная регрессия. `deploy.yml` — staging/production.

### Боевые сценарии

> 🎭 **Сценарий «Срочное приказание»**
>
> Ночью пришёл баг-репорт: кнопка «Купить» не работает на мобильных. Нужен hotfix.
> 1. Создаём ветку `hotfix/button-mobile` от `main`.
> 2. Фиксим, пушим, создаём PR.
> 3. GitHub Actions запускает `ci.yml` и `chromatic.yml`.
> 4. Chromatic показывает: «Ваш фикс изменил padding кнопки на 2px».
> 5. Дизайнер проверяет diff, нажимает Accept.
> 6. PR мержится, `main` получает auto-accept baseline.
> 7. `deploy.yml` (с `if: github.ref == 'refs/heads/main'`) деплоит в прод.
>
> Всё заняло 15 минут. Без CI — часа два и пара нервных срывов.

> 🎭 **Сценарий «Диверсия в тылу»**
>
> Разработчик случайно удалил `fetch-depth: 0` при рефакторинге workflow. Следующий PR с изменением одного токена вызывает съёмку всех 200 stories. Квота сгорает.
> **Что делать:**
> 1. GitHub → Actions → находим упавший/зависший workflow.
> 2. Отменяем его (Cancel workflow).
> 3. Возвращаем `fetch-depth: 0`.
> 4. Пушим фикс, проверяем, что TurboSnap работает (в логе Chromatic должно быть `Using TurboSnap`).

### 🎯 Мини-квиз: «Генерал или Капрал?»

> **Вопрос 1.** Зачем `paths` фильтр в `chromatic.yml`?
>
> <details>
> <summary>Показать ответ</summary>
> Если разработчик обновил только README или документацию, Chromatic не запускается. Экономит минуты CI и квоту. Фильтр: запускать только при изменении `src/**` или `package.json`.
> </details>

> **Вопрос 2.** Как передать собранный `dist/` из `build` job в `deploy` job?
>
> <details>
> <summary>Показать ответ</summary>
> Через `actions/upload-artifact@v4` в `build` и `actions/download-artifact@v4` в `deploy`. Job'ы изолированы — без артефактов файлы не передаются.
> </details>

> **Вопрос 3.** Почему `deploy` job должен иметь `needs: build`?
>
> <details>
> <summary>Показать ответ</summary>
> Чтобы деплой случился только после успешной сборки. Без `needs` deploy запустится параллельно и может задеплоить сломанный билд или вообще не найти артефакт.
> </details>

---

## 🏆 Финальный Квиз: Аустерлиц GitHub Actions

*Это не шутки. Здесь вопросы, которые задают на собеседованиях senior+ уровня. Ответьте на все — и вы маршал. Ошибётесь — ещё один поход.*

---

### Вопрос 1: Стратегия окружений
**У вас есть staging и production. Как сделать так, чтобы:**
- На staging деплоилось автоматически после merge в `develop`.
- На production деплоилось только после approve от релиз-менеджера.
- На production использовался отдельный токен, отличный от staging.

Напишите минимальный `deploy.yml` с использованием `environment`, `secrets`, `if` и `needs`.

<details>
<summary>Показать ответ</summary>

```yaml
name: Deploy
on:
  push:
    branches: [develop, main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/

  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-files
          path: dist/
      - run: |
          echo "Deploying to staging with token: ${{ secrets.STAGING_DEPLOY_TOKEN }}"
          npm run deploy:staging

  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: production  # Требует approve!
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-files
          path: dist/
      - run: |
          echo "Deploying to production with token: ${{ secrets.PROD_DEPLOY_TOKEN }}"
          npm run deploy:production
```

**Почему это работает:**
- `environment: production` создаёт защищённое окружение. В настройках репозитория можно добавить required reviewers.
- Разные secrets для staging и production scoping через `secrets.*`.
- `if` гарантирует, что production деплой не запустится из `develop`.

</details>

---

### Вопрос 2: Матрица и fail-fast
**Вам нужно протестировать приложение на Node 18, 20, 22 и на Ubuntu + Windows. Как написать job с матрицей? Как сделать так, чтобы при падении одной комбинации остальные продолжали работать?**

<details>
<summary>Показать ответ</summary>

```yaml
jobs:
  test-matrix:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest]
      fail-fast: false  # Важно! По умолчанию true — при первом падении всё отменяется.
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

**Будет запущено 6 job'ов** (3 версии Node × 2 ОС). С `fail-fast: false` вы увидите, какие комбинации падают, а не только первую.

</details>

---

### Вопрос 3: Reusable Workflow с параметрами
**У вас 5 микрофронтендов. Все используют одинаковый pipeline: checkout → setup-node → install → lint → test → build. Как избежать дублирования workflow-файлов?**

<details>
<summary>Показать ответ</summary>

Создаём reusable workflow:

```yaml
# .github/workflows/reusable-ci.yml
name: Reusable CI
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
        default: '22'
    secrets:
      npm-token:
        required: false

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build
```

И вызываем из каждого микрофронтенда:

```yaml
# .github/workflows/app1.yml
name: App1 CI
on: push
jobs:
  call-reusable:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: '22'
```

**Плюс:** единый стандарт, изменения в одном месте. **Минус:** сложнее debug — ошибка может быть в reusable workflow, а не в вызывающем.

</details>

---

### Вопрос 4: Debug вывода предыдущего шага
**В job `build` есть шаг, который генерирует build ID: `echo "id=$(date +%s)" >> $GITHUB_OUTPUT`. Как использовать это значение в job `deploy`, который зависит от `build`?**

<details>
<summary>Показать ответ</summary>

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      build-id: ${{ steps.generate-id.outputs.id }}
    steps:
      - id: generate-id
        run: echo "id=$(date +%s)" >> $GITHUB_OUTPUT

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying build ${{ needs.build.outputs.build-id }}"
```

**Ключевые моменты:**
- Шагу нужен `id` для обращения к его `outputs`.
- Job нужен `outputs` блок для экспорта.
- Зависимая job обращается через `needs.<job-name>.outputs.<output-name>`.

</details>

---

### Вопрос 5: Conditional steps и secrets
**У вас есть workflow, который деплоит на Vercel. Для preview-деплоев (PR) нужен один токен, для production — другой. Как написать условные шаги с правильными secrets?**

<details>
<summary>Показать ответ</summary>

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy Preview
        if: github.event_name == 'pull_request'
        run: vercel --token=${{ secrets.VERCEL_PREVIEW_TOKEN }}

      - name: Deploy Production
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: vercel --prod --token=${{ secrets.VERCEL_PROD_TOKEN }}
```

**Важно:** `secrets.*` маскируются в логах. Даже если кто-то напечатает `${{ secrets.VERCEL_PROD_TOKEN }}`, GitHub заменит его на `***`.

</details>

---

### Вопрос 6: Race condition в artifacts
**Job `build` сохраняет артефакт с именем `build-files`. Job `deploy` и `test-e2e` оба зависят от `build` и скачивают этот артефакт. Что произойдёт, если `build` упадёт на шаге `upload-artifact`?**

<details>
<summary>Показать ответ</summary>

Если `build` упадёт до `upload-artifact` — артефакт не сохранится. Job'ы `deploy` и `test-e2e` не запустятся, потому что `needs: build` ждёт успешного завершения.

Если `build` упадёт ПОСЛЕ `upload-artifact` — артефакт сохранится, но job'ы `deploy` и `test-e2e` всё равно не запустятся, потому что `build` завершился с ошибкой.

**Мораль:** `needs` ждёт **успешного** завершения. Если нужно запускать даже при падении — используйте `if: always()` или `if: failure()` в зависимых job'ах (но осторожно).

</details>

---

### Вопрос 7: Cron и таймзоны
**Вы настроили `schedule: - cron: '0 9 * * 1'`. В какой таймзоне это выполнится? Как сделать так, чтобы cron работал по московскому времени (UTC+3)?**

<details>
<summary>Показать ответ</summary>

GitHub Actions cron работает в **UTC**.

`0 9 * * 1` = 9:00 UTC = 12:00 MSK (летом) или 11:00 MSK (зимой, если нет DST).

Для московского времени:
```yaml
on:
  schedule:
    - cron: '0 6 * * 1'  # 6:00 UTC = 9:00 MSK
```

**Подвох:** UTC не учитывает переход на летнее/зимнее время. Если вам критично точное время — используйте `workflow_dispatch` и внешний scheduler, или принимайте смещение ±1 час.

</details>

---

### Вопрос 8: Self-hosted runner vs GitHub-hosted
**Ваша компания использует self-hosted runner на старом сервере под столом. Он падает каждый второй запуск. Какие риски self-hosted runners и когда они оправданы?**

<details>
<summary>Показать ответ</summary>

**Риски self-hosted:**
- Единая точка отказа (сервер под столом может отключиться).
- Состояние между запусками (файлы из прошлого запуска могут мешать).
- Безопасность (runner имеет доступ к внутренней сети, secrets могут утекать через злоумышленный PR).
- Обслуживание (обновления, чистка диска, мониторинг).

**Когда оправданы:**
- Доступ к внутренним ресурсам (VPN, приватные репозитории, внутренние API).
- Нужны специфические ресурсы (GPU, больше RAM, специфическое железо).
- Compliance (данные не должны покидать периметр компании).
- Бесплатные минуты GitHub исчерпаны.

**Рекомендация:** Для open-source и стандартных задач — GitHub-hosted. Для enterprise с особными требованиями — self-hosted, но с redundancy и изоляцией.

</details>

---

### Вопрос 9: Параллельные workflow и required checks
**У вас два workflow: `ci.yml` (lint/test/build) и `chromatic.yml` (visual regression). В branch protection rules вы добавили `ci` как required check. Что произойдёт, если `chromatic.yml` упадёт, а `ci.yml` пройдёт?**

<details>
<summary>Показать ответ</summary>

PR можно будет смержить, потому что `chromatic` не добавлен в required checks. Только `ci` блокирует merge.

**Что делать:**
- Если Chromatic критичен — добавьте его в required checks.
- Если Chromatic в soft mode (`exitZeroOnChanges: true`) — он всегда зелёный, даже при diff. Добавлять в required checks имеет смысл только в strict mode.

**Подвох:** required checks ищут по **имени job**, а не имени файла. Если job в `chromatic.yml` называется `chromatic`, то в branch protection добавляется `chromatic`.

</details>

---

### Вопрос 10: Composite Action vs Reusable Workflow
**В чём разница между `composite` action и `reusable workflow`? Когда что использовать?**

<details>
<summary>Показать ответ</summary>

| | Composite Action | Reusable Workflow |
|---|---|---|
| **Что это** | Группа steps, объединённых в один «мета-шаг» | Целый workflow, который вызывается из другого |
| **Где живёт** | `.github/actions/my-action/action.yml` | `.github/workflows/reusable.yml` |
| **Вызывается через** | `uses: ./.github/actions/my-action` | `uses: ./.github/workflows/reusable.yml` |
| **Может содержать jobs** | ❌ Нет, только steps | ✅ Да, полноценный workflow |
| **Может использовать runs-on** | ❌ Нет | ✅ Да |
| **Когда использовать** | Повторяющаяся последовательность шагов (например, setup-node + install + cache) | Повторяющийся pipeline целиком |

**Пример Composite Action:**
```yaml
# .github/actions/setup-project/action.yml
name: Setup Project
runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: 22
        cache: npm
    - run: npm ci
      shell: bash
```

**Использование:**
```yaml
steps:
  - uses: actions/checkout@v4
  - uses: ./.github/actions/setup-project
  - run: npm run lint
```

**Composite** — для DRY внутри steps. **Reusable workflow** — для DRY на уровне целых pipeline.

</details>

---

## 🎖️ Заключение: Диплом Маршала

Если вы дошли до этого места — поздравляю. Вы теперь знаете:
- Как GitHub Actions оркестрирует pipeline от commit до deploy.
- Почему `npm ci`, а не `npm install`.
- Как Chromatic ловит визуальные регрессии до того, как они доедут до пользователя.
- Когда использовать soft mode, а когда strict.
- Как разделять workflow по доменам и почему `needs` — это ваш лучший друг.
- Что такое environments, artifacts, matrix, reusable workflows и composite actions.

**Ваше следующее задание:**
1. Создайте в своём проекте `ci.yml` с lint/test/build.
2. Добавьте `chromatic.yml` с TurboSnap и логикой для main/PR.
3. Настройте branch protection rules с required checks.
4. Пройдите этот курс ещё раз через неделю — забудете половину, это нормально.

> *«В поле зрения — вся Европа. Но сначала — green CI.»*
> — Вы, через 5 минут после merge

---

## 🔗 Связанные концепт-ноуты
- [[Chromatic — Визуальная регрессия и CI-CD flow]]
- [[GitHub Actions — Workflow файлы и CI-CD архитектура]]
- [[System-Design-Flashcards]] — флеш-карты для повторения

---

## ❓ Self-Check Questions (для Obsidian SR)

> [!faq]- Почему в CI используем `npm ci`, а не `npm install`?
> `npm ci` строго читает `package-lock.json` и не обновляет его. `npm install` может обновить версии зависимостей, нарушая воспроизводимость сборки.

> [!faq]- Зачем `needs:` в job'ах GitHub Actions?
> Создаёт зависимость между job'ами. Job с `needs: [lint, test]` не запустится, пока `lint` и `test` не завершатся успешно.

> [!faq]- В чём разница между `exitZeroOnChanges: true` и strict mode в Chromatic?
> Soft mode (`exitZeroOnChanges: true`) — Chromatic показывает diff, но GitHub check зелёный. Strict mode — GitHub check Pending до Accept в Chromatic UI.

> [!faq]- Что будет, если baseline Chromatic создан с багом?
> Chromatic считает баг нормой. Все последующие PR проходят зелёным, пока кто-то не пересоздаст baseline вручную.

> [!faq]- Зачем `fetch-depth: 0` для TurboSnap?
> TurboSnap анализирует git history, чтобы понять, какие stories изменились. Без полной истории Chromatic снимает все stories.
