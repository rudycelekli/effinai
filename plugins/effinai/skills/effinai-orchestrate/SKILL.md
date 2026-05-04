---
name: effinai-orchestrate
description: >
  Intelligent auto-orchestration — describe any task and Effin.AI automatically determines 
  the optimal agents, skills, topology, model tier, and consensus mechanism. Use when 
  the user describes a complex task, wants to build something, or needs multi-agent coordination.
argument-hint: "[task description]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  Read
  Write
  Edit
  Glob
  Grep
  Agent
  mcp__effinai__effinai_orchestrate
  mcp__effinai__effinai_swarm_init
  mcp__effinai__effinai_agent_spawn
  mcp__effinai__effinai_route
  mcp__effinai__effinai_status
metadata:
  priority: 10
  promptSignals:
    phrases:
      - "build"
      - "implement"
      - "create"
      - "orchestrate"
      - "spawn swarm"
      - "multi-agent"
    anyOf:
      - "build"
      - "implement"
      - "orchestrate"
      - "swarm"
      - "agents"
    minScore: 4
  docs:
    - "https://github.com/rudycelekli/effinai-swarm"
retrieval:
  aliases:
    - "auto orchestrate"
    - "smart orchestration"
    - "task orchestration"
  intents:
    - "build a feature with multiple agents"
    - "orchestrate a complex task"
    - "auto-select agents for a task"
  entities:
    - "Effin.AI"
    - "orchestration"
    - "swarm"
---

# Effin.AI Auto-Orchestration

The most powerful way to use Effin.AI — just describe what you want and the system handles everything.

## How It Works

1. **Analyze** — Parse task description for complexity, domain, and requirements
2. **Select Topology** — hierarchical (anti-drift), mesh (research), ring (pipeline), star (fan-out), adaptive (unknown)
3. **Pick Agents** — From 223 specialized types, choose the optimal combination
4. **Choose Skills** — Match relevant skills from 98 built-in library
5. **Route Models** — 3-tier routing: Booster (<1ms, $0), Fast (~500ms, haiku), Full (2-5s, opus)
6. **Execute** — With anti-drift monitoring, consensus, and self-learning

## Quick Start

```bash
# Auto-orchestrate — no manual setup needed
effin "build a payment system with Stripe integration"

# Or via MCP tool
# effinai_orchestrate { task: "implement OAuth2 authentication" }
```

## Agent Presets (Auto-Selected)

| Task Type | Agents Spawned | Topology |
|-----------|---------------|----------|
| Bug fix | researcher, coder, tester | hierarchical |
| Feature | architect, coder, tester, reviewer | hierarchical |
| Refactor | architect, coder, reviewer | hierarchical |
| Performance | optimizer, coder, tester | hierarchical |
| Security | security-auditor, coder, reviewer | hierarchical |
| Research | researcher, coder | mesh |
| Full | coordinator + all specialists | adaptive |

## Anti-Drift Protection

Every orchestrated task includes automatic drift detection:
- Task keyword overlap scoring (40% threshold)
- Automatic realignment messages on divergence
- Reassignment after 2 failed realignments
- All drift events logged for system improvement

## Self-Learning

Successful patterns are stored with TF-IDF embeddings in HNSW index:
- Pattern boost: +20% score on success
- Pattern decay: -10% on staleness
- After 3+ matches, context auto-injected into future tasks

## MCP Tools Available

- `effinai_orchestrate` — Intelligent auto-orchestration (the main entry point)
- `effinai_route` — Get model routing recommendation for a task
- `effinai_swarm_init` — Initialize swarm with specific topology
- `effinai_agent_spawn` — Spawn specific agent type
- `effinai_status` — Full system status dashboard
