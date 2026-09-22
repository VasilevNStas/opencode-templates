---
type: commands
title: "Quick Commands"
description: "Build, test, run, and utility command reference"
timestamp: <YYYY-MM-DD>
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
## HoC estimate
```bash
git fetch origin master
git diff --stat origin/master...HEAD | tail -1
gem install --user-install hoc
hoc origin/master..HEAD
```
## Branch for an issue
Adjust to your workflow (GitHub flow, GitFlow, trunk-based).
```bash
git checkout master
git pull
git checkout -b <issue-number>
```
## References
- [1] <link to project's contributing guide>