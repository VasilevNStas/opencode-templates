# init-opencode — подробное руководство

## Назначение

`init-opencode` устанавливает или обновляет шаблонную коллекцию файлов `.opencode/`
в проекте. Эта коллекция используется OpenCode (и другими AI-агентами) как
контекст: описывает архитектуру, правила, командный справочник и рабочие процессы.

После установки в `.opencode/` агенты могут читать контекст проекта, не нуждаясь
в полном доступе ко всей кодовой базе

---

## Установка скрипта

### Через symlink (рекомендуется)

```bash
# Клонируйте этот репозиторий куда-нибудь на диск
git clone https://github.com/anomalyco/opencode-templates ~/Projects/opencode-templates

# Создайте символическую ссылку в PATH
mkdir -p ~/.local/bin
ln -sf ~/Projects/opencode-templates/bin/init-opencode ~/.local/bin/init-opencode
```

### Через копирование

```bash
git clone https://github.com/anomalyco/opencode-templates ~/Projects/opencode-templates
cp ~/Projects/opencode-templates/bin/init-opencode ~/.local/bin/
chmod +x ~/.local/bin/init-opencode
```

### Проверка

```bash
init-opencode --help
```

Если видит вывод с usage — всё работает

---

## Переменная окружения OPENCODE_TEMPLATE_REPO

По умолчанию скрипт ищет шаблоны в `$HOME/Projects/opencode-templates/template/`
Если репозиторий лежит в другом месте, настройте переменную:

```bash
export OPENCODE_TEMPLATE_REPO=/path/to/your/opencode-templates
```

Можно добавить в `~/.zshrc` или `~/.bash_profile`, чтобы работало всегда

---

## Режимы работы

### 1. Установка (по умолчанию)

Скопирует **все** файлы из `template/` в `.opencode/` целевого проекта

```bash
init-opencode ~/Projects/my-app
```

Что происходит:

1. Проверяет что `template/` существует
2. Если `.opencode/` уже есть — бэкапит его с датой: `.opencode.bak.20260923-143022/`
3. Копирует все `.md` файлы из `template/`
4. Создаёт пустые поддиректории: `issue/`, `playbook/`, `pr/`, `analysis/`, `runbooks/`, `archive/`
5. Записывает `.template-version` с информацией о версии и дате установки

### 2. Установка с анализом (--analyze)

То же самое, но подсказывает агенту автоматически запустить заполнение шаблона

```bash
init-opencode --analyze ~/Projects/my-app
```

Агент получит инструкцию проанализировать репозиторий и заполнить все шаблоны
реальными данными вместо заглушек типа `<full build command>`

### 3. Обновление (--update)

Обновит только те файлы, которые менялись в шаблоне. **Не тронет** пользовательские данные

```bash
init-opencode --update ~/Projects/my-app
```

Как работает обновление:

| Файл                              | Поведение              | Почему                                         |
| --------------------------------- | ---------------------- | ----------------------------------------------- |
| AGENTS.md                         | ✅ Обновится            | Шаблон рабочей логики — должен оставаться актуальным |
| index.md                          | ✅ Обновится            | Оглавление bundle                               |
| _security.md                      | ✅ Обновится            | Политика безопасности — критично держать свежей  |
| _templates.md                     | ✅ Обновится            | Шаблоны PR/issue/playbook                       |
| analysis/_finding.md             | ✅ Обновится            | Шаблон для findings                             |
| WORK_LOG.md                       | ❌ Не обновится         | Содержит историю ваших сессий                   |
| _backlog.md                       | ❌ Не обновится         | Пользовательский бэклог                           |
| _concepts.md                      | ❌ Не обновится         | Заполняется под конкретный проект                |
| _decisions.md                     | ❌ Не обновится         | ADR-журнал решений команды                       |

Проверка различий:

```bash
init-opencode --diff ~/Projects/my-app
```

Покажет diff для каждого файла перед тем как обновлять. Удобно проверить
что именно изменится

---

## Структура установленных файлов

После `init-opencode ~/Projects/my-app`:

```
my-app/.opencode/
├── AGENTS.md                 ← Главная точка входа
├── WORK_LOG.md               ← История рабочих сессий
├── index.md                  ← Оглавление bundle
├── log.md                    ← Указатель на changelog
├── SPEC_REFERENCE.md         ← Выдержка из OKF spec
├── _setup.md                 ← Как поднять проект локально
├── _concepts.md              ← Архитектура системы
├── _env.md                   ← Карта окружений (dev/staging/prod)
├── _codestyle.md             ← Правила код-стайла
├── _commands.md              ← Справочник команд
├── _files.md                 ← Где что в кодовой базе
├── _glossary.md              ← Глоссарий доменных терминов
├── _security.md              ← Политика безопасности
├── _troubleshooting.md       ← Частые проблемы и решения
├── _decisions.md             ← ADR (Architecture Decision Records)
├── _backlog.md               ← Планируемые задачи
├── _worklog.md               ← Шаблон записи в лог сессий
├── _templates.md             ← Шаблоны для PR/issue/playbook
├── _ci.md                    ← CI воркфлоу и диагностика сбоев
├── _meta.md                  ← Организация самого bundle
├── _testing.md               ← Стратегия тестирования
├── _api.md                   ← API справочник (если есть публичный API)
├── _release.md               ← Процесс релизов
├── _performance.md           ← Мониторинг производительности
├── issue/                    ← Пусто (заполняется при работе над issue)
├── playbook/                 ← Пусто (стратегии решения сложных задач)
├── pr/                       ← Пусто (описания pull requests)
├── analysis/                 ← Пусто (результаты глубокого анализа)
├── runbooks/                 ← Пусто (процедуры реагирования на инциденты)
└── archive/                  ← Пусто (архив завершённых issue)
```

Каждый файл начинается с YAML frontmatter:

```yaml
---
type: commands
title: "Quick Commands"
description: "Build, test, run, and utility command reference"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [commands, reference]
---
```

Это формат **OKF v0.2** (Open Knowledge Format). Он позволяет AI-агенту
структурировано понимать каждый документ без чтения полного текста.

---

## Типичные сценарии использования

### Новый проект

```bash
cd ~/Projects/my-app
init-opencode .
# Откройте OpenCode и скажите: "Analyze the repository and fill in the template."
```

### Проект с уже установленным bundle (например от коллеги)

```bash
init-opencode --analyze .
```

Установит свежие шаблоны и сразу запустит заполнение.

### Шаблон обновили (добавили новые файлы или изменили структуру)

```bash
# Сначала посмотрите что изменится
init-opencode --diff .

# Потом обновите
init-opencode --update .
```

### Обновился OS, потерялся PATH

```bash
# Пересоздайте symlink
rm -f ~/.local/bin/init-opencode
ln -s ~/Projects/opencode-templates/bin/init-opencode ~/.local/bin/init-opencode
```

---

## Часто задаваемые вопросы

### Q: Могу ли я редактировать файлы после установки?

Да. Все редактирования безопасны. Единственное предупреждение: `--update`
НЕ перезапишет файлы из списка NEVER_OVERWRITE (`WORK_LOG.md`, `_backlog.md`,
`_concepts.md`, `_decisions.md`). Это сделано намеренно — эти файлы содержат
пользовательские данные.

### Q: Что если я случайно удалил .opencode/?

Не страшно:

```bash
# Найдите последнюю резервную копию
ls -la ~/.opencode.bak.*  # нет, это в корне проекта:
ls -la /Users/vstas/AI_Projects/my-app/.opencode.bak.*

# Или просто переустановите
init-opencode .
```

Бэкапы сохраняются с timestamp. Старые можно удалить вручную через пару недель.

### Q: Can I add my own files?

Yes! Just put them directly in `.opencode/`. The update mechanism only touches
files listed in ALWAYS_OVERWRITE. Your custom files stay untouched.

### Q: How do I know if my bundle is up-to-date?

Check `.opencode/.template-version`:

```bash
cat .opencode/.template-version
# version: 0.2.0
# installed: 2026-09-22
# source: /Users/vstas/Projects/opencode-templates
```

Compare `version` with contents of `/Users/vstas/Projects/opencode-templates/VERSION`.
If they differ, run `init-opencode --update .`.

### Q: What's the difference between install and update?

| Команда   | Делает                                       | Кого трогаем              | Когда использовать          |
| --------- | ------------------------------------------- | ------------------------ | -------------------------- |
| `install` | Полная замена, с бэкапом старого bundle      | Все файлы                | При первом запуске          |
| `update`  | Только изменение, старые данные не трогаем    | ONLY_ALWAYS_OVERWRITE    | После изменений в шаблоне   |
| `diff`    | Покажет что поменяется без записи            | N/A                      | Предпросмотр перед обновлением |

### Q: Why does --update skip certain files?

Files like `WORK_LOG.md` accumulate project history. Overwriting them would
erase your session records, decisions made over months, and backlog items.

These are intentionally classified as user-owned vs. template-owned.
Template-owned files (like `_templates.md` or `AGENTS.md`) serve structural
purposes — their job is to guide AI agents correctly, so staying current
matters more than preserving any local modifications.

### Q: How does agent lazy loading work?

When an AI-agent starts a session, it reads `AGENTS.md` first (~70 lines).
That file contains a table mapping topic → filename. Agent only loads the
specific files it needs for the current task, not all 25+ files at once.

This saves tokens and keeps context manageable. For example:
- Working on a failing test? Read `_testing.md` + `_ci.md`
- Setting up a new dev environment? Read `_setup.md` + `_commands.md`
- Troubleshooting locally? Read `_troubleshooting.md` + `_env.md`

---

## Совместимость

Формат совместим с OKF v0.1 и v0.2. Потребители, ориентированные на v0.1,
будут видеть только `timestamp` → `generated.at` (fallback), `sources:`
игнорируются, но не ломают чтение.

Потребители OKF v0.2 получают дополнительные сигналы доверия:
- `generated.by/at` — кто и когда создал документ
- `status: stable/draft` — жизненный цикл концепта
- `sources` — provenance с credibility signals (author, last_modified)
