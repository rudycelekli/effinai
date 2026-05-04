---
name: cost-tracking
description: "LLM cost management with token tracking, budget alerts, and optimization strategies"
version: "1.0.0"
tags: [cost, optimization, monitoring, budget]
---

# Cost Tracking

Track, budget, alert, and optimize LLM token usage across models, agents, and tasks.

## Token Pricing Reference

| Model | Input (per 1M tokens) | Output (per 1M tokens) | Cache Read | Cache Write |
|-------|----------------------|------------------------|------------|-------------|
| Claude Opus | $15.00 | $75.00 | $1.50 | $18.75 |
| Claude Sonnet | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Haiku | $0.25 | $1.25 | $0.03 | $0.30 |
| GPT-4o | $2.50 | $10.00 | $1.25 | - |
| GPT-4o-mini | $0.15 | $0.60 | $0.075 | - |

Update these rates when providers change pricing.

## Tracking Schema

Record every LLM call with:

```typescript
interface TokenUsage {
  timestamp: string;        // ISO 8601
  model: string;            // e.g., "claude-sonnet-4-20250514"
  agentId: string;          // Which agent made the call
  taskId: string;           // Which task it belongs to
  inputTokens: number;
  outputTokens: number;
  cacheReadTokens: number;
  cacheWriteTokens: number;
  latencyMs: number;
  costUsd: number;          // Calculated from tokens * rates
  purpose: string;          // e.g., "code-generation", "review", "planning"
}
```

## Cost Calculation

```typescript
function calculateCost(usage: TokenUsage, rates: ModelRates): number {
  return (
    (usage.inputTokens / 1_000_000) * rates.inputPer1M +
    (usage.outputTokens / 1_000_000) * rates.outputPer1M +
    (usage.cacheReadTokens / 1_000_000) * rates.cacheReadPer1M +
    (usage.cacheWriteTokens / 1_000_000) * rates.cacheWritePer1M
  );
}
```

## Budget Configuration

```yaml
budgets:
  daily:
    limit: 50.00
    currency: USD
  weekly:
    limit: 250.00
  monthly:
    limit: 800.00
  per_task:
    limit: 10.00
  per_agent:
    limit: 25.00

alerts:
  thresholds: [50, 75, 90, 100]
  channels:
    - type: console    # Always log to stdout
    - type: webhook    # Optional: POST to URL
      url: "${COST_ALERT_WEBHOOK}"
    - type: memory     # Store alert in agent memory
      namespace: "cost-alerts"
```

## Alert System

Trigger alerts at configured thresholds:

```
[COST ALERT] Daily budget 50% reached: $25.12 / $50.00
  Top consumers:
    - coder-agent: $12.40 (49%)
    - researcher-agent: $8.20 (33%)
    - reviewer-agent: $4.52 (18%)

[COST ALERT] Per-task budget 90% reached: $9.02 / $10.00
  Task: implement-auth-module
  Recommendation: Switch remaining work to Haiku model
```

**At 100% threshold**: Log a hard warning. Optionally pause non-critical agents. Never silently exceed budget without notification.

## Cost Optimization Strategies

### 1. Model Routing (Highest Impact)

Route tasks to the cheapest model that can handle them:

| Task Complexity | Recommended Model | Cost Ratio |
|----------------|-------------------|------------|
| Simple transforms (rename, format) | Agent Booster (no LLM) | 0x |
| Bug fixes, simple code gen | Haiku | 1x |
| Feature implementation, refactoring | Sonnet | 12x |
| Architecture, security analysis | Opus | 60x |

**Rule**: Default to Haiku. Escalate only when task fails or complexity score exceeds threshold.

### 2. Prompt Caching

Cache identical or near-identical prompts:
- System prompts: Cache aggressively (same across all calls)
- File contents: Cache when file hasn't changed since last read
- Expected savings: 30-50% on input tokens for repetitive workflows

### 3. Prompt Optimization

- Remove redundant context from prompts
- Use structured output formats (less verbose)
- Batch related questions into single calls instead of multiple round-trips
- Trim file contents to relevant sections instead of sending entire files

### 4. Agent Lifecycle Management

- Terminate idle agents after 5 minutes of inactivity
- Limit max concurrent agents (fewer agents = less parallel spend)
- Reuse agent context instead of spawning new agents for related tasks

### 5. Output Token Control

- Request concise responses for simple tasks
- Use structured output (JSON) instead of prose for machine-consumed results
- Set max_tokens appropriately per task type

## Reporting

### Daily Summary

```
=== Daily Cost Report (2025-03-15) ===
Total: $42.18 / $50.00 (84%)

By Model:
  Haiku:  $3.20  (8%)   | 12.8M tokens
  Sonnet: $28.90 (69%)  | 3.2M tokens
  Opus:   $10.08 (24%)  | 0.2M tokens

By Agent:
  coder:      $18.40
  researcher: $12.10
  tester:     $6.88
  reviewer:   $4.80

By Task:
  auth-module:     $15.20
  api-refactor:    $12.80
  test-coverage:   $8.18
  code-review:     $6.00

Optimization Opportunities:
  - 23 Haiku-eligible calls routed to Sonnet (potential savings: $4.20)
  - 8 duplicate prompts detected (potential cache savings: $2.10)
```

### Weekly Trend

```
Week    | Cost    | Tokens   | Avg $/task | Trend
--------|---------|----------|------------|------
Week 1  | $180.20 | 45.2M   | $6.80      | --
Week 2  | $210.50 | 52.1M   | $7.20      | +17%
Week 3  | $165.30 | 48.8M   | $5.10      | -21%
Week 4  | $142.80 | 51.2M   | $4.40      | -14%
```

## Integration Commands

```bash
# Store cost data in memory
npx @claude-flow/cli@latest memory store \
  --namespace cost-tracking \
  --key "daily-$(date +%Y-%m-%d)" \
  --value '{"total": 42.18, "budget": 50.00}'

# Search cost history
npx @claude-flow/cli@latest memory search \
  --query "cost report" \
  --namespace cost-tracking

# Trigger optimization analysis
npx @claude-flow/cli@latest hooks worker dispatch --trigger optimize
```

## Implementation Checklist

- [ ] Token counting hooked into every LLM call
- [ ] Cost calculated and stored per call
- [ ] Budget thresholds configured
- [ ] Alert channels connected
- [ ] Daily/weekly reports generated
- [ ] Model routing optimized based on cost data
- [ ] Prompt caching enabled where applicable
