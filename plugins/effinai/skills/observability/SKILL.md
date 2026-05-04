---
name: observability
description: "Design comprehensive observability strategies including SLI/SLO frameworks, golden signals monitoring, alert optimization, dashboard generation, distributed tracing, and log aggregation patterns."
version: "1.0.0"
tags: [observability, monitoring, slo, alerting, dashboards, tracing, logging]
createdBy: "built-in"
status: "active"
---

# Observability Designer

## Overview

Design production-ready observability strategies that provide deep insights into system behavior, performance, and reliability. Combines the three pillars (metrics, logs, traces) with SLI/SLO design, golden signals monitoring, and alert optimization.

## Activation

When the user asks about monitoring strategy, SLOs, alert design, dashboard creation, distributed tracing, or log aggregation.

## SLI/SLO/SLA Framework

- **SLI (Service Level Indicators):** Measurable signals indicating service health
- **SLO (Service Level Objectives):** Reliability targets based on user experience
- **SLA (Service Level Agreements):** Customer-facing commitments with consequences
- **Error Budget:** Calculate and track error budget consumption
- **Burn Rate Alerting:** Multi-window burn rate alerts for proactive SLO protection

## Three Pillars of Observability

### Metrics
- **Golden Signals:** Latency, traffic, errors, saturation
- **RED Method:** Rate, Errors, Duration for request-driven services
- **USE Method:** Utilization, Saturation, Errors for resource monitoring
- **Business Metrics:** Revenue, user engagement, feature adoption

### Logs
- **Structured Logging:** JSON-based with consistent fields
- **Log Levels:** Appropriate DEBUG, INFO, WARN, ERROR, FATAL usage
- **Correlation IDs:** Request tracing through distributed systems
- **Log Sampling:** Volume management for high-throughput systems

### Traces
- **Distributed Tracing:** End-to-end request flow visualization
- **Span Design:** Meaningful boundaries and metadata
- **Trace Sampling:** Head-based, tail-based, adaptive sampling
- **Service Maps:** Automatic dependency discovery

## Dashboard Design Principles

### Information Architecture
- **Hierarchy:** Overview -> Service -> Component -> Instance drill-down
- **Golden Ratio:** 80% operational metrics, 20% exploratory
- **Cognitive Load:** Maximum 7+/-2 panels per dashboard screen
- **User Journey:** Role-based personas (SRE, Developer, Executive)

### Visualization Best Practices
- Time series for trends, heatmaps for distributions, gauges for status
- Red for critical, amber for warning, green for healthy
- Reference lines for SLO targets, capacity thresholds, historical baselines
- Default to meaningful windows (4h for incidents, 7d for trends)

## Alert Design and Optimization

### Alert Classification
- **Critical:** Service down, SLO burn rate high
- **Warning:** Approaching thresholds, non-user-facing issues
- **Info:** Deployment notifications, capacity planning

### Alert Fatigue Prevention
- High precision (few false positives) over high recall
- Hysteresis: different thresholds for firing and resolving
- Suppression during known outages
- Related alerts grouped into single notifications

### Alert Rule Design
- Statistical methods for threshold determination
- Appropriate averaging windows and percentile calculations
- Clear firing conditions and automatic resolution criteria
- Validation against historical data

## Golden Signals Framework

### Latency
- P50, P95, P99 response time tracking
- Queue latency, network latency, database latency

### Traffic
- Requests per second with burst detection
- Bandwidth, active user sessions, feature usage

### Errors
- 4xx/5xx tracking, error budget consumption
- Error type classification, silent failures detection

### Saturation
- CPU, memory, disk, network utilization
- Queue depth, connection pool saturation, rate limiting

## Cost Optimization

- **Metric Retention:** Tiered retention based on importance
- **Log Sampling:** Intelligent sampling to reduce ingestion costs
- **Cardinality Management:** High-cardinality metric detection
- **Query Efficiency:** Optimized metric and log queries

## Scripts

| Script | Purpose |
|--------|---------|
| `slo_designer.py` | Generate SLI/SLO frameworks from service description |
| `alert_optimizer.py` | Analyze and optimize existing alert configurations |
| `dashboard_generator.py` | Create Grafana-compatible dashboard specs |

## Success Metrics

| Metric | Description |
|--------|-------------|
| **MTTD** | Mean Time to Detection |
| **MTTR** | Mean Time to Resolution |
| **Alert Precision** | Percentage of actionable alerts |
| **SLO Achievement** | Percentage of targets met consistently |

## Integration Patterns

- **Prometheus/Grafana:** Metric collection and dashboards
- **Elasticsearch/Kibana:** Log analysis
- **Jaeger/Zipkin:** Distributed tracing
- **PagerDuty/Slack:** Alert routing and escalation
- **CI/CD:** Deployment correlation, canary monitoring
