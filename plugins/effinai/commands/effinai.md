---
name: effinai
description: Main Effin.AI command — auto-orchestrate any task with 586 MCP tools, 223 agents, 98 skills
---

Run the Effin.AI orchestrator for: $ARGUMENTS

## Steps

1. Analyze the task description to determine complexity and domain
2. Use `effinai_orchestrate` MCP tool to auto-select topology, agents, skills, and model tier
3. If orchestrate is unavailable, manually:
   - Initialize swarm: `effin swarm init --topology hierarchical --max-agents 8`
   - Spawn appropriate agents based on task type
   - Execute with anti-drift monitoring
4. Store successful patterns in memory for future learning
5. Report results with confidence scoring
