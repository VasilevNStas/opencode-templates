---
type: book
title: "Knowledge Bundles for AI Agents, part III"
description: "A practical guide to OKF and agent-ready codebases"
timestamp: 2026-09-23
tags: [okf, agents, opencode, guide]
---

# Part III — Workflows

_В [Part I](../textbook_ru/guide_ru_part_I.md) и [Part II](../textbook_ru/guide_ru_part_II.md) мы разобрали **что** лежит в bundle и **почему**. В Part III рассмотрим **как** этим пользоваться. Не файлы, а процессы. Сценарии, которые повторяются снова и снова._

_Шесть глава — шесть workflow'ов: `issue`, `session`, `CI`, `incident`, `analysis`, `planning`. Каждый — от начала до конца._

---
## Глава 11. Жизненный цикл

### 11.1 Что такое «issue lifecycle»

**Issue lifecycle** — последовательность шагов от «взяли задачу» до «PR смержен и заархивирован»

Это **самый частый workflow**. Если вы работаете с проектом хотя бы раз в неделю, вы проходите его постоянно.

Три стадии:

```text

    Start         During         End
      │             │             │
      ▼             ▼             ▼
  ┌────────┐    ┌────────┐    ┌────────┐
  │ Открыть│ →  │ Работа │ →  │  PR    │
  │  issue │    │  над X │    │ merge  │
  └────────┘    └────────┘    └────────┘
      │              │            │
      ▼              ▼            ▼
    SUMMARY       WORK_LOG     PR desc
    PLAYBOOK      decisions    archive
```

**Start:** создаём артефакты
**During:** работаем
**End:** завершаем

### 11.2 Роли файлов в lifecycle

| Файл                           | Роль                      | Когда создаётся    |
| ------------------------------ | ------------------------- | ------------------ |
| `issue/PROJECT_SUMMARY_<N>.md` | Что делаем и как проверим | Start              |
| `playbook/PLAYBOOK_<N>.md`     | Стратегия решения         | Start              |
| `WORK_LOG.md`                  | Хронология сессий         | During             |
| `_decisions.md`                | Значимые решения          | During (если есть) |
| `pr/PR_<N>.md`                 | Описание PR               | End                |
| `archive/`                     | Куда убираем после merge  | End                |

**Три артефакта на issue:** SUMMARY, PLAYBOOK, PR
**Один общий:** WORK_LOG
**Опционально:** ADR

### 11.3 Start

#### Шаг 1: понять задачу

Прежде чем что-то создавать — убедитесь, что понимаете issue

Три вопроса:
1. **Что** нужно сделать?
2. **Зачем** это нужно? (какую проблему решает)
3. **Как** поймём, что сделали? (критерии готовности)

Если не можете ответить на все три — вернитесь к issue. Задайте вопросы автору. **Не начинайте работу с размытым пониманием.**
#### Шаг 2: прочитать контекст

Перед работой полезно прочитать:
- `_concepts.md` — как устроен проект;
- `_codestyle.md` — как писать код;
- `_files.md` — где что лежит;
- `_glossary.md` — если встретили незнакомый термин;

Не читайте всё подряд. Только то, что **нужно для этой задачи**.
#### Шаг 3: создать ветку

```bash
git checkout master
git pull
git checkout -b <issue-number>
```

Имя ветки — номер issue. `123`, `#123`, `feature/123` — зависит от вашей branch discipline (см. `~/.config/opencode/AGENTS.md`).
#### Шаг 4: создать PROJECT_SUMMARY

Из шаблона в `_templates.md`:

```markdown
---
type: project-summary
issue: "#123"
title: "Add JSON parser"
status: draft
generated:
  by: human:<your-name>
  at: 2026-09-21T10:30:00Z
tags: [feature, parser]
---

# Project Summary: #123 Add JSON parser

## Issue Overview

[Ссылка на issue] Нужен парсер для JSON с поддержкой streaming.

## Problem

Входные файлы могут быть 100MB+. Текущий парсер загружает всё в память

## Solution

(заполняется позже, по ходу)

## Verification

- [ ] `bundle exec rubocop` — 0 offenses
- [ ] `bundle exec rspec` — all pass
- [ ] Parse 100MB file < 5 seconds
```

**На этой стадии `Solution` может быть пустым.** Заполнится в процессе.
**`Problem` — самое важное.** Хорошо сформулированная проблема — половина решения.
#### Шаг 5: создать PLAYBOOK

Если задача нетривиальная — создайте PLAYBOOK:

````markdown
---
type: playbook
issue: "#123"
title: "Add JSON parser"
status: draft
generated:
  by: human:<your-name>
  at: 2026-09-21T10:30:00Z
tags: [feature, parser]
---

# Playbook #123: Add JSON parser

## Context

Streaming требует инкрементального чтения. Нужно не сломать существующий API

## Strategy

1. Write `StreamReader` for incremental file reading
2. Rewrite `Parser` to consume tokens from stream
3. Keep existing in-memory mode as fallback
4. Benchmark on 100MB file

## Known Pitfalls

- Streaming requires buffering. Don't load whole file
- AST nodes must be immutable

## Verification Commands

```bash
bundle exec rspec spec/parser_spec.rb
bundle exec rake benchmark
```
````

**PLAYBOOK не обязателен для простых задач** Если изменение — одна строка, PLAYBOOK избыточен.

**Признаки, что PLAYBOOK нужен:**
- Больше 3 файлов будет затронуто
- Есть выбор между подходами
- Есть риски и грабли
#### Шаг 6: начать запись в WORK_LOG

Первая сессия по issue:

```markdown
## 2026-09-21

### Session 47 — Start #123

Created PROJECT_SUMMARY_123.md and PLAYBOOK_123.md. Read _concepts.md,
understood the parser architecture. Started with StreamReader.
**Decision:** Streaming via Enumerator, not thread-based. Simpler.
**Problem:** —
**Next:** Finish StreamReader, then rewrite Parser.
---
```

**Decision и Problem могут быть пустыми** в первой сессии. Но **Next обязателен** — иначе непонятно, где остановились.
### 11.4 During

#### Работа по сессиям

Каждая сессия — новый блок в WORK_LOG:

```markdown
## 2026-09-22

### Session 48 — StreamReader done

| #    | What       | Files                    | Status           |Complexity|
| ---- | ---------- | ------------------------ | ---------------- | -------- |
| 123  |StreamReader|`lib/stream_reader.rb`, `spec/`|in-progress  | medium   |
| **Decision:** Used `Enumerator::Lazy` for chunk reading. Alternative |
| was manual buffering — more code, no benefit.                        |
| **Problem:** Buffer overflow on very large lines. Fixed by splitting |
| lines at 1MB.                                                        |
| **Next:** Rewrite Parser to consume from stream.                     |
---
```

**Правила:**
- **Одна сессия = один блок**;
- **Decision** — только неочевидное;
- **Problem** — что пошло не так и как решили;
- **Next** — обязателен;
#### Когда создавать ADR

Если в процессе принимается **значимое** решение — создайте ADR в `_decisions.md`
Триггеры:
- Выбор технологии
- Изменение API
- Отказ от подхода
**Пример:**

```markdown
## ADR-005: Streaming via Enumerator

- **Date:** 2026-09-22
- **Status:** accepted
- **Issue:** #123
- **Deciders:** @you

### Context

Need streaming JSON parsing for files >100MB. Two options: Enumerator
and thread-based producer/consumer

### Decision

Use Enumerator::Lazy

### Alternatives considered

- **Thread-based** — more complex, no benefit for single-file parsing

### Consequences

- **Positive:** simpler code, no thread management
- **Negative:** one file at a time (can't parallelize)
- **Follow-up:** benchmark against thread-based for huge files
```

**В WORK_LOG — короткая ссылка:**

> **Decision:** Streaming via Enumerator. See ADR-005.
#### Обновление PROJECT_SUMMARY

По ходу работы обновляйте:
- `Solution` — как решаете;
- `Files Changed` — что затронуто;
- `Key Discoveries` — что узнали;

**Статус:** `draft` → `in-progress`
### 11.5 End

#### Шаг 1: финализировать PROJECT_SUMMARY

Перед PR:
- `Solution` — заполнен;
- `Verification` — все чекбоксы отмечены;
- `Files Changed` — актуален;
- `Key Discoveries` — записаны;
- `status: completed;`

```markdown
## Verification

- [x] `bundle exec rubocop` — 0 offenses
- [x] `bundle exec rspec` — all pass
- [x] Parse 100MB file < 5 seconds (actual: 3.2s)
```
#### Шаг 2: создать PR description

Из шаблона в `_templates.md`:

```markdown
---
type: pr
issue: "#123"
title: "Add streaming mode to JSON parser"
status: ready-for-review
---

## Description

Adds a streaming mode to the parser, allowing files >100MB to be parsed
without loading everything into memory. Existing in-memory mode remains
default

## Related Issue

Fixes #123

## Changes

| File                        | Change                         |
| --------------------------- | ------------------------------ |
| `lib/json/stream_reader.rb` | New — incremental file reading |
| `lib/json/parser.rb`        | Added streaming mode           |
| `spec/json/parser_spec.rb`  | 5 new tests                    |
| `benchmark/parse.rb`        | New benchmark                  |

## Verification

- [x] Build passes
- [x] Tests pass
- [x] Lint passes
- [x] No unrelated changes

## Notes for Reviewers

The `StreamReader` uses `Enumerator::Lazy`. Pay attention to buffer
management in `StreamReader#read_chunk`.
```

**`Fixes #123`** — магическое слово GitHub. PR автоматически закроет issue при merge.

**Если PR не закрывает issue полностью** — `Related to #123`
#### Шаг 3: push, открыть PR

```bash
git add .
git commit -m "Add streaming mode to JSON parser"
git push -u origin 123
```

Затем — открыть PR на GitHub. Скопировать description из `pr/PR_123.md`. **Убрать frontmatter** — GitHub отрендерит его как текст.

**Проверить:*
- CI запустился;
- Description корректный;
- Reviewers назначены;
#### Шаг 4: дождаться ревью

Пока ждёте — работайте над следующей issue. **Не блокируйтесь.**
Если CI упал — см. Chapter 13.
Если ревьюер просит изменения:
- Правите код;
- Обновляете PR (тот же PR, не новый);
- Отвечаете на комментарии;

**Не забывайте:** PR — итеративный. 2–3 раунда ревью — норма.
#### Шаг 5: merge

После approval:
- Squash или merge — зависит от проекта;
- PR закрыт, issue закрыт автоматически (если `Fixes #N`);
- Ветка удалена;
#### Шаг 6: архивировать

Переместить три файла:

```bash
mkdir -p .opencode/archive/123
mv .opencode/issue/PROJECT_SUMMARY_123.md .opencode/archive/123/
mv .opencode/playbook/PLAYBOOK_123.md .opencode/archive/123/
mv .opencode/pr/PR_123.md .opencode/archive/123/
```

**Или** без поддиректории:

```bash
mv .opencode/issue/PROJECT_SUMMARY_123.md .opencode/archive/
mv .opencode/playbook/PLAYBOOK_123.md .opencode/archive/
mv .opencode/pr/PR_123.md .opencode/archive/
```

Структура архива — на ваш выбор. Поддиректория по номеру issue — удобнее, когда issues много.
#### Шаг 7: запись в WORK_LOG

```markdown
## 2026-09-25

### Session 50 — Merged #123

| #   | What             | Files   | Status | Complexity |
| --- | ---------------- | ------- | ------ | ---------- |
| 123 | Streaming parser | 4 files | merged | medium     |

**Decision:** —

**Problem:** —

**Next:** Move to #124 (caching)
---

**Статус:** `merged`

**Next:** следующая issue
```
### 11.6 Полный пример

#### До

```text
.opencode/
├── issue/          (пусто)
├── playbook/       (пусто)
├── pr/             (пусто)
├── archive/        (пусто)
└── WORK_LOG.md     (пустой)
```
#### Start

```text
.opencode/
├── issue/
│   └── PROJECT_SUMMARY_123.md   ← создан
├── playbook/
│   └── PLAYBOOK_123.md          ← создан
├── pr/             (пусто)
├── archive/        (пусто)
└── WORK_LOG.md     ← Session 47
```
#### During (3 сессии)

```text
.opencode/
├── issue/
│   └── PROJECT_SUMMARY_123.md   ← обновлён (Solution, Files)
├── playbook/
│   └── PLAYBOOK_123.md          ← обновлён (Pitfalls)
├── pr/             (пусто)
├── archive/        (пусто)
├── _decisions.md   ← ADR-005
└── WORK_LOG.md     ← Session 47, 48, 49
```
#### End

```text
.opencode/
├── issue/          (пусто)
├── playbook/       (пусто)
├── pr/             (пусто)
├── archive/
│   └── 123/
│       ├── PROJECT_SUMMARY_123.md
│       ├── PLAYBOOK_123.md
│       └── PR_123.md
├── _decisions.md   ← ADR-005
└── WORK_LOG.md     ← Session 47, 48, 49, 50
```

**Цикл завершён.** Следующая issue — то же самое.
### 11.7 Что делать если...

#### Задача оказалась больше, чем думали

Иногда issue растёт в процессе. Что делать:
1. **Остановитесь.** Пересмотрите scope;
2. **Разбейте** на несколько issues;
3. **Первая issue** — минимально работающее решение;
4. **Остальное** — в `_backlog.md`;

**Не пихайте всё в один PR.** Маленькие PR быстрее ревьюятся.
#### Задача оказалась ненужной

Бывает. Issue устарела, или решение пришло извне. Что делать:
1. **Закройте issue** с объяснением;
2. **Удалите** `issue/PROJECT_SUMMARY_123.md` и `playbook/PLAYBOOK_123.md`;
3. **Не архивируйте** — незачем;
4. **Запишите** в WORK_LOG: «Closed #123 as obsolete»;
#### PR не принимают

Ревьюер против подхода. Что делать:
1. **Обсудите.** Не спорьте, а поймите;
2. **Возможно**, подход действительно плох;
3. **Скорректируйте** или закройте PR;
4. **Если PR закрыт** — обновите `_decisions.md` (ADR: почему не пошли этим путём);
5. 
**Пример ADR:**

```markdown
## ADR-006: Rejected streaming via Enumerator

- **Date:** 2026-09-25
- **Status:** rejected
- **Issue:** #123

### Context

Considered Enumerator::Lazy for streaming

### Decision

Rejected. Team prefers thread-based approach

### Alternatives considered

- **Enumerator::Lazy** — rejected because we need parallel parsing

### Consequences

- **Positive:** —
- **Negative:** implementation more complex
```

Отклонённые решения — **тоже ADR**. Они документируют, что **не** сработало.
#### Забыли про issue

Начали что-то делать, а потом вспомнили: «А где issue?»
**Правило:** не начинайте работу без issue. Даже для мелочи.
Исключение: срочный хотфикс. Тогда:
1. Создайте issue **после** фикса;
2. Опишите, что сделали;
3. Пометьте `hotfix`;
#### Работаете над несколькими issues параллельно

Возможно, но осторожно:
- **Одна ветка = одна issue.** Не смешивайте;
- **Один WORK_LOG** — все сессии в одном файле;
- **Один PROJECT_SUMMARY** на issue;

**Не создавайте один PR для двух issues.** Даже если связаны.
### 11.8 Anti-patterns

**Anti-pattern 1: PR без issue**

«Сделал мелкое изменение, issue не нужен.» Не надо. Даже мелкое — фиксируйте. Через месяц не вспомните, зачем.

**Anti-pattern 2: PR на 5000 строк**
Огромный PR невозможно ревьюить. Правило: **< 400 строк** — ревьюится за час. Больше — разбивайте.

**Anti-pattern 3: обновление PR новым PR**
Ревьюер попросил изменения — **не закрывайте PR и не открывайте новый**. Обновляйте существующий. Иначе теряется история обсуждения.

**Anti-pattern 4: не обновлять PROJECT_SUMMARY**
Создали, забыли. Через месяц смотрите — не помните, что там.

**Anti-pattern 5: WORK_LOG не ведётся**
Пропустили сессию, ещё одну. Через неделю — каша.

**Anti-pattern 6: архив не используется**
Файлы issue накапливаются в `issue/`. Пора чистить.

**Anti-pattern 7: нет ADR**
Приняли решение, не записали. Через полгода не помните, почему так.

**Anti-pattern 8: ревьюер не назначен.**
PR открыт, никто не смотрит. Назначайте ревьюеров.

### 11.9 Метрики

Что можно измерять:

| Метрика                | Что показывает                     |
| ---------------------- | ---------------------------------- |
| **Cycle time**         | От открытия issue до merge         |
| **Review time**        | От PR до approval                  |
| **Sessions per issue** | Сколько сессий заняла issue        |
| **Files per issue**    | Масштаб изменений                  |
| **HoC**                | Объём изменений (если используете) |

**Не превращайте метрики в KPI.** Они для понимания, не для давления.

**Cycle time 3 дня** — норма. **2 недели** — что-то не так. **1 день** — либо задача мелкая, либо слишком спешите.

### 11.10 Связь с другими файлами

```text
AGENTS.md (Issue workflow section)
    │
    ├─→ _templates.md         (шаблоны)
    ├─→ _worklog.md           (формат сессий)
    ├─→ _decisions.md         (ADR)
    ├─→ _backlog.md           (что дальше)
    ├─→ _codestyle.md         (как писать)
    ├─→ _commands.md          (команды)
    └─→ _ci.md                (если CI упал)
```

**Issue lifecycle — центральный workflow.** Он связывает почти все файлы bundle.

### 11.11 Упражнение

Возьмите **текущую** issue (или выберите одну из недавних). Пройдитесь по шагам:

1. **Start:** создан ли PROJECT_SUMMARY? PLAYBOOK? Заполнены ли `Problem` и `Verification`?
2. **During:** ведётся ли WORK_LOG? Есть ли Decision/Problem/Next?
3. **End:** готов ли PR description? Все ли чекбоксы отмечены? Перемещены ли файлы в archive?

**Честно ответьте:**
- Сколько сессий заняла issue? Совпадает с записями в WORK_LOG?
- Были ли решения, которые заслуживают ADR?
- Был ли PR слишком большим?

**Если что-то не так** — исправьте процесс, а не только эту issue.

### 11.12 Что дальше

В следующей главе — **session lifecycle**. Issue — это задача. Session — это **атом времени**. Одна issue может занять несколько сессий. Session lifecycle описывает, что делать в начале и в конце каждой сессии.
## Глава 12. Жизненный цикл сессии

### 12.1 Что такое сессия

**Сессия** — это отрезок времени, в течение которого вы работаете над проектом без длинного перерыва.

Примеры:
- Утро понедельника: 2 часа работы над issue #123
- Вечер среды: 45 минут на багфикс
- Суббота: 4 часа на рефакторинг

**Сессия не равна дню** Может быть две сессии в один день. Может быть одна сессия на три дня (если работаете с перерывами в рамках одного подхода).

**Сессия не равна issue** Одна issue может занять 5 сессий. Одна сессия может затронуть 2 issues.

**Граница сессии:** когда вы **закрываете ноутбук** или **переключаетесь на что-то другое надолго**

### 12.2 Зачем нужен session lifecycle

Проблема, которую он решает: **память между сессиями**

Вы возвращаетесь к проекту через день, через неделю, через месяц. Вы **не помните**:
- Где остановились;
- Что решили в прошлый раз;
- Почему пошли именно этим путём;
- Что делать дальше;

Session lifecycle — это ритуал, который **сохраняет контекст** между сессиями

**Аналогия:** git-коммиты. Вы не работаете без коммитов — иначе потеряете изменения. WORK_LOG — это коммиты вашей **головы**.

### 12.3 Роль WORK_LOG

`WORK_LOG.md` — единственный артефакт session lifecycle. Он ведётся **постоянно**, а не только при работе над issue.

**Структура WORK_LOG:**

```text
WORK_LOG.md
├── Intro (frontmatter + правила)
├── Session N    ← самая свежая
├── Session N-1
├── Session N-2
└── ...
```

**Правило «newest first»** Новые сессии — **сверху**, сразу после intro. Не в конец файла.

**Почему:** когда вы возвращаетесь к проекту, первое, что нужно — **последняя сессия**. Она должна быть первой в файле.

### 12.4 Start сессии

#### Шаг 1: прочитать последнюю сессию

Откройте `WORK_LOG.md`. Прочитайте **первую** запись после intro.

```text
## 2026-09-21
### Session 47 — Start #123
Created PROJECT_SUMMARY_123.md and PLAYBOOK_123.md. Read _concepts.md.
Started with StreamReader.
**Decision:** Streaming via Enumerator, not thread-based.
**Problem:** —
**Next:** Finish StreamReader, then rewrite Parser.
```

**Что вы узнаёте:**
- Что делали в прошлый раз;
- Какие решения приняли;
- Что делать сейчас (секция **Next**);

**Если Next пустой** — плохо. Значит, в прошлый раз не записали. Придётся вспоминать.
#### Шаг 2: проверить состояние

Перед началом — убедитесь, что всё в порядке:

```bash
# Проверить незакоммиченные изменения
git status
# Проверить текущую ветку
git branch --show-current
# Проверить состояние CI (если открыт PR)
gh pr status
```

**Типичные ситуации:**
- **Незакоммиченные изменения** — что это? Забыли закоммитить? Или эксперимент?
- **Не та ветка** — переключились случайно? Переключитесь обратно.
- **CI красный** — сначала починить, потом новая работа.
#### Шаг 3: перечитать Next

Секция **Next** из прошлой сессии — это ваш план на эту сессию. Не догма, но отправная точка.

**Пример:**

> **Next:** Finish StreamReader, then rewrite Parser

Возможно, за ночь пришла новая идея. Возможно, обстоятельства изменились. Но **начните с Next** — если нет причин иначе.
#### Шаг 4: открыть нужные файлы

В зависимости от задачи:
- Работа над issue → `PROJECT_SUMMARY_<N>.md`, `PLAYBOOK_<N>.md`;
- Новый код → `_codestyle.md`, `_concepts.md`, `_files.md`;
- Багфикс → `_troubleshooting.md`, `_files.md`;
- CI упал → `_ci.md`;

**Не открывайте всё подряд** Только то, что нужно
### 12.5 During сессии

#### Работа

Работа — вне книги. Это код, ревью, обсуждения. Session lifecycle описывает **обрамление**: что до и что после.

Но в течение сессии полезно:

**Обновлять PROJECT_SUMMARY по ходу** Если нашли что-то важное — запишите сразу. Через час забудете.

**Держать WORK_LOG открытым** Не в прямом смысле, а мысленно. Если принимаете решение — думайте: «это стоит записать?»

**Не переключаться между issues** Одна сессия — одна issue. Если переключились — это уже другая сессия.
#### Что фиксировать

**Decision** — неочевидные решения.
- Плохо: «Использовал `each` вместо `map`» — тривиально
- Хорошо: «Использовал `Enumerator::Lazy` вместо thread-based» — есть выбор

**Problem** — что пошло не так и как решили
- Плохо: «Ошибка в коде» — неинформативно
- Хорошо: «Buffer overflow on lines >1MB. Fixed by splitting at 1MB.» — конкретно

**Discovery** — что узнали нового
- «Оказалось, что PostgreSQL `IN` имеет предел в 32k значений. Обойдём через `ANY(ARRAY[...])`»

**Blocker** — что мешает продолжать
- «Waiting for API key from ops. Can't test integration»

**Next** — что делать в следующей сессии. **Обязателен.**
#### Промежуточная запись (опционально)

Если сессия длинная — можно писать в WORK_LOG **дважды**: в середине и в конце. Особенно если в середине было важное решение.

Это не обязательно. Но полезно, если сессия занимает больше 3 часов.

### 12.6 End сессии

#### Шаг 1: остановиться

Когда решаете закончить — **остановитесь**. Не бросайте на середине мысли.

**Правило:** заканчивайте сессию на **логической границе**.
- Функция дописана;
- Тест проходит;
- Коммит сделан;

**Не заканчивайте** на середине функции. Завтра будете 20 минут вспоминать, что хотели.
#### Шаг 2: закоммитить или отложить

Если работа готова к коммиту:

```bash
git add .
git commit -m "Add StreamReader"

```

Если работа **не** готова — но вы не хотите терять:

```bash
# Временный коммит
git add .
git commit -m "WIP: StreamReader in progress"
# Или stash
git stash push -m "StreamReader WIP"
```

**Никогда не оставляйте незакоммиченные изменения на ночь** Завтра не вспомните, что делали
#### Шаг 3: записать сессию в WORK_LOG

Главный шаг. Откройте `WORK_LOG.md`. Вставьте новую запись **после intro, перед первой существующей `## <date>`**.

**Полная форма** (для обычной сессии):

```markdown
## 2026-09-22

### Session 48 — StreamReader done

| #   | What   | Files           | Status      | Complexity |
| --- | ------ | --------------- | ----------- | ---------- |
| 123 | StreamReader|`lib/stream_reader.rb`, `spec/`| in-progress | medium|
| **Decision:** Used `Enumerator::Lazy` for chunk reading.               |
| **Problem:** Buffer overflow on lines >1MB. Fixed by splitting at 1MB. |
| **Next:** Rewrite Parser to consume from stream.                       |
---
```

**Краткая форма** (для тривиальной сессии):

```markdown
## 2026-09-22

### Session 49 — Typo fix in README

Fixed typo. No code changes.
**Next:** Continue #123.
---
```

**Правила:**
- **Одна сессия = один блок**
- **Номер сессии сквозной.** Session 47, 48, 49... Не сбрасывается
- **Дата ISO 8601.** `2026-09-22`, не `22/09/2026`
- **Секция Next обязательна**
#### Шаг 4: обновить задачи

Если что-то новое появилось — запишите в `_backlog.md`:
- Идеи: «А что если...» — в секцию Ideas;
- Задачи: «Надо не забыть...» — в Items;
- Техдолг: «Этот код плохой, потом переделать» — в Tech debt;

**Правило:** если **Next** содержит конкретную задачу — она должна быть и в backlog. WORK_LOG фиксирует факт «остановились здесь», backlog — «это надо сделать».

#### Шаг 5: создать ADR (если было решение)

Если в сессии принято **значимое** решение — создайте ADR в `_decisions.md`.

**Что значит «значимое»:**
- Влияет на архитектуру;
- Влияет на публичный API;
- Влияет на процесс;

**Что НЕ значит:**
- «Использовал `each` вместо `map`»;
- «Переименовал метод»;
- «Добавил тест»;

**Связь:** в WORK_LOG — короткая ссылка: «Decision: Used Enumerator. See ADR-005.»

#### Шаг 6: закрыть ноутбук

Всё. Сессия закончена. В следующий раз начнёте с **Step 1 of Start** — прочитать последнюю запись.
### 12.7 Пример: полный цикл

#### Session 47 (Start)

```markdown
## 2026-09-21

### Session 47 — Start #123

Created `issue/PROJECT_SUMMARY_123.md` and `playbook/PLAYBOOK_123.md`.
Read `_concepts.md` — understood parser architecture.
**Decision:** Streaming via Enumerator, not thread-based.
**Problem:** —
**Next:** Finish StreamReader, then rewrite Parser.
---
```

#### Session 48 (During)

```markdown
## 2026-09-22

### Session 48 — StreamReader done

| #   | What   | Files          | Status      | Complexity |
| --- | ------ | -------------- | ----------- | ---------- |
| 123 | StreamReader|`lib/stream_reader.rb`, `spec/stream_reader_spec.rb`| in-progress | medium     |
| **Decision:** Used `Enumerator::Lazy` for chunk reading.               |
| **Problem:** Buffer overflow on lines >1MB. Fixed by splitting at 1MB. |
| **Next:** Rewrite Parser to consume from stream.                       |

---
```

#### Session 49 (During, короткая)

```markdown
## 2026-09-23

### Session 49 — Parser rewrite started

Started rewriting `Parser#parse` to accept stream. Not finished yet.
**Next:** Finish Parser rewrite, run tests.
---
```

#### Session 50 (End)

```markdown
## 2026-09-25

### Session 50 — Merged #123

| #  | What | Files   | Status | Complexity |
| -- | ---- | ------- | ------ | ---------- |
| 123| Streaming parser | 4 files | merged | medium     |
| **Decision:** —                   |
| **Problem:** —                    |
| **Next:** Move to #124 (caching)  |

---
```

**Четыре сессии, одна issue.** Каждая запись — 5–15 строк. В сумме — 40 строк на всю issue. Через полгода вы прочитаете их за 2 минуты и вспомните всё.

### 12.8 Что если...

#### Сессия короче 15 минут

Иногда — быстро поправить опечатку, ответить на комментарий в PR, обновить README.

**Записывать ли?** Зависит
- **Не записывать:** если это часть другой работы (например, ответ на ревью в рамках issue #123);
- **Записать кратко:** если это отдельное действие, которое не вписывается в контекст;

**Пример краткой записи:**

```markdown
## 2026-09-23

### Session 49 — Reply to review on #123

Addressed reviewer comments: renamed `read_chunk` to `read_chunks`,
added test for edge case.
**Next:** Wait for re-review.
---
```

5 строк. Не таблица, не Decision/Problem. Просто факт.

#### Сессия прервана

Телефонный звонок, срочное дело, отключили электричество. Что делать:
1. **Не паниковать.** WORK_LOG можно записать позже;
2. **Если возможно** — закоммитить WIP;
3. **Когда вернётесь** — восстановите контекст по git status / git log / open files;
4. **Запишите** в WORK_LOG, что сессия была прервана. Это тоже информация.

**Пример:**

```markdown
## 2026-09-22

### Session 48 — Interrupted

Started StreamReader. Session interrupted at 14:30 — no commit.
See `git status` for current state.
**Next:** Resume StreamReader.
---
```

#### Забыли записать сессию

Бывает. Вы работали вчера, а сегодня понимаете: «Я не записал WORK_LOG».

**Что делать:**
1. **Вспомните**, что делали. Используйте `git log`, `git reflog`, файлы
2. **Запишите** с пометкой: `(recorded late)`
3. **Впредь** — записывайте в конце сессии, пока помните

**Пример:**

```markdown
## 2026-09-22

### Session 48 — StreamReader (recorded late)

Created `StreamReader` with `Enumerator::Lazy`. Added 5 tests.
**Decision:** Enumerator::Lazy for chunks.
**Next:** Parser rewrite.
---
```

Пометка «recorded late» — честно. Не скрывайте, что запись сделана задним числом.
#### Несколько issues в одной сессии

Возможно, но **осторожно**
- **Основная issue** — та, над которой работаете
- **Побочная** — мелкая (ответ на ревью, хотфикс)

**В WORK_LOG:**

```markdown
## 2026-09-22

### Session 48 — #123 + hotfix #999

| #     | What   | Files    | Status      | Complexity |
| ----- | ------ | -------- | ----------- | ---------- |
| 123   | StreamReader| `lib/stream_reader.rb` | in-progress | medium  |
| 999   | Fix typo in help | `lib/cli.rb`      | merged      | low     |
| **Decision:** —          |
| **Problem:** —           |
| **Next:** Continue #123. |

---
```

Две строки в таблице — две issues.

**Не злоупотребляйте.** Если issues много — это уже не одна сессия, а несколько.

#### Сессия без issue

Бывает: исследование, эксперимент, чтение кода. Issue нет.

**Что делать:**
- **Записать в WORK_LOG** как обычно
- **Статус:** `research` или `exploration`
- **Не создавать** PROJECT_SUMMARY — это не работа над issue

**Пример:**

```markdown
## 2026-09-22

### Session 48 — Research: streaming libraries

Explored 3 Ruby streaming libraries: `Enumerator::Lazy`, `IO#each_line`,
`StringIO`. Compared performance on 10MB file.
**Decision:** Enumerator::Lazy — fastest, simplest.
**Next:** Apply to #123.
---
```
### 12.9 Anti-patterns

**Anti-pattern 1: WORK_LOG не ведётся**
Пропустили сессию. Потом ещё одну. Через неделю — каша.

**Anti-pattern 2: WORK_LOG ведётся нерегулярно**
Записываете только крупные сессии. Мелкие — нет. Через месяц непонятно, что происходило.

**Anti-pattern 3: одна запись на весь день.**
Две сессии в день — должны быть две записи. Иначе теряется гранулярность.

**Anti-pattern 4: нет секции Next**
Через месяц непонятно, где остановились. Приходится вспоминать.

**Anti-pattern 5: запись в конец файла**
Нарушение правила «newest first». Свежая сессия должна быть **сверху**.

**Anti-pattern 6: тривиальные сессии с таблицей**
«Исправил опечатку» в таблице с колонками Files/Complexity — абсурд.

**Anti-pattern 7: значимые решения только в WORK_LOG**
Если решение архитектурное — оно в `_decisions.md`. В WORK_LOG — короткая ссылка.

**Anti-pattern 8: WORK_LOG в git**
Это личная память. Не коммитится. В `.gitignore`.

**Anti-pattern 9: слишком длинные записи.**
20 строк на сессию — много. 5–15 — норма.

**Anti-pattern 10: секция Problem всегда пустая**
Если «проблем не было» — это не проблема. Но если они есть и не записываются — теряется ценная информация.

### 12.10 Связь с другими файлами

```text

                 WORK_LOG.md
                     │
     ┌───────────────┼───────────────┐
     ▼               ▼               ▼
_worklog.md     _backlog.md    _decisions.md
(формат)        (Next → task)  (значимые решения)
     │
     └─→ _templates.md   (PROJECT_SUMMARY, PLAYBOOK)
         _codestyle.md   (что читать при работе)
         _ci.md          (если CI упал в сессии)
```

**WORK_LOG — центральный узел.** Через него проходят все сессии, все issues.
### 12.11 Упражнение

**Часть 1: ретроспектива**

Откройте ваш `WORK_LOG.md` (если есть). Ответьте:
1. **Сколько сессий записано?** Совпадает с реальностью?
2. **Есть ли секция Next в каждой?** Если нет — как вы восстанавливаете контекст?
3. **Есть ли Decision?** Или только «сделал X»?
4. **Есть ли Problem?** Или «всё было хорошо»?
5. **Записи в правильном порядке** (newest first)?

**Часть 2: практика**

Следующая сессия — сделайте ритуал:
1. **Start:** прочитайте последнюю запись. Что в Next?
2. **During:** держите WORK_LOG мысленно открытым. Что стоит записать?
3. **End:** запишите. 5–15 строк. С Next.

Повторите 5 сессий. Через неделю посмотрите на WORK_LOG — стало ли проще возвращаться к проекту?

**Часть 3: сравнение.**

Возьмите сессию, которую вы **не** записали. Что вы помните о ней через неделю? Что потеряно?
Возьмите сессию, которую записали. Что восстанавливается? Что помогает?
**Если разница очевидна** — вы поняли ценность session lifecycle.

### 12.12 Метрики

Что можно измерять:

| Метрика                    | Что показывает                    |
| -------------------------- | --------------------------------- |
| **Sessions per week**      | Активность                        |
| **Average session length** | Как долго работаете без перерыва  |
| **Sessions per issue**     | Насколько issue крупная           |
| **Gap between sessions**   | Как часто возвращаетесь к проекту |

**Не превращайте в KPI.** Это для самопонимания.

**Что полезно знать:**
- Если sessions per issue > 10 — issue слишком крупная. Разбивайте.
- Если gap между сессиями > 7 дней — проект «остывает». Плохой знак.
- Если average session length < 30 минут — вас что-то отвлекает.

### 12.13 Связь с issue lifecycle

**Issue lifecycle** — про задачу. **Session lifecycle** — про время.

```text
Issue:    Start ──────────────────────── End
              │                          │
Sessions:    S47  S48  S49  S50  S51   S52
              │    │    │    │    │     │
              └────┴────┴────┴────┴─────┘
              каждая — блок в WORK_LOG
```

Одна issue — много сессий. Каждая сессия — отдельная запись.

**Правило:** не пытайтесь уместить всю issue в одну сессию. Работайте по сессиям. Каждая сессия — отдельный блок.

### 12.14 Что дальше

В следующей главе — **When CI fails**. Это отдельный workflow: CI упал, надо диагностировать и починить.
## Глава 13. Что делать если CI упал?

### 13.1 Что это за workflow

**CI failed workflow** — последовательность действий, когда CI-пайплайн на PR упал.
Триггер: вы запушили изменения, GitHub Actions (или другой CI) показал красный статус.
**Это не инцидент** Прод работает, пользователи не страдают. Это **рабочая проблема**: PR не может быть смержен, пока CI красный.
**Срочность:** P2–P3. Не «горит», но блокирует.
### 13.2 Три категории CI-падений

Не все падения одинаковы. Есть три категории, и подход к каждой разный.
#### Категория 1: очевидные

CI упал по причине, которую видно сразу:
- Lint: 3 offenses;
- Опечатка в тесте;
- Забыли добавить файл;

**Что делать:** исправить, запушить, дождаться зелёного CI

**Время:** 5–15 минут

#### Категория 2: диагностируемые

CI упал, но причина не сразу ясна. Нужна диагностика:
- Тест флакует (падает через раз);
- Timeout на одном job;
- Версия зависимости изменилась;

**Что делать:** открыть `_ci.md`, найти в Common failures, воспроизвести локально, исправить.

**Время:** 15 минут – 2 часа.
#### Категория 3: загадочные

CI упал, но **непонятно почему**:
- Локально всё работает;
- В логах нет явной ошибки;
- Проблема воспроизводится через раз;

**Что делать:** систематическая диагностика. См. секцию 13.6.

**Время:** часы. Иногда — дни.

### 13.3 Workflow

```text
CI red
  │
  ▼
1. Открыть _ci.md
  │
  ▼
2. Определить упавший workflow
  │
  ▼
3. Прочитать логи
  │
  ▼
4. Найти в Common failures
  │
  ├─── Нашёл ──→ Применить fix ──→ Push ──→ CI green? ✓
  │
  └─── Не нашёл ──→ Воспроизвести локально
                       │
                       ├─── Воспроизвёл ──→ Диагностика ──→ Fix ──→ Push ──→ ✓
                       │
                       └─── Не воспроизвёл ──→ Секция 13.6
                                                   │
                                                   ▼
                                            Систематическая диагностика
                                                   │
                                                   ▼
                                               Fix ──→ Push ──→ ✓
                                                   │
                                                   ▼
                                            Записать в _ci.md
```
### 13.4 Шаг 1: открыть `_ci.md`

Прежде чем лезть в логи — откройте `_ci.md`

**Что там:**
- Таблица `Workflows` — какие workflow'ы есть, что проверяют, как воспроизвести локально;
- Таблица `Common failures` — типичные ошибки с причинами и фиксами;
- Секция `Local reproduction` — как запустить CI локально;
- Секция `When CI passes locally but fails remotely` — пять проверок;

**Почему это первое действие:**
- Если ошибка уже в `Common failures` — вы найдёте fix за 30 секунд;
- Если нет — узнаете, как воспроизвести локально;
- Если `_ci.md` устарел — это тоже сигнал: обновить его после решения;

**Пример:**

CI упал с `Lint: 3 offenses`. Открываете `_ci.md`, ищете в Common failures:

| Failure            | Likely cause     | Fix                      |
| ------------------ | ---------------- | ------------------------ |
| `Lint: 3 offenses` | Style violations | `bundle exec rubocop -a` |

**Готово.** Запускаете `bundle exec rubocop -a`, коммитите, пушите.

### 13.5 Шаг 2: диагностика

Если в `_ci.md` ошибки нет — начинается диагностика.
#### Определить упавший workflow

На GitHub: вкладка Actions → красный job.

На GitLab: Pipelines → failed job.

**Что смотреть:**
- **Какой workflow** упал. `test.yml`? `lint.yml`? `deploy-staging.yml`?
- **Какой job** упал. Один? Несколько? Все?
- **Какой шаг** упал. Установка зависимостей? Тесты? Сборка?

**Если упало всё сразу:**
- Скорее всего, проблема общая: версия Ruby, сеть, кэш;
- Проверьте установку зависимостей — часто там;

**Если упал один job:**
- Проблема локальная для этого job'а;
- Смотрите логи именно этого job'а;
#### Прочитать логи

Логи — главный источник информации. Читайте **снизу вверх**: ошибка обычно в конце.

**Что искать:**
- **`Error`**, **`Failure`**, **`Exception`** — явные ошибки;
- **`Expected X, got Y`** — несоответствие;
- **`Timeout`** — превышение лимита;
- **`undefined method`**, **`NameError`** — ошибки кода;
- **`Cannot find`**, **`No such file`** — отсутствующие зависимости;

**Что НЕ искать:**
- Warning'и, если они не критичны;
- Info-сообщения;
- «Успешные» логи перед ошибкой;

**Пример:**

```text
Run bundle exec rspec
...
Failures:
  1) Parser handles streaming input
     Failure/Error: expect(parser.parse(stream)).to eq(expected)
     NoMethodError:
       undefined method `read_chunks' for #<StreamReader>
     # ./lib/parser.rb:42:in `parse'
```

**Что видно:** `StreamReader` не имеет метода `read_chunks`. Кто-то переименовал метод, но не обновил вызов в `Parser#parse`.
#### Найти в Common failures

Откройте `_ci.md`, таблицу Common failures. Поищите по ключевым словам из ошибки.
**Если нашли** — примените fix
**Если нет** — переходите к следующему шагу
#### Воспроизвести локально

Запустите ту же команду, что упала в CI:

```bash
bundle exec rspec
```

**Результат:**
- **Упало так же** → воспроизвели. Диагностика упрощается
- **Прошло** → не воспроизвели. См. секцию 13.6

### 13.6 Когда локально работает, а CI — нет

Это **самая частая и самая неприятная** категория. Локально тесты зелёные, на CI — красные.
Пять проверок из `_ci.md`:
#### Проверка 1: версии

**Проблема:** локально Ruby 3.3, на CI — 3.2. Или Node 20 vs 18.

**Как проверить:**
- Локально: `ruby --version`, `node --version`
- В CI: посмотрите YAML workflow, шаг «Setup Ruby» или «Setup Node»

**Часто источник:** `.ruby-version`, `.tool-versions`, `.nvmrc` — но CI не читает эти файлы, если явно не указано.

**Fix:** обновить версию в workflow или локально.
#### Проверка 2: environment variables

**Проблема:** CI имеет env vars, которых нет локально. Или наоборот.

**Как проверить:**
- В workflow: секция `env:`, `secrets:`
- Локально: `printenv | sort`

**Часто источник:** тесты зависят от `DATABASE_URL`, `API_KEY`, которых нет локально.

**Fix:** добавить переменные в локальный `.env` или пометить тесты как skip, если env отсутствует.
#### Проверка 3: параллелизм

**Проблема:** CI запускает тесты в параллель. Race conditions проявляются там, но не локально.

**Как проверить:**
- В workflow: `parallel:`, `matrix:`.
- Посмотрите на shared state в тестах: БД, файлы, time.

**Часто источник:** тесты, которые не изолированы.

**Fix:** изолировать тесты. Каждый тест — своя БД, свой файл, свой Time.
#### Проверка 4: кэш

**Проблема:** CI-кэш устарел. Старая версия `Gemfile.lock` или `package-lock.json`.

**Как проверить:**
- В workflow: `actions/cache` или аналог.
- Посмотрите, что именно кэшируется.

**Fix:** очистить кэш в настройках CI. Или добавить версию в ключ кэша:

```yaml
- uses: actions/cache@v4
  with:
    key: ${{ runner.os }}-gems-${{ hashFiles('Gemfile.lock') }}
```

**Обратите внимание на `hashFiles`** — ключ меняется при изменении lock-файла.
#### Проверка 5: debug logging

**Проблема:** обычные логи не показывают, что произошло.
**Fix:** включить подробные логи.

Для GitHub Actions:

```yaml
- name: Enable debug logging
  run: echo "ACTIONS_STEP_DEBUG=true" >> $GITHUB_ENV
```

Или через secrets: `ACTIONS_STEP_DEBUG=true`.

**После включения:** перезапустите job. Логи станут подробнее.
### 13.7 Частые причины

Пять типов CI-падений, которые встречаются чаще всего.
#### Тип 1: lint

```text
Lint: 3 offenses
```

**Причина:** стиль нарушен
**Fix:** `bundle exec rubocop -a` (или `--autocorrect`)
**Профилактика:** настройте pre-commit hook с lint

#### Тип 2: flaky tests

```text
Parser handles edge case — failed
```

**Причина:** тест падает через раз. Зависит от порядка, времени, БД.
**Fix:**
1. **Воспроизведите** локально, запустив тест в цикле:
   ```bash
   for i in {1..50}; do bundle exec rspec spec/parser_spec.rb || break; done
   ```
2. **Найдите** источник нестабильности
3. **Изолируйте** тест
4. **Если не получается** — пометьте `skip` или `pending` и создайте issue

**См. также:** `_backlog.md`, секция Tech debt.
#### Тип 3: timeout

```text
Timeout: job exceeded 60 minutes
```

**Причина:** тест или сборка занимает слишком много времени.
**Fix:**
- **Увеличить лимит** — если тест легитимно долгий
- **Найти медленный тест** — профайлер (`rspec --profile`)
- **Параллелизовать** — если тестов много
- **Пропустить** — если тест долгий и не критичный
#### Тип 4: зависимости

```text
Could not find gem 'foo' in locally installed gems
```

**Причина:** новая зависимость не попала в lock-файл. Или lock устарел.
**Fix:**
```bash
bundle install
git add Gemfile.lock
git commit -m "Update Gemfile.lock"
git push
```
**Профилактика:** всегда коммитьте lock-файл.
#### Тип 5: environment

```text
Error: DATABASE_URL is not set
```

**Причина:** переменная окружения не задана в CI.
**Fix:**
- Добавить в workflow:
  ```yaml
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
  ```
- Или настроить secrets в настройках репозитория

### 13.8 После фикса

Когда CI стал зелёным — **не расслабляйтесь сразу**. Три действия.
#### Действие 1: записать в `_ci.md`

**Правило:** если диагностика заняла **больше 15 минут** — запишите.
**Куда:** таблица Common failures.
**Формат:**

| Failure                      | Likely cause                     | Fix                   |
| ---------------------------- | -------------------------------- | --------------------- |
| `NoMethodError: read_chunks` | Renamed method, call not updated | `grep -r read_chunks` |

**Почему:** через полгода та же ошибка может повториться. `_ci.md` сэкономит время.
#### Действие 2: записать в WORK_LOG

Стандартная запись сессии:

```markdown
## 2026-09-22

### Session 48 — Fix CI on #123

| #   | What          | Files           | Status      | Complexity |
| --- | ------------- | --------------- | ----------- | ---------- |
| 123 | Fix CI: renamed method call | `lib/parser.rb` | in-progress | low        |
| **Problem:** CI failed with `NoMethodError: read_chunks`. Forgot to |
| update `Parser#parse` after renaming `StreamReader#read_chunks`.    |
| **Next:** Continue with #123.                                       |

---
```

**Problem** здесь — главная секция. Что произошло, как нашли, как починили.
#### Действие 3: профилактика (если возможно)

**Что можно сделать**, чтобы проблема не повторилась:
- **Pre-commit hook** — если это lint;
- **CI-проверка** — если это забытый lock-файл;
- **Рефакторинг** — если это хрупкий тест;
- **Update workflow** — если это версия зависимости;

**Задача в `_backlog.md`:**

```text
| B-042 | P2 | Add pre-commit hook for lint | — | — | Read pre-commit docs |
```

**Не всё требует профилактики.** Иногда это разовый случай. Но если проблема повторяется — профилактика обязательна.

### 13.9 Худший случай: CI сломан, а прод — нет

Иногда CI падает по причинам, не связанным с вашим кодом:
- GitHub Actions лежит;
- Runner сломался;
- Квота исчерпана;

**Что делать:**
1. **Проверьте статус CI-платформы** [githubstatus.com](https://www.githubstatus.com/) — для GitHub;
2. **Если проблема на стороне платформы** — подождите. Ничего не сделаете.
3. **Если это runner** — перезапустите job;
4. **Если квота** — свяжитесь с админом или перейдите на платный план;

**Не паникуйте.** CI красный, но не из-за вашего кода — не ваша проблема.

### 13.10 Что если CI никогда не был зелёным

Ситуация: вы только что клонировали проект, запушили изменение, CI упал. Вы **не знаете**, было ли зелёно до вас.

**Что делать:**
1. **Проверьте историю CI** — на GitHub: Actions → All workflows. Найдите последний зелёный коммит на master.
2. **Если master тоже красный** — проблема не в вашем PR. Возможно, это known issue.
3. **Проверьте issues** — может быть открыт issue про сломанный CI;
4. **Если нужно** — создайте issue;

**Не пытайтесь починить CI всего проекта в рамках своего PR.** Это отдельная работа.
### 13.11 Anti-patterns

**Anti-pattern 1: игнорировать CI**
«Красный — ну и ладно, смержат». Не смержат. CI — gate. Если игнорируете — PR не пройдёт.

**Anti-pattern 2: перезапускать без разбора**
«CI упал — нажму «Re-run»». Иногда помогает (flaky test). Но если упало дважды — это не флак, это проблема. Разбирайтесь.

**Anti-pattern 3: отключить проверку**
«Отключу lint в workflow, чтобы CI прошёл». Не надо. Проверка нужна.

**Anti-pattern 4: починить только симптом**
«Тест падает — добавлю `sleep 5`». Симптом уйдёт, проблема останется. Ищите **причину**.

**Anti-pattern 5: не записывать в `_ci.md`**
Через полгода — та же ошибка, снова час диагностики.

**Anti-pattern 6: не обновлять `_ci.md` при изменении workflow**
Добавили новый workflow — забыли записать в `_ci.md`. Через месяц `_ci.md` описывает то, чего нет.

**Anti-pattern 7: длинная диагностика без записи**
3 часа искали причину. Нашли. Не записали. Через месяц — снова 3 часа.

**Anti-pattern 8: закрыть PR из-за CI**
CI упал — «ладно, заброшу этот PR». Не надо. Почините.

### 13.12 Связь с другими файлами

```text
                _ci.md
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
_commands.md   _codestyle.md   _troubleshooting.md
(как запустить) (lint rules)   (если локальная проблема)
     │
     └─→ WORK_LOG.md    (запись сессии)
         _backlog.md    (профилактика)
         runbooks/      (если CI деплоит в prod и там инцидент)
```

**`_ci.md` — центральный файл.** Он открывается первым и обновляется последним.
### 13.13 Упражнение

**Часть 1: разбор последнего падения**

Вспомните последний раз, когда CI упал на вашем PR. Ответьте:
1. **Сколько времени** заняла диагностика? 5 минут? Час? Три?
2. **Открывали ли вы `_ci.md`?** Если нет — почему?
3. **Была ли эта ошибка в Common failures?** Если да — сэкономили время? Если нет — записали после?
4. **Могли бы вы предотвратить** это падение? Pre-commit hook? Локальный запуск?

**Часть 2: обновление `_ci.md`**

Откройте ваш `_ci.md`. Ответьте:
1. **Все ли workflow'ы** перечислены? Сверьте с `.github/workflows/`;
2. **У каждого ли workflow есть `Reproduce locally`?**
3. **Сколько строк в Common failures?** Если меньше 3 — допишите те, что помните;
4. **Проверьте ссылку** на платформу и workflow-файлы;

**Часть 3: профилактика.**

Возьмите **одну** частую CI-ошибку из вашей практики. Что можно сделать, чтобы она не повторялась?
- **Pre-commit hook** — для lint, форматирования;
- **Dependabot** — для устаревших зависимостей;
- **CI-проверка** — для забытых lock-файлов;
- **Фикс flaky теста** — если он стабильно нестабильный;

Создайте задачу в `_backlog.md`, если это не быстро.

### 13.14 Метрики

Что можно измерять:

| Метрика                   | Что показывает          |
| ------------------------- | ----------------------- |
| **CI failure rate**       | Как часто CI падает     |
| **Time to fix**           | От красного до зелёного |
| **Flaky rate**            | Процент flaky падений   |
| **Common failures count** | Размер таблицы          |

**Что полезно знать:**
- **CI failure rate > 20%** — что-то не так. Много flaky или плохие проверки.
- **Time to fix > 1 час** — `_ci.md` плохо работает.
- **Flaky rate > 5%** — серьёзная проблема.

**Не превращайте в KPI** Это для понимания.
## Глава 14. Инциденты в продакшене

### 14.1 Что такое инцидент

**Инцидент** — событие, из-за которого **пользователи** испытывают проблемы.

Примеры:
- Прод-сервер не отвечает;
- Оплата не проходит;
- Данные повреждены;
- API отвечает с ошибками;

**Ключевое отличие от других проблем:** инцидент влияет на **пользователей**, а не только на разработчиков.

| Проблема             | Кто страдает     | Срочность |
| -------------------- | ---------------- | --------- |
| Локально не работает | Разработчик      | P3        |
| CI упал              | Разработчик      | P2–P3     |
| **Инцидент в prod**  | **Пользователи** | **P0–P1** |

**Что меняется при инциденте:**
- **Срочность.** Каждая минута = потерянные деньги / пользователи;
- **Давление.** Вы работаете **под стрессом**;
- **Аудитория.** Не только вы, но и вся команда;
- **Цена ошибки.** Ошибка в проде = ещё больше проблем;

### 14.2 Три фазы инцидента

```text
    Detection          Response           Post-incident
       │                  │                    │
       ▼                  ▼                    ▼
   Алерт сработал    Runbook → Fix        Разбор → ADR
                     эскалация            → backlog
```

**Detection:** как узнали об инциденте. Обычно — алерт или жалоба пользователей.
**Response:** что делаем. Runbook — центральный инструмент.
**Post-incident:** что после. Разбор, обновление runbook, ADR.
### 14.3 Detection

#### Как узнаём

Три источника:

**1. Автоматический алерт.**

Настроен мониторинг (Datadog, Grafana, Prometheus). Метрика превысила порог → алерт в Slack, PagerDuty, email.

**Пример:**

> Alert: `PostgresReplicationLag > 60s` for more than 5 minutes.

**2. Жалоба пользователя.**

Пользователь пишет в поддержку: «Сайт не работает». Поддержка уведомляет команду.

**3. Случайное обнаружение.**

Разработчик смотрит в Grafana или логи и замечает проблему.

#### Что делать при detection

**Первое действие — оценить серьёзность.** Три вопроса:
1. **Сколько пользователей затронуто?** Все? Часть? Один?
2. **Что именно не работает?** Полный простой? Частичная деградация?
3. **Есть ли обходной путь?** Могут ли пользователи обойти проблему?

**На основе ответов — определить severity:**

| Severity | Что                              | Пример                     |
| -------- | -------------------------------- | -------------------------- |
| **P0**   | Полный простой, все пользователи | Сайт не открывается        |
| **P1**   | Частичная деградация, критично   | Оплата не проходит         |
| **P2**   | Деградация, не критично          | Медленная загрузка отчётов |
| **P3**   | Минорная проблема                | Мелкий визуальный баг      |

**P0 и P1 — немедленное реагирование.** P2 — в рабочее время. P3 — обычный backlog.

#### Кого уведомить

При P0:
- **Канал `#incidents`** (или аналог) — «Starting incident response. Alert X, severity P0.»
- **On-call lead** — звонок или mention
- **Команда** — не обязательно всех, но тех, кто может помочь

**Принцип:** лучше переуведомить, чем недоуведомить. Лишние люди отпишутся; отсутствие нужного — потеря времени.
### 14.4 Response

#### Шаг 1: открыть `runbooks/index.md`

Первый шаг — открыть индекс runbook'ов.

```text
runbooks/
├── index.md         ← открываем
├── _runbook.md
├── db-failover.md
├── rollback-release.md
└── rotate-api-key.md
```

**Что ищем:** подходящий runbook по симптому.

| Симптом                       | Runbook               |
| ----------------------------- | --------------------- |
| БД не отвечает                | `db-failover.md`      |
| Последний релиз что-то сломал | `rollback-release.md` |
| Утёк API-ключ                 | `rotate-api-key.md`   |

**Если подходящего runbook'а нет** — см. секцию 14.8.

#### Шаг 2: следовать runbook'у

Открываем runbook. Читаем **When to use** — убеждаемся, что он подходит.

**Читаем Prerequisites** — что нужно до начала:
- Access: VPN, SSH, credentials;
- Tools: `psql`, `kubectl`, `aws-cli`;
- People to notify: кого предупредить перед началом;

**Если чего-то нет** — либо получаем, либо эскалируем.
#### Шаг 3: выполнять шаги

**По порядку. Не пропускать. Не импровизировать.**

Каждый шаг — команда + Expected. Запускаем, сверяем с Expected.

**Если Expected совпало** — идём дальше.
**Если нет** — стоп. Смотрим секцию «If it doesn't work».

**Главное правило:** `Stop. Do not improvise.`

Под давлением хочется «попробовать ещё что-то». **Не надо.** Импровизация в проде без понимания — путь к ещё большей аварии.
#### Шаг 4: эскалация

Если runbook не сработал — эскалация.

**Кому:** указано в runbook'е, секция «If it doesn't work».
**Как:** позвонить, mention в Slack, открыть тикет. Зависит от канала.

**Что сообщить:**
- Какой инцидент;
- Что попробовали (по runbook'у);
- Какие результаты;
- Что нужно;

**Не «ничего не работает, помогите»** — конкретика.

**Пример:**

> P0 incident: DB failover. Followed `db-failover.md`. Step 3 failed —  
> replica not in `streaming` state. `pg_stat_replication` shows  
> `catchup`. Waiting 10 min, no change. Need DBA help.

#### Шаг 5: зафиксировать логи

Пока инцидент активен — **собираем логи**. Пригодятся для post-mortem.

**Что собирать:**
- Логи приложения;
- Метрики (Grafana, Datadog screenshots);
- Вывод команд;
- Таймлайн событий;

**Куда:** временный файл или канал `#incidents`.
**Если не собирать сразу** — потом не восстановите.
#### Шаг 6: зафиксировать в WORK_LOG (опционально)

Если инцидент длится больше часа — запишите в WORK_LOG.

**Краткая запись:**

```markdown
## 2026-09-22

### Session 48 — P0 incident: DB failover

Alert `PostgresReplicationLag > 60s`. Followed `db-failover.md`.
Failover completed at 15:30. Service restored.
**Problem:** Replica lag grew to 5 min due to slow query.
**Next:** Post-incident review.
---
```

**Не полный формат** — инцидент ещё не закончен. Полная запись — после post-incident.
### 14.5 Verification

После того как выполнили runbook — **проверяем**, что инцидент разрешён.

**Стандартные проверки:**

- □ Алерт исчез в мониторинге
- □ Метрика вернулась в норму
- □ Приложение отвечает 200
- □ Логи не показывают ошибок
- □ Ключевая функциональность работает

**Для каждого инцидента — свои проверки.** Они в runbook'е, секция Verification.

**Не расслабляйтесь сразу.** Метрика может вернуться, но проблема остаться. Подождите 5–10 минут, проверьте ещё раз.
### 14.6 Что делать после

Инцидент разрешён. Что дальше — три действия.
#### Действие 1: сообщить

**В канал `#incidents`:**

> Incident resolved. DB failover completed at 15:30. Service restored.  
> Post-incident review scheduled.

**Заинтересованным лицам** — email или Slack DM: «Инцидент решён, детали позже».
#### Действие 2: post-incident review

**В течение 48 часов** — разбор инцидента
**Кто участвует:** те, кто реагировал + смежные команды

**Что обсуждаем:**
- Что произошло (timeline);
- Почему произошло (root cause);
- Что сработало хорошо;
- Что можно улучшить;

**Формат:** не «кто виноват», а «как улучшить». Это **blameless post-mortem**.
**Артефакты:**
- Запись в `_decisions.md` (если инцидент повлёк изменение процесса);
- Задачи в `_backlog.md`;
- Обновление runbook'а;
#### Действие 3: обновить runbook

Если процедура шла **не по runbook'у** — обновите runbook
**Пример:**
- В runbook'е шаг 3 — команда `pg_ctl promote`;
- В реальности использовали `pg_ctlcluster 16 main promote`;
- Обновите runbook;
**Также:**
- Обновите `Last tested` в `index.md`;
- Добавьте новые pitfalls;
- Уточните `Expected`;

**Правило:** runbook улучшается после каждого использования.
### 14.7 Post-incident review

Разберём подробнее.
#### Timeline

**Первое:** восстановить timeline.

```text
14:55 — Alert PostgresReplicationLag > 60s
15:00 — On-call acknowledges
15:05 — Opened runbooks/db-failover.md
15:10 — Step 1 (check lag): lag = 5min
15:15 — Step 2 (check replica state): catchup
15:20 — Step 3 (trigger failover): failed
15:25 — Escalated to DBA
15:30 — DBA manual failover
15:35 — Service restored
15:40 — Alert cleared
```

**Источники:** логи CI, Slack, мониторинг, memory участников.
#### Root cause

**Что вызвало инцидент?**
- Медленный запрос → replication lag;
- Или: сбой сети → реплика отстала;
- Или: баг в коде → нагрузка;

**Пять почему (5 Whys):**
1. Почему лаг вырос? — Медленный запрос;
2. Почему запрос медленный? — Отсутствует индекс;
3. Почему нет индекса? — Забыли добавить в миграции;
4. Почему забыли? — Не было ревью миграции;
5. Почему не было? — Нет процесса;

**Root cause:** нет процесса ревью миграций

**Не останавливайтесь на первом уровне.** «Медленный запрос» — симптом. «Нет процесса» — причина.
#### Что сработало

Обязательно отметьте **что пошло хорошо**:
- Runbook был — не пришлось изобретать;
- Escalation сработала быстро;
- Команда координировалась;

**Зачем:** положительное подкрепление. И понимание, что сохранить.
#### Что улучшить

**Конкретные действия:**
1. **Добавить ревью миграций** → задача в `_backlog.md`;
2. **Обновить runbook** → добавить шаг про `pg_ctlcluster`;
3. **Настроить алерт** → на медленные запросы;
4. **Обновить ADR** → если это меняет процесс;

**Каждое действие** — с owner и deadline
#### Blameless

**Не ищем виноватого.** Ищем **системные проблемы**
- Плохо: «Иван забыл добавить индекс»;
- Хорошо: «Процесс не требует ревью миграций»;

**Почему:** если искать виноватого — люди начнут скрывать ошибки. Если искать систему — люди будут открыто разбирать.
### 14.8 Если нет runbook'а

Бывает: инцидент, для которого runbook'а нет.

**Что делать:**
1. **Импровизируйте осторожно.** Не экспериментируйте в проде без понимания;
2. **Эскалируйте.** Кто-то в команде знает систему лучше;
3. **Соберите информацию** — логи, метрики, состояние системы;
4. **После** — **напишите runbook**. Обязательно;

**Правило:** каждый инцидент без runbook'а → новый runbook
#### Аварийный runbook

Если инцидент серьёзный, а runbook'а нет — **пишите на ходу**:

```markdown
## Emergency runbook: <incident>

**Trigger:** <what happened>
**Steps taken:**

1. <action 1> → <result>
2. <action 2> → <result>
   **Resolution:** <how resolved>
   **Next:** Convert to formal runbook
```

После инцидента — преобразуйте в формальный runbook.
### 14.9 Когда инцидент — не ваш

Возможные ситуации:
- **Инцидент в сервисе, которым вы не владеете.** Уведомите владельца.
- **Инцидент в инфраструктуре** (AWS, GCP). Проверьте status page, эскалируйте.
- **Инцидент в стороннем API** (Stripe, Twilio). Уведомите, подождите.

**Не пытайтесь починить чужое.** Уведомьте, эскалируйте, документируйте.
### 14.10 Anti-patterns

**Anti-pattern 1: паника**
Бегать, кричать, метаться. Вредит больше, чем помогает.

**Anti-pattern 2: импровизация**
«Попробую что-нибудь ещё». Без понимания — риск усугубить.

**Anti-pattern 3: молчание**
Не уведомили команду. Работаете в одиночку. Команда не знает, что происходит.

**Anti-pattern 4: работа без runbook'а**
Игнорируете runbook — «я лучше знаю». Runbook — результат опыта. Следуйте.

**Anti-pattern 5: пропуск post-incident**
Разрешили — забыли. Через месяц — та же проблема.

**Anti-pattern 6: поиск виноватых**
«Кто это сделал?» Разрушает доверие.

**Anti-pattern 7: не записывать**
Инцидент был, разобрали, не записали. Через полгода — с нуля.

**Anti-pattern 8: слишком длинный runbook**
500 строк. Нечитаемо под давлением.

**Anti-pattern 9: runbook без тестирования**
Написан «по памяти». Не работает.

**Anti-pattern 10: prod changes без уведомления**
Внесли изменение в prod. Команда не знает. Второй инцидент.
### 14.11 Связь с другими файлами

```text
              runbooks/index.md
                    │
                    ▼
              runbooks/<scenario>.md
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
_env.md       _security.md      _backlog.md
(где что)     (секреты)         (follow-up)
     │
     └─→ _decisions.md   (post-incident ADR)
         WORK_LOG.md     (запись сессии)
         _ci.md          (если CI деплоит и что-то пошло не так)
```

**`runbooks/` — центральный инструмент.** Открывается первым, обновляется последним.
### 14.12 Упражнение

**Часть 1: ретроспектива**

Вспомните последний инцидент в вашей практике. Ответьте:
1. **Был ли runbook?** Если да — следовали ли ему?
2. **Сколько времени заняло реагирование?** 10 минут? Час? Больше?
3. **Что было root cause?** Не симптом, а причина.
4. **Что улучшили после?** Или забыли?

**Часть 2: аудит runbook'ов.

Откройте `runbooks/index.md`. Ответьте:
1. **Сколько runbook'ов?** Покрывают ли они **самые частые** инциденты?
2. **Все ли имеют `Last tested`?** Если нет — когда последний раз проверяли?
3. **У всех ли есть секция `If it doesn't work`?**
4. **Все ли команды работают?** Попробуйте в staging.

**Часть 3: тестирование**

Возьмите **один** runbook. Проведите **dry-run** в staging:

1. Откройте runbook;
2. Следуйте шагам;
3. Записывайте, где отклонились;
4. Обновите runbook;

**Правило:** непротестированный runbook — не runbook.
### 14.13 Метрики

| Метрика                         | Что показывает                   |
| ------------------------------- | -------------------------------- |
| **MTTD** (Mean Time To Detect)  | От начала до обнаружения         |
| **MTTR** (Mean Time To Resolve) | От обнаружения до разрешения     |
| **Incidents per month**         | Частота                          |
| **P0 count**                    | Сколько серьёзных                |
| **Runbook coverage**            | Процент инцидентов с runbook'ами |

**Что полезно знать:**

- **MTTR > 1 час** — runbook'и плохо работают;
- **Incidents per month растёт** — что-то системное;
- **Runbook coverage < 80%** — много инцидентов без процедур;

**Не превращайте в KPI.** Это для понимания.
### 14.14 Что дальше

В следующей главе — **Deep analysis**. Это workflow для случаев, когда нужно **исследовать**проект: аудит безопасности, поиск узких мест производительности, оценка техдолга. Результат — находки в `analysis/`
## Глава 15. Глубокий анализ

### 15.1 Что такое deep analysis

**Deep analysis** — целенаправленное исследование проекта с целью **найти проблемы, риски или возможности**.

Отличие от обычной работы:

| Обычная работа           | Deep analysis           |
| ------------------------ | ----------------------- |
| Решить конкретную задачу | Понять общую картину    |
| Issue → fix              | Исследование → findings |
| Известно, что делать     | Неизвестно, что найдём  |
| Ограничено scope         | Широкий охват           |
| Результат: PR            | Результат: `analysis/`  |

**Примеры deep analysis:**
- Аудит безопасности
- Поиск узких мест производительности
- Оценка техдолга
- Ревизия архитектуры
- Подготовка к рефакторингу

### 15.2 Когда запускать

Три триггера:

**1. Плановое исследование**
Раз в квартал / полгода — обзор проекта. Что накопилось? Что устарело?

**2. После крупного инцидента**
Что-то сломалось. Инцидент разрешили, но хочется понять — есть ли ещё подобные места?

**3. Перед большим изменением**
Планируется рефакторинг, миграция, новая фича. Сначала — анализ, потом — работа.

**Не запускайте** analysis:
- Без конкретной цели;
- Ради процесса;
- Когда нужно просто решить задачу;

**Deep analysis — это дорого.** Часы или дни. Запускайте, когда есть причина.
### 15.3 Отличие от issue

| Issue            | Deep analysis         |
| ---------------- | --------------------- |
| Решить проблему  | Найти проблемы        |
| Известная задача | Неизвестный результат |
| `issue/`         | `analysis/`           |
| PROJECT_SUMMARY  | Finding               |
| PR в конце       | Отчёт в конце         |

**Deep analysis может породить issues.** Один finding → одна задача в `_backlog.md`. Но сам анализ — не issue.
### 15.4 Workflow

```text
1. Определить цель
        │
        ▼
2. Ограничить scope
        │
        ▼
3. Исследовать
        │
        ▼
4. Зафиксировать findings
        │
        ▼
5. Обновить analysis/index.md
        │
        ▼
6. Приоритезировать
        │
        ▼
7. Перенести в _backlog.md
```

Семь шагов. Разберём каждый.

### 15.5 Шаг 1: определить цель

**Что ищем?** Конкретный вопрос, на который хотим ответить.

**Примеры целей:**
- «Есть ли SQL-инъекции в коде?»
- «Какие запросы самые медленные?»
- «Где дублируется логика?»
- «Какие зависимости устарели?»

**Плохая цель:** «Провести аудит». Слишком широко. Не поймёте, когда закончить.

**Хорошая цель:** «Проверить безопасность работы с пользовательским вводом». Конкретно. Есть критерий завершения.

**Что делать с целью:** записать. В начало `analysis/index.md` как `Reports` — новый отчёт.
### 15.6 Шаг 2: ограничить scope

**Что включаем, что исключаем?**

Deep analysis легко расширяется. «Пока проверяю SQL — вижу проблему с N+1 — надо посмотреть производительность — а вот ещё...»

**Правило:** ограничить scope **явно**.

**Пример:**

```markdown
## Reports

| Date  | Report      | Scope                                | Status |
| ----- | ----------- | ------------------------------------ | ------ |
| 2026-09-22|[Security audit](./security-audit.md)|User input handling: forms, API, file uploads|open|
```

Scope: «User input handling: forms, API, file uploads». Не «весь проект».
**Почему это важно:**
- Без scope — анализ никогда не закончится;
- С scope — знаете, когда остановиться;
- Что вне scope — в отдельный отчёт, если найдётся важное;
### 15.7 Шаг 3: исследовать

Основная фаза. Как исследовать — зависит от цели.
#### Инструменты

**Для безопасности:**
- `brakeman` (Ruby), `bandit` (Python), `gosec` (Go);
- `bundle audit`, `npm audit` — уязвимости зависимостей;
- Ручной ревью опасных мест: SQL, `eval`, десериализация;

**Для производительности:**
- Профайлеры: `stackprof`, `rbspy`, `py-spy`;
- Slow query log в БД;
- `EXPLAIN ANALYZE` для запросов;
- Flamegraph;

**Для техдолга:**
- `rubocop --format offenses`, `eslint --format json`;
- `flog`, `flay` — сложность;
- Метрики: размер файлов, длина методов, дублирование;

**Для архитектуры:**
- Граф зависимостей: `bundle viz`, `madge`;
- Ручной обзор модулей;
- Циклы в зависимостях;
#### Как записывать

**Не пытайтесь держать всё в голове.** Записывайте сразу.
**Формат:** временные заметки. Потом — оформите в findings.

**Пример заметок:**

```text
- lib/auth/session.rb:42 — `eval` на user input. Risk: RCE.
- app/controllers/users_controller.rb:120 — N+1 при list users.
- Gemfile: `rails 6.1.4` — CVE-2023-1234.
- app/models/user.rb — 800 lines. Method `authenticate` — 120 lines.
```

**Правило:** каждая находка — с указанием файла и строки.
#### Сколько времени

Deep analysis может занять:
- **Малый анализ** (1 тема) — 2–4 часа;
- **Средний** (модуль, компонент) — 1–3 дня;
- **Большой** (весь проект) — неделя+;

**Не растягивайте.** Если анализ длится больше недели — вы либо выбрали слишком широкий scope, либо застряли на деталях.
#### Прерывание

Deep analysis может прерываться на другие задачи. Это нормально.
**Записывайте состояние:**
- Что уже проверили.
- Что нашли.
- Где остановились.

**В WORK_LOG:**

```markdown
## 2026-09-22

### Session 48 — Security audit (part 1)

Started security audit on user input handling. Covered forms and API.
Found 3 issues:

- `eval` in `session.rb:42`
- SQL injection risk in `search.rb:88`
- Missing CSRF on `webhooks_controller.rb`
  **Next:** Continue with file uploads.

---
```
### 15.8 Шаг 4: зафиксировать findings

Каждая находка → отдельный файл в `analysis/`.
#### Файл finding

Из шаблона `_finding.md`:

````markdown
---
type: finding
title: "Eval on user input in Session"
description: "Direct eval on params allows RCE"
severity: high
status: todo
timestamp: 2026-09-22
tags: [finding, security]
---

# Finding: Eval on user input in Session

## Summary

`Auth::Session#load` calls `eval` on a value taken directly from
`params[:session_data]`. An attacker can inject Ruby code executed on
the server.

## Evidence

- File: `lib/auth/session.rb:42`
- Code:
  ```ruby
  def load
    eval(params[:session_data])
  end
  ```
- Repro: `curl -X POST /login -d 'session_data=system("id")'`

## Impact

Remote code execution. Attacker can execute any command on the server
with app's privileges.

## Recommendation

Use `JSON.parse` instead of `eval`. Validate and sign session data
before parsing.

## Effort estimate

M

## Related

- Report: [Security audit 2026-09](./security-audit.md)
- Issue: (create after prioritization)
````
#### Формат полей

**Title** — короткий, конкретный. «Eval on user input» — хорошо. «Security issue» — плохо.
**Description** — одно предложение. Развёрнутая версия title.
**Severity** — high / medium / low. См. секцию 15.9.
**Status** — todo / in-progress / addressed / wontfix / duplicate
**Summary** — 1–2 предложения. Что нашли.
**Evidence** — файл, строки, код, repro. Без evidence finding — гипотеза
**Impact** — что произойдёт, если не чинить. Это **критично**
**Recommendation** — что делать. Конкретное действие
**Effort estimate** — S / M / L
**Related** — ссылки на report, issue, ADR
#### ID findings

**Сквозной, не переиспользуется.**
- `F-001`, `F-002`, ... `F-N`.
- Удалили F-003 — следующий всё равно F-004.

**Имя файла:** `F-XXX-<short-title>.md`. Например: `F-001-eval-injection.md`.
### 15.9 Severity vs Priority

**Важное различие.**

| Severity                    | Priority         |
| --------------------------- | ---------------- |
| Насколько серьёзна проблема | Когда её чинить  |
| Объективно                  | Субъективно      |
| Оценка ущерба               | Оценка срочности |
| Не меняется со временем     | Может меняться   |

**Severity:**
- **high** — RCE, утечка данных, падение prod;
- **medium** — часть функциональности сломана, ошибки в логах;
- **low** — косметика, потенциальные проблемы;

**Priority:** берётся из `_backlog.md` (P0–P3).

**Пример:** finding severity=high, priority=P3. Проблема серьёзная, но редко проявляется. Можно отложить.
**Другой пример:** finding severity=low, priority=P0. Проблема мелкая, но блокирует релиз.
**Severity отвечает на «что если не чинить». Priority отвечает на «когда чинить».**
### 15.10 Шаг 5: обновить `analysis/index.md`

Каждое finding → строка в сводной таблице.

```markdown
## Findings summary

| ID  | Severity| Finding    | Report                                  | Status |
| --- | ------- | ---------- | --------------------------------------- | ------ |
| F-001|high    | Eval on user input|[link](./F-001-eval-injection.md) | todo   |
| F-002|high    |SQL injection in search|[link](./F-002-sql-injection.md)|todo  |
| F-003|medium  | Missing CSRF on webhooks | [link](./F-003-csrf.md)|todo   |
| F-004|medium  | N+1 in users list |[link](./F-004-n-plus-one.md)  | todo   |
| F-005 | low | Duplicate validators | [link](./F-005-validators.md) | todo   |
```

**Порядок:** по severity (high сверху). Или по дате обнаружения.
**Report:** ссылка на отчёт (если finding часть большого отчёта).
**Status:** синхронизирован с frontmatter finding.
### 15.11 Шаг 6: приоритезировать

Findings найдены. Что **делать**?

**Три категории:**

**1. Чинить сейчас.**
- Severity high, легко чинить;
- Или: severity любая, но блокирует что-то;

**2. Чинить потом.**
- Severity medium, effort большой;
- Или: severity low, но полезно;

**3. Не чинить.**
- Severity low, effort большой;
- Или: не стоит того;

**Для каждой категории:**
- «Сейчас» → task в `_backlog.md` с priority P0–P1;
- «Потом» → task в `_backlog.md` с priority P2–P3;
- «Не чинить» → finding.status = `wontfix`. Обоснование — в finding;

**Пример решения:**

| Finding                    | Severity | Effort | Решение           |
| -------------------------- | -------- | ------ | ----------------- |
| F-001 Eval injection       | high     | S      | P0, чинить сейчас |
| F-002 SQL injection        | high     | S      | P0, чинить сейчас |
| F-003 CSRF                 | medium   | S      | P1                |
| F-004 N+1                  | medium   | M      | P2                |
| F-005 Duplicate validators | low      | S      | P3                |

**Не оставляйте всё.** Если 20 findings и все P0 — приоритезация сломана.
### 15.12 Шаг 7: перенести в `_backlog.md`

Каждый finding, который **чиним** — становится задачей.
**Формат:**

```text
| B-042 | P0 | Fix eval injection (F-001) | — | — | Read F-001, write test |
| B-043 | P0 | Fix SQL injection (F-002) | — | — | Read F-002, fix query |
| B-044 | P1 | Add CSRF on webhooks (F-003) | — | — | Add token validation |
| B-045 | P2 | Fix N+1 (F-004) | — | B-044 | Wait for B-044 |
| B-046 | P3 | Merge duplicate validators (F-005) | — | — | Refactor User model |
```

**Ссылка на finding** — в колонке `Item`. «Fix eval injection (F-001)»
**Связь:** finding остаётся в `analysis/` со статусом `in-progress` при старте работы
**После завершения:** finding.status = `addressed`. Строка в `_backlog.md` удаляется
### 15.13 Report vs Finding

Различие важно.

| Report              | Finding                   |
| ------------------- | ------------------------- |
| Крупный отчёт       | Одна находка              |
| Охватывает scope    | Один аспект               |
| `security-audit.md` | `F-001-eval-injection.md` |
| Содержит overview   | Содержит детали           |
| Один на анализ      | Много на отчёт            |

**Один report → много findings.**
#### Формат report
Report — это markdown-файл с `type: report` (или без специального типа):

```markdown
---
type: report
title: "Security Audit 2026-09"
description: "Audit of user input handling"
timestamp: 2026-09-22
tags: [report, security, audit]
---

# Security Audit 2026-09

## Scope

User input handling: forms, API, file uploads.

## Method

Manual review + `brakeman` scan + manual testing.

## Findings

- [F-001 Eval injection](./F-001-eval-injection.md) — high
- [F-002 SQL injection](./F-002-sql-injection.md) — high
- [F-003 CSRF](./F-003-csrf.md) — medium

## Summary

3 critical issues found. All in legacy code (pre-2024). Modern code
uses parameterized queries and Rails protections.

## Recommendations

1. Fix F-001 and F-002 immediately (P0).
2. Schedule F-003 for next sprint (P1).
3. Add automated security scan to CI.

## References

- [1] [OWASP Top 10](url)
```

**Report** — это **обзор**. Findings — **детали**.

**Не обязательно.** Если findings мало и они не связаны — report можно не делать. Только findings + index.
### 15.14 Прерывание и продолжение

Deep analysis может прерываться. Как возвращаться:
**1. Прочитать `analysis/index.md`.**
Список findings. Что уже нашли.

**2. Прочитать report.**
Если есть — что покрыто, что нет.

**3. Прочитать последнюю сессию в WORK_LOG.**
Где остановились.

**4. Продолжить с того же места.**
Не начинать заново. Использовать уже найденное.

**Пример записи в WORK_LOG:**

```markdown
## 2026-09-23

### Session 49 — Security audit (part 2)

Continued from Session 48. Covered file uploads. Found 2 more issues:

- Unrestricted file types in `uploads_controller.rb:30`
- Missing size limit in `uploads_controller.rb:45`
  **Next:** Wrap up audit; update `analysis/index.md`; prioritize.

---
```
### 15.15 Завершение

Когда анализ закончен:

**1. Все findings записаны**;
**2. `analysis/index.md` обновлён**;  
**3. Report написан** (если применимо);  
**4. Приоритеты расставлены**;
**5. Задачи в `_backlog.md`**;
**6. Report.status = `completed`**;

**Что дальше:**
- Работа по findings — обычные issues.
- Findings постепенно становятся `addressed`.
- Report можно архивировать после закрытия всех findings.
### 15.16 Частые ошибки

**Ошибка 1: analysis без цели.**
«Проведём аудит». Слишком широко. Не понятно, когда закончить.

**Ошибка 2: слишком широкий scope.**
«Проверить всё». Никогда не закончите. Ограничьте.

**Ошибка 3: findings без evidence.**
«Похоже, тут медленно» — не finding. С фактами — finding.

**Ошибка 4: findings без impact.**
«Нашли проблему» — и что? Без impact не приоритезировать.

**Ошибка 5: findings без recommendation.**
Что делать — не написано. Следующий читатель начнёт с нуля.

**Ошибка 6: Recommendation — «надо подумать».**
Не действие, а размышление. Нужен конкретный шаг.

**Ошибка 7: Severity = Priority.**
Путают. Severity — серьёзность. Priority — срочность.

**Ошибка 8: все findings — high.**
Если всё high — приоритезация не работает.

**Ошибка 9: нет ID.**
Finding без `F-XXX` невозможно сослаться.

**Ошибка 10: ID переиспользуются.**
Удалили F-001 → не надо делать новый F-001.

**Ошибка 11: findings в `index.md`, а не в отдельных файлах.**
Index — сводка. Детали — в файлах.

**Ошибка 12: дублирование с `_backlog.md`.**
Finding — «есть проблема». Backlog — «делаем X».

**Ошибка 13: статус не обновляется.**
Finding `addressed`, а в index.md — `todo`.

**Ошибка 14: findings не переносятся в backlog.**
Нашли — забыли. Через месяц никто не помнит.

**Ошибка 15: analysis никогда не заканчивается.**
Растёт бесконечно. Ограничивайте scope.

**Ошибка 16: findings годами в `todo`.**
Не чинят, не закрывают. Раз в квартал — ревизия.
### 15.17 Anti-patterns

**Anti-pattern 1: analysis ради analysis.**
«Хочу посмотреть на код». Без цели. Результат — ничего.

**Anti-pattern 2: findings в голове.**
Нашли, не записали. Через день забыли.

**Anti-pattern 3: analysis без границ.**
Бесконечное исследование. Ничего не закончено.

**Anti-pattern 4: findings без приоритезации.**
20 находок, все «надо починить». С чего начать?

**Anti-pattern 5: analysis вместо работы.**
Вечно анализировать, никогда не чинить.

**Anti-pattern 6: analysis как замена issue.**
«У нас есть issue #123» → нет, это finding. Issue — задача, finding — состояние.

**Anti-pattern 7: findings не обновляются.**
Нашли, починили, статус `todo`. Рассинхрон.
### 15.18 Связь с другими файлами

```text
                analysis/index.md
                      │
                      ▼
              analysis/F-XXX.md
                      │
     ┌────────────────┼────────────────┐
     ▼                ▼                ▼
_backlog.md      _decisions.md     _concepts.md
(задачи)         (решения)         (что изменилось)
     │
     └─→ WORK_LOG.md      (сессии)
         _files.md        (evidence)
         _security.md     (если про безопасность)
```

**`analysis/` — центральный узел.** Открывается при исследовании, обновляется по ходу, становится задачами.

### 15.19 Варианты по типам analysis

#### Security audit
- **Tools:** brakeman, bundle audit, ручной ревью;
- **Scope:** user input, authentication, authorization, secrets;
- **Severity:** high для RCE/SQLi, medium для CSRF/XSS, low для info leaks;
#### Performance analysis
- **Tools:** профайлеры, slow query log;
- **Scope:** критичные endpoints, hot paths;
- **Severity:** high для timeout, medium для slow, low для оптимизаций;
#### Tech debt assessment
- **Tools:** rubocop, flog, flay;
- **Scope:** модуль / компонент;
- **Severity:** high для блокирующих, medium для замедляющих, low для косметики;
#### Architecture review
- **Tools:** граф зависимостей;
- **Scope:** модули, компоненты;
- **Severity:** high для циклов, medium для coupling, low для naming;
### 15.20 Упражнение

**Часть 1: запустите малый analysis.**

Выберите **одну** тему. Например: «Проверить безопасность работы с файлами».
1. **Цель:** что ищем?
2. **Scope:** какие файлы, какие аспекты?
3. **Исследование:** 1–2 часа;
4. **Findings:** запишите каждую находку;
5. **Обновление:** `analysis/index.md`;
6. **Приоритезация:** что чинить?
7. **Backlog:** создайте задачи;

**Часть 2: ретроспектива существующих findings.**

Если у вас уже есть `analysis/`, ответьте:
1. **Сколько findings?** Все ли с evidence, impact, recommendation?
2. **Все ли имеют статус?** Как много `addressed`?
3. **Все ли связаны с backlog?** Или висят?
4. **Есть ли findings годами в `todo`?** Пора ревизовать.

**Часть 3: аудит недавнего рефакторинга.**

Вспомните крупный рефакторинг за последние 3 месяца. Ответьте:
1. **Проводили ли analysis перед ним?** Если нет — почему?
2. **Были ли сюрпризы?** Что нашли по ходу?
3. **Если бы провели analysis — сэкономили бы время?**

**Вывод:** для следующего крупного изменения — сначала analysis.
### 15.21 Метрики

| Метрика                   | Что показывает               |
| ------------------------- | ---------------------------- |
| **Findings per analysis** | Плотность проблем            |
| **High severity %**       | Критичность                  |
| **Addressed %**           | Скорость реакции             |
| **Avg age**               | Как долго findings живут     |
| **Backlog conversion**    | % findings, ставших задачами |

**Что полезно знать:**
- **High severity > 30%** — что-то системное;
- **Addressed < 50%** — findings не приоритезируются;
- **Avg age > 6 месяцев** — нужна ревизия;
### 15.22 Что дальше

В следующей главе — **Planning**. Это workflow для случаев, когда нужно **спланировать** работу: посмотреть на backlog, выбрать задачи, оценить сроки.
## Глава 16. Планирование

### 16.1 Что такое planning

**Planning** — периодический ритуал, когда вы **смотрите на будущее** и решаете, что делать дальше.
Отличие от других workflow:

| Workflow          | Про что                      |
| ----------------- | ---------------------------- |
| Issue lifecycle   | Про **одну** задачу          |
| Session lifecycle | Про **один** отрезок времени |
| CI failure        | Про **реакцию** на проблему  |
| Incident          | Про **реакцию** на аварию    |
| Deep analysis     | Про **исследование**         |
| **Planning**      | **Про выбор задач**          |

Planning — **проактивный**. Вы не реагируете на что-то, вы решаете, что делать.
### 16.2 Когда планировать

Три горизонта:
**Недельное планирование.**
Каждое воскресенье вечером или понедельник утром. 15–30 минут. Что делать на неделе?
**Спринтовое планирование** (если используете скрам).
Раз в 1–2 недели. 1–2 часа. Что войдёт в спринт?

**Квартальное планирование.**
Раз в квартал. Полдня. Какие крупные цели?
**Для solo-разработчика:** недельное — оптимально. Квартальное — если есть стратегические цели.

**Не планируйте слишком часто.** Ежедневное планирование — overkill. Планирование раз в месяц — слишком редко, backlog разрастается.
### 16.3 Роль `_backlog.md`

`_backlog.md` — **источник истины** для планирования.

**Что там:**
- **Items** — приоритезированные задачи (P0–P3);
- **Ideas** — сырые идеи без приоритета;
- **Tech debt** — технический долг;

**Чего там НЕТ:**
- **GitHub Issues** — подтверждённые задачи. Backlog — черновики;
- **Roadmap** — стратегия на квартал. Backlog — конкретные задачи;
- **Календарь** — сроки. Backlog — без дат (или с условными);

**Ключевое правило:** backlog — это **не roadmap**. Backlog — inbox для того, что «надо бы сделать». Roadmap — отдельный документ для стратегии.

### 16.4 Три горизонта планирования

```text
       Quarterly
       ┌───────────────────────────┐
       │ Цели на 3 месяца          │
       │ ┌───────────────────────┐ │
       │ │ Sprint / Week         │ │
       │ │ ┌───────────────────┐ │ │
       │ │ │ Daily / Session   │ │ │
       │ │ │ ┌───────────────┐ │ │ │
       │ │ │ │ Issue         │ │ │ │
       │ │ │ └───────────────┘ │ │ │
       │ │ └───────────────────┘ │ │
       │ └───────────────────────┘ │
       └───────────────────────────┘
```

**Issue** — атом. **Session** — когда работаем. **Week/Sprint** — что входит. **Quarterly** — зачем всё это.

**Планирование идёт сверху вниз:** цели квартала → задачи спринта → конкретные issues.

**Но не всегда.** Иногда снизу вверх: накопились issues → сгруппировали в спринт → поняли цель квартала.
### 16.5 Workflow планирования

```text
1. Прочитать backlog
        │
        ▼
2. Разобрать Ideas
        │
        ▼
3. Пересмотреть приоритеты
        │
        ▼
4. Оценить (effort)
        │
        ▼
5. Выбрать задачи
        │
        ▼
6. Создать issues
        │
        ▼
7. Обновить backlog
```

Семь шагов. Разберём.
### 16.6 Шаг 1: прочитать backlog

Откройте `_backlog.md`. Прочитайте:
- **Items** — что там сейчас?
- **Ideas** — что накопилось?
- **Tech debt** — что болит?

**Вопросы:**
1. **Сколько items?** Если 50+ — пора чистить;
2. **Есть ли P0?** Если да — почему они ещё не сделаны?
3. **Есть ли старые items?** Что висит больше месяца?
4. **Есть ли items без Next action?** Они «висят»;
### 16.7 Шаг 2: разобрать Ideas

Ideas — это **сырые мысли**. Раз в цикл их надо разобрать.

**Для каждой идеи:**
- **Актуальна?** Если нет — удалить;
- **Есть потенциал?** Если да — перевести в Items с приоритетом;
- **Это отдельная задача?** Если нет — слить с другой;
- **Слишком расплывчато?** Уточнить или удалить;

**Правило:** Ideas не должны жить в этом разделе больше месяца. Либо в Items, либо удалить.

**Пример:**

```text
## Ideas (unprioritized)
- Try Elixir for a small service.
- What if we used GraphQL instead of REST?
- Add dark mode.
```

**Ревизия:**
- **Elixir** — «не сейчас, но интересно». Оставить.
- **GraphQL** — «уже обсуждали, решили не делать». Удалить. Или → ADR (rejected).
- **Dark mode** — «фича, хочу». → Items с приоритетом P3.
### 16.8 Шаг 3: пересмотреть приоритеты

Приоритеты не статичны. То, что было P1 месяц назад, может быть P3 сейчас.
**Вопросы для каждого item:**
1. **Всё ещё актуально?** Если нет — удалить.
2. **Всё ещё P0/P1?** Или можно понизить?
3. **P0 остались P0?** Если да — почему не сделаны? Может, не P0 на самом деле?
4. **P2/P3 ещё нужны?** Или удалить?

**Правило:** раз в цикл — **ревизия приоритетов**. Иначе backlog превращается в кладбище.

**Типичные проблемы:**
- **Много P0.** 5 P0 — сломанная приоритезация. P0 = «горит».
- **Всё P1.** Если всё важно — ничего не важно.
- **P3 живут годами.** Если не сделали за год — не сделаете. Удалить.
### 16.9 Шаг 4: оценить

**Effort** — насколько задача большая.

| Размер | Время    | Пример                    |
| ------ | -------- | ------------------------- |
| **S**  | < 1 день | Опечатка, мелкий фикс     |
| **M**  | 1–3 дня  | Небольшая фича            |
| **L**  | 3+ дня   | Крупная фича, рефакторинг |

**Для каждой задачи** — оценка S/M/L.

**Что если не уверены:**
- **По умолчанию — M.** Лучше переоценить.
- **Если задача > L** — **разбить**. Большие задачи плохо оцениваются.

**Оценка в часах — фикция.** S/M/L даёт порядок величины. Этого достаточно для планирования.

**Не используйте story points** для solo-проектов. Это для команд с несколькими разработчиками.
### 16.10 Шаг 5: выбрать задачи

Это **главный шаг**. Что делать в цикле?
#### Сколько выбрать

**Для недельного цикла:**
- 5–10 задач категории S;
- Или 2–3 задачи категории M;
- Или 1 задача категории L;

**Правило:** 30–50% времени на задачи + остаток на unexpected.

**Почему не 100%:** всегда появляется что-то срочное. Инцидент, баг, ревью. Если план на 100% — не хватит времени.
#### Баланс

**Идеальное соотношение:**
- **60%** — новая функциональность (features, improvements);
- **30%** — техдолг, рефакторинг;
- **10%** — эксперименты, исследования;

**Не 100% features.** Техдолг накопится. Через полгода — легаси, которое страшно трогать.
**Не 100% техдолг.** Без новых фич проект стагнирует.

**Пример недельного цикла:**

```text
- Feature: Add caching to Parser (M)
- Tech debt: Fix N+1 in User#posts (S)
- Bug: Fix flaky spec/parser_spec.rb (S)
- Chore: Update dependencies (S)
- Research: Explore streaming libraries (S)
```

5 задач, баланс: 1 feature, 1 tech debt, 1 bug, 1 chore, 1 research.
#### Критерии выбора

**Включайте задачу, если:**
- Она **P0/P1**;
- Она **разблокирует** другие;
- Она **быстрая** (S) и полезная;
- Она **интересна** (мотивация важна);

**Исключайте, если:**
- Она **заблокирована** (Blocked by);
- Нет **Next action**;
- Нет **ясности**, что делать;
### 16.11 Шаг 6: создать issues

Для каждой выбранной задачи — создайте issue на GitHub/GitLab.

**Формат issue:**

```markdown
# Add caching to Parser

## Problem

Parsing same file twice executes full pipeline. Cache would speed up
repeated parses by ~10x.

## Solution

Cache parsed AST keyed by file content hash. TTL 1 hour.

## Acceptance criteria

- [ ] Cache implemented
- [ ] Tests for cache hits/misses
- [ ] Benchmark shows improvement

## Related

From: _backlog.md B-042
```

**Ссылка на backlog** — важно. Связывает мотивацию (backlog) с задачей (issue).
**После создания issue:** удалите из `_backlog.md`. Backlog — для тех, **кто ещё не в работе**.
**Правило:** если задача в GitHub Issues — она **не** в backlog.
### 16.12 Шаг 7: обновить backlog

После создания issues:
	**1. Удалить выбранные items** из `_backlog.md`;
	**2. Переместить Ideas** — актуальные в Items, остальные удалить;
	**3. Обновить Tech debt** — что-то починили, что-то появилось;
	**4. Проверить Next action** — для оставшихся items;

**Итоговый backlog:** пустее, чем был. Если не пустее — вы добавили больше, чем взяли. Возможно, это нормально (идей много). Но следите за размером.
### 16.13 Если backlog разросся

**Симптом:** 50+ items. Планирование занимает час.

**Что делать:**

**1. Массовая чистка.**
- Items старше 6 месяцев → удалить или архивировать;
- Items без Next action → либо добавить, либо удалить;
- Ideas старше месяца → удалить;

**2. Разделить.**
- `_backlog.md` — активные items (P0–P2);
- `_backlog-ideas.md` — идеи, которые может быть;
- `_backlog-archive.md` — завершённые / отменённые;

**3. Пересмотреть процесс.**
- Почему так много? Вы добавляете всё подряд?
- Есть ли фильтр «стоит ли записывать»?

**Правило:** backlog должен быть **читаемым за 5 минут**. Если дольше — пора чистить.
### 16.14 Quarterly planning

Раз в 3 месяца — стратегическое планирование.

**Вопросы:**
1. **Какие цели** на квартал? (3–5 крупных целей)
2. **Какие фичи** критичны?
3. **Какой техдолг** должен быть починен?
4. **Какие метрики** улучшить?

**Артефакт:** документ с целями. Не в `_backlog.md` — отдельный файл.

**Пример:**

```markdown
# Q4 2026 Goals

1. **Performance:** Parse 100MB files in <3s (currently 10s).
2. **Security:** Fix all high-severity findings from audit.
3. **Quality:** Test coverage > 90% (currently 75%).
4. **Features:** Add streaming mode to public API.
5. **Tech debt:** Merge duplicate validators, refactor User model.

## How to measure

- Weekly: check benchmark, coverage.
- Monthly: review findings.
- End of quarter: retrospective.
```

**Quarterly planning — не для всех.** Solo-разработчик может не иметь стратегических целей. Team — обычно нужен.
### 16.15 Приоритезация

Как определить P0 vs P1 vs P2 vs P3?

**P0: Critical.**
- Ломает prod;
- Ломает CI;
- Блокирует релиз;
- Безопасность;

**Примеры:**
- «Прод не работает»;
- «Уязвимость с RCE»;
- «Тесты падают на master»;

**P1: Important.**
- Влияет на пользователей;
- Влияет на скорость разработки;
- Обещанная фича с дедлайном;

**Примеры:**
- «Медленная загрузка страницы»;
- «Хрупкий тест, падает через раз»;
- «Фича X для клиента Y»;

**P2: Desirable.**
- Улучшения;
- Рефакторинг;
- Мелкие фичи;

**Примеры:**
- «Добавить caching»;
- «Улучшить error messages»;
- «Обновить README»;

**P3: Nice-to-have.**
- Идеи;
- Косметика;
- Может быть когда-нибудь;

**Примеры:**
- «Поддержка dark mode»;
- «Попробовать Elixir»;
- «Рефакторинг класса с 800 строками»;

**Правило:**
- **P0 — редкие.** Больше 2-3 одновременно — сломанная приоритезация
- **P1 — текущий цикл.** Что делаете прямо сейчас
- **P2 — когда есть время.** Следующий цикл
- **P3 — backlog идей.** Может быть никогда
### 16.16 Anti-patterns

**Anti-pattern 1: планирование без цели.**
«Надо что-то делать». Без понимания зачем. Результат — хаос.

**Anti-pattern 2: слишком много задач на цикл.**
«Возьму 20 задач». Ни одну не успеете.

**Anti-pattern 3: 100% загрузка.**
Не остаётся времени на unexpected. План падает в первую неделю.

**Anti-pattern 4: нет техдолга в плане.**
Только features. Через полгода — легаси.

**Anti-pattern 5: backlog = roadmap.**
Backlog для конкретных задач. Roadmap — отдельно.

**Anti-pattern 6: не удалять завершённые.**
Backlog растёт, планирование замедляется.

**Anti-pattern 7: приоритеты не пересматриваются.**
P1 висит полгода. Либо это не P1, либо не выполняется.

**Anti-pattern 8: планирование раз в год.**
Backlog превращается в свалку. Каждый раз — уборка.

**Anti-pattern 9: планирование ежедневно.**
Overkill для задач. Вы и так знаете, что делать.

**Anti-pattern 10: нет баланса.**
100% features или 100% tech debt. Оба плохи.
### 16.17 Связь с другими файлами

```text
              _backlog.md
                  │
     ┌────────────┼────────────┐
     ▼            ▼            ▼
analysis/      _decisions.md  WORK_LOG.md
(findings      (Follow-up    (что делали
 → задачи)      → задачи)     за неделю)
     │
     └─→ GitHub Issues
              │
              ▼
         issue/PROJECT_SUMMARY.md
              │
              ▼
         Issue lifecycle
```

**`_backlog.md` — центральный узел.** Он собирает:
- **Из `analysis/`** — findings
- **Из `_decisions.md`** — follow-up
- **Из WORK_LOG** — «Next» из сессий
- **Из Ideas** — сырые идеи

**Из backlog → GitHub Issues → работа.**
### 16.18 Планирование для solo vs team

**Solo:**
- Недельное планирование — оптимально
- Quarterly — если есть стратегические цели
- Backlog — личный
- Приоритеты — субъективны

**Team:**
- Спринтовое планирование
- Daily standup
- Backlog — общий
- Приоритеты — consensus

**Компромисс:** если team маленькая (2-3 человека) — недельное планирование + месячное ревью.

### 16.19 Упражнение

**Часть 1: ревизия backlog.**

Откройте `_backlog.md`. Ответьте:
1. **Сколько items?** 5–15 — норма. 50+ — пора чистить
2. **Сколько P0?** Больше 3 — пересмотрите
3. **Есть ли items старше 3 месяцев?** Что с ними?
4. **Есть ли items без Next action?** Добавьте или удалите
5. **Сколько в Ideas?** Разберите

**Часть 2: планирование цикла.**

Сделайте полное планирование на следующую неделю:

1. **Прочитайте** backlog;
2. **Разберите** Ideas;
3. **Пересмотрите** приоритеты;
4. **Оцените** (S/M/L);
5. **Выберите** 5–10 задач;
6. **Создайте** issues;
7. **Обновите** backlog;

Замерьте время. Оптимум: 15–30 минут для недельного цикла.

**Часть 3: ретроспектива.**

Через неделю — сравните план и факт:
1. **Сколько задач сделали?** Из плана?
2. **Сколько unexpected было?**
3. **План был реалистичный?**
4. **Что менять** в следующий раз?
### 16.20 Метрики

| Метрика                       | Что показывает |
| ----------------------------- | -------------- |
| **Backlog size**              | Размер         |
| **Items closed per week**     | Скорость       |
| **Items added per week**      | Прирост        |
| **Avg age of items**          | Свежесть       |
| **P0 count**                  | Критичность    |
| **Feature / Tech debt ratio** | Баланс         |

**Что полезно знать:**
- **Backlog растёт** — вы добавляете больше, чем делаете. Либо замедлитесь, либо чистите.
- **Avg age > 3 месяцев** — items не актуальны. Удалить.
- **Feature / Tech debt > 9:1** — техдолг накапливается.
- **Feature / Tech debt < 1:1** — проект стагнирует.

**Не превращайте в KPI.** Это для понимания.
### 16.21 Итог Part III

Part III — Workflows — завершён. Мы разобрали шесть workflow'ов:
1. **Issue lifecycle** — от открытия до merge
2. **Session lifecycle** — ритуал начала и конца сессии
3. **When CI fails** — диагностика и фикс
4. **Incident in prod** — реакция на аварию
5. **Deep analysis** — исследование проекта
6. **Planning** — выбор задач

**Общий принцип:** каждый workflow связан с файлами bundle. Issue → templates, WORK_LOG, decisions. Session → WORK_LOG. CI → `_ci.md`. Incident → runbooks. Analysis → `analysis/`. Planning → `_backlog.md`.

**Всё сходится в bundle.** Workflows оживляют файлы, файлы поддерживают workflows.

### 16.22 Что дальше

В **[Part IV— Operations](../textbook_ru/guide_ru_part_IV.md)** — эксплуатация bundle:
- Установка через `init-opencode`.
- Обновление.
- Расширение bundle (новые файлы, типы).
- Anti-patterns (общие).
- Philosophy.

Это мета-уровень: не «как работать с проектом», а «как работать с самим bundle».