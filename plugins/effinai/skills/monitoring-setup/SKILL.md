---
name: monitoring-setup
description: "Set up monitoring, logging, and alerting for applications"
version: "1.0.0"
tags: [devops, monitoring, logging, alerting, observability]
createdBy: "built-in"
status: "active"
---

# Monitoring Setup

## Activation
When the user asks to add monitoring, set up logging, create alerts, or improve observability.

## Workflow

### Step 1: Assess Current State
1. Check for existing monitoring (Prometheus, Datadog, New Relic)
2. Check for structured logging (winston, pino, bunyan)
3. Check for error tracking (Sentry, Bugsnag)
4. Identify critical paths that need monitoring

### Step 2: Implement Structured Logging

```typescript
// Use pino for Node.js (fastest structured logger)
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  redact: ['req.headers.authorization', 'password', 'token'],
});

// Usage
logger.info({ userId, action: 'login' }, 'User logged in');
logger.error({ err, requestId }, 'Payment processing failed');
```

Key logging principles:
- Use structured JSON logs (not console.log strings)
- Include correlation IDs (requestId) across all logs
- Redact sensitive data (passwords, tokens, PII)
- Log at appropriate levels: error, warn, info, debug
- Include context: userId, requestId, operation name

### Step 3: Add Application Metrics

**Key metrics to track (RED method):**
- **Rate**: Requests per second
- **Errors**: Error rate / error percentage
- **Duration**: Response time (p50, p95, p99)

```typescript
// Express middleware for request metrics
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info({
      method: req.method,
      path: req.route?.path || req.path,
      statusCode: res.statusCode,
      duration,
      requestId: req.id,
    }, 'request completed');
  });
  next();
});
```

**Business metrics:**
- User signups per hour
- Orders completed / failed
- API quota usage
- Queue depth and processing time

### Step 4: Set Up Health Checks

```typescript
app.get('/health', (req, res) => {
  res.json({ status: 'ok', uptime: process.uptime() });
});

app.get('/health/ready', async (req, res) => {
  const checks = {
    database: await checkDb(),
    redis: await checkRedis(),
    externalApi: await checkExternalApi(),
  };
  const healthy = Object.values(checks).every(c => c.status === 'ok');
  res.status(healthy ? 200 : 503).json({ status: healthy ? 'ready' : 'degraded', checks });
});
```

### Step 5: Configure Alerting

Define alerts for:
| Alert | Condition | Severity |
|-------|-----------|----------|
| High Error Rate | Error rate > 5% for 5 min | Critical |
| Slow Responses | p95 > 2s for 10 min | Warning |
| Service Down | Health check failing for 1 min | Critical |
| High Memory | Memory > 85% for 5 min | Warning |
| Disk Space | Disk > 90% | Critical |
| Queue Backlog | Queue depth > 1000 for 15 min | Warning |

### Step 6: Error Tracking

```typescript
// Sentry integration
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // 10% of transactions
});

// Capture unhandled errors
app.use(Sentry.Handlers.errorHandler());
```

## Rules
- Always redact sensitive data from logs
- Use correlation IDs to trace requests across services
- Set up alerts for symptoms (high error rate) not causes (CPU spike)
- Include runbook links in alert descriptions
- Use appropriate log levels — don't log everything at INFO
- Monitor the monitoring system itself
- Keep retention costs in mind — log rotation and sampling
