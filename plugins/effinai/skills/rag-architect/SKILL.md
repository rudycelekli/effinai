---
name: rag-architect
description: "Design, implement, and optimize production-grade RAG pipelines. Covers document chunking, embedding models, vector databases, retrieval strategies, query transformation, evaluation frameworks, and cost optimization."
version: "1.0.0"
tags: [rag, retrieval, embeddings, vector-search, ai, architecture]
createdBy: "built-in"
status: "active"
---

# RAG Architect

## Overview

Comprehensive tools and knowledge for designing, implementing, and optimizing production-grade RAG pipelines. Covers the entire ecosystem from document chunking strategies to evaluation frameworks.

## Activation

When the user asks to design RAG pipelines, optimize retrieval strategies, choose embedding models, implement vector search, or build knowledge retrieval systems.

## 1. Document Chunking Strategies

### Fixed-Size Chunking
- Character or token-based splitting (512, 1024, 2048)
- 10-20% overlap to maintain context continuity
- Best for: uniform documents, consistent chunk sizes needed

### Sentence-Based Chunking
- Boundary detection via NLTK, spaCy, or regex
- Sentence grouping until size threshold
- Best for: narrative text, articles, books

### Semantic Chunking
- Topic modeling via TF-IDF, embedding similarity
- Heading-aware splitting respecting H1/H2/H3 hierarchy
- Best for: long-form content, technical manuals, research papers

### Recursive Chunking
- Hierarchical: try larger chunks first, recursively split if needed
- Best for: mixed content types, chunk count optimization

### Document-Aware Chunking
- File type detection: PDF pages, Word sections, HTML elements
- Metadata preservation: headers, footers, page numbers
- Best for: multi-format collections where metadata matters

## 2. Embedding Model Selection

| Category | Examples | Dimensions | Speed |
|----------|----------|-----------|-------|
| Fast | all-MiniLM-L6-v2 | 384 | ~14k tok/s |
| Balanced | all-mpnet-base-v2 | 768 | ~2.8k tok/s |
| Quality | text-embedding-ada-002 | 1536 | API-bound |
| Code | CodeBERT, GraphCodeBERT | varies | varies |
| Multilingual | LaBSE, multilingual-e5 | varies | varies |

## 3. Vector Database Selection

| Database | Type | Best For |
|----------|------|----------|
| **Pinecone** | Managed | Production, auto-scaling, when managed preferred |
| **Weaviate** | Open source | Complex data types, GraphQL API |
| **Qdrant** | Rust-based | High-performance, resource-constrained |
| **Chroma** | Embedded | Development, testing, small-scale |
| **pgvector** | PostgreSQL ext | Existing Postgres infra, ACID compliance |

## 4. Retrieval Strategies

### Dense Retrieval
Semantic similarity via embedding cosine similarity. Handles paraphrasing well but may miss exact keyword matches.

### Sparse Retrieval
Keyword-based (TF-IDF, BM25). Exact matching, interpretable, but misses semantic similarity.

### Hybrid Retrieval
Dense + sparse with score fusion (Reciprocal Rank Fusion or weighted combination). Best of both worlds.

### Reranking
Two-stage: initial retrieval followed by cross-encoder reranking. Higher precision at cost of additional latency.

## 5. Query Transformation

### HyDE (Hypothetical Document Embeddings)
Generate hypothetical answer, embed that instead of query. Improves retrieval by matching document style.

### Multi-Query Generation
Generate 3-5 query variations, retrieve for each, deduplicate. Increases recall, handles ambiguity.

### Step-Back Prompting
Generate broader version of specific query. Retrieves general context that helps answer specific questions.

## 6. Context Window Optimization

- **Relevance-based ordering**: most relevant chunks first
- **Diversity optimization**: avoid redundant information
- **Token budget management**: fit within model context limits
- **Context compression**: summarize less relevant chunks, extract key facts only

## 7. Evaluation Frameworks

| Metric | Target | What it measures |
|--------|--------|-----------------|
| Faithfulness | >90% | Answers grounded in retrieved context |
| Context relevance | >0.8 | Retrieved chunks relevant to query |
| Answer relevance | >0.85 | Answer addresses original question |
| Precision@K | varies | Top-K results that are relevant |
| MRR | varies | Rank of first relevant result |
| RAGAS | composite | Comprehensive RAG evaluation |

## 8. Production Patterns

### Caching
- Query-level, semantic, chunk-level, multi-level (Redis hot, disk warm)

### Streaming
- Progressive loading, incremental generation, real-time updates

### Fallbacks
- Graceful degradation, cache fallbacks, alternative sources

## 9. Cost Optimization

- **Batch processing** for embeddings to reduce API costs
- **Cache embeddings** to avoid recomputation
- **Only re-embed changed documents**
- **Quantization** to reduce vector storage
- **Query routing**: simple queries to cheaper methods
- **Metadata filters** to reduce search space

## 10. Guardrails & Safety

- **Content filtering**: toxicity, PII detection, content validation
- **Query safety**: injection prevention, rate limiting, access controls
- **Response safety**: hallucination detection, confidence scoring, source attribution

## Common Pitfalls

| Problem | Solution |
|---------|----------|
| Chunks break mid-sentence | Boundary-aware chunking with overlap |
| Low retrieval precision | Better embedding model, add reranking, tune threshold |
| High latency | Optimize indexing, implement caching, faster embeddings |
| Inconsistent quality | Comprehensive evaluation, quality scoring, fallbacks |
| Scalability issues | Caching, database sharding, auto-scaling |
