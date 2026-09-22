# OpenCode Project Template

Полное руководство по `.opencode/` — локальному knowledge bundle для
работы с AI-агентом через [OpenCode](https://opencode.ai).

---

## Содержание

- [Что это и зачем](#что-это-и-зачем)
- [Ключевые концепции](#ключевые-концепции)
- [Как это работает](#как-это-работает)
- [Структура bundle](#структура-bundle)
- [Файлы шаблона](#файлы-шаблона)
- [Динамические артефакты](#динамические-артефакты)
- [Как файлы связаны](#как-файлы-связаны)
- [OKF и этот шаблон](#okf-и-этот-шаблон)
- [Словарь `type`](#словарь-type)
- [Использование](#использование)
- [Сценарии работы](#сценарии-работы)
- [init-opencode](#init-opencode)
- [Предостережения](#предостережения)
- [FAQ](#faq)
- [Философия](#философия)

---

## Что это и зачем

Когда AI-агент работает с проектом через OpenCode, он читает `AGENTS.md`.
Проблема в том, что разработчики кладут туда **всё подряд** — архитектуру,
code style, CI-таблицы, списки файлов, историю решений. Файл раздувается
до 500–600 строк, из которых в каждый момент нужны 50. Остальное — шум,
который жрёт токены и отвлекает агента.

Этот шаблон решает проблему через **ленивую загрузку**:

- `AGENTS.md` — только ядро: что за проект, как собрать, ключевые
  компоненты, куда идти дальше.
- Всё детальное — в тематических файлах (`_*.md`), которые читаются
  только когда задача требует.
- Динамические артефакты (issue, playbook, PR, analysis, runbooks) —
  в отдельных директориях.

**Ключевой принцип:** `AGENTS.md` — не энциклопедия, а оглавление.
Энциклопедия — в `_*.md` файлах.

---

## Ключевые концепции

### Bundle

**Bundle** — термин из OKF. Это **директория с markdown-концептами**,
организованная по определённым правилам: frontmatter, cross-linking,
reserved names.

В этом шаблоне bundle — это `.opencode/` в корне проекта.

```
my-project/
├── .opencode/          ← ← ← bundle
│   ├── AGENTS.md
│   ├── _concepts.md
│   └── ...
├── src/
└── README.md
```

Bundle — **локальный**, никогда не коммитится в репозиторий проекта.
Он — ваша личная база знаний и рабочий инструмент.

### Lazy loading

Файлы `_*.md` **не загружаются автоматически**. Агент читает их только
когда решает, что задача соответствует описанию в `AGENTS.md`.

Пример:
- Вы говорите: «CI упал, разберись».
- Агент видит в `AGENTS.md`: `_ci.md — when CI fails`.
- Читает `_ci.md`, диагностирует.

Это экономит контекст: агент не тратит токены на файлы, которые сейчас
не нужны.

### OKF

**Open Knowledge Format** — открытый формат от Google Cloud для
представления знаний в виде markdown-файлов с YAML frontmatter.

Каждый файл — **concept document**. У него две части:

1. **Frontmatter** — метаданные (обязательное поле `type`).
2. **Body** — markdown с содержимым.

Bundle следует OKF v0.1 **с расширениями** (см. раздел
[OKF и этот шаблон](#okf-и-этот-шаблон)).

### Lazy vs eager

| Eager (всё сразу) | Lazy (по запросу) |
|-------------------|-------------------|
| Один большой AGENTS.md | Ядро + тематические файлы |
| 500+ строк всегда в контексте | ~60 строк + файлы по запросу |
| Агент тонет в шуме | Агент фокусируется |

---

## Как это работает

### Иерархия AGENTS.md

OpenCode при старте сессии собирает `AGENTS.md` со всех уровней вверх по
дереву директорий и склеивает в один контекст:

```
~/.config/opencode/AGENTS.md                  ← глобальные правила
~/Projects/<org>/.opencode/AGENTS.md          ← правила организации
~/Projects/<org>/<repo>/.opencode/AGENTS.md   ← правила репозитория
```

Ничего указывать вручную не нужно — OpenCode делает это сам.

### Как читается bundle

1. Открывается `AGENTS.md` — точка входа.
2. Агент получает минимальный контекст: что за проект, стек, build.
3. По таблице reference files агент решает, что читать дальше.
4. Читает нужные `_*.md` файлы.
5. Работает.

### Роль человека

Вы **не управляете загрузкой файлов вручную**. Вы формулируете задачу
естественным языком. Агент сам решает, какие файлы ему нужны.

Примеры:

| Что вы говорите | Что читает агент |
|-----------------|------------------|
| «Добавь новый компонент» | `_concepts.md`, `_codestyle.md`, `_files.md` |
| «CI красный» | `_ci.md`, `_troubleshooting.md` |
| «Работаем над issue #123» | `_templates.md`, `_concepts.md` |
| «Нужен глубокий анализ» | `_concepts.md`, создаёт `analysis/*` |

---

## Структура bundle

Полная структура `.opencode/`:

```
.opencode/
├── .gitignore                    ← защита от коммита
├── .template-version             ← версия шаблона (machine-readable)
│
├── index.md                      ← OKF entry point
├── log.md                        ← OKF log pointer
├── AGENTS.md                     ← главный файл
├── SPEC_REFERENCE.md             ← выдержка из OKF v0.1
│
├── _concepts.md                  ← архитектура
├── _setup.md                     ← локальный запуск
├── _env.md                       ← карта окружений
├── _codestyle.md                 ← стиль и конвенции
├── _commands.md                  ← шпаргалка команд
├── _files.md                     ← карта файлов
├── _glossary.md                  ← доменные термины
├── _security.md                  ← работа с секретами
├── _troubleshooting.md           ← локальные проблемы
├── _decisions.md                 ← ADR-журнал
├── _backlog.md                   ← будущие задачи
├── _worklog.md                   ← шаблон WORK_LOG
├── _templates.md                 ← шаблоны issue/PR
├── _ci.md                        ← CI-workflow и ошибки
├── _meta.md                      ← мета о bundle
│
├── WORK_LOG.md                   ← создаётся при первой сессии
│
├── issue/                        ← активные PROJECT_SUMMARY
├── playbook/                     ← активные PLAYBOOK
├── pr/                           ← черновики PR-описаний
├── analysis/                     ← находки анализа
│   ├── index.md
│   └── _finding.md
├── runbooks/                     ← процедуры инцидентов
│   ├── index.md
│   └── _runbook.md
└── archive/                      ← завершённые issue/playbook/pr
```

---

## Файлы шаблона

### Ядро

| Файл | Роль | Длина |
|------|------|-------|
| `AGENTS.md` | Точка входа. Контекст проекта + навигация | 60–90 строк |
| `index.md` | OKF-индекс bundle | ~35 строк |
| `log.md` | Указатель на хронологические логи | ~20 строк |

### Onboarding — понимание проекта

| Файл | Когда читать |
|------|--------------|
| `_setup.md` | Первый запуск с нуля |
| `_concepts.md` | Архитектура, паттерны, data flow |
| `_glossary.md` | Незнакомый доменный термин |

### Daily work — ежедневные задачи

| Файл | Когда читать |
|------|--------------|
| `_templates.md` | Новая issue — SUMMARY, PLAYBOOK, PR |
| `_worklog.md` | Начало/продолжение сессии |
| `_backlog.md` | Планирование, идеи, техдолг |
| `_decisions.md` | «Почему так сделано» — ADR |
| `_codestyle.md` | Пишешь код — SPDX, lint, конвенции |
| `_commands.md` | Нужна команда — build/test/run |

### When things break — диагностика

| Файл | Когда читать |
|------|--------------|
| `_ci.md` | CI упал |
| `_troubleshooting.md` | Локальное окружение сломано |
| `runbooks/` | Инцидент в prod |

### Navigation & safety — навигация и безопасность

| Файл | Когда читать |
|------|--------------|
| `_files.md` | Ищешь, где что лежит |
| `_env.md` | Карта окружений — dev, staging, prod |
| `_security.md` | Секреты, уязвимости |
| `analysis/` | Результаты глубокого анализа |
| `_meta.md` | Как устроен сам bundle |

### Служебные

| Файл | Роль |
|------|------|
| `SPEC_REFERENCE.md` | Выдержка из OKF v0.1 |
| `.gitignore` | Защита от коммита |
| `.template-version` | Версия шаблона |

---

## Динамические артефакты

Эти директории **наполняются по мере работы**. Их содержимое —
уникально для каждого проекта и каждой issue.

### `issue/`

Активные `PROJECT_SUMMARY_<N>.md` — фиксация текущих issue.

Создаются в начале работы над issue. После мержа PR — перемещаются
в `archive/`.

### `playbook/`

Активные `PLAYBOOK_<N>.md` — стратегия решения issue.

Создаются, когда решение неочевидно. Могут отсутствовать для простых
задач.

### `pr/`

Черновики `PR_<N>.md` — описания pull request.

Создаются перед открытием PR. После мержа — в `archive/`.

### `analysis/`

Находки глубокого анализа: аудит, performance, техдолг.

- `index.md` — сводная таблица.
- `_finding.md` — шаблон одной находки.
- `F-001-xxx.md`, `F-002-xxx.md` — конкретные findings.

Findings **не являются задачами** — это отчёты о состоянии. Чтобы
действовать — создайте задачу в `_backlog.md`.

### `runbooks/`

Процедуры реагирования на инциденты в prod.

- `index.md` — индекс всех runbook'ов.
- `_runbook.md` — шаблон.
- `db-failover.md`, `rollback-release.md` — конкретные процедуры.

Читаются **под давлением**. Формат: команда + Expected + «if it doesn't
work».

### `archive/`

Завершённые issue/playbook/pr. Перемещаются после мержа PR.

Может быть организован по датам или по номерам issue.

---

## Как файлы связаны

```
                    AGENTS.md
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
    Onboarding      Daily work     When things break
        │               │               │
        ├─_setup        ├─_templates    ├─_ci
        ├─_concepts     ├─_worklog      ├─_troubleshooting
        └─_glossary     ├─_backlog      └─runbooks/
                        ├─_decisions
                        ├─_codestyle
                        └─_commands
                        
                        Navigation & safety
                        │
                        ├─_files
                        ├─_env
                        ├─_security
                        ├─analysis/
                        └─_meta
```

Ключевые связи между файлами:

```
_setup.md ─────────→ _env.md          (non-local envs)
_setup.md ─────────→ _security.md     (secrets rules)
_setup.md ─────────→ _troubleshooting.md (if verify fails)

_codestyle.md ─────→ _commands.md     (commands)
_codestyle.md ─────→ _concepts.md     (testing strategy)

_ci.md ────────────→ _troubleshooting.md (local issues)
_ci.md ────────────→ runbooks/         (prod incidents)

_worklog.md ───────→ _decisions.md     (significant decisions)
_worklog.md ───────→ _backlog.md       (Next → tasks)

_templates.md ─────→ issue/            (PROJECT_SUMMARY)
_templates.md ─────→ playbook/         (PLAYBOOK)
_templates.md ─────→ pr/               (PR description)

analysis/ ─────────→ _backlog.md       (finding → task)
_decisions.md ─────→ _backlog.md       (Follow-up → task)
```

**Правило «link, don't duplicate»:** если что-то дублируется в другом
файле — ставьте ссылку, а не копию.

---

## OKF и этот шаблон

### Что такое OKF

**Open Knowledge Format** — открытый формат от Google Cloud для
представления знаний. Минимализм: нет центрального реестра схем, нет
обязательного tooling.

Полная спецификация:
[SPEC.md](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md).
Локальная выдержка: [`SPEC_REFERENCE.md`](SPEC_REFERENCE.md).

### Что мы взяли из OKF

| Концепция OKF | В нашем шаблоне |
|---------------|-----------------|
| Bundle | Вся `.opencode/` директория |
| Concept document | Каждый `_*.md` файл |
| YAML frontmatter | Блок `---` с `type`, `title`, `description` |
| `type` (REQUIRED) | `project-context`, `architecture`, `ci`, ... |
| Cross-linking | Относительные markdown-ссылки между файлами |
| Citations | `## References` / `## Citations` |
| `index.md` | Наш `index.md` — OKF entry point |
| `log.md` | Наш `log.md` — указатель на логи |

### Что мы добавили поверх OKF

| Расширение | Зачем |
|-----------|-------|
| `AGENTS.md` как entry point | OpenCode читает именно его при старте |
| Префикс `_*.md` | Визуально маркирует reference files |
| Ленивая загрузка | Экономия контекста агента |
| `WORK_LOG.md` | Локальная хронология (вместо OKF `log.md`) |
| Директории `issue/`, `playbook/`, `pr/` | Динамические артефакты workflow |
| `analysis/` | Структурированный вывод анализа |
| `runbooks/` | Процедуры инцидентов |
| Расширенный словарь `type` | Специфичные для наших задач типы |

### Толерантность OKF

OKF требует, чтобы потребители **не отвергали** bundle из-за:

- Отсутствия опциональных полей frontmatter.
- Неизвестных значений `type`.
- Неизвестных дополнительных ключей frontmatter.
- Битых cross-ссылок.
- Отсутствующих `index.md`.

Мы следуем этому принципу: bundle остаётся полезным по мере роста,
рефакторинга и частичной генерации агентами.

---

## Словарь `type`

OKF не регистрирует `type` централизованно, но для консистентности
внутри одного проекта полезно придерживаться фиксированного набора.

| `type` | Файл |
|--------|------|
| `project-context` | `AGENTS.md` |
| `index` | `index.md` |
| `log` | `log.md` |
| `meta` | `_meta.md` |
| `spec-reference` | `SPEC_REFERENCE.md` |
| `architecture` | `_concepts.md` |
| `setup` | `_setup.md` |
| `env` | `_env.md` |
| `codestyle` | `_codestyle.md` |
| `commands` | `_commands.md` |
| `files` | `_files.md` |
| `glossary` | `_glossary.md` |
| `security` | `_security.md` |
| `troubleshooting` | `_troubleshooting.md` |
| `ci` | `_ci.md` |
| `decision-log` | `_decisions.md` |
| `backlog` | `_backlog.md` |
| `worklog` | `_worklog.md`, `WORK_LOG.md` |
| `templates` | `_templates.md` |
| `analysis-index` | `analysis/index.md` |
| `finding` | `analysis/F-*.md` |
| `runbook-index` | `runbooks/index.md` |
| `runbook` | `runbooks/*.md` |
| `project-summary` | `issue/PROJECT_SUMMARY_<N>.md` |
| `playbook` | `playbook/PLAYBOOK_<N>.md` |
| `pr` | `pr/PR_<N>.md` |

Потребители (в т.ч. агент) должны **толерантно** относиться к
неизвестным `type` — это требование OKF. Но производители (мы)
стараются не плодить новые без нужды.

---

## Использование

### Быстрый старт

```bash
# Установка шаблона в новый проект
init-opencode ~/Projects/my-app

# Установка в существующий проект
init-opencode --analyze ~/Projects/existing-repo
```

### Что делать после инициализации

**Новый проект:** скажите агенту — «начинаем новый проект, помоги
заполнить шаблон». Агент задаст вопросы и заполнит файлы.

**Существующий проект:** скажите — «проанализируй репозиторий и
заполни шаблон». Агент изучит код, CI, структуру и заполнит все
шаблоны.

### Переменные окружения

| Переменная | По умолчанию | Назначение |
|------------|--------------|------------|
| `OPENCODE_TEMPLATE_REPO` | `~/Projects/opencode-templates` | Путь к репозиторию шаблонов |

Переопределите, если клон шаблона лежит в другом месте:

```bash
OPENCODE_TEMPLATE_REPO=~/work/opencode-templates \
  init-opencode ~/Projects/my-app
```

### Файл версии

В корне репозитория шаблонов лежит файл `VERSION` (например,
`v0.1.0`). `init-opencode` читает его и пишет
`.opencode/.template-version` с метаданными:

```
version: v0.1.0
installed: 2026-09-21
source: /home/user/Projects/opencode-templates
```

`--diff` и `--update` используют этот файл для обнаружения расхождений.

### Что перезаписывается

`--update` трогает только файлы из списка «always overwritten».
Полные списки:

**Никогда не перезаписываются** (ваши данные):

- `WORK_LOG.md`
- `_concepts.md`
- `_setup.md`
- `_decisions.md`
- `_backlog.md`
- `_meta.md`
- Всё содержимое `analysis/`, `runbooks/`, `issue/`, `playbook/`,
  `pr/`, `archive/`

**Всегда перезаписываются** (шаблонные):

- `AGENTS.md`, `index.md`, `log.md`, `SPEC_REFERENCE.md`
- `_codestyle.md`, `_ci.md`, `_commands.md`, `_files.md`,
  `_glossary.md`, `_security.md`, `_troubleshooting.md`,
  `_templates.md`, `_env.md`, `_worklog.md`

Если вы кастомизировали файл из «always overwritten» — скопируйте
его под другим именем (например, `_codestyle.local.md`) или перенесите
в список «never» в скрипте.

### Проверка установки

```bash
# Проверить, что скрипт доступен
which init-opencode

# Пробный прогон в тестовой директории
mkdir -p /tmp/test-project
init-opencode /tmp/test-project
ls -la /tmp/test-project/.opencode/

# Сразу после установки update должен ничего не менять
init-opencode --update /tmp/test-project
# → nothing to update
```
### Обновление

```bash
# Посмотреть, что изменится
init-opencode --diff ~/Projects/my-app

# Применить обновление
init-opencode --update ~/Projects/my-app
```

**Never overwritten:** `WORK_LOG.md`, `_decisions.md`, `_backlog.md`,
`_concepts.md`, `_setup.md`, `analysis/*`, `runbooks/*`, `issue/*`,
`playbook/*`, `pr/*`, `archive/*`.

**Always overwritten:** `_codestyle.md`, `_ci.md`, `_commands.md`,
`_files.md`, `_glossary.md`, `_security.md`, `_troubleshooting.md`,
`_templates.md`, `AGENTS.md`, `index.md`, `log.md`, `_meta.md`.

---

## Сценарии работы

### Сценарий 1: новый проект

```
1. init-opencode ~/Projects/my-app
2. Открываете OpenCode в my-app/
3. Говорите: "начинаем новый проект, помоги заполнить шаблон"
4. Агент задаёт вопросы, заполняет файлы
5. Вы уточняете
6. Агент фиксирует результат
```

### Сценарий 2: существующий проект

```
1. init-opencode --analyze ~/Projects/existing-repo
2. Открываете OpenCode
3. Говорите: "проанализируй репозиторий и заполни шаблон"
4. Агент изучает структуру, код, CI
5. Заполняет все файлы
6. Опционально: создаёт analysis/ с находками
```

### Сценарий 3: работа над issue

```
1. Говорите: "работаем над issue #123"
2. Агент создаёт issue/PROJECT_SUMMARY_123.md и playbook/PLAYBOOK_123.md
3. Читает _concepts.md (понимает архитектуру)
4. Вы обсуждаете подход
5. Агент пишет код, сверяясь с _codestyle.md
6. Запускает команды из _commands.md
7. Создаёт pr/PR_123.md
8. После мержа — перемещает всё в archive/
```

### Сценарий 4: CI упал

```
1. Говорите: "CI красный, разберись"
2. Агент читает _ci.md
3. Определяет упавший workflow по логам
4. Находит причину в таблице Common failures
5. Обсуждаете фикс
6. Если проблема новая — добавляет строку в _ci.md
```

### Сценарий 5: локальное окружение сломалось

```
1. Говорите: "тесты не запускаются, ошибка X"
2. Агент читает _troubleshooting.md
3. Ищет симптом по Quick index
4. Применяет фикс
5. Если решения нет — гуглит, решает, добавляет запись
```

### Сценарий 6: инцидент в prod

```
1. Алерт сработал
2. Открываете runbooks/index.md
3. Находите подходящий runbook
4. Следуете шагам
5. Если runbook не сработал — эскалация
6. Post-incident: обновление runbook, _decisions.md, _backlog.md
```

### Сценарий 7: глубокий анализ

```
1. Говорите: "нужен аудит безопасности"
2. Агент читает _concepts.md, _security.md
3. Анализирует код
4. Создаёт analysis/F-001-xxx.md, F-002-xxx.md
5. Обновляет analysis/index.md
6. Вы решаете, что чинить
7. Переносите в _backlog.md
```

### Сценарий 8: планирование

```
1. Говорите: "что дальше делать?"
2. Агент читает _backlog.md, WORK_LOG.md
3. Показывает приоритеты P0/P1
4. Обсуждаете
5. Берёте задачу — начинается Сценарий 3
```

---

## init-opencode

Скрипт `~/.local/bin/init-opencode` копирует шаблон в целевой проект.

### Использование

```bash
# Новый проект
init-opencode ~/Projects/my-app

# Существующий проект
init-opencode --analyze ~/Projects/existing-repo

# Обновление
init-opencode --update ~/Projects/my-app

# Preview изменений
init-opencode --diff ~/Projects/my-app

# Справка
init-opencode --help
```

### Что делает

**Установка:**
1. Создаёт `.opencode/` в целевой директории.
2. Если `.opencode/` уже существует — бекапит в `.opencode.bak.<timestamp>`.
3. Копирует `template/` содержимое.
4. Создаёт пустые директории (`issue/`, `playbook/`, `pr/`, `archive/`).
5. Создаёт `.template-version`.
6. Выводит следующие шаги.

**Обновление:**
1. Читает `.template-version`.
2. Сравнивает с текущей версией шаблона.
3. Обновляет «always overwritten» файлы.
4. Оставляет нетронутыми «never overwritten».
5. Обновляет `.template-version`.

### Установка скрипта

Скрипт уже должен быть в `~/.local/bin/`. Если нет — скачайте из
репозитория шаблонов:

```bash
curl -fsSL <repo-url>/raw/main/bin/init-opencode \
  -o ~/.local/bin/init-opencode
chmod +x ~/.local/bin/init-opencode
```

---

## Предостережения

### `.opencode/` никогда не коммитится

- Добавлен в `.git/info/exclude` (локально для вашего клона) или в
  `.gitignore` (для всех).
- Внутри самого `.opencode/` есть `.gitignore` — вторая линия обороны.
- Рекомендуется `.git/info/exclude` — не заражает репозиторий.

### `git clean -dfX` удаляет `.opencode/`

Всегда используйте:

```bash
git clean -dfX -e .opencode/
```

Или добавьте `.opencode/` в `.gitignore`.

### `make clean` может удалить `.opencode/`

Перед `make clean` сделайте бэкап или добавьте `-e .opencode/`.

### Тематические файлы не читаются автоматически

Агент читает их только когда задача соответствует описанию в
`AGENTS.md`. Если нужна архитектура — скажите «расскажи про
архитектуру».

### `WORK_LOG.md` — локальный

Не синхронизируется. Если нужно поделиться — скопируйте в issue
вручную.

### `_security.md` — не для секретов

Это **правила** работы с секретами, не хранилище. Никогда не кладите
туда реальные ключи.

### Runbooks — не для локальных проблем

Runbooks — для prod. Локальные проблемы — в `_troubleshooting.md`.

### Analysis — не task list

Findings описывают состояние. Чтобы действовать — создайте задачу в
`_backlog.md`.

---

## FAQ

### Вопрос: где хранить клон репозитория шаблонов?

Рекомендуется `~/Projects/<org>/opencode-templates/`. Один постоянный
клон, из него пушите и из него `init-opencode` тянет файлы.

Не путать с `~/.config/opencode/` — там живёт глобальный `AGENTS.md`.

### Вопрос: как обновить шаблон в проекте?

```bash
init-opencode --diff ~/Projects/my-app   # посмотреть
init-opencode --update ~/Projects/my-app # применить
```

Файлы с вашими данными (`WORK_LOG.md`, `_decisions.md`, `_backlog.md`,
`_concepts.md`, `_setup.md`) не перезаписываются.

### Вопрос: что такое bundle?

OKF-термин. Директория `.opencode/` целиком. Не файл, не репозиторий —
именно директория с концептами.

### Вопрос: зачем `index.md` и `log.md`, если есть `AGENTS.md` и `WORK_LOG.md`?

Формальная OKF-конформность. `index.md` и `log.md` — резервированные
имена. У нас они работают как **указатели** на наши основные файлы
(`AGENTS.md`, `WORK_LOG.md`).

### Вопрос: почему AGENTS.md не хранит code style?

Потому что code style нужен только когда пишешь код. В остальное
время — лишние токены в контексте. Когда задача дойдёт до кода — агент
сам прочитает `_codestyle.md`.

### Вопрос: что если хочу, чтобы агент всегда знал X?

Положите X в `AGENTS.md`. Это единственный файл, который всегда в
контексте. Но помните: чем больше `AGENTS.md`, тем меньше внимания
к деталям.

### Вопрос: как часто обновлять тематические файлы?

- `_concepts.md` — при архитектурных изменениях.
- `_ci.md` — при изменении `.github/workflows/`.
- `_troubleshooting.md` — после каждой решённой проблемы (>10 мин).
- `_backlog.md` — при появлении/завершении задач.
- `_decisions.md` — при значимых решениях.
- `_meta.md` — при обновлении шаблона.
- `_env.md` — при изменении окружений.

### Вопрос: у меня монорепозиторий, что делать?

Для каждого микросервиса/пакета — свой `.opencode/AGENTS.md`. Общие
правила — в родительском `.opencode/AGENTS.md` на уровне корня
монорепозитория.

### Вопрос: `_decisions.md` растёт бесконечно?

Да, но это ок. Принятые ADR **не редактируются** (кроме `Status`).
Старые записи — история. Если 100+ ADR — пора задуматься о чистке
(статус `deprecated`).

### Вопрос: зачем `_backlog.md`, если есть GitHub Issues?

Backlog — **черновик**, Issues — **подтверждённые задачи**. Backlog
дёшев для записи, Issues требует формулировки, меток, ассайна.
Правильный flow: идея → backlog → решение делать → Issue.

### Вопрос: как понять, что писать в `_decisions.md`, а что в `_worklog.md`?

- **`_decisions.md`** — значимые решения (архитектура, API, процесс).
- **`_worklog.md`** — все сессии, включая мелкие решения.

Если решение повлияет на других через полгода — ADR. Если это
«локальное решение в рамках сессии» — WORK_LOG.

### Вопрос: нужен ли `analysis/` для маленького проекта?

Скорее нет. `analysis/` — для случаев, когда вы проводите аудиты,
ищете проблемы производительности, исследуете техдолг. Для типичного
solo-проекта это может быть избыточно.

### Вопрос: как быть с секретами в `runbooks/`?

Не хранить реальные значения. Использовать placeholders:
`<DB_PASSWORD>`, «получить из Vault по пути ...». См. `_security.md`.

---

## Философия

### Скорость важнее качества

Шаблон следует философии
[Don't Aim for Quality, Aim for Speed](https://www.yegor256.com/2018/03/06/speed-vs-quality.html):

**Ваша задача — закрывать задачи быстро.** Качество — ответственность
проекта (CI, ревью, статический анализ), не ваша.

На практике:
- **Режьте углы.** Пишите рабочий код, ревьюеры поймают проблемы.
- **Маленькие PR.** Быстрее писать и ревьюить.
- **Не изучайте весь код.** Меняйте то, что требует задача.
- **Не бойтесь сломать.** CI поймает регрессии.

### Зачем всё это

Шаблон не про токены и экономию (хотя и это важно). Он про **ясность**.

Когда агент видит чистый, структурированный `AGENTS.md`, он:
- Быстро понимает проект.
- Точно знает, где искать детали.
- Не отвлекается на шум.
- Принимает лучшие решения.

Плохой `AGENTS.md` — как захламлённый рабочий стол. Хороший — как
органайзер с подписанными ящиками.

### OKF как фундамент

Мы строим на OKF, потому что:
- **Простота.** Markdown + frontmatter. Никаких бинарных форматов.
- **Портативность.** Работает с `cat`, `git clone`, любым редактором.
- **Стандарт.** Google Cloud, открытый формат, стабильная спека.
- **Расширяемость.** OKF явно разрешает добавление полей и расширений.

Наш шаблон — OKF-вдохновлённый bundle с расширениями для задач
AI-агента. Не строго конформный, но следующий духу спецификации.

---

*Полное руководство по OKF v0.1: [SPEC.md](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md).*
*Локальная выдержка: [`SPEC_REFERENCE.md`](SPEC_REFERENCE.md).*