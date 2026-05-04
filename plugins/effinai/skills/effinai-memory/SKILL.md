---
name: effinai-memory
description: >
  Persistent vector memory with HNSW indexing, TF-IDF embeddings, pattern learning 
  with boost/decay scoring. Use when storing knowledge, searching patterns, or 
  leveraging self-learning capabilities.
argument-hint: "[store|search|list] [--key K] [--value V] [--query Q]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  Read
  mcp__effinai__effinai_memory_store
  mcp__effinai__effinai_memory_search
  mcp__effinai__effinai_memory_list
  mcp__effinai__effinai_memory_delete
  mcp__effinai__effinai_memory_stats
  mcp__effinai__effinai_memory_export
  mcp__effinai__effinai_memory_import
  mcp__effinai__effinai_memory_namespaces
  mcp__effinai__effinai_learn
  mcp__effinai__effinai_pattern_store
  mcp__effinai__effinai_pattern_search
  mcp__effinai__effinai_pattern_boost
  mcp__effinai__effinai_pattern_decay
  mcp__effinai__effinai_learning_stats
metadata:
  priority: 7
  promptSignals:
    phrases:
      - "remember"
      - "store in memory"
      - "search memory"
      - "pattern"
      - "learn"
    anyOf:
      - "memory"
      - "remember"
      - "pattern"
      - "learn"
    minScore: 4
---

# Effin.AI Memory & Self-Learning

## How It Works

```
New Task --> Search Patterns --> Inject Context --> Execute --> Quality Gate
    ^                                                             |
    +-- Store Pattern (boost +20%) <------------------------------+
    +-- Decay Pattern (-10%) <-- Curator lifecycle review
```

## Memory Operations

```bash
effin memory store --key "auth-pattern" --value "JWT with refresh tokens" --namespace patterns
effin memory search --query "authentication" --namespace patterns
effin memory list --namespace patterns
effin memory stats
```

## Self-Learning

- **Pattern Boost**: +20% score on success
- **Pattern Decay**: -10% on staleness
- After 3+ successful matches, context auto-injected into future tasks
- TF-IDF embeddings for semantic similarity
- HNSW index for fast vector search

## MCP Tools (14)

### Memory
- `effinai_memory_store` — Store key-value with tags and namespace
- `effinai_memory_search` — Semantic vector search (HNSW)
- `effinai_memory_list` — List entries by namespace
- `effinai_memory_delete` — Delete entry
- `effinai_memory_stats` — Statistics
- `effinai_memory_export/import` — Backup/restore

### Learning
- `effinai_learn` — Store successful pattern
- `effinai_pattern_store` — Store with verdict (success/failure)
- `effinai_pattern_search` — Search by similarity
- `effinai_pattern_boost` — Boost successful pattern (+20%)
- `effinai_pattern_decay` — Decay stale patterns (-10%)
- `effinai_learning_stats` — Learning metrics
