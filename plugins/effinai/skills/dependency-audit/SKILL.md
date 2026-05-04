---
name: dependency-audit
description: "Dependency vulnerability and update check with upgrade recommendations"
version: "1.0.0"
tags: [engineering, dependencies, security, maintenance]
createdBy: "built-in"
status: "active"
---

# Dependency Audit

## Activation
When the user asks to check dependencies, update packages, or audit for vulnerable packages.

## Workflow

### Step 1: Detect Package Manager
```bash
# Detect ecosystem
ls package.json package-lock.json yarn.lock pnpm-lock.yaml Pipfile requirements.txt Gemfile go.mod Cargo.toml 2>/dev/null
```

### Step 2: Vulnerability Scan
**Node.js:**
```bash
npm audit 2>/dev/null
npm outdated 2>/dev/null
```

**Python:**
```bash
pip list --outdated 2>/dev/null
pip-audit 2>/dev/null
```

### Step 3: Analyze Results

For each dependency, assess:

| Package | Current | Latest | Severity | Breaking Changes? | Action |
|---------|---------|--------|----------|--------------------|--------|
| [pkg] | [ver] | [ver] | Critical/High/Med/Low/None | Yes/No | Update/Pin/Replace/Ignore |

### Step 4: Categorize Updates

**Immediate (Security Critical)**
- Packages with known CVEs rated Critical or High
- Provide the exact update command

**Recommended (Major Updates)**
- Major version bumps with breaking changes
- List the breaking changes from changelogs
- Provide migration steps

**Routine (Minor/Patch)**
- Safe to update in batch
- Provide bulk update command

**Deprecated Packages**
- Packages no longer maintained
- Suggest modern alternatives

### Step 5: Check for Unused Dependencies
```bash
# Node.js - check for unused deps
npx depcheck 2>/dev/null || echo "Install depcheck for unused dependency detection"
```

### Step 6: Generate Update Plan
```
## Dependency Audit Report

### Vulnerabilities Found
| Package | Severity | CVE | Fix Version |
|---------|----------|-----|-------------|

### Update Plan
1. **Immediate**: `npm update pkg1 pkg2` (security fixes)
2. **This Sprint**: Upgrade [pkg] from v2 to v3 (breaking changes: [list])
3. **Backlog**: Evaluate replacing [deprecated-pkg] with [alternative]

### Unused Dependencies (safe to remove)
- [pkg]: Not imported anywhere

### Recommendations
- [ ] Set up automated dependency updates (Dependabot/Renovate)
- [ ] Pin exact versions in production
- [ ] Add `npm audit` to CI pipeline
```

## Rules
- Never auto-update major versions without user confirmation
- Always check for breaking changes before recommending major updates
- Consider the project's Node/Python/etc version compatibility
- Recommend Dependabot or Renovate for ongoing maintenance
- Check if lock files are committed (they should be)
