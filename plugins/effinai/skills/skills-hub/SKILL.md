---
name: skills-hub
description: "Unified skill marketplace — search, install, publish, audit, and manage skills from multiple registries with security scanning"
version: "1.0.0"
tags: [skills, marketplace, registry, management, security]
createdBy: "built-in"
status: "active"
---

# Skills Hub

## Activation
When the user wants to discover, install, publish, update, or manage skills from external registries and marketplaces.

## Concept

A unified interface for skill lifecycle management across multiple registries. Skills are fetched, quarantined, security-scanned, and installed with user confirmation. Trust levels (builtin, trusted, community) determine the verification workflow.

## Core Operations

### Search Skills
```
/skills search <query> [--source official|github|community] [--limit N]
```
Searches across all configured registries, returns results with name, description, source, trust level, and identifier.

### Browse Skills
```
/skills browse [--page N] [--size N] [--source all]
```
Paginated browsing of all available skills. Official skills shown first, deduplicated by name (preferring higher trust).

### Install a Skill
```
/skills install <identifier> [--category <cat>] [--force] [--name <name>]
```

**Installation pipeline:**
1. Resolve identifier (short name -> full identifier via search)
2. Fetch skill bundle from source
3. Quarantine the bundle in isolated directory
4. Run security scan (detect shell commands, suspicious patterns, injection risks)
5. Check install policy against scan results
6. Show disclaimer (official vs third-party warning)
7. Confirm with user
8. Install from quarantine to skills directory
9. Invalidate prompt cache so skill appears immediately

### Inspect Without Installing
```
/skills inspect <identifier>
```
Preview skill metadata and SKILL.md content (first 50 lines) without installing.

### List Installed Skills
```
/skills list [--source hub|builtin|local] [--enabled-only]
```
Shows all installed skills with source type, trust level, and enabled/disabled status.

### Update Skills
```
/skills check [name]     # Check for upstream updates
/skills update [name]    # Apply available updates
```

### Security Audit
```
/skills audit [name]     # Re-run security scan on installed hub skills
```

### Uninstall
```
/skills uninstall <name>
```

### Publish to Registry
```
/skills publish <path> --to github --repo owner/repo
```
Self-scans the skill, then creates a fork + PR to the target repository with all skill files.

### Snapshot Export/Import
```
/skills snapshot export <file>    # Export current config to JSON
/skills snapshot import <file>    # Re-install skills from snapshot
```
Portable skill configuration for sharing setups across machines or teams.

### Custom Sources (Taps)
```
/skills tap list              # List configured taps
/skills tap add owner/repo    # Add custom GitHub repo source
/skills tap remove owner/repo # Remove a tap
```

## Security Model

### Trust Levels
- **builtin** (bright_cyan): Shipped with the system, always available
- **trusted** (green): From verified/official sources
- **community** (yellow): Third-party, requires user review

### Security Scan
Every installed skill goes through:
1. Quarantine in isolated directory
2. Automated security scanning for:
   - Shell command injection
   - Suspicious file patterns
   - Prompt injection attempts
   - Unsafe code patterns
3. Verdict: safe/suspicious/dangerous
4. Dangerous verdict blocks installation entirely

### Third-Party Warning
```
You are installing a third-party skill at your own risk.
External skills can contain instructions that influence agent behavior,
shell commands, and scripts. Review the installed files before use.
```

## Skill Format
Each skill is a directory containing at minimum a `SKILL.md` file with YAML frontmatter:
```yaml
---
name: skill-name
description: "What this skill does"
---
```

## Best Practices
- Start with official/trusted skills before community ones
- Always review the SKILL.md preview before installing
- Run periodic audits on installed hub skills
- Use snapshots to replicate setups across environments
- Add custom taps for team-internal skill registries
