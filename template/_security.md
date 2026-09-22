---
type: security
title: "Security & Secrets"
description: "How secrets are handled, what must never be committed"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
stale_after: 2027-12-31
tags: [security, secrets]
sources:
  - id: owasp-secrets
    resource: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
    title: OWASP Secrets Management Cheat Sheet
    author: team:owasp
    last_modified: 2026-01-15
  - id: github-sensitive-data
    resource: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository
    title: Removing sensitive data from a repository
    author: github-support
    last_modified: 2026-03-20
---
# Security & Secrets
## Hard rules
- **Never commit** real secrets, keys, tokens, or passwords
- **Never log** PII or credentials
- **Never paste** secrets into issues, PRs, or chat
- Secrets come from environment variables or a secret manager
- Files with secrets are listed in `.gitignore` and `.git/info/exclude`
## Where secrets live
| Environment | Source | How to get |
|-------------|--------|-----------|
| Local dev | `.env` (never committed) | <from team lead / vault> |
| CI | GitHub Secrets | <repo settings> |
| Staging | <vault path> | <access request> |
| Prod | <vault path> | <on-call only> |
## What's safe to commit
- `.env.example` with placeholder values only
- Public keys (never private keys)
- Configuration without secrets
## What must NEVER be committed
- `.env`, `.env.*` (except `.env.example`)
- `*.pem`, `*.key`, `id_rsa*`, `*.p12`
- Database dumps with real data
- Credentials in test fixtures or docs
- Screenshots containing credentials or tokens
- directory `.opencode`
## If a secret leaked
1. **Revoke/rotate** the secret at its source immediately
2. Notify <security contact>
3. Do not try to hide it with `git rebase` — history is already out
4. Record the incident in [_decisions.md](_decisions.md) as an ADR
Full procedure: see [runbooks/](runbooks/index.md)
## Reporting vulnerabilities
<Internal channel — email, Slack, ticket system.>
For public reporting (external researchers), see `SECURITY.md` in the
repository root if present.
- <contact / security@ / bug bounty>
- Don't open public issue — use <private channel>.