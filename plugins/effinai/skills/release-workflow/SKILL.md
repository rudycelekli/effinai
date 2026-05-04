---
name: release-workflow
description: "End-to-end software release management. Automated changelog generation from conventional commits, semantic version bumping, release readiness checklists, hotfix procedures, rollback planning, and feature flag integration."
version: "1.0.0"
tags: [release, versioning, changelog, deployment, semver, hotfix]
createdBy: "built-in"
status: "active"
---

# Release Manager

## Overview

End-to-end software release management. From parsing conventional commits to generating changelogs, determining version bumps, and orchestrating release processes for reliable, predictable, well-documented releases.

## Activation

When the user asks to plan releases, manage changelogs, coordinate deployments, create release branches, or automate versioning.

## Semantic Versioning (SemVer)

**MAJOR.MINOR.PATCH**
- **MAJOR**: incompatible API changes
- **MINOR**: backwards compatible new functionality
- **PATCH**: backwards compatible bug fixes

Pre-release: `1.0.0-alpha.1` < `1.0.0-beta.1` < `1.0.0-rc.1` < `1.0.0`

## Conventional Commits

```
<type>[optional scope]: <description>
[optional body]
[optional footer(s)]
```

| Type | Version Bump | Description |
|------|-------------|-------------|
| `feat` | MINOR | New feature |
| `fix` | PATCH | Bug fix |
| `feat!` / `BREAKING CHANGE` | MAJOR | Breaking change |
| `perf` | PATCH | Performance improvement |
| `docs`, `style`, `test`, `chore`, `ci` | NO BUMP | Non-functional |

## Changelog Structure

```markdown
## [1.2.0] - 2024-01-15
### Added
- OAuth2 authentication support (#123)
### Fixed
- Race condition in user creation (#134)
### Breaking Changes
- Removed legacy payment API
```

## Release Branch Workflows

### Git Flow
```
main <- release/1.2.0 <- develop <- feature/login
```
Best for: teams needing structured releases with parallel feature development.

### Trunk-based Development
```
main <- feature/login (short-lived, 1-3 days)
```
Best for: fast integration, CI-friendly, feature flags for incomplete features.

### GitHub Flow
```
main <- feature/login (PR-based)
```
Best for: simple, lightweight, fast deployment cycles.

## Feature Flag Integration

```python
if feature_flag("new_payment_flow", user_id):
    return new_payment_processor.process(payment)
else:
    return legacy_payment_processor.process(payment)
```

1. Deploy code with feature behind flag (disabled)
2. Gradually enable for percentage of users
3. Monitor metrics and error rates
4. Full rollout or quick rollback based on data
5. Remove flag in subsequent release

## Release Readiness Checklists

### Pre-Release
- [ ] All planned features implemented and tested
- [ ] Breaking changes documented with migration guide
- [ ] API documentation updated
- [ ] Database migrations tested
- [ ] Security review completed
- [ ] Performance testing passed thresholds

### Quality Gates
- [ ] Unit test coverage >= 85%
- [ ] Integration tests passing
- [ ] Static analysis clean
- [ ] Security scan passed
- [ ] Dependency audit clean

### Documentation
- [ ] CHANGELOG.md updated
- [ ] Migration guide for breaking changes
- [ ] Deployment notes prepared
- [ ] Rollback procedure documented

## Deployment Coordination

### Sequence
1. **Pre-deployment** (T-24h): Final validation, freeze code
2. **Database migrations** (T-2h): Run and validate schema changes
3. **Blue-green deployment** (T-0): Switch traffic gradually
4. **Post-deployment** (T+1h): Monitor metrics and logs
5. **Rollback window** (T+4h): Decision point for rollback

## Hotfix Procedures

| Severity | SLA | Process |
|----------|-----|---------|
| **P0 Critical** | Fix within 2h | Emergency deployment, all hands |
| **P1 High** | Fix within 24h | Expedited review |
| **P2 Medium** | Next release | Normal process |

### Emergency Response
1. Page on-call team
2. Assess severity and impact
3. Create hotfix branch from last stable release
4. Minimal fix (root cause only)
5. Expedited testing
6. Emergency deployment
7. Post-incident root cause analysis

## Rollback Planning

### Triggers
- Error rate spike: >2x baseline within 30 minutes
- Performance degradation: >50% latency increase
- Core functionality broken
- Security incident or data corruption

### Types
- **Code:** Revert to previous Docker image; prefer feature flag disable
- **Database:** Only for non-destructive migrations; forward-only preferred
- **Infrastructure:** Blue-green switch, load balancer revert

## Release Metrics (DORA)

| Metric | Description |
|--------|-------------|
| **Lead Time** | From commit to production |
| **Deployment Frequency** | Releases per week/month |
| **MTTR** | Mean Time to Recovery |
| **Change Failure Rate** | % of releases causing incidents |

## Anti-Patterns

- Manual deployments (error-prone, inconsistent)
- Monolithic releases (large, infrequent, high risk)
- No rollback plan (unable to recover quickly)
- Hero culture (relying on individuals instead of process)
- Last-minute changes (risk without proper testing)

## Tools

| Script | Purpose |
|--------|---------|
| `changelog_generator.py` | Parse git logs, generate structured changelogs |
| `version_bumper.py` | Determine version bumps from conventional commits |
| `release_planner.py` | Assess release readiness, generate coordination plans |
