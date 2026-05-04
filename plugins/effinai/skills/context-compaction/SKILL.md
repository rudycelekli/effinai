---
name: context-compaction
description: "Smart conversation context management with reactive, proactive, and semantic compaction strategies"
version: "1.0.0"
tags: [context, compaction, optimization, conversation, memory]
---

# Context Compaction

## Activation
When managing long conversations that approach context window limits, or when the agent needs to efficiently compress conversation history while retaining critical information.

## Concept

Context compaction prevents context window overflow by intelligently summarizing older conversation turns while preserving recent context and critical information. Three modes with increasing sophistication. Ported from jcode's CompactionManager.

## Modes

### Reactive (Default)
Trigger compaction when context reaches a fixed threshold.
```
Threshold: 80% of token budget
Critical: 95% (emergency hard-compact)
```

### Proactive
Predict when compaction will be needed based on token growth rate.
```
Track EWMA of per-turn token count over rolling window (20 turns)
Predict turns until threshold based on growth rate
Trigger compaction early to avoid mid-response overflow
```

### Semantic
Compact based on topic shifts and relevance scoring.
```
Embed each turn's content (first 512 chars)
Detect topic shifts via embedding similarity drops
Keep high-relevance turns verbatim, summarize low-relevance
Falls back to proactive if embeddings unavailable
```

## Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Token Budget | 200,000 | Max context window size |
| Compaction Threshold | 80% | Trigger reactive compaction |
| Critical Threshold | 95% | Emergency hard-compact |
| Manual Min Threshold | 10% | Minimum for manual /compact |
| Recent Turns to Keep | 10 | Never summarize these |
| Min Turns to Keep | 2 | Absolute minimum in emergency |
| System Overhead | 18,000 tokens | Reserved for system prompt + tools |
| Context Guard | 90% | Truncate oversized tool outputs |
| Single Output Max | 30% | Max context fraction for one tool |

## Context Overflow Guard

Before returning tool output, check if it would overflow:
```
if (current_tokens + output_tokens) > (budget * 0.90):
    truncate output to fit
if output_tokens > (budget * 0.30):
    truncate even if room exists (prevent single-output dominance)
```

Truncation message:
```
[kept content]
WARNING: OUTPUT TRUNCATED - This output was Xk tokens which would 
exceed the context window. Use more targeted queries.
```

## Compaction Process

### Step 1: Identify Compactable Range
```
total_messages - recent_turns_to_keep = compactable_range
Never compact: system messages, the first user message
Always compact: old tool results (largest token consumers)
```

### Step 2: Generate Summary
Use the model itself to summarize the compactable range:
```
"Summarize our conversation so you can continue this work later.
Write in natural language with these sections:
- Key decisions made
- Current state of the task  
- Important context and constraints
- What was tried and what worked/failed
- Next steps"
```

### Step 3: Replace
```
[System prompt]
[Summary of turns 1-N]  <- replaces all old messages
[Recent turn N+1]
[Recent turn N+2]
...
[Current turn]
```

### Emergency Hard-Compact (>95%)
When context is critically full:
1. Truncate all tool results to 4000 chars
2. Drop intermediate tool-use/tool-result pairs
3. Keep only system, first user message, and recent turns
4. No summary generation (would require API call that might fail)

## Conversation Search Integration

When conversation history has been compacted, a conversation_search tool lets the agent retrieve specific information from the pre-compaction history:

| Action | Description |
|--------|-------------|
| `query: "auth flow"` | Keyword search in compacted history |
| `turns: {start: 5, end: 10}` | Get specific turn range |
| `stats: true` | Conversation statistics |

## Best Practices

1. **Monitor context usage**: Track effective_token_count / token_budget ratio
2. **Compact proactively**: Don't wait for 80% - compact when you detect topic shifts
3. **Preserve decisions**: Summaries must capture WHY decisions were made, not just WHAT
4. **Tool output discipline**: Use targeted queries (small line ranges, specific patterns) to avoid oversized tool outputs
5. **Manual compaction**: Use /compact when switching to a new subtask within the same session
