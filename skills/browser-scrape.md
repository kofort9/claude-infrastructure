---
name: browser-scrape
description: Use Playwright MCP to screen-scrape external UIs for data APIs don't expose (activity feeds, pulse updates)
arguments: url (required), extraction-type (optional - activity, table, content)
---

# Browser Scrape Skill

Screen-scrape your own accounts' data from web UIs using Playwright MCP. Pragmatic solution for extracting activity feeds, pulse updates, and other data that APIs don't expose.

## Prerequisites

### Playwright MCP Setup

**The Problem:** Standard `npx` fails in Claude Code subprocesses because nvm shell functions aren't available.

**The Fix:** npx-wrapper that uses system npm.

**Files:**
- Wrapper: `/Users/you/bin/npx-wrapper`
- Config: `~/.claude/settings.json` → `mcpServers.playwright`

**Verify working:**
```bash
claude mcp list | grep playwright
# Should show: ✓ Connected
```

**If broken, recreate wrapper:**
```bash
cat > /Users/you/bin/npx-wrapper << 'EOF'
#!/bin/bash
exec /opt/homebrew/bin/npx "$@"
EOF
chmod +x /Users/you/bin/npx-wrapper
```

## Usage

```
/browser-scrape <url> [extraction-type]
```

**Examples:**
- `/browser-scrape https://linear.app/kaxfhq/inbox activity` - Linear activity feed
- `/browser-scrape https://linear.app/kaxfhq/my-issues table` - Issues table
- `/browser-scrape https://github.com/user/repo/pulse content` - GitHub pulse

## Primary Use Case: Linear Pulse Updates

Linear's API exposes issues and documents, but NOT:
- Activity feed (who did what when)
- Notifications/inbox items
- Status change timeline with context
- Comment threads in activity context

### Extracting Linear Activity

1. **Navigate to Linear inbox/activity:**
```
mcp__playwright__browser_navigate url="https://linear.app/kaxfhq/inbox"
```

2. **Wait for auth (if needed):**
- Playwright will open browser window
- Log in if prompted
- Activity loads

3. **Screenshot for reference:**
```
mcp__playwright__browser_screenshot
```

4. **Extract activity items:**
```
mcp__playwright__browser_evaluate script="
  Array.from(document.querySelectorAll('[data-testid=\"activity-item\"]'))
    .map(el => ({
      text: el.innerText,
      time: el.querySelector('time')?.dateTime
    }))
"
```

5. **Save to pulse file:**
Append extracted activity to:
`/Users/you/Documents/Obsidian/sources/linear/pulse-YYYY-MM-DD.md`

## Extraction Patterns

### Activity Feed Pattern
For timeline-style feeds (Linear inbox, GitHub activity):
```javascript
// Generic activity extraction
Array.from(document.querySelectorAll('[class*="activity"], [class*="feed"], [class*="timeline"]'))
  .slice(0, 20)
  .map(el => ({
    content: el.innerText.trim().slice(0, 500),
    timestamp: el.querySelector('time')?.dateTime || 'unknown'
  }))
```

### Table Pattern
For structured data (issues lists, PR lists):
```javascript
// Generic table extraction
const rows = document.querySelectorAll('table tbody tr, [role="row"]');
Array.from(rows).slice(0, 50).map(row => {
  const cells = row.querySelectorAll('td, [role="cell"]');
  return Array.from(cells).map(c => c.innerText.trim());
})
```

### Content Pattern
For article/document content:
```javascript
// Main content extraction
document.querySelector('main, article, [role="main"]')?.innerText.trim()
```

## Output Format

Save extracted data with provenance:

```markdown
---
source: browser-scrape
url: https://linear.app/kaxfhq/inbox
captured: 2025-12-22T20:00:00Z
extraction_type: activity
---

# Linear Activity - December 22, 2025

## Activity Feed

| Time | Activity |
|------|----------|
| 2h ago | Kofi completed KHQ-75: Skills infrastructure |
| 3h ago | Kofi moved KHQ-71 to Done |
| ... | ... |

---
*Extracted via Playwright MCP screen-scrape*
```

## Troubleshooting

### "Failed to connect"
1. Check wrapper exists: `ls -la /Users/you/bin/npx-wrapper`
2. Check MCP config in `~/.claude/settings.json`
3. Restart Claude Code

### Auth issues
- Playwright opens visible browser - log in manually
- Session cookies persist between runs
- For persistent auth, use Playwright's storage state

### Slow extraction
- Linear is a React SPA - wait for hydration
- Use `browser_wait_for` before extracting
- Increase timeout for heavy pages

## Integration with Project Pulse

This skill extends `/project-pulse` by adding browser-based data:

1. Run `/project-pulse` for API-based data (git, Linear API)
2. Run `/browser-scrape linear activity` for UI-only data
3. Combine into unified pulse snapshot

## Security Note

Only scrape your own accounts. This skill is for personal data sovereignty - extracting YOUR data from services YOU use. Not for scraping others' data.
