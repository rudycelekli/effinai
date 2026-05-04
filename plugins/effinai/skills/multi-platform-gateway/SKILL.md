---
name: multi-platform-gateway
description: "Deploy agents across 20+ platforms — CLI, Telegram, Discord, Slack, WhatsApp, email, Home Assistant, webhooks, cron jobs, and API servers"
version: "1.0.0"
tags: [integration, platforms, gateway, deployment, multi-channel]
createdBy: "built-in"
status: "active"
---

# Multi-Platform Gateway

## Activation
When the user wants to deploy an agent to messaging platforms, set up cross-platform communication, or create a multi-channel bot.

## Supported Platforms (20+)

| Platform | Type | Toolset |
|----------|------|---------|
| CLI | Terminal | hermes-cli |
| Telegram | Messaging | hermes-telegram |
| Discord | Messaging | hermes-discord |
| Slack | Workspace | hermes-slack |
| WhatsApp | Messaging | hermes-whatsapp |
| Signal | Messaging | hermes-signal |
| BlueBubbles | iMessage | hermes-bluebubbles |
| Email | Email | hermes-email |
| Home Assistant | IoT | hermes-homeassistant |
| Mattermost | Workspace | hermes-mattermost |
| Matrix | Messaging | hermes-matrix |
| DingTalk | Workspace | hermes-dingtalk |
| Feishu | Workspace | hermes-feishu |
| WeCom | Workspace | hermes-wecom |
| Weixin | Messaging | hermes-weixin |
| QQBot | Messaging | hermes-qqbot |
| Webhook | Events | hermes-webhook |
| API Server | REST | hermes-api-server |
| Cron | Scheduled | hermes-cron |

## Architecture

### Platform Registry
Each platform is defined with:
- **label**: Display name with emoji
- **default_toolset**: Which tools are available on this platform

Plugin platforms can register dynamically at runtime alongside built-in platforms.

### Per-Platform Configuration

```yaml
platforms:
  telegram:
    enabled: true
    extra:
      bot_token: "${TELEGRAM_BOT_TOKEN}"
      
  discord:
    enabled: true
    extra:
      bot_token: "${DISCORD_BOT_TOKEN}"
      
  slack:
    enabled: true
    extra:
      bot_token: "${SLACK_BOT_TOKEN}"
      app_token: "${SLACK_APP_TOKEN}"
      
  cron:
    enabled: true
    extra:
      jobs:
        - name: "daily-report"
          schedule: "0 9 * * *"
          prompt: "Generate daily status report"
          deliver: "slack"
          deliver_chat_id: "#reports"
```

### Toolset Per Platform
Each platform gets a tailored set of tools appropriate for its context:
- CLI gets full filesystem and terminal access
- Messaging platforms get communication-focused tools
- Cron gets scheduled execution tools
- API Server gets request/response handling

## Gateway Setup

### Wizard-Based Setup
```bash
effin gateway setup
```
Interactive wizard walks through:
1. Select platforms to enable
2. Configure API tokens for each
3. Set up default skills and tools per platform
4. Configure delivery targets

### Manual Configuration
Edit `config.yaml` directly with platform blocks.

### Environment Variables
```bash
TELEGRAM_BOT_TOKEN=your-token
DISCORD_BOT_TOKEN=your-token
SLACK_BOT_TOKEN=your-token
SLACK_APP_TOKEN=your-app-token
WEBHOOK_ENABLED=true
WEBHOOK_PORT=8644
```

## Running the Gateway
```bash
effin gateway run
```
Starts all enabled platform adapters. Each adapter:
- Connects to its platform API
- Listens for incoming messages/events
- Routes to the agent with appropriate toolset
- Delivers responses back to the platform

## Cron Jobs

Schedule recurring agent tasks:
```yaml
cron:
  enabled: true
  extra:
    jobs:
      - name: "security-scan"
        schedule: "0 2 * * 1"  # Every Monday at 2 AM
        prompt: "Run a security scan on the codebase"
        skills: ["security-scan"]
        deliver: "slack"
        deliver_chat_id: "#security"
```

## Plugin Platform Registration

Third-party platforms can register dynamically:
```python
from gateway.platform_registry import platform_registry

platform_registry.register(
    name="custom-platform",
    label="Custom Platform",
    emoji="🔌",
    adapter_class=CustomAdapter,
    default_toolset="hermes-custom",
)
```

## Cross-Platform Delivery

Events from one platform can be delivered to another:
- Receive GitHub webhook -> Summarize -> Post to Slack
- Cron job runs -> Generate report -> Send via email
- Telegram message -> Process -> Update Notion
