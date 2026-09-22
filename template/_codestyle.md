---
type: codestyle
title: "Code Style Conventions"
description: "SPDX headers, linting, language conventions, and testing setup"
timestamp: <YYYY-MM-DD>
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

- **Framework:** <test framework>
- **Run all:** <command>
- **Single test:** <command with name filter>
- **Patterns:** <fixtures, mocking, golden files, etc.>

Testing **strategy** (what we test and why) — see [_concepts.md](_concepts.md).

## References

- [1] [Official style guide](<url>)
- [2] [Linter config](<path/to/config>)