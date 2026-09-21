---
type: troubleshooting
title: "Troubleshooting"
description: "Local (non-CI) problems and their fixes"
timestamp: <YYYY-MM-DD>
tags: [troubleshooting, dev-env]
---
# Troubleshooting
Проблемы **локальной** разработки. Падения CI — см. [`_ci.md`](_ci.md).
## Quick index
| Symptom | Section |
|---------|---------|
| `port already in use` | [Port conflicts](#port-conflicts) |
| `migration fails` | [Migrations](#migrations) |
| `gem install fails` | [Dependencies](#dependencies) |
| `tests hang` | [Tests](#tests) |
---
## Port conflicts
**Symptom:** `Address already in use` при запуске.
**Cause:** порт занят другим процессом или предыдущим запуском.
**Fix:**
```bash
lsof -i :<port>
kill -9 <pid>
# или смените порт через <env var>