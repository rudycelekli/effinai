---
name: shell-hooks
description: "Lifecycle hooks system — run shell scripts on pre/post tool calls, LLM calls, session events, and API requests with allowlisting and security controls"
version: "1.0.0"
tags: [hooks, automation, lifecycle, security, extensibility]
createdBy: "built-in"
status: "active"
---

# Shell Hooks

## Activation
When the user wants to run custom scripts at specific lifecycle events (before/after tool calls, LLM calls, session start/end, API requests) or when managing hook configuration and security.

## Concept

Shell hooks are user-defined scripts that fire at lifecycle events. They receive JSON payloads via stdin and can influence agent behavior by returning structured JSON to stdout. Security is enforced via an allowlist — hooks must be explicitly approved before they fire at runtime.

## Supported Events

| Event | When It Fires | Payload Includes |
|-------|---------------|------------------|
| `pre_tool_call` | Before a tool executes | tool_name, args, session_id |
| `post_tool_call` | After a tool completes | tool_name, args, result, duration_ms |
| `pre_llm_call` | Before LLM API call | session_id, user_message, model |
| `post_llm_call` | After LLM API response | session_id, model |
| `pre_api_request` | Before API request | model, provider, approx_input_tokens |
| `post_api_request` | After API response | usage, finish_reason, duration |
| `on_session_start` | Session begins | session_id |
| `on_session_end` | Session ends | session_id |
| `on_session_reset` | Session resets | session_id |
| `subagent_stop` | Child agent completes | parent_session_id, child_summary |

## Configuration

Define hooks in `config.yaml`:

```yaml
hooks:
  - event: pre_tool_call
    command: ./hooks/audit-tool-use.sh
    matcher: "terminal"          # Only fire for terminal tool
    timeout: 5                   # Seconds

  - event: post_llm_call
    command: ./hooks/log-usage.sh
    timeout: 10

  - event: on_session_start
    command: ./hooks/load-context.sh
    timeout: 3
```

## Hook Management Commands

### List All Hooks
```bash
hooks list
```
Shows configured hooks grouped by event, with allowlist status and modification tracking.

### Test a Hook
```bash
hooks test <event> [--for-tool terminal] [--payload-file custom.json]
```
Fires hooks with synthetic payload matching the real format. Useful for development.

### Revoke Allowlist Entry
```bash
hooks revoke <command>
```
Removes a previously approved hook from the allowlist. The hook will not fire until re-approved.

### Health Check
```bash
hooks doctor
```
Checks all configured hooks for:
1. Script exists and is executable
2. Listed in allowlist
3. Script unchanged since approval (mtime drift detection)
4. Produces valid JSON on synthetic payload (only if already approved)

## Security Model

### Allowlist
Hooks are stored in `shell-hooks-allowlist.json`:
```json
{
  "approvals": [
    {
      "event": "pre_tool_call",
      "command": "./hooks/audit.sh",
      "approved_at": "2024-01-15T10:30:00Z",
      "script_mtime_at_approval": "2024-01-15T09:00:00Z"
    }
  ]
}
```

### Modification Tracking
If a script's modification time changes after approval, the doctor warns:
```
Script modified since approval (was 2024-01-15, now 2024-01-20)
— review changes, then revoke + re-approve to refresh
```

### First Approval
On first encounter, the hook triggers a TTY prompt:
```
New shell hook detected: ./hooks/audit.sh
  Event: pre_tool_call
  Review the script before approving.
Approve? [y/N]:
```

Or approve all at once: `--accept-hooks`

## Hook Script Pattern

Scripts receive JSON on stdin and output JSON on stdout:

```bash
#!/bin/bash
# hooks/audit-tool-use.sh

# Read payload from stdin
PAYLOAD=$(cat)
TOOL_NAME=$(echo "$PAYLOAD" | jq -r '.tool_name')
COMMAND=$(echo "$PAYLOAD" | jq -r '.args.command // empty')

# Log the tool use
echo "[$(date)] Tool: $TOOL_NAME Command: $COMMAND" >> /tmp/audit.log

# Return structured response (or empty for observer-only)
echo '{"action": "allow"}'
```

## Use Cases

- **Audit Logging**: Record all tool executions and LLM calls
- **Cost Tracking**: Log token usage per session from post_api_request
- **Security Gates**: Block dangerous commands in pre_tool_call
- **Context Loading**: Inject project context on session_start
- **Notifications**: Alert on specific events via external services
- **Metrics Collection**: Feed usage data to monitoring systems
