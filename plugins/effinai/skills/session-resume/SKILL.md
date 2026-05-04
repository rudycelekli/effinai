---
name: session-resume
description: "Cross-session context preservation and resume — search past sessions, restore working context, and maintain continuity across conversations"
version: "1.0.0"
tags: [session, resume, context, persistence, continuity, memory]
---

# Session Resume

## Activation
When the agent needs to recall what happened in a previous session, resume interrupted work, search across past conversations, or maintain context continuity between sessions.

## Concept

A cross-session search and resume system that treats all past sessions as a searchable knowledge base. Unlike simple chat history, this tool understands session metadata (provider, model, timestamps, working directory), filters out noise (tool calls, system messages), and groups results by session to avoid duplicate floods. Ported from jcode's `SessionSearchTool` and session persistence system.

The core insight: past sessions contain solutions, decisions, and context that are valuable for current work. This skill makes that knowledge retrievable by query, date range, provider, model, or bookmarked status.

## Architecture

```
Session Search
     |
     +-- Snapshot files (full session state at save points)
     +-- Journal files (incremental message log between snapshots)
     +-- Metadata index (session id, timestamps, model, provider, working dir)
     |
     v
Pre-filter (date, provider, model, source) -> Deserialize (up to 500 sessions)
     |
     v
Text search across messages -> Group by session -> Rank by relevance
     |
     v
Context window (messages before/after each hit for surrounding context)
```

## Search Parameters

### Basic Search
```
query: "authentication middleware"
# Full-text search across all past session messages
# Returns grouped results: one best hit per session for diversity
```

### Filtered Search
```
query: "database migration"
working_dir: "/projects/myapp"       # Only sessions from this project
after: "2026-01-01"                  # Sessions after this date
before: "2026-03-15"                 # Sessions before this date
provider: "anthropic"                # Filter by AI provider
model: "claude"                      # Filter by model name substring
source: "jcode"                      # Source: jcode, claude, codex, or all
saved: true                          # Only bookmarked/saved sessions
```

### Result Control
```
query: "error handling pattern"
limit: 20                            # Max total results (default: 10, max: 50)
max_per_session: 3                   # Max hits from one session (default: 1)
max_scan_sessions: 2000              # Max sessions to scan (default: 1000)
context_before: 2                    # Include N messages before each hit
context_after: 2                     # Include N messages after each hit
```

### Content Filtering
```
query: "deploy script"
include_current: false               # Exclude current session (default: false)
include_tools: false                 # Exclude raw tool calls (default: false)
include_system: false                # Exclude system messages (default: false)
include_external: true               # Include sessions from other tools (default: true)
role: "assistant"                    # Only assistant messages (or "user", "metadata")
```

## Session Persistence

### What Gets Persisted
| Component | Storage | Description |
|-----------|---------|-------------|
| **Snapshot** | JSON file | Complete session state at save points |
| **Journal** | JSONL file | Incremental message log between snapshots |
| **Metadata** | Indexed fields | Session ID, timestamps, model, provider, working directory |
| **Bookmarks** | Flag | User-saved sessions for important conversations |

### Resume Workflow
```
1. Search for the session to resume:
   query: "the auth refactoring I was working on"
   working_dir: "/projects/myapp"
   
2. Review results to identify the correct session
   -> Returns session ID, timestamp, key messages

3. Load session context:
   - Recent messages from that session
   - Working directory state
   - File modifications made
   - Decisions and rationale captured
   
4. Continue work with full context restored
```

## Use Cases

### Resume Interrupted Work
```
"What was I working on yesterday in /projects/api?"
-> Search: query="", working_dir="/projects/api", after="2026-05-03"
-> Returns: session summary, last active files, pending tasks
```

### Recall a Solution
```
"How did I fix the CORS issue last month?"
-> Search: query="CORS fix", after="2026-04-01", before="2026-04-30"
-> Returns: the conversation where CORS was debugged, with solution context
```

### Cross-Project Knowledge
```
"Have I implemented OAuth anywhere before?"
-> Search: query="OAuth implementation", include_external=true
-> Returns: hits from all projects where OAuth was discussed/implemented
```

### Review Past Decisions
```
"Why did we choose PostgreSQL over MongoDB for this project?"
-> Search: query="PostgreSQL MongoDB decision", working_dir="/projects/myapp"
-> Returns: the discussion with reasoning and trade-offs
```

## Performance Notes

- Pre-filtering by date, provider, and working directory happens before deserialization for speed
- Up to 500 session files are deserialized per search (configurable via `max_scan_sessions`)
- 8 parallel threads scan session files for I/O throughput
- Results are grouped by session (default 1 hit per session) to maximize diversity
- Journal files are searched alongside snapshots to capture recent unsaved messages
- Metadata matches (session title, model name) are returned as first-class results alongside message matches
