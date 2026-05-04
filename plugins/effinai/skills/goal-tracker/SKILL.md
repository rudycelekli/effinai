---
name: goal-tracker
description: "Structured goal management with milestones, progress tracking, session attachment, and side-panel visualization"
version: "1.0.0"
tags: [goals, milestones, tracking, productivity, planning]
---

# Goal Tracker

## Activation
When the user sets multi-session objectives that need persistent tracking, or when work spans multiple sessions and needs structured progress visibility.

## Concept

A persistent goal management system that survives across sessions. Goals have structured milestones, success criteria, progress tracking, and checkpoint updates. They sync to memory for context-aware retrieval and display in a side panel. Ported from jcode's goal system.

## Goal Structure

```yaml
id: "auth-refactor"           # Auto-generated, sanitized
title: "Refactor authentication module"
scope: project | global       # Per-project or user-wide
status: draft | active | blocked | completed | archived
description: "Modernize auth to use JWT with refresh tokens"
why: "Current session-based auth doesn't scale for microservices"

success_criteria:
  - "All endpoints use JWT validation"
  - "Refresh token rotation implemented"
  - "Migration path documented"

milestones:
  - id: "design"
    title: "Design new auth flow"
    status: "completed"
    steps:
      - content: "Document current auth flow"
        status: "completed"
      - content: "Design JWT token structure"
        status: "completed"
  - id: "implement"
    title: "Implement JWT auth"
    status: "in_progress"
    steps:
      - content: "Create JWT middleware"
        status: "completed"
      - content: "Add refresh endpoint"
        status: "pending"

current_milestone_id: "implement"
progress_percent: 45
next_steps:
  - "Implement refresh token endpoint"
  - "Add rate limiting to refresh"
blockers:
  - "Need Redis for token blacklist"

updates:
  - at: "2025-01-10"
    summary: "Completed JWT middleware, passes all existing tests"
  - at: "2025-01-09"
    summary: "Designed token structure, reviewed with team"
```

## Actions

| Action | Description |
|--------|-------------|
| `create` | Create a new goal with title, scope, optional details |
| `update` | Update goal fields (status, progress, milestones, etc.) |
| `checkpoint` | Add a progress update note |
| `show` | Display a specific goal in side panel |
| `list` | Show all goals overview |
| `focus` | Bring goal display to foreground |
| `resume` | Auto-resume the most relevant active goal |

## Scopes

| Scope | Storage | Use Case |
|-------|---------|----------|
| **Project** | Per-working-directory | Codebase-specific goals |
| **Global** | User-level | Cross-project objectives |

## Session Attachment

Goals attach to sessions for automatic context:
```
1. On session start: check for attached goal or find resumable goal
2. Display goal in side panel with current status
3. On goal update: refresh side panel display
4. Goal context injected into memory for relevance
```

## Memory Sync

Goals automatically sync to the memory system:
- Memory ID: `goal:{goal_id}`
- Category: `Custom("goal")`
- Trust: High
- Tags: `goal`, `goal:{id}`, `goal_status:{status}`, `goal_scope:{scope}`, title terms
- Content: Structured summary of goal state

This enables the memory system to surface relevant goals when the agent is working on related topics.

## Status Sorting

Goals display in priority order:
1. **Active** goals first
2. **Blocked** goals second
3. **Draft** goals third
4. Within same status: most recently updated first
5. Completed and archived goals last

## ID Collision Handling

If a goal ID already exists, auto-increment:
```
auth-refactor -> auth-refactor-2 -> auth-refactor-3
```

## Side Panel Display

### Goal Detail View
```
# Goal: Refactor authentication module

**Status:** active
**Scope:** project
**Progress:** 45%
**Updated:** 2025-01-10

## Description
Modernize auth to use JWT with refresh tokens...

## Current milestone
### Implement JWT auth
- [x] Create JWT middleware
- [ ] Add refresh endpoint

## Next steps
1. Implement refresh token endpoint
2. Add rate limiting to refresh

## Blockers
- Need Redis for token blacklist

## Recent updates
- 2025-01-10: Completed JWT middleware, passes all existing tests
```

### Header Badge
When a goal is active, show a compact badge:
```
[target emoji] Auth refactor (45%)
```

## Workflow

### Starting a New Feature
```
1. Create goal with title, description, success criteria
2. Add milestones for major phases
3. Goal attaches to current session
4. Work through milestones, checkpointing progress
5. Mark completed when success criteria met
```

### Resuming Across Sessions
```
1. New session starts
2. System finds most relevant resumable goal
3. Goal displayed with current state
4. Agent has full context of where things left off
```
