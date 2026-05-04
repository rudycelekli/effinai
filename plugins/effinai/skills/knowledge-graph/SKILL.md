---
name: knowledge-graph
description: "Build and query knowledge graphs from codebases — entity extraction, relationship mapping, dependency analysis, and architectural visualization"
version: "1.0.0"
tags: [knowledge-graph, codebase, entities, dependencies, architecture, analysis]
---

# Knowledge Graph

## Activation
When the agent needs to understand codebase architecture, map dependencies between modules, trace how components interact, or answer structural questions like "what depends on this module?" or "what's the blast radius of changing this interface?"

## Concept

Builds a property graph from codebase analysis where nodes represent code entities (files, classes, functions, modules, APIs) and edges represent relationships (imports, calls, extends, implements). Supports Cypher-style queries, adjacency indexing for fast neighbor lookups, and hyperedges for N-ary relationships. The graph enables architectural reasoning that file-by-file analysis cannot achieve.

## Graph Construction

### Entity Extraction
Scan the codebase and extract entities:

| Entity Type | Examples | Extraction Method |
|-------------|----------|-------------------|
| **File** | `src/auth/middleware.ts` | File system scan |
| **Module** | `@app/auth` | Package/module declarations |
| **Class** | `AuthService` | AST parsing |
| **Function** | `validateToken()` | AST parsing |
| **Interface** | `IAuthProvider` | AST/type declarations |
| **API Endpoint** | `POST /api/login` | Route declarations |
| **Database Table** | `users` | Schema files, ORM models |
| **Config** | `JWT_SECRET` | Environment/config files |
| **Package** | `express@4.18` | Package manifests |

### Relationship Discovery
Detect edges between entities:

| Relationship | Source -> Target | Detection |
|-------------|-----------------|-----------|
| `IMPORTS` | File -> File/Module | Import/require statements |
| `CALLS` | Function -> Function | Call expressions in AST |
| `EXTENDS` | Class -> Class | Inheritance declarations |
| `IMPLEMENTS` | Class -> Interface | Implementation declarations |
| `EXPORTS` | File -> Function/Class | Export statements |
| `DEPENDS_ON` | Module -> Package | Package manifest |
| `READS` | Function -> Database Table | Query analysis |
| `WRITES` | Function -> Database Table | Mutation analysis |
| `SERVES` | Function -> API Endpoint | Route handler registration |
| `CONTAINS` | File/Module -> Entity | Nesting/ownership |

### Property Assignment
Each node and edge carries properties:

```
Node properties:
  - name: "AuthService"
  - kind: "class"
  - file_path: "src/auth/service.ts"
  - line_range: [15, 142]
  - complexity: 23
  - last_modified: "2026-04-15"
  - test_coverage: 0.85

Edge properties:
  - weight: 0.9 (frequency/importance)
  - call_count: 47
  - is_direct: true
  - context: "Called in request handler"
```

## Query Language

### Cypher-Style Queries
```cypher
-- Find all classes that depend on AuthService
MATCH (c:Class)-[:IMPORTS|CALLS]->(a:Class {name: "AuthService"})
RETURN c.name, c.file_path

-- Find the dependency chain from API to database
MATCH path = (api:Endpoint)-[:SERVES]-(handler:Function)-[:CALLS*1..5]-(db:Table)
RETURN path

-- Find circular dependencies
MATCH (a:Module)-[:IMPORTS]->(b:Module)-[:IMPORTS]->(a)
RETURN a.name, b.name

-- Find most-depended-on modules
MATCH (dep:Module)<-[:DEPENDS_ON]-(consumer:Module)
RETURN dep.name, count(consumer) as dependency_count
ORDER BY dependency_count DESC
LIMIT 10
```

### Programmatic Queries
```
# Neighbors
graph.get_neighbors(node_id, direction="outgoing", edge_type="IMPORTS")

# Shortest path
graph.shortest_path(from_id, to_id)

# Subgraph extraction
graph.subgraph(center_node, depth=2)

# Label-based lookup
graph.find_by_label("Class")

# Property-based lookup
graph.find_by_property("complexity", ">", 50)
```

## Analysis Capabilities

### Dependency Analysis
```
Input: A module or class name
Output:
  - Direct dependencies (imports)
  - Transitive dependencies (full chain)
  - Reverse dependencies (who depends on this)
  - Circular dependencies (if any)
  - Dependency depth (longest chain)
```

### Blast Radius
```
Input: A file or function to be changed
Output:
  - Direct consumers (first-order impact)
  - Transitive consumers (N-hop impact)
  - Affected test files
  - Affected API endpoints
  - Risk score (based on consumer count and criticality)
```

### Architecture Visualization
```
Output: Mermaid diagram of the dependency graph

graph TD
    AuthModule --> UserModel
    AuthModule --> JWTUtil
    AuthModule --> SessionStore
    APIGateway --> AuthModule
    APIGateway --> RateLimiter
    BillingModule --> UserModel
```

### Coupling Analysis
```
Output:
  - Afferent coupling (Ca): incoming dependencies
  - Efferent coupling (Ce): outgoing dependencies
  - Instability (I): Ce / (Ca + Ce)
  - Abstractness (A): abstract types / total types
  - Distance from main sequence: |A + I - 1|
```

### Dead Code Detection
```
Find entities with zero incoming edges (never used):
  - Exported functions never imported
  - Classes never instantiated
  - API endpoints never called
  - Config variables never read
```

## Indexing

### Adjacency Index
Lock-free concurrent adjacency lists for O(1) neighbor lookups.

### Label Index
Fast lookup of all nodes with a given label (e.g., all "Class" nodes).

### Property Index
B-tree index on node/edge properties for range queries.

### Edge Type Index
Fast lookup of all edges of a given type (e.g., all "IMPORTS" edges).

## Hyperedges

For N-ary relationships that connect more than two entities:

```
Example: A function call that involves caller, callee, and a shared resource
Hyperedge {
  id: "he_1",
  nodes: ["fn_authenticate", "fn_getUser", "table_users"],
  relation_type: "AUTH_FLOW",
  properties: { "description": "Authentication reads user from DB" }
}
```

## Integration

- **Graph RAG**: The knowledge graph is the foundation for graph-based retrieval
- **Agent Grep**: Search results feed entity discovery
- **Blast Radius**: Used by PR review and refactoring skills to assess change impact
- **Architecture Review**: Provides structural data for architectural analysis
