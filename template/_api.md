---
type: api
title: "API Reference"
description: "Public API — endpoints, methods, commands, authentication flow"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [api, integration]
---

# API Reference

Complete reference for public interfaces consumers interact with. For internal implementation details, see [_concepts.md](_concepts.md).

## Endpoints

### Base URL

```
<https://api.example.com/v1>
```

### Authentication

- **Method:** <Bearer token / API key / OAuth 2.0 / JWT>
- **Header:** `Authorization: Bearer <token>`
- **Token lifecycle:** <expires in X hours / refresh mechanism>
- **Scopes/roles:** <list required scopes and what they control>

### Rate Limiting

| Endpoint | Limit | Window |
|----------|-------|--------|
| `/api/...` | <requests per minute/hour> | <sliding window / fixed> |

Exceeding limits returns HTTP `<status code>` with `Retry-After` header.

## Methods / Commands

### <Endpoint/Command Name>

- **HTTP Method:** `<GET / POST / PUT / DELETE / PATCH>`
- **Path:** `<path pattern, e.g. /users/:id>`
- **Auth required:** `<yes / scope_name>`
- **Request schema:**

```json
{
  "<field>": "<type>",
  "<required_field>": "<type>"
}
```

- **Response schema (200 OK):**

```json
{
  "<response_field>": "<type>",
  "<paginated_data>": "<array or object>"
}
```

- **Error responses:**

| Status | Meaning | Action |
|--------|---------|--------|
| `<code>` | <error description> | <what to do> |

- **Example curl:**

```bash
curl -X <METHOD> <full_url> \
  -H "Authorization: Bearer <token>"
```

---

## CLI Commands

If the project exposes a CLI, document its core commands here:

| Command | Purpose | Flags | Example |
|---------|---------|-------|---------|
| `<command>` | <what it does> | `<--flag>` | `<example usage>` |

### Common flags

| Flag | Default | Description |
|------|---------|-------------|
| `--help` | — | Show help for command |
| `--version` | — | Print version |
| `<env_flag>` | `<default>` | Target environment |

## Versioning

- **Scheme:** <SemVer / semantic versioning / other>
- **Strategy:** <breaking vs non-breaking changes policy>
- **Deprecation process:** <how old versions are sunsetted>
- **Migration path:** <link to changelog or migration guide>

## SDKs and Libraries

Officially supported client libraries:

| Language | Library | Install |
|----------|---------|---------|
| <lang> | <package name> | `<install command>` |

## Webhooks

For event-driven integrations:

| Event | Payload fields | Retry policy |
|-------|---------------|--------------|
| `<event_name>` | `<key, type, meaning>` | <number> retries with backoff |

Subscription endpoint: `<URL to register webhooks>`
