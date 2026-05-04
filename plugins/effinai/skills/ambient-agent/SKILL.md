---
name: ambient-agent
description: "Background autonomous agent that runs between sessions — memory maintenance, proactive work, scheduled tasks, and overnight operations"
version: "1.0.0"
tags: [automation, background, autonomous, scheduling, proactive]
---

# Ambient Agent

## Activation
When the user wants background autonomous work between interactive sessions, scheduled task execution, overnight operations, or proactive codebase maintenance.

## Concept

The ambient agent is a background process that runs autonomously when no interactive session is active. It performs scheduled tasks, memory maintenance, proactive work, and can run extended overnight operations. Ported from jcode's ambient mode and overnight runner.

## Architecture

### Lifecycle
```
User Session Active -> Ambient Paused
User Session Ends   -> Ambient Resumes
Scheduled Task Due  -> Ambient Wakes
Overnight Start     -> Extended Autonomous Run
Morning             -> Report Delivered
```

### Ambient Cycle
Each cycle the agent:
1. Check scheduled task queue for due items
2. Review memory graph health (stale entries, missing embeddings, fragmentation)
3. Gather recent session summaries for context
4. Execute highest-priority pending work
5. Schedule next wake time
6. End cycle with summary

### Ambient Status
| Status | Description |
|--------|-------------|
| **Idle** | No work pending, sleeping |
| **Running** | Actively executing a cycle |
| **Scheduled** | Sleeping until next_wake time |
| **Paused** | User session active, ambient suspended |
| **Disabled** | Turned off by user |

## Scheduled Tasks

### Creating a Schedule
```
schedule:
  context: "Run test suite and report failures"
  scheduled_for: "2025-01-15T03:00:00Z"
  priority: high
  target: ambient | session | spawn
  working_dir: "/path/to/project"
  relevant_files: ["src/main.rs", "tests/"]
  git_branch: "feature/new-api"
```

### Priority Levels
| Priority | Description |
|----------|-------------|
| **High** | Execute immediately when due |
| **Normal** | Execute in next available cycle |
| **Low** | Execute when no higher-priority work exists |

### Target Types
| Target | Behavior |
|--------|----------|
| **Ambient** | Hand task to background ambient agent |
| **Session** | Deliver reminder to a specific interactive session |
| **Spawn** | Create a new session derived from parent |

## Overnight Mode

Extended autonomous operation for large tasks that benefit from uninterrupted runtime.

### Starting Overnight
```
/overnight start 8h "Refactor authentication module and add comprehensive tests"
```

### Overnight Manifest
```
run_id: unique identifier
parent_session_id: originating session
coordinator_session_id: overnight coordinator
started_at: timestamp
target_wake_at: when to prepare morning report
handoff_ready_at: 30min before wake for report prep
mission: user's stated objective
max_agents_guidance: 3-5 agents
```

### Overnight Phases
```
1. Preflight    — Analyze mission, create task cards, validate approach
2. Execution    — Work through task cards with coordinated agents
3. Handoff Prep — 30min before wake: stop new work, prepare report
4. Morning Report — Summary of what was done, what needs review
```

### Overnight Commands
| Command | Description |
|---------|-------------|
| `/overnight start <duration> "<mission>"` | Begin overnight run |
| `/overnight status` | Check current overnight progress |
| `/overnight log` | View event log |
| `/overnight review` | See morning report |
| `/overnight cancel` | Request graceful stop |

## Memory Maintenance

The ambient agent performs these memory health tasks:
- **Backfill embeddings**: Generate embeddings for memories that lack them
- **Consolidate duplicates**: Merge near-duplicate memories
- **Prune stale entries**: Archive memories with low scores and no recent access
- **Rebuild clusters**: Re-cluster memories based on updated embeddings
- **Cross-session learning**: Extract patterns from recent session transcripts

## Proactive Work

When no scheduled tasks are pending, the ambient agent can:
- Run test suites and report new failures
- Check for dependency updates
- Scan for security vulnerabilities
- Update documentation that's drifted from code
- Optimize frequently-accessed code paths
- Generate missing test coverage

## Safety System
- Never make destructive changes without user permission
- Request permission tool for irreversible actions
- Resource budgets prevent runaway consumption
- User directives can inject instructions mid-cycle

## Integration with Effin.AI

### As Swarm Agent
```bash
effin agent spawn --type ambient-agent --background
```

### Via Memory System
```bash
effin memory store --key "ambient-schedule" --value '{"task":"run tests","at":"3am"}' --namespace ambient
```

### Via Hooks
```bash
effin hooks fire --event session-end --data '{"extract_learnings": true}'
```
