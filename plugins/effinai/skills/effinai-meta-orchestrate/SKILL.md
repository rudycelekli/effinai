---
name: effinai-meta-orchestrate
description: >
  Meta-orchestration via Multica — orchestrate multiple Effin.AI instances, coordinate 
  swarm fleets across organizations, visual management dashboard. Use when managing 
  multiple orchestrators or scaling beyond a single swarm.
argument-hint: "[connect|status|orchestrate|agents|runtimes]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  mcp__effinai__effinai_multica_status
  mcp__effinai__effinai_multica_connect
  mcp__effinai__effinai_multica_agents
  mcp__effinai__effinai_multica_runtimes
  mcp__effinai__effinai_multica_issues
  mcp__effinai__effinai_multica_create_issue
  mcp__effinai__effinai_multica_orchestrate
  mcp__effinai__effinai_multica_orchestration_status
  mcp__effinai__effinai_multica_autopilots
  mcp__effinai__effinai_multica_create_autopilot
  mcp__effinai__effinai_multica_workspaces
  mcp__effinai__effinai_multica_chat
metadata:
  priority: 6
  promptSignals:
    phrases:
      - "meta orchestrate"
      - "orchestrate orchestrators"
      - "multica"
      - "multiple swarms"
      - "fleet"
    anyOf:
      - "multica"
      - "meta-orchestrat"
      - "fleet"
    minScore: 4
---

# Effin.AI Meta-Orchestration

Orchestrate multiple Effin.AI instances — the only system that coordinates orchestrators.

## Setup

```bash
effin multica connect --url https://your-instance.example.com --token sk-...
```

## Capabilities

- **Visual Dashboard** — Next.js web UI with board view, agent assignment, real-time WebSocket streaming
- **Multi-Swarm Orchestration** — Break complex tasks across multiple Effin.AI runtimes
- **Agent Fleet Management** — View and manage agents across all connected instances
- **Autopilot Workflows** — Automated recurring task execution
- **Cross-Org Collaboration** — Multi-workspace support with team isolation

## Usage

```bash
effin multica status
effin multica agents
effin multica runtimes
effin multica orchestrate --task "Build entire microservices platform"
effin multica orchestration-status --id <id>
effin multica autopilots
```

## 12 MCP Tools
`multica_status`, `multica_connect`, `multica_agents`, `multica_runtimes`, `multica_issues`, `multica_create_issue`, `multica_orchestrate`, `multica_orchestration_status`, `multica_autopilots`, `multica_create_autopilot`, `multica_workspaces`, `multica_chat`
