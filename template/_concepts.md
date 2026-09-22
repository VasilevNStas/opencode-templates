---
type: architecture
title: "<project-name> — Architecture"
description: "<one-line summary of system architecture>"
timestamp: <YYYY-MM-DD>
tags: [architecture, <language>]
---

# Architecture

## Overview

<2–3 sentences: what the system does, what style (monolith, microservices,
pipeline, event-driven), how components relate>

## Key components

| Component | Purpose | Dependencies |
|-----------|---------|--------------|
| `<name>` | `<what it does>` | `<depends on>` |

## Data flow

<How data moves through the system — inputs, processing, outputs>

```
[Input] → [Component A] → [Component B] → [Output]
         ↘ [Component C] ↗
```

## Key patterns

<Architectural patterns used: decorator, pipeline, pub-sub, EO, CQRS, ...>

## Deployment

- **Type:** <Docker / Heroku / bare metal / serverless>
- **Entry:** <how the system starts>
- **Env vars:** <key configuration variables>

## Error handling

<How the system handles failures — retries, fallbacks, logging>

## Configuration

<How the system is configured — env vars, config files, CLI flags>

## Testing strategy

- **Unit:** <framework, coverage target>
- **Integration:** <what is integrated>
- **E2E:** <end-to-end tests>

## Citations

- [1] [Design doc](<url>)