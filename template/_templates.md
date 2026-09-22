---
type: templates
title: "Issue and PR Workflow Templates"
description: "Standard templates for PROJECT_SUMMARY, PLAYBOOK, and PR descriptions"
timestamp: <YYYY-MM-DD>
tags: [templates, workflow, issue, pr]
---

# Issue Workflow Templates

Three templates for the issue lifecycle. For the process, see
[AGENTS.md](AGENTS.md#issue-workflow)

## File locations

| Template | File | Directory |
|----------|------|-----------|
| PROJECT_SUMMARY | `PROJECT_SUMMARY_<N>.md` | `issue/` |
| PLAYBOOK | `PLAYBOOK_<N>.md` | `playbook/` |
| PR description | `PR_<N>.md` | `pr/` |

After the PR is merged, move all three to `archive/`

---

## 1. PROJECT_SUMMARY

```markdown
---
type: project-summary
issue: "#<N>"
title: "<title>"
status: draft | in-progress | completed
timestamp: <YYYY-MM-DD>
tags: [<tag>]
---

# Project Summary: #<N> <title>

## Issue Overview

<Issue link, one-line description>

## Problem

<What needs to be done and why>

## Solution

<How it was resolved, which files changed>

## Verification

- [ ] `<lint command>` — 0 offenses
- [ ] `<test command>` — all pass
- [ ] `<build command>` — passes
- [ ] <specific verification for this change>

## Key Discoveries

<What was learned, insights worth remembering>

## Files Changed

| File | Change |
|------|--------|
| `<path>` | <what changed> |

## References

- [1] [Issue #<N>](<url>)
- [2] [Related PR](<url>)
```

---

## 2. PLAYBOOK

```markdown
---
type: playbook
issue: "#<N>"
title: "<title>"
status: draft | in-progress | completed
timestamp: <YYYY-MM-DD>
tags: [<tag>]
---

# Playbook #<N>: <title>

## Context

<Brief problem description, why this approach>

## Strategy

<Step-by-step plan>

1. <step 1>
2. <step 2>
3. <step 3>

## Patterns Used

<Which design/architectural patterns applied, why>

## Known Pitfalls

<What to watch out for, how to avoid>

## Verification Commands

```bash
<command to verify the change>
```
---

## 3. PR description

```markdown
---
type: pr
issue: "#<N>"
title: "<PR title>"
status: draft | ready-for-review | in-review | merged
---

## Description

<What was changed and why, 2-3 sentences>

## Related Issue

Fixes #<N>

## Changes

| File | Change |
|------|--------|
| `<path>` | <what changed> |

## Verification

- [ ] Build passes
- [ ] Tests pass
- [ ] Lint passes
- [ ] No unrelated changes

## Notes for Reviewers

<What to check especially carefully>

## References

- [1] [Related discussion](<url>)
```

---
## References

- [1] [Design doc](<url>)

## Lifecycle

1. **Start** — create `issue/PROJECT_SUMMARY_<N>.md` and
   `playbook/PLAYBOOK_<N>.md` from templates above
2. **During** — update both as work progresses; log sessions in
   [WORK_LOG.md](WORK_LOG.md); record significant decisions in
   [_decisions.md](_decisions.md)
3. **End** — create `pr/PR_<N>.md` from template; open the PR with the
   description; post `@<reviewer> please review` (or use the platform's
   reviewer UI); after merge, move all three files to `archive/`