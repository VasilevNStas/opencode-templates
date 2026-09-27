---
type: book
title: "Knowledge Bundles for AI Agents, part IV"
description: "A practical guide to OKF and agent-ready codebases"
timestamp: 2026-09-23
tags: [okf, agents, opencode, guide]
---
# Part IV — Operations

_[Part I](../textbook_ru/guide_ru_part_I.md), [Part II](../textbook_ru/guide_ru_part_II.md), [Part III](../textbook_ru/guide_ru_part_III.md) были про **использование** bundle. Part IV — про **эксплуатацию**: как его устанавливать, обновлять, расширять, поддерживать. Это мета-уровень: не «как работать с проектом через bundle», а «как работать с самим bundle»._

_Четыре главы: install/update, extending, anti-patterns, philosophy._

---

## Chapter 17. `init-opencode`: install and update
## Глава 17. `init-opencode`: установка и обновление

### 17.1 Зачем нужен installer

Bundle состоит из 20+ файлов. Копировать их вручную — рутина, которая быстро надоедает.

**Installer решает три задачи:**

1. **Install** — скопировать шаблон в проект
    
2. **Update** — обновить шаблонные файлы, не тронув пользовательские
    
3. **Diff** — показать, что изменится, до обновления


Без installer'а вы бы делали `cp -R template/ .opencode/` вручную. Это работает один раз. При обновлении — перезапишет все ваши данные.

### 17.2 Архитектура

`init-opencode` — bash-скрипт, ~200 строк. Живёт в репозитории шаблонов:

```text

opencode-templates/
├── VERSION                ← номер версии (например, v0.1.0)
├── README.md
├── README_en.md
├── README_ru.md
├── LICENSE
├── bin/
│   └── init-opencode      ← скрипт
└── template/              ← что копируется
    ├── AGENTS.md
    ├── _*.md
    ├── SPEC_REFERENCE.md
    ├── index.md
    ├── log.md
    ├── .gitignore
    ├── analysis/
    └── runbooks/
```
**Две ключевые директории:**

- `bin/` — скрипт, **не** копируется в проект.
    
- `template/` — содержимое, **копируется** в `.opencode/`.


**VERSION-файл** — просто строка `v0.1.0`. Используется в `.template-version`.

### 17.3 Установка скрипта

**Первое действие** после клонирования репозитория — поставить скрипт в PATH.

```bash
git clone <repo-url> ~/Projects/opencode-templates
ln -s ~/Projects/opencode-templates/bin/init-opencode \
      ~/.local/bin/init-opencode
```
**Symlink предпочтительнее копии.** Если правите скрипт в репозитории — изменения сразу доступны.

**Проверка:**

```bash
which init-opencode
# → /home/user/.local/bin/init-opencode
init-opencode --help
```
**Если `~/.local/bin` не в PATH:**

```bash
# В ~/.bashrc или ~/.zshrc
export PATH="$HOME/.local/bin:$PATH"
```
### 17.4 Режимы

Четыре режима:

```bash
init-opencode <project-dir>              # install
init-opencode --analyze <project-dir>    # install + hint for analysis
init-opencode --update <project-dir>     # update
init-opencode --diff <project-dir>       # preview
init-opencode --help                     # help
```
### 17.5 Install

#### Что делает

1. Проверяет, что `template/` существует
    
2. **Бэкапит** существующий `.opencode/` (если есть)
    
3. Создаёт `.opencode/`
    
4. Копирует содержимое `template/`
    
5. Создаёт пустые директории (`issue/`, `playbook/`, `pr/`, `archive/`)
    
6. Создаёт `.template-version` с метаданными
    
7. Выводит next steps


#### Бэкап

**Критичная часть.** Если `.opencode/` уже есть — он **не** перезаписывается. Вместо этого:

```bash
mv .opencode/ .opencode.bak.20260922-153045/
```
Timestamp — `YYYYMMDD-HHMMSS`. Можно откатиться, если что-то пошло не так.

**Почему не перезапись:** `.opencode/` содержит ваши данные. Перезапись = потеря `WORK_LOG.md`, `_decisions.md`, `_backlog.md`, `_concepts.md`.

#### Два режима install

**Обычный:**

```bash
init-opencode ~/Projects/my-app
```
Подсказка в конце: _«Starting a new project — help me fill in the template.»_

**С анализом:**

```bash
init-opencode --analyze ~/Projects/existing-repo
```
Подсказка: _«Analyze the repository and fill in the template.»_

**Разница — в подсказке для агента.** Сама установка одинакова.

#### Что получается

```text
my-app/
├── .opencode/
│   ├── .gitignore
│   ├── .template-version
│   ├── index.md
│   ├── log.md
│   ├── AGENTS.md
│   ├── SPEC_REFERENCE.md
│   ├── _*.md            (14 файлов)
│   ├── issue/           (пустая)
│   ├── playbook/        (пустая)
│   ├── pr/              (пустая)
│   ├── analysis/
│   │   ├── index.md
│   │   └── _finding.md
│   ├── runbooks/
│   │   ├── index.md
│   │   └── _runbook.md
│   └── archive/         (пустая)
├── src/
└── README.md
```
**`.template-version`:**

```text
version: v0.1.0
installed: 2026-09-22
source: /home/user/Projects/opencode-templates
```
### 17.6 Update

#### Что делает

1. Читает `.template-version`
    
2. Проверяет, что `.opencode/` существует
    
3. Для каждого файла из списка **ALWAYS_OVERWRITE**:
    
    - Сравнивает с версией из `template/` (через `cmp -s`)
        
    - Если отличается — копирует
        
4. Обновляет `.template-version`
    
5. Сообщает, сколько файлов обновлено


#### Два списка

**NEVER_OVERWRITE** (пользовательские):

- `WORK_LOG.md`
    
- `_concepts.md`
    
- `_setup.md`
    
- `_decisions.md`
    
- `_backlog.md`
    
- `_meta.md`
    
- Всё в `analysis/`, `runbooks/`, `issue/`, `playbook/`, `pr/`, `archive/`


**ALWAYS_OVERWRITE** (шаблонные):

- `AGENTS.md`, `index.md`, `log.md`, `SPEC_REFERENCE.md`
    
- `_codestyle.md`, `_ci.md`, `_commands.md`, `_files.md`
    
- `_glossary.md`, `_security.md`, `_troubleshooting.md`
    
- `_templates.md`, `_env.md`, `_worklog.md`


**Файлы из NEVER не участвуют в цикле вообще.** Мы их просто не трогаем.

#### Порядок действий при update

```bash
# 1. Посмотреть, что изменится
init-opencode --diff ~/Projects/my-app
# 2. Проверить каждое изменение
# (diff покажет точные различия)
# 3. Применить
init-opencode --update ~/Projects/my-app
```
**Никогда не запускайте `--update` без предварительного `--diff`.** Может быть, вы кастомизировали один из ALWAYS-файлов — и потеряете изменения.

#### Что если кастомизировали ALWAYS-файл

Например, добавили свою секцию в `_codestyle.md`. При update она **потеряется**.

**Решения:**

**Вариант A: сохранить локально.**

```bash
cp .opencode/_codestyle.md .opencode/_codestyle.local.md
```
Потом в `_codestyle.md` — ссылка: _«See also `_codestyle.local.md` for project-specific rules.»_

**Вариант B: перенести в NEVER.**

Отредактируйте `bin/init-opencode`, добавьте файл в `NEVER_OVERWRITE`. Тогда update его не тронет.

**Вариант C: не обновлять.**

Просто не запускайте `--update`. Bundle продолжит работать на старой версии.

**Вариант A — рекомендую.** Локальная копия безопаснее всего.

### 17.7 Diff

#### Что делает

1. Проверяет, что `.opencode/` существует
    
2. Для каждого ALWAYS-файла:
    
    - Сравнивает с `template/`
        
    - Если отличается — показывает `diff -u`
        
3. Сообщает, сколько файлов изменится


**Ничего не пишет.** Только preview.

#### Формат вывода

````text

=== _codestyle.md ===
--- .opencode/_codestyle.md    2026-09-15 ...
+++ template/_codestyle.md     2026-09-22 ...
@@ -10,6 +10,10 @@
 ## Lint — 0 offenses
 
 ```bash
 bundle exec rubocop
```
+Lint must pass with **zero offenses** before any commit.  
+  
+See _ci.md for CI-specific lint configuration.
````

````text

**`diff -u`** — unified diff. Понимают `patch` и `git apply`.
**Что смотреть:**
- **Новые секции** — что появилось.
- **Удалённые секции** — что исчезло.
- **Изменённые строки** — что поменялось.
#### Применение
Если diff устраивает:
```bash
init-opencode --update ~/Projects/my-app
```
````
Если хотите **только часть** изменений:

```bash
# Сохранить diff
init-opencode --diff ~/Projects/my-app > /tmp/changes.diff
# Отредактировать /tmp/changes.diff вручную
# Применить выборочно
cd ~/Projects/my-app && patch -p1 < /tmp/changes.diff
```
**Продвинутое использование.** Обычно `--update` достаточно.

### 17.8 Environment

#### Переменная `OPENCODE_TEMPLATE_REPO`

По умолчанию — `~/Projects/opencode-templates`. Переопределяется:

```bash
OPENCODE_TEMPLATE_REPO=~/work/opencode-templates \
  init-opencode ~/Projects/my-app
```
**Когда полезно:**

- Шаблон в нестандартном месте
    
- Несколько версий шаблона (stable, dev)
    
- CI, где `$HOME` другой


**В CI:**

```yaml

- name: Install bundle
  env:
    OPENCODE_TEMPLATE_REPO: /opt/opencode-templates
  run: init-opencode ${{ github.workspace }}
```
### 17.9 Обработка ошибок

Скрипт использует `set -euo pipefail`. Это значит: любая ошибка → скрипт останавливается.

#### Типичные ошибки

**`template not found at ...`**

Причина: `OPENCODE_TEMPLATE_REPO` не задан или неверный путь.

Fix: проверьте путь, задайте env var.

**`directory not found: <target>`**

Причина: целевая директория не существует.

Fix: создайте директорию или проверьте путь.

**`no .opencode/ in <target> — run install first`**

Причина: `--update` или `--diff` на проекте без bundle.

Fix: сначала `install`.

**`unknown option: --foo`**

Причина: опечатка или неподдерживаемая опция.

Fix: `init-opencode --help`.

#### Что делать при сбое install

Если install упал на середине:

1. Проверьте `.opencode/` — часть файлов скопирована?
    
2. Если да — удалите `.opencode/`
    
3. Проверьте бэкап: `.opencode.bak.*` — там старые данные
    
4. Запустите install заново


**Бэкап гарантирует**, что старые данные не потеряны.

### 17.10 VERSION-файл

#### Формат

```text
v0.1.0
```
Одна строка. SemVer.

#### Что делать при обновлении шаблона

Правя `template/`, увеличьте версию:

```text
v0.1.0 → v0.1.1   (bug fix)
v0.1.1 → v0.2.0   (new features, backward compatible)
v0.2.0 → v1.0.0   (breaking changes)
```
**SemVer для шаблонов:**

- **Patch** (`v0.1.1`) — исправления в существующих файлах
    
- **Minor** (`v0.2.0`) — новый файл, новая секция, совместимо
    
- **Major** (`v1.0.0`) — структурные изменения, breaking


**Что такое breaking changes:**

- Переименование файлов
    
- Удаление файлов
    
- Изменение структуры frontmatter
    
- Несовместимое изменение `init-opencode`


### 17.11 Как добавить новый файл в шаблон

Допустим, вы придумали `_api.md` — новый reference file.

**1. Создать `template/_api.md`.**

**2. Обновить `template/AGENTS.md`.**

Добавить в reference files:

```markdown
| [_api.md](_api.md) | Public API — methods, commands, endpoints |
```

**3. Обновить `bin/init-opencode`.**

Добавить `_api.md` в `ALWAYS_OVERWRITE` (или `NEVER` — зависит от типа).

**4. Обновить `VERSION`.**

`v0.1.0 → v0.2.0` (minor — new feature).

**5. Обновить `README_en.md` и `README_ru.md`.**

В секции `Template files` → `Navigation & safety`.

**6. Обновить словарь `type`** в README.

Добавить строку: `| api | _api.md |`.

**7. Закоммитить и запушить.**

Теперь при следующем `init-opencode --update` пользователи получат новый файл.

**Тонкость:** существующие пользователи получат `_api.md` как новый файл. Если у них уже был свой `_api.md` — `--update` перезапишет его. Это риск. Возможно, стоит добавить в `NEVER`.

### 17.12 Как изменить существующий файл

Допустим, вы хотите добавить секцию в `_concepts.md`.

**1. Отредактировать `template/_concepts.md`.**

**2. Обновить `VERSION`.**

`v0.1.0 → v0.1.1` (patch — исправление).

**3. Обновить README**, если структура изменилась.

**4. Commit, push.**

**5. Пользователи** получат обновление через `--update`.

**Проблема:** `_concepts.md` в `NEVER_OVERWRITE`. Значит, **пользователи не получат обновление автоматически**.

**Решения:**

- Если хотите, чтобы обновление доходило — перенесите `_concepts.md` в `ALWAYS`
    
- Но тогда пользовательские `_concepts.md` перезапишутся


**Компромисс:** добавьте новую секцию в отдельный файл. Например, `_concepts-advanced.md`. Пользователи, кому надо — прочитают. Остальным — не мешает.

### 17.13 Как решить, что класть в NEVER vs ALWAYS

**Критерий:** файл заполняется **пользователем под проект** или **одинаков во всех проектах**?

**NEVER** (пользовательские):

- `_concepts.md` — архитектура проекта уникальна.
    
- `_setup.md` — версии, команды уникальны.
    
- `_decisions.md` — ADR уникальны.
    
- `_backlog.md` — задачи уникальны.
    
- `_meta.md` — версия и даты уникальны.
    
- `WORK_LOG.md` — сессии уникальны.


**ALWAYS** (шаблонные):

- `AGENTS.md` — структура одинакова, заполняется плейсхолдерами.
    
- `index.md` — структура одинакова.
    
- `log.md` — структура одинакова.
    
- `SPEC_REFERENCE.md` — выдержка из спеки, не уникальна.
    
- `_codestyle.md` — структура одинакова.
    
- `_ci.md` — структура одинакова.
    
- `_commands.md` — структура одинакова.
    
- `_files.md` — структура одинакова.
    
- `_glossary.md` — структура одинакова.
    
- `_security.md` — структура одинакова.
    
- `_troubleshooting.md` — структура одинакова.
    
- `_templates.md` — структура одинакова.
    
- `_env.md` — структура одинакова.
    
- `_worklog.md` — шаблон, не данные.


**Пограничные случаи:**

- **`_env.md`** — может быть и там, и там. Структура общая, но URL'ы уникальны. Я поставил в ALWAYS — структура обновляется, значения теряются.
    
- **`_meta.md`** — версия уникальна, но структура одинакова. Я поставил в NEVER.


**Всё зависит от ваших приоритетов.** Что важнее: свежая структура или сохранённые данные?

### 17.14 Установка в существующий проект

Сценарий: у вас уже есть проект с `.opencode/` (ручной). Хотите перейти на `init-opencode`.

**1. Бэкап.**

```bash
mv .opencode .opencode.old
```
**2. Install.**

```bash
init-opencode ~/Projects/my-app
```
**3. Проверить разницу.**

```bash
diff -r .opencode.old .opencode
```
**4. Перенести данные вручную.**

- `WORK_LOG.md` — скопировать.
    
- `_decisions.md` — скопировать.
    
- `_concepts.md` — скопировать.
    
- `_backlog.md` — скопировать.


**5. Удалить бэкап.**

```bash
rm -rf .opencode.old
```
**Проблема:** ручной перенос. Но это один раз.

### 17.15 Работа с несколькими проектами

`init-opencode` работает с одним проектом за раз.

**Для нескольких:**

```bash

for project in ~/Projects/*; do
  [ -d "$project/.git" ] || continue
  init-opencode "$project"
done
```
**Осторожно:** проверяйте каждый проект отдельно. Не все должны получать bundle.

### 17.16 Обновление через CI

Можно автоматизировать обновление bundle:

```yaml
# .github/workflows/update-bundle.yml
name: Update bundle
on:
  schedule:
    - cron: '0 9 * * 1'  # Каждый понедельник в 9:00
  workflow_dispatch:
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Clone template
        run: git clone <repo-url> /tmp/opencode-templates
      - name: Install init-opencode
        run: ln -s /tmp/opencode-templates/bin/init-opencode /usr/local/bin/
      - name: Update bundle
        run: init-opencode --update .
      - name: Check for changes
        run: |
          if [ -n "$(git status --porcelain)" ]; then
            git add .
            git commit -m "Update opencode bundle"
            git push
          fi
```
**Что нужно:**

- Bundle **должен** коммититься (иначе CI не работает)
    
- Или — bundle в отдельном репозитории, CI обновляет его


**Обычно не нужно.** Bundle — локальный. Но для команды, где все используют один шаблон, — полезно.

### 17.17 Тестирование скрипта

Перед релизом новой версии — протестируйте `init-opencode`.

**Базовый сценарий:**

```bash
# Чистый install
mkdir /tmp/test1
init-opencode /tmp/test1
ls /tmp/test1/.opencode/
cat /tmp/test1/.opencode/.template-version
# Diff (ничего не должно меняться)
init-opencode --diff /tmp/test1
# → no differences
# Update (ничего не должно меняться)
init-opencode --update /tmp/test1
# → nothing to update
# Симулировать изменение в template
echo "# New line" >> template/_codestyle.md
# Diff должен показать изменение
init-opencode --diff /tmp/test1
# Update должен применить
init-opencode --update /tmp/test1
grep "New line" /tmp/test1/.opencode/_codestyle.md
# Проверить, что NEVER-файлы не тронуты
echo "# Custom" >> /tmp/test1/.opencode/_decisions.md
init-opencode --update /tmp/test1
grep "Custom" /tmp/test1/.opencode/_decisions.md
# → должен остаться
```
**Полный набор тестов:**

1. Install в чистую директорию
    
2. Install в директорию с `.opencode/` (проверить бэкап)
    
3. Update сразу после install (no-op)
    
4. Update после изменения template
    
5. NEVER-файлы не тронуты
    
6. Diff показывает корректные различия
    
7. `--help` работает
    
8. Неверный аргумент → ошибка


### 17.18 Anti-patterns

**Anti-pattern 1: `--update` без `--diff`.**

Может перезаписать кастомизированные файлы

**Anti-pattern 2: игнорировать бэкап.**

`init-opencode` создаёт `.opencode.bak.*`. Не удаляйте сразу — проверьте

**Anti-pattern 3: копия скрипта вместо symlink.**

Обновление скрипта не подхватится. Используйте symlink

**Anti-pattern 4: кастомизировать ALWAYS-файлы без сохранения.**

Изменения потеряются при update

**Anti-pattern 5: не увеличивать VERSION.**

Пользователи не узнают, что есть обновление

**Anti-pattern 6: breaking changes без major версии.**

Пользователи потеряют данные

**Anti-pattern 7: не тестировать скрипт.**

Пользователи столкнутся с багами

**Anti-pattern 8: зависеть от `$HOME/Projects/opencode-templates`.**

Используйте `OPENCODE_TEMPLATE_REPO`

### 17.19 Связь с другими файлами

```text
init-opencode
    │
    ├─→ template/           (что копируется)
    ├─→ VERSION             (версия)
    ├─→ .template-version   (машинные метаданные)
    └─→ _meta.md            (человеческие метаданные)
```
**Скрипт связывает:**

- Репозиторий шаблонов (источник)
    
- Проект (назначение)
    
- Метаданные версии


### 17.20 Упражнение

**Часть 1: установка.**

Если ещё не установили `init-opencode`:

1. Клонируйте репозиторий шаблонов
    
2. Создайте symlink
    
3. Проверьте `--help`


**Часть 2: тестирование.**

Создайте тестовый проект. Прогоните:

1. `init-opencode /tmp/test-project`
    
2. Проверьте структуру
    
3. `init-opencode --diff /tmp/test-project` — должно быть `no differences`
    
4. Измените что-то в `template/`
    
5. `init-opencode --diff /tmp/test-project` — покажет изменения
    
6. `init-opencode --update /tmp/test-project`


**Часть 3: аудит вашего репозитория.**

1. **VERSION** — актуальна?
    
2. **NEVER_OVERWRITE** — все файлы на месте? Ничего не забыли?
    
3. **ALWAYS_OVERWRITE** — все файлы на месте?
    
4. **README** — упоминает `init-opencode`?


### 17.21 Что дальше

В следующей главе — **Extending the bundle**. Как добавлять новые файлы, типы, изменять структуру. Что делать, когда стандартного набора не хватает.

## Chapter 18. Extending the bundle
## Глава 18. Расширения

### 18.1 Зачем расширять

Стандартный набор из 20+ файлов покрывает **большинство** проектов. Но не все.

**Когда стандарта не хватает:**

- Проект специфичен (например, встроенная система)
    
- Есть практики, которых нет в шаблоне
    
- Появилась новая категория знаний


**Расширение — это нормально.** OKF явно разрешает. Наш шаблон — тоже.

**Но осторожно.** Каждое расширение — это долг. Больше файлов — больше поддержки.

**Правило:** расширяйте, только если **реально** нужно. Не «на всякий случай».

### 18.2 Пять типов расширений

```text

1. Новый reference file (_api.md, _deploy.md, ...)
2. Новая директория (metrics/, incidents/, ...)
3. Новый type в frontmatter
4. Новая секция в существующем файле
5. Новое правило в init-opencode
```
Разберём каждый тип.

### 18.3 Тип 1: новый reference file

**Когда:** есть категория знаний, которой нет в стандарте.

**Примеры:**

- `_api.md` — публичный API (методы, эндпоинты)
    
- `_deploy.md` — процесс деплоя
    
- `_release.md` — процесс релиза
    
- `_performance.md` — характеристики производительности
    
- `_testing.md` — если тесты сложные и не умещаются в `_codestyle.md`
    
- `_monitoring.md` — метрики, алерты, dashboards
    
- `_compliance.md` — GDPR, SOC2, HIPAA


**Когда НЕ создавать:**

- Если умещается в существующий файл. Тесты — в `_codestyle.md`. Метрики — в `_env.md`.
    
- Если это одноразовая информация. Не заслуживает отдельного файла.
    
- Если файл будет пустым. Пустой файл хуже отсутствующего.


#### Как создать

**1. Определить назначение.**

Одно предложение: _«Этот файл отвечает на вопрос X»_.

**2. Создать `template/_<name>.md`.**

С frontmatter:

```markdown
---
type: <name>
title: "<Title>"
description: "<one-line>"
timestamp: <YYYY-MM-DD>
tags: [<name>]
---
# <Title>
<content>
```
**3. Обновить `AGENTS.md`.**

Добавить в reference files:

```markdown
| [_<name>.md](_<name>.md) | <when to read> |
```

**4. Обновить `init-opencode`.**

Добавить в `ALWAYS_OVERWRITE` (или `NEVER` — зависит от типа).

**5. Обновить словарь `type`.**

В README: `| <name> | _<name>.md |`.

**6. Обновить `VERSION`.**

Minor bump: `v0.1.0 → v0.2.0`.

**7. Обновить README_en.md и README_ru.md.**

В секции `Template files` — в соответствующую группу.

#### Пример: `_api.md`

```markdown
---
type: api
title: "Public API"
description: "Public methods, commands, and endpoints"
timestamp: 2026-09-22
tags: [api, reference]
---

# Public API
The public surface of the library. Everything here is guaranteed by
semver. Everything else is internal

## Methods
### `Parser.parse(input)`
Parses input and returns AST
- **Input:** `String` or `IO`
- **Returns:** `AST::Node`
- **Raises:** `ParserError` on invalid input
- **Since:** v1.0.0
  
### `Parser.parse_stream(io)`
Same as `parse`, but reads incrementally
- **Input:** `IO`-like object
- **Returns:** `Enumerator<AST::Node>`
- **Since:** v1.2.0
  
## Commands (CLI)
### `json-parser parse <file>`
Parses file and prints AST to stdout
- **Options:** `--pretty`, `--stream`
- **Exit codes:** 0 success, 1 parse error, 2 IO error
  
## Stability
- **Stable:** everything in this document
- **Deprecated:** `Parser.parse_legacy` — removed in v2.0.0
- **Internal:** everything under `JSON::Parser::Internal`
```
**Почему полезен:** агент по `_api.md` понимает, что можно менять, а что — публичный контракт.

**Где в `AGENTS.md`:**

```markdown

**Navigation & safety**
| File | When to read |
|------|-------------|
| [_api.md](_api.md) | Public API — methods, commands, endpoints |
```
### 18.4 Тип 2: новая директория

**Когда:** нужна отдельная категория динамических артефактов.

**Примеры:**

- `metrics/` — замеры производительности по сессиям
    
- `incidents/` — история инцидентов (post-mortem)
    
- `experiments/` — эксперименты, A/B-тесты
    
- `meetings/` — заметки со встреч
    
- `research/` — исследовательские заметки


**Когда НЕ создавать:**

- Если это часть существующей категории. Post-mortem — в `runbooks/`
    
- Если директория будет почти пустой


#### Как создать

**1. Определить назначение.**

**2. Создать директорию в `template/`.**

**3. Создать `index.md` внутри.**

```markdown
---
type: <name>-index
title: "<Title>"
description: "<one-line>"
timestamp: <YYYY-MM-DD>
tags: [<name>, index]
---

# <Title>
<описание>
## Index
| Date | Item | Status |
|------|------|--------|
| ... | ... | ... |
```

**4. Создать шаблон `_<name>.md`**

**5. Обновить `AGENTS.md`**

Добавить в reference files или в Issue workflow

**6. Обновить `init-opencode`**

Добавить директорию в `EMPTY_DIRS`

**7. Обновить `.gitignore` внутри `.opencode/`**

Добавить `<name>/*.md` с исключениями для `index.md` и `_<name>.md`

**8. Обновить `VERSION`**

Minor bump

#### Пример: `incidents/`

```text

.opencode/
├── incidents/
│   ├── index.md
│   ├── _postmortem.md
│   ├── 2026-09-15-db-failover.md
│   └── 2026-08-20-api-outage.md
```
**`incidents/index.md`:**

```markdown
---
type: incidents-index
title: "Incidents"
description: "Post-mortem records"
timestamp: 2026-09-22
tags: [incidents, index]
---

# Incidents
Post-mortem records for past incidents. For runbooks (procedures),
see `runbooks/`

## Index
| Date | Incident | Severity | Status |
|------|----------|----------|--------|
| 2026-09-15 | DB failover | P0 | resolved |
| 2026-08-20 | API outage | P1 | resolved |
```
**`incidents/_postmortem.md`:**

```markdown
---
type: postmortem
title: "<incident title>"
incident-date: <YYYY-MM-DD>
severity: P0 | P1 | P2
status: draft | final
timestamp: <YYYY-MM-DD>
tags: [postmortem, <area>]
---

# Post-mortem: <title>

## Timeline
- HH:MM — <event>
- HH:MM — <event>
  
## Root cause
<5 Whys>

## Impact
<Who was affected, how long, financial>

## What went well

## What could be improved

## Action items
- [ ] <action 1> — owner, deadline
- [ ] <action 2>
      
## Related
- Runbook: [<runbook>](../runbooks/<file>.md)
- ADR: [ADR-XXX](../_decisions.md#adr-xxx)
```
**Где в `AGENTS.md`:**

```markdown

**When things break**
| File | When to read |
|------|-------------|
| [incidents/](incidents/index.md) | Past incident records |
```
**Почему полезно:** post-mortem — не runbook. Runbook — процедура. Post-mortem — разбор. Разные жанры.

### 18.5 Тип 3: новый `type`

**Когда:** есть категория концептов, для которой нет подходящего типа в словаре.

**Примеры:**

- `type: runbook-index` — уже есть
    
- `type: postmortem` — новый
    
- `type: metrics` — новый
    
- `type: experiment` — новый


**Когда НЕ создавать:**

- Если есть близкий. `project-summary` vs `summary`
    
- Если это разовый файл. Не плодите типы ради одного файла
    
- Если можно обойтись без `type`. Но OKF требует `type` в каждом файле


#### Как добавить

**1. Выбрать имя.**

Осмысленное, короткое, в нижнем регистре. С дефисами если нужно: `runbook-index`.

**2. Использовать в frontmatter нового файла.**

**3. Обновить словарь в README.**

Добавить строку: `| <type> | <file> |`.

**4. Обновить `_meta.md`** (если расширение видимо).

В таблице `OKF base + extensions` → `Custom type values`.

**5. Обновить `VERSION`.**

Minor bump.

#### Принципы именования типов

**Используйте:**

- Существительные: `postmortem`, `finding`, `playbook`
    
- Единственное число: `runbook`, а не `runbooks`
    
- Одно слово где возможно: `ci`, не `continuous-integration`
    
- Дефисы для составных: `runbook-index`, `project-summary`


**Не используйте:**

- Глаголы: `decision-log` — существительное, ок. `log-decision` — плохо
    
- Общие слова: `file`, `doc`, `thing`
    
- Сокращения без причины: `pdca`, `okr` — если они не общеприняты в команде


#### Список типов в шаблоне

Текущий словарь (26 типов):

```text
project-context, index, log, meta, spec-reference,
architecture, setup, env, codestyle, commands, files, glossary,
security, troubleshooting, ci, decision-log, backlog, worklog, templates,
analysis-index, finding, runbook-index, runbook,
project-summary, playbook, pr
```
**Не создавайте новый тип, если можно использовать существующий.** Например, `postmortem`— новое, но если у вас один post-mortem в год — используйте `type: finding` или `type: log`.

### 18.6 Тип 4: новая секция в существующем файле

**Самый частый тип расширения.** Не новый файл, не новый тип — просто новая секция.

**Примеры:**

- В `_codestyle.md` — секция «Commit message format»
    
- В `_concepts.md` — секция «Performance characteristics»
    
- В `_setup.md` — секция «Troubleshooting during setup»
    
- В `_ci.md` — секция «Nightly jobs»


**Когда:**

- Секция логически принадлежит файлу
    
- Файл не разрастётся до неприличия
    
- Секция не пересекается с другой


#### Как добавить

**1. Добавить секцию в `template/_<file>.md`.**

**2. Обновить `VERSION`.**

Patch bump: `v0.1.0 → v0.1.1`.

**3. Commit, push.**

#### Пример: Commit format в `_codestyle.md`

```markdown
## Commit message format
Format: `<type>: <subject>`
Types:
- `feat` — new feature
- `fix` — bug fix
- `docs` — documentation
- `refactor` — refactoring
- `test` — tests
- `chore` — maintenance
Subject: imperative mood, no period, < 72 chars.
Examples:
- `feat: add streaming mode to parser`
- `fix: handle nil input in Session#load`
```
**Польза:** агент будет писать коммиты в правильном формате.

### 18.7 Тип 5: новое правило в `init-opencode`

**Когда:** меняется поведение installer'а.

**Примеры:**

- Добавить файл в NEVER/ALWAYS
    
- Добавить новую директорию
    
- Изменить логику бэкапа
    
- Добавить новую опцию


#### Как добавить

**1. Отредактировать `bin/init-opencode`.**

**2. Протестировать.** См. Chapter 17.17.

**3. Обновить `VERSION`.**

Minor или major — в зависимости от breaking

**4. Обновить README** (если интерфейс изменился).

**5. Commit, push.**

#### Пример: новая опция `--dry-run`

```bash

# Было
cmd_update() {
  ...
}
# Стало
cmd_update() {
  local dry_run="$1"
  for f in "${ALWAYS_OVERWRITE[@]}"; do
    if ! cmp -s "$TEMPLATE_DIR/$f" "$oc/$f" 2>/dev/null; then
      if [ "$dry_run" = "yes" ]; then
        info "would update $f"
      else
        cp "$TEMPLATE_DIR/$f" "$oc/$f"
        info "updated $f"
      fi
    fi
  done
}
```
**Обновить `usage()`** с описанием новой опции.

**Обновить README_en.md и README_ru.md.**

### 18.8 Полный пример: добавление `_api.md`

Разберём end-to-end.

#### Шаг 1: создать файл

`template/_api.md`:

```markdown
---
type: api
title: "Public API"
description: "Public methods, commands, and endpoints"
timestamp: 2026-09-22
tags: [api, reference]
---

# Public API
The public surface of the project. Everything here is guaranteed by
semver. Everything else is internal

## Methods
### `<Class>.<method>(<args>)`
<description>
- **Input:** <types>
- **Returns:** <type>
- **Raises:** <exceptions>
- **Since:** <version>
```
#### Шаг 2: обновить AGENTS.md

В `template/AGENTS.md`, в reference files:

```markdown
**Navigation & safety**
| File | When to read |
|------|-------------|
| [_api.md](_api.md) | Public API — methods, commands, endpoints |
```
#### Шаг 3: обновить init-opencode

В `bin/init-opencode`:

```bash
ALWAYS_OVERWRITE=(
  ...
  "_api.md"
  ...
)
```
#### Шаг 4: обновить VERSION

```text
v0.2.0 → v0.3.0
```
Minor bump: новый файл.

#### Шаг 5: обновить README

В `README_en.md`, секция `Template files` → `Navigation & safety`:

```markdown
| `_api.md` | Public API — methods, commands, endpoints |
```

В `README_ru.md` — то же самое.

В словаре `type`:

```markdown
| `api` | `_api.md` |
```

#### Шаг 6: обновить `_meta.md`

В `template/_meta.md`, секция `OKF base + extensions`:

```markdown
| Extension | What we added |
|-----------|---------------|
| ... | ... |
| `_api.md` | Public API reference |
```
#### Шаг 7: commit, push

```bash
git add .
git commit -m "Add _api.md reference file"
git push
```
#### Шаг 8: обновить в проектах

```bash
cd ~/Projects/my-app
init-opencode --diff .   # покажет новый файл
init-opencode --update . # применит
```
**Готово.** Новый файл появился во всех проектах при следующем update.

### 18.9 Расширение в проекте (без изменения шаблона)

Иногда нужно расширить bundle **для одного проекта**, не трогая шаблон.

**Пример:** в проекте есть специфичная практика, которой нет в других.

**Что делать:**

**1. Создать файл прямо в `.opencode/`.**

**2. Добавить в `AGENTS.md` этого проекта.**

**3. Не добавлять в `init-opencode`.**

**Проблема:** при следующем `--update` файл не тронется (его нет в списках), но и `AGENTS.md`**будет перезаписан** (он в ALWAYS).

**Решение:** создайте `AGENTS.local.md` и ссылайтесь на него из `AGENTS.md`:

```markdown
All shared workflow rules from `~/.config/opencode/AGENTS.md`.
Project-specific extensions:
- [_local.md](_local.md) — project-specific rules
```
**`AGENTS.local.md` не в списках**, `--update` его не тронет.

**Альтернатива:** редактируйте `AGENTS.md` проекта и **не запускайте `--update`**. Но тогда не получите обновлений.

### 18.10 Anti-patterns

**Anti-pattern 1: расширение без нужды.**

«Добавлю `_api.md`, вдруг пригодится.» Не надо. Только если реально нужно.

**Anti-pattern 2: дублирование.**

Добавили `_testing.md`, а тесты уже в `_codestyle.md`. Рассинхрон неизбежен.

**Anti-pattern 3: плодить типы.**

10 новых типов за месяц. Никто не запомнит. Используйте существующие.

**Anti-pattern 4: не обновлять `init-opencode`.**

Добавили файл, не добавили в списки. Пользователи его не получат.

**Anti-pattern 5: не обновлять `VERSION`.**

Пользователи не узнают о новой версии.

**Anti-pattern 6: breaking changes без major.**

Пользователи потеряют данные.

**Anti-pattern 7: расширять в проекте без `.local`.**

`AGENTS.md` перезапишется при update.

**Anti-pattern 8: не тестировать расширение.**

Файл добавлен, но не работает.

**Anti-pattern 9: слишком длинные файлы.**

`_codestyle.md` на 500 строк. Пора разбить.

**Anti-pattern 10: расширять bundle вместо кода.**

Bundle — для знаний. Код — отдельно.

### 18.11 Стратегия расширения

**Принципы:**

**1. Минимализм.**

Новый файл — только если без него **реально** плохо.

**2. Консистентность.**

Новый файл — по той же структуре, что остальные.

**3. Документирование.**

`AGENTS.md`, README, словарь типов — всё обновлено.

**4. Тестирование.**

`init-opencode` протестирован.

**5. Обратная совместимость.**

Minor bump для новых файлов, major для переименований.

**6. Эволюция, не революция.**

Не переделывайте всё сразу. Один файл за раз.

### 18.12 Связь с другими файлами

```text

              template/
                  │
     ┌────────────┼────────────┐
     ▼            ▼            ▼
 AGENTS.md    _meta.md      README
 (навигация)  (расширения)  (документация)
     │
     └─→ init-opencode
              │
              ▼
         .opencode/
```
**Расширение bundle — это правка нескольких файлов.** Не одного.

### 18.13 Упражнение

**Часть 1: аудит.**

Есть ли в вашем проекте знания, которых **нет** в bundle?

- Специфичные практики
    
- Необычные процессы
    
- Регулярные задачи


**Что стоит добавить?** Создайте `_<name>.md` или добавьте секцию.

**Часть 2: ревизия существующего расширения.**

Если у вас уже есть расширения:

1. **Они нужны?** Или можно удалить?
    
2. **Они актуальны?** Или устарели?
    
3. **Они задокументированы?** В `AGENTS.md`, README, словаре типов?


**Часть 3: end-to-end.**

Добавьте **один** новый reference file в свой шаблон:

1. Создайте `template/_<name>.md`
    
2. Обновите `AGENTS.md`
    
3. Обновите `init-opencode`
    
4. Обновите `VERSION`
    
5. Обновите README
    
6. Обновите `_meta.md`
    
7. Commit, push
    
8. Проверьте в тестовом проекте


**Замерьте время.** Оптимум: 20–30 минут на один файл.

### 18.14 Что дальше

В следующей главе — **Anti-patterns**. Общие ошибки, которые встречаются при работе с bundle. Не в отдельных файлах, а во всей системе.

## Chapter 19. Anti-patterns
## Глава 19. Анти паттерны

### 19.1 Что такое anti-pattern

**Anti-pattern** — это повторяющаяся ошибка, которая кажется правильной, но на самом деле вредит.

В отличие от простой ошибки, anti-pattern:

- **Кажется хорошей идеей.** Логика понятна, намерения благие.
    
- **Повторяется.** Один раз — случайность. Много раз — паттерн.
    
- **Имеет последствия.** Не сразу, но накапливаются.
    
- **Не очевидна.** Трудно заметить без рефлексии.


**Пример простой ошибки:** забыли обновить `timestamp`.

**Пример anti-pattern:** превратить `AGENTS.md` в энциклопедию, потому что «пусть агент знает всё».

### 19.2 Зачем эта глава

Предыдущие главы касались anti-patterns в контексте отдельных файлов и workflows. Здесь — **системный взгляд**.

Цель: **научиться замечать** anti-patterns. Не «не делать ошибок» (это невозможно), а «видеть проблему рано».

**Каждая категория ниже** — про свой уровень:

- **Structural** — как устроен bundle
    
- **Content** — как пишем
    
- **Process** — как используем
    
- **Relational** — как связываем
    
- **Evolutionary** — как развиваем


### 19.3 Structural anti-patterns

Проблемы **устройства** bundle.

#### Anti-pattern 1: энциклопедический AGENTS.md

**Симптом:** `AGENTS.md` на 200+ строк.

**Причина:** «пусть агент знает всё, чтобы не спрашивал».

**Почему плохо:**

- Шум в контексте
    
- Потеря фокуса агентом
    
- Расширяется бесконтрольно
    
- Дублирует reference files


**Как лечить:**

- Вынести всё, что «только когда пишешь код» → `_codestyle.md`
    
- CI-таблицы → `_ci.md`
    
- Карту файлов → `_files.md`
    
- Architecture details → `_concepts.md`


**Правило:** `AGENTS.md` — оглавление. Максимум 90 строк.

#### Anti-pattern 2: плоские reference files

**Симптом:** 20 строк в таблице reference files, без группировки

**Причина:** «чем проще, тем лучше»

**Почему плохо:**

- Глаз не находит нужное
    
- Порядок случайный
    
- Добавить новый файл некуда


**Как лечить:**

- Группировка по ситуации: onboarding, daily, break, navigation
    
- 3–5 групп — оптимум


#### Anti-pattern 3: файлы без назначения

**Симптом:** файл существует, но непонятно, когда его читать

**Причина:** «создам на всякий случай» или «так в шаблоне было»

**Почему плохо:**

- Не используется
    
- Устаревает
    
- Захламляет bundle


**Как лечить:**

- Для каждого файла сформулировать одно предложение: **«Этот файл отвечает на вопрос X»**
    
- Если не получается — удалить файл


#### Anti-pattern 4: дублирование между файлами

**Симптом:** одна информация в двух местах

**Причина:** «надо убедиться, что увидят»

**Почему плохо:**

- Рассинхрон неизбежен
    
- При обновлении забываете обновить копию
    
- Через месяц — противоречия


**Как лечить:**

- **Правило «link, don't duplicate».**
    
- Оставить в одном месте, в другом — ссылку
    
- Проверка: **«если изменится X — придётся править в двух файлах?»** Если да — дублирование


#### Anti-pattern 5: bundle в корне репозитория

**Симптом:** `_concepts.md`, `_ci.md` лежат рядом с `src/`, `README.md`

**Причина:** «чтобы было видно»

**Почему плохо:**

- Загрязняет репозиторий
    
- Смешивается с публичной документацией
    
- Не понятно, что это локальное
    
- Риск случайного коммита


**Как лечить:**

- Bundle в `.opencode/`
    
- В `.git/info/exclude`


#### Anti-pattern 6: неполный bundle

**Симптом:** bundle из 3–4 файлов. Никаких директорий

**Причина:** «взял только то, что нужно»

**Почему плохо:**

- Динамические артефакты некуда класть
    
- Workflows не работают
    
- Bundle не самодостаточен


**Как лечить:**

- Использовать `init-opencode` — он ставит полный bundle
    
- Если ручная установка — не забывать про `issue/`, `playbook/`, `pr/`, `analysis/`, `runbooks/`, `archive/`


### 19.4 Content anti-patterns

Проблемы **содержимого** файлов.

#### Anti-pattern 7: файлы-пустышки

**Симптом:** файл с frontmatter и одной строкой «Not filled yet»

**Причина:** «создам скелет, потом заполню»

**Почему плохо:**

- «Потом» не наступает
    
- Агент читает пустоту
    
- Захламляет bundle


**Как лечить:**

- Если не заполнено — удалить
    
- Или заполнить **сразу**. Хотя бы минимально
    
- **Пустой файл хуже отсутствующего.** Отсутствие = «не нужно». Пустой = «нужно, но забыли»


#### Anti-pattern 8: устаревшие файлы

**Симптом:** `timestamp` — полгода назад. Содержимое не соответствует реальности

**Причина:** обновили код, забыли про bundle

**Почему плохо:**

- Агент работает по ложным данным
    
- Хуже, чем отсутствие файла
    
- Подрывает доверие к bundle


**Как лечить:**

- Обновлять при каждом значимом изменении
    
- Раз в квартал — ревизия `timestamp`
    
- Если файл не актуален — обновить или удалить


#### Anti-pattern 9: длинные файлы

**Симптом:** `_concepts.md` на 500 строк

**Причина:** «добавлю ещё секцию, будет полнее»

**Почему плохо:**

- Файл перестаёт читаться
    
- Агент тонет в деталях
    
- Сложно поддерживать


**Как лечить:**

- Если файл > 100 строк — разбить или сократить
    
- **`_concepts.md`** — 80 строк оптимум
    
- **`_setup.md`** — 60 строк
    
- **`_troubleshooting.md`** — 80–100 строк (растёт)
    
- **`_ci.md`** — 60–80 строк
    
- **`AGENTS.md`** — 60–90 строк


#### Anti-pattern 10: файлы без timestamp

**Симптом:** `timestamp: <YYYY-MM-DD>` — плейсхолдер

**Причина:** «заполню потом» или «не важно»

**Почему плохо:**

- Невозможно понять актуальность
    
- Агент не отличает свежее от старого
    
- Нарушение OKF-конвенции


**Как лечить:**

- Заполнять **сразу** при создании файла
    
- Обновлять при каждом изменении


#### Anti-pattern 11: файлы с секретами

**Симптом:** `_security.md` содержит токен. `_env.md` содержит пароль

**Причина:** «для удобства» или «не подумал»

**Почему плохо:**

- Утечка при первом коммите
    
- Даже если не коммитится — рискует
    
- Нарушение базовых правил


**Как лечить:**

- **Никогда** не хранить секреты в bundle
    
- Только ссылки: «Vault, путь X»
    
- Если случайно попал — отозвать, ротировать, очистить


#### Anti-pattern 12: смешение языков

**Симптом:** `AGENTS.md` на английском, `_concepts.md` на русском, `_ci.md` на смеси

**Причина:** разные авторы, разные периоды

**Почему плохо:**

- Непрофессионально
    
- Агенту сложнее
    
- Сложно поддерживать


**Как лечить:**

- Выбрать **один** язык для bundle
    
- Рекомендую английский — родной для LLM
    
- Личные пометки — на русском, но не смешивать


### 19.5 Process anti-patterns

Проблемы **использования** bundle

#### Anti-pattern 13: bundle не используется

**Симптом:** bundle есть, но вы им не пользуетесь. Не пишете в WORK_LOG, не обновляете backlog.

**Причина:** «нет времени» или «забываю»

**Почему плохо:**

- Bundle устаревает
    
- Через месяц бесполезен
    
- Лучше не иметь, чем иметь мёртвый


**Как лечить:**

- Начать с **одного** файла — WORK_LOG
    
- Ввести ритуал: конец сессии → запись
    
- Через неделю — заметите пользу


#### Anti-pattern 14: bundle используется частично

**Симптом:** WORK_LOG ведётся, backlog — нет. Или наоборот.

**Причина:** «этот файл удобный, тот — нет»

**Почему плохо:**

- Динамика теряется
    
- Связи между файлами не работают
    
- Bundle не самодостаточен


**Как лечить:**

- Использовать **весь** bundle
    
- Если какой-то файл не нужен — удалить, а не игнорировать
    
- **Пустой файл хуже отсутствующего.**


#### Anti-pattern 15: bundle без ритуала

**Симптом:** bundle используется **иногда**. Когда вспомните.

**Причина:** нет триггеров

**Почему плохо:**

- Нерегулярно → бесполезно
    
- Пропустили сессию → пропустили ещё
    
- Через месяц — заброшен


**Как лечить:**

- Ввести **триггеры**:
    
    - Начало сессии → прочитать WORK_LOG
        
    - Конец сессии → записать WORK_LOG
        
    - Новая идея → в backlog
        
    - Значимое решение → ADR
        

#### Anti-pattern 16: bundle заменяет реальную работу

**Симптом:** бесконечно правите bundle, но не пишете код.

**Причина:** bundle — комфортнее кода. Или — прокрастинация.

**Почему плохо:**

- Bundle — инструмент, не цель
    
- Цель — закрывать задачи
    
- Бесконечное «улучшение» bundle — форма прокрастинации


**Как лечить:**

- **Правило:** bundle обновляется **по мере работы**. Не вместо.
    
- Если за день не сделали ни одной задачи, но правили bundle — что-то не так


#### Anti-pattern 17: bundle для галочки

**Симптом:** bundle есть, но вы работаете **как раньше**

**Причина:** «руководство требует» или «модно»

**Почему плохо:**

- Bundle становится мёртвым грузом
    
- Тратите время на поддержку без пользы
    
- Лучше не иметь


**Как лечить:**

- Честно ответить: **«Помогает ли bundle?»**
    
- Если нет — либо понять, почему, либо удалить


### 19.6 Relational anti-patterns

Проблемы **связей** между файлами и людьми

#### Anti-pattern 18: битые ссылки

**Симптом:** `_setup.md` ссылается на `_env.md`, а его нет

**Причина:** удалили файл, забыли ссылку

**Почему плохо:**

- Агент пытается читать — не находит
    
- Раздражает
    
- Подрывает доверие


**Как лечить:**

- При удалении файла — grep по ссылкам
    
- Регулярная проверка: `grep -r '\[.*\](.*\.md)' .` и проверка


#### Anti-pattern 19: циклы ссылок

**Симптом:** A ссылается на B, B на C, C на A

**Причина:** добавили ссылки «для связности»

**Почему плохо:**

- Агент может уйти в цикл
    
- Запутывает
    
- Бессмысленно


**Как лечить:**

- Держать граф ссылок **ацикличным**
    
- Если цикл — заменить одну из ссылок на текстовое упоминание


#### Anti-pattern 20: файлы, ссылающиеся сами на себя

**Симптом:** `_meta.md` ссылается на `_meta.md`

**Причина:** копипаст или поспешность

**Почему плохо:**

- Бессмысленно
    
- Выглядит как баг


**Как лечить:**

- Проверка: `grep -r "$(basename "$f")" "$f"`


#### Anti-pattern 21: bundle для одного человека

**Симптом:** bundle написан так, что понятен только автору. Сокращения без расшифровки, внутренние шутки, личные метки.

**Причина:** «я же понимаю»

**Почему плохо:**

- Агент не понимает
    
- Коллега не понимает
    
- Вы через полгода не понимаете


**Как лечить:**

- **Правило «другого человека»**: если коллега прочитает — поймёт?
    
- Писать для будущего себя, не для текущего


#### Anti-pattern 22: bundle без контекста

**Симптом:** `_concepts.md` описывает компоненты, но неясно **зачем** они.

**Причина:** пишете «что», забываете «почему»

**Почему плохо:**

- Нет понимания мотивации
    
- Агенту сложно принимать решения
    
- «Почему» — важнее, чем «что»


**Как лечить:**

- В `_concepts.md` — не только «что», но и «почему так»
    
- Или ссылка на ADR: _«Почему pipeline — см. ADR-005.»_


### 19.7 Evolutionary anti-patterns

Проблемы **развития** bundle

#### Anti-pattern 23: bundle не растёт

**Симптом:** bundle создан год назад, с тех пор не менялся

**Причина:** «работает — не трогай»

**Почему плохо:**

- Устарел
    
- `_troubleshooting.md` не пополняется
    
- `_backlog.md` неактуален
    
- Bundle — **живая система**


**Как лечить:**

- `_troubleshooting.md` — после каждой решённой проблемы
    
- `_backlog.md` — при появлении/завершении задач
    
- `_ci.md` — при изменении workflows


#### Anti-pattern 24: bundle растёт бесконтрольно

**Симптом:** `_backlog.md` с 100 пунктами. `_files.md` с 200 файлами.

**Причина:** добавляете всё, не удаляете

**Почему плохо:**

- Файлы нечитаемы
    
- Планирование замедляется
    
- Полезное тонет в мусоре


**Как лечить:**

- Раз в месяц — ревизия
    
- `_backlog.md` — удалять неактуальное
    
- `_files.md` — только 15–30 ключевых файлов
    
- `_decisions.md` — статус `deprecated` для устаревших ADR


#### Anti-pattern 25: расширения без надобности

**Симптом:** 30 файлов в bundle. Половина не используется

**Причина:** «а вдруг пригодится»

**Почему плохо:**

- Bundle сложнее
    
- Больше поддержки
    
- Новому человеку непонятно


**Как лечить:**

- **Минимализм.** Только то, что **реально** нужно
    
- Ревизия раз в полгода: какой файл не читал ни разу? Удалить.


#### Anti-pattern 26: breaking changes без версии

**Симптом:** переименовали `_concepts.md` в `_architecture.md`, не увеличив major версию.

**Причина:** «незначительное изменение»

**Почему плохо:**

- Пользователи теряют файл
    
- Ссылки битые
    
- `--update` может сломаться


**Как лечить:**

- **SemVer.** Breaking changes — major bump
    
- Переименования — breaking
    
- Удаления — breaking


#### Anti-pattern 27: нет миграции

**Симптом:** новая версия шаблона сломала существующие bundle

**Причина:** изменили структуру без миграции

**Почему плохо:**

- Пользователи в панике
    
- Данные потеряны
    
- Доверие подорвано


**Как лечить:**

- Для major изменений — **migration guide**
    
- `init-opencode --migrate` — опциональный режим
    
- Или — предоставить скрипт миграции


### 19.8 Мета-anti-pattern: слишком серьёзно

**Симптом:** вы часами правите `_concepts.md`, чтобы было «идеально»

**Причина:** перфекционизм

**Почему плохо:**

- Bundle — **инструмент**, не цель
    
- Время уходит на полировку
    
- Задачи не закрываются


**Как лечить:**

- **Помните философию:** speed over quality
    
- Bundle может быть несовершенным
    
- Лучше — работающий, чем идеальный


### 19.9 Как замечать anti-patterns

**Рефлексия раз в месяц.**

Задайте вопросы:

**Про структуру:**

- `AGENTS.md` растёт?
    
- Файлы дублируются?
    
- Есть пустые файлы?


**Про контент:**

- Все `timestamp` свежие?
    
- Все ссылки работают?
    
- Нет секретов?


**Про процесс:**

- Все файлы используются?
    
- Ритуалы соблюдаются?
    
- Bundle помогает или мешает?


**Про связи:**

- Ссылки работают?
    
- Нет циклов?
    
- Понятно без контекста?


**Про эволюцию:**

- Bundle растёт?
    
- Или устаревает?
    
- Расширения нужны?


**Если нашли 3+ проблемы** — пора чистить

### 19.10 Чистка bundle

**Когда чистить:**

- Раз в квартал — планово
    
- При обнаружении anti-pattern
    
- Перед крупным изменением


**Как чистить:**

**Шаг 1: инвентаризация.**


```bash
# Список файлов и размеров
ls -la .opencode/
# Проверить timestamps
grep -l "timestamp: <" .opencode/*.md
# Проверить ссылки
grep -r '\[.*\](.*\.md)' .opencode/ | while read line; do
  # ... проверка существования
done

```
**Шаг 2: категоризация.**

Для каждого файла:

- **Используется** — оставить
    
- **Используется редко** — оставить, но проверить
    
- **Не используется** — удалить
    
- **Пустой** — удалить или заполнить


**Шаг 3: ревизия контента.**

Для каждого оставшегося файла:

- Актуален? Обновить.
    
- Дублируется? Объединить.
    
- Устарел? Удалить.


**Шаг 4: ревизия структуры.**

- `AGENTS.md` — оптимальный размер?
    
- Группировка в reference files — правильная?
    
- Словарь `type` — все типы используются?


**Шаг 5: документация.**

После чистки — обновить:

- README (если что-то изменилось).
    
- `_meta.md` (расширения).
    
- `VERSION` (если правки значительные).


### 19.11 Профилактика

**Как предотвращать anti-patterns:**

**1. Ритуалы.**

- Начало сессии: прочитать WORK_LOG.
    
- Конец сессии: записать WORK_LOG.
    
- Новая идея: в backlog.
    
- Значимое решение: ADR.


**2. Регулярная ревизия.**

- Раз в месяц: проверить backlog.
    
- Раз в квартал: полная чистка bundle.
    
- Раз в полгода: аудит структуры.


**3. Правило другого человека.**

- Перед добавлением файла: «Коллега поймёт?»
    
- Перед секцией: «Это в другом файле?»
    
- Перед ссылкой: «Она работает?»


**4. Минимализм.**

- Новый файл — только если **реально** нужен.
    
- Новая секция — только если **действительно** важна.
    
- Новая директория — только если есть что положить.


**5. Консистентность.**

- Один язык
    
- Один стиль
    
- Одна структура


### 19.12 Связь с другими файлами

Anti-patterns могут быть **в любом файле** и **в любом workflow**. Эта глава — **синтез** предыдущих глав.

Каждая глава Parts II–III содержала свои anti-patterns. Здесь — **системный уровень**: что объединяет проблемы.

### 19.13 Упражнение

**Часть 1: аудит своего bundle.**

Откройте ваш `.opencode/`. Пройдитесь по anti-patterns выше. Отметьте те, что у вас есть.

**Часть 2: план чистки.**

Для каждой найденной проблемы:

- **Что именно** не так
    
- **Что делать** (обновить / удалить / объединить)
    
- **Когда** (сегодня / на выходных / в конце квартала)


**Часть 3: ритуалы.**

Определите **три ритуала**, которые вы введёте:

- Например: конец сессии → 5 строк в WORK_LOG
    
- Раз в неделю → ревизия backlog
    
- Раз в месяц → проверка `timestamp` во всех файлах
   

**Запишите** их в `_meta.md` или в `AGENTS.md` своего проекта.

### 19.14 Что дальше

В следующей главе — **Philosophy**. Почему bundle устроен именно так. Философия, которая стоит за всеми решениями: минимализм, speed over quality, OKF.


## Chapter 20. Philosophy
## Глава 20. Философия

### 20.1 Зачем эта глава

Все предыдущие главы отвечали на вопросы **«что»** и **«как»**:

- **Что** лежит в bundle
    
- **Как** этим пользоваться
    
- **Как** расширять
    
- **Что** не делать


Эта глава — про **«почему»**.

Почему bundle устроен именно так? Почему файлы называются с подчёркиванием? Почему `WORK_LOG.md` не коммитится? Почему `AGENTS.md` — маленький, а не большой?

Ответы — в **философии**. Не абстрактной, а практической: набор принципов, из которых следуют конкретные решения.

### 20.2 Шесть принципов

Всё устройство bundle сводится к шести принципам:

1. **Speed over quality.**
    
2. **Минимализм.**
    
3. **Ленивая загрузка.**
    
4. **Локальность.**
    
5. **Ясность через структуру.**
    
6. **OKF как фундамент.**

Разберём каждый.

### 20.3 Принцип 1: Speed over quality

#### Идея

**Ваша задача — закрывать задачи быстро.** Качество — ответственность **проекта**, не ваша.

Проект фильтрует изменения через:

- CI (тесты, линт, сборка)
    
- Ревью (человек проверяет)
    
- Статический анализ (RuboCop, ESLint)
    
- Мониторинг (метрики после деплоя)

Если что-то плохое прошло все фильтры — это **проблема процесса**, а не ваша.

#### Откуда это взято

Философия изложена в статье [Don't Aim for Quality, Aim for Speed](https://www.yegor256.com/2018/03/06/speed-vs-quality.html) Yegor Bugayenko.

**Основная мысль:**

> Программист генерирует изменения. Проект их фильтрует. Не пытайтесь быть фильтром для себя — это замедляет и не помогает.

#### Что это значит на практике

**Режьте углы.** Не полируйте. Пишите рабочий код — ревьюеры и CI поймают проблемы.

**Маленькие PR.** Быстрее писать, быстрее ревьюить, быстрее мержить.

**Не изучайте весь код.** Меняйте только то, что требует задача.

**Не бойтесь сломать.** CI поймает регрессии. Если не поймает — это проблема CI.

#### Как это отражено в bundle

**Bundle не требует идеальности.**

- `_concepts.md` может быть неполным. Заполните по мере необходимости
    
- `_troubleshooting.md` — обогащается после каждой проблемы
    
- `_backlog.md` — не roadmap, а черновик


**Bundle не тормозит.** Правило «за 5 секунд»:

- Найти нужный файл — за 5 секунд
    
- Записать сессию — за 5 минут
    
- Обновить backlog — за 2 минуты


Если что-то занимает дольше — **это проблема дизайна**

#### Что это НЕ значит

**Не значит «пишите плохой код».** Значит — **не блокируйтесь** на качестве.

**Не значит «не тестируйте».** Тесты — часть работы.

**Не значит «не рефакторьте».** Рефакторинг — часть работы.

**Не значит «не думайте».** Значит — **не застревайте** на перфекционизме.

#### Ловушка

**Прокрастинация через качество.** «Ещё немного отполирую, потом отправлю». Через 3 дня PR всё ещё не открыт.

**Правильно:** открыть PR, получить ревью, поправить.

#### В контексте bundle

**Bundle тоже не должен быть идеальным.**

- Если файл не идеален — оставьте.
    
- Если ссылка битая — поправьте позже.
    
- Если ADR написан коряво — важно, чтобы **был**.


**Лучше работающий несовершенный bundle, чем идеальный неработающий.**

### 20.4 Принцип 2: Минимализм

#### Идея

**Каждый файл, каждая секция, каждая строка** должны **заслуживать** своё место.

Не «добавим на всякий случай», а «без этого не работает».

#### Откуда это взято

Прямо из OKF:

> **Key principle:** minimalism. No central schema registry, no mandatory tooling.

OKF подчёркивает: знаний должно быть **ровно столько, сколько нужно**.

#### Что это значит на практике

**Bundle — не энциклопедия.** Не пытайтесь описать всё.

**`AGENTS.md` — не свалка.** Только то, что нужно **каждой** задаче.

**Файлы не плодятся.** Новый файл — только если **реально** нужен.

**Секции не дублируются.** Если информация есть в другом файле — ссылка.

#### Как это отражено в bundle

**Длина файлов ограничена.**

- `AGENTS.md` — 60–90 строк.
    
- `_setup.md` — 60 строк.
    
- `_concepts.md` — 80 строк.
    
- `_files.md` — 15–30 записей.


Если файл растёт больше — сигнал пересмотреть.

**Удаление — часть работы.**

- Устаревшие ADR — `deprecated`
    
- Выполненные задачи — удаляются из `_backlog.md`
    
- Ненужные файлы — удаляются


**Пустой файл хуже отсутствующего.**

- Отсутствие файла = «не нужно»
    
- Пустой файл = «нужно, но забыли»
    
- Пустой файл **врёт**. Отсутствующий — честен.


#### Ловушка

**«А вдруг пригодится».**

- «Добавлю секцию про X — вдруг понадобится.»
    
- «Создам файл `_api.md` — вдруг будет API.»


**Правильно:** добавить, когда **реально** понадобилось.

#### Что это НЕ значит

**Не значит «не документируйте».** Значит — **документируйте нужное**.

**Не значит «не расширяйте».** Значит — расширяйте **осознанно**.

**Не значит «не пишите много».** Значит — пишите **по делу**.

### 20.5 Принцип 3: Ленивая загрузка

#### Идея

**Читать только то, что нужно для текущей задачи.**

Не «загрузить всё на старте», а «загрузить по требованию».

#### Почему это важно

**Контекстное окно ограничено.** У LLM есть предел. Чем ближе к нему, тем хуже ответы.

**Внимание размывается.** Даже если контекст большой, модель теряет фокус на длинном вводе (эффект «lost in the middle»).

**Стоимость растёт.** Каждый токен — деньги или латентность.

#### Как это работает в bundle

**`AGENTS.md` — маленький.** 60–90 строк.

**`_*.md` — читаются по триггеру.** Таблица в `AGENTS.md` говорит, **когда** читать какой файл.

**Директории — читаются редко.** `analysis/`, `runbooks/` — только при необходимости.

#### Пример

**Задача:** «Добавь метод в User».

**Что читает агент:**

- `AGENTS.md` — контекст
    
- `_codestyle.md` — как писать код
    
- `_files.md` — где User
    
- `_concepts.md` — архитектура


**Что НЕ читает:**

- `_ci.md` — не про CI
    
- `_security.md` — не про секреты
    
- `runbooks/` — не инцидент
    
- `analysis/` — не анализ


**Экономия:** 4 файла вместо 20.

#### Ловушка

**Grep вместо таблицы.**

«Почему не grep?» — потому что:

- Grep ищет **слова**, не **смысл**.
    
- Задача «напиши код» не содержит слова «codestyle».
    
- Таблица в `AGENTS.md` — **явное** знание. Grep — догадки.
    

**Eager loading.**

«Загрузим всё сразу — агент точно найдёт». Нет. Утонет в шуме.

#### Что это НЕ значит

**Не значит «агент не может читать много».** Может. Но **не должен** без причины.

**Не значит «все файлы маленькие».** `_concepts.md` может быть 80 строк. Это ок.

**Не значит «нет общего контекста».** `AGENTS.md` — общий контекст. Reference files — детали.

### 20.6 Принцип 4: Локальность

#### Идея

**Bundle — личный.** Не коммитится. Не делится автоматически. Не публичен.

#### Почему это важно

**Свобода.** Вы можете писать что угодно. Не боитесь, что увидят коллеги.

**Честность.** Записываете **как есть**, не «для отчётности».

**Безопасность.** Секреты, внутренние URL'ы — не утекут.

**Скорость.** Не нужно согласовывать формат с командой.

#### Как это отражено в bundle

**`.opencode/` в `.git/info/exclude`.**

- Локально для вашего клона
    
- Не заражает репозиторий


**`.gitignore` внутри `.opencode/`.**

- Вторая линия обороны
    
- На случай, если `.opencode/` попадёт в git


**`WORK_LOG.md` — личный.**

- Не синхронизируется
    
- Если нужно поделиться — копируйте в issue вручную


**`_decisions.md` — локальный.**

- Может быть расшарен, если нужно
    
- Но по умолчанию — только для вас


#### Ловушка

**«Поделюсь bundle с командой».**

Можно — но осознанно. Через:

- Копию в публичный репозиторий
    
- Или через issue/PR


**По умолчанию — не делится.**

**«Закоммичу bundle — пусть будет».**

Не надо. Bundle — инструмент, не артефакт проекта.

#### Что это НЕ значит

**Не значит «bundle бесполезен для команды».** Может быть полезен — но как **шаблон**, не как данные.

**Не значит «нельзя делиться».** Можно. Но это **отдельное действие**.

**Не значит «секретно».** Просто **личное**.

### 20.7 Принцип 5: Ясность через структуру

#### Идея

**Структура важнее содержания.** Форма важнее текста.

#### Почему это важно

**Агент читает структуру.** Заголовки, списки, таблицы — парсятся лучше, чем свободный текст.

**Человек читает структуру.** Глаз находит секцию за секунды.

**Поддержка проще.** Структурированный файл легко обновить.

#### Как это отражено в bundle

**Frontmatter — обязателен.**

- Метаданные парсятся
    
- `type`, `title`, `description` — понятны агенту без чтения body
    

**Секции — стандартные.**

- `## Overview` — везде
    
- `## Key components` — в `_concepts.md`
    
- `## Symptom`, `## Cause`, `## Fix` — в `_troubleshooting.md`
    

**Таблицы вместо абзацев.**

- Где возможно — таблица.
    
- Пара «ключ → значение» читается лучше, чем «X значит Y, а Z значит W»


**Заголовки вместо переходов.**

- Не «Далее рассмотрим...», а `## Следующая секция`
    
- Заголовок — якорь для чтения
 

#### Ловушка

**Красивые абзацы.**

«Ну, я тут подумал, что, возможно, стоит рассмотреть...»

**Правильно:** заголовок + список + таблица. Действие вместо рефлексии.

**Длинные секции.**

500 строк в одном файле без подзаголовков.

**Правильно:** подзаголовки каждые 20–30 строк.

#### Что это НЕ значит

**Не значит «нет свободного текста».** Есть. Но структура **предпочтительнее**.

**Не значит «нет абзацев».** Есть. Но короткие.

**Не значит «только таблицы».** Таблицы — где уместно. Списки — где уместно. Абзацы — где ничего другого.

### 20.8 Принцип 6: OKF как фундамент

#### Идея

**Bundle следует OKF.** Не полностью, но по духу.

OKF — открытый формат от Google Cloud. Минимализм, markdown + frontmatter, никакой центральной схемы.

#### Почему именно OKF

**Простота.**

- Markdown + YAML
    
- Читается `cat`
    
- Копируется `git clone`


**Портативность.**

- Не привязан к инструменту
    
- Работает с любым редактором
    
- Переносится между системами
 

**Стандарт.**

- Google Cloud
    
- Открытый
    
- Стабильная спека
 

**Расширяемость.**

- OKF явно разрешает добавление полей
    
- Не требует центральной регистрации типов
 

#### Что взято из OKF

**Структура bundle.**

- `index.md`, `log.md` — резервированные имена
    
- Concept documents — остальные файлы
 

**Frontmatter.**

- Обязательное поле `type`
    
- Рекомендуемые: `title`, `description`, `resource`, `tags`, `timestamp`
 

**Cross-linking.**

- Markdown-ссылки между концептами
    
- Ссылка утверждает «наличие отношения»


**Citations.**

- `## Citations` — конвенциональная секция


**Толерантность.**

- Не отвергать bundle из-за неизвестных полей
    
- Битые ссылки — допустимы
 

#### Что добавлено поверх OKF

**`AGENTS.md` вместо `index.md`.**

- OKF использует `index.md`
    
- OpenCode читает `AGENTS.md`
    
- Мы используем `AGENTS.md` + `index.md` как указатель


**`_*.md` префикс.**

- Конвенция из SASS/Jekyll
    
- Служебные файлы помечаются


**`WORK_LOG.md`.**

- Аналог OKF `log.md`
    
- Но локальный


**Директории.**

- `issue/`, `playbook/`, `pr/`, `analysis/`, `runbooks/`, `archive/`
    
- OKF не запрещает — мы добавляем


**Расширенный словарь `type`.**

- 26 типов
    
- Специфичны для нашей задачи
  

#### Ловушка

**Строгая OKF-конформность.**

Пытаться соответствовать **всем** требованиям OKF. Результат — `AGENTS.md` как `index.md`, отказ от `_*.md` префикса.

**Правильно:** OKF-вдохновлённый, а не строго конформный.

**Игнорирование OKF.**

«Свой формат лучше». Результат — изобретение велосипеда, потеря портативности.

**Правильно:** следовать духу OKF.

#### Что это НЕ значит

**Не значит «OKF — догма».** OKF можно нарушать, если есть причина.

**Не значит «OKF идеален».** У OKF есть слабости (нет стандартных типов, нет валидации).

**Не значит «нельзя отклоняться».** Можно. Мы отклоняемся в нескольких местах.

### 20.9 Как принципы связаны

Принципы не независимы. Они **поддерживают** друг друга.

```text

            Speed over quality
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    Минимализм  Ленивая      Локальность
        │       загрузка         │
        │           │           │
        └───────────┼───────────┘
                    ▼
            Ясность через структуру
                    │
                    ▼
                OKF как фундамент
```
**Пример:**

- **Speed** требует не тормозить на перфекционизме
    
- **Минимализм** требует не писать лишнее
    
- **Ленивая загрузка** требует не грузить всё
    
- **Локальность** требует не блокироваться на согласовании
    
- **Ясность** требует структурировать
    
- **OKF** даёт формат


Все вместе: **быстро, минимально, по запросу, локально, структурированно, по стандарту**.

### 20.10 Что философия НЕ решает

**Философия — не волшебство.** Она не решает:

- **Плохой код.** Bundle не улучшает качество
    
- **Слабое ревью.** Bundle не заменяет ревьюеров
    
- **Плохую архитектуру.** Bundle не переделывает систему
    
- **Отсутствие дисциплины.** Bundle не заставит его использовать


**Bundle — инструмент.** Как молоток. Хорош в умелых руках, бесполезен в неумелых.

### 20.11 Когда философия не работает

Философия **не универсальна**. Она может не подойти:

**Большие команды.**

Если 20+ разработчиков — нужны стандарты. Bundle локальный — не подходит для стандартизации.

**Строгие регуляторы.**

HIPAA, PCI-DSS. Нужны формальные процедуры, аудит, версии. Bundle — не для этого.

**Очень маленькие проекты.**

Скрипт на 100 строк. Bundle из 20 файлов — overkill.

**Короткие проекты.**

Прототип на неделю. Bundle не окупится.

**В этих случаях** — либо адаптируйте философию, либо используйте другой инструмент.

### 20.12 Как применять философию

**Три вопроса перед каждым решением:**

**1. Это ускоряет?**

- Если нет — почему делаете?
    
- Если да — делайте.


**2. Это минимально?**

- Можно ли меньше?
    
- Что можно удалить?
  

**3. Это ясно?**

- Поймёт ли коллега?
    
- Поймёт ли агент?
    
- Пойму ли я через год?
  

**Если три «да»** — решение правильное.

### 20.13 Что дальше

Part IV закончен. Мы разобрали:

- **Chapter 17.** `init-opencode`: install and update.
    
- **Chapter 18.** Extending the bundle.
    
- **Chapter 19.** Anti-patterns.
    
- **Chapter 20.** Philosophy.
  

Осталась **Part V — Appendices**:

- **A.** Type dictionary.
    
- **B.** Full bundle structure.
    
- **C.** OKF spec extract.
    
- **D.** FAQ.
    

Это справочники. Их можно читать выборочно.




====---=============================================================

## Что дальше в книге

Part IV готов. Осталось:

- **Part V — Appendices** (A–D).
    

Продолжаем с Appendix A — Type dictionary?






===---==============================================

## Что дальше в книге

Part IV — почти готов:

- **Chapter 17.** `init-opencode`: install and update ✅
    
- **Chapter 18.** Extending the bundle ✅
    
- **Chapter 19.** Anti-patterns ✅
    
- **Chapter 20.** Philosophy — следующий, последний в Part IV
    

Потом:

- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 20?






==---================================================================

## Что дальше в книге

Part IV — половина:

- **Chapter 17.** `init-opencode`: install and update ✅
    
- **Chapter 18.** Extending the bundle ✅
    
- **Chapter 19.** Anti-patterns — следующий
    
- **Chapter 20.** Philosophy
    

Потом:

- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 19?




==---============================================================================

## Что дальше в книге

Part IV — начали:

- **Chapter 17.** `init-opencode`: install and update ✅
    
- **Chapter 18.** Extending the bundle — следующий
    
- **Chapter 19.** Anti-patterns
    
- **Chapter 20.** Philosophy
    

Потом:

- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 18?










==---===========================================================================

## Что дальше в книге

Part III готов. Осталось:

- **Part IV — Operations** (главы 17–20).
    
- **Part V — Appendices** (A–D).


===---========================================

## Что дальше в книге

Part III — 5 из 6:

- **Chapter 11.** Issue lifecycle ✅
    
- **Chapter 12.** Session lifecycle ✅
    
- **Chapter 13.** When CI fails ✅
    
- **Chapter 14.** Incident in prod ✅
    
- **Chapter 15.** Deep analysis ✅
    
- **Chapter 16.** Planning — следующий, последний в Part III
    

Потом:

- **Part IV — Operations** (главы 17–20).
    
- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 16?


===---==============================================

## Что дальше в книге

Part III — 4 из 6:

- **Chapter 11.** Issue lifecycle ✅
    
- **Chapter 12.** Session lifecycle ✅
    
- **Chapter 13.** When CI fails ✅
    
- **Chapter 14.** Incident in prod ✅
    
- **Chapter 15.** Deep analysis — следующий
    
- **Chapter 16.** Planning
    

Потом:

- **Part IV — Operations** (главы 17–20).
    
- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 15?

===-===================================================
### 13.15 Что дальше

В следующей главе — **Incident in prod**. Это самый серьёзный workflow: прод упал, пользователи страдают, надо действовать быстро. Runbook — центральный инструмент.

---

## Что дальше в книге

Part III — половина:

- **Chapter 11.** Issue lifecycle ✅
    
- **Chapter 12.** Session lifecycle ✅
    
- **Chapter 13.** When CI fails ✅
    
- **Chapter 14.** Incident in prod — следующий
    
- **Chapter 15.** Deep analysis
    
- **Chapter 16.** Planning
    

Потом:

- **Part IV — Operations** (главы 17–20).
    
- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 14?


======---=================================================================

## Что дальше в книге

Part III — на 1/3:

- **Chapter 11.** Issue lifecycle ✅
    
- **Chapter 12.** Session lifecycle ✅
    
- **Chapter 13.** When CI fails — следующий
    
- **Chapter 14.** Incident in prod
    
- **Chapter 15.** Deep analysis
    
- **Chapter 16.** Planning
    

Потом:

- **Part IV — Operations** (главы 17–20).
    
- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 13?









===----------------------------================================

## Что дальше в книге

Part II готов. Осталось:

- **Part III — Workflows** (главы 11–16).
    
- **Part IV — Operations** (главы 17–20).
    
- **Part V — Appendices** (A–D).





===----------------------------================================

### 9.11 Что дальше
В следующей главе — **utility**: `index.md`, `log.md`, `SPEC_REFERENCE.md`, `.gitignore`, `.template-version`. Пять служебных файлов, которые не читаются при работе, но обеспечивают OKF-конформность и защиту.
---
## Что дальше в книге
Part II — почти готов:
- **Chapter 5.** AGENTS.md ✅
- **Chapter 6.** Onboarding ✅
- **Chapter 7.** Daily work ✅
- **Chapter 8.** Diagnostics ✅
- **Chapter 9.** Navigation & safety ✅
- **Chapter 10.** Utility — следующий, последний в Part II
Потом:
- **Part III — Workflows** (главы 11–16).
- **Part IV — Operations** (главы 17–20).
- **Part V — Appendices** (A–D).
Продолжаем с Chapter 10?



===-================================================================

## Что дальше в книге

Part II — на 80% готов:

- **Chapter 5.** AGENTS.md ✅
    
- **Chapter 6.** Onboarding ✅
    
- **Chapter 7.** Daily work ✅
    
- **Chapter 8.** Diagnostics ✅
    
- **Chapter 9.** Navigation & safety — следующий
    
- **Chapter 10.** Utility
    

Потом:

- **Part III — Workflows** (главы 11–16).
    
- **Part IV — Operations** (главы 17–20).
    
- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 9?

==------=============================================================
=======================================================================---

## Что дальше в книге

Мы прошли:

- **Part I — Foundations** (главы 1–4).
    
- **Part II** — начали. Главы 5 (AGENTS.md) и 6 (Onboarding) готовы.
    

Осталось в Part II:

- **Chapter 7.** Daily work: `_templates`, `_worklog`, `_backlog`, `_decisions`, `_codestyle`, `_commands`.
    
- **Chapter 8.** Diagnostics: `_ci`, `_troubleshooting`, `runbooks/`.
    
- **Chapter 9.** Navigation: `_files`, `_env`, `_security`, `analysis/`, `_meta`.
    
- **Chapter 10.** Utility: `index.md`, `log.md`, `SPEC_REFERENCE.md`, `.gitignore`, `.template-version`.
    

Потом:

- **Part III — Workflows** (главы 11–16).
    
- **Part IV — Operations** (главы 17–20).
    
- **Part V — Appendices** (A–D).
    

Продолжаем с Chapter 7 или хотите что-то поменять в темпе/структуре?