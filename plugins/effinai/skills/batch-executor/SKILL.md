---
name: batch-executor
description: "Parallel execution of multiple independent tool calls in a single request for maximum throughput"
version: "1.0.0"
tags: [parallel, batch, performance, tools, optimization]
---

# Batch Executor

## Activation
When the agent needs to execute multiple independent tool calls simultaneously, such as reading multiple files, running multiple searches, or performing parallel operations.

## Concept

Execute up to 10 independent tool calls in parallel within a single request. Instead of sequential tool calls (each requiring a round-trip), batch them together for significant latency reduction. Ported from jcode's batch tool.

## Usage

### Batch Request Format
```json
{
  "tool_calls": [
    {"tool": "read", "parameters": {"file": "src/main.rs"}},
    {"tool": "read", "parameters": {"file": "src/lib.rs"}},
    {"tool": "grep", "parameters": {"pattern": "TODO", "path": "."}},
    {"tool": "bash", "parameters": {"command": "git status"}}
  ]
}
```

### Constraints
| Parameter | Value |
|-----------|-------|
| Max parallel calls | 10 |
| Min calls | 1 |
| Tool name aliases | Supported (file_read -> read, shell_exec -> bash) |

## When to Batch

### Good Candidates (Independent Operations)
- Reading multiple files simultaneously
- Running multiple grep searches
- Checking multiple file existences
- Running independent bash commands
- Multiple API calls

### Bad Candidates (Dependencies)
- Reading a file, then editing based on contents
- Running a command, then checking its output
- Any operation where result B depends on result A

## Progress Tracking

Each subcall has a tracked state:
| State | Description |
|-------|-------------|
| **Running** | Currently executing |
| **Succeeded** | Completed successfully |
| **Failed** | Completed with error |

Progress is reported as subcalls complete:
```
[1/4] read src/main.rs ......... done
[2/4] read src/lib.rs .......... done  
[3/4] grep TODO ................ done
[4/4] git status ............... running
```

## Results Format

Results returned in order, with each subcall's output:
```
## Tool 1: read (src/main.rs)
[file contents]

## Tool 2: read (src/lib.rs) 
[file contents]

## Tool 3: grep (TODO)
[grep results]

## Tool 4: bash (git status)
[git output]
```

Failed subcalls include error messages but don't prevent other subcalls from completing.

## Performance Impact

### Sequential (4 file reads)
```
Turn 1: read file A        ~2s round-trip
Turn 2: read file B        ~2s round-trip  
Turn 3: read file C        ~2s round-trip
Turn 4: read file D        ~2s round-trip
Total: ~8s + 4 API round-trips
```

### Batched (4 file reads)
```
Turn 1: batch [A, B, C, D]  ~2s (parallel)
Total: ~2s + 1 API round-trip
```

**4x faster, 75% fewer API calls**

## Best Practices

1. **Always batch independent reads**: When you need to read multiple files, batch them
2. **Batch search + read**: Search for files, then batch-read the results
3. **Don't over-batch**: Limit to operations you actually need right now
4. **Handle partial failures**: Some subcalls may fail; process successful results and retry failures individually
5. **Watch output size**: 10 large file reads may exceed context limits; use the context compaction skill
