---
name: ai-capture
description: Auto-detect platform from URL and route to appropriate capture command
created: 2025-12-29
---

# AI Capture - Universal URL Router

Paste any supported URL and automatically route to the correct capture command.

## Supported Platforms

| Domain | Routes To |
|--------|-----------|
| `x.com`, `twitter.com` | `/x` |
| `perplexity.ai` | `/perplexity` |
| `chatgpt.com` | `/chatgpt-single` |
| `claude.ai` | `/claude-web` |
| `gemini.google.com` | `/gemini` |
| `tiktok.com` | `/tiktok` |

## Usage

```
/ai-capture <url> [notes]
```

## Workflow

1. Extract domain from URL
2. Match against supported platforms
3. Route to appropriate capture command with URL + notes
4. If no match, inform user of supported platforms

## Implementation

```javascript
const url = args.trim().split(/\s+/)[0];
const notes = args.trim().slice(url.length).trim();

const routes = {
  'x.com': 'x',
  'twitter.com': 'x',
  'perplexity.ai': 'perplexity',
  'chatgpt.com': 'chatgpt-single',
  'claude.ai': 'claude-web',
  'gemini.google.com': 'gemini',
  'tiktok.com': 'tiktok'
};

const domain = new URL(url).hostname.replace('www.', '');
const matchedRoute = Object.entries(routes).find(([d]) => domain.includes(d));

if (matchedRoute) {
  // Invoke: /matchedRoute[1] url notes
} else {
  // List supported platforms
}
```

## Examples

```
/ai-capture https://x.com/levie/status/123456
→ Routes to /x

/ai-capture https://www.perplexity.ai/search/query interesting research
→ Routes to /perplexity with notes "interesting research"

/ai-capture https://chatgpt.com/share/abc-123
→ Routes to /chatgpt-single
```
