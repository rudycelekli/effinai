---
name: browser-agent
description: "Full browser automation with AI-optimized snapshots, form filling, tab management, and JavaScript evaluation — ported from jcode's browser tool"
version: "1.0.0"
tags: [browser, automation, web, scraping, testing, ui]
---

# Browser Agent

## Activation
When the agent needs to interact with a real browser: navigating pages, clicking elements, filling forms, taking screenshots, extracting page content, or running JavaScript. Use for web scraping, UI testing, form automation, or any task requiring a live browser.

## Concept

A browser automation tool that communicates with a running browser instance via a bridge extension. Unlike headless scraping, this controls a real browser with full JavaScript execution, cookie handling, and rendering. The agent sees AI-optimized snapshots of the page (annotated DOM with interactable elements labeled) rather than raw HTML, making it practical to navigate complex SPAs and dynamic pages.

Ported from jcode's `BrowserTool` which uses a Firefox bridge extension with native messaging for low-latency control.

## Architecture

```
Agent -> Browser Tool -> Browser Bridge Extension -> Browser Instance
                              |
                    Native Messaging (JSON over stdio)
                              |
                    Extension Content Script (DOM access)
```

### Provider Protocol
The browser tool uses a provider abstraction allowing different browser backends:

| Provider | Browser | Transport | Status |
|----------|---------|-----------|--------|
| `firefox_agent_bridge` | Firefox | Native messaging extension | Primary |
| `chrome` | Chrome | CDP (Chrome DevTools Protocol) | Planned |
| `safari` | Safari | WebDriver | Planned |

## Actions

### Setup & Status
```
action: "status"
# Check if the browser bridge is installed and connected. Always call this first.
# Returns: bridge version, connection state, active tab info

action: "setup"
# First-time install or repair of the browser bridge extension.
# Only run when status shows the bridge is not ready.
```

### Tab Management
```
action: "list_tabs"
# List all open browser tabs with id, title, URL

action: "new_tab"
url: "https://example.com"
# Open a URL in a new tab

action: "select_tab"
tab_id: 42
# Switch to a specific tab by ID

action: "get_active_tab"
# Get info about the currently active tab

action: "list_frames"
tab_id: 42
# List all frames/iframes in a tab
```

### Navigation & Content
```
action: "open"
url: "https://example.com"
wait: true          # Wait for page load (default: true)
new_tab: false      # Open in current tab (default)
# Navigate to a URL

action: "snapshot"
format: "annotated"  # Options: annotated, text, textFast, html, title
tab_id: 42           # Optional: specific tab
frame_id: 0          # Optional: specific frame
# Get page content. "annotated" labels interactive elements with [ref=N] markers
# that can be used with click/type actions. This is the primary way agents "see" the page.

action: "get_content"
selector: "#main-content"
# Get text content of a specific element

action: "interactables"
# List all interactive elements (buttons, links, inputs) with their ref IDs
```

### Interaction
```
action: "click"
selector: "[ref='5']"    # Click by annotated ref from snapshot
# OR
selector: "#submit-btn"  # Click by CSS selector
# OR
x: 150
y: 300                   # Click by coordinates

action: "type"
selector: "#search-input"
text: "query string"
clear: true              # Clear existing text first (default: false)
submit: true             # Press Enter after typing (default: false)

action: "press"
key: "Enter"             # Send a keyboard key
# Supports: Enter, Tab, Escape, ArrowDown, etc.

action: "select"
selector: "#country-dropdown"
text: "United States"    # Select an option by visible text

action: "fill_form"
fields:
  - selector: "#name"
    value: "John Doe"
  - selector: "#email"
    value: "john@example.com"
  - selector: "#agree"
    checked: true
# Fill multiple form fields in one action

action: "upload"
selector: "#file-input"
path: "/path/to/file.pdf"
# Upload a file to a file input element
```

### Scrolling
```
action: "scroll"
position: "down"         # Options: up, down, top, bottom
# OR
scroll_to:
  x: 0
  y: 1500               # Scroll to specific coordinates
behavior: "smooth"       # Options: smooth, instant
```

### JavaScript & Screenshots
```
action: "eval"
script: "document.title"
page_world: true         # Run in page context (access page variables)
# Execute JavaScript and return the result

action: "screenshot"
selector: "#chart"       # Optional: screenshot specific element
format: "png"            # Output format
path: "/tmp/screenshot.png"  # Optional: save to file
# Capture a screenshot. Returns base64-encoded image.

action: "wait"
selector: "#loading-spinner"
timeout_ms: 5000
# Wait for an element to appear/disappear
```

### Provider Commands
```
action: "provider_command"
provider_action: "custom_action"
params: { "key": "value" }
# Send raw commands to the browser provider for extension-specific features
```

## Workflow Patterns

### Web Scraping Pattern
```
1. status -> Check bridge is ready
2. open url -> Navigate to target page
3. snapshot format=annotated -> See the page structure
4. click/type as needed -> Interact with the page
5. get_content selector -> Extract specific data
6. eval script -> Run JS for complex extraction
```

### Form Automation Pattern
```
1. open url -> Navigate to form page
2. snapshot -> Identify form fields
3. fill_form fields=[...] -> Fill all fields at once
4. screenshot -> Verify form state visually
5. click selector="#submit" -> Submit
6. wait selector=".success" -> Confirm submission
```

### Multi-Tab Research Pattern
```
1. list_tabs -> See current tabs
2. new_tab url=X -> Open research targets in parallel tabs
3. select_tab + snapshot -> Read each tab's content
4. eval -> Extract structured data from each page
```

## Important Notes

- Always call `status` before first use to verify the bridge is connected
- Only call `setup` if status indicates the bridge is not ready
- Use `snapshot format=annotated` as the primary way to "see" pages -- it provides labeled interactive elements
- The `[ref=N]` markers in annotated snapshots can be used directly as selectors: `selector="[ref='5']"`
- For SPAs, use `wait` after navigation to ensure content has loaded
- `eval` with `page_world: true` can access page-level JavaScript variables and APIs
- Screenshots are returned as base64 and can be saved to files for visual verification
