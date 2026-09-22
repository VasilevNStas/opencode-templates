---
type: project-context
title: "<org>/<repo>"
description: "<one-line description>"
resource: "<repo-url>"
timestamp: <YYYY-MM-DD>
tags: [language, type]
---
# Project Context: <org>/<repo>
## Overview
- **Type:** <gem / app / action / doc / cli / library / service>
- **Stack:** <languages, frameworks, databases>
- **Build:** <build command>
- **CI:** <N> CI workflows (<GitHub Actions / GitLab CI / etc.>)
- **License:** <MIT / Apache-2.0 / ...>
**First time here?** Start with [_setup.md](_setup.md) to get running locally.
### Key components
| Component | What it does |
|-----------|-------------|
| `<name>` | `<purpose>` |
---
## Issue workflow
Every issue follows the same lifecycle. Active files live in `.opencode/`.
| Stage | File | Purpose |
|-------|------|---------|
| Start | `issue/PROJECT_SUMMARY_<N>.md` | Issue overview, problem, solution, verification |
| Start | `playbook/PLAYBOOK_<N>.md` | Strategy, patterns, pitfalls, verification commands |
| During | `.opencode/WORK_LOG.md` | Session-by-session progress (see [_worklog.md](_worklog.md)) |
| During | `_decisions.md` | Non-obvious choices worth an ADR |
| End | `pr/PR_<N>.md`| PR description  Use template from [_templates.md](_templates.md) |
| End | `.opencode/archive/` | Move completed issue files here |
Templates and full workflow: [_templates.md](_templates.md).
---
### Reference files (lazy-loaded)
**Onboarding — understanding the project**
| File | When to read |
|------|-------------|
| [_setup.md](_setup.md) | Getting the project running locally from scratch |
| [_concepts.md](_concepts.md) | Architecture, key patterns, data flow |
| [_glossary.md](_glossary.md) | Unknown domain term or abbreviation |
**Daily work — everyday tasks**
| File | When to read |
|------|-------------|
| [_templates.md](_templates.md) | Working on a new issue — SUMMARY and PLAYBOOK templates |
| [_worklog.md](_worklog.md) | Starting/continuing work — session log template |
| [_backlog.md](_backlog.md) | Planning — future work, ideas, tech debt |
| [_decisions.md](_decisions.md) | Why something is the way it is — ADR log |
| [_codestyle.md](_codestyle.md) | Writing code — SPDX, lint, conventions |
| [_commands.md](_commands.md) | Need a command — build/test/run reference |
**When things break — diagnostics**
| File | When to read |
|------|-------------|
| [_ci.md](_ci.md) | CI fails — workflows and common failures |
| [_troubleshooting.md](_troubleshooting.md) | Local env broken — non-CI problems |
| [runbooks/](runbooks/index.md) | Incident response — step-by-step recovery procedures |
**Navigation & safety — reference and safety**
| File | When to read |
|------|-------------|
| [_files.md](_files.md) | Navigating codebase — key files table |
| [_env.md](_env.md) | Environment map — dev, staging, prod |
| [_security.md](_security.md) | Handling secrets, reporting vulnerabilities |
| [analysis/](analysis/index.md) | Deep analysis findings, risks, refactoring ideas |
| [_meta.md](_meta.md) | How this bundle is organized, template version |
**Optional — include only if applicable**
| File | When to read |
|------|-------------|
| [_api.md](_api.md) | Public API — methods, commands, endpoints |
| [_release.md](_release.md) | Release process — versioning, publishing, rollback |
---
_This `.opencode/` directory is an OKF bundle. Entry point: this file. Bundle index: [index.md](index.md). Change log pointer: [log.md](log.md)._
---
_All shared workflow rules (branch discipline, commits, PR requirements, mindset) are inherited from `~/.config/opencode/AGENTS.md`_
---