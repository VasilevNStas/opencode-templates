---
type: runbook
title: "<scenario>"
description: "<one-line summary of when to use this runbook>"
severity: P0 | P1 | P2
last-tested: <YYYY-MM-DD>
timestamp: <YYYY-MM-DD>
tags: [runbook, area]
---

# Runbook: <scenario>

## When to use

<Exact trigger — what alert, error, or event means this runbook applies>

## Prerequisites

- <Access / credentials needed>
- <Tools that must be installed>
- <People to notify before starting>

## Steps

1. **<Step title>**

   ```bash
   <command>
   ```

   Expected: <what you should see>.

2. **<Step title>**

   ```bash
   <command>
   ```

   Expected: <what you should see>.

3. **<Step title>**

   ...

## Verification

<How to confirm the incident is resolved>

- [ ] <check 1>
- [ ] <check 2>
- [ ] <check 3>

## If it doesn't work

1. **Stop** Do not improvise
2. Escalate to <contact / channel>
3. Capture logs: `<command to capture logs>`

## Rollback

<How to undo the changes made by this runbook, if needed>

## Post-incident

- Record the incident in [_decisions.md](../_decisions.md) if it changed a process
- Update this runbook if the procedure differed
- Add follow-up tasks to [_backlog.md](../_backlog.md)

## References

- [1] [Related dashboard](<url>)
- [2] [Monitoring alert definition](<url>)