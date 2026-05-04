---
name: memory-graph
description: "Graph-based memory with tags, clusters, semantic links, and cascade retrieval for cross-session knowledge persistence"
version: "1.0.0"
tags: [memory, graph, knowledge, persistence, embeddings, cross-session]
---

# Memory Graph

## Activation
When the agent needs to store, retrieve, link, or navigate structured knowledge across sessions. Use when simple key-value memory is insufficient and relationships between memories matter.

## Concept

A graph-structured memory system where memories are nodes connected by typed edges. Unlike flat memory stores, the graph enables:

- **Tag-based organization**: Memories have explicit tags that create navigable categories
- **Semantic links**: Weighted edges between related memories (RelatesTo, Supersedes, Reinforces)
- **Cascade retrieval**: BFS traversal through the graph to find transitively related knowledge
- **Embedding search + graph walk**: First find semantically similar memories via embeddings, then walk the graph to discover related context that pure embedding search would miss
- **Deduplication**: Automatic duplicate detection using embedding similarity (threshold 0.85) with cross-store dedup between project and global scopes
- **Reinforcement**: When a duplicate is found, the existing memory is reinforced rather than creating a new entry

## Architecture

### Node Types
| Type | Description |
|------|-------------|
| **Memory** | A knowledge entry with content, category, tags, trust level, embedding |
| **Tag** | An organizational label connecting related memories |
| **Cluster** | Auto-discovered grouping of semantically similar memories |

### Edge Types
| Edge | Description |
|------|-------------|
| **HasTag** | Memory belongs to a tag category |
| **InCluster** | Memory belongs to an auto-discovered cluster |
| **RelatesTo** | Semantic relationship with weight (0.0-1.0) |
| **Supersedes** | Newer memory replaces older one |
| **Reinforces** | Multiple observations strengthen a memory |

### Memory Categories
| Category | Weight | Use Case |
|----------|--------|----------|
| **Correction** | 50 | User corrections to agent behavior |
| **Preference** | 30 | User preferences and style choices |
| **Fact** | 20 | Verified factual information |
| **Entity** | 10 | Named entities (people, projects, APIs) |
| **Custom(name)** | 5 | Domain-specific categories |

### Trust Levels
| Level | Multiplier | Source |
|-------|-----------|--------|
| **High** | 1.5x | User-confirmed, multi-reinforced |
| **Medium** | 1.0x | Default, single-source |
| **Low** | 0.7x | Unverified, single observation |

## Operations

### Store a Memory
```
1. Generate embedding for content
2. Check for duplicates (cosine similarity > 0.85) in both project and global graphs
3. If duplicate found: reinforce existing memory, return its ID
4. If new: add to graph, auto-tag, save
```

### Retrieve Relevant Memories
```
1. Embed the current context (user message + recent conversation)
2. Find candidate memories by embedding similarity (threshold 0.5)
3. Filter out already-injected memories (prevent repetition)
4. Apply gap filter: drop trailing low-relevance results when score gap > 25% of range
5. Optionally verify with sidecar model for precision
6. Format and inject into system prompt
```

### Cascade Retrieval
```
Given seed memories with scores:
1. Initialize frontier with seed IDs and their scores
2. BFS through graph edges (HasTag, RelatesTo, InCluster)
3. Propagate scores through edges with decay (score * edge_weight * 0.7)
4. Return top-k results by accumulated score
5. Depth-limited (default: 2 hops)
```

### Memory Scoring
```python
score = 0
# Recency (decays over time)
score += 100 / (1 + age_hours / 24)
# Access frequency
score += sqrt(access_count) * 10
# Category importance (Correction=50, Preference=30, Fact=20, Entity=10, Custom=5)
score += category_weight
# Trust multiplier (High=1.5, Medium=1.0, Low=0.7)
score *= trust_multiplier
# Consolidation strength
score += ln(strength) * 5
```

## Scopes

| Scope | Storage | Use Case |
|-------|---------|----------|
| **Project** | Per-working-directory hash | Codebase-specific patterns |
| **Global** | User-level | Cross-project preferences |
| **All** | Both scopes merged | Default retrieval scope |

## Workflow

### Session Start
1. Load project and global memory graphs
2. Embed recent conversation context
3. Find and inject relevant memories (max 10)
4. Track which memories were injected to avoid repeats

### During Session
- Agent can explicitly store memories via memory tool
- Memories auto-tagged based on content analysis
- Links created between related memories

### Session End
1. Extract learnings from conversation transcript using sidecar model
2. Store new memories with embeddings
3. Auto-deduplicate against existing memories
4. Update reinforcement counts

## Key Design Principles (from jcode)
- **Embedding-first retrieval**: Local embeddings (~30ms) for fast candidate selection
- **Gap filtering**: Natural score gaps indicate noise boundary, cut there
- **Cross-store dedup**: Check both project and global stores before creating new memories
- **Reinforcement over duplication**: Strengthen existing memories when duplicates found
- **Lazy migration**: Old flat memory stores auto-migrate to graph format on first load
