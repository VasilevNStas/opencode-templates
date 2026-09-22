---
type: ci
title: "CI Workflows"
description: "CI workflow reference and common failure patterns"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [ci, workflows]
---

# CI Workflows

Diagnostics for CI failures. For local problems, see
[_troubleshooting.md](_troubleshooting.md). For CI-specific commands,
see below.

**All workflows must pass** before a PR can be merged.

## Workflows

| Workflow | What it checks | Reproduce locally |
|----------|----------------|-------------------|
| `<name>.yml` | <purpose> | `<command>` |
| `<name>.yml` | <purpose> | `<command>` |

## Common failures

| Failure | Likely cause | Fix |
|---------|-------------|-----|
| `<error message>` | <root cause> | `<command>` |
| `Lint: N offenses` | Style violations | `<lint command>` |
| `Tests failed` | Real test failure | `<single test command>` |

## Local reproduction

Full CI simulation (if supported by the platform):

```bash
# GitHub Actions
act -j <job-name>

# GitLab CI
gitlab-runner exec shell <job-name>
```

Faster alternative — run the individual commands from `Workflows` above.

## When CI passes locally but fails remotely

1. **Check versions.** Local tool versions may differ from CI (Ruby, Node,
   PostgreSQL, etc.). Verify via `.tool-versions`, `.ruby-version`, or the
   workflow YAML.
2. **Check environment variables.** CI has its own set. See
   [_security.md](_security.md) and [_env.md](_env.md).
3. **Check parallelism.** CI may run tests in parallel — race conditions
   appear there first.
4. **Check caching.** CI cache may be stale. Try clearing it.
5. **Re-run the job** with debug logging enabled.