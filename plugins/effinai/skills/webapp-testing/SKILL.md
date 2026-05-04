---
name: webapp-testing
description: "Test web applications using Playwright — verify frontend functionality, capture screenshots, debug UI behavior, manage server lifecycle"
version: "1.0.0"
tags: [testing, web, playwright, browser, automation]
createdBy: "built-in"
status: "active"
---

# Web Application Testing

## Activation
When the user asks to test a web application, verify UI behavior, capture screenshots, or automate browser interactions.

## Decision Tree

```
Is it static HTML?
├─ Yes → Read HTML to identify selectors → Write Playwright script
└─ No (dynamic webapp) → Is the server already running?
    ├─ No → Use server lifecycle helper + Playwright script
    └─ Yes → Reconnaissance-then-action:
        1. Navigate and wait for networkidle
        2. Take screenshot or inspect DOM
        3. Identify selectors from rendered state
        4. Execute actions with discovered selectors
```

## Server Lifecycle Management

**Single server:**
```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**Multiple servers (backend + frontend):**
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

## Playwright Script Pattern

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)  # ALWAYS headless
    page = browser.new_page()
    page.goto('http://localhost:5173')
    page.wait_for_load_state('networkidle')  # CRITICAL: Wait for JS
    
    # Reconnaissance
    page.screenshot(path='/tmp/inspect.png', full_page=True)
    content = page.content()
    buttons = page.locator('button').all()
    
    # Actions with discovered selectors
    page.click('text=Submit')
    page.fill('#email', 'test@example.com')
    
    # Assertions
    assert page.locator('.success-message').is_visible()
    
    browser.close()
```

## Common Pitfall

- DO NOT inspect the DOM before waiting for `networkidle` on dynamic apps
- DO wait for `page.wait_for_load_state('networkidle')` before inspection

## Best Practices

- Use `sync_playwright()` for synchronous scripts
- Always close the browser when done
- Use descriptive selectors: `text=`, `role=`, CSS selectors, IDs
- Add appropriate waits: `page.wait_for_selector()`, `page.wait_for_timeout()`
- Use helper scripts as black boxes — run with `--help` first, don't read source

## Common Testing Patterns

### Element Discovery
```python
buttons = page.locator('button').all()
links = page.locator('a').all()
inputs = page.locator('input').all()
```

### Form Testing
```python
page.fill('#username', 'testuser')
page.fill('#password', 'testpass')
page.click('button[type="submit"]')
page.wait_for_selector('.dashboard')
```

### Console Log Capture
```python
logs = []
page.on('console', lambda msg: logs.append(msg.text))
# ... navigate and interact
print(logs)
```

### Screenshot Comparison
```python
page.screenshot(path='before.png')
# ... make changes
page.screenshot(path='after.png')
```
