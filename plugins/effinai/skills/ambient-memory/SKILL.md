---
name: ambient-memory
description: "Automatic memory consolidation across sessions — graph-based organization with tags, clusters, semantic links, cascade retrieval, and sidecar verification"
version: "1.0.0"
tags: [memory, consolidation, graph, embeddings, learning, ambient]
---

# Ambient Memory

## Activation
When the agent needs to automatically extract, organize, and consolidate learnings across sessions. Use for building persistent knowledge that survives session boundaries, organizing memories into a queryable graph, and maintaining memory health over time.

## Concept

A multi-layered memory system that mimics human memory: relevant memories "pop up" when triggered by context rather than requiring explicit recall. The system is fully async and non-blocking -- results from turn N are available at turn N+1. Memories form a connected graph with explicit tags, automatic clusters, and semantic links. Ported from jcode's memory architecture with graph-based organization, cascade retrieval, and sidecar-verified consolidation.

Key design principles:
1. **Fully async** -- the main agent never blocks waiting for memory operations
2. **Graph-based** -- memories connect via tags, clusters, and semantic relationships
3. **Cascade retrieval** -- embedding hits trigger BFS traversal to surface related memories
4. **Hybrid grouping** -- combines explicit tags, automatic HDBSCAN clusters, and semantic links

## Memory Graph Architecture

### Node Types
| Node | Description | Data |
|------|-------------|------|
| **Memory** | Core entry (fact, preference, procedure, correction) | Content, metadata, embedding vector |
| **Tag** | Explicit label (user-defined or inferred) | Name, description, usage count |
| **Cluster** | Auto-discovered grouping via embedding similarity | Centroid embedding, member count |

### Edge Types
| Edge | From -> To | Description |
|------|-----------|-------------|
| `HasTag` | Memory -> Tag | Memory carries this label |
| `InCluster` | Memory -> Cluster | Memory belongs to auto-discovered group |
| `RelatesTo` | Memory -> Memory | Semantic relationship (weighted 0.0-1.0) |
| `Supersedes` | Memory -> Memory | Newer memory replaces older one |
| `Contradicts` | Memory -> Memory | Conflicting information flagged |
| `DerivedFrom` | Memory -> Memory | Procedural knowledge derived from facts |

## Memory Operations

### Remember (Store)
```
action: "remember"
content: "This project uses JWT with refresh tokens for auth"
category: "fact"           # fact, preference, entity, correction
tags: ["auth", "jwt"]      # Explicit tags
scope: "project"           # project (cwd-specific) or global
```

Categories:
- **fact** -- Technical knowledge, patterns, conventions
- **preference** -- User preferences, style choices
- **entity** -- Named entities (people, services, APIs)
- **correction** -- Error corrections, "actually X not Y"

### Recall (Retrieve)
```
action: "recall"
query: "authentication approach"
scope: "project"
limit: 10
mode: "cascade"            # cascade (graph BFS) or direct (embedding only)
```

Cascade retrieval flow:
1. Embed the query using all-MiniLM-L6-v2
2. Find top-K similar memories by cosine similarity
3. For each hit, BFS-traverse the graph to find connected memories
4. Sidecar (lightweight LLM) verifies relevance of candidates
5. Return ranked, verified results

### Search (Text Match)
```
action: "search"
query: "database migration"
scope: "all"
```

### Link (Create Relationships)
```
action: "link"
from_id: "mem_abc123"
to_id: "mem_def456"
weight: 0.85               # Relationship strength 0.0-1.0
```

### Related (Graph Traversal)
```
action: "related"
id: "mem_abc123"
depth: 2                   # BFS traversal depth (default: 2)
```

### Tag Management
```
action: "tag"
id: "mem_abc123"
tags: ["security", "production"]
```

### Forget
```
action: "forget"
id: "mem_abc123"
```

## Ambient Consolidation Cycle

The ambient agent periodically runs memory maintenance:

### Cycle Steps
1. **Health Check** -- Scan for stale entries, missing embeddings, graph fragmentation
2. **Re-cluster** -- Run HDBSCAN on memory embeddings to discover/update clusters
3. **Link Discovery** -- Find memories with high cosine similarity that lack explicit links
4. **Contradiction Detection** -- Identify memories with conflicting information
5. **Supersession** -- Mark outdated memories when newer versions exist
6. **Compaction** -- Merge redundant memories, prune low-value entries
7. **Tag Inference** -- Infer tags from content, file paths, and entity recognition

### Health Metrics
```
graph_health:
  total_memories: 342
  orphan_memories: 12       # No tags or cluster membership
  stale_entries: 8          # Not accessed in 30+ days
  missing_embeddings: 3     # Need re-embedding
  fragmentation: 0.15       # Graph connectivity score (0=disconnected, 1=fully connected)
  contradictions: 2         # Flagged conflicting memories
```

## Tag Conventions

### Automatic Tags
- `#project:<name>` -- Inferred from working directory
- `#lang:<language>` -- Inferred from file extensions
- `#domain:<area>` -- Inferred from content (auth, database, api, etc.)

### Category Tags
- `#preference` -- User style/behavior preferences
- `#correction` -- Error corrections and clarifications
- `#pattern` -- Reusable code patterns
- `#decision` -- Architecture/design decisions with rationale

## Integration with Ambient Agent

The ambient-memory skill works in tandem with the ambient-agent skill:

1. **Session End** -- Agent extracts key learnings and stores as memories
2. **Between Sessions** -- Ambient agent runs consolidation cycles
3. **Session Start** -- Relevant memories are surfaced based on working directory and recent context
4. **During Session** -- New memories stored async, available next turn

## Performance Characteristics

| Operation | Latency | Notes |
|-----------|---------|-------|
| Remember | <10ms | Async, non-blocking |
| Recall (direct) | ~50ms | Embedding similarity only |
| Recall (cascade) | ~200ms | Embedding + BFS + sidecar verification |
| Consolidation cycle | 5-30s | Runs in background ambient agent |
| Re-clustering | 10-60s | Depends on memory count |
