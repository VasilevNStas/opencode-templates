# OpenCode Project Template

A local knowledge bundle for working with AI agents through
[OpenCode](https://opencode.ai). Lazy-loaded markdown files, OKF-inspired,
never committed to your project.

## What's inside

- **`AGENTS.md`** — entry point, minimal project context
- **`_*.md`** — topical reference files (architecture, setup, CI,
  codestyle, security, and more) — loaded on demand
- **Dynamic directories** — `issue/`, `playbook/`, `pr/`, `analysis/`,
  `runbooks/`, `archive/`
- **OKF v0.1** — every file has YAML frontmatter with a required `type`

Result: instead of one 500-line `AGENTS.md`, you get a focused ~60-line
core plus 15+ topical files the agent reads only when needed

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
├── README.md              ← this file
├── README_en.md           ← full guide (EN)
├── README_ru.md           ← full guide (RU)
├── bin/
│   └── init-opencode      ← installer script
└── template/              ← what gets copied into projects
    ├── AGENTS.md
    ├── SPEC_REFERENCE.md
    ├── index.md
    ├── log.md
    ├── _concepts.md
    ├── _setup.md
    ├── _env.md
    ├── _codestyle.md
    ├── _commands.md
    ├── _files.md
    ├── _glossary.md
    ├── _security.md
    ├── _troubleshooting.md
    ├── _decisions.md
    ├── _backlog.md
    ├── _worklog.md
    ├── _templates.md
    ├── _ci.md
    ├── _meta.md
    ├── analysis/
    └── runbooks/
```

Only the contents of `template/` are copied — no `.git`, no READMEs, no
installer. See [README_en.md](README_en.md#how-it-works) for details.

## Key principles

- **Lazy loading** `AGENTS.md` is a table of contents, not an
  encyclopedia
- **Local only** `.opencode/` is never committed to your project
- **OKF-inspired** Markdown + frontmatter. Readable with `cat`,
  portable via `git clone`
- **Simplicity** No build step, no tooling, no central registry

## License

MIT. See [LICENSE](LICENSE)