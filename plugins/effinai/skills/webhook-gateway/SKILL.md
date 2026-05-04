---
name: webhook-gateway
description: "Create and manage webhook subscriptions — receive external events, route to agents, deliver to Slack/Discord/Telegram with HMAC verification"
version: "1.0.0"
tags: [integration, webhooks, events, gateway, automation]
createdBy: "built-in"
status: "active"
---

# Webhook Gateway

## Activation
When the user wants to receive webhooks from external services, route events to agents, or set up event-driven automation.

## Concept

Dynamic webhook subscriptions that persist to disk and are hot-reloaded without gateway restart. Each subscription gets a unique URL, HMAC secret for verification, event filtering, agent routing, and multi-platform delivery.

## Core Operations

### Subscribe (Create/Update)
```bash
webhook subscribe <name> \
  --events "push,pull_request" \
  --secret "your-hmac-secret" \
  --prompt "Analyze this GitHub event and summarize changes" \
  --skills "code-review,security-scan" \
  --deliver slack \
  --deliver-chat-id "#engineering"
```

Creates a subscription with:
- **Unique URL**: `http://localhost:8644/webhooks/<name>`
- **HMAC Secret**: For signature validation (auto-generated if not provided)
- **Event Filter**: Only process matching event types
- **Agent Prompt**: Instructions for the agent processing the event
- **Skills**: Skills to enable for the processing agent
- **Delivery Target**: Where to send results (log, slack, discord, telegram, github_comment)

### Direct Delivery Mode
```bash
webhook subscribe alerts \
  --deliver telegram \
  --deliver-chat-id "123456" \
  --deliver-only \
  --prompt "Alert: {payload}"
```
With `--deliver-only`, events are forwarded directly to the delivery target without agent processing. Zero LLM cost.

### List Subscriptions
```bash
webhook list
```
Shows all dynamic subscriptions with URLs, events, and delivery targets.

### Remove Subscription
```bash
webhook remove <name>
```

### Test Subscription
```bash
webhook test <name> --payload '{"event_type": "test", "message": "Hello"}'
```
Sends a test POST with proper HMAC signature to verify the subscription works.

## HMAC Signature Verification

All incoming webhooks are verified using HMAC-SHA256:
```
X-Hub-Signature-256: sha256=<hex-digest>
```

The signature is computed as:
```python
import hmac, hashlib
sig = "sha256=" + hmac.new(
    secret.encode(), payload.encode(), hashlib.sha256
).hexdigest()
```

## Subscription Schema

```json
{
  "description": "Agent-created subscription: alerts",
  "events": ["push", "release"],
  "secret": "auto-generated-token",
  "prompt": "Analyze this event and create a summary",
  "skills": ["code-review"],
  "deliver": "slack",
  "deliver_extra": {"chat_id": "#engineering"},
  "deliver_only": false,
  "created_at": "2024-01-15T10:30:00Z"
}
```

## Setup

The webhook platform must be enabled in configuration:

```yaml
platforms:
  webhook:
    enabled: true
    extra:
      host: "0.0.0.0"
      port: 8644
      secret: "global-hmac-secret"
```

Or via environment variables:
```bash
WEBHOOK_ENABLED=true
WEBHOOK_PORT=8644
WEBHOOK_SECRET=your-global-secret
```

## Use Cases

- **GitHub Events**: Push notifications, PR reviews, issue tracking
- **CI/CD Alerts**: Build failures, deployment notifications
- **Monitoring**: Alert routing from PagerDuty, Datadog, etc.
- **Custom Integrations**: Any service that sends webhooks
- **Cross-Platform Relay**: Receive webhook, deliver to Slack/Discord/Telegram
