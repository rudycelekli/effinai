---
name: swarm-coordinator
description: "Multi-agent swarm coordination with spawn, communicate, plan graphs, and agent lifecycle management"
version: "1.0.0"
tags: [swarm, multi-agent, coordination, parallel, orchestration]
---

# Swarm Coordinator

## Activation
When a task requires multiple agents working in parallel, or when the agent needs to spawn, coordinate, and manage sub-agents for complex work.

## Concept

A coordination system for multi-agent swarms where a coordinator agent manages workers through structured communication. Supports plan graphs with dependency tracking, agent lifecycle management, and inter-agent messaging. Ported from jcode's communicate/swarm tool and subagent system.

## Agent Lifecycle

### Spawning
```
swarm spawn:
  role: "coder"
  model: "sonnet"           # optional model override
  task: "Implement JWT middleware"
  working_dir: "/project"   # inherit or override
```

Spawn options:
| Option | Description |
|--------|-------------|
| `role` | Agent specialization (coder, tester, reviewer, researcher) |
| `model` | Model to use (defaults to parent's swarm_model or main model) |
| `task` | Initial task description |
| `session_id` | Resume an existing agent session |

### Agent States
| State | Description |
|-------|-------------|
| **Spawning** | Being created |
| **Ready** | Idle, waiting for work |
| **Working** | Actively processing a task |
| **Completed** | Finished its task |
| **Failed** | Encountered an error |
| **Stopped** | Manually stopped |

## Communication

### Send Message to Agent
```
swarm send:
  target: "agent-session-id"
  message: "Authentication module is ready, you can start integration tests"
  delivery: "interrupt" | "queue"
```

### Delivery Modes
| Mode | Description |
|------|-------------|
| **Interrupt** | Deliver immediately via soft interrupt |
| **Queue** | Queue for next turn boundary |

### Broadcast
```
swarm broadcast:
  message: "All agents: switching to feature branch v2"
  filter: "working"  # only to agents in specific state
```

## Plan Graphs

Structure complex work as a dependency graph:

```
plan create:
  tasks:
    - id: "design"
      description: "Design the API schema"
      assignee: null  # auto-assign
    - id: "implement"
      description: "Implement the endpoints"
      depends_on: ["design"]
    - id: "test"
      description: "Write integration tests"
      depends_on: ["implement"]
    - id: "docs"
      description: "Write API documentation"
      depends_on: ["design"]  # can start after design, parallel with implement
```

### Plan Graph Status
```
plan status:
  active: ["implement", "docs"]
  next_ready: ["test"]
  completed: ["design"]
  blocked: []
```

### Task Assignment
```
plan assign:
  task_id: "implement"
  agent_id: "auto"  # or specific agent session ID
```

Auto-assignment finds a ready or completed agent and gives it the next task.

## Swarm Actions

| Action | Description |
|--------|-------------|
| `spawn` | Create a new agent |
| `list` | List all agents and their states |
| `status` | Get detailed agent status |
| `send` | Send message to specific agent |
| `broadcast` | Send message to all/filtered agents |
| `await` | Wait for agent(s) to reach target state |
| `stop` | Stop a specific agent |
| `cleanup` | Remove completed/failed agents |
| `plan create` | Create a task dependency graph |
| `plan status` | Get plan execution status |
| `plan assign` | Assign a task to an agent |
| `plan complete` | Mark a task as complete |
| `run` | Spawn agent, run task, await completion, return result |

## Coordination Patterns

### Fan-Out / Fan-In
```
1. Coordinator spawns N agents with independent tasks
2. Each agent works in parallel
3. Coordinator awaits all to complete
4. Coordinator synthesizes results
```

### Pipeline
```
1. Agent A produces output
2. Output passed as input to Agent B
3. Agent B produces output
4. Output passed to Agent C
```

### Supervisor
```
1. Coordinator monitors agent health
2. If agent fails, coordinator retries or reassigns
3. If agent stuck, coordinator sends interrupt with guidance
4. Coordinator enforces timeouts
```

## Agent Cleanup

After work is done, clean up to free resources:
```
swarm cleanup:
  target_statuses: ["ready", "completed", "failed", "stopped"]
  max_age_hours: 24
  dry_run: false
```

## Model Routing for Agents

Agent model selection priority:
1. Explicit model in spawn request
2. Parent session's subagent_model setting
3. Config-level swarm_model
4. Parent session's own model

This enables cost-efficient swarms where workers use cheaper models while the coordinator uses a more capable one.
