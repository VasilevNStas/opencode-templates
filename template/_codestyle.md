---
type: codestyle
title: "Code Style Conventions"
description: "SPDX headers, linting, language conventions, and testing setup"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [codestyle, language]
---

# Code Style

## SPDX headers

Required on all **source code** files. Markdown, YAML, JSON, and generated
files are exempt unless the project explicitly requires otherwise.

<language>:

```<ext>
# SPDX-FileCopyrightText: Copyright (c) <year> <author>
# SPDX-License-Identifier: <SPDX-ID>
```

`<SPDX-ID>` must match the license declared in [AGENTS.md](AGENTS.md).

## Lint — 0 offenses

```bash
<lint command>
```

Lint must pass with **zero offenses** before any commit.

## Language conventions

- <convention 1>
- <convention 2>
- <convention 3>

## Testing

- Deep testing strategy: [_testing.md](_testing.md)