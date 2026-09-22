---
type: commands
title: "Quick Commands"
description: "Build, test, run, and utility command reference"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [commands, reference]
---

# Quick Commands
Copy-paste ready. For setup order, see [_setup.md](_setup.md).
For lint rules, see [_codestyle.md](_codestyle.md).
## Build
```bash
<full build command>
```
## Test
```bash
# All tests
<all tests command>

# Single test
<single test command>

# Tests with coverage
<coverage command>
```
## Lint
```bash
<lint command>
```
## Run locally
```bash
<run command>
```
## HoC estimate (optional, Ruby-based)

```bash
git fetch origin master
git diff --stat origin/master...HEAD | tail -1
gem install --user-install hoc
hoc origin/master..HEAD
```
<!-- Replace `origin/master` with your target branch if different -->
<!-- For non-Ruby projects, remove this section or adapt as needed -->
## Branch for an issue
Adjust to your workflow (GitHub flow, GitFlow, trunk-based).
```bash
git checkout master
git pull
git checkout -b <issue-number>
```