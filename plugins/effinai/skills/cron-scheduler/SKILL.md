---
name: cron-scheduler
description: "Schedule recurring agent tasks with cron expressions — run prompts on a schedule with multi-platform delivery"
version: "1.0.0"
tags: [automation, scheduling, cron, recurring, tasks]
createdBy: "built-in"
status: "active"
---

# Cron Scheduler

## Activation
When the user wants to schedule recurring tasks, set up automated reports, or create time-based agent jobs.

## Concept

Cron jobs are scheduled agent tasks that execute on a recurring basis. Each job specifies a cron expression, a prompt for the agent, optional skills to enable, and a delivery target for the output.

## Configuration

### In config.yaml
```yaml
platforms:
  cron:
    enabled: true
    extra:
      jobs:
        - name: "daily-standup-summary"
          schedule: "0 9 * * 1-5"        # 9 AM weekdays
          prompt: "Generate a standup summary from yesterday's git commits and open PRs"
          skills: ["code-review"]
          deliver: "slack"
          deliver_chat_id: "#team-standup"

        - name: "weekly-security-scan"
          schedule: "0 2 * * 1"           # Monday 2 AM
          prompt: "Run a security scan on the codebase and report vulnerabilities"
          skills: ["security-scan"]
          deliver: "email"
          deliver_chat_id: "security@company.com"

        - name: "hourly-monitoring"
          schedule: "0 * * * *"           # Every hour
          prompt: "Check system health metrics and alert if any thresholds exceeded"
          deliver: "telegram"
          deliver_chat_id: "123456789"

        - name: "monthly-report"
          schedule: "0 10 1 * *"          # 1st of month at 10 AM
          prompt: "Generate monthly performance report with key metrics"
          skills: ["performance-audit"]
          deliver: "slack"
          deliver_chat_id: "#reports"
```

## Cron Expression Reference

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-7, 0=Sun)
│ │ │ │ │
* * * * *
```

### Common Patterns
| Expression | Meaning |
|-----------|---------|
| `0 9 * * 1-5` | 9 AM weekdays |
| `0 2 * * 1` | Monday 2 AM |
| `0 * * * *` | Every hour |
| `*/15 * * * *` | Every 15 minutes |
| `0 10 1 * *` | 1st of month at 10 AM |
| `0 0 * * 0` | Sunday midnight |
| `0 8,12,17 * * *` | 8 AM, noon, and 5 PM |

## Job Properties

| Property | Required | Description |
|----------|----------|-------------|
| `name` | Yes | Unique identifier for the job |
| `schedule` | Yes | Cron expression |
| `prompt` | Yes | Agent instruction text |
| `skills` | No | Skills to enable for the agent |
| `deliver` | No | Delivery target (log, slack, discord, telegram, email) |
| `deliver_chat_id` | No | Target channel/address |
| `deliver_only` | No | Skip agent processing, forward directly |

## Delivery Targets

| Target | Description | Required Config |
|--------|-------------|-----------------|
| `log` | Write to local log (default) | None |
| `slack` | Post to Slack channel | SLACK_BOT_TOKEN, channel |
| `discord` | Post to Discord channel | DISCORD_BOT_TOKEN, channel |
| `telegram` | Send to Telegram chat | TELEGRAM_BOT_TOKEN, chat_id |
| `email` | Send via email | Email config |
| `github_comment` | Comment on GitHub PR/issue | GITHUB_TOKEN |

## Use Cases

### Daily Standup Automation
```yaml
- name: "standup"
  schedule: "0 9 * * 1-5"
  prompt: |
    Review yesterday's git commits and open PRs.
    Generate a standup summary:
    - What was completed
    - What's in progress
    - Any blockers
  deliver: "slack"
  deliver_chat_id: "#daily-standup"
```

### Dependency Monitoring
```yaml
- name: "dep-check"
  schedule: "0 3 * * *"
  prompt: "Check all dependencies for security vulnerabilities and outdated versions"
  skills: ["dependency-audit"]
  deliver: "slack"
  deliver_chat_id: "#security"
```

### Database Backup Verification
```yaml
- name: "backup-verify"
  schedule: "0 6 * * *"
  prompt: "Verify database backups completed successfully and test restore"
  deliver: "email"
  deliver_chat_id: "ops@company.com"
```

## Management

```bash
# Start the gateway (includes cron scheduler)
effin gateway run

# View scheduled jobs
effin status --cron

# Manually trigger a job
effin cron run <job-name>
```

## Best Practices
- Use descriptive job names for easy identification
- Set appropriate schedules (avoid unnecessary frequency)
- Route critical alerts to immediate channels (Slack/Telegram)
- Route reports to async channels (email)
- Include specific skills for specialized tasks
- Test prompts manually before scheduling
