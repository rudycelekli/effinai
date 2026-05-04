---
name: composio-connect
description: "Connect to 1000+ external apps via Composio — send emails, create issues, post messages, update databases across Gmail, Slack, GitHub, Notion, and more"
version: "1.0.0"
tags: [integration, automation, apps, composio, api]
createdBy: "built-in"
status: "active"
---

# Composio Connect

## Activation
When the user wants to take real actions in external apps: send emails, create issues, post messages, update databases, or automate cross-app workflows.

## What Changes

| Without Connect | With Connect |
|-----------------|--------------|
| "Here's a draft email..." | Sends the email |
| "You should create an issue..." | Creates the issue |
| "Post this to Slack..." | Posts it |
| "Add this to Notion..." | Adds it |

## Supported Apps (1000+)

- **Email:** Gmail, Outlook, SendGrid
- **Chat:** Slack, Discord, Teams, Telegram
- **Dev:** GitHub, GitLab, Jira, Linear
- **Docs:** Notion, Google Docs, Confluence
- **Data:** Sheets, Airtable, PostgreSQL
- **CRM:** HubSpot, Salesforce, Pipedrive
- **Storage:** Drive, Dropbox, S3
- **Social:** Twitter, LinkedIn, Reddit

## Setup

### 1. Get API Key
Get your free key at [platform.composio.dev](https://platform.composio.dev)

### 2. Set Environment Variable
```bash
export COMPOSIO_API_KEY="your-key"
```

### 3. Install
```bash
pip install composio          # Python
npm install @composio/core    # TypeScript
```

## How It Works

Uses Composio Tool Router:
1. User asks to do something
2. Tool Router finds the right tool (1000+ options)
3. OAuth handled automatically
4. Action executes and returns result

## Implementation

```python
from composio import Composio
import os

composio = Composio(api_key=os.environ["COMPOSIO_API_KEY"])
session = composio.create(user_id="user_123")

# MCP integration
mcp_config = {
    "type": "http",
    "url": session.mcp.url,
    "headers": {"x-api-key": os.environ["COMPOSIO_API_KEY"]},
}
```

## Auth Flow

First time using an app:
1. Agent detects authorization needed
2. Provides OAuth link to user
3. User authorizes and confirms
4. Connection persists for future use

## Usage Examples

**Send Email:**
```
Email sarah@acme.com - Subject: "Shipped!" Body: "v2.0 is live"
```

**Create GitHub Issue:**
```
Create issue in my-org/repo: "Mobile timeout bug" with label:bug
```

**Post to Slack:**
```
Post to #engineering: "Deploy complete - v2.4.0 live"
```

**Chain Actions:**
```
Find GitHub issues labeled "bug" from this week, summarize, post to #bugs on Slack
```

## Framework Support
- Claude Agent SDK: `pip install composio claude-agent-sdk`
- OpenAI Agents: `pip install composio openai-agents`
- Vercel AI: `npm install @composio/core @composio/vercel`
- LangChain: `pip install composio-langchain`
- Any MCP Client: Use `session.mcp.url`

## Troubleshooting
- **Auth required**: Click link, authorize, say "connected"
- **Action failed**: Check permissions in target app
- **Tool not found**: Be specific: "Slack #general" not "send message"
