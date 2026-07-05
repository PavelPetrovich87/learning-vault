---
status: implemented
tags: [research, obsidian, debugging, electron, playbook]
created: 2026-07-05
updated: 2026-07-05
source:
  title: "Incident: Obsidian hangs on 'Loading cache' (2026-07-05)"
  author: self
  url:
  date_read: 2026-07-05
---

# Debugging Obsidian Vault Hang — Playbook & Lessons

> [!abstract] TL;DR
> Obsidian перестал загружать vault, висел на splash "Loading cache" с рендерером на 100%+ CPU. Дибажил час, потратил время на TCC/плагины/`_reference/`, но реальный диагноз — **комбо stale cache после auto-update + vault без ignore-фильтров**. Главный урок: подключись к Chrome DevTools Protocol **первым делом**, это за 30 секунд покажет, жив ли main thread.

---

## 🔬 Фаза 1: EXTRACTION — Симптомы и факты

### Что видел
- Процесс Obsidian живёт (`state=S`), но окна/файлов волта в `lsof` не видно.
- Renderer (Helper) жрёт **100%+ CPU**, main thread сидит в `mach_msg_trap` (event loop).
- `~/Library/Application Support/obsidian/obsidian.log` пишется, но **ни одной строки про vault load**.
- Splash "Loading cache" висит минуты.

### Неочевидные сигналы, которые я прочитал неправильно
- **`lsof | grep my-learning-vault` = 0** → я решил "vault не открылся". **На самом деле** Obsidian читает файлы через worker'ы и metadata-cache, main process их в `lsof` почти не показывает. Это норма, не симптом.
- **`sample <pid>` показывает main thread в `-[NSApplication run]`** → я решил "deadlock". **На самом деле** это просто idle event loop Electron'а. Renderer отдельно может пахать.
- **TCC-логи (`TCCAccessRequest`)** → я решил "модальный privacy-dialog блокирует". **На самом деле** Obsidian делает эти запросы routinely (AppleEvents, WindowServer), они не блокирующие.

### Novelty
`lsof`-проверка "открыт ли vault" — **не работает** для Electron-приложений. Main process = GUI shell, файлами занимается worker. Это развеяло главный ложный сигнал.

---

## 🧠 Фаза 2: ANALYSIS — Причины и не-причины

### Не-причины (что я зря проверял)
| Гипотеза | Почему была | Верификация | Вердикт |
|---|---|---|---|
| TCC / Privacy dialog | Видел `TCCAccessRequest` в логе | Выдали Full Disk Access — ничего не изменилось | ❌ Шум |
| Сломанный плагин SR | `obsidian-spaced-repetition` парсит md | Полностью снёс `.obsidian` — те же симптомы | ❌ Не оно |
| Кривой `workspace.json` | Подозрение на зависший leaf | JSON валиден, reset не помог | ❌ Не оно |
| `_reference/` (54k файлов) | Раздувает индексацию | Тестовый vault с 1 файлом тоже зависал | ⚠️ Усугубляло, но не первопричина |

### Что реально сломалось
Комбинация трёх факторов, точно разделить вклад каждого не удалось (переустановка совпала с чисткой кэшей):

1. **Auto-update рассинхрон.** Bundle в `/Applications/` стоял `1.12.4` (Feb 27), а auto-updater 4 июля скачал `obsidian-1.12.7.asar` в `~/Library/Application Support/obsidian/`. Старый binary грузил новый JS-код — возможна несовместимость.
2. **Stale Electron caches.** `GPUCache/`, `DawnGraphiteCache/`, `Code Cache/` остались от прошлой версии. Новый asar при старте их пережёвывал.
3. **Vault без `userIgnoreFilters`.** 54 156 файлов (из них 20 874 в `_reference/` + 29 765 в `node_modules`) вместо 360 реальных заметок. "Loading cache" на таком объёме занимает минуты и выглядит как зависание.

### Lessons — куда смотреть в следующий раз

> [!important] Главный урок
> **Первым делом подключай DevTools Protocol**, не `lsof`/`sample`. 30 секунд — и точно знаешь, жив ли JS main thread и на чём он.

---

## 🛠 Фаза 3: APPLICATION — Debug playbook

### Шаг 0 (30 сек): DevTools Protocol — главная проверка

```bash
# Запустить с remote debugging
/Applications/Obsidian.app/Contents/MacOS/Obsidian --remote-debugging-port=9222 &

# Получить список таргетов (page + workers)
curl -s http://localhost:9222/json | python3 -m json.tool
```

Если в ответе есть `type=page` с `title="...vault-name - Obsidian X.Y.Z"` — **vault загружен**, main thread жив. Дальше можно дёргать `Runtime.evaluate` через WS и смотреть реальное состояние.

Если page-таргета нет или title пустой — vault реально не открылся. Тогда уже смотри конфиги.

### Шаг 1 (1 мин): Проверить, что это не версия
```bash
/usr/libexec/PlistBuddy -c "Print :CFBundleShortVersionString" /Applications/Obsidian.app/Contents/Info.plist
ls -la "$HOME/Library/Application Support/obsidian/"*.asar
```
Если bundle ≠ asar — авто-апдейт оставил рассинхрон. Переустановить с [obsidian.md/download](https://obsidian.md/download) (GitHub releases: `https://github.com/obsidianmd/obsidian-releases/releases/download/vX.Y.Z/Obsidian-X.Y.Z.dmg`, это universal).

### Шаг 2 (1 мин): Чистка stale Electron caches
```bash
pkill -f Obsidian
rm -rf "$HOME/Library/Application Support/obsidian/"{Cache,Code\ Cache,GPUCache,DawnGraphiteCache,DawnWebGPUCache}
```
**Не трогать**: `IndexedDB/`, `Local Storage/`, `obsidian.json` (vault registry), `*.asar`.

### Шаг 3 (2 мин): Ignore-фильтры для раздутого vault
Если в vault лежат клонированные репы / `node_modules` — Obsidian индексирует каждый файл. Положить в `.obsidian/app.json`:
```json
{
  "userIgnoreFilters": [
    "_reference/",
    "node_modules/",
    ".kilocode/",
    "CC_leaked/"
  ]
}
```
Это убавило мой vault с **54 156 → 360** файлов для индексации (в 150 раз).

### Шаг 4 (только если 0-3 не помогли): Safe-mode проверки
- Полностью снести `.obsidian` (mv в `.obsidian.disabled`) → если висит и с fresh config, дело **не** в плагинах/workspace.
- Создать тестовый vault **вне** `~/Documents` (например `~/obsidian-test`) → если тоже висит, дело в app, не в vault.

### Что НЕ делать (мои ошибки)
- ❌ Не трать время на TCC/Privacy, пока не проверишь через DevTools. TCC-логи в macOS unified log шумные и routinely-нормальные для Electron.
- ❌ Не делай вывод по `lsof | grep vault` — main process не держит файлы волта.
- ❌ Не удаляй `.obsidian/` целиком "на всякий случай" — потеряешь workspace, history плагинов, SR schedule. Сначала mv в backup.

---

## ✅ Фаза 4: ACTION — Где живёт решение

### Что осталось в vault после инцидента
- `.obsidian/app.json` — добавлен `userIgnoreFilters` (полезный побочный эффект).
- Старый bundle удалён ( был `/Applications/Obsidian.app.old-1.12.4`).
- `.obsidian/` полностью восстановлен из backup.

### Open Questions
- [ ] Воспроизвести на чистой машине: bundle 1.12.4 + asar 1.12.7 — реально ломает загрузку, или совпадение?
- [ ] Настроить pin версии auto-updater'а, чтобы не ловить рассинхрон после апдейтов.

### Related Notes
- [[AGENTS]] — vault conventions (упоминает `_reference/` как excluded zone, но это для AI-агентов, не для Obsidian)

### Status History
- `2026-07-05`: researching →analyzing→implemented — инцидент случился и был разрешён в той же сессии. Плейбук зафиксирован.

---

## 🗣️ Разговор с собой

Главная ошибка дибага — я **верил инструментам, которые не понимал**. `lsof`, `sample`, macOS unified log — каждый из них что-то "показывал", и я лепил гипотезы на основе этих показаний, не проверив, что именно они измеряют в контексте Electron-приложения. В итоге потратил час на TCC, плагины, `workspace.json`, когда 30 секунд с DevTools'ом всё расставили бы по местам.

Второй урок — **корреляция ≠ причинность**. Уборка vault совпала по времени с поломкой (обе в начале июля), и я даже написал пользователю "скорее всего, дело в уборке". Надо было сразу: тестовый vault вне Documents → если тоже висит, значит не vault, а app. Это отделило бы "что чинить" от "что виновато" за 1 минуту.

И третье — **не навреди**. В дибаге я: mv-нул `.obsidian`, удалил asar, reset workspace. Каждое действие требовало потом отката. Лучше одно изменение за раз и всегда с backup.
