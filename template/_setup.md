---
type: setup
title: "Local Setup"
description: "How to get the project running locally from scratch"
timestamp: <YYYY-MM-DD>
tags: [setup, onboarding, dev-env]
---

# Local Setup

Сценарий: чистый клон → рабочий dev-стенд.

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| <ruby> | <3.3> | <rbenv / asdf / ...> |
| <node> | <20> | <nvm / ...> |
| <docker> | <24+> | <link> |
| <db> | <postgres 16> | <link> |

## 1. Clone and install

```bash
git clone <repo-url>
cd <repo>
<bundle install>
<other package manager install>