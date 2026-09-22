# OpenCode Project Template

A complete guide to `.opencode/` — a local knowledge bundle for working
with an AI agent through [OpenCode](https://opencode.ai).

---

## Table of contents

- [What it is and why](#what-it-is-and-why)
- [Key concepts](#key-concepts)
- [How it works](#how-it-works)
- [Bundle structure](#bundle-structure)
- [Template files](#template-files)
- [Dynamic artifacts](#dynamic-artifacts)
- [How files relate](#how-files-relate)
- [OKF and this template](#okf-and-this-template)
- [`type` dictionary](#type-dictionary)
- [Usage](#usage)
- [Workflow scenarios](#workflow-scenarios)
- [init-opencode](#init-opencode)
- [Caveats](#caveats)
- [FAQ](#faq)
- [Philosophy](#philosophy)

---

## What it is and why

When an AI agent works on a project through OpenCode, it reads
`AGENTS.md`. The problem: developers dump **everything** there —
architecture, code style, CI tables, file lists, decision history. The
file grows to 500–600 lines, of which 50 are needed at any given moment.
The rest is noise — it burns tokens and distracts the agent.

This template solves the problem through **lazy loading**:

- `AGENTS.md` — only the core: what the project is, how to build it,
  key components, where to go next.
- Everything detailed — in topical files (`_*.md`) that are read only
  when the task requires it.
- Dynamic artifacts (issue, playbook, PR, analysis, runbooks) — in
  separate directories.

**Key principle:** `AGENTS.md` is not an encyclopedia, it's a table of
contents. The encyclopedia lives in `_*.md` files.

---

## Key concepts

### Bundle

**Bundle** is an OKF term. It's a **directory of concept documents**
organized by specific rules: frontmatter, cross-linking, reserved names.

In this template, the bundle is `.opencode/` at the project root.

```
my-project/
├── .opencode/          ← ← ← the bundle
│   ├── AGENTS.md
│   ├── _concepts.md
│   └── ...
├── src/
└── README.md
```

The bundle is **local** — never committed to the project repository.
It's your personal knowledge base and working tool.

### Lazy loading

`_*.md` files are **not loaded automatically**. The agent reads them
only when it decides the task matches a description in `AGENTS.md`.

Example:
- You say: "CI is red, figure it out."
- The agent sees in `AGENTS.md`: `_ci.md — when CI fails`.
- It reads `_ci.md` and diagnoses.

This saves context: the agent doesn't spend tokens on files that aren't
relevant right now.

### OKF

**Open Knowledge Format** — an open format from Google Cloud for
representing knowledge as markdown files with YAML frontmatter.

Every file is a **concept document**. It has two parts:

1. **Frontmatter** — metadata (required field `type`).
2. **Body** — markdown content.

The bundle follows OKF v0.2 **with extensions** (see
[OKF and this template](#okf-and-this-template)).

### Lazy vs eager

| Eager (everything at once) | Lazy (on demand) |
|----------------------------|------------------|
| One large AGENTS.md | Core + topical files |
| 500+ lines always in context | ~60 lines + files on demand |
| Agent drowns in noise | Agent stays focused |

---

## How it works

### AGENTS.md hierarchy

At session start, OpenCode collects `AGENTS.md` from every level up the
directory tree and merges them into a single context:

```
~/.config/opencode/AGENTS.md                  ← global rules
~/Projects/<org>/.opencode/AGENTS.md          ← organization rules
~/Projects/<org>/<repo>/.opencode/AGENTS.md   ← repository rules
```

Nothing needs to be configured manually — OpenCode does it on its own.

### How the bundle is read

1. `AGENTS.md` is opened — the entry point.
2. The agent gets minimal context: what the project is, stack, build.
3. Using the reference files table, the agent decides what to read next.
4. Reads the relevant `_*.md` files.
5. Works.

### Your role

You **don't manage file loading manually**. You state a task in natural
language. The agent decides which files it needs.

Examples:

| What you say | What the agent reads |
|--------------|----------------------|
| "Add a new component" | `_concepts.md`, `_codestyle.md`, `_files.md` |
| "CI is red" | `_ci.md`, `_troubleshooting.md` |
| "Working on issue #123" | `_templates.md`, `_concepts.md` |
| "I need a deep analysis" | `_concepts.md`, creates `analysis/*` |

---

## Bundle structure

Full structure of `.opencode/`:

```
.opencode/
├── .gitignore                    ← commit protection
├── .template-version             ← template version (machine-readable)
│
├── index.md                      ← OKF entry point
├── log.md                        ← OKF log pointer
├── AGENTS.md                     ← main file
├── SPEC_REFERENCE.md             ← extract of OKF v0.2
│
├── _concepts.md                  ← architecture
├── _setup.md                     ← local setup
├── _env.md                       ← environment map
├── _codestyle.md                 ← style and conventions
├── _commands.md                  ← command cheat sheet
├── _files.md                     ← file map
├── _glossary.md                  ← domain terms
├── _security.md                  ← secrets handling
├── _troubleshooting.md           ← local problems
├── _decisions.md                 ← ADR log
├── _backlog.md                   ← future work
├── _worklog.md                   ← WORK_LOG template
├── _templates.md                 ← issue/PR templates
├── _testing.md                   ← testing practice (fixtures, mocking, coverage)
├── _api.md                       ← API reference (endpoints, methods)
├── _release.md                   ← release process and rollback
├── _performance.md               ← benchmarks, profiling, SLO targets
├── _ci.md                        ← CI workflows and failures
├── _meta.md                      ← bundle meta
│
├── WORK_LOG.md                   ← created on first session
│
├── issue/                        ← active PROJECT_SUMMARY
├── playbook/                     ← active PLAYBOOK
├── pr/                           ← PR description drafts
├── analysis/                     ← analysis findings
│   ├── index.md
│   └── _finding.md
├── runbooks/                     ← incident procedures
│   ├── index.md
│   └── _runbook.md
└── archive/                      ← completed issue/playbook/pr
```

---

## Template files

### Core

| File | Role | Length |
|------|------|--------|
| `AGENTS.md` | Entry point. Project context + navigation | 60–90 lines |
| `index.md` | OKF bundle index | ~35 lines |
| `log.md` | Pointer to chronological logs | ~20 lines |

### Onboarding — understanding the project

| File | When to read |
|------|--------------|
| `_setup.md` | First run from scratch |
| `_concepts.md` | Architecture, patterns, data flow |
| `_glossary.md` | Unfamiliar domain term |

### Daily work — everyday tasks

| File | When to read |
|------|--------------|
| `_templates.md` | New issue — SUMMARY, PLAYBOOK, PR |
| `_worklog.md` | Starting/continuing a session |
| `_testing.md` | Testing practice — fixtures, mocking, golden files, coverage |
| `_backlog.md` | Planning, ideas, tech debt |
| `_decisions.md` | "Why is it like this" — ADR |
| `_codestyle.md` | Writing code — SPDX, lint, conventions |
| `_commands.md` | Need a command — build/test/run |

### When things break — diagnostics

| File | When to read |
|------|--------------|
| `_ci.md` | CI failed |
| `_troubleshooting.md` | Local environment broken |
| `runbooks/` | Incident in prod |

### Navigation & safety — navigation and safety

| File | When to read |
|------|--------------|
| `_files.md` | Looking for where things live |
| `_env.md` | Environment map — dev, staging, prod |
| `_security.md` | Secrets, vulnerabilities |
| `analysis/` | Deep analysis findings |
| `_meta.md` | How the bundle itself is organized |
| `_api.md` | Public API — methods, commands, endpoints (optional) |
| `_release.md` | Release process — versioning, publishing, rollback (optional) |
| `_performance.md` | Performance tracking — benchmarks, profiling, SLO targets (optional) |

### Utility

| File | Role |
|------|------|
| `SPEC_REFERENCE.md` | Extract of OKF v0.2 |
| `.gitignore` | Commit protection |
| `.template-version` | Template version |

---

## Dynamic artifacts

These directories **fill up as you work**. Their contents are unique
to each project and each issue.

### `issue/`

Active `PROJECT_SUMMARY_<N>.md` — issue snapshots.

Created at the start of work on an issue. After the PR is merged —
moved to `archive/`.

### `playbook/`

Active `PLAYBOOK_<N>.md` — issue solution strategy.

Created when the solution isn't obvious. May be absent for simple tasks.

### `pr/`

`PR_<N>.md` drafts — pull request descriptions.

Created before opening the PR. After merge — moved to `archive/`.

### `analysis/`

Deep analysis findings: audit, performance, tech debt.

- `index.md` — summary table.
- `_finding.md` — single finding template.
- `F-001-xxx.md`, `F-002-xxx.md` — specific findings.

Findings are **not tasks** — they're reports on state. To act, create a
task in `_backlog.md`.

### `runbooks/`

Incident response procedures for prod.

- `index.md` — index of all runbooks.
- `_runbook.md` — template.
- `db-failover.md`, `rollback-release.md` — specific procedures.

Read **under pressure**. Format: command + Expected + "if it doesn't
work".

### `archive/`

Completed issue/playbook/pr. Moved here after PR merge.

Can be organized by date or by issue number.

---

## How files relate

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

Key relationships between files:

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

**Rule "link, don't duplicate":** if something appears in two files,
link instead of copy.

---

## OKF and this template

### What OKF is

**Open Knowledge Format** — an open format from Google Cloud for
representing knowledge. Minimalism: no central schema registry, no
mandatory tooling.

Full spec:
[SPEC.md](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md).
Local extract: [`SPEC_REFERENCE.md`](SPEC_REFERENCE.md).

### What we borrowed from OKF

| OKF concept | In this template |
|-------------|------------------|
| Bundle | The whole `.opencode/` directory |
| Concept document | Every `_*.md` file |
| YAML frontmatter | The `---` block with `type`, `title`, `description` |
| `type` (REQUIRED) | `project-context`, `architecture`, `ci`, ... |
| Cross-linking | Relative markdown links between files |
| Citations | `## References` / `## Citations` |
| `index.md` | Our `index.md` — OKF entry point |
| `log.md` | Our `log.md` — pointer to logs |

### What we added on top of OKF

| Extension | Why |
|-----------|-----|
| `AGENTS.md` as entry point | OpenCode reads exactly this at startup |
| `_*.md` prefix | Visually marks reference files |
| Lazy loading | Saves agent context |
| `WORK_LOG.md` | Local chronology (instead of OKF `log.md`) |
| Directories `issue/`, `playbook/`, `pr/` | Dynamic workflow artifacts |
| `analysis/` | Structured analysis output |
| `runbooks/` | Incident procedures |
| Extended `type` dictionary | Types specific to our tasks |

### OKF tolerance

OKF requires consumers **not to reject** a bundle because of:

- Missing optional frontmatter fields.
- Unknown `type` values.
- Unknown additional frontmatter keys.
- Broken cross-links.
- Missing `index.md`.

We follow this principle: the bundle stays useful as it grows, gets
refactored, and is partially generated by agents.

---

## `type` dictionary

OKF does not register `type` centrally, but for consistency within a
single project it's useful to stick to a fixed set.

| `type` | File |
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
| `testing` | `_testing.md` |
| `api` | `_api.md` |
| `release` | `_release.md` |
| `performance` | `_performance.md` |
| `analysis-index` | `analysis/index.md` |
| `finding` | `analysis/F-*.md` |
| `runbook-index` | `runbooks/index.md` |
| `runbook` | `runbooks/*.md` |
| `project-summary` | `issue/PROJECT_SUMMARY_<N>.md` |
| `playbook` | `playbook/PLAYBOOK_<N>.md` |
| `pr` | `pr/PR_<N>.md` |

Consumers (including the agent) must **tolerate** unknown `type` values —
this is an OKF requirement. But producers (us) try not to multiply new
ones without need.

---

## Usage

### Quick start

```bash
# Install template into a new project
init-opencode ~/Projects/my-app

# Install into an existing project
init-opencode --analyze ~/Projects/existing-repo
```

### What to do after initialization

**New project:** tell the agent — "starting a new project, help me fill
in the template". The agent will ask questions and fill files.

**Existing project:** tell it — "analyze the repository and fill in the
template". The agent will study the code, CI, and structure and fill
all templates.

### Updating

```bash
# Preview what will change
init-opencode --diff ~/Projects/my-app

# Apply the update
init-opencode --update ~/Projects/my-app
```

**Never overwritten:** `WORK_LOG.md`, `_decisions.md`, `_backlog.md`,
`_concepts.md`, `_setup.md`, `analysis/*`, `runbooks/*`, `issue/*`,
`playbook/*`, `pr/*`, `archive/*`.

**Always overwritten:** `_codestyle.md`, `_ci.md`, `_commands.md`,
`_files.md`, `_glossary.md`, `_security.md`, `_troubleshooting.md`,
`_templates.md`, `AGENTS.md`, `index.md`, `log.md`, `_meta.md`.

---

## Workflow scenarios

### Scenario 1: new project

```
1. init-opencode ~/Projects/my-app
2. Open OpenCode in my-app/
3. Say: "starting a new project, help me fill in the template"
4. Agent asks questions, fills files
5. You refine
6. Agent commits the result
```

### Scenario 2: existing project

```
1. init-opencode --analyze ~/Projects/existing-repo
2. Open OpenCode
3. Say: "analyze the repository and fill in the template"
4. Agent studies structure, code, CI
5. Fills all files
6. Optionally: creates analysis/ with findings
```

### Scenario 3: working on an issue

```
1. Say: "working on issue #123"
2. Agent creates issue/PROJECT_SUMMARY_123.md and playbook/PLAYBOOK_123.md
3. Reads _concepts.md (understands the architecture)
4. You discuss the approach
5. Agent writes code, following _codestyle.md
6. Runs commands from _commands.md
7. Creates pr/PR_123.md
8. After merge — moves everything to archive/
```

### Scenario 4: CI failed

```
1. Say: "CI is red, figure it out"
2. Agent reads _ci.md
3. Identifies the failing workflow from logs
4. Finds the cause in the Common failures table
5. You discuss the fix
6. If the problem is new — adds a row to _ci.md
```

### Scenario 5: local environment broken

```
1. Say: "tests don't run, error X"
2. Agent reads _troubleshooting.md
3. Finds the symptom via Quick index
4. Applies the fix
5. If no solution — searches, solves, adds an entry
```

### Scenario 6: prod incident

```
1. Alert fires
2. Open runbooks/index.md
3. Find the matching runbook
4. Follow the steps
5. If the runbook didn't help — escalate
6. Post-incident: update runbook, _decisions.md, _backlog.md
```

### Scenario 7: deep analysis

```
1. Say: "I need a security audit"
2. Agent reads _concepts.md, _security.md
3. Analyzes the code
4. Creates analysis/F-001-xxx.md, F-002-xxx.md
5. Updates analysis/index.md
6. You decide what to fix
7. Move items to _backlog.md
```

### Scenario 8: planning

```
1. Say: "what should we do next?"
2. Agent reads _backlog.md, WORK_LOG.md
3. Shows P0/P1 priorities
4. You discuss
5. Take a task — Scenario 3 begins
```

---

## init-opencode

The script `~/.local/bin/init-opencode` copies the template into a
target project.

### Usage

```bash
# New project
init-opencode ~/Projects/my-app

# Existing project
init-opencode --analyze ~/Projects/existing-repo

# Update
init-opencode --update ~/Projects/my-app

# Preview changes
init-opencode --diff ~/Projects/my-app

# Help
init-opencode --help
```

### What it does

**Install:**
1. Creates `.opencode/` in the target directory.
2. If `.opencode/` already exists — backs it up to `.opencode.bak.<timestamp>`.
3. Copies the `template/` contents.
4. Creates empty directories (`issue/`, `playbook/`, `pr/`, `archive/`).
5. Creates `.template-version`.
6. Prints the next steps.

**Update:**
1. Reads `.template-version`.
2. Compares with the current template version.
3. Updates "always overwritten" files.
4. Leaves "never overwritten" files untouched.
5. Updates `.template-version`.

### Environment

| Variable | Default | Purpose |
|----------|---------|---------|
| `OPENCODE_TEMPLATE_REPO` | `~/Projects/opencode-templates` | Path to the template repository |

Override if your template clone lives elsewhere:

```bash
OPENCODE_TEMPLATE_REPO=~/work/opencode-templates \
  init-opencode ~/Projects/my-app
```

### Version file

The template repository contains a `VERSION` file at its root (e.g.,
`v0.1.0`). `init-opencode` reads it and writes `.opencode/.template-version`
with metadata:

```
version: v0.1.0
installed: 2026-09-21
source: /home/user/Projects/opencode-templates
```

`--diff` and `--update` use this to detect drift.

### What gets overwritten

`--update` only touches "always overwritten" files. The full lists:

**Never overwritten** (user-owned):

- `WORK_LOG.md`
- `_concepts.md`
- `_setup.md`
- `_decisions.md`
- `_backlog.md`
- `_meta.md`
- Everything in `analysis/`, `runbooks/`, `issue/`, `playbook/`,
  `pr/`, `archive/`

**Always overwritten** (template-owned):

- `AGENTS.md`, `index.md`, `log.md`, `SPEC_REFERENCE.md`
- `_codestyle.md`, `_ci.md`, `_commands.md`, `_files.md`,
  `_glossary.md`, `_security.md`, `_troubleshooting.md`,
  `_templates.md`, `_env.md`, `_worklog.md`

If you customized an "always overwritten" file, copy it to a new name
(e.g., `_codestyle.local.md`) or move it to the "never" list in the
script.

### Verify installation

```bash
# Check the script is in PATH
which init-opencode

# Dry run in a test directory
mkdir -p /tmp/test-project
init-opencode /tmp/test-project
ls -la /tmp/test-project/.opencode/

# Update should be a no-op right after install
init-opencode --update /tmp/test-project
# → nothing to update
```
### Installing the script

The script should already be in `~/.local/bin/`. If not — download it
from the template repository:

```bash
curl -fsSL <repo-url>/raw/main/bin/init-opencode \
  -o ~/.local/bin/init-opencode
chmod +x ~/.local/bin/init-opencode
```

---

## Caveats

### `.opencode/` is never committed

- Added to `.git/info/exclude` (locally for your clone) or to
  `.gitignore` (for everyone).
- Inside `.opencode/` itself there's a `.gitignore` — a second line of
  defense.
- `.git/info/exclude` is recommended — it doesn't pollute the
  repository.

### `git clean -dfX` deletes `.opencode/`

Always use:

```bash
git clean -dfX -e .opencode/
```

Or add `.opencode/` to `.gitignore`.

### `make clean` may delete `.opencode/`

Before `make clean`, back it up or add `-e .opencode/`.

### Topical files are not read automatically

The agent reads them only when a task matches a description in
`AGENTS.md`. If you need architecture — say "tell me about the
architecture".

### `WORK_LOG.md` is local

Not synchronized. If you need to share — copy it into an issue manually.

### `_security.md` is not for secrets

It's **rules** for handling secrets, not storage. Never put real keys
there.

### Runbooks are not for local problems

Runbooks are for prod. Local problems — in `_troubleshooting.md`.

### Analysis is not a task list

Findings describe state. To act, create a task in `_backlog.md`.

---

## FAQ

### Q: Where should I store the template repository clone?

Recommended: `~/Projects/<org>/opencode-templates/`. One permanent
clone — you push from it, and `init-opencode` pulls files from it.

Don't confuse it with `~/.config/opencode/` — that's where the global
`AGENTS.md` lives.

### Q: How do I update the template in a project?

```bash
init-opencode --diff ~/Projects/my-app   # preview
init-opencode --update ~/Projects/my-app # apply
```

Files with your data (`WORK_LOG.md`, `_decisions.md`, `_backlog.md`,
`_concepts.md`, `_setup.md`) are not overwritten.

### Q: What is a bundle?

An OKF term. The whole `.opencode/` directory. Not a file, not a
repository — specifically a directory of concept documents.

### Q: Why `index.md` and `log.md` if we have `AGENTS.md` and `WORK_LOG.md`?

Formal OKF conformance. `index.md` and `log.md` are reserved names. In
our template they act as **pointers** to our primary files (`AGENTS.md`,
`WORK_LOG.md`).

### Q: Why doesn't AGENTS.md store code style?

Because code style is only needed when writing code. Otherwise it's
wasted tokens in the context. When the task reaches code, the agent
reads `_codestyle.md` by itself.

### Q: What if I want the agent to always know X?

Put X in `AGENTS.md`. It's the only file always in context. But
remember: the larger `AGENTS.md`, the less attention to details.

### Q: How often should I update topical files?

- `_concepts.md` — on architectural changes.
- `_ci.md` — when `.github/workflows/` changes.
- `_troubleshooting.md` — after every solved problem (>10 min).
- `_backlog.md` — when tasks appear or complete.
- `_decisions.md` — on significant decisions.
- `_meta.md` — when the template is updated.
- `_env.md` — when environments change.

### Q: I have a monorepo — what do I do?

For each microservice/package — its own `.opencode/AGENTS.md`. Shared
rules — in the parent `.opencode/AGENTS.md` at the monorepo root.

### Q: Does `_decisions.md` grow forever?

Yes, but that's fine. Accepted ADRs are **not edited** (except
`Status`). Old entries are history. If you have 100+ ADRs — consider
pruning (mark as `deprecated`).

### Q: Why `_backlog.md` if there are GitHub Issues?

The backlog is a **draft**, Issues are **confirmed tasks**. The backlog
is cheap to write, Issues require formulation, labels, assignee.
Correct flow: idea → backlog → decision to act → Issue.

### Q: How do I decide what goes in `_decisions.md` vs `_worklog.md`?

- **`_decisions.md`** — significant decisions (architecture, API, process).
- **`_worklog.md`** — every session, including small decisions.

If a decision will affect others in six months — ADR. If it's a "local
decision within a session" — WORK_LOG.

### Q: Do I need `analysis/` for a small project?

Probably not. `analysis/` is for when you run audits, hunt performance
problems, investigate tech debt. For a typical solo project it may be
overkill.

### Q: How do I handle secrets in `runbooks/`?

Don't store real values. Use placeholders: `<DB_PASSWORD>`, "get from
Vault at path ...". See `_security.md`.

---

## Philosophy

### Speed over quality

The template follows the philosophy of
[Don't Aim for Quality, Aim for Speed](https://www.yegor256.com/2018/03/06/speed-vs-quality.html):

**Your job is to close tasks fast.** Quality is the project's
responsibility (CI, review, static analysis), not yours.

In practice:
- **Cut corners.** Write working code; reviewers will catch problems.
- **Small PRs.** Faster to write and review.
- **Don't study the whole codebase.** Change only what the task needs.
- **Don't be afraid to break.** CI will catch regressions.

### Why all this

The template isn't about tokens and savings (though that matters too).
It's about **clarity**.

When the agent sees a clean, structured `AGENTS.md`, it:
- Quickly understands the project.
- Knows exactly where to find details.
- Doesn't get distracted by noise.
- Makes better decisions.

A bad `AGENTS.md` is like a cluttered desk. A good one is like an
organizer with labeled drawers.

### OKF as foundation

We build on OKF because:
- **Simplicity.** Markdown + frontmatter. No binary formats.
- **Portability.** Works with `cat`, `git clone`, any editor.
- **Standard.** Google Cloud, open format, stable spec.
- **Extensibility.** OKF explicitly permits extra fields and extensions.

Our template is an OKF-inspired bundle with extensions for AI agent
tasks. Not strictly conformant, but following the spirit of the spec.

---

*Full OKF v0.2 guide: [SPEC.md](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md).*
*Local extract: [`SPEC_REFERENCE.md`](SPEC_REFERENCE.md).*