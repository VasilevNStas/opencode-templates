---
type: troubleshooting
title: "Troubleshooting"
description: "Local (non-CI) problems and their fixes"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [troubleshooting, dev-env]
---
# Troubleshooting
Local development problems. For CI failures, see [_ci.md](_ci.md)
For initial setup, see [_setup.md](_setup.md)
**Rule:** every time you spend more than 10 minutes solving a local problem,
add it here. Future you will thank present you.
## Quick index

| Symptom | Section |
|---------|---------|
| `Address already in use` / port conflict | [Port conflicts](#port-conflicts) |
| Database does not exist / connection error | [Database errors](#database-errors) |
| Dependency install fails (gem/npm) | [Dependency install fails](#dependency-install-fails) |
| Tests hang or timeout | [Tests hang](#tests-hang) |
| Migrations fail | [Migrations fail](#migrations-fail) |
| `<other symptom>` | [<section>](#<anchor>) |
---
## Port conflicts
**Symptom:**
```
Error: listen EADDRINUSE: address already in use :::3000
```
**Cause:** another process (or a previous run) is holding the port.
**Fix:**
```bash
# Find the process
lsof -i :3000
# Kill it
kill -9 <PID>
# Or change the port
PORT=3001 <run command>
```
## Database errors
**Symptom:**
```
PG::ConnectionBad: FATAL: database "myapp_dev" does not exist
```
**Cause:** the database hasn't been created, or the connection config
points to the wrong database.
**Fix:**
```bash
<db create>
<db migrate>
```
If it still fails, check `.env` — `DATABASE_URL` might be wrong.
---
## Dependency install fails
**Symptom:**
```
ERROR: While executing gem ... (Gem::FilePermissionError)
```
**Cause:** trying to install a gem system-wide without permission.
**Fix:**
```bash
# Ruby
bundle install --path vendor/bundle
# Or globally (user-install)
gem install --user-install <gem>
```
For Node:
```bash
npm ci --no-optional
```
---
## Tests hang
**Symptom:** tests run forever, no output.
**Cause:** often a missing mock, a real network call, or a deadlock.
**Fix:**
1. Run with a timeout: `<test command> --timeout 30`.
2. Check for real network calls — mock them.
3. Check recent changes to concurrency code.
---
## Migrations fail
**Symptom:**
```
MIGRATION_ERROR: column "X" already exists
```
**Cause:** a migration was applied out of order, or the schema is out
of sync with the migration files.
**Fix:**
```bash
<db rollback>
<db migrate>
```
If rollback fails, reset from scratch (only in dev!):
```bash
<db drop>
<db create>
<db migrate>
<db seed>
```
---
## Still stuck?
1. Check [_setup.md](_setup.md) — maybe a step was skipped.
2. Check [_ci.md](_ci.md) — maybe it's not a local problem.
3. Search the project issue tracker for the error message.
4. **After solving — add the fix here.**