---
name: cache-optimizer
description: "LLM cache management — track cache warmth, detect violations, optimize token usage with append-only validation and prefix hash tracking"
version: "1.0.0"
tags: [cache, optimization, tokens, cost, performance, llm]
---

# Cache Optimizer

## Activation
When the agent needs to optimize LLM token usage, track cache effectiveness, detect cache violations, or minimize API costs. Use when working with providers that support prompt caching (Anthropic, OpenAI, Fireworks/OpenRouter) or when debugging unexpected cost spikes.

## Concept

LLM providers cache the prefix of conversation messages so that repeated requests sharing the same prefix only charge for new tokens. This skill tracks the append-only property of message lists, detects when the cache is invalidated (messages removed, reordered, or modified), and provides strategies to maximize cache hit rates. Ported from jcode's `CacheTracker` which uses stable message hashing to validate prefix consistency across turns.

## Cache Tracking Architecture

### How LLM Caching Works
```
Turn 1: [system, user1]                    -> All tokens charged (cache miss)
Turn 2: [system, user1, asst1, user2]      -> Only user2 tokens charged (cache hit on prefix)
Turn 3: [system, user1, asst1, user2, ...]  -> Only new tokens charged (cache hit)

VIOLATION: [system, MODIFIED_user1, ...]    -> All tokens re-charged (prefix changed!)
```

### Prefix Hash Chain
The tracker maintains a hash chain over message prefixes:
```
hash_0 = hash(message_0)
hash_1 = extend_hash(hash_0, hash(message_1))
hash_2 = extend_hash(hash_1, hash(message_2))
...
```

Each turn, the tracker verifies that `hash_N` at the previous message count matches the stored hash. If it doesn't, a cache violation occurred.

## Cache Violation Detection

### Violation Types
| Type | Cause | Impact |
|------|-------|--------|
| **Messages removed** | Compaction, context window trim | Full re-charge of all tokens |
| **Messages modified** | Content edit, system prompt change | Re-charge from modification point |
| **Messages reordered** | Conversation restructuring | Full re-charge |
| **Prefix mismatch** | Tool output changed, metadata drift | Partial re-charge |

### Violation Record
```json
{
  "turn": 15,
  "message_count": 42,
  "expected_hash": "a1b2c3d4e5f6g7h8",
  "actual_hash": "x9y8z7w6v5u4t3s2",
  "reason": "Messages removed: had 45 messages, now have 42"
}
```

## Optimization Strategies

### 1. Stable System Prompts
```
Problem: Changing the system prompt invalidates the entire cache
Solution: 
- Keep system prompt static across turns
- Put dynamic content in user messages, not system prompt
- Use a versioned system prompt that only changes on explicit updates
```

### 2. Append-Only Message Management
```
Problem: Removing old messages to fit context window breaks the cache
Solution:
- Never remove messages from the prefix
- Use a "compaction boundary" -- compact messages before the boundary, 
  keep messages after it unchanged
- When compaction is needed, compact ALL messages before the boundary
  into a single summary message, then append new messages
```

### 3. Context Window Budget
```
Problem: Large contexts cost more tokens even with caching
Solution:
- Track token counts per message category:
  * System prompt: ~X tokens (fixed cost)
  * Cached prefix: ~Y tokens (free on cache hit)
  * New content: ~Z tokens (always charged)
- Budget allocation:
  * Reserve 20% for system prompt
  * Reserve 30% for agent reasoning
  * Use 50% for conversation history
```

### 4. Provider-Specific Optimization

#### Anthropic (Claude)
- Prompt caching enabled by default for conversations
- Cache key is the exact message prefix
- Cache TTL: 5 minutes of inactivity
- Strategy: Keep conversations active, avoid long pauses

#### OpenAI (GPT)
- Automatic caching for shared prefixes
- Cache key includes model version
- Strategy: Batch related requests, stable system prompts

#### Fireworks (via OpenRouter)
- Automatic caching but no metrics reported
- The CacheTracker's client-side validation is critical here
- Strategy: Use prefix hash tracking to detect silent cache misses

## Monitoring Dashboard

### Per-Turn Metrics
```
Turn 15:
  Messages: 42 (was 40)
  Prefix hash: a1b2c3d4 (matches previous: YES)
  Cache status: HIT
  New tokens: 1,234
  Cached tokens: 15,678
  Savings: $0.047

Turn 16:
  Messages: 38 (was 42) -- VIOLATION!
  Prefix hash: x9y8z7w6 (matches previous: NO)
  Cache status: MISS (compaction removed 4 messages)
  All tokens: 14,234
  Excess cost: $0.043
```

### Session Summary
```
Session totals:
  Turns: 45
  Cache hits: 38 (84.4%)
  Cache violations: 3
  Violation causes:
    - Turn 16: Compaction removed messages
    - Turn 29: System prompt updated
    - Turn 41: Context overflow trim
  Total tokens: 234,567
  Cached tokens: 189,234 (80.7%)
  Estimated savings: $1.23
```

## Integration Points

- **Context Compaction**: Coordinate with compaction skill to minimize cache violations
- **Session Resume**: When resuming a session, note that cache is cold (full re-charge on first turn)
- **Multi-Agent**: Each agent has its own cache -- spawning new agents starts cold
- **Cost Analysis**: Feed cache metrics into cost-optimizer for budget tracking

## Best Practices

1. Call `record_request(messages)` before every LLM API call
2. Check the return value for cache violations
3. Log violations with their reasons for post-session analysis
4. When compaction is unavoidable, do it in one step rather than gradually removing messages
5. Keep a hash history (last 10 prefix hashes) for debugging intermittent violations
6. Monitor cache hit rate -- if it drops below 70%, investigate the cause
