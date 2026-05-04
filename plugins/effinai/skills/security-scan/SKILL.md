---
name: security-scan
description: "Security vulnerability scanning aligned with OWASP Top 10"
version: "1.0.0"
tags: [engineering, security, owasp, audit]
createdBy: "built-in"
status: "active"
---

# Security Scan

## Activation
When the user asks to scan for security issues, perform a security audit, or check for vulnerabilities.

## Workflow

### Step 1: Scope the Scan
Determine scan target:
- Full codebase scan
- Specific module or feature
- API endpoints only
- Authentication/authorization flow

### Step 2: OWASP Top 10 Checks

**A01: Broken Access Control**
- Missing authorization checks on endpoints
- Direct object references without ownership validation
- CORS misconfiguration (wildcard origins with credentials)
- Missing CSRF protection on state-changing operations
- Path traversal via user input in file operations
- Search for: `req.params`, `req.query` used in file paths or DB queries without validation

**A02: Cryptographic Failures**
- Hardcoded secrets, API keys, passwords in source code
- Weak hashing (MD5, SHA1 for passwords — should use bcrypt/argon2)
- Missing HTTPS enforcement
- Sensitive data in logs or error messages
- Weak random number generation (Math.random for security purposes)
- Search for: `password`, `secret`, `key`, `token`, `MD5`, `SHA1`

**A03: Injection**
- SQL injection: String concatenation in queries
- NoSQL injection: Unsanitized objects in MongoDB queries
- Command injection: User input in exec/spawn calls
- XSS: Unescaped user input in HTML output, dangerouslySetInnerHTML
- Template injection: User input in template strings
- Search for: `exec(`, `eval(`, `innerHTML`, `dangerouslySetInnerHTML`, raw SQL strings

**A04: Insecure Design**
- Missing rate limiting on authentication endpoints
- No account lockout after failed attempts
- Predictable resource IDs (sequential integers)
- Missing input validation schemas

**A05: Security Misconfiguration**
- Default credentials or configurations
- Verbose error messages in production
- Unnecessary HTTP methods enabled
- Missing security headers (CSP, HSTS, X-Frame-Options)
- Debug mode enabled in production config

**A06: Vulnerable Components**
```bash
npm audit 2>/dev/null
pip-audit 2>/dev/null
```

**A07: Authentication Failures**
- Weak password policies
- Missing MFA support
- Session tokens in URLs
- No session expiration/rotation
- JWT without expiration or with weak signing

**A08: Data Integrity Failures**
- Deserialization of untrusted data
- Missing integrity checks on downloads/updates
- CI/CD pipeline without artifact verification

**A09: Logging & Monitoring Failures**
- Authentication events not logged
- Missing audit trail for sensitive operations
- PII in logs without masking
- No alerting on suspicious activity

**A10: Server-Side Request Forgery (SSRF)**
- User-supplied URLs fetched server-side without validation
- Internal network access from user input
- DNS rebinding vulnerabilities

### Step 3: Automated Checks
```bash
# Check for secrets in git history
git log --all --diff-filter=A --name-only --format="" | head -50

# Check for .env files committed
git ls-files | grep -i "\.env"

# Check npm audit
npm audit --json 2>/dev/null | head -50

# Check for TODO/FIXME security items
grep -rn "TODO.*secur\|FIXME.*secur\|HACK.*auth" --include="*.ts" --include="*.js" . 2>/dev/null | head -20
```

### Step 4: Output Report

```
## Security Scan Report

### Critical Vulnerabilities
- [OWASP-ID] [File:Line] Description
  Impact: [what could go wrong]
  Fix: [specific remediation]

### High Severity
- ...

### Medium Severity
- ...

### Low Severity / Informational
- ...

### Scan Summary
- Files scanned: [N]
- Critical: [N] | High: [N] | Medium: [N] | Low: [N]
- Recommended immediate actions: [list]
```

## Rules
- NEVER expose actual secrets found — redact them in output
- Prioritize exploitable vulnerabilities over theoretical risks
- Provide specific fix code, not just descriptions
- Check .env.example for what secrets are expected
- Consider the threat model (public API vs internal tool)
