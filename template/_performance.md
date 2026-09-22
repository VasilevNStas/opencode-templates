---
type: performance
title: "Performance and Benchmarks"
description: "Profiling tools, benchmarks, SLA/SLO targets, capacity planning"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
stale_after: 2027-12-31
tags: [performance, profiling, monitoring]
sources:
  - id: google-sre-workbook
    resource: https://sre.google/sre-workbook/table-of-contents/
    title: Google SRE Workbook
    author: team:sre-google
    last_modified: 2025-12-01
  - id: hpbn-book
    resource: https://hpbn.co/
    title: High Performance Browser Networking
    author: niels-prohaska
    last_modified: 2014-06-01
---

# Performance and Benchmarks

Document for tracking and improving system performance. For general error handling in [_concepts.md](_concepts.md). For incident procedures, see runbooks/.

## SLA and SLO Targets

| Metric | Target | Measurement | Alert threshold |
|--------|--------|-------------|-----------------|
| **Latency (P50)** | <e.g. 50ms> | p50 over last 1h | > <value> |
| **Latency (P99)** | <e.g. 200ms> | p99 over last 1h | > <value> |
| **Availability** | <e.g. 99.9%> | uptime % per month | < <value> |
| **Throughput** | <e.g. 1k req/s> | requests/sec avg | < <value> |
| **Error rate** | <e.g. <0.1%> | 5xx / total requests | > <value> |

## Profiling Tools

| Tool | Use case | Install / Run |
|------|----------|---------------|
| `<tool_1>` | <e.g. CPU profiling, memory leaks> | `<command>` |
| `<tool_2>` | <e.g. flame graphs> | `<command>` |
| `<tool_3>` | <e.g. network tracing> | `<command>` |

### Running a profile

```bash
# Basic profiling command
<profiling-command --output report.json>

# Generate a visual report
<report-generator-command report.json>
```

Output: reports land in `<reports_dir>`, reviewed quarterly or before major releases.

## Benchmarking

Benchmarks track regressions and validate improvements:

- **Location:** `bench/` directory at repository root (or `<benchmark-dir>`)
- **Framework:** <benchmark framework name, e.g. Hyperfine, Go bench, k6>
- **Command:** `<benchmark-command>`
- **CI integration:** run benchmarks on every PR; fail if regression exceeds `<X>%`

### Recording baseline results

After validating a change:

```bash
<benchmark-command --save baseline_YYYYMMDD.json>
```

Baseline files are checked into version control for diff comparison across time.

## Bottleneck Tracking

Known performance bottlenecks — updated during analysis sessions:

| # | Component | Symptom | Impact | Status |
|---|-----------|---------|--------|--------|
| P-001 | <component_name> | <symptom description> | <severity: high/medium/low> | identified / optimized / resolved |

### How to diagnose a bottleneck

1. Check current dashboards: <link_to_dashboards>
2. Compare against SLA/SLO targets above
3. Profile with tool from the table above
4. Create a task in [_backlog.md](_backlog.md) or an issue tracker
5. If urgent: open an incident via runbooks/

## Capacity Planning

| Resource | Current usage | Growth rate | Action when to scale |
|----------|--------------|-------------|---------------------|
| **CPU** | <e.g. 40% avg peak> | <e.g. +5%/quarter> | > 70% sustained |
| **Memory** | <e.g. 6GB of 8GB> | <rate> | > 80% headroom needed |
| **Disk** | <e.g. 200GB of 500GB> | <rate> | > 70% full |
| **Network bandwidth** | <current / max> | <rate> | > 60% saturation |

Scaling triggers are managed through <infra-ticket-system> or auto-scaling policies.

## CI Performance Checks

Performance-sensitive checks enforced in CI:

| Check | Threshold | Command |
|-------|-----------|---------|
| Response time | <P99 < X ms> | `<ci-performance-check>` |
| Memory leak detection | <delta < X MB/hour> | `<memory-leak-check>` |
| Load test smoke | <error rate < Y%> | `<load-test-smoke>` |
