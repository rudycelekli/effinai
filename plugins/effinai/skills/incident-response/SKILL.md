---
name: incident-response
description: "Incident response runbook generation and triage assistance"
version: "1.0.0"
tags: [devops, incident, response, runbook, on-call]
createdBy: "built-in"
status: "active"
---

# Incident Response

## Activation
When the user reports a production incident, outage, or asks to create incident response procedures.

## Triage Workflow (Active Incident)

### Step 1: Assess Severity
| Severity | Impact | Response Time |
|----------|--------|---------------|
| SEV1 | Complete outage, data loss | Immediate, all hands |
| SEV2 | Major feature broken, degraded for many users | 15 min, on-call + lead |
| SEV3 | Minor feature broken, workaround available | 1 hour, on-call |
| SEV4 | Cosmetic issue, edge case | Next business day |

### Step 2: Diagnose
```bash
# Check application health
curl -s http://localhost:3000/health | jq .

# Check recent deployments
git log --oneline -10

# Check error logs
# [project-specific log commands]

# Check resource usage
docker stats 2>/dev/null || echo "Not using Docker locally"
```

Questions to answer:
1. When did it start? (Check deploy times, cron schedules)
2. What changed? (Recent deploys, config changes, traffic spikes)
3. Who is affected? (All users, specific segment, specific region)
4. Is it getting worse? (Check error rate trend)

### Step 3: Mitigate
Prioritize stopping the bleeding over finding root cause:
1. **Rollback** — If recent deploy caused it, revert immediately
2. **Feature flag** — Disable the broken feature
3. **Scale up** — If traffic-related, add capacity
4. **Failover** — Switch to backup/secondary system
5. **Block** — Rate limit or block abusive traffic

### Step 4: Communicate
```
## Incident Update
**Status**: Investigating / Identified / Monitoring / Resolved
**Severity**: SEV[1-4]
**Impact**: [Who/what is affected]
**Started**: [Time]
**Current Action**: [What's being done now]
**Next Update**: [Time of next update]
```

### Step 5: Resolve and Document

## Runbook Generation Workflow

When asked to create a runbook:

```markdown
# Runbook: [Service/Component Name]

## Service Overview
- **Purpose**: [What this service does]
- **Dependencies**: [Upstream and downstream services]
- **On-Call**: [Team/rotation]

## Common Alerts and Responses

### Alert: [Alert Name]
**Trigger**: [What triggers this alert]
**Impact**: [What users experience]
**Steps**:
1. Check [specific dashboard/log]
2. If [condition], then [action]
3. If [other condition], then [other action]
4. Escalate to [team] if not resolved in [time]

### Alert: High Error Rate
**Steps**:
1. Check error logs for the most common error
2. Check if a recent deployment correlates with the spike
3. If deploy-related: rollback via [command]
4. If traffic-related: check rate limiting, scale up
5. If dependency-related: check [dependency] health

## Rollback Procedures
1. [Exact rollback commands for this service]
2. Verify rollback: [health check commands]
3. Notify: [communication channels]

## Escalation Path
1. On-call engineer (0-15 min)
2. Team lead (15-30 min)
3. Engineering manager (30-60 min)
4. VP Engineering (60+ min, SEV1 only)

## Post-Incident
- [ ] Write incident timeline
- [ ] Identify root cause
- [ ] Create action items to prevent recurrence
- [ ] Schedule post-mortem meeting
- [ ] Update this runbook with lessons learned
```

## Rules
- During active incidents: mitigate first, root cause later
- Keep stakeholders informed with regular updates
- Document everything during the incident (timeline, actions, decisions)
- Never blame individuals in post-mortems
- Every incident should result in at least one preventive action item
- Update runbooks with lessons from each incident
