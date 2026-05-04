---
name: federation-protocol
description: "Multi-org agent federation with trust levels, PII gating, and compliance modes"
version: "1.0.0"
tags: [federation, security, compliance, multi-org]
---

# Federation Protocol

Enable agents from different organizations to collaborate securely with graduated trust, PII protection, and regulatory compliance.

## Trust Levels

Agents operate at one of four trust levels, each with specific capabilities:

| Level | Name | Capabilities | Verification Required |
|-------|------|-------------|----------------------|
| 0 | **Untrusted** | Read public data, submit requests for review | None |
| 1 | **Verified** | Read shared data, execute pre-approved tasks | Identity verification + org validation |
| 2 | **Trusted** | Read/write shared data, spawn sub-agents within scope | Verified + security audit passed |
| 3 | **Privileged** | Full access within federation scope, approve trust promotions | Trusted + manual approval by org admin |

### Trust Promotion

```
Untrusted --[identity check]--> Verified --[security audit]--> Trusted --[admin approval]--> Privileged
```

**Demotion** is immediate: any policy violation drops the agent to Untrusted and triggers an incident report.

### Trust Verification Payload

```typescript
interface TrustCertificate {
  agentId: string;
  orgId: string;
  trustLevel: 0 | 1 | 2 | 3;
  issuedAt: string;
  expiresAt: string;          // Trust certificates expire (max 24h)
  capabilities: string[];     // Explicit list of allowed actions
  restrictions: string[];     // Explicit list of denied actions
  signature: string;          // Signed by issuing org's key
}
```

## Cross-Org Agent Communication

### Message Envelope

All cross-org messages use a standardized envelope:

```typescript
interface FederatedMessage {
  id: string;
  from: { agentId: string; orgId: string; trustLevel: number };
  to: { agentId: string; orgId: string };
  payload: EncryptedPayload;    // Encrypted with recipient's public key
  piiScan: PIIScanResult;       // Pre-scanned before transmission
  timestamp: string;
  ttl: number;                  // Message expires after TTL seconds
  replyTo?: string;             // For request-response patterns
}
```

### Communication Patterns

**Request-Response**: Agent A sends a request, Agent B responds. Both messages are logged and auditable.

**Event Broadcast**: An org publishes events to a shared topic. Subscribers receive events filtered by their trust level.

**Task Delegation**: A privileged agent delegates a task to a verified agent in another org. The task runs in a sandboxed scope with explicit resource limits.

### Message Routing Rules

| Sender Trust | Receiver Trust | Allowed |
|-------------|---------------|---------|
| Untrusted | Any | Only via approval queue |
| Verified | Verified+ | Direct, read-only payloads |
| Trusted | Trusted+ | Direct, read/write payloads |
| Privileged | Any | Direct, full capability |

## PII Detection and Gating

Before any data crosses org boundaries, it passes through the PII detection pipeline (see `pii-detection` skill for full patterns).

**Federation PII Policy**:
- Trust Level 0-1: All PII is BLOCKED from crossing boundaries
- Trust Level 2: PII is REDACTED (replaced with placeholders)
- Trust Level 3: PII is HASHED (deterministic for cross-referencing, not reversible)
- No trust level permits raw PII transmission by default

Override requires explicit compliance mode configuration (see below).

## Compliance Modes

### HIPAA Mode

```yaml
compliance: hipaa
rules:
  - pii_policy: BLOCK           # No PHI crosses boundaries ever
  - audit_log: required         # Every access logged with retention
  - encryption: AES-256-GCM    # At rest and in transit
  - access_review: quarterly    # Mandatory access reviews
  - breach_notification: 72h    # Notify within 72 hours
  - baa_required: true          # Business Associate Agreement must exist
  - minimum_trust: 2            # Only Trusted+ agents can participate
```

### SOC 2 Mode

```yaml
compliance: soc2
rules:
  - pii_policy: REDACT
  - audit_log: required
  - encryption: TLS-1.3
  - access_review: annual
  - change_management: required  # All changes go through approval
  - incident_response: documented
  - minimum_trust: 1
```

### GDPR Mode

```yaml
compliance: gdpr
rules:
  - pii_policy: HASH
  - audit_log: required
  - encryption: AES-256-GCM
  - data_residency: EU           # Data stays in EU regions
  - right_to_erasure: supported  # Must be able to delete all user data
  - consent_tracking: required   # Record consent for data processing
  - dpo_notification: on_breach  # Notify Data Protection Officer
  - minimum_trust: 1
```

## Federation Setup

### Step 1: Register Organization

```bash
npx @claude-flow/cli@latest federation register \
  --org-id "acme-corp" \
  --public-key "./keys/acme-public.pem" \
  --compliance "soc2"
```

### Step 2: Establish Trust Relationship

```bash
npx @claude-flow/cli@latest federation trust \
  --from-org "acme-corp" \
  --to-org "partner-inc" \
  --initial-trust 1 \
  --scope "shared-project-x"
```

### Step 3: Configure Shared Scope

```yaml
federation_scope:
  name: "shared-project-x"
  orgs: ["acme-corp", "partner-inc"]
  shared_resources:
    - type: memory_namespace
      name: "project-x-shared"
      access: read-write
    - type: task_queue
      name: "project-x-tasks"
      access: submit-and-execute
  pii_policy: REDACT
  compliance: soc2
  max_agents_per_org: 5
  message_ttl: 3600
```

### Step 4: Deploy Federated Agents

```bash
npx @claude-flow/cli@latest agent spawn \
  --type coder \
  --name "acme-coder-1" \
  --federation-scope "shared-project-x" \
  --trust-level 2
```

## Audit Trail

Every federation interaction is logged:

```typescript
interface AuditEntry {
  timestamp: string;
  action: "message_sent" | "message_received" | "trust_change" | "pii_detected" | "policy_violation";
  sourceOrg: string;
  targetOrg: string;
  agentId: string;
  trustLevel: number;
  details: string;
  piiDetected: boolean;
  complianceMode: string;
}
```

Audit logs are stored in each org's own infrastructure. Shared audit summaries (without PII) are available to all federation members.

## Security Invariants

- Trust certificates are short-lived (max 24h) and must be refreshed
- Private keys never cross org boundaries
- PII scanning happens BEFORE encryption, not after
- Compliance mode cannot be downgraded without admin approval
- All federation traffic is encrypted end-to-end
- Rate limiting prevents any single org from overwhelming the federation

## Implementation Checklist

- [ ] Trust certificate issuance and validation
- [ ] Message envelope encryption and signing
- [ ] PII scanning pipeline integrated at boundary
- [ ] Compliance mode enforcement
- [ ] Audit logging for all cross-org interactions
- [ ] Trust promotion/demotion workflows
- [ ] Rate limiting per org
- [ ] Certificate expiry and renewal
