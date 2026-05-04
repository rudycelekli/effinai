---
name: changelog
description: "Auto-generate changelogs from git history using conventional commits"
version: "1.0.0"
tags: [documentation, changelog, git, release]
createdBy: "built-in"
status: "active"
---

# Changelog Generator

## Activation
When the user asks to generate a changelog, prepare release notes, or document recent changes.

## Workflow

### Step 1: Determine Scope
```bash
# Find last tag/release
git describe --tags --abbrev=0 2>/dev/null || echo "No tags found"

# Get commits since last tag (or all commits)
git log $(git describe --tags --abbrev=0 2>/dev/null || echo "")..HEAD --oneline --no-merges
```

### Step 2: Categorize Commits
Parse commits into categories using conventional commit prefixes:

| Prefix | Category | Emoji |
|--------|----------|-------|
| feat: | Features | n/a |
| fix: | Bug Fixes | n/a |
| perf: | Performance | n/a |
| docs: | Documentation | n/a |
| refactor: | Code Changes | n/a |
| test: | Tests | n/a |
| chore: | Maintenance | n/a |
| BREAKING CHANGE: | Breaking Changes | n/a |

If commits don't follow conventional commits, infer category from the diff content.

### Step 3: Generate Changelog

```markdown
# Changelog

## [X.Y.Z] - YYYY-MM-DD

### Breaking Changes
- Description of breaking change (#PR)

### Features
- Description of new feature (#PR)

### Bug Fixes
- Description of fix (#PR)

### Performance
- Description of optimization (#PR)

### Other Changes
- Refactoring, docs, tests, chores
```

### Step 4: Determine Version Bump
Based on the changes:
- **MAJOR** (X.0.0): Breaking changes present
- **MINOR** (0.X.0): New features, no breaking changes
- **PATCH** (0.0.X): Bug fixes only

### Step 5: Update Files
1. Prepend to CHANGELOG.md (create if doesn't exist)
2. Update version in package.json if applicable
3. Suggest git tag command:
   ```bash
   git tag -a vX.Y.Z -m "Release X.Y.Z"
   ```

## Rules
- Keep entries concise — one line per change
- Link to PRs/issues when available
- Group related changes together
- Include contributor attribution for external contributions
- Follow Keep a Changelog format (https://keepachangelog.com)
- Only create/update CHANGELOG.md when the user explicitly requests it
