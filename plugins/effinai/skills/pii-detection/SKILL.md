---
name: pii-detection
description: "PII detection and handling with regex patterns, 4 policies, and compliance integration"
version: "1.0.0"
tags: [security, privacy, pii, compliance]
---

# PII Detection

Detect personally identifiable information in text, code, and data. Apply configurable handling policies per PII type.

## PII Types and Regex Patterns

### Email Addresses
```regex
[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}
```
Example match: `user@example.com`

### Phone Numbers (US/International)
```regex
(\+?1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}
```
Example match: `+1 (555) 123-4567`, `555-123-4567`

### Social Security Numbers
```regex
\b\d{3}[-\s]?\d{2}[-\s]?\d{4}\b
```
Example match: `123-45-6789`, `123 45 6789`
Note: Validate format (no group starts with 000, 666, or 9xx).

### Credit Card Numbers
```regex
\b(?:4\d{3}|5[1-5]\d{2}|3[47]\d{2}|6(?:011|5\d{2}))[- ]?\d{4}[- ]?\d{4}[- ]?\d{1,4}\b
```
Example match: `4111-1111-1111-1111`, `5500 0000 0000 0004`
Note: Validate with Luhn algorithm after regex match.

### API Keys and Secrets
```regex
(?:sk|pk|api|key|token|secret|password|bearer)[-_]?[a-zA-Z0-9]{20,}
```
```regex
(?:ghp|gho|ghu|ghs|ghr)_[A-Za-z0-9_]{36,}
```
```regex
sk-(?:ant-)?[a-zA-Z0-9-]{20,}
```
Example match: `sk-ant-api03-xxxxx`, `ghp_xxxxxxxxxxxxxxxxxxxx`

### Passwords in Code
```regex
(?:password|passwd|pwd|secret)\s*[:=]\s*['"][^'"]{4,}['"]
```
Example match: `password = "hunter2"`, `pwd: 'my-secret'`

### Physical Addresses (US)
```regex
\d{1,5}\s\w+\s(?:St|Street|Ave|Avenue|Blvd|Boulevard|Dr|Drive|Ln|Lane|Rd|Road|Ct|Court|Way|Pl|Place)\.?(?:\s(?:Apt|Suite|Unit|#)\s?\w+)?
```
Example match: `123 Main St Apt 4`

### Person Names (heuristic)
Names are detected contextually rather than by regex alone:
- Adjacent to labels: `name:`, `patient:`, `client:`, `user:`
- In structured data: JSON/CSV fields named `firstName`, `lastName`, `fullName`
- Capitalized word pairs not matching known non-name patterns

### Dates of Birth
```regex
(?:DOB|date.of.birth|born|birthday)\s*[:=]?\s*\d{1,2}[/.-]\d{1,2}[/.-]\d{2,4}
```
Example match: `DOB: 03/15/1990`, `date_of_birth = 1990-03-15`

### IP Addresses
```regex
\b(?:(?:25[0-5]|2[0-4]\d|[01]?\d?\d)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d?\d)\b
```
Example match: `192.168.1.100`
Note: Exclude RFC 1918 private ranges and localhost if configured.

## Handling Policies

### BLOCK
Reject the entire content. Nothing passes through.
```
Input:  "Contact john@example.com for details"
Output: [BLOCKED: PII detected - email address]
```
Use when: Untrusted contexts, cross-org federation at low trust levels, HIPAA PHI.

### REDACT
Replace PII with type-labeled placeholders.
```
Input:  "Contact john@example.com or call 555-123-4567"
Output: "Contact [EMAIL_REDACTED] or call [PHONE_REDACTED]"
```
Use when: Logs, shared reports, SOC 2 compliance, moderate trust federation.

### HASH
Replace PII with deterministic hashes (allows cross-referencing without exposing data).
```
Input:  "Contact john@example.com"
Output: "Contact [EMAIL:sha256:a1b2c3d4...]"
```
Use when: Analytics, cross-org de-duplication, GDPR pseudonymization.

Implementation: `SHA-256(PII_value + org_salt)` - the salt prevents rainbow table attacks while allowing same-org matching.

### PASS
Allow PII through unchanged. Log the decision.
```
Input:  "Contact john@example.com"
Output: "Contact john@example.com"  (logged: PII passed, type=email, justification=...)
```
Use when: Internal processing within trust boundary, explicit user consent recorded.

## Policy Configuration

```yaml
pii_policies:
  default: REDACT

  by_type:
    ssn: BLOCK              # Never pass SSNs
    credit_card: BLOCK      # Never pass credit cards
    api_key: BLOCK          # Never pass secrets
    password: BLOCK         # Never pass passwords
    email: REDACT           # Redact by default
    phone: REDACT
    address: REDACT
    name: HASH              # Hash for cross-referencing
    dob: REDACT
    ip_address: PASS        # Often needed for debugging

  by_context:
    federation: BLOCK       # Strictest for cross-org
    logging: REDACT         # Redact in logs
    analytics: HASH         # Hash for analytics
    internal: PASS          # Trust internal processing

  by_compliance:
    hipaa: BLOCK            # Override everything to BLOCK
    gdpr: HASH              # GDPR pseudonymization
    soc2: REDACT            # SOC 2 data protection
```

## Detection Pipeline

```
Input Text
    |
    v
[1. Regex Scan] -- Match all PII patterns against text
    |
    v
[2. Validation] -- Luhn check for CC, format check for SSN, etc.
    |
    v
[3. Context Check] -- Is this in code? In a log? In user data?
    |
    v
[4. Policy Lookup] -- Determine handling policy based on type + context
    |
    v
[5. Apply Policy] -- BLOCK / REDACT / HASH / PASS
    |
    v
[6. Audit Log] -- Record what was detected, where, and how it was handled
    |
    v
Output Text
```

## Scan Function

```typescript
interface PIIScanResult {
  hasPII: boolean;
  findings: PIIFinding[];
  policiesApplied: string[];
  processedText: string;          // Text after policy application
}

interface PIIFinding {
  type: "email" | "phone" | "ssn" | "credit_card" | "api_key" |
        "password" | "address" | "name" | "dob" | "ip_address";
  value: string;                   // Original value (only in audit log, never returned)
  position: { start: number; end: number };
  confidence: number;              // 0.0-1.0, regex=0.9, heuristic=0.6
  policyApplied: "BLOCK" | "REDACT" | "HASH" | "PASS";
}
```

## Integration Points

### With Federation Protocol
PII scanning runs BEFORE any data crosses org boundaries. The federation trust level determines which policy applies.

### With Cost Tracking
PII scanning adds latency. Track scan time per document and optimize regex compilation (compile once, reuse).

### With Trajectory Learning
NEVER record PII in trajectory logs. Apply REDACT policy to all trajectory data before storage.

### CLI Integration
```bash
# Scan a file for PII
npx @claude-flow/cli@latest security scan --target ./data/export.json --pii-check

# Configure PII policies
npx @claude-flow/cli@latest config set pii.default_policy REDACT
npx @claude-flow/cli@latest config set pii.ssn_policy BLOCK
```

## Testing PII Detection

Test each pattern with true positives, true negatives, and edge cases:

| Type | True Positive | True Negative | Edge Case |
|------|--------------|---------------|-----------|
| Email | user@example.com | user@localhost | user+tag@sub.example.co.uk |
| Phone | 555-123-4567 | 123-456 | +44 20 7946 0958 |
| SSN | 123-45-6789 | 000-00-0000 | 078-05-1120 (invalid) |
| CC | 4111111111111111 | 1234567890123456 | 4111-1111-1111-1111 |
| API Key | sk-ant-api03-xxx | sk-short | ghp_xxxxxxxxxxxxxxxx |

## Implementation Checklist

- [ ] All 10 PII type patterns implemented and tested
- [ ] Luhn validation for credit cards
- [ ] SSN format validation (exclude invalid ranges)
- [ ] All 4 policies (BLOCK, REDACT, HASH, PASS) implemented
- [ ] Policy configuration per type, context, and compliance mode
- [ ] Audit logging for every PII detection event
- [ ] Performance: <10ms scan time for documents under 10KB
- [ ] False positive rate below 5% on test corpus
