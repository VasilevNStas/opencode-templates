---
type: worklog
title: "Work Log Template"
description: "Format and rules for WORK_LOG.md"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: draft
tags: [worklog, log]
---

# Work Log Template

`WORK_LOG.md` is a chronological record of all work sessions. It lives in
`.opencode/` and is **never committed**

- Newest entries first
- Grouped by date (ISO 8601: `YYYY-MM-DD`)
- One entry per session

See [AGENTS.md](AGENTS.md#issue-workflow) for how sessions fit into the
issue lifecycle

## Entry format

```markdown
## <YYYY-MM-DD>

### Session <N> — <short title>

| # | What | Files | Status | Complexity |
|---|------|-------|--------|-----------|
| <PR#> | <short description> | <files> | open / merged | low / medium / high |

**Decision:** <non-obvious choices and why>

**Problem:** <issues encountered, how resolved>

**Next:** <what to do next session>

---
```

For trivial sessions (typo fix, one-line change), skip the table:

```markdown
## <YYYY-MM-DD>

### Session <N> — <short title>

<one-paragraph description>

**Next:** <what to do next session>

---
```

## Rules

- **Newest first** Insert new entries right after this file's intro,
  before the first `## <date>` heading
- **One entry per session.** Two sessions in one day → two entries
- **Log:** PRs, CI fixes, discoveries, blockers, decisions
- **Note WHY,** not just what. Context for future self
- **Link, don't duplicate**
  - Significant decisions → [_decisions.md](_decisions.md)
  - Future tasks → [_backlog.md](_backlog.md)
  - Issue details → `issue/PROJECT_SUMMARY_<N>.md`
- **Local only** `WORK_LOG.md` is never committed

## How to add a new entry

1. Copy the entry template from above
2. Fill in date, session number, and content
3. Open `.opencode/WORK_LOG.md`
4. Insert the new entry **immediately after the intro paragraph** and
   **before the first existing `## <date>` heading**