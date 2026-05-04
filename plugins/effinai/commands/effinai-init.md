---
name: effinai-init
description: Initialize Effin.AI — install MCP server, register tools, set up project, verify everything works
---

# Effin.AI Initialization

Set up everything for: $ARGUMENTS

## Step 1: Install the MCP Server

Run this to register the Effin.AI MCP server with Claude Code (586 tools):

```bash
claude mcp add effin -- npx -y effinai@latest
```

If `effinai` is not on npm yet, install from the private repo:

```bash
cd /path/to/effinai-swarm/swarmmy && npm install && npm link
claude mcp add effin -- effin
```

## Step 2: Initialize Project

```bash
effin init
```

This creates `.effinai/` with:
- `config.json` — swarm config (topology, consensus, strategy)
- `memory/` — vector memory store + HNSW index
- `messages/` — file-based inter-agent communication
- `agents/` — your custom agent definitions
- `skills/` — your custom skills
- `swarm/` — active swarm state
- `logs/` — execution logs

## Step 3: Health Check

```bash
effin doctor
```

Verifies: Node.js, git, config, memory database, MCP server.

## Step 4: Verify MCP Tools

After restart, all 586 tools are available. Test with:
- `effinai_status` — system status
- `effinai_agent_types` — list all 227 agent types  
- `effinai_skill_list` — list all 110 skills

## Step 5: Start Using

You're ready! Just describe what you want:

- "Build a payment system" → auto-orchestrates agents, skills, topology
- `/effinai-status` → system dashboard
- `/effinai-doctor` → health check

Skills auto-trigger based on what you type. No manual activation needed.
