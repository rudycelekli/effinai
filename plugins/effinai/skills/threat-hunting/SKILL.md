---
name: threat-hunting
description: "Active threat detection and security hunting — vulnerability scanning, attack surface mapping, secret detection, dependency auditing, and OWASP coverage"
version: "1.0.0"
tags: [security, threats, hunting, vulnerabilities, owasp, audit]
---

# Threat Hunting

## Activation
When the agent needs to proactively search for security vulnerabilities, detect leaked secrets, audit dependencies for known CVEs, map attack surfaces, or verify compliance with security standards. Use before deployments, during security reviews, or on a scheduled basis.

## Concept

Active threat hunting goes beyond automated scanning by combining static analysis patterns with contextual understanding of the codebase. The agent systematically hunts for vulnerabilities across multiple attack categories, maps the attack surface, correlates findings, and prioritizes by exploitability. Unlike passive scanning that only flags known patterns, threat hunting investigates suspicious code paths and reasons about potential attack chains.

## Hunt Categories

### 1. Secret Detection
```
Hunt for exposed credentials, API keys, tokens, and sensitive data:

Patterns:
  - API keys: /[A-Za-z0-9_]{20,}/ in assignment context
  - AWS keys: /AKIA[0-9A-Z]{16}/
  - Private keys: /-----BEGIN (RSA |EC |DSA )?PRIVATE KEY-----/
  - JWTs: /eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+/
  - Connection strings: /(mysql|postgres|mongodb|redis):\/\/[^@]+@/
  - Generic secrets: variable names containing "secret", "password", "token", "key"

Analysis:
  - Check if secrets are in .gitignore
  - Check git history for previously committed secrets
  - Verify .env files are not tracked
  - Check for hardcoded credentials in test files (acceptable with caveats)
```

### 2. Injection Vulnerabilities
```
SQL Injection:
  - Raw SQL with string concatenation/interpolation
  - Missing parameterized queries
  - Dynamic table/column names from user input
  - ORM raw query methods

Command Injection:
  - exec/spawn/system calls with user-controlled input
  - Shell command construction with string interpolation
  - Missing input sanitization before shell execution

XSS (Cross-Site Scripting):
  - innerHTML/dangerouslySetInnerHTML with user data
  - Unescaped template variables
  - DOM manipulation with unsanitized input
  - Missing Content-Security-Policy headers

Path Traversal:
  - File operations with user-controlled paths
  - Missing path normalization/validation
  - Relative path resolution without restriction
```

### 3. Authentication & Authorization
```
Hunt for:
  - Endpoints missing authentication middleware
  - Hardcoded admin credentials
  - Broken access control (horizontal/vertical privilege escalation)
  - Missing CSRF protection on state-changing endpoints
  - Session fixation vulnerabilities
  - Weak password policies
  - Missing rate limiting on auth endpoints
  - JWT without expiration or with weak signing algorithm
  - Token storage in localStorage (XSS-accessible)
```

### 4. Dependency Vulnerabilities
```
Hunt for:
  - Known CVEs in dependencies (npm audit, cargo audit, pip audit)
  - Outdated dependencies with known vulnerabilities
  - Typosquatting risk (package names similar to popular packages)
  - Dependency confusion (private vs public registry)
  - Unnecessary dependencies that increase attack surface
  - Dependencies with no maintenance activity
  - Transitive vulnerabilities (dependency of a dependency)
```

### 5. Data Exposure
```
Hunt for:
  - PII in logs (email, phone, SSN, credit card)
  - Sensitive data in error messages returned to clients
  - Verbose error responses in production config
  - Missing data encryption at rest
  - Missing TLS enforcement
  - Debug endpoints exposed in production
  - Stack traces in API responses
  - Database dumps or backups in accessible locations
```

### 6. Infrastructure & Configuration
```
Hunt for:
  - CORS misconfiguration (wildcard origins with credentials)
  - Missing security headers (HSTS, X-Frame-Options, CSP)
  - Debug mode enabled in production config
  - Default credentials in configuration files
  - Exposed admin panels or management endpoints
  - Missing request size limits
  - Missing timeout configurations
  - Insecure TLS versions allowed
```

## Attack Surface Mapping

### Process
```
1. Enumerate all entry points:
   - HTTP/API endpoints
   - WebSocket handlers
   - Message queue consumers
   - Cron jobs
   - CLI commands
   - File upload handlers

2. For each entry point, document:
   - Input sources (query params, body, headers, cookies)
   - Authentication requirements
   - Authorization level
   - Input validation applied
   - Data stores accessed
   - External services called

3. Rank by risk:
   - Unauthenticated + user input + database access = CRITICAL
   - Authenticated + user input + external call = HIGH
   - Internal only + validated input = LOW
```

### Output Format
```
Attack Surface Map:
  POST /api/auth/login        [CRITICAL] Unauthenticated, handles credentials
    Inputs: email (body), password (body)
    Validates: email format
    Missing: rate limiting, account lockout
    Stores: users (read), sessions (write), audit_log (write)
    
  GET /api/users/:id          [HIGH] Authenticated, path parameter
    Inputs: id (path param)
    Validates: none
    Missing: authorization check (any user can access any id)
    Stores: users (read)
    
  POST /api/upload             [CRITICAL] Authenticated, file upload
    Inputs: file (multipart)
    Validates: file size (10MB limit)
    Missing: file type validation, virus scan, path sanitization
    Stores: filesystem (write)
```

## OWASP Top 10 Coverage

| # | Category | Hunts |
|---|----------|-------|
| A01 | Broken Access Control | Auth bypass, IDOR, missing ACLs |
| A02 | Cryptographic Failures | Weak algorithms, missing encryption, key exposure |
| A03 | Injection | SQLi, XSS, command injection, LDAP injection |
| A04 | Insecure Design | Missing threat modeling, business logic flaws |
| A05 | Security Misconfiguration | Default configs, debug mode, missing headers |
| A06 | Vulnerable Components | CVE scanning, outdated dependencies |
| A07 | Auth Failures | Credential stuffing, session management, MFA bypass |
| A08 | Data Integrity Failures | Deserialization, CI/CD pipeline security |
| A09 | Logging Failures | Missing audit logs, PII in logs |
| A10 | SSRF | Server-side request forgery, internal network access |

## Hunt Report Format

```
Threat Hunt Report
==================
Date: 2026-05-04
Scope: /projects/myapp
Duration: 45 seconds

CRITICAL (2):
  [THR-001] Hardcoded API key in src/config/stripe.ts:15
    Impact: Financial data exposure
    Evidence: STRIPE_SECRET_KEY = "sk_live_..."
    Remediation: Move to environment variable, rotate key immediately
    
  [THR-002] SQL injection in src/api/users.ts:42
    Impact: Full database access
    Evidence: `SELECT * FROM users WHERE id = ${req.params.id}`
    Remediation: Use parameterized query

HIGH (3):
  [THR-003] Missing authorization check on GET /api/users/:id
  [THR-004] No rate limiting on POST /api/auth/login
  [THR-005] JWT signed with HS256 (symmetric, shared secret)

MEDIUM (5):
  ...

LOW (8):
  ...

Coverage: 8/10 OWASP categories checked
Attack surface: 23 endpoints mapped, 4 high-risk
```

## Workflow

### Full Security Hunt
```
1. Map attack surface (enumerate all entry points)
2. Scan for secrets (git history + current files)
3. Audit dependencies (CVE database check)
4. Hunt injection vulnerabilities (pattern + context analysis)
5. Check auth/authz (middleware coverage analysis)
6. Review configuration (headers, CORS, TLS)
7. Correlate findings (attack chains)
8. Generate prioritized report
```

### Pre-Deployment Quick Hunt
```
1. Scan for secrets (current files only)
2. Check for critical CVEs in dependencies
3. Verify auth middleware on all endpoints
4. Check security headers in production config
5. Generate pass/fail report
```

## Integration

- **Knowledge Graph**: Map which endpoints connect to which data stores for impact analysis
- **Agent Grep**: Pattern-based vulnerability detection across the codebase
- **Code Review**: Include security hunt findings in PR reviews
- **Ambient Agent**: Schedule periodic threat hunts between sessions
