---
name: accessibility-audit
description: "WCAG accessibility compliance audit for web applications"
version: "1.0.0"
tags: [quality, accessibility, wcag, a11y]
createdBy: "built-in"
status: "active"
---

# Accessibility Audit

## Activation
When the user asks to check accessibility, WCAG compliance, or a11y issues.

## Workflow

### Step 1: Identify Frontend Files
Find components, pages, and templates:
- React/Vue/Svelte components
- HTML templates
- CSS/SCSS files

### Step 2: WCAG 2.1 Level AA Checks

**Perceivable:**
- [ ] All images have alt text (`<img>` without `alt` attribute)
- [ ] Videos have captions/transcripts
- [ ] Color is not the only way to convey information
- [ ] Text has sufficient contrast ratio (4.5:1 for normal, 3:1 for large text)
- [ ] Content is readable at 200% zoom
- [ ] No text embedded in images

**Operable:**
- [ ] All interactive elements are keyboard accessible
- [ ] Focus order is logical (no tabindex > 0)
- [ ] Focus is visible (no `outline: none` without alternative)
- [ ] No keyboard traps
- [ ] Skip navigation link present
- [ ] Touch targets are at least 44x44px
- [ ] No content that flashes more than 3 times per second

**Understandable:**
- [ ] Language attribute set on `<html>` element
- [ ] Form inputs have associated labels (`<label for>` or `aria-label`)
- [ ] Error messages are descriptive and associated with inputs
- [ ] Consistent navigation across pages
- [ ] Abbreviations are explained

**Robust:**
- [ ] Valid HTML (proper nesting, closed tags)
- [ ] ARIA roles used correctly (not overriding native semantics)
- [ ] Custom components have proper ARIA attributes
- [ ] Status messages use `aria-live` regions

### Step 3: Common Code Patterns to Flag

```typescript
// BAD: Click handler on non-interactive element
<div onClick={handleClick}>Click me</div>

// GOOD: Use button or add role + keyboard handler
<button onClick={handleClick}>Click me</button>
// or
<div role="button" tabIndex={0} onClick={handleClick} onKeyDown={handleKeyDown}>Click me</div>

// BAD: Image without alt
<img src="photo.jpg" />

// GOOD: Descriptive alt (or empty alt for decorative)
<img src="photo.jpg" alt="Team celebrating product launch" />
<img src="divider.png" alt="" role="presentation" />

// BAD: Form without labels
<input type="email" placeholder="Email" />

// GOOD: Associated label
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// BAD: Removing focus outline
*:focus { outline: none; }

// GOOD: Custom focus indicator
*:focus-visible { outline: 2px solid #4A90D9; outline-offset: 2px; }
```

### Step 4: Output Report

```
## Accessibility Audit Report

### Critical (WCAG A)
- [File:Line] Issue description
  Fix: [specific fix]

### Important (WCAG AA)
- [File:Line] Issue description
  Fix: [specific fix]

### Enhancement (WCAG AAA)
- [File:Line] Issue description
  Fix: [specific fix]

### Summary
- Total issues: [N]
- Critical: [N] | Important: [N] | Enhancement: [N]
- Estimated fix effort: [time]
```

## Rules
- Focus on WCAG AA as the minimum standard
- Provide specific fix code for each issue
- Don't suggest ARIA where native HTML elements suffice
- Check both static markup and dynamic content
- Consider screen reader experience, not just visual
- Test keyboard navigation flow, not just individual elements
