---
type: release
title: "Release Process"
description: "Versioning, changelog, publishing, and rollback procedures"
generated: { by: human:creator, at: 2026-09-22T00:00:00Z }
status: stable
tags: [release, deployment, devops]
---

# Release Process

End-to-end process for shipping a new version of the project. For environment configuration, see [_env.md](_env.md).

## Versioning

- **Scheme:** <SemVer (MAJOR.MINOR.PATCH) / other>
- **Bumping rules:**
  - **MAJOR** — breaking API or data schema changes
  - **MINOR** — new features, backward-compatible
  - **PATCH** — bug fixes, backward-compatible
- **Command to bump:** `<bump-version command>`
- **Tagging convention:** `v<version>` (e.g., `v1.2.3`)

## Changelog

- **Tool:** <standard-changelog / keep-a-changelog / git log>
- **Location:** `CHANGELOG.md` at repository root
- **Format:** <adopted format or custom template>
- **Rule:** every PR that touches shipped code must include a changelog entry in the `Unreleased` section
- **Example entry:**

```markdown
### Added
- `<feature description with PR link>`

### Fixed
- `<bug fix description with PR link>`

### Changed
- `<behavior change description with PR link>`

### Removed
- `<removed feature with deprecation notice if applicable>`

### Security
- `<security fix CVE or advisory link>`
```

## Publish Commands

### Build and tag

```bash
<build-and-tag-command>
```

### Push to registry

| Artifact type | Registry | Command |
|---------------|----------|---------|
| Docker image | <registry name> | `<docker push command>` |
| Package (npm/rubygems/etc.) | <package manager registry> | `<publish command>` |
| Documentation | <hosting service> | `<doc deploy command>` |
| Binary artifacts | <CDN / S3 / Releases page> | `<upload command>` |

### Promotion gates

| Stage | Trigger | Gate |
|-------|---------|------|
| Staging | Push to `master` branch | CI passes, automated tests green |
| Production | Tag push (`v*`) or manual trigger | Approval from maintainer(s), smoke test passes |

## Rollback

### When to roll back

- Severity P0 incident (data corruption, service outage, security breach)
- Regression detected within first hour of deployment

### Rollback procedure

1. **Assess:** check severity and blast radius using analysis/ and runbooks/
2. **Revert the tag:** `<revert-tag-command>`
3. **Re-deploy previous version:** `<deploy-previous-version-command>`
4. **Verify:** run health checks and monitor error rates
5. **Communicate:** notify stakeholders via <channel>
6. **Document:** record decision in [_decisions.md](_decisions.md)

### Data migration rollback

If the release included database migrations:

1. **Stop writing to new tables/columns** immediately
2. **Run reverse migration** (if reversible): `<reverse-migration-command>`
3. If migration is irreversible: restore from snapshot → consult [<DBA/infra-team>]

## Pre-release Checklist

- [ ] All CI workflows passing
- [ ] Changelog updated in `Unreleased` section
- [ ] Dependencies audited (`<audit-command>`)
- [ ] Migration scripts tested against staging copy
- [ ] Release notes drafted for end users
- [ ] Backward compatibility confirmed (no concurrent vX and vX+1 clients)
- [ ] Monitoring dashboards reviewed (alert thresholds appropriate)

## Post-release Verification

Within 30 minutes of production deploy:

1. Smoke test the main flow: `<smoke-test-command>`
2. Check error rate dashboard: <link>
3. Verify canary instances are healthy
4. Monitor key metrics for anomalies
