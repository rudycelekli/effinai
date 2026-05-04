---
name: agent-grep
description: "Intelligent codebase search with smart DSL, confidence-scored file tracking, structural inference, and context-aware result ranking"
version: "1.0.0"
tags: [search, code, intelligence, structural, inference, context]
---

# Agent Grep (Advanced)

## Activation
When the agent needs deep codebase understanding beyond basic text search: tracing call chains, understanding file relationships, scoring confidence in file knowledge, or building a mental model of unfamiliar code. This extends the basic agentgrep skill with jcode's harness context and confidence tracking system.

## Concept

Agent Grep maintains a live "harness context" that tracks what the agent knows about the codebase with explicit confidence scores. Every file read, search result, and tool observation updates this context. When searching, the system uses this accumulated knowledge to score results, prune irrelevant hits, and prioritize regions the agent hasn't explored yet. Ported from jcode's `AgentGrepHarnessContext` system.

## Harness Context

The agent maintains a running context of codebase knowledge:

### Known Regions
```json
{
  "path": "src/auth/middleware.ts",
  "start_line": 45,
  "end_line": 92,
  "body_confidence": 0.95,
  "current_version_confidence": 0.80,
  "prune_confidence": 0.10,
  "source_strength": "read",
  "reasons": ["read_file", "edit_target"]
}
```

### Known Files
```json
{
  "path": "src/auth/middleware.ts",
  "structure_confidence": 0.90,
  "body_confidence": 0.70,
  "current_version_confidence": 0.85,
  "prune_confidence": 0.15,
  "source_strength": "outline",
  "reasons": ["outline_viewed", "grep_hit"]
}
```

### Known Symbols
```json
{
  "path": "src/auth/middleware.ts",
  "symbol": "authenticateRequest",
  "kind": "function",
  "structure_confidence": 0.95,
  "body_confidence": 0.80,
  "current_version_confidence": 0.85,
  "prune_confidence": 0.10,
  "source_strength": "read",
  "reasons": ["smart_hit", "trace_target"]
}
```

### Confidence Scores Explained

| Score | Meaning | Range |
|-------|---------|-------|
| `body_confidence` | How well the agent knows the actual code content | 0.0-1.0 |
| `structure_confidence` | Knowledge of file structure (exports, classes, functions) | 0.0-1.0 |
| `current_version_confidence` | Likelihood the agent's knowledge is current (decays over time/edits) | 0.0-1.0 |
| `prune_confidence` | Confidence this region can be safely ignored | 0.0-1.0 |

### Source Strength Hierarchy
| Source | Strength | How Acquired |
|--------|----------|-------------|
| `read` | Strongest | Agent read the actual file content |
| `edit` | Strong | Agent edited the file |
| `outline` | Medium | Agent viewed structural outline |
| `grep_hit` | Medium | File appeared in search results |
| `smart_hit` | Medium | File appeared in smart DSL results |
| `trace` | Weak | File referenced in traces/imports |
| `inferred` | Weakest | Inferred from naming conventions |

## Context-Aware Search

### Smart Mode with Context
```
mode: "smart"
query: "function that handles authentication AND modifies user session"
max_files: 10
max_regions: 5
full_region: "auto"        # auto, always, never -- controls full code inclusion
debug_plan: true           # Show the query plan
debug_score: true          # Show scoring breakdown
```

The smart engine:
1. Parses the DSL query into structured terms and relations
2. Checks harness context for already-known files matching the pattern
3. Searches unvisited files, boosting results in areas adjacent to known code
4. Scores results combining text relevance + structural proximity + novelty

### Trace Exposure
The system tracks "trace exposure" -- observations from tool calls that reveal codebase structure without explicit search:
- `Bash("grep -r 'import' src/")` outputs reveal file relationships
- `Read("src/index.ts")` reveals imports and exports
- `Edit("src/auth.ts")` confirms file structure and current content

These observations automatically update the harness context with appropriate confidence scores.

### Find Mode with Structure Inference
```
mode: "find"
glob: "*.test.ts"
path: "src/"
max_files: 50
paths_only: true
```

Find mode builds a file tree and infers project structure:
- Test files near source files suggest colocated test patterns
- `index.ts` files suggest barrel exports
- Directory naming conventions suggest architectural patterns

### Outline with Confidence
```
mode: "outline"
file: "src/auth/middleware.ts"
```

Returns the structural skeleton with confidence annotations:
```
src/auth/middleware.ts [structure: 0.95, body: 0.70]
  export function authenticateRequest [body: 0.95] -- READ
  export function refreshToken [body: 0.30] -- OUTLINE ONLY
  class AuthProvider [body: 0.10] -- INFERRED
    method validate [body: 0.0] -- UNKNOWN
```

## Workflow: Exploring Unfamiliar Code

1. **Outline the entry point** -- Get structure of main files
2. **Smart search for the target concept** -- Use DSL to find relevant code
3. **Read high-confidence hits** -- Examine the most promising results
4. **Trace dependencies** -- Follow imports and calls
5. **Review confidence gaps** -- Identify what's still unknown
6. **Targeted reads** -- Fill in low-confidence regions that matter

The harness context ensures the agent doesn't waste time re-reading files it already knows, and prioritizes exploring unknown areas.

## Integration with Memory

Agent Grep's harness context is ephemeral (per-session), but key findings can be persisted:
- Store discovered patterns in ambient-memory for cross-session recall
- Known symbols and their relationships feed into the knowledge graph
- File structure inferences become project-level memories
