---
name: effinai-security
description: >
  Security scanning, PII detection, compliance checking (GDPR/SOC2/HIPAA), threat 
  modeling, and audit logging. Use when performing security audits, scanning for 
  vulnerabilities, or checking compliance.
argument-hint: "[scan|pii|compliance|threat-model]"
allowed-tools: |
  Bash(npx effinai*)
  Bash(effin *)
  Read
  Glob
  Grep
  mcp__effinai__effinai_security_scan
  mcp__effinai__effinai_pii_detect
  mcp__effinai__effinai_compliance_check
  mcp__effinai__effinai_threat_model
  mcp__effinai__effinai_audit_log
metadata:
  priority: 8
  promptSignals:
    phrases:
      - "security scan"
      - "vulnerability"
      - "PII"
      - "compliance"
      - "OWASP"
      - "threat model"
      - "audit"
    anyOf:
      - "security"
      - "vulnerability"
      - "PII"
      - "compliance"
      - "OWASP"
      - "threat"
    minScore: 4
---

# Effin.AI Security & Compliance

## Capabilities

### OWASP Top 10 Scanning
- SQL injection, XSS, CSRF detection
- Authentication/authorization weaknesses
- Security misconfiguration
- Cryptographic failures

### PII Detection
- Email, phone, SSN, API keys
- Credit card numbers
- Social security numbers
- Custom pattern matching

### Compliance Checking
- GDPR compliance validation
- SOC2 requirements check
- HIPAA compliance audit

### Threat Modeling
- Automated threat model generation
- Attack surface analysis
- Risk scoring

### Audit Logging
- Immutable audit trail entries
- Compliance-ready logs

## MCP Tools (5)
- `effinai_security_scan` — OWASP Top 10 scan
- `effinai_pii_detect` — PII detection
- `effinai_compliance_check` — GDPR/SOC2/HIPAA check
- `effinai_threat_model` — Generate threat model
- `effinai_audit_log` — Write audit trail entry
