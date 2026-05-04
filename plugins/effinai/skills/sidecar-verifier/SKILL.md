---
name: sidecar-verifier
description: "Lightweight secondary model for fast verification tasks — memory relevance, content classification, and quality checks"
version: "1.0.0"
tags: [verification, sidecar, cost-optimization, quality, memory]
---

# Sidecar Verifier

## Activation
When the agent needs fast, cheap verification of content relevance, classification, or quality checks without consuming the primary model's context window or budget.

## Concept

A lightweight secondary model (sidecar) that handles quick verification tasks. The sidecar runs in parallel with the main agent, using a smaller/cheaper model for tasks that don't require full reasoning capability. Ported from jcode's Sidecar system.

## Architecture

```
Main Agent (Opus/Sonnet)          Sidecar (Haiku/GPT-4-mini)
  |                                  |
  |-- "Is this memory relevant?" --> |
  |                                  |-- Quick check (~500ms)
  |<-- "Yes, because..." ------------|
  |                                  |
  |-- "Classify this content" -----> |
  |                                  |-- Fast classification
  |<-- "category: bug_fix" ----------|
```

## Use Cases

### Memory Relevance Verification
After embedding search returns candidate memories:
```
For each candidate (parallel, batch of 5):
  Ask sidecar: "Is this memory relevant to the current context?"
  Input: memory content + current conversation context
  Output: (is_relevant: bool, reason: string)
```

Benefits:
- Embedding search is fast but imprecise (finds ~30 candidates)
- Sidecar filters to truly relevant ones (~5-10 memories)
- Main model never sees irrelevant memories (saves context tokens)

### Memory Extraction
After a session ends, extract learnings:
```
Input: Session transcript
Output: List of memories with:
  - content: what was learned
  - category: correction | preference | fact | entity
  - trust: high | medium | low
```

### Content Classification
Quick classification without full reasoning:
```
Input: Code diff or message
Output: Category (bug_fix, feature, refactor, docs, test)
```

### Quality Gate
Before injecting content into the main agent:
```
Input: Proposed injection content
Output: (quality_score: 0-1, should_inject: bool, reason: string)
```

## Model Selection

Auto-selects the best available backend:

| Priority | Provider | Model | Cost | Latency |
|----------|----------|-------|------|---------|
| 1 | Anthropic | claude-haiku-4-5 | $0.0002/call | ~500ms |
| 2 | OpenAI | gpt-4o-mini | $0.0001/call | ~400ms |
| 3 | Local | Small local model | Free | ~200ms |

## Parallel Verification Pipeline

```
Step 1: Embedding Search (local, ~30ms)
  -> 10-30 candidate memories

Step 2: Sidecar Verification (parallel batches of 5, ~500ms per batch)
  -> 3-8 verified relevant memories

Step 3: Format and inject into main agent context
  -> Only high-quality, verified context
```

## Configuration

```yaml
sidecar:
  enabled: true
  model: "claude-haiku-4-5"    # or auto-detect
  max_tokens: 1024             # keep responses short
  timeout_ms: 5000             # fail fast
  batch_size: 5                # parallel verification batch
  fallback: "embedding_only"   # if sidecar unavailable
```

## Cost Analysis

Without sidecar:
- All 30 candidate memories injected into main context
- Main model processes irrelevant context
- ~10k extra tokens per turn at primary model rate

With sidecar:
- 30 sidecar calls at ~$0.0002 each = $0.006
- Only 5-8 relevant memories injected
- Saves ~7k tokens at primary model rate
- Net savings: 60-80% on memory-related context costs

## Integration with Effin.AI

The sidecar pattern applies to any verification task:
```bash
# Route simple verification to cheapest capable model
effin route --task "classify this code change" --prefer-cheap
```

The routing system can automatically delegate verification tasks to the sidecar tier, keeping the primary model focused on reasoning-heavy work.
