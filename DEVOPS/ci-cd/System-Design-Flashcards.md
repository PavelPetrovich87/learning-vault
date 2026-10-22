<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards -->

# 🎴 System-Design-Flashcards


<!-- В GitHub Actions workflow: В чём разница между `npm ci` и `npm install` в GitHub Actions? :: `npm ci` строго читает `package-lock.json` и не обновляет его, гарантируя воспроизводимость. `npm install` может обновить версии зависимостей. В CI всегда используем `npm ci`. -->

<!-- #flashcards/github-actions/ci-cd/npm -->

<!-- В GitHub Actions workflow: Зачем нужен `needs:` в секции `jobs:` workflow? :: `needs` создаёт зависимость между job'ами. Job не запустится, пока указанные job'ы не завершатся успешно. Например, `needs: [lint, test]` гарантирует, что deploy произойдёт только после прохождения проверок. -->

<!-- #flashcards/github-actions/ci-cd/jobs -->

<!-- В GitHub Actions: В чём разница между GitHub `secrets` и `vars`? :: `secrets` — шифрованные, скрыты в логах, для токенов и паролей. `vars` — открытые переменные, видны в логах, для URL и имён. -->

<!-- #flashcards/github-actions/ci-cd/security -->

<!-- В GitHub Actions workflow: Зачем использовать `concurrency` с `cancel-in-progress: true`? :: Автоматически отменяет устаревшие запуски workflow при новом push'e. Экономит минуты CI, так как старые коммиты уже не актуальны. -->

<!-- #flashcards/github-actions/ci-cd/optimization -->

<!-- В GitHub Actions workflow: Что делает `strategy.matrix` в job? :: Запускает одну и ту же job несколько раз с разными параметрами. Например, `node-version: [18, 20, 22]` запустит job 3 раза для каждой версии Node.js. Полезно для тестирования совместимости. -->

<!-- #flashcards/github-actions/ci-cd/testing -->

<!-- В GitHub Actions + Chromatic: Почему `actions/checkout` без `fetch-depth: 0` ломает TurboSnap в Chromatic? :: TurboSnap анализирует git history, чтобы определить, какие stories изменились. По умолчанию `actions/checkout` клонирует только последний коммит (`fetch-depth: 1`). Без полной истории Chromatic не может сделать анализ и снимает все stories, тратя квоту. -->

<!-- #flashcards/github-actions/chromatic/turbo-snap -->

<!-- В GitHub Actions workflow: Как передать файлы из одной job в другую в GitHub Actions? :: Через artifacts: `actions/upload-artifact` в исходной job сохраняет файлы, `actions/download-artifact` в зависимой job загружает их. Для простых значений используются `outputs` в job. -->

<!-- #flashcards/github-actions/ci-cd/artifacts -->