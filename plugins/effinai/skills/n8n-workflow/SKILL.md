---
name: n8n-workflow
description: "Create, configure, and manage n8n automation workflows — node composition, trigger setup, webhook handling, and API integration"
version: "1.0.0"
tags: [automation, workflow, n8n, integration, webhooks, api]
---

# n8n Workflow

## Activation
When the agent needs to create automated workflows, connect services via APIs, set up webhook handlers, schedule recurring tasks, or build data pipelines. Use for any automation that connects multiple services or requires trigger-based execution.

## Concept

n8n is a workflow automation platform where workflows are directed graphs of nodes. Each node performs an action (HTTP request, data transform, send email, query database) and passes data to the next node. This skill generates n8n-compatible workflow JSON definitions that can be imported into any n8n instance. The agent designs workflows by composing nodes, configuring connections, and setting up triggers.

## Workflow Structure

### Anatomy of a Workflow
```json
{
  "name": "My Workflow",
  "nodes": [...],
  "connections": {...},
  "settings": {
    "executionOrder": "v1"
  },
  "active": false
}
```

### Node Types

#### Triggers (Starting Points)
| Node | Description | Configuration |
|------|-------------|---------------|
| `n8n-nodes-base.webhook` | HTTP webhook endpoint | Method, path, auth |
| `n8n-nodes-base.cron` | Scheduled execution | Cron expression |
| `n8n-nodes-base.emailReadImap` | Email arrival | IMAP settings, folder, filter |
| `n8n-nodes-base.githubTrigger` | GitHub event | Repo, event type (push, PR, issue) |
| `n8n-nodes-base.httpRequest` | Poll an API | URL, interval, method |

#### Actions
| Node | Description | Configuration |
|------|-------------|---------------|
| `n8n-nodes-base.httpRequest` | Make HTTP request | URL, method, headers, body |
| `n8n-nodes-base.code` | Run JavaScript/Python | Custom code |
| `n8n-nodes-base.if` | Conditional branch | Condition expression |
| `n8n-nodes-base.switch` | Multi-way branch | Cases and conditions |
| `n8n-nodes-base.merge` | Combine data streams | Mode (append, merge, join) |
| `n8n-nodes-base.splitInBatches` | Process items in batches | Batch size |
| `n8n-nodes-base.set` | Set/transform data | Field mappings |
| `n8n-nodes-base.function` | Transform with code | JavaScript function |

#### Integrations
| Node | Description |
|------|-------------|
| `n8n-nodes-base.slack` | Send Slack messages |
| `n8n-nodes-base.gmail` | Send/read Gmail |
| `n8n-nodes-base.googleSheets` | Read/write Google Sheets |
| `n8n-nodes-base.postgres` | Query PostgreSQL |
| `n8n-nodes-base.mongodb` | Query MongoDB |
| `n8n-nodes-base.openAi` | Call OpenAI API |
| `n8n-nodes-base.airtable` | Airtable operations |
| `n8n-nodes-base.notion` | Notion pages/databases |
| `n8n-nodes-base.jira` | Jira issue management |

## Workflow Patterns

### 1. Webhook -> Process -> Notify
```
Trigger: Webhook receives POST request
  -> Code node: Validate and transform payload
  -> If node: Check condition
    -> True: Slack notification
    -> False: Log and ignore
```

### 2. Scheduled Data Pipeline
```
Trigger: Cron (every 6 hours)
  -> HTTP Request: Fetch data from API
  -> Code: Transform/clean data
  -> Postgres: Upsert into database
  -> Google Sheets: Update dashboard
  -> Slack: Post summary
```

### 3. GitHub PR Review Pipeline
```
Trigger: GitHub PR event (opened/updated)
  -> HTTP Request: Fetch PR diff
  -> Code: Analyze changes
  -> OpenAI: Generate review comments
  -> HTTP Request: Post comments to PR
  -> Slack: Notify team
```

### 4. Error Alert Pipeline
```
Trigger: Webhook (error reporting service)
  -> Code: Parse error, check severity
  -> Switch: Route by severity
    -> Critical: PagerDuty + Slack #incidents
    -> High: Slack #alerts + Jira ticket
    -> Low: Log only
```

### 5. Data Sync Pipeline
```
Trigger: Cron (every 15 minutes)
  -> Postgres: Query new records since last sync
  -> SplitInBatches: Process 50 at a time
  -> HTTP Request: Push to external API
  -> Merge: Collect results
  -> Code: Generate sync report
  -> Email: Send report to admin
```

## Node Configuration

### HTTP Request Node
```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/data",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        {"name": "Content-Type", "value": "application/json"}
      ]
    },
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        {"name": "key", "value": "={{$json.value}}"}
      ]
    },
    "options": {
      "timeout": 30000,
      "retry": { "maxRetries": 3 }
    }
  }
}
```

### Code Node (JavaScript)
```json
{
  "type": "n8n-nodes-base.code",
  "parameters": {
    "jsCode": "const items = $input.all();\nconst results = items.map(item => {\n  return {\n    json: {\n      processed: true,\n      value: item.json.value * 2\n    }\n  };\n});\nreturn results;"
  }
}
```

### Conditional Node
```json
{
  "type": "n8n-nodes-base.if",
  "parameters": {
    "conditions": {
      "options": {
        "caseSensitive": true
      },
      "conditions": [
        {
          "leftValue": "={{$json.status}}",
          "rightValue": "error",
          "operator": { "type": "string", "operation": "equals" }
        }
      ]
    }
  }
}
```

## Connections Format

```json
{
  "connections": {
    "Webhook": {
      "main": [[{"node": "Process Data", "type": "main", "index": 0}]]
    },
    "Process Data": {
      "main": [[{"node": "Check Condition", "type": "main", "index": 0}]]
    },
    "Check Condition": {
      "main": [
        [{"node": "Send Alert", "type": "main", "index": 0}],
        [{"node": "Log Only", "type": "main", "index": 0}]
      ]
    }
  }
}
```

## Data Flow

### Expression Syntax
```
{{$json.fieldName}}           # Current item's field
{{$node["NodeName"].json}}    # Output from a specific node
{{$env.API_KEY}}              # Environment variable
{{$now.toISO()}}              # Current timestamp
{{$items("NodeName")}}        # All items from a node
{{$input.first().json}}       # First input item
```

### Error Handling
```json
{
  "settings": {
    "errorWorkflow": "error-handler-workflow-id"
  },
  "nodes": [
    {
      "name": "On Error",
      "type": "n8n-nodes-base.errorTrigger",
      "parameters": {}
    }
  ]
}
```

## Deployment

### Import to n8n
```bash
# Via CLI
n8n import:workflow --input=workflow.json

# Via API
curl -X POST http://localhost:5678/api/v1/workflows \
  -H "X-N8N-API-KEY: $N8N_API_KEY" \
  -H "Content-Type: application/json" \
  -d @workflow.json

# Activate
curl -X PATCH http://localhost:5678/api/v1/workflows/{id} \
  -H "X-N8N-API-KEY: $N8N_API_KEY" \
  -d '{"active": true}'
```

### Testing
```
1. Generate workflow JSON
2. Import to n8n instance (or use n8n CLI test mode)
3. Trigger manually via webhook or test execution
4. Verify data flows correctly through all nodes
5. Check error handling paths
6. Activate for production
```

## Integration

- **Ambient Agent**: Generate n8n workflows for scheduled maintenance tasks
- **Threat Hunting**: Create alert workflows for security findings
- **Test Generator**: Webhook-triggered test execution workflows
- **Session Resume**: Persist workflow designs across sessions for iteration
