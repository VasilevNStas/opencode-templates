---
type: env
title: "Environments"
description: "Map of deployment environments — URLs, ownership, access"
timestamp: <YYYY-MM-DD>
tags: [env, infrastructure]
---
# Environments
Non-local environments. For local setup, see [_setup.md](_setup.md).
For secrets handling, see [_security.md](_security.md).
## Overview
| Environment | Purpose | URL | Owner |
|-------------|---------|-----|-------|
| Staging | Pre-prod testing | <https://staging.example.com> | <@team> |
| Prod | Live users | <https://example.com> | <@team> |
| <Other> | <Purpose> | <URL> | <@team> |
## Where things live
| Component | Staging | Prod |
|-----------|---------|------|
| Database | <provider, region> | <provider, region> |
| Cache | <provider> | <provider> |
| Logs | <where> | <where> |
| Monitoring | <where> | <where> |
| Error tracking | <where> | <where> |
## Access
| Environment | How to get access |
|-------------|-------------------|
| Staging | <SSO group / VPN / ticket> |
| Prod | <on-call only / SSO + approval> |
## Deploy
| Environment | How to deploy | Trigger |
|-------------|---------------|---------|
| Staging | <CI workflow name> | Push to `master` |
| Prod | <CI workflow name> | Tag `v*` / manual |
For incident procedures, see [runbooks/](runbooks/index.md).
## Prod restrictions
- **Never** modify prod data directly — use migrations or scripts.
- **Never** deploy outside the process — see Deploy above.
- **Never** share prod credentials — see [_security.md](_security.md).
- **Always** announce changes in <channel> before running.
## References
- [1] [Infra dashboard](<url>)
- [2] [Deploy runbook](runbooks/<file>.md)