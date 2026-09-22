# OpenCode Project Template

A local knowledge bundle for working with AI agents through
[OpenCode](https://opencode.ai). Lazy-loaded markdown files, OKF-inspired,
never committed to your project.

## What's inside

- **`AGENTS.md`** — entry point, minimal project context + workflow
- **`_*.md`** — topical reference files (architecture, setup, CI, codestyle, security, testing, API, performance, release, and more) — loaded on demand
- **Dynamic directories** — `issue/`, `playbook/`, `pr/`, `analysis/`, `runbooks/`, `archive/`
- **OKF v0.2** — YAML frontmatter with `type`, `generated`, `status`, `sources` for trust signals

Result: instead of one 500-line `AGENTS.md`, you get a focused ~60-line core plus 25+ topical files the agent reads only when needed

## Installation
Clone the template repository and link the installer:
```bash
git clone <repo-url> ~/Projects/opencode-templates
ln -s ~/Projects/opencode-templates/bin/init-opencode \
      ~/.local/bin/init-opencode
```
Ensure `~/.local/bin` is in `$PATH`. Then verify:
```bash
init-opencode --help
```
If you prefer a copy over a symlink:
```bash
cp ~/Projects/opencode-templates/bin/init-opencode ~/.local/bin/
chmod +x ~/.local/bin/init-opencode
```

## Quick start

```bash
# Install into a new project
init-opencode ~/Projects/my-app

# Install into an existing project
init-opencode --analyze ~/Projects/existing-repo

# Update an existing bundle
init-opencode --update ~/Projects/my-app
```

Then open OpenCode in the project and say:

> "Analyze the repository and fill in the template"

## Documentation

| Language | File |
|----------|------|
| English | [README_en.md](README_en.md) |
| Русский | [README_ru.md](README_ru.md) |

Full guides cover: bundle structure, OKF conformance, `type` dictionary,
workflow scenarios, FAQ, and the philosophy behind the template

## Structure

```
opencode-templates/
├── README.md              ← this file (summary)
├── README_en.md           ← full guide (EN)
├── README_ru.md           ← full guide (RU)
├── bin/
│   └── init-opencode      ← installer script
└── template/              ← what gets copied into projects
    ├── AGENTS.md          ← entry point, project context + workflow
    ├── SPEC_REFERENCE.md  ← OKF v0.2 spec extract with trust signals
    ├── index.md           ← bundle table of contents (no frontmatter per OKF v0.2)
    ├── log.md             ← change history pointer
    ├── WORK_LOG.md        ← chronological work sessions
    ├── _concepts.md       ← architecture, patterns, data flow
    ├── _setup.md          ← get the project running locally
    ├── _env.md            ← deployment environments map
    ├── _codestyle.md      ← linting, conventions, SPDX headers
    ├── _commands.md       ← build/test/run command reference
    ├── _files.md          ← key files navigation guide
    ├── _glossary.md       ← domain-specific terms
    ├── _security.md       ← secrets handling, vulnerability reporting (with sources)
    ├── _troubleshooting.md← local (non-CI) problem diagnostics
    ├── _decisions.md      ← ADR log for significant choices
    ├── _backlog.md        ← future work, tech debt, ideas
    ├── _worklog.md        ← work log entry format rules
    ├── _templates.md      ← issue/pr/playbook templates (v0.2 compatible)
    ├── _ci.md             ← CI workflows and failure patterns
    ├── _meta.md           ← bundle organization and template version
    ├── _testing.md        ← testing strategy, fixtures, mocking (with sources)
    ├── _api.md            ← endpoints, auth, CLI commands, versioning
    ├── _release.md        ← versioning, changelog, rollback procedures
    ├── _performance.md    ← profiling, benchmarks, SLA/SLO targets (with sources)
    ├── analysis/
    │   ├── index.md       ← analysis findings index (draft)
    │   └── _finding.md    ← individual finding template
    └── runbooks/
        ├── index.md       ← incident response index (draft)
        └── _runbook.md    ← single runbook template
```

Subdirectories contain dynamic artifacts: `issue/`, `playbook/`, `pr/`, `archive/` — generated during issue resolution

## Key principles

- **Lazy loading** `AGENTS.md` is a table of contents, not an
  encyclopedia — agent reads topical files only when needed
- **Local only** `.opencode/` is never committed to your project
- **OKF v0.2 compliant** Markdown + YAML frontmatter with trust signals (`generated`, `status`, `sources`)
- **Readable by everyone** If you can `cat` a file, you can read OKF; if you can `git clone`, you can distribute it
- **No tooling required** No build step, no central registry, consumers may add validators but they're optional

## License

MIT See [LICENSE](LICENSE)