---
name: migration-plan
description: "Database and system migration planning with rollback strategy"
version: "1.0.0"
tags: [productivity, migration, database, planning]
createdBy: "built-in"
status: "active"
---

# Migration Plan

## Activation
When the user asks to plan a migration (database, system, framework, API version, cloud provider).

## Workflow

### Step 1: Assess Current State
1. Identify what is being migrated (database, framework, API, infrastructure)
2. Map all dependencies on the current system
3. Identify data that needs to be preserved
4. Estimate scope: number of files, endpoints, queries affected
5. Identify breaking changes between old and new

### Step 2: Generate Migration Plan

```markdown
# Migration Plan: [From] -> [To]

## Overview
- **What**: [System/component being migrated]
- **Why**: [Business/technical reason]
- **Risk Level**: Low/Medium/High/Critical
- **Estimated Effort**: [time]
- **Rollback Window**: [time]

## Pre-Migration Checklist
- [ ] Full backup of current system/data
- [ ] All tests passing on current version
- [ ] Stakeholders notified
- [ ] Monitoring and alerting in place
- [ ] Rollback procedure tested
- [ ] Feature flags for gradual cutover (if applicable)

## Phase 1: Preparation
1. [Create compatibility layer / adapter]
2. [Set up new system alongside old]
3. [Write migration scripts]
4. [Test migration on staging with production data copy]

## Phase 2: Migration
1. [Step-by-step migration instructions]
2. [Verification after each step]
3. [Data integrity checks]

## Phase 3: Verification
- [ ] All automated tests pass
- [ ] Manual smoke tests pass
- [ ] Performance benchmarks meet baseline
- [ ] No data loss or corruption
- [ ] Monitoring shows normal metrics

## Phase 4: Cleanup
- [ ] Remove old system/code
- [ ] Remove compatibility layers
- [ ] Update documentation
- [ ] Archive old backups per retention policy

## Rollback Plan
1. [Exact steps to revert if migration fails]
2. [Data restoration procedure]
3. [Communication plan for stakeholders]
4. [Decision criteria: when to rollback vs push forward]

## Risks and Mitigations
| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| [risk] | High/Med/Low | High/Med/Low | [plan] |
```

### Step 3: Database-Specific Guidance

**Schema Migration:**
- Always write reversible migrations (up + down)
- Test down migrations on staging
- Use transactions for DDL when supported
- Add new columns as nullable first, backfill, then add NOT NULL

**Data Migration:**
- Batch large data migrations (1000-10000 rows at a time)
- Include progress logging
- Validate row counts before and after
- Keep old columns during transition period (expand-contract pattern)

**Zero-Downtime Pattern:**
1. Add new column (nullable)
2. Deploy code that writes to both old and new columns
3. Backfill new column from old data
4. Deploy code that reads from new column
5. Drop old column in future migration

## Rules
- Never migrate without a tested rollback plan
- Always backup before migration
- Test on staging with production-like data first
- Use the expand-contract pattern for zero-downtime changes
- Document the point of no return clearly
- Include verification steps between each phase
