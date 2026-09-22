---
type: meta
title: "Bundle Meta"
description: "What this .opencode/ bundle is and how to work with it"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [meta, okf]
---

# Bundle Meta

This `.opencode/` directory is an [OKF](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
bundle with extensions. It is **local only** — never committed to the
project repository

## What's here

- **Project context** — [AGENTS.md](AGENTS.md), entry point
- **Reference files** — `_*.md`, lazy-loaded
- **Dynamic artifacts** — `issue/`, `playbook/`, `pr/`, `analysis/`,
  `runbooks/`, `archive/`
- **Work log** — [WORK_LOG.md](WORK_LOG.md), local only

Full index: [index.md](index.md)

## Template version

- **Version:** <vX.Y.Z>
- **Installed:** <YYYY-MM-DD>
- **Last updated:** <YYYY-MM-DD>
- **Source:** <repo-url>

Version details also in [`.template-version`](.template-version) (machine-readable)

## OKF base + extensions

This bundle follows OKF v0.1 with the following extensions:

| Extension | What we added |
|-----------|---------------|
| `AGENTS.md` as entry point | OKF uses `index.md`; we use `AGENTS.md` and provide `index.md` as a pointer |
| `_*.md` naming | Prefix `_` marks reference files (partials) |
| `WORK_LOG.md` | Local work log, analogous to OKF's `log.md` |
| Subdirectories | `issue/`, `playbook/`, `pr/`, `analysis/`, `runbooks/`, `archive/` for dynamic content |
| Custom `type` values | `project-context`, `setup`, `worklog`, `backlog`, `decision-log`, `testing`, `api`, `release`, `performance`, etc. |

Local extract of the spec: [SPEC_REFERENCE.md](SPEC_REFERENCE.md).

## How to update

If this bundle was installed via `init-opencode`:

```bash
# From the project root
init-opencode --update <project-dir>
```

- **Never overwritten:** `WORK_LOG.md`, `_decisions.md`, `_backlog.md`,
  `_concepts.md`, `analysis/*`, `runbooks/*`, `issue/*`, `playbook/*`,
  `pr/*`, `archive/*`
- **Always overwritten:** `_codestyle.md`, `_ci.md`, `_commands.md`,
  `_files.md`, `_glossary.md`, `_security.md`, `_troubleshooting.md`,
  `_templates.md`, `_testing.md`, `_api.md`, `_release.md`, `_performance.md`,
  `AGENTS.md`, `index.md`, `log.md`, `_meta.md`
- **Diff preview:**
  ```bash
  init-opencode --diff <project-dir>
  ```

## Rules

- **Never commit** this directory. See [`.gitignore`](.gitignore) and
  project's `.git/info/exclude`
- **Never put secrets** in any file here. See [_security.md](_security.md)
- **Update `generated.at`** in a file's frontmatter whenever you edit it