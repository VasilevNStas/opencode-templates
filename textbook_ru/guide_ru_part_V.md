---
type: book
title: "Knowledge Bundles for AI Agents, part V"
description: "A practical guide to OKF and agent-ready codebases"
timestamp: 2026-09-23
tags: [okf, agents, opencode, guide]
---

# Part V — Appendices

_[Part I](../textbook_ru/guide_ru_part_I.md), [Part II](../textbook_ru/guide_ru_part_II.md), [Part III](../textbook_ru/guide_ru_part_III.md), [Part IV](../textbook_ru/guide_ru_part_IV.md) — это основное содержание. Appendices — справочники. Их можно читать выборочно, использовать как reference. Не обязательно читать подряд._

_Четыре приложения: словарь типов, полная структура bundle, выдержка из OKF, FAQ._

---
## Appendix A. Type dictionary

### A.1 Что такое `type`

**`type`** — обязательное поле OKF frontmatter. Строка, идентифицирующая **вид концепта**.

```yaml
---
type: architecture
title: "..."
---
```
Агент по `type` понимает, **что за файл** перед ним, не читая тело
### A.2 Правила именования

**Используйте:**
- **Существительные.** `architecture`, `finding`, `playbook`; Не глаголы
- **Единственное число.** `runbook`, не `runbooks`
- **Нижний регистр.** `project-context`, не `ProjectContext`
- **Дефисы для составных.** `analysis-index`, `runbook-index`
- **Короткие имена.** Одно слово где возможно

**Не используйте:**
- Глаголы: `log-decision` — плохо
- Общие слова: `file`, `doc`, `thing`
- Сокращения без причины: `pdca`, `okr` — если они не общеприняты
- Версии: `architecture-v2` — не надо
- Множественное число: `findings` — используйте `finding`
### A.3 Полный словарь

26 значений. Сгруппированы по назначению.
#### Core

| `type`            | File                                                 | Purpose                           |
| ----------------- | ---------------------------------------------------- | --------------------------------- |
| `project-context` | [`AGENTS.md`](../template/AGENTS.md)                 | Точка входа, контекст проекта     |
| `index`           | [`index.md`](../template/index.md)                   | OKF-индекс bundle                 |
| `log`             | [`log.md`](../template/log.md)                       | Указатель на хронологические логи |
| `meta`            | [`_meta.md`](../template/_meta.md)                   | Метаданные bundle                 |
| `spec-reference`  | [`SPEC_REFERENCE.md`](../template/SPEC_REFERENCE.md) | Выдержка из OKF                   |

#### Onboarding

| `type`         | File                                       | Purpose          |
| -------------- | ------------------------------------------ | ---------------- |
| `setup`        | [`_setup.md`](../template/_setup.md)       | Локальный запуск |
| `architecture` | [`_concepts.md`](../template/_concepts.md) | Архитектура      |
| `glossary`     | [`_glossary.md`](../template/_glossary.md) | Доменные термины |

#### Daily work

| `type`         | File                                                                               | Purpose          |
| -------------- | ---------------------------------------------------------------------------------- | ---------------- |
| `templates`    | [`_templates.md`](../template/_templates.md)                                       | Шаблоны issue/PR |
| `worklog`      | [`_worklog.md`](../template/_worklog.md), [`WORK_LOG.md`](../template/WORK_LOG.md) | Сессии           |
| `backlog`      | [`_backlog.md`](../template/_backlog.md)                                           | Будущие задачи   |
| `decision-log` | [`_decisions.md`](../template/_decisions.md)                                       | ADR              |
| `codestyle`    | [`_codestyle.md`](../template/_codestyle.md)                                       | Стиль            |
| `commands`     | [`_commands.md`](../template/_commands.md)                                         | Команды          |

#### Diagnostics

| `type`            | File                                                     | Purpose              |
| ----------------- | -------------------------------------------------------- | -------------------- |
| `ci`              | [`_ci.md`](../template/_ci.md)                           | CI workflows         |
| `troubleshooting` | [`_troubleshooting.md`](../template/_troubleshooting.md) | Локальные проблемы   |
| `runbook-index`   | [`runbooks/index.md`](../template/runbooks/index.md)     | Индекс runbook'ов    |
| `runbook`         | `runbooks/*.md`                                          | Процедуры инцидентов |

#### Navigation & safety

| `type`           | File                                                 | Purpose              |
| ---------------- | ---------------------------------------------------- | -------------------- |
| `files`          | [`_files.md`](../template/_files.md)                 | Карта файлов         |
| `env`            | [`_env.md`](../template/_env.md)                     | Карта окружений      |
| `security`       | [`_security.md`](../template/_security.md)           | Правила безопасности |
| `analysis-index` | [`analysis/index.md`](../template/analysis/index.md) | Индекс findings      |
| `finding`        | `analysis/F-*.md`                                    | Конкретный finding   |

#### Issue lifecycle

| `type`            | File                           | Purpose           |
| ----------------- | ------------------------------ | ----------------- |
| `project-summary` | `issue/PROJECT_SUMMARY_<N>.md` | Фиксация issue    |
| `playbook`        | `playbook/PLAYBOOK_<N>.md`     | Стратегия решения |
| `pr`              | `pr/PR_<N>.md`                 | Описание PR       |

### A.4 Разбор каждого типа

#### `project-context`

**Файл:** [`AGENTS.md`](../template/AGENTS.md)

**Назначение:** точка входа в bundle. Контекст проекта + навигация
**Когда используется:** один раз на bundle. Это **главный** файл
**Не путать с:**
- `meta` — тот про bundle, `project-context` — про проект
- `architecture` — тот описывает систему, `project-context` — только ссылается

**Ключевые секции:** Overview, Key components, Issue workflow, Reference files
**Размер:** 60–90 строк
#### `index`

**Файл:** [`index.md`](../template/index.md)

**Назначение:** OKF-индекс. Точка входа bundle по спеке
**Когда используется:** один раз на bundle. **Резервированное имя OKF**
**Не путать с:**
- `project-context` — тот основной, `index` — формальный
- `*-index` (analysis-index, runbook-index) — те индексы директорий

**Ключевые секции:** Entry point, Reference files, Dynamic artifacts.
**Размер:** ~35 строк
#### `log`

**Файл:** [`log.md`](../template/_log.md)

**Назначение:** указатель на хронологические логи. **Резервированное имя OKF**
**Когда используется:** один раз на bundle
**Особенность:** не хранит лог сам, а ссылается на `WORK_LOG.md`, `_decisions.md`, `_meta.md`

**Ключевые секции:** Log pointers, What this file is for
**Размер:** ~20 строк
#### `meta`

**Файл:** [`_meta.md`](../template/_meta.md)

**Назначение:** метаданные bundle. Что это, откуда, как обновлять
**Когда используется:** один раз на bundle
**Не путать с:**
- `project-context` — тот про проект, `meta` — про bundle
- `spec-reference` — тот про OKF, `meta` — про конкретный bundle

**Ключевые секции:** What's here, Template version, OKF base + extensions, How to update, Rules
**Размер:** 60–70 строк
#### `spec-reference`

**Файл:** [`SPEC_REFERENCE.md`](../template/SPEC_REFERENCE.md)

**Назначение:** выдержка из OKF v0.1. Офлайн-доступ к спеке
**Когда используется:** один раз на bundle
**Не путать с:**
- `meta` — тот про конкретный bundle
- `project-context` — тот про проект
**Особенность:** английский язык (для консистентности с LLM)

**Ключевые секции:** What OKF is, Bundle structure, Reserved names, Concept document, Conformance
**Размер:** ~120 строк
#### `setup`

**Файл:** [`_setup.md`](../template/_setup.md)

**Назначение:** локальный запуск проекта с нуля
**Когда используется:** один раз на проект
**Не путать с:**
- `env` — тот про удалённые окружения
- `commands` — тот про команды после setup

**Ключевые секции:** Prerequisites, Clone, Configuration, Database, Run, Verify, First-day reading order
**Размер:** ~60 строк
#### `architecture`

**Файл:** [`_concepts.md`](../template/_concepts.md)

**Назначение:** как устроен проект
**Когда используется:** один раз на проект, обновляется при изменениях
**Не путать с:**
- `decision-log` — тот про «почему так», `architecture` — про «что»
- `files` — тот про файлы, `architecture` — про компоненты

**Ключевые секции:** Overview, Key components, Data flow, Key patterns, Deployment, Error handling, Testing strategy
**Размер:** 60–100 строк
#### `glossary`

**Файл:** [`_glossary.md`](../template/_glossary.md)

**Назначение:** доменные термины, аббревиатуры, синонимы
**Когда используется:** один раз на проект, обновляется при появлении терминов
**Не путать с:**
- `architecture` — тот описывает компоненты, `glossary` — определяет термины
- Общеизвестные термины (HTTP, JSON) — не сюда

**Ключевые секции:** Terms, Abbreviations, Synonyms
**Размер:** 20–60 строк
#### `templates`

**Файл:** [`_templates.md`](../template/_templates.md)

**Назначение:** шаблоны PROJECT_SUMMARY, PLAYBOOK, PR
**Когда используется:** один раз на проект
**Не путать с:**
- Сами динамические файлы (`PROJECT_SUMMARY_<N>.md`) — это не `templates`, а `project-summary`
- `project-context` — Issue workflow секция там, здесь — формы

**Ключевые секции:** File locations, PROJECT_SUMMARY, PLAYBOOK, PR description, Lifecycle
**Размер:** 100–130 строк
#### `worklog`

**Файл:** [`_worklog.md`](../template/_worklog.md), [`WORK_LOG.md`](../template/WORK_LOG.md)

**Назначение:** шаблон и правила + сами записи
**Когда используется:** `_worklog.md` — один раз. `WORK_LOG.md` — растёт
**Не путать с:**
- `decision-log` — тот про значимые решения, `worklog` — про все сессии
- `log` — OKF-указатель, `worklog` — реальные данные

**Ключевые секции:** Entry format, Rules, How to add a new entry
**Размер:** `_worklog.md` — 60–80 строк. `WORK_LOG.md` — растёт
#### `backlog`

**Файл:** [`_backlog.md`](../template/_backlog.md)

**Назначение:** inbox для будущих задач
**Когда используется:** один раз, обновляется при появлении/завершении задач
**Не путать с:**
- GitHub Issues — те подтверждённые, backlog — черновики
- `decision-log` — тот про «почему», `backlog` — про «что делать»
- Roadmap — отдельный документ

**Ключевые секции:** Priorities, Items, Ideas, Tech debt
**Размер:** 60–100 строк
#### `decision-log`

**Файл:** [`_decisions.md`](../template/_decisions.md)

**Назначение:** журнал архитектурных решений (ADR)
**Когда используется:** один раз, обновляется при значимых решениях
**Не путать с:**
- `worklog` — тот про все сессии, `decision-log` — только значимые решения
- «Decision» секция в WORK_LOG — короткая ссылка, здесь — полный ADR

**Ключевые секции:** Index, ADR-XXX записи (Context, Decision, Alternatives, Consequences)
**Размер:** растёт. 30–50 строк на ADR
#### `codestyle`

**Файл:** [`_codestyle.md`](../template/_codestyle.md)

**Назначение:** стиль кода, lint, тестирование (тактика)
**Когда используется:** один раз, обновляется при изменениях
**Не путать с:**
- `architecture` — тот про стратегию тестирования, `codestyle` — про тактику
- `commands` — тот про команды после setup, `codestyle` — про lint как gate

**Ключевые секции:** SPDX headers, Lint, Language conventions, Testing
**Размер:** 40–50 строк
#### `commands`

**Файл:** [`_commands.md`](../template/_commands.md)

**Назначение:** шпаргалка команд
**Когда используется:** один раз, обновляется при изменениях
**Не путать с:**
- `setup` — тот пошаговая инструкция, `commands` — каталог
- `codestyle` — lint как правило там, здесь — lint как команда

**Ключевые секции:** Build, Test, Lint, Run, Utilities
**Размер:** 35–45 строк
#### `ci`

**Файл:** [`_ci.md`](../template/_ci.md)

**Назначение:** диагностика CI-провалов
**Когда используется:** один раз, обновляется при изменениях workflows
**Не путать с:**
- `troubleshooting` — тот про локально, `ci` — про CI
- `runbook` — тот про прод, `ci` — про CI

**Ключевые секции:** Workflows, Common failures, Local reproduction, When CI passes locally but fails remotely
**Размер:** 60–80 строк
#### `troubleshooting`

**Файл:** [`_troubleshooting.md`](../template/_troubleshooting.md)

**Назначение:** локальные проблемы и их решения
**Когда используется:** один раз, активно растёт
**Не путать с:**
- `ci` — тот про CI, `troubleshooting` — про локально
- `runbook` — тот про прод, `troubleshooting` — про локально

**Ключевые секции:** Quick index, секции симптомов (Symptom/Cause/Fix), Still stuck
**Размер:** 60–100 строк, растёт
#### `runbook-index`

**Файл:** [`runbooks/index.md`](../template/runbooks/index.md)

**Назначение:** индекс runbook'ов
**Когда используется:** один раз
**Не путать с:**
- `index` — тот OKF-индекс bundle, `runbook-index` — только runbook'ов
- `analysis-index` — тот про findings

**Ключевые секции:** If you're in the middle of an incident, Index, Creating a new runbook
**Размер:** ~30 строк
#### `runbook`

**Файл:** `runbooks/<scenario>.md`

**Назначение:** процедура реагирования на инцидент
**Когда используется:** много раз. Один на сценарий
**Не путать с:**
- `troubleshooting` — тот локально, `runbook` — про прод
- `postmortem` — тот разбор, `runbook` — процедура

**Ключевые секции:** When to use, Prerequisites, Steps, Verification, If it doesn't work, Rollback, Post-incident
**Размер:** 60–100 строк
#### `files`

**Файл:** [`_files.md`](../template/_files.md)

**Назначение:** карта ключевых файлов
**Когда используется:** один раз, обновляется при изменениях структуры
**Не путать с:**
- `architecture` — тот про компоненты, `files` — про файлы
- `codestyle` — тот про конвенции имён, `files` — про существующие файлы

**Ключевые секции:** Entry points, Core modules, Data layer, Configuration, Tests
**Размер:** 40–60 строк
#### `env`

**Файл:** [`_env.md`](../template/_env.md)

**Назначение:** карта окружений (dev, staging, prod)
**Когда используется:** один раз, обновляется при изменениях
**Не путать с:**
- `setup` — тот про локально, `env` — про удалённые
- `security` — тот про секреты, `env` — про окружения

**Ключевые секции:** Overview, Where things live, Access, Deploy, Prod restrictions
**Размер:** 60–80 строк
#### `security`

**Файл:** [`_security.md`](../template/_security.md)

**Назначение:** правила работы с секретами
**Когда используется:** один раз, обновляется при изменениях
**Не путать с:**
- `SECURITY.md` в корне — тот публичный, `_security.md` — внутренний
- `env` — тот про окружения, `security` — про правила

**Ключевые секции:** Hard rules, Where secrets live, What's safe to commit, What must NEVER be committed, If a secret leaked, Reporting vulnerabilities.
**Размер:** 60 строк

#### `analysis-index`

**Файл:** [`analysis/index.md`](../template/analysis/index.md)

**Назначение:** индекс findings
**Когда используется:** один раз, обновляется при добавлении findings
**Не путать с:**
- `index` — тот OKF-индекс bundle, `analysis-index` — про findings
- `runbook-index` — тот про runbook'и

**Ключевые секции:** Reports, Findings summary
**Размер:** 30–40 строк
#### `finding`

**Файл:** `analysis/F-XXX-<name>.md`

**Назначение:** одна находка анализа
**Когда используется:** много раз. Один на находку
**Не путать с:**
- `project-summary` — тот про issue, `finding` — про найденную проблему
- `backlog` items — те задачи, `finding` — состояние

**Ключевые секции:** Summary, Evidence, Impact, Recommendation, Effort estimate, Related
**Размер:** 25–40 строк
#### `project-summary`

**Файл:** `issue/PROJECT_SUMMARY_<N>.md`

**Назначение:** фиксация работы над issue
**Когда используется:** по одному на issue. Активные — в `issue/`, завершённые — в `archive/`
**Не путать с:**
- `playbook` — тот стратегия, `project-summary` — фиксация
- `pr` — тот для PR, `project-summary` — для issue

**Ключевые секции:** Issue Overview, Problem, Solution, Verification, Key Discoveries, Files Changed.
**Размер:** 30–50 строк
#### `playbook`

**Файл:** `playbook/PLAYBOOK_<N>.md`

**Назначение:** стратегия решения issue
**Когда используется:** по одному на issue (если нужно). Активные — в `playbook/`, завершённые — в `archive/`
**Не путать с:**
- `project-summary` — тот фиксация, `playbook` — план
- `runbook` — тот про инцидент, `playbook` — про issue

**Ключевые секции:** Context, Strategy, Patterns Used, Known Pitfalls, Verification Commands.
**Размер:** 25–40 строк.
#### `pr`

**Файл:** `pr/PR_<N>.md`

**Назначение:** описание pull request.
**Когда используется:** по одному на PR. Активные — в `pr/`, завершённые — в `archive/`.
**Не путать с:**
- `project-summary` — тот для issue, `pr` — для PR
- Frontmatter может мешать на GitHub — убирать перед вставкой

**Ключевые секции:** Description, Related Issue, Changes, Verification, Notes for Reviewers
**Размер:** 20–30 строк
### A.5 Когда добавлять новый тип

**Триггеры:**
1. **Новый файл** в bundle, которому не подходит существующий тип
2. **Новая категория** знаний (например, `postmortem`)
3. **Новая директория** с своими артефактами

**НЕ добавляйте, если:**
- Есть близкий тип. `summary` vs `project-summary` — используйте существующий
- Это разовый файл. Не плодите типы ради одного
- Можно обойтись существующим. `postmortem` vs `finding` — если post-mortem'ов мало

### A.6 Процесс добавления нового типа

**Шаг 1: выбрать имя**
- Существительное
- Единственное число
- Нижний регистр
- Дефисы для составных
**Шаг 2: использовать в frontmatter.**
**Шаг 3: обновить словарь** в README.
**Шаг 4: обновить `_meta.md`** — в таблице `OKF base + extensions` → `Custom type values`.
**Шаг 5: обновить `VERSION`.** Minor bump.
**Шаг 6: обновить `AGENTS.md`** — если новый файл добавляется в reference files.
**Шаг 7: обновить `init-opencode`** — если новый файл.

### A.7 Примеры возможных новых типов

**`api`** — для `_api.md`. Публичный API.
**`deploy`** — для `_deploy.md`. Процесс деплоя.
**`release`** — для `_release.md`. Процесс релиза.
**`performance`** — для `_performance.md`. Характеристики.
**`monitoring`** — для `_monitoring.md`. Метрики, алерты.
**`compliance`** — для `_compliance.md`. GDPR, SOC2.
**`postmortem`** — для `incidents/<date>.md`. Разбор инцидента.
**`experiment`** — для `experiments/<name>.md`. A/B-тест.
**`research`** — для `research/<topic>.md`. Исследование.
**`metrics`** — для `metrics/<date>.md`. Замер производительности.
**`report`** — для `analysis/<audit>.md`. Большой отчёт анализа.

### A.8 Толерантность к неизвестным типам

**OKF требует:** потребители **не должны** отвергать bundle из-за неизвестных `type`
Это значит:
- Если чужой агент увидит ваш `type: my-custom`, он не должен падать
- Он обработает как «неизвестный тип» — прочитает как обычный markdown

**На практике:** большинство агентов толерантны. Но:
- Используйте **осмысленные** имена
- Документируйте типы в `_meta.md`
- Не полагайтесь на специфичные типы для критичной логики
### A.9 Что дальше

В следующем приложении — **Full bundle structure**. Полная карта файлов и директорий с описанием каждого.

---
## Appendix B. Full bundle structure

### B.1 Назначение приложения

Это **справочник по структуре**. Полная карта `.opencode/`: что где лежит, для чего, какие размеры.
Используйте как reference. Не для чтения подряд.
### B.2 Полное дерево

```text
.opencode/
│
├── .gitignore                    ← защита от коммита
├── .template-version             ← версия шаблона (machine-readable)
│
├── index.md                      ← OKF entry point
├── log.md                        ← OKF log pointer
├── AGENTS.md                     ← главный файл
├── SPEC_REFERENCE.md             ← выдержка из OKF v0.1
├── _meta.md                      ← мета о bundle
│
├── _concepts.md                  ← архитектура
├── _setup.md                     ← локальный запуск
├── _env.md                       ← карта окружений
├── _codestyle.md                 ← стиль
├── _commands.md                  ← шпаргалка команд
├── _files.md                     ← карта файлов
├── _glossary.md                  ← доменные термины
├── _security.md                  ← работа с секретами
├── _troubleshooting.md           ← локальные проблемы
├── _decisions.md                 ← ADR-журнал
├── _backlog.md                   ← будущие задачи
├── _worklog.md                   ← шаблон WORK_LOG
├── _templates.md                 ← шаблоны issue/PR
├── _ci.md                        ← CI-workflow
│
├── WORK_LOG.md                   ← создаётся при первой сессии
│
├── issue/                        ← активные PROJECT_SUMMARY
├── playbook/                     ← активные PLAYBOOK
├── pr/                           ← черновики PR-описаний
│
├── analysis/                     ← находки анализа
│   ├── index.md
│   ├── _finding.md
│   └── F-XXX-*.md
│
├── runbooks/                     ← процедуры инцидентов
│   ├── index.md
│   ├── _runbook.md
│   └── <scenario>.md
│
└── archive/                      ← завершённые issue/playbook/pr
    └── <N>/
        ├── PROJECT_SUMMARY_<N>.md
        ├── PLAYBOOK_<N>.md
        └── PR_<N>.md
```
### B.3 Разбор по директориям

#### `.opencode/` — корень bundle

Всё, что внутри — часть bundle. Всё, что снаружи — не часть
**Локальный.** В `.git/info/exclude`
**Самодостаточный.** Содержит всё нужное для работы
#### `issue/` — активные issue

**Что лежит:** `PROJECT_SUMMARY_<N>.md` для issue в работе
**Сколько:** обычно 1–3. Больше — multitasking, плохо
**Когда создаётся:** Start issue lifecycle
**Когда очищается:** после merge PR → `archive/`
**Не коммитится:** в `.opencode/.gitignore`
#### `playbook/` — активные playbook'и

**Что лежит:** `PLAYBOOK_<N>.md` для issue в работе
**Сколько:** столько же, сколько issue. Или меньше — playbook не обязателен
**Когда создаётся:** Start issue lifecycle (если задача нетривиальная)
**Когда очищается:** после merge PR → `archive/`
#### `pr/` — активные PR

**Что лежит:** `PR_<N>.md` — черновики описаний PR
**Сколько:** столько же, сколько открытых PR
**Когда создаётся:** End issue lifecycle, перед открытием PR
**Когда очищается:** после merge → `archive/`
#### `analysis/` — находки анализа

**Что лежит:**
- `index.md` — сводная таблица
- `_finding.md` — шаблон
- `F-XXX-<name>.md` — конкретные findings

**Сколько:** растёт по мере анализа.
**Особенность:** `index.md` и `_finding.md` — часть bundle, коммитятся. Findings — локальные.
#### `runbooks/` — процедуры инцидентов

**Что лежит:**
- `index.md` — индекс
- `_runbook.md` — шаблон
- `<scenario>.md` — конкретные процедуры

**Сколько:** 5–15 обычно.
**Особенность:** `index.md` и `_runbook.md` — часть bundle, коммитятся. Конкретные runbook'и — обычно локальные, но могут быть публичными.
#### `archive/` — завершённые артефакты

**Что лежит:** `PROJECT_SUMMARY`, `PLAYBOOK`, `PR` для завершённых issue
**Структура:** обычно `<N>/` поддиректория для каждой issue. Или плоско, без поддиректорий — на ваш выбор
**Сколько:** растёт бесконечно. Ревизия раз в год
### B.4 Файлы — сводная таблица

| Файл                  | `type`            | Роль               | Размер  | Коммитится |
| --------------------- | ----------------- | ------------------ | ------- | ---------- |
| `AGENTS.md`           | `project-context` | Точка входа        | 60–90   | Да         |
| `index.md`            | `index`           | OKF-индекс         | 35      | Да         |
| `log.md`              | `log`             | OKF-указатель      | 20      | Да         |
| `SPEC_REFERENCE.md`   | `spec-reference`  | Выдержка OKF       | 120     | Да         |
| `_meta.md`            | `meta`            | Мета bundle        | 60–70   | Да         |
| `_concepts.md`        | `architecture`    | Архитектура        | 60–100  | Да         |
| `_setup.md`           | `setup`           | Локальный запуск   | 60      | Да         |
| `_env.md`             | `env`             | Окружения          | 60–80   | Да         |
| `_codestyle.md`       | `codestyle`       | Стиль              | 40–50   | Да         |
| `_commands.md`        | `commands`        | Команды            | 35–45   | Да         |
| `_files.md`           | `files`           | Карта файлов       | 40–60   | Да         |
| `_glossary.md`        | `glossary`        | Термины            | 20–60   | Да         |
| `_security.md`        | `security`        | Секреты            | 60      | Да         |
| `_troubleshooting.md` | `troubleshooting` | Локальные проблемы | 60–100  | Да         |
| `_decisions.md`       | `decision-log`    | ADR                | растёт  | Да         |
| `_backlog.md`         | `backlog`         | Будущие задачи     | 60–100  | Да         |
| `_worklog.md`         | `worklog`         | Шаблон WORK_LOG    | 60–80   | Да         |
| `_templates.md`       | `templates`       | Шаблоны            | 100–130 | Да         |
| `_ci.md`              | `ci`              | CI                 | 60–80   | Да         |
| `WORK_LOG.md`         | `worklog`         | Сессии             | растёт  | Нет        |
| `.gitignore`          | —                 | Защита             | 20      | Да         |
| `.template-version`   | —                 | Версия             | 3       | Нет        |

### B.5 Динамические артефакты — сводная таблица

| Файл                           | `type`            | Роль               | Где       | Коммитится |
| ------------------------------ | ----------------- | ------------------ | --------- | ---------- |
| `issue/PROJECT_SUMMARY_<N>.md` | `project-summary` | Issue snapshot     | issue/    | Нет        |
| `playbook/PLAYBOOK_<N>.md`     | `playbook`        | Strategy           | playbook/ | Нет        |
| `pr/PR_<N>.md`                 | `pr`              | PR description     | pr/       | Нет        |
| `analysis/index.md`            | `analysis-index`  | Findings index     | analysis/ | Да         |
| `analysis/_finding.md`         | `finding`         | Finding template   | analysis/ | Да         |
| `analysis/F-XXX-*.md`          | `finding`         | Specific finding   | analysis/ | Нет        |
| `runbooks/index.md`            | `runbook-index`   | Runbooks index     | runbooks/ | Да         |
| `runbooks/_runbook.md`         | `runbook`         | Runbook template   | runbooks/ | Да         |
| `runbooks/<scenario>.md`       | `runbook`         | Specific runbook   | runbooks/ | Зависит    |
| `archive/<N>/*.md`             | varies            | Archived artifacts | archive/  | Нет        |

### B.6 Жизненный цикл файлов

#### Создаётся один раз
- `AGENTS.md`
- `index.md`, `log.md`
- `SPEC_REFERENCE.md`
- `_meta.md`
- `_setup.md`, `_concepts.md`, `_glossary.md`
- `_codestyle.md`, `_commands.md`, `_files.md`
- `_env.md`, `_security.md`
- `_troubleshooting.md`, `_ci.md`
- `_worklog.md`, `_templates.md`
- `.gitignore`, `.template-version`
- `analysis/index.md`, `analysis/_finding.md`
- `runbooks/index.md`, `runbooks/_runbook.md`

#### Создаётся при первой сессии
- `WORK_LOG.md`
#### Растёт постоянно
- `WORK_LOG.md` — после каждой сессии
- `_troubleshooting.md` — после каждой проблемы (>10 мин)
- `_ci.md` — после каждой новой CI-ошибки
- `_backlog.md` — при появлении/завершении задач
- `_decisions.md` — при значимых решениях
#### Создаётся на issue
- `issue/PROJECT_SUMMARY_<N>.md` — Start
- `playbook/PLAYBOOK_<N>.md` — Start
- `pr/PR_<N>.md` — End
#### Создаётся при анализе
- `analysis/F-XXX-*.md` — при находке
#### Создаётся при инциденте
- `runbooks/<scenario>.md` — после инцидента (новый runbook)
#### Удаляется / архивируется
- `issue/*` — после merge → `archive/`
- `playbook/*` — после merge → `archive/`
- `pr/*` — после merge → `archive/`
- `analysis/F-XXX-*.md` — после fix → статус `addressed`, может остаться
### B.7 Что коммитится, что нет

#### Коммитится (шаблонные)

Все статические файлы + `analysis/index.md` + `analysis/_finding.md` + `runbooks/index.md` + `runbooks/_runbook.md`.
**Почему:** это часть bundle-шаблона. Если bundle в git (редкий случай) — они нужны.
#### Не коммитится (пользовательские)
- `WORK_LOG.md` — личная память
- `.template-version` — машинный маркер
- `issue/*` — динамические
- `playbook/*` — динамические
- `pr/*` — динамические
- `archive/*` — динамические
- `analysis/F-XXX-*.md` — конкретные findings
- `runbooks/<scenario>.md` — конкретные runbook'и (по умолчанию)

**Почему:** это ваши данные, уникальные для проекта.
**Тонкость:** `.opencode/` целиком в `.git/info/exclude`. Значит, **ничего** не коммитится по умолчанию. `.gitignore` внутри `.opencode/` — вторая линия обороны.
### B.8 Связи между файлами

```text
                      AGENTS.md
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   Onboarding        Daily work       Diagnostics
        │                 │                 │
   _setup.md         _templates.md     _ci.md
   _concepts.md      _worklog.md       _troubleshooting.md
   _glossary.md      _backlog.md       runbooks/
                     _decisions.md
                     _codestyle.md
                     _commands.md
                          │
                          ▼
                    Navigation & safety
                          │
                     _files.md
                     _env.md
                     _security.md
                     analysis/
                     _meta.md
                          │
                          ▼
                    Utility
                          │
                     index.md
                     log.md
                     SPEC_REFERENCE.md
                     .gitignore
                     .template-version
```

**Ключевые cross-links:**
- `_setup.md` → `_env.md`, `_security.md`, `_troubleshooting.md`;
- `_codestyle.md` → `_commands.md`, `_concepts.md`;
- `_ci.md` → `_troubleshooting.md`, `runbooks/`;
- `_worklog.md` → `_decisions.md`, `_backlog.md`;
- `_templates.md` → `issue/`, `playbook/`, `pr/`;
- `analysis/` → `_backlog.md`;
- `_decisions.md` → `_backlog.md`;
- `_meta.md` → `AGENTS.md`, `SPEC_REFERENCE.md`;
### B.9 Размеры bundle

**Стандартный bundle:**
- ~20 файлов в корне;
- 6 директорий;
- ~1500 строк markdown;

**С динамикой (через месяц):**
- - `WORK_LOG.md` (100–300 строк);
- - 2–5 findings;
- - 2–3 ADR;
- - 1–5 backlog items;
- - записи в `_troubleshooting.md`;
- ~2500 строк markdown;

**С динамикой (через год):**
- - `WORK_LOG.md` (1000–3000 строк);
- - 20–50 findings;
- - 10–30 ADR;
- - 20–50 backlog items;
- - архив issues;
- ~8000–15000 строк markdown;

**Bundle растёт, но остаётся читаемым.** Потому что каждый файл — про своё.

### B.10 Чего в bundle НЕТ

**Важно понимать границы.**
- **Нет кода проекта.** Код в `src/`, `lib/`, `app/`
- **Нет публичной документации.** Она в `docs/`, `README.md`
- **Нет `.git`.** Bundle — не репозиторий
- **Нет CI-конфигов.** Они в `.github/`, `.gitlab-ci.yml`
- **Нет секретов.** Только правила
- **Нет секретов в `_security.md`.** Только правила
- **Нет roadmap.** Отдельный документ
- **Нет диаграмм (обычно).** Только ASCII или ссылки на внешние
- **Нет бинарных файлов.** Только markdown
- **Нет PDF / изображений.** Если нужно — ссылки на внешние
### B.11 Bundle vs repository

| Bundle               | Repository           |
| -------------------- | -------------------- |
| `.opencode/`         | Весь git-репозиторий |
| Markdown             | Код + конфиги + docs |
| Локальный            | Распространяется     |
| Не коммитится        | Коммитится           |
| Один на разработчика | Один на проект       |
| Растёт с вами        | Растёт с командой    |

**Bundle — не часть репозитория.** Он **рядом** с ним.

### B.12 Как выглядит bundle в разных проектах

#### Минимальный (для скрипта, гем'а)
```text
.opencode/
├── AGENTS.md
├── _setup.md
├── _codestyle.md
├── _commands.md
├── _files.md
└── index.md, log.md
```
Может быть без директорий, если нет issue-цикла.
#### Стандартный (для web-приложения)

Полная структура, описанная выше.
#### Расширенный (для сложной системы)

```text
.opencode/
├── ... (стандарт)
├── _api.md
├── _release.md
├── _monitoring.md
├── incidents/
│   ├── index.md
│   └── _postmortem.md
└── metrics/
    ├── index.md
    └── _metric.md
```

Больше файлов, если есть специфика.

---
## Appendix C. OKF spec extract

### C.1 Назначение приложения

Это **выдержка из спецификации OKF v0.1** с комментариями о том, **как она применена в нашем bundle**.
Полный текст спеки: [SPEC.md](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md).

Локальная версия без комментариев: `SPEC_REFERENCE.md` в самом bundle.
Здесь — расширенная версия **для читателя книги**. С пояснениями, примерами и связью с нашими решениями.
### C.2 Что такое OKF

**Open Knowledge Format (OKF)** — открытый формат от Google Cloud для представления знаний в виде markdown-файлов.
Официальная цитата из спеки:

> OKF takes the position that knowledge is best represented in commonly accessible, established formats that are:
> - Readable by humans without tooling.
> - Parseable by agents without bespoke SDKs.
> - Diffable in version control.
> - Portable across tools, organizations, and time.

**Разбор:**

**Readable by humans without tooling.** `cat file.md` — и вы видите содержимое. Никаких бинарных форматов, никаких архивов.

**Parseable by agents without bespoke SDKs.** Структура формальна настолько, чтобы её распарсил любой агент. YAML frontmatter, markdown body, ссылки. Никаких «кастомных SDK».

**Diffable in version control.** Изменения видны в `git diff`. Строка 42 изменилась — видно, что именно.

**Portable across tools, organizations, and time.** Знание живёт вне конкретного инструмента. Открывается в любом редакторе, коммитится в git, передаётся коллеге.
### C.3 Ключевой принцип: минимализм

Из спеки:

> No central schema registry, no mandatory tooling. If you can `cat` a file, you can read OKF. If you can `git clone`, you can distribute it.

**Три следствия:**

1. **Нет реестра схем.** Не регистрируете типы централизованно. Придумали тип — используете;
2. **Нет обязательного tooling.** Нет CLI, без которого «не работает». Нет парсера, без которого «не читается»;
3. **Читается через `cat`.** Если файл выглядит как текст — он OKF;

**Как это отражено в нашем bundle:**
- Мы используем `init-opencode` — но **не обязательно**. Bundle работает и без него;
- Мы используем `type` — но не регистрируем их нигде. Просто словарь в README;
- Bundle читается в любом редакторе. Даже в `nano`;
### C.4 Структура bundle

Из спеки:
```text
bundle/
├── index.md           # table of contents (progressive disclosure)
├── log.md             # change history
├── <concept>.md       # concept document
└── <subdirectory>/
    ├── index.md
    ├── <concept>.md
    └── ...
```

**Что такое bundle:** директория с markdown-концептами, организованная по правилам OKF

**Как это отражено у нас:**
Наш bundle — `.opencode/`:
```text
.opencode/
├── index.md              ← OKF index
├── log.md                ← OKF log
├── AGENTS.md             ← главный concept (расширение)
├── _concepts.md          ← concept
├── _setup.md             ← concept
├── ...
├── issue/                ← поддиректория (расширение)
│   └── (динамические файлы)
├── analysis/             ← поддиректория
│   ├── index.md          ← index поддиректории
│   └── F-XXX-*.md        ← concepts
└── runbooks/             ← поддиректория
    ├── index.md
    └── <scenario>.md
```
**Мы расширили:** добавили `AGENTS.md` как основную точку входа, добавили директории `issue/`, `playbook/`, `pr/` для динамических артефактов.
### C.5 Резервированные имена

Из спеки:

| File       | Purpose                     |
| ---------- | --------------------------- |
| `index.md` | Directory table of contents |
| `log.md`   | Change history              |

**Все остальные `.md` файлы — concept documents.**

**Как это отражено у нас:**
- `index.md` — есть. Точка входа.
- `log.md` — есть. Указатель на логи.
- `AGENTS.md` — расширение. В OKF нет, но он нужен для OpenCode.

**Наша адаптация:** в чистом OKF `index.md` — главный файл. У нас — `AGENTS.md`. `index.md`работает как формальный указатель.

### C.6 Concept document

Из спеки:

> Each concept is a UTF-8 markdown file with two parts:
> 1. **YAML frontmatter** (required)
> 2. **Body** (markdown)

#### Frontmatter

Из спеки:
```yaml
---
type: <Type name> # REQUIRED
title: <Optional display name>
description: <Optional one-line summary>
resource: <Optional canonical URI>
tags: [<tag>, <tag>]
generated:
  by: human:<your-name>
  at: <ISO 8601 datetime>
---
```

**Разбор:**
**Required:** `type` — строка, идентифицирующая вид концепта.
**Recommended:** `title`, `description`, `resource`, `tags`, `timestamp`.
**Extensions:** производители могут добавлять любые дополнительные ключи. Потребители должны сохранять неизвестные ключи и не отвергать документы с нераспознанными полями.

**Как это отражено у нас:**

Мы используем все поля. Плюс расширения:
- `severity` — в findings;
- `status` — в project-summary, playbook, pr, finding;
- `issue` — в project-summary, playbook, pr;
- `last-tested` — в runbook;
- `incident-date` — в postmortem (если используется);

**Почему `type` обязателен:** агент по нему понимает вид документа, не читая тело. `type: ci` → «это про CI». `type: finding` → «это находка».

**Наш словарь типов:** 26 значений. См. Appendix A.
#### Body

Из спеки:

> Standard markdown. Structural markdown (headings, lists, tables, code blocks) recommended over free text.

**Conventional sections:**

| Heading       | Purpose                                  |
| ------------- | ---------------------------------------- |
| `# Schema`    | Structured description of fields/columns |
| `# Examples`  | Usage examples                           |
| `# Citations` | External sources                         |

**Как это отражено у нас:**
- Мы используем структурный markdown везде;
- `## Citations` — в `_setup.md`, `SPEC_REFERENCE.md`;
- `## References` — в остальных. Взаимозаменяемо;
- `# Schema`, `# Examples` — по необходимости;

**Почему структурный markdown:** помогает и чтению человеком, и извлечению агентом.
### C.7 Cross-linking

Из спеки:

> Links between concepts are standard markdown links. Two forms:
> **Absolute (from bundle root):**
> markdown
> See [customers table](/tables/customers.md)
> Recommended form — stable when files move.
> **Relative:**
> markdown
> See [neighboring concept](./other.md)

**Semantics:** ссылка от A к B утверждает наличие отношения. Конкретный тип отношения передаётся окружающим текстом.

**Broken links:** потребители должны толерантно обрабатывать. Это может быть просто ещё не написанное знание.

**Как это отражено у нас:**
- Мы используем **относительные** ссылки. Потому что bundle встроен в проект, и абсолютные пути зависят от корня репозитория;
- `[link](_setup.md)` — относительная, работает при перемещении;
- `/tables/customers.md` — абсолютная, работает от корня bundle;

**Почему мы выбрали относительные:**

Наш bundle — `.opencode/`, но абсолютная ссылка `/AGENTS.md` указывала бы на корень **репозитория**, а не bundle. Это не то, что нужно. Относительные ссылки работают корректно.

**Толерантность к битым ссылкам:** мы проверяем ссылки вручную раз в квартал. Но OKF не требует.

### C.8 Citations

Из спеки:

markdown

# Citations

[1] [BigQuery public dataset announcement](https://cloud.google.com/...)
[2] [Internal runbook](https://wiki.internal/...)

**Как это отражено у нас:**

- `## Citations` в `SPEC_REFERENCE.md`, `_setup.md`;
- `## References` в остальных;
- Формат `[N] [title](url)` — везде;

**Почему два разных названия:**

- **Citations** — ссылки на **внешние источники** (статьи, документация);
- **References** — ссылки на **связанные артефакты** (issue, PR, документы);

Семантически разные. OKF использует `Citations`. Мы используем оба — по контексту.

### C.9 Conformance

Из спеки:

> Bundle conforms to OKF v0.1 if:
> 1. Every `.md` file (except `index.md`, `log.md`) contains parseable YAML frontmatter.
> 2. Every frontmatter contains a non-empty `type` field.
> 3. `index.md` and `log.md` follow the described structure.

**Consumers must NOT reject a bundle because of:**
- Missing optional frontmatter fields;
- Unknown `type` values;
- Unknown additional frontmatter keys;
- Broken cross-links;
- Missing `index.md;`

Это **сознательное решение**: OKF должен оставаться полезным по мере роста, рефакторинга и частичной генерации агентами.

**Как это отражено у нас:**

**Conformant:**
- Все `.md` файлы (кроме `index.md`, `log.md`) имеют frontmatter.
- Все frontmatter имеют `type`.
- `index.md` и `log.md` следуют структуре.

**Non-conformant (но допустимо):**
- Мы используем `AGENTS.md` как основную точку входа. OKF-спека этого не запрещает.
- Мы добавили директории (`issue/`, `playbook/`, `pr/`, `analysis/`, `runbooks/`, `archive/`). OKF-спека разрешает поддиректории.
- Мы добавили типы. OKF-спека разрешает неизвестные типы.

**Наш bundle — OKF-вдохновлённый.** Не строго конформный, но следует духу спецификации.

### C.10 Цели OKF

Из спеки:

> 1. Define a universal format that enrichment agents can write to.
> 2. Tell consumption agents how to read and traverse the knowledge.
> 3. Make knowledge exchangeable across systems and organizations.
> 4. Standardize a minimal set of required fields.

**Разбор:**

**Цель 1:** универсальный формат для агентов-обогатителей. **У нас:** bundle — формат, в который пишут и люди, и агенты.

**Цель 2:** потребители-агенты понимают, как читать. **У нас:** `AGENTS.md` + reference files с триггерами.

**Цель 3:** знания обмениваются между системами. **У нас:** bundle можно скопировать, передать, использовать в другом проекте.

**Цель 4:** минимальный набор обязательных полей. **У нас:** только `type` обязателен.

### C.11 Non-goals OKF

Из спеки:

> - Defining a fixed taxonomy of concept types.
> - Prescribing storage or query infrastructure.
> - Replacing domain-specific schemas (Avro, Protobuf, OpenAPI).

**Разбор:**

**Non-goal 1:** OKF не определяет типы. **У нас:** мы определили свой словарь — но это **наша**надстройка, не OKF.

**Non-goal 2:** OKF не предписывает, где хранить. **У нас:** мы выбрали `.opencode/`, но это наш выбор.

**Non-goal 3:** OKF не заменяет схемы данных. **У нас:** `_api.md`, `_concepts.md` ссылаются на Protobuf/OpenAPI, не заменяют.

### C.12 Что мы добавили поверх OKF

Пять расширений.

#### Расширение 1: `AGENTS.md` как entry point

**OKF:** `index.md` — точка входа.

**У нас:** `AGENTS.md` — основная, `index.md` — формальный указатель.

**Почему:** OpenCode читает `AGENTS.md` при старте. Использовать `index.md` было бы несовместимо.

**Влияние на OKF-конформность:** формально нарушает. Но OKF разрешает поддиректории и не запрещает дополнительные файлы.

#### Расширение 2: `_*.md` префикс

**OKF:** все `.md` файлы равны (кроме `index.md`, `log.md`).

**У нас:** `_*.md` — reference files. Без `_` — основные (`AGENTS.md`, `index.md`, `log.md`, `SPEC_REFERENCE.md`).

**Почему:** визуальная маркировка. Служебные файлы выделяются.

**Конвенция:** из SASS, Jekyll, Ruby (partials).

#### Расширение 3: `WORK_LOG.md`

**OKF:** `log.md` — история изменений.

**У нас:** `WORK_LOG.md` — хронология сессий. Локальный, не коммитится.

**Почему:** `log.md` в OKF — общий. Нам нужен специфичный для сессий.

**Связь:** `log.md` у нас — указатель на `WORK_LOG.md`, `_decisions.md`, `_meta.md`.

#### Расширение 4: Директории

**OKF:** поддиректории разрешены.
**У нас:** шесть директорий для динамических артефактов.
- `issue/` — активные issue;
- `playbook/` — стратегии;
- `pr/` — черновики PR;
- `analysis/` — findings;
- `runbooks/` — процедуры инцидентов;
- `archive/` — завершённые;

**Почему:** OKF не описывает workflow. Мы описываем через директории.
#### Расширение 5: Расширенный словарь типов

**OKF:** типы не регистрируются централизованно.
**У нас:** 26 значений в словаре. Документированы в Appendix A.
**Почему:** консистентность внутри проекта. Агент понимает наши типы.
### C.13 Что мы НЕ взяли из OKF

Три вещи, которые OKF описывает, но мы не используем.
#### Не взяли 1: Абсолютные ссылки

**OKF:** рекомендует абсолютные (`/tables/customers.md`).

**У нас:** относительные.

**Почему:** bundle встроен в проект. Абсолютные пути зависят от корня репозитория, что неудобно.

#### Не взяли 2: `# Schema` и `# Examples`

**OKF:** конвенциональные секции.

**У нас:** используем по необходимости. Не в каждом файле.

**Почему:** не все файлы описывают schema или examples. Это для данных.

#### Не взяли 3: `log.md` как реальный changelog

**OKF:** `log.md` — история изменений.

**У нас:** `log.md` — указатель. Реальные логи — в `WORK_LOG.md`, `_decisions.md`, `_meta.md`.

**Почему:** один общий лог — плохо. Пять специфичных — лучше.

### C.14 Tolerance — почему это важно

**Ключевая идея OKF:** потребители должны **толерантно** относиться к bundle.

Это значит:
- **Отсутствие опциональных полей** — не ошибка;
- **Неизвестные `type`** — не ошибка;
- **Неизвестные дополнительные поля** — не ошибка;
- **Битые cross-links** — не ошибка;
- **Отсутствие `index.md`** — не ошибка;

**Почему:** OKF должен оставаться полезным по мере роста, рефакторинга, частичной генерации агентами.

**Пример:**

Если агент видит bundle с `type: my-custom-thing`, он **не должен** падать. Он должен обработать как «неизвестный тип» — прочитать как обычный markdown.

**Как это отражено у нас:**
- Мы не полагаемся на строгую валидацию;
- Битые ссылки — допустимы;
- Пустые секции — не ошибка (но лучше удалить);
- Неизвестные типы — читаются как markdown;

### C.15 Как читать спеку

**Полный текст:** [SPEC.md](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md).

**Что смотреть:**
1. **Bundle structure** — как организовывать файлы;
2. **Concept document** — что писать в frontmatter и body;
3. **Conformance** — что считается конформным;
4. **Goals и Non-goals** — зачем OKF и что он не делает;

**Что игнорировать для нашего bundle:**
- Абсолютные ссылки — используем относительные;
- `# Schema`, `# Examples` — по необходимости;
- Строгая конформность — мы OKF-вдохновлённые;
### C.16 Citations

[1] [OKF SPEC.md — полный текст](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
[2] [Google Cloud — Knowledge Catalog](https://github.com/GoogleCloudPlatform/knowledge-catalog)

---
## Appendix D. FAQ

### D.1 О чём это приложение

Частые вопросы, которые возникают при работе с bundle. Сгруппированы по темам.
Если ваш вопрос не здесь — возможно, ответ в одной из глав. См. оглавление.

### D.2 Общие вопросы

#### Q: Что такое bundle?

OKF-термин. **Директория с markdown-концептами**, организованная по правилам OKF.
В нашем шаблоне bundle — это `.opencode/` в корне проекта.
Не файл, не репозиторий — именно директория.
См. Chapter 3.

#### Q: Зачем всё это?

Проблема: `AGENTS.md` разрастается до 500+ строк. Агент тонет в шуме, теряет фокус.
Решение: **ленивая загрузка**. `AGENTS.md` — маленький. Детали — в reference files, читаются по запросу.
Экономия: контекст, токены, внимание.
См. Chapter 1.

#### Q: Чем это отличается от обычной документации?

| Обычная документация | Bundle                  |
| -------------------- | ----------------------- |
| Для людей            | Для людей **и** агентов |
| Публичная            | Локальная               |
| Коммитится           | Не коммитится           |
| Статичная            | Растёт с вами           |
| Один автор           | Вы + агент              |

Bundle — **инструмент**, не артефакт проекта.

#### Q: А если у меня уже есть хороший AGENTS.md?

Отлично. Bundle — не замена, а **расширение**.
Если ваш `AGENTS.md` — 60 строк, и агент справляется — возможно, bundle вам не нужен.
Bundle окупается, когда:
- `AGENTS.md` растёт
- Есть повторяющиеся задачи (issue, CI, troubleshooting)
- Нужны динамические артефакты (analysis, runbooks)

#### Q: Это обязательно?

**Нет.** Bundle — инструмент. Не догма.
OKF сам говорит: минимализм. Если вам хватает одного `AGENTS.md` — используйте один.
Bundle помогает, когда проект сложный. Для скрипта на 100 строк — overkill.
См. Chapter 20 (Philosophy, раздел «Когда философия не работает»).
#### Q: Куда девать bundle в монорепозитории?

Для каждого сервиса/пакета — свой `.opencode/AGENTS.md`.
Общие правила — в родительском `.opencode/AGENTS.md` на уровне корня монорепозитория.
OpenCode склеит все уровни автоматически.
См. Chapter 5.
### D.3 О структуре

#### Q: Почему `_` в начале имён?

Конвенция из SASS, Jekyll, Ruby (partials). Обозначает **служебный** файл.
Визуально отделяет reference files от основных (`AGENTS.md`, `index.md`, `log.md`).
Не обязательна, но полезна для консистентности.
#### Q: Почему `AGENTS.md`, а не `index.md`?

OKF использует `index.md`. Но OpenCode читает `AGENTS.md`.
Мы используем `AGENTS.md` как основную точку входа, `index.md` — как формальный OKF-индекс (указатель).
См. Chapter 5.
#### Q: Почему `_concepts.md` с `s`?

Файл описывает **несколько** компонентов, концепций, паттернов. Множественное число отражает содержание.
Аналогично: `_commands.md`, `_files.md`, `_templates.md`.
Но `_setup.md` — единственное число, потому что это **один** процесс.
#### Q: Что делать, если у меня нет CI?

Удалите `_ci.md` из bundle.
Пустой файл хуже отсутствующего.
#### Q: Что если нет prod?

Удалите `runbooks/` и упростите `_env.md`.
Bundle — не догма. Адаптируйте.
#### Q: Что если нет issue-трекера?

Удалите `issue/`, `playbook/`, `pr/` и `_templates.md`.
Используйте `WORK_LOG.md` для записи работы.
### D.4 О файлах

#### Q: Почему `WORK_LOG.md` не коммитится?

Это **личная память**. Не часть проекта.
Если нужно поделиться — скопируйте в issue вручную.
См. Chapter 7 (`_worklog.md`).
#### Q: Почему `_security.md` не содержит секретов?

Это **правила** работы с секретами, не хранилище.
Если положить реальный секрет — утечёт при первом коммите, передаче коллеге, синхронизации.
См. Chapter 9 (`_security.md`).
#### Q: Почему `_decisions.md` иммутабелен?

Чтобы сохранить историю.
Если редактировать старые ADR, через год никто не поймёт, почему решение менялось.
Иммутабельность = возможность проследить эволюцию.
Аналогия: git-коммиты. Не редактируете старые — делаете новые.
См. Chapter 7 (`_decisions.md`).
#### Q: Зачем `_backlog.md`, если есть GitHub Issues?

Backlog — **черновик**, Issues — **подтверждённые задачи**.
Backlog дёшев для записи. Issues требует формулировки, меток, ассайна.
Workflow: идея → backlog → решение делать → GitHub Issue.
См. Chapter 7 (`_backlog.md`).
#### Q: Почему `runbooks/` отдельно от `_troubleshooting.md`?

| `_troubleshooting.md`       | `runbooks/`               |
| --------------------------- | ------------------------- |
| Локально                    | Прод                      |
| Разработчик                 | On-call                   |
| Не срочно                   | Срочно                    |
| Много проблем в одном файле | Один инцидент = один файл |

Разные аудитории, разная срочность.
См. Chapter 8.
#### Q: Зачем `analysis/`, если findings можно в `_backlog.md`?

Разные сущности:
- **Finding** — «есть проблема». Состояние.
- **Backlog item** — «делаем X». Задача.
Finding может стать backlog item. Но не обязан.
См. Chapter 9 (`analysis/`).

#### Q: Что писать в `_decisions.md` vs `WORK_LOG.md`?

- **`_decisions.md`** — значимые решения (архитектура, API, процесс).
- **`WORK_LOG.md`** — все сессии, включая мелкие решения.
Правило: если решение повлияет на других через полгода — ADR. Если «локальное решение в сессии» — WORK_LOG.
См. Chapter 7.
### D.5 Об использовании

#### Q: Как часто обновлять файлы?

| Файл                  | Когда                              |
| --------------------- | ---------------------------------- |
| `_concepts.md`        | При архитектурных изменениях       |
| `_ci.md`              | При изменении `.github/workflows/` |
| `_troubleshooting.md` | После каждой проблемы (>10 мин)    |
| `_backlog.md`         | При появлении/завершении задач     |
| `_decisions.md`       | При значимых решениях              |
| `_meta.md`            | При обновлении шаблона             |
| `_env.md`             | При изменении окружений            |
| `_files.md`           | При изменении структуры            |
| `_setup.md`           | При изменении процесса запуска     |

#### Q: Что если я хочу, чтобы агент всегда знал X?

Положите X в `AGENTS.md`. Это единственный файл, который **всегда** в контексте.
Но помните: чем больше `AGENTS.md`, тем меньше внимания к деталям.
Правило: только то, что нужно **каждой** задаче.
#### Q: Как понять, что писать в `AGENTS.md`, а что в reference files?

**Тест:** «Это нужно каждой задаче или только некоторым?»
- Каждой → `AGENTS.md`
- Некоторым → reference file
Пример:
- «Проект — Ruby gem» → `AGENTS.md` (нужно всегда).
- «Тесты запускаются через `bundle exec rspec`» → `_commands.md` (нужно когда тестируешь)
#### Q: Агент не читает нужный файл. Что делать?

Три причины:
1. **Триггер сформулирован размыто.** «When to read: sometimes useful» — плохо. «When to read: writing code» — хорошо.
2. **Задача неоднозначна.** «Расскажи про проект» — агент не знает, что читать. «Расскажи про архитектуру» — понятно.
3. **Модель ошиблась.** Скажите прямо: «Прочитай `_concepts.md` и объясни X».
См. Chapter 4.
#### Q: Можно ли использовать bundle без OpenCode?

Да. Bundle — markdown. Читается в любом редакторе.
`init-opencode` работает независимо от OpenCode. `AGENTS.md` можно переименовать в `CLAUDE.md` (для Claude Code), `GEMINI.md`, `.cursorrules` и т.п.
OKF-формат универсален.
#### Q: Что если я работаю в команде?

Bundle — **личный**. У каждого свой.
Если нужно поделиться шаблоном — используйте один репозиторий шаблонов, но **данные**(WORK_LOG, decisions, backlog) остаются личными.
Для команды — публичная документация в `docs/`, а не bundle.
См. Chapter 20 (Philosophy, раздел «Локальность»).
### D.6 Об `init-opencode`

#### Q: Где хранить клон репозитория шаблонов?

Рекомендуется `~/Projects/opencode-templates/`
Не путать с `~/.config/opencode/` — там глобальный `AGENTS.md`
См. Chapter 17
#### Q: Что если `init-opencode` не в PATH?

```bash
# В ~/.bashrc или ~/.zshrc
export PATH="$HOME/.local/bin:$PATH"
```
Затем `source ~/.bashrc` (или `~/.zshrc`).
Проверка: `which init-opencode`.
#### Q: `--update` перезаписал мой файл. Что делать?

Файлы из списка **NEVER_OVERWRITE** не перезаписываются. Если что-то потерялось:
1. Проверьте бэкап: `.opencode.bak.<timestamp>/`
2. Скопируйте файл из бэкапа
3. На будущее: кастомизированные файлы храните как `<file>.local.md`
См. Chapter 17

#### Q: Как обновить шаблон?

```bash
cd ~/Projects/opencode-templates
git pull
```
Скрипт `init-opencode` подхватит изменения автоматически (если symlink).
Для проектов:
```bash
init-opencode --diff ~/Projects/my-app # preview
init-opencode --update ~/Projects/my-app # apply
```
См. Chapter 17
#### Q: Как добавить новый файл в шаблон?

Семь шагов:
1. Создать `template/_<name>.md`
2. Обновить `template/AGENTS.md`
3. Обновить `bin/init-opencode` (списки NEVER/ALWAYS)
4. Обновить `VERSION`
5. Обновить README
6. Обновить `_meta.md`
7. Commit, push
См. Chapter 18

### D.7 О проблемах

#### Q: Битые ссылки в bundle. Что делать?

OKF **толерантен** к битым ссылкам. Не критично.
Но полезно проверять:
```bash
grep -r '\[._\](._\.md)' .opencode/ | while read line; do
# проверка существования
done
```
Раз в квартал — ревизия
#### Q: Bundle устаревает. Что делать?

**Ревизия раз в квартал.**
- Проверить `timestamp` во всех файлах
- Проверить актуальность содержимого
- Удалить неактуальное
См. Chapter 19 (Anti-patterns, раздел «Чистка bundle»)
#### Q: Bundle разросся. Что делать?

**Признаки:**
- `AGENTS.md` > 150 строк
- `_backlog.md` > 50 items
- `_troubleshooting.md` > 300 строк
- 10+ findings в `analysis/`

**Что делать:**
- Массовая чистка
- Удалить устаревшее
- Разбить на подфайлы
- Пересмотреть процесс
См. Chapter 19
#### Q: Секрет попал в git. Что делать?

**Порядок действий:**
1. **Revoke/rotate** секрет немедленно
2. Уведомить security contact
3. **Не пытаться скрыть через `git rebase`** — история уже утекла
4. Записать инцидент в `_decisions.md`
См. Chapter 9 (`_security.md`)
#### Q: Bundle не помогает. Что делать?

Честно ответить: **«Помогает ли?»**
Если нет:
1. Понять, **почему**. Может, файлы не используются? Может, ритуалы не соблюдаются?
2. Либо исправить процесс
3. Либо удалить bundle. Мёртвый bundle хуже отсутствующего.
См. Chapter 19 (Anti-patterns)
### D.8 О философии

#### Q: Speed over quality — значит писать плохой код?

**Нет.** Значит — **не блокироваться** на перфекционизме.
Качество — ответственность проекта (CI, ревью). Ваша задача — закрывать задачи быстро.
См. Chapter 20 (Philosophy, раздел «Speed over quality»).
#### Q: Минимализм — значит не документировать?

**Нет.** Значит — документировать **нужное**.
Не «на всякий случай», а «без этого не работает».
См. Chapter 20
#### Q: Ленивая загрузка — значит все файлы маленькие?

**Не обязательно.** `_concepts.md` может быть 80 строк. Это ок.
Но `AGENTS.md` — маленький. Reference files — по запросу.
См. Chapter 4
#### Q: Почему bundle локальный? Я хочу поделиться с командой.

Можно. Но осознанно:
- Скопировать в публичный репозиторий
- Или через issue/PR
**По умолчанию — не делится.** Потому что bundle — **личный**
Правила, которые подходят вам — могут не подойти команде
См. Chapter 20 (Philosophy, раздел «Локальность»)
#### Q: Почему OKF, а не свой формат?

OKF — стандарт от Google Cloud. Даёт:
- **Портативность.** Работает с любым редактором
- **Стандарт.** Открытый, стабильный
- **Расширяемость.** Явно разрешает добавления
- **Совместимость.** Другие OKF-инструменты понимают
Свой формат — изобретение велосипеда
См. Chapter 2 и Chapter 20
### D.9 О книге

#### Q: Откуда этот материал?

Из серии обсуждений, в которых разбирались:
- OKF-спецификация
- Принципы ленивой загрузки
- Структура bundle
- Каждый файл в отдельности
- Workflows
- Anti-patterns
- Философия
Материал собирался как учебное пособие для тех, кто внедряет AI-агентов.
#### Q: Как читать эту книгу?

**Линейно** — от начала до конца
**По частям:**
- **Part I (Foundations)** — обязательно
- **Part II (Files)** — справочник. Выборочно
- **Part III (Workflows)** — после основ
- **Part IV (Operations)** — про эксплуатацию
- **Part V (Appendices)** — справочники
См. Preface
#### Q: Можно ли использовать эту книгу для обучения команды?

Да. Материал самодостаточен
Рекомендуется:
1. Прочитать Part I всем
2. Разобрать Part II по частям (по одному файлу в неделю)
3. Обсудить workflows
4. Задать упражнения из глав
#### Q: Есть ли упражнения?

Да. В каждой главе Part II и Part III — секция «Упражнение»
Практические задания: аудит bundle, ретроспектива, тестирование
#### Q: Где брать обновления?

Книга — часть репозитория шаблонов
Обновления — через `git pull` в репозитории
Версия — в `VERSION`
### D.10 Что если вашего вопроса нет

**Возможные источники:**
1. **Оглавление книги.** Возможно, ответ в одной из глав
2. **Оглавление главы.** Каждая глава структурирована: концепция → применение → anti-patterns → упражнение
3. **`SPEC_REFERENCE.md`** — выдержка из OKF
4. **Сам bundle** — `AGENTS.md` и reference files содержат ответы для конкретного проекта

**Если ничего не подошло:**
- Создайте issue в репозитории шаблонов
- Или напишите в обсуждении
### D.11 Итог книги

Мы прошли:
- **[Part I — Foundations](../textbook_ru/guide_ru_part_I.md)** Зачем bundle, OKF, lazy loading
- **[Part II — Files](../textbook_ru/guide_ru_part_II.md)** Каждый файл по отдельности
- **[Part III — Workflows](../textbook_ru/guide_ru_part_III.md)** Шесть сценариев работы
- **[Part IV — Operations](../textbook_ru/guide_ru_part_IV.md)** Установка, расширение, anti-patterns, философия
- **[Part V — Appendices](../textbook_ru/guide_ru_part_V.md)** Справочники

**Главные идеи:**
1. **`AGENTS.md` — оглавление, не энциклопедия**
2. **Ленивая загрузка экономит контекст**
3. **OKF даёт формат, мы добавляем методологию**
4. **Bundle — инструмент, не цель**
5. **Speed over quality, минимализм, локальность**
6. **Bundle растёт с вами**

**Что делать дальше:**
- Установить bundle в свой проект
- Начать с `AGENTS.md` и `WORK_LOG.md`
- Постепенно добавлять остальное
- Регулярно ревизовать

**Bundle — это долгосрочная инвестиция.** Первые пару недель — непривычно. Через месяц — не сможете без него.

---
## Конец книги

Вы дочитали до конца. Если что-то осталось неясным — вернитесь к соответствующей главе.
Если хотите углубиться:
- [OKF SPEC.md](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md) — полная спецификация
- Репозиторий шаблонов — `README_en.md`, `README_ru.md`
- Сам bundle — `.opencode/` в вашем проекте
**Удачи**

---

_Конец_