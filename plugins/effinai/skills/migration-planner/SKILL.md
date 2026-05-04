---
name: migration-planner
description: "Zero-downtime migration planning with compatibility validation, rollback strategies, data reconciliation, and risk assessment. Covers database, service, and infrastructure migrations."
version: "1.0.0"
tags: [migration, database, infrastructure, zero-downtime, rollback, planning]
createdBy: "built-in"
status: "active"
---

# Migration Architect

## Overview

Comprehensive tools and methodologies for planning, executing, and validating complex system migrations with minimal business impact. Combines proven migration patterns with automated planning for successful transitions between systems, databases, and infrastructure.

## Activation

When the user asks about database migrations, service migrations, infrastructure moves, zero-downtime deployments, or rollback planning.

## Migration Patterns

### Database Migrations

#### Expand-Contract Pattern
1. **Expand:** Add new columns/tables alongside existing schema
2. **Dual Write:** Application writes to both old and new schema
3. **Migration:** Backfill historical data to new schema
4. **Contract:** Remove old columns/tables after validation

#### Change Data Capture (CDC)
- Stream database changes to target system
- Maintain eventual consistency during migration
- Enable zero-downtime migrations for large datasets

#### Dual-Write Pattern
- Write to both source and target during migration
- Implement compensation patterns for write failures
- Use distributed transactions where consistency is critical

### Service Migrations

#### Strangler Fig Pattern
1. Route traffic through proxy/gateway
2. Implement new service functionality incrementally
3. Remove old service components as new ones prove stable
4. Monitor performance and error rates throughout

#### Parallel Run Pattern
1. Run both old and new services simultaneously
2. Route shadow traffic to both systems
3. Compare outputs to validate correctness
4. Shift traffic percentage based on confidence

#### Canary Deployment
1. Deploy new service to small percentage of users
2. Monitor key metrics (latency, errors, business KPIs)
3. Gradually increase traffic as confidence grows
4. Full rollout once validation passes

### Infrastructure Migrations

#### Cloud-to-Cloud
1. Inventory resources and dependencies
2. Map services to target cloud equivalents
3. Migrate non-critical workloads first (pilot)
4. Production migration with IaC for consistency

#### On-Premises to Cloud
- **Lift and Shift:** Minimal changes, quick migration
- **Re-architecture:** Redesign for cloud-native patterns
- **Hybrid:** Keep sensitive data on-premises, compute in cloud

## Feature Flags for Migrations

```python
class MigrationFeatureFlag:
    def __init__(self, flag_name, rollout_percentage=0):
        self.flag_name = flag_name
        self.rollout_percentage = rollout_percentage

    def is_enabled_for_user(self, user_id):
        hash_value = hash(f"{self.flag_name}:{user_id}")
        return (hash_value % 100) < self.rollout_percentage
```

## Circuit Breaker Pattern

Automatic fallback to legacy systems when new systems show degraded performance:
- Track failure count and threshold
- States: CLOSED (normal), OPEN (fallback), HALF_OPEN (testing recovery)
- Gradual return to normal operation

## Data Validation and Reconciliation

### Validation Strategies
1. **Row Count:** Compare record counts between source and target
2. **Checksums:** Generate checksums for critical data subsets
3. **Business Logic:** Run critical queries on both systems, compare aggregates

### Delta Detection
```sql
SELECT 'missing_in_target' as issue_type, source_id
FROM source_table s
WHERE NOT EXISTS (SELECT 1 FROM target_table t WHERE t.id = s.id)
UNION ALL
SELECT 'extra_in_target', target_id
FROM target_table t
WHERE NOT EXISTS (SELECT 1 FROM source_table s WHERE s.id = t.id);
```

## Rollback Strategies

### Database Rollback
- Schema version control with rollback scripts for each step
- Point-in-time recovery using backups
- Transaction log replay for precise rollback points

### Service Rollback
- Blue-green: switch traffic back to previous version
- Rolling rollback: gradually shift traffic back
- Feature flag disable preferred over code rollback

### Rollback Triggers
- Error rate spike: >2x baseline within 30 minutes
- Performance degradation: >50% latency increase
- Core functionality broken
- Data corruption detected

## Risk Assessment Framework

### Technical Risks
- Data loss or corruption
- Service downtime or degraded performance
- Integration failures with dependent systems

### Business Risks
- Revenue impact from disruption
- Customer experience degradation
- Compliance and regulatory concerns

### Mitigations
- Comprehensive testing (unit, integration, load, chaos)
- Gradual rollout with automated rollback triggers
- Data validation and reconciliation processes

## Migration Runbooks

### Pre-Migration
- [ ] Plan reviewed and approved
- [ ] Rollback procedures tested
- [ ] Monitoring configured
- [ ] Team roles defined
- [ ] Backups verified
- [ ] Performance benchmarks established
- [ ] Security review completed

### During Migration
- [ ] Execute phases in planned order
- [ ] Monitor KPIs continuously
- [ ] Validate data at each checkpoint
- [ ] Communicate progress to stakeholders
- [ ] Execute rollback if success criteria not met

### Post-Migration
- [ ] All success criteria met
- [ ] System health checks pass
- [ ] Data reconciliation complete
- [ ] Monitor performance over 72 hours
- [ ] Decommission legacy systems
- [ ] Conduct retrospective

## Communication Templates

### Executive Summary
```
Migration Status: [IN_PROGRESS | COMPLETED | ROLLED_BACK]
Current Phase: [X of Y]
Overall Progress: [X%]
Risk Assessment: [LOW | MEDIUM | HIGH]
Rollback Status: [AVAILABLE | NOT_AVAILABLE]
```

## Tools

| Script | Purpose |
|--------|---------|
| `migration_planner.py` | Automated migration plan generation |
| `compatibility_checker.py` | Schema and API compatibility analysis |
| `rollback_generator.py` | Comprehensive rollback procedure generation |
