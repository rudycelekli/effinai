---
name: graph-rag
description: "Knowledge graph traversal with entity relationships for retrieval-augmented generation — local, global, and hybrid search with community detection"
version: "1.0.0"
tags: [rag, knowledge-graph, retrieval, entities, communities, search]
---

# Graph RAG

## Activation
When the agent needs to answer complex multi-hop queries, synthesize information across multiple documents, or understand relational context that simple chunk-based RAG cannot capture. Use for questions like "How do all the authentication components relate?" or "What's the full dependency chain for this feature?"

## Concept

Graph RAG builds a knowledge graph of entities and relationships from source documents, then detects communities of related entities at multiple granularity levels. This achieves 30-60% improvement on complex multi-hop queries compared to naive chunk-based RAG because it leverages structural relationships, not just embedding similarity. Ported from RuVector's `GraphRAGPipeline` which implements a Leiden-inspired community detection algorithm with local + global + hybrid search modes.

## Architecture

```
Documents -> Entity Extraction -> KnowledgeGraph
                                      |
                             CommunityDetection (Leiden-inspired)
                                      |
                          Level 0 (fine) + Level 1 (coarse)
                                      |
                             GraphRAGPipeline
                            /        |        \
                       Local      Global     Hybrid
```

## Data Model

### Entities
```json
{
  "id": "entity_auth_service",
  "name": "AuthService",
  "entity_type": "Service",
  "description": "Handles JWT token creation, validation, and refresh",
  "embedding": [0.12, -0.34, ...]
}
```

Entity types: `Person`, `Organization`, `Service`, `Module`, `Concept`, `Technology`, `File`, `Function`, `API`

### Relations
```json
{
  "source_id": "entity_auth_service",
  "target_id": "entity_user_model",
  "relation_type": "DEPENDS_ON",
  "weight": 0.9,
  "description": "AuthService queries UserModel for credential validation"
}
```

Relation types: `DEPENDS_ON`, `CALLS`, `IMPLEMENTS`, `EXTENDS`, `IMPORTS`, `AUTHORED_BY`, `RELATED_TO`, `PART_OF`, `USES`

### Communities
```json
{
  "id": "community_auth",
  "entities": ["entity_auth_service", "entity_user_model", "entity_jwt_util", "entity_session_store"],
  "summary": "Authentication subsystem: JWT-based auth with session management, user credential validation, and token refresh flows",
  "level": 0
}
```

## Pipeline Configuration

```json
{
  "max_hops": 2,
  "community_resolution": 1.0,
  "local_weight": 0.6,
  "global_weight": 0.4,
  "max_context_entities": 20,
  "max_community_summaries": 5
}
```

| Parameter | Default | Description |
|-----------|---------|-------------|
| `max_hops` | 2 | Maximum graph traversal depth for local search |
| `community_resolution` | 1.0 | Higher = more/smaller communities |
| `local_weight` | 0.6 | Weight of local results in hybrid mode |
| `global_weight` | 0.4 | Weight of global results in hybrid mode |
| `max_context_entities` | 20 | Max entities in retrieval context |
| `max_community_summaries` | 5 | Max community summaries for global context |

## Search Modes

### Local Search
```
Starts from a seed entity, traverses N hops through the graph, collects
all connected entities and their relationships.

Best for: "How does AuthService handle token refresh?"
-> Finds AuthService, follows edges to JWTUtil, SessionStore, UserModel
-> Returns the local subgraph with descriptions
```

**Algorithm:**
1. Find seed entities matching the query (by name or embedding)
2. BFS traversal up to `max_hops` depth
3. Collect all visited entities and their connecting relations
4. Rank by structural proximity (closer = more relevant)

### Global Search
```
Uses pre-computed community summaries to answer broad questions
without traversing the graph at query time.

Best for: "What are the main architectural components?"
-> Returns top-K community summaries ranked by relevance to the query
```

**Algorithm:**
1. Embed the query
2. Compare against all community summary embeddings
3. Return top-K most relevant community summaries
4. Include member entity names for drill-down

### Hybrid Search
```
Combines local structural traversal with global community context.

Best for: "How does the auth system relate to the billing module?"
-> Local: traverses from auth entities
-> Global: finds communities spanning auth + billing
-> Merges with configurable weights (default: 60% local, 40% global)
```

## Building the Knowledge Graph

### Step 1: Entity Extraction
```
For each document/code file:
1. Extract named entities (classes, functions, services, APIs, people)
2. Classify entity types
3. Generate descriptions
4. Compute embedding vectors
```

### Step 2: Relation Discovery
```
For each pair of entities in the same document:
1. Detect relationships (imports, calls, extends, depends)
2. Assign relation types and weights
3. Generate relation descriptions

For entities across documents:
1. Cross-reference by name/type
2. Detect indirect relationships (shared dependencies, similar descriptions)
```

### Step 3: Community Detection
```
Run Leiden-inspired algorithm on the entity graph:
1. Initialize: each entity is its own community
2. Move entities to neighboring communities if modularity improves
3. Aggregate communities and repeat
4. Produce hierarchy: Level 0 (fine-grained) -> Level 1 (coarse)
5. Generate natural-language summary for each community
```

## Retrieval Result Format

```json
{
  "entities": [
    {"id": "...", "name": "AuthService", "entity_type": "Service", "description": "..."}
  ],
  "relations": [
    {"source_id": "...", "target_id": "...", "relation_type": "DEPENDS_ON", "description": "..."}
  ],
  "community_summaries": [
    "Authentication subsystem: JWT-based auth with session management..."
  ],
  "context_text": "## Relevant Entities\n- AuthService (Service): Handles JWT...\n\n## Relationships\n- AuthService DEPENDS_ON UserModel: ...\n\n## Broader Context\n- Authentication subsystem: ..."
}
```

The `context_text` field is pre-formatted for direct injection into LLM prompts.

## Integration with Other Skills

- **Hybrid Search**: Graph RAG entities can be indexed in the hybrid search system for BM25 + vector retrieval
- **Ambient Memory**: Knowledge graph entities map to memory graph nodes
- **Agent Grep**: Structural code search feeds entity extraction
- **Session Resume**: Knowledge graph persists across sessions
