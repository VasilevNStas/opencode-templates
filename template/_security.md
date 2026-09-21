---
type: security
title: "Security & Secrets"
description: "How secrets are handled, what must never be committed"
timestamp: <YYYY-MM-DD>
tags: [security, secrets]
---
# Security & Secrets
## Hard rules
- **Никогда не коммитить** реальные секреты, ключи, токены, пароли.
- **Никогда не логировать** PII / credentials.
- Секреты — только через env vars или secret manager (`<vault / AWS SM / GCP SM>`).
- Файлы с секретами перечислены в `.gitignore` и `.git/info/exclude`.
## Where secrets live
| Environment | Source | How to get |
|-------------|--------|-----------|
| Local dev | `.env` (не коммитится) | <от тимлида / из vault> |
| CI | GitHub Secrets | <repo settings> |
| Staging | <vault path> | <access request> |
| Prod | <vault path> | <on-call only> |
## What's safe to commit
- `.env.example` — с placeholder-значениями
- Публичные ключи (не приватные)
- Конфиги без секретов
## What must NEVER be committed
- `.env`
- `*.pem`, `*.key`, `id_rsa*`
- <project-specific patterns>
- Дампы БД с реальными данными
## If a secret leaked
1. Немедленно **отозвать / ротировать** секрет в источнике.
2. Уведомить <security contact / lead>.
3. Не пытаться «спрятать» через `git rebase` — история уже утекла.
4. Зафиксировать инцидент в `_decisions.md` (ADR) с follow-up.
## Reporting vulnerabilities
- <contact / security@ / bug bounty>
- Не открывайте публичный issue — используйте <private channel>.
## References
- [1] [OWASP Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [2] [GitHub: removing sensitive data](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)