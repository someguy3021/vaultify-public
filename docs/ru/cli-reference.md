# Справка CLI и MCP

> Русский · [English](../cli-reference.md)
>
> Часть публичной документации vaultify. Английская версия — источник истины;
> эта страница — её русская редакция.
>
> Страница повторяет собственный `help` CLI и `tools/list` MCP-сервера — ничего
> не выдумано. Вывод CLI — на русском по дизайну.

## Состав команд

```
vaultify init
vaultify where
vaultify deploy <dir>
vaultify vault add <путь> [--name N] [--git own|shared|none] [--layout strict|flex|single]
vaultify vault list | show <имя> [--json] | rm <имя> | map <имя> [--preset P] [--set тип=папка]...
vaultify scan [--vault имя] [--json]
vaultify search <запрос> [--vault имя] [--limit N] [--json]
vaultify status [--json]
vaultify sync [--dry]
vaultify task new "титул" [--kind работа|вопрос] [--urgency N] [--link путь] [--source S] [--body S]
vaultify task board [--json] | show VT-N | move VT-N <статус> [--note S]
vaultify task set VT-N [--title S] [--body S] [--urgency N] [--link P] [--kind K]
vaultify task rm VT-N --yes
vaultify memory save "текст" [--topic S] [--harness S]
vaultify memory handoff [--done S]... [--left S]... [--gotchas S]... [--next S] [--topic S] [--harness S]
vaultify memory resume [H-NNN] | close H-NNN | list
vaultify mcp
vaultify integrations install <zcode|dsh>
vaultify doctor
vaultify help
```

## Команды

### Настройка

- **`vaultify init`** — создать дом данных `<install>/home/` (env `VAULTIFY_HOME`
  переопределяет место): настройки, реестр vault'ов, `memory/`, git-репозиторий.
- **`vaultify where`** — показать папку развёртывания (артефакты) и дом данных.
- **`vaultify deploy <dir>`** — развернуть артефакты (`vaultify.exe`, ui-exe если
  собран) в одну папку; дом данных после init — `<dir>/home`. Переезд = перенести
  папку целиком + заново `integrations install` из нового места (пути в хуках
  абсолютные — сознательно).
- **`vaultify doctor`** — диагностика окружения: node, git, мета-репа, свежесть
  индекса, разводка MCP.

### Vault'ы

- **`vault add <путь>`** — зарегистрировать папку с markdown как vault. `--name`
  задаёт имя в реестре (по умолчанию выводится из имени папки); `--git` выбирает
  стратегию синка `own | shared | none` (по умолчанию — авто-детект); `--layout`
  выбирает пресет `strict | flex | single` (`flex` — дефолт, авто-детектит
  существующую структуру папок).
- **`vault list`** — реестр: имя, путь, git-режим, пресет layout'а.
- **`vault show <имя>`** — детали vault'а; с `--json` — полная запись вместе
  с layout-маппингом (её читает дистиллирующий скилл).
- **`vault rm <имя>`** — убрать vault из реестра (файлы не трогаются).
- **`vault map <имя>`** — пересмотреть маппинг «тип заметки → папка» руками:
  `--preset` меняет пресет, `--set тип=папка` задаёт отдельные маппинги.

### Индекс и поиск

- **`vaultify scan`** — перестроить `index.db` (производный, перестраиваемый
  индекс, вне git) по всем зарегистрированным vault'ам; `--vault` ограничивает
  скан одним vault'ом.
- **`vaultify search <запрос>`** — префиксный полнотекстовый-подобный поиск по
  заголовку, тегам и телу всех vault'ов, плюс задачи и память; возвращает путь,
  заголовок, сниппет. `--vault` сужает область, `--limit` режет число результатов.

### Статус и синк

- **`vaultify status`** — сводные: vault'ы (git-состояние: грязные,
  впереди/позади), доска задач, счётчики памяти.
- **`vaultify sync`** — commit + pull --rebase + push для каждого git-хранилища
  (каждый vault по своей стратегии `own/shared/none` и мета-репа); `--dry`
  показывает, что произошло бы, ничего не меняя.

### Доска задач (канбан)

Статусы: `inbox / doing / waiting / someday / done`. Идентификаторы задач —
`VT-1`, `VT-2`, …

- **`task new "титул"`** — создать задачу в инбоксе. `--kind` — вид задачи:
  `работа` или `вопрос`; `--urgency` — приоритет P0–P3; `--link` прикладывает
  путь к заметке/vault'у; `--source` фиксирует, откуда задача; `--body` — текст.
- **`task board`** — отрисовать канбан-доску.
- **`task show VT-N`** — карточка задачи с историей переходов.
- **`task move VT-N <статус>`** — перевести задачу по канбану; `--note`
  записывает причину перехода (ожидается для `waiting`).
- **`task set VT-N`** — правка полей: `--title`, `--body`, `--urgency`,
  `--link`, `--kind`.
- **`task rm VT-N --yes`** — удалить задачу (флаг подтверждает).

### Память

Сырой слой лежит в мета-репе markdown'ом: `memory/sessions/S-NNNN.md`
(неизменяемые итоги сессий) и `memory/handoffs/H-NNNN.md`
(структурированные handoff'ы).

- **`memory save "текст"`** — записать сырую сессию: ценный вывод или решение.
  `--topic` называет тему/проект, `--harness` фиксирует, какой харнесс писал
  (zcode, claude-code, …).
- **`memory handoff`** — структурированный handoff следующей сессии или
  следующему харнессу: `--done` (повторяемый), `--left` (повторяемый),
  `--gotchas` (повторяемый), `--next` (с чего продолжить), плюс `--topic`
  и `--harness`.
- **`memory resume [H-NNN]`** — подобрать handoff: последний открытый или
  конкретный `H-NNN` по id.
- **`memory close H-NNN`** — пометить handoff закрытым.
- **`memory list`** — записанные сессии и handoff'ы.

### Агенты

- **`vaultify mcp`** — поднять MCP-сервер (stdio JSON-RPC); тот же бинарь
  в режиме MCP. Список инструментов — ниже.
- **`vaultify integrations install <zcode|dsh>`** — установить интеграцию
  в харнесс (см. ниже).
- **`vaultify help`** — встроенная справка (на русском).

## Интеграции с харнессом

`vaultify integrations install zcode` (или `dsh`) ставит из канонических
шаблонов с подстановкой машинных токенов:

- **скилл** — `vaultify` (этикет двух слоёв памяти, дистилляция по layout-маппингу,
  работа с поиском и доской);
- **слэш-команды** (таргет zcode) — `/vt-today`, `/vt-remember`, `/vt-distill`;
- **SessionStart-хук** — подаёт в контекст сессии доску, свежайшую память
  и грязные vault'и; путь к CLI прошивается дефолтом хука;
- **мерж конфига** (таргет zcode) — хук добавляется в группу `SessionStart`,
  MCP-сервер регистрируется; перед этим пишется бэкап конфига харнесса
  (`config.json.bak-vaultify`), чужие записи не трогаются;
- **файл-маркер** (`vaultify.json`) в каталоге харнесса — по нему `doctor`
  и переустановка находят разводку.

Пути можно перенаправить для тестов и нестандартных сетапов: `VAULTIFY_HOME`
(мета-репа), у установщика — `VAULTIFY_USER_HOME` (дом пользователя) и
`VAULTIFY_ZCODE` / `VAULTIFY_DSH` (каталоги харнессов).

`vaultify doctor` проверяет результат: node, git, мета-репа, маркер,
регистрация MCP.

## Инструменты MCP (8)

MCP-сервер (`vaultify mcp`) отдаёт восемь инструментов поверх stdio JSON-RPC:

| Инструмент | Что делает |
|---|---|
| `vault_search` | Полнотекстовый-подобный поиск по всем подключённым Obsidian vault'ам (заметки); вернёт путь, заголовок, сниппет. Параметры: `query` (обязателен), `vault` (один vault), `limit` (по умолчанию 5) |
| `memory_save` | Сохранить ценный вывод/решение текущей сессии в сырой слой памяти; пиши сжатые выводы, не лог. Параметры: `text` (обязателен), `topic`, `harness` |
| `handoff` | Записать handoff: что сделано, что осталось, грабли, с чего продолжить; другой харнесс подберёт через `resume`. Параметры: `done`, `left`, `gotchas` (массивы строк), `next` (обязателен), `topic`, `harness` |
| `resume` | Подобрать последний открытый handoff (или конкретный `H-NNN`) и вернуть его текст. Параметр: `id` (необязателен) |
| `task_list` | Канбан-доска: все открытые задачи плюс последние закрытые |
| `task_add` | Создать задачу (инбокс). Параметры: `title` (обязателен), `kind` (`работа`/`вопрос`), `urgency`, `link`, `source` |
| `task_move` | Перевести задачу по канбану: `inbox\|doing\|waiting\|someday\|done`. Параметры: `id`, `status` (обязательны), `note` (причина, для `waiting`) |
| `vault_status` | Сводка: git-состояние vault'ов и мета-репы, счётчики доски и памяти |
