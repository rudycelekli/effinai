---
name: effinai-tools
description: >
  External tool integrations — stealth web scraping, browser automation, media generation, 
  500+ app connections via Composio, and 8 model providers (Anthropic, OpenAI, Google, 
  xAI, Kimi, DeepSeek, Ollama, LM Studio). Use for scraping, browser automation, or 
  connecting external services.
argument-hint: "[list|scrape|install|start] [--url URL]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  mcp__effinai__effinai_tools_list
  mcp__effinai__effinai_scrape
  mcp__effinai__effinai_composio_apps
  mcp__effinai__effinai_composio_actions
  mcp__effinai__effinai_composio_execute
  mcp__effinai__effinai_composio_connect
  mcp__effinai__effinai_composio_status
  mcp__effinai__effinai_providers_list
  mcp__effinai__effinai_cost_track
  mcp__effinai__effinai_cost_report
  mcp__effinai__effinai_budget_check
  mcp__effinai__effinai_benchmark
metadata:
  priority: 5
  promptSignals:
    phrases:
      - "scrape"
      - "browser"
      - "composio"
      - "external tool"
      - "providers"
      - "cost"
    anyOf:
      - "scrape"
      - "composio"
      - "provider"
      - "cost"
      - "budget"
    minScore: 4
---

# Effin.AI External Tools & Integrations

## Built-In Tools
| Tool | Category | Description |
|------|----------|-------------|
| Stealth Scraper | scraping | Anti-bot bypass with fingerprint spoofing |
| Stealth Browser | browser | Undetectable automation |
| OpenFang | agents | Autonomous agent OS |
| Swarm Simulation | simulation | Multi-persona debate engine |
| Trend Prediction | simulation | Multi-agent forecasting |
| GitNexus | coding | Codebase knowledge graph |
| Media Gen | media | 30+ AI models for images/video |
| Multi-Model | agents | GPT/Gemini/Grok via Claude interface |

## 500+ App Integrations
Native API access to Slack, GitHub, Jira, Notion, Salesforce, Stripe, AWS, and 500+ more via Composio connector.

## 8 Model Providers
Anthropic, OpenAI, Google, xAI, Kimi, DeepSeek, Ollama, LM Studio

## Cost Tracking
- `effinai_cost_track` — Track token usage
- `effinai_cost_report` — Cost by model/agent
- `effinai_budget_check` — Budget threshold alerts
- `effinai_benchmark` — Performance benchmarking
