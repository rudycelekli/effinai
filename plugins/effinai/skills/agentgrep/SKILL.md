---
name: agentgrep
description: "AI-optimized code search with smart DSL, structural outline, and context-aware find — designed for agent workflows not human grep"
version: "1.0.0"
tags: [search, code, grep, codebase, navigation, ai-optimized]
---

# AgentGrep

## Activation
When the agent needs to search, navigate, or understand a codebase with more intelligence than basic grep. Especially useful for tracing code relationships, finding structural patterns, and exploring unfamiliar codebases.

## Concept

A search tool designed specifically for AI agent workflows. Unlike grep which returns raw line matches, agentgrep understands code structure, provides contextual results, and supports a smart DSL for relational queries. Ported from jcode's agentgrep integration.

## Modes

### 1. Smart Mode (Relational Queries)
Find code by relationships, not just text patterns:
```
query: "function that calls authenticate AND returns Promise"
query: "class that implements Logger AND has method flush"
query: "file that imports express AND exports router"
```

Smart DSL operators:
| Operator | Description |
|----------|-------------|
| `AND` | Both terms must appear in the same region |
| `OR` | Either term can appear |
| `NEAR(n)` | Terms within n lines of each other |
| `IN(type)` | Restrict to specific code regions (function, class, etc.) |

### 2. Grep Mode (Fast Text Search)
Traditional text search with agent-optimized output:
```
query: "authenticate"
regex: true
path: "src/"
glob: "*.ts"
```

Output includes:
- File path and line number
- Matching line with surrounding context
- Total match count per file

### 3. Outline Mode (Structural View)
Get the structural skeleton of a file:
```
file: "src/auth/handler.ts"
```

Output:
```
src/auth/handler.ts
  class AuthHandler
    constructor(config: AuthConfig)
    async authenticate(token: string): Promise<User>
    async refresh(refreshToken: string): Promise<TokenPair>
    private validateToken(token: string): boolean
  interface AuthConfig
    secretKey: string
    tokenExpiry: number
  function createAuthHandler(config: AuthConfig): AuthHandler
```

### 4. Find Mode (File Discovery)
Find files by name pattern with smart filtering:
```
query: "auth"
glob: "*.ts"
file_type: "typescript"
```

Respects .gitignore, excludes node_modules/vendor by default.

## Context Awareness

AgentGrep tracks what the agent has already seen (via bash history and previous tool calls) to:
- **Boost novel results**: Prioritize files/regions the agent hasn't visited
- **Tune known files**: Suppress results from files the agent already read
- **Track exposure**: Know which parts of the codebase the agent has explored

## Smart Query Examples

### Find implementations
```
"function authenticate" -> finds all authenticate function definitions
"class.*Repository" -> finds repository pattern classes
```

### Trace dependencies
```
"import.*auth AND export" -> finds modules that re-export auth
"require.*database AND query" -> finds files using database queries
```

### Find patterns
```
"try.*catch AND async" -> finds async error handling
"useState AND useEffect" -> finds React components with both hooks
```

## Output Optimization

Unlike raw grep, agentgrep formats output for agent consumption:
- **Deduplication**: Groups matches by file, not one line per match
- **Context windows**: Shows enough surrounding code to understand the match
- **Truncation**: Limits output to prevent context overflow
- **Relevance ranking**: Most relevant matches first, not just file order

## Integration

```bash
# In agent workflows, prefer agentgrep over grep/rg
agentgrep smart "function that handles authentication"
agentgrep outline src/main.rs
agentgrep find --type typescript "controller"
agentgrep grep --regex "TODO|FIXME|HACK" --glob "*.py"
```
