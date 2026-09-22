---
type: testing
title: "Testing Strategy and Practice"
description: "Detailed testing setup — fixtures, mocking, golden files, coverage, CI"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [testing, quality]
sources:
  - id: gilded-rose-kata
    resource: https://gildedrose.codecrafters.io/
    title: Gilded Rose Kata
    author: codecrafters-team
    last_modified: 2026-01-01
  - id: xunit-patterns
    resource: https://xunitpatterns.com/
    title: xUnit Test Patterns
    author: eric-evans
    last_modified: 2007-09-01
---

# Testing Strategy and Practice

This file covers the **how** of testing. For strategy (what to test and why), see [_concepts.md](_concepts.md). For quick commands, see [_commands.md](_commands.md).

## Test Categories

### Unit tests

- **Scope:** single function or method in isolation
- **Framework:** <test framework name>
- **Run:** `<unit test command>`
- **Coverage target:** <percentage, e.g. 80%>
- **Golden rule:** no network calls, no disk I/O, no random data
- **Patterns:** dependency injection for collaborators, pure functions preferred

### Integration tests

- **Scope:** interaction between real components (database, cache, external API)
- **Framework:** <integration test framework>
- **Run:** `<integration test command>`
- **Scope:** only critical paths — happy path + one error scenario per endpoint
- **Database:** use a dedicated test database; reset before each test suite
- **Mocking:** only slow or non-deterministic external services (HTTP gateways)

### End-to-end tests (E2E)

- **Scope:** user-facing flows across multiple systems
- **Framework:** <e2e framework, e.g. Cypress, Playwright, Robot Framework>
- **Run:** `<e2e test command>`
- **When to write:** critical revenue paths or core UX flows
- **CI integration:** run on every PR with flaky-test detection

## Fixtures and Test Data

### Fixtures

- Location: `fixtures/` at repository root or within test directory
- Format: <JSON / YAML / factory classes>, avoid raw SQL dumps unless necessary
- Lifecycle: regenerate when schema changes break compatibility
- Seed data: create via `<db seed>` so new team members get consistent state

### Factories

- Use factories over hardcoded fixtures when objects have complex relationships
- Prefer builder patterns for objects with many optional fields
- Keep factories close to their domain models (not a single factory directory)

### Golden Files

- Location: `__snapshots__/` or similar, version-controlled
- When to use: output that must remain stable (HTML rendering, CLI output, serialized format)
- Update process: explicit flag (`--update-snapshots`), never auto-update in CI
- Review: diff snapshots carefully during code review

## Mocking and Stubbing

### What to mock

- External services (payment gateways, email providers)
- Slow operations (>100ms) in unit tests
- Non-deterministic behavior (random, time-based)

### What NOT to mock

- Internal helpers and utilities
- Domain logic — it is the code under test
- Real dependencies that are fast and reliable

### Best practices

- Define mocks as interfaces/protocols to avoid tight coupling
- Limit mock depth to 1–2 levels; deeper indirection signals poor design
- Use `no_mocks_allowed` mode in CI to catch accidental overshrinkage

## Test Utilities

Shared helper methods used across test suites:

| Utility | Purpose |
|---------|---------|
| `<helper_1>` | <description> |
| `<helper_2>` | <description> |

Location: <path/to/test/utilities>

## Flaky Tests

Flaky tests degrade trust in CI. Manage them proactively:

1. **Detect:** track flakiness in CI dashboard or logs
2. **Quarantine:** move failing-but-intermittent tests to a separate file; do not leave them in the main suite
3. **Fix:** investigate root cause (race conditions, stale state, timing)
4. **Remove:** if unreproducible after 3 attempts, delete with an ADR explaining why

## Coverage

- **Tool:** <coverage tool>
- **Command:** `<coverage command>`
- **Threshold:** minimum <X>% line coverage enforced in CI
- **Reporting:** generated HTML at `<report_path>`; reviewed quarterly
- **Limitations:** coverage measures execution, not correctness. Use as a guardrail, not a goal

## Test Environments

| Environment | Used for | Reset frequency |
|-------------|----------|-----------------|
| In-memory | Fast unit tests | Per-run |
| Dedicated DB | Integration tests | Per-run (via migration/rollback) |
| Full stack | E2E tests | Manual or CI-provisioned |

## Performance Tests (brief)

For deep profiling and benchmarking, see [_performance.md](_performance.md). Quick smoke checks live here:

```bash
# Smoke performance check (if available)
<smoke performance command>
```
