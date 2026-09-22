---
type: setup
title: "Local Setup"
description: "How to get the project running locally from scratch"
timestamp: <YYYY-MM-DD>
tags: [setup, onboarding, dev-env]
---
# Local Setup

Scenario: fresh clone → working dev environment

For non-local environments (staging, prod, on-prem), see [_env.md](_env.md)
For commands after setup, see [_commands.md](_commands.md)
## Prerequisites
| Tool | Version | Install |
|------|---------|---------|
| <tool> | <version> | <install command or link> |
## 1. Clone and install dependencies
```bash
git clone <repo-url>
cd <repo>
# bundle install          ← Ruby
# npm ci                  ← Node
# other package manager install
```
## 2. Configuration
Local secrets live in `.env` — see [_security.md](_security.md) for rules.
| Env var | Purpose | Default | Required |
|------|------|------|------|
|`<VAR>`|<what it does>|<default or —>|yes / no|
```bash
cp .env.example .env
# edit .env with local values
```
## 3. Database
```bash
<db create>
<db migrate>
<db seed>
```
## 4. Run
```bash
<run command>
```
Open <http://localhost:PORT>

## 5. Verify
```bash
<test command>
<lint command>
```
If something fails, see [_troubleshooting.md](_troubleshooting.md)
For details, see [_commands.md](_commands.md)
If veryfy fails, see [_troubleshooting.md](_troubleshooting.md)
## First-day reading order
1. [AGENTS.md](AGENTS.md) — what this project is
2. [_concepts.md](_concepts.md) — how it's built
3. [_codestyle.md](_codestyle.md) — how to write code
4. [_commands.md](_commands.md) — common commands
5. [_files.md](_files.md) — where things live

## Citations
- [1] [Official install guide](<url>)
- [2] [Team wiki: dev environment](<url>)