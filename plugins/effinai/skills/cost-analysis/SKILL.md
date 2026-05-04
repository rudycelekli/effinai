---
name: cost-analysis
description: "Cloud infrastructure cost analysis and optimization recommendations"
version: "1.0.0"
tags: [devops, cost, cloud, optimization, finops]
createdBy: "built-in"
status: "active"
---

# Cost Analysis

## Activation
When the user asks to analyze cloud costs, optimize infrastructure spending, or review resource utilization.

## Workflow

### Step 1: Identify Infrastructure
1. Check for infrastructure-as-code files:
   - Terraform (.tf files)
   - CloudFormation (template.yaml)
   - Pulumi
   - Docker Compose
   - Kubernetes manifests
2. Check for cloud provider configurations
3. Identify all deployed services and their resource allocations

### Step 2: Analyze Resource Allocation

**Compute:**
| Resource | Current Spec | Utilization | Right-Sized | Monthly Savings |
|----------|-------------|-------------|-------------|-----------------|
| [service] | [instance type] | [CPU/mem %] | [recommended] | [$amount] |

Common waste patterns:
- Over-provisioned instances (CPU < 20%, memory < 30%)
- Running 24/7 when only needed during business hours
- Using on-demand when reserved/spot would work
- Running in expensive regions without latency need

**Storage:**
- Unused EBS volumes / persistent disks
- Old snapshots and backups beyond retention
- S3/GCS storage class optimization (Infrequent Access, Glacier)
- Large log files without rotation

**Database:**
- Over-provisioned RDS/Cloud SQL instances
- Multi-AZ when single-AZ sufficient for dev/staging
- Read replicas not being used
- Unused database instances

**Network:**
- Cross-region data transfer
- NAT Gateway costs (often surprisingly high)
- Idle load balancers
- Unused elastic IPs

### Step 3: Generate Optimization Plan

```
## Cost Optimization Report

### Current Monthly Estimate
| Category | Monthly Cost |
|----------|-------------|
| Compute | $X |
| Database | $X |
| Storage | $X |
| Network | $X |
| Other | $X |
| **Total** | **$X** |

### Quick Wins (implement this week)
1. [Action] — Save $X/month
   Risk: Low
   Steps: [commands/changes]

### Medium-Term (implement this month)
1. [Action] — Save $X/month
   Risk: Medium
   Steps: [approach]

### Strategic (plan for next quarter)
1. [Action] — Save $X/month
   Risk: [assessment]
   Requires: [prerequisites]

### Projected Savings
| Timeframe | Savings | % Reduction |
|-----------|---------|-------------|
| Quick wins | $X/mo | X% |
| + Medium-term | $X/mo | X% |
| + Strategic | $X/mo | X% |
```

### Step 4: Implementation Recommendations

**Auto-scaling:**
- Set up horizontal pod autoscaling or EC2 auto-scaling
- Define min/max based on traffic patterns
- Use predictive scaling for known traffic patterns

**Scheduling:**
- Shut down dev/staging environments outside business hours
- Use Lambda/Cloud Functions for infrequent tasks instead of always-on servers

**Commitment discounts:**
- Reserved Instances or Savings Plans for stable workloads
- Spot/Preemptible instances for fault-tolerant workloads

**Architecture changes:**
- Move to serverless for variable workloads
- Use CDN for static content
- Implement caching to reduce backend calls

## Rules
- Always estimate savings with specific dollar amounts
- Consider reliability trade-offs for each optimization
- Don't recommend changes that compromise security or availability
- Prioritize by savings-to-effort ratio
- Account for data transfer costs (often overlooked)
- Recommend monitoring/alerting for cost anomalies
- Consider reserved instance commitments carefully (1yr vs 3yr)
