---
name: effinai-codebase-intel
description: >
  Codebase intelligence via knowledge graph — index any repo into a queryable graph 
  of symbols, dependencies, call chains, and clusters. Hybrid search (BM25 + semantic), 
  impact analysis, architecture docs, and self-awareness. Use when exploring code, 
  analyzing impact, or understanding architecture.
argument-hint: "[analyze|query|search|impact|self-analyze]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(npx gitnexus*)
  Bash(effin *)
  Read
  Glob
  Grep
  mcp__effinai__effinai_nexus_analyze
  mcp__effinai__effinai_nexus_query
  mcp__effinai__effinai_nexus_search
  mcp__effinai__effinai_nexus_context
  mcp__effinai__effinai_nexus_impact
  mcp__effinai__effinai_nexus_dependencies
  mcp__effinai__effinai_nexus_clusters
  mcp__effinai__effinai_nexus_callgraph
  mcp__effinai__effinai_nexus_rename
  mcp__effinai__effinai_nexus_changes
  mcp__effinai__effinai_nexus_architecture
  mcp__effinai__effinai_nexus_wiki
  mcp__effinai__effinai_nexus_stats
  mcp__effinai__effinai_nexus_serve
  mcp__effinai__effinai_nexus_self_analyze
  mcp__effinai__effinai_nexus_explain
metadata:
  priority: 7
  promptSignals:
    phrases:
      - "understand codebase"
      - "knowledge graph"
      - "impact analysis"
      - "call graph"
      - "dependencies"
      - "architecture"
      - "what would break"
    anyOf:
      - "knowledge graph"
      - "impact"
      - "call graph"
      - "codebase"
      - "architecture"
    minScore: 4
  pathPatterns:
    - '**/*.ts'
    - '**/*.js'
    - '**/*.py'
chainTo:
  - pattern: 'import.*from'
    targetSkill: effinai-agents
    message: "Complex imports detected — consider spawning agents for analysis"
    skipIfFileContains: 'test'
---

# Effin.AI Codebase Intelligence

## What It Provides

- **Knowledge Graph** — Every symbol, dependency, and call chain indexed
- **Hybrid Search** — BM25 keyword + semantic embeddings + reciprocal rank fusion
- **Impact Analysis** — Know exactly what breaks before you change anything
- **Cluster Detection** — Leiden algorithm identifies natural module boundaries
- **Architecture Docs** — Auto-generated Mermaid diagrams from the graph
- **Self-Awareness** — Effin.AI can explain its own capabilities and code paths
- **14 Languages** — TypeScript, JavaScript, Python, Java, Go, Rust, C/C++, and more
- **Web UI** — Browser-based WebGL graph visualization

## Usage

```bash
# Index codebase
npx gitnexus analyze

# Query
# effinai_nexus_query "find all functions that call auth middleware"
# effinai_nexus_impact "src/core/swarm-coordinator.ts"
# effinai_nexus_clusters --format mermaid

# Self-analysis
# effinai_nexus_self_analyze --aspect full
# effinai_nexus_explain --capability "anti-drift"

# Web UI
npx gitnexus serve
```

## 16 MCP Tools
`nexus_analyze`, `nexus_query`, `nexus_search`, `nexus_context`, `nexus_impact`, `nexus_dependencies`, `nexus_clusters`, `nexus_callgraph`, `nexus_rename`, `nexus_changes`, `nexus_architecture`, `nexus_wiki`, `nexus_stats`, `nexus_serve`, `nexus_self_analyze`, `nexus_explain`
