---
name: effinai-agents
description: >
  Spawn, manage, and coordinate 223 specialized AI agent types across engineering, 
  design, sales, marketing, finance, game dev, legal, healthcare, spatial computing, 
  and more. Use when spawning agents, listing agent types, or managing agent lifecycle.
argument-hint: "[spawn|list|stop|types] [--type agent-type] [--name name]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  Read
  mcp__effinai__effinai_agent_spawn
  mcp__effinai__effinai_agent_list
  mcp__effinai__effinai_agent_stop
  mcp__effinai__effinai_agent_types
  mcp__effinai__effinai_agent_logs
  mcp__effinai__effinai_agent_assign
  mcp__effinai__effinai_agent_health
  mcp__effinai__effinai_agent_pool
  mcp__effinai__effinai_agent_send_message
  mcp__effinai__effinai_agent_broadcast
  mcp__effinai__effinai_agent_metrics
metadata:
  priority: 8
  promptSignals:
    phrases:
      - "spawn agent"
      - "create agent"
      - "agent types"
      - "list agents"
      - "stop agent"
    anyOf:
      - "agent"
      - "spawn"
    minScore: 4
---

# Effin.AI Agent Management — 223 Types

## Agent Categories

### Core Development (20+)
coder, senior-developer, backend-dev, frontend-dev, mobile-dev, debugger, refactorer, rapid-prototyper, embedded-engineer, kernel-developer, compiler-engineer, solidity-engineer, blockchain-dev

### Architecture & Design (10+)
architect, system-architect, cloud-architect, api-designer, database-admin, dba-postgres, dba-mysql, design-system, ux-designer, ui-designer

### Testing & Quality (8+)
tester, qa-engineer, test-automation, reviewer, penetration-tester, performance-benchmarker, performance-engineer, reliability-engineer

### DevOps & Infrastructure (12+)
devops-engineer, cicd-engineer, kubernetes-specialist, terraform-specialist, monitoring-specialist, incident-responder, cache-specialist, redis-specialist, kafka-specialist

### Security (6+)
security-auditor, security-architect, threat-modeler, threat-detection-engineer, compliance-auditor, privacy-officer

### AI & ML (8+)
ml-developer, mlops-engineer, ai-prompt-engineer, ai-safety-researcher, computer-vision, nlp-specialist, llm-fine-tuner, rag-architect

### Game Development (15+)
game-designer, game-artist, game-audio, level-designer, narrative-designer, multiplayer-engineer, godot-developer, unity-architect, unreal-developer, roblox-experience-designer

### Spatial Computing & XR (8+)
xr-developer, xr-interface-architect, webxr-developer, visionos-engineer, metaverse-architect, spatial-ui-designer, macos-metal-engineer

### Sales & Marketing (25+)
sales-strategist, growth-hacker, brand-strategist, content-strategist, seo-specialist, email-marketer, paid-media-auditor, ad-creative-strategist

### Finance & Legal (15+)
financial-analyst, budget-planner, tax-strategist, legal-analyst, contract-specialist, ip-specialist, regulatory-analyst

### Healthcare & Science (6+)
clinical-analyst, health-informatics, medical-writer, psychologist, anthropologist

### HR & Operations (15+)
recruiter, hr-specialist, training-specialist, culture-architect, supply-chain-analyst, procurement-specialist

## Commands

```bash
effin agent spawn --type coder --name my-coder --task "Add login endpoint"
effin agent list
effin agent types
effin agent stop --name my-coder
effin agent logs --name my-coder
```

## 3-Tier Model Routing

| Tier | Latency | Cost | Use Cases |
|------|---------|------|-----------|
| Booster | <1ms | $0 | Simple transforms |
| Fast (Haiku) | ~500ms | $0.0002 | Bug fixes, simple tasks |
| Full (Opus) | 2-5s | $0.015 | Architecture, security |

All agents use **ultrathink mode** (extended thinking) by default.
