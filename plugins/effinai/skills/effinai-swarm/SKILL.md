---
name: effinai-swarm
description: >
  Multi-agent swarm coordination with 5 topologies, 3 consensus mechanisms, file-based 
  messaging, and anti-drift detection. Use when coordinating multiple agents, setting up 
  swarm topologies, or managing agent teams.
argument-hint: "[--topology hierarchical|mesh|ring|star|adaptive] [--max-agents N]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  Read
  Glob
  Grep
  mcp__effinai__effinai_swarm_init
  mcp__effinai__effinai_swarm_status
  mcp__effinai__effinai_swarm_stop
  mcp__effinai__effinai_swarm_topology
  mcp__effinai__effinai_swarm_agents
  mcp__effinai__effinai_swarm_consensus
  mcp__effinai__effinai_swarm_message
  mcp__effinai__effinai_agent_team_create
metadata:
  priority: 8
  promptSignals:
    phrases:
      - "swarm"
      - "topology"
      - "multi-agent"
      - "coordinate agents"
      - "consensus"
    anyOf:
      - "swarm"
      - "topology"
      - "consensus"
      - "multi-agent"
    minScore: 4
---

# Effin.AI Swarm Coordination

## 5 Topologies

| Topology | Best For | How It Works |
|----------|---------|-------------|
| `hierarchical` | Most tasks (anti-drift) | Coordinator dispatches, workers report back |
| `mesh` | Research/exploration | All agents communicate freely |
| `ring` | Pipeline tasks (A->B->C) | Sequential pass-through |
| `star` | Fan-out/fan-in | Central hub with spokes |
| `adaptive` | Unknown complexity | Starts hierarchical, adapts as needed |

## 3 Consensus Mechanisms
- **Raft** — Leader-based, tolerates f < n/2 (default)
- **Quorum** — Configurable threshold for decisions
- **Gossip** — Epidemic protocol for eventual consistency

## Anti-Drift Detection
1. Task hash from original description
2. Agent outputs compared against task keywords (overlap scoring)
3. Divergence > 40% triggers realignment message
4. After 2 failures, task reassigned to new agent

## Commands

```bash
effin swarm init --topology hierarchical --max-agents 8 --strategy specialized
effin swarm status
effin swarm stop
effin swarm route --task "description"
```

## MCP Tools
- `effinai_swarm_init` — Initialize with topology
- `effinai_swarm_status` — Get current state
- `effinai_swarm_topology` — Change topology at runtime
- `effinai_swarm_agents` — List agents in swarm with roles
- `effinai_swarm_consensus` — Consensus status and history
- `effinai_swarm_message` — Inter-agent messaging
- `effinai_agent_team_create` — Create named agent teams
