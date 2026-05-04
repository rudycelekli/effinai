---
name: type-safety
description: "Add or improve TypeScript type safety across the codebase"
version: "1.0.0"
tags: [quality, typescript, types, safety]
createdBy: "built-in"
status: "active"
---

# Type Safety

## Activation
When the user asks to improve type safety, remove `any` types, add TypeScript types, or fix type errors.

## Workflow

### Step 1: Assess Current State
```bash
# Count type issues
npx tsc --noEmit 2>&1 | tail -5

# Find 'any' usage
grep -rn ": any\|as any\|<any>" --include="*.ts" --include="*.tsx" src/ | wc -l

# Check tsconfig strictness
cat tsconfig.json | grep -E "strict|noImplicit|strictNull"
```

### Step 2: Prioritize Fixes

**High Priority (fix first):**
- `any` in function parameters and return types (public API surface)
- `any` in data models and interfaces
- Missing return types on exported functions
- Type assertions (`as any`, `as unknown as X`)
- `@ts-ignore` / `@ts-expect-error` without explanation

**Medium Priority:**
- Implicit `any` from untyped dependencies
- Generic functions without constraints
- Union types that should be discriminated
- Missing nullability annotations

**Low Priority:**
- Internal utility functions with inferred types
- Test files (unless testing types specifically)
- Generated code

### Step 3: Fix Patterns

**Replace `any` with proper types:**
```typescript
// Before
function process(data: any): any { ... }

// After
interface ProcessInput { id: string; values: number[] }
interface ProcessResult { success: boolean; output: string }
function process(data: ProcessInput): ProcessResult { ... }
```

**Add discriminated unions:**
```typescript
// Before
interface ApiResponse { status: string; data?: any; error?: string }

// After
type ApiResponse =
  | { status: 'success'; data: ResponseData }
  | { status: 'error'; error: string; code: number }
```

**Narrow types with guards:**
```typescript
function isUser(value: unknown): value is User {
  return typeof value === 'object' && value !== null && 'id' in value;
}
```

**Use const assertions for literals:**
```typescript
const ROLES = ['admin', 'user', 'guest'] as const;
type Role = typeof ROLES[number]; // 'admin' | 'user' | 'guest'
```

**Add generic constraints:**
```typescript
// Before
function getProperty<T>(obj: T, key: string): any

// After
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K]
```

### Step 4: Strengthen tsconfig.json
Recommend enabling (incrementally):
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true
  }
}
```

### Step 5: Verify
```bash
npx tsc --noEmit
```
Ensure zero type errors after changes.

## Rules
- Fix types incrementally — don't try to fix everything at once
- Start with public API surfaces (exports), work inward
- Use `unknown` instead of `any` when type is truly unknown
- Prefer type inference where it's clear and correct
- Don't add redundant type annotations where TS infers correctly
- Use `satisfies` operator for validation without widening
- Document complex types with JSDoc comments
