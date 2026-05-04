---
name: hybrid-search
description: "Hybrid search combining sparse (BM25) + dense (vector) retrieval with reciprocal rank fusion, SPLADE support, and multiple normalization strategies"
version: "1.0.0"
tags: [search, hybrid, bm25, vector, fusion, retrieval]
---

# Hybrid Search

## Activation
When the agent needs retrieval that handles both exact keyword matches and semantic similarity. Use when pure vector search misses exact terms (e.g., error codes, API names, configuration keys) or when pure keyword search misses conceptually related results.

## Concept

Combines two complementary retrieval strategies: BM25 (sparse, keyword-based) which excels at exact term matching, and vector similarity (dense, embedding-based) which captures semantic meaning. Results from both are normalized and fused using configurable strategies. Ported from RuVector's `HybridSearch` and `SparseIndex` implementations which include full BM25 with IDF scoring, SPLADE-compatible sparse vectors, and three fusion strategies (RRF, Linear Combination, DBSF).

## Architecture

```
Query
  |
  +---> BM25 Index -----> Sparse Scores ----+
  |     (keyword match)                      |
  |                                          +--> Fusion --> Final Ranking
  |                                          |
  +---> Vector Index ---> Dense Scores  -----+
        (embedding)
```

## Components

### BM25 (Sparse Retrieval)

Standard BM25 with IDF weighting for keyword matching:

```
Parameters:
  k1: 1.2    # Term frequency saturation (higher = more weight to repeated terms)
  b:  0.75   # Length normalization (0 = no normalization, 1 = full normalization)
```

**Scoring formula:**
```
score(q, d) = sum over terms t in q:
  IDF(t) * (tf(t,d) * (k1 + 1)) / (tf(t,d) + k1 * (1 - b + b * |d| / avgdl))

IDF(t) = ln((N - df(t) + 0.5) / (df(t) + 0.5) + 1)
```

**Indexing:**
```
bm25.index_document(doc_id, text)    # Add document to inverted index
bm25.build_idf()                      # Compute IDF scores (call after all docs indexed)
bm25.score(query, doc_id, doc_text)   # Score a specific document
bm25.get_candidate_docs(query)        # Get all docs containing query terms
```

### Sparse Vectors (SPLADE-Compatible)

For learned sparse representations (SPLADE, SPLADE++, etc.):

```
SparseVector:
  indices: [14, 89, 203, 1567]     # Non-zero dimension indices (sorted)
  values:  [0.8, 1.2, 0.3, 0.95]  # Corresponding weights

Operations:
  dot(a, b)    # Merge-intersection dot product, O(|a| + |b|)
  l2_norm()    # Euclidean norm for normalization
  nnz()        # Number of non-zero entries
```

**Sparse Index:**
- Inverted index with posting lists for sub-linear search
- Batch insert and search operations
- WASM-compatible (no system dependencies)

### Vector Search (Dense Retrieval)

Uses HNSW-indexed embeddings for semantic similarity:
- Cosine similarity (converted from distance: similarity = 1 - distance)
- Supports any embedding model (all-MiniLM-L6-v2, text-embedding-3-small, etc.)
- O(log n) search complexity

## Normalization Strategies

Before fusing scores from different retrievers, normalize them to a common scale:

| Strategy | Formula | Best For |
|----------|---------|----------|
| **MinMax** | `(x - min) / (max - min)` | When score distributions are roughly uniform |
| **ZScore** | `(x - mean) / std` | When score distributions are Gaussian |
| **None** | Raw scores | When scores are already on comparable scales |

## Fusion Strategies

### Reciprocal Rank Fusion (RRF)
```
RRF_score(d) = sum over rankings r:
  1 / (k + rank_r(d))

k = 60 (default constant to reduce impact of high-rank outliers)
```

Best for: Combining rankings from very different systems. Robust, parameter-free (other than k).

### Linear Combination
```
combined_score(d) = alpha * vector_score(d) + beta * keyword_score(d)

Default: alpha=0.7, beta=0.3
```

Best for: When you want direct control over the balance between semantic and lexical matching.

### Distribution-Based Score Fusion (DBSF)
```
Uses the score distributions from each retriever to determine fusion weights dynamically.
Retrievers with tighter score distributions get higher weight.
```

Best for: When score distributions vary significantly across queries.

## Configuration

```json
{
  "vector_weight": 0.7,
  "keyword_weight": 0.3,
  "normalization": "MinMax",
  "fusion_strategy": "RRF",
  "fusion_config": {
    "rrf_k": 60,
    "top_k": 20
  }
}
```

## Usage Patterns

### Code Search
```
Query: "handleAuthError function"

BM25 finds:  exact matches for "handleAuthError" (rank 1-3 are perfect)
Vector finds: semantically similar error handlers (rank 1 might be "processAuthFailure")

Hybrid fuses: exact match at rank 1, semantic alternatives at rank 2-5
```

### Documentation Search
```
Query: "how to configure rate limiting"

BM25 finds:  docs mentioning "rate limiting" literally
Vector finds: docs about "throttling", "request quotas", "API limits"

Hybrid fuses: comprehensive results covering all terminology
```

### Error Investigation
```
Query: "ECONNREFUSED port 5432"

BM25 finds:  exact error code matches (critical for debugging)
Vector finds: conceptually related connection issues

Hybrid fuses: exact error first, related solutions following
```

## Performance Characteristics

| Operation | Complexity | Notes |
|-----------|-----------|-------|
| BM25 index | O(n * avg_doc_len) | One-time indexing |
| BM25 query | O(q * postings) | Sublinear via inverted index |
| IDF build | O(vocabulary) | After all docs indexed |
| Sparse dot product | O(min(a, b)) | Merge-intersection on sorted indices |
| Vector search | O(log n) | HNSW approximate |
| Fusion (RRF) | O(k * num_retrievers) | Fast rank-based merge |
| Fusion (Linear) | O(k) | Direct score combination |

## Integration with Other Skills

- **Graph RAG**: Hybrid search over entity descriptions + embeddings
- **Agent Grep**: BM25 for exact code patterns, vectors for semantic code search
- **Ambient Memory**: Hybrid retrieval over memory entries (keyword + semantic)
- **Knowledge Graph**: Search entities by name (BM25) or description (vector)
