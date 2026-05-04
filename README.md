# Effin.AI — Claude Code Plugin Marketplace

**One plugin. Everything included.**

586 MCP tools. 223 agent types. 98 skills. Self-learning swarms. Anti-drift consensus. Autonomous PRD-to-code. Meta-orchestration. Codebase intelligence. Domain verticals. 500+ app integrations. 8 model providers.

## Install (3 Steps)

### Step 1: Install the Plugin
```bash
/plugin marketplace add rudycelekli/effinai
/plugin install effinai@effinai
/reload-plugins
```

### Step 2: Register the MCP Server (586 tools)
```bash
claude mcp add effin -- npx -y effinai@latest
```
Then restart Claude Code. All 586 MCP tools become available.

### Step 3: Initialize Your Project
```bash
/effinai-init
```
Or manually:
```bash
effin init       # Creates .effinai/ directory
effin doctor     # Verify everything works
effin status     # See the dashboard
```

**That's it.** You now have 586 MCP tools, 227 agents, 110 skills, and everything else.

## How to Use

### Just Type Naturally
The skills auto-trigger based on what you say. No commands to memorize:
- "Build a login system" → auto-orchestrates coder + tester + reviewer agents
- "Review this code for security" → triggers security scan + code review
- "What if we migrate to microservices?" → runs multi-agent debate simulation
- "Predict AI adoption trends" → runs trend prediction analysis

### Slash Commands
```bash
/effinai "build a payment system"   # Auto-orchestrate any task
/effinai-status                      # System dashboard
/effinai-doctor                      # Health check
/effinai-init                        # Setup wizard
```

### Direct CLI
```bash
effin agent spawn --type coder --name my-coder --task "Add auth"
effin swarm init --topology hierarchical --max-agents 8
effin memory search --query "authentication patterns"
effin run --prd tasks.json
effin simulate --scenario "Should we use GraphQL?"
effin predict --topic "serverless adoption" --horizon long
effin multica orchestrate --task "Build entire platform"
```

### MCP Tools (in Claude Code)
All 586 tools are available directly. Key ones:
- `effinai_orchestrate` — describe any task, everything auto-selected
- `effinai_agent_spawn` — spawn any of 227 agent types
- `effinai_memory_search` — semantic vector search
- `effinai_nexus_self_analyze` — Effin.AI explains its own architecture
- `effinai_security_scan` — OWASP vulnerability scan
- `effinai_multica_orchestrate` — meta-orchestrate across swarm fleets

## What You Get

### 12 Skills (Auto-Triggered)
| Skill | What It Does |
|-------|-------------|
| `effinai-orchestrate` | Auto-orchestration — describe any task, Effin.AI handles the rest |
| `effinai-swarm` | 5 topologies, 3 consensus mechanisms, anti-drift detection |
| `effinai-agents` | 223 specialized agent types across every domain |
| `effinai-memory` | HNSW vector memory + self-learning with boost/decay |
| `effinai-autonomous` | PRD-to-code execution loop |
| `effinai-simulate` | Multi-persona debate + trend prediction |
| `effinai-codebase-intel` | Knowledge graph, impact analysis, architecture docs |
| `effinai-security` | OWASP, PII, compliance, threat modeling |
| `effinai-meta-orchestrate` | Orchestrate multiple Effin.AI instances via Multica |
| `effinai-skills` | 98 built-in skills with curator lifecycle |
| `effinai-tools` | External tools + 500+ app integrations + cost tracking |
| `effinai-domain-verticals` | Finance, healthcare, legal, IoT, quantum, trading |

### 4 Agents
| Agent | Model | Role |
|-------|-------|------|
| `effinai-coordinator` | Opus | Swarm orchestration, anti-drift, consensus |
| `effinai-coder` | Sonnet | Full-stack development, quality gates |
| `effinai-researcher` | Sonnet | Codebase intelligence, pattern analysis |
| `effinai-security` | Sonnet | Security audit, PII, compliance |

### 3 Commands
| Command | Description |
|---------|-------------|
| `/effinai` | Main orchestration entry point |
| `/effinai-status` | System status dashboard |
| `/effinai-doctor` | Health checks and diagnostics |

### 586 MCP Tools (Auto-Registered)
The `.mcp.json` automatically registers the Effin.AI MCP server, giving you access to all 586 tools in Claude Code.

## Agent Types (223)

**Core Development** — coder, backend-dev, frontend-dev, mobile-dev, debugger, refactorer, and more

**Architecture** — architect, system-architect, cloud-architect, api-designer, database-admin

**Testing** — tester, qa-engineer, test-automation, performance-benchmarker

**DevOps** — devops-engineer, cicd-engineer, kubernetes-specialist, terraform-specialist

**Security** — security-auditor, security-architect, threat-modeler, penetration-tester

**AI/ML** — ml-developer, mlops-engineer, ai-prompt-engineer, nlp-specialist, llm-fine-tuner

**Game Dev** — game-designer, unity-architect, unreal-developer, godot-developer, roblox-experience-designer

**Spatial/XR** — xr-developer, visionos-engineer, metaverse-architect, webxr-developer

**Sales** — sales-strategist, sales-engineer, deal-closer, account-manager

**Marketing** — growth-hacker, brand-strategist, seo-specialist, content-strategist

**Finance** — financial-analyst, budget-planner, tax-strategist, investment-researcher

**Legal** — legal-analyst, contract-specialist, ip-specialist, regulatory-analyst

**Healthcare** — clinical-analyst, health-informatics, medical-writer

**HR/Ops** — recruiter, hr-specialist, operations-manager, supply-chain-analyst

...and 100+ more specialized types.

## Requirements

- Claude Code CLI
- Node.js 18+
- npm

## License

MIT
