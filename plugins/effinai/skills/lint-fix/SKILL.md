---
name: lint-fix
description: "Auto-fix linting issues across the codebase"
version: "1.0.0"
tags: [quality, linting, formatting, code-style]
createdBy: "built-in"
status: "active"
---

# Lint Fix

## Activation
When the user asks to fix linting errors, format code, or enforce code style.

## Workflow

### Step 1: Detect Linting Setup
```bash
# Check for linting config
ls .eslintrc* eslint.config.* .prettierrc* biome.json .pylintrc .flake8 .rubocop.yml 2>/dev/null

# Check package.json for lint scripts
cat package.json | grep -E '"lint"|"format"|"eslint"|"prettier"' 2>/dev/null
```

### Step 2: Run Linter to Assess Scope
```bash
# ESLint
npx eslint . --ext .ts,.tsx,.js,.jsx 2>&1 | tail -20

# or Biome
npx biome check . 2>&1 | tail -20

# Prettier check
npx prettier --check "src/**/*.{ts,tsx}" 2>&1 | tail -10
```

### Step 3: Auto-Fix What's Possible
```bash
# ESLint auto-fix
npx eslint . --ext .ts,.tsx,.js,.jsx --fix

# Prettier format
npx prettier --write "src/**/*.{ts,tsx,js,jsx,json,css,md}"

# Biome auto-fix
npx biome check --apply .
```

### Step 4: Handle Remaining Issues
For issues that can't be auto-fixed:
1. Group by rule type
2. Present to user with examples:
   ```
   ## Manual Fixes Required

   ### no-explicit-any (15 occurrences)
   - src/api/handler.ts:23 — Replace `any` with proper type
   - src/utils/parse.ts:45 — Replace `any` with `unknown`

   ### prefer-const (3 occurrences)
   - [files and fixes]
   ```
3. Offer to fix them manually if straightforward

### Step 5: Verify Clean State
```bash
# Run linter again — should be clean
npx eslint . --ext .ts,.tsx,.js,.jsx 2>&1 | tail -5
npx prettier --check "src/**/*.{ts,tsx}" 2>&1 | tail -5
```

### Step 6: Prevent Future Issues
Recommend if not present:
- Pre-commit hook (husky + lint-staged)
- CI lint check
- Editor settings (.editorconfig, .vscode/settings.json)

## Rules
- Always run auto-fix first before manual fixes
- Don't change linting rules to suppress errors (unless genuinely wrong)
- Respect the project's existing code style configuration
- If no linter is configured, suggest setting one up before fixing
- Group similar issues together for efficient fixing
- Run tests after lint fixes to ensure no behavioral changes
