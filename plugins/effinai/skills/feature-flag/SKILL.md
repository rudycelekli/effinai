---
name: feature-flag
description: "Feature flag implementation patterns for safe rollouts"
version: "1.0.0"
tags: [productivity, feature-flags, rollout, deployment]
createdBy: "built-in"
status: "active"
---

# Feature Flag Implementation

## Activation
When the user asks to add a feature flag, implement gradual rollout, or toggle features.

## Workflow

### Step 1: Choose the Pattern

**Simple Boolean Flag (config-based):**
```typescript
// config/features.ts
export const FEATURES = {
  NEW_DASHBOARD: process.env.FEATURE_NEW_DASHBOARD === 'true',
  DARK_MODE: process.env.FEATURE_DARK_MODE === 'true',
} as const;

// Usage
if (FEATURES.NEW_DASHBOARD) {
  renderNewDashboard();
} else {
  renderLegacyDashboard();
}
```

**Percentage Rollout:**
```typescript
function isEnabled(flagName: string, userId: string, percentage: number): boolean {
  // Deterministic hash ensures same user always gets same result
  const hash = simpleHash(`${flagName}:${userId}`) % 100;
  return hash < percentage;
}
```

**User Segment Flag:**
```typescript
interface FeatureFlag {
  name: string;
  enabled: boolean;
  allowList?: string[];    // Specific user IDs
  percentage?: number;      // Rollout percentage
  segments?: string[];      // User segments (beta, internal, premium)
}
```

### Step 2: Implement the Flag System

1. Create a feature flags configuration file
2. Add the flag check utility function
3. Wrap the new feature code with the flag check
4. Ensure the old code path remains as the fallback
5. Add the flag to environment/config documentation

### Step 3: Usage Patterns

**Backend (API route):**
```typescript
app.get('/api/data', (req, res) => {
  if (isFeatureEnabled('new-data-format', req.user.id)) {
    return res.json(newFormatResponse(data));
  }
  return res.json(legacyFormatResponse(data));
});
```

**Frontend (React):**
```typescript
function Dashboard() {
  const { isEnabled } = useFeatureFlags();
  
  if (isEnabled('new-dashboard')) {
    return <NewDashboard />;
  }
  return <LegacyDashboard />;
}
```

### Step 4: Cleanup Plan
Feature flags are temporary. For each flag, document:
- When it was introduced
- Expected removal date
- What to keep (new code) vs remove (old code + flag checks)

```typescript
// FEATURE FLAG: new-checkout (added 2024-01-15, remove by 2024-03-01)
// Keep: NewCheckoutFlow, Remove: LegacyCheckout + this flag check
```

## Rules
- Every flag must have a documented removal plan
- Use deterministic hashing for percentage rollouts (not random)
- Always have a working fallback (the old code path)
- Test both flag-on and flag-off code paths
- Keep flag names descriptive and consistent (kebab-case)
- Log when a flag is evaluated for debugging rollout issues
- Never nest feature flags (if A and if B inside A)
