---
name: session-search
description: "Cross-session RAG search across all past sessions for recalling decisions, solutions, and context from previous work"
version: "1.0.0"
tags: [search, rag, cross-session, history, recall]
---

# Session Search

## Activation
When the agent needs to recall information from past sessions: previous decisions, solutions that worked, patterns discovered, or context from earlier conversations.

## Concept

A RAG (Retrieval-Augmented Generation) system that searches across all past session transcripts. Optimized for agent recall rather than raw text search. Ported from jcode's session_search tool.

## Design Principles

- **Current session excluded by default**: Prevents the search from finding itself
- **System messages hidden**: Filters out system reminders and tool definitions
- **Tool results hidden by default**: Removes noisy tool call/result pairs to surface conclusions
- **Session metadata searchable**: Session names, working directories, dates are first-class results
- **Grouped by session**: Results deduplicated per session to avoid flooding with hits from one verbose session
- **Journal persistence**: Searches both snapshot files and incremental journal entries

## Search Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `query` | required | Search query text |
| `limit` | 10 | Max results to return (max 50) |
| `max_per_session` | 1 | Max hits from a single session (max 20) |
| `include_current` | false | Include active session in results |
| `include_tools` | false | Include tool call/result messages |
| `include_system` | false | Include system/display messages |
| `working_dir` | none | Filter to sessions from a specific project |
| `role_filter` | none | Restrict to user, assistant, or metadata |
| `date_from` | none | Only sessions after this date |
| `date_to` | none | Only sessions before this date |
| `max_scan_sessions` | 1000 | Max sessions to scan (max 10,000) |

## Search Process

### Step 1: Pre-filter
```
1. List all session snapshot files (sorted by modification time, newest first)
2. Apply date filters if specified
3. Apply working_dir filter if specified  
4. Raw text scan of file contents for query terms (fast pre-filter)
5. Limit to MAX_DESERIALIZE (500) candidates
```

### Step 2: Deep Search
```
For each candidate session (parallel, 8 threads):
1. Deserialize session snapshot
2. Load journal entries if present
3. Search through messages:
   - Skip system messages (unless include_system)
   - Skip tool messages (unless include_tools)
   - Match query against message content
   - Score by relevance (keyword frequency, recency, position)
4. Collect up to max_per_session hits
```

### Step 3: Format Results
```
For each hit:
- Session name and date
- Working directory
- Matching message with surrounding context (up to 5 context messages)
- Role (user/assistant)
- Session ID for reference
```

## Result Format

```
## Session: "Auth refactor" (2025-01-10)
Working dir: /home/user/project

> **User**: How should we handle token refresh?

**Assistant**: We decided on a dual-token approach: short-lived access 
tokens (15min) with long-lived refresh tokens (30 days). The refresh 
endpoint should be rate-limited to prevent token farming...

---
```

## Use Cases

### Recall a Previous Decision
```
query: "database migration strategy"
-> Finds the session where you discussed and decided on the approach
```

### Find a Solution That Worked
```
query: "fixed the CORS error"
role_filter: "assistant"
-> Finds assistant messages explaining successful fixes
```

### Check What Was Tried Before
```
query: "performance optimization attempts"
include_tools: true
-> Includes the actual commands and code changes that were tried
```

### Find Work on a Specific Project
```
query: "deployment pipeline"
working_dir: "/home/user/my-project"
-> Only searches sessions from that project
```

## Performance

- Pre-filtering via raw text scan avoids deserializing irrelevant sessions
- Parallel file scanning (8 threads) for large session archives
- Session grouping (max_per_session=1) prevents any single verbose session from dominating results
- Recency bias: newer sessions scored higher when relevance is equal
