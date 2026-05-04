---
name: adr-lifecycle
description: "Architecture Decision Records workflow with templates, tracking, and review triggers"
version: "1.0.0"
tags: [architecture, decisions, documentation, governance]
---

# ADR Lifecycle

Manage Architecture Decision Records from proposal through deprecation. ADRs capture the WHY behind technical choices so future developers understand context.

## ADR Format

Every ADR follows this structure:

```markdown
# ADR-{NNN}: {Title}

**Date**: YYYY-MM-DD
**Status**: proposed | accepted | deprecated | superseded by ADR-{NNN}
**Deciders**: {list of people/agents involved}
**Tags**: {relevant domain tags}

## Context

What is the technical or business situation that requires a decision?
Include constraints, forces, and tensions that make this non-trivial.

## Decision

What is the change we are making? State it as an imperative:
"We will use PostgreSQL as our primary datastore."

## Consequences

### Positive
- What becomes easier or better

### Negative
- What becomes harder or worse

### Neutral
- What changes but isn't clearly better or worse

## Alternatives Considered

| Alternative | Pros | Cons | Why Rejected |
|------------|------|------|--------------|
| Option A | ... | ... | ... |
| Option B | ... | ... | ... |

## References
- Links to relevant code, PRs, external docs
```

## Status Lifecycle

```
proposed --> accepted --> [active use]
                |
                +--> deprecated (no longer applies)
                |
                +--> superseded by ADR-{NNN} (replaced by newer decision)
```

### Status Transitions

**proposed -> accepted**: Requires review by at least one architect-level agent or senior developer. The decision must have documented alternatives and clear consequences.

**accepted -> deprecated**: Triggered when the context that motivated the decision no longer applies. Document why in an amendment section.

**accepted -> superseded**: Triggered when a new ADR explicitly replaces this one. Both ADRs must cross-reference each other.

## File Organization

```
docs/adr/
  ADR-001-use-postgresql.md
  ADR-002-adopt-event-sourcing.md
  ADR-003-api-versioning-strategy.md
  INDEX.md                          # Auto-generated table of all ADRs
```

## Creating an ADR

1. Assign the next sequential number
2. Fill in all template sections - no section may be left empty
3. Set status to `proposed`
4. Link to the code or PR that motivates the decision
5. List at least 2 alternatives considered (even if obviously inferior)
6. Submit for review

## Review Triggers

Re-evaluate existing ADRs when:

- **Dependency major version upgrade**: Check if the ADR's assumptions still hold
- **Performance regression**: Check if architecture decisions contributed
- **Security incident**: Review security-related ADRs
- **Team scaling**: Decisions made for 3 developers may not work for 30
- **Cost threshold breach**: Review cost-related architectural choices
- **Quarterly review**: Scan all accepted ADRs for staleness

## ADR Index Template

```markdown
# Architecture Decision Records

| ADR | Title | Status | Date | Tags |
|-----|-------|--------|------|------|
| 001 | Use PostgreSQL | accepted | 2025-01-15 | database, storage |
| 002 | Adopt Event Sourcing | superseded by 005 | 2025-02-01 | architecture |
```

## Linking ADRs to Code

Add ADR references in code comments at the point where the decision manifests:

```typescript
// Architecture: ADR-003 - API versioning via URL path prefix
// See: docs/adr/ADR-003-api-versioning-strategy.md
app.use('/api/v2', routerV2);
```

## Example: Complete ADR

```markdown
# ADR-007: Use JWT with Refresh Tokens for Authentication

**Date**: 2025-03-10
**Status**: accepted
**Deciders**: security-architect, system-architect, coder-lead
**Tags**: security, authentication, api

## Context

Our API needs stateless authentication for horizontal scaling.
Sessions stored server-side would require shared session store,
adding a single point of failure and latency.

## Decision

We will use short-lived JWTs (15 min) with long-lived refresh
tokens (7 days) stored in HttpOnly cookies. Access tokens are
sent via Authorization header. Refresh tokens rotate on each use.

## Consequences

### Positive
- Stateless auth enables horizontal scaling without shared state
- Short token lifetime limits exposure window
- Refresh rotation detects token theft

### Negative
- Token revocation requires a deny-list (adds some state)
- Clock skew between services can cause false rejections
- More complex client-side token management

### Neutral
- Token size (~800 bytes) is larger than session IDs but acceptable

## Alternatives Considered

| Alternative | Pros | Cons | Why Rejected |
|------------|------|------|--------------|
| Server sessions + Redis | Simple, instant revocation | Shared state, SPOF | Scaling complexity |
| Opaque tokens + introspection | Simple tokens, central control | Extra network hop per request | Latency budget exceeded |

## References
- RFC 7519 (JWT)
- OWASP Token Best Practices
- PR #142: Auth implementation
```

## Quality Checklist

Before accepting any ADR:
- [ ] Context explains WHY a decision is needed (not just WHAT)
- [ ] Decision is stated as a clear imperative
- [ ] At least 2 alternatives are documented with rejection reasons
- [ ] Consequences include both positive AND negative impacts
- [ ] Tags are assigned for searchability
- [ ] Code locations where decision applies are identified
- [ ] Review date or trigger conditions are set

## Agent Assignments

| Role | Responsibility |
|------|---------------|
| `system-architect` | Draft and review ADRs |
| `security-architect` | Review security-related ADRs |
| `researcher` | Gather context and alternatives |
| `reviewer` | Validate completeness and clarity |
