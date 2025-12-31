---
name: extract-patterns
description: "Extract patterns from session logs (auto or deep mode)"
arguments:
  - name: mode
    description: "auto (fast heuristics) or deep (LLM analysis)"
    required: false
  - name: date
    description: "Date to process (YYYY-MM-DD), defaults to today"
    required: false
---

# Pattern Extraction

Extract workflow patterns, corrections, and discoveries from session logs.

## Modes

- **auto** (default): Fast Python-based heuristic extraction
- **deep**: Full LLM analysis for nuanced pattern detection

## Workflow

### Auto Mode (Fast)

```bash
python3 ~/.claude/scripts/extract-session-patterns.py
```

This runs the heuristic extractor that:
- Scans for pattern indicator keywords
- Extracts stats changes (e.g., "Insights: 107 → 117 (+10)")
- Identifies section patterns from headers
- Writes to `~/.claude/patterns/recovery/corrections.jsonl`

### Deep Mode (LLM Analysis)

1. Read the session log for the specified date
2. Analyze each section for:
   - **Workflow patterns**: Recurring approaches, sequences, batching strategies
   - **Corrections**: Errors caught and fixed, lessons learned
   - **Discoveries**: Novel techniques, unexpected solutions
   - **Architecture decisions**: Design choices, agent/hook/skill patterns
   - **Security patterns**: Mitigations, protections added

3. For each pattern found, write to `~/.claude/patterns/recovery/corrections.jsonl`:

```json
{
  "id": "domain:short-name:date",
  "domain": "workflow|architecture|security|discovery",
  "pattern_type": "success|correction|novel|principle",
  "timestamp": "ISO8601",
  "observation": "What was observed",
  "outcome": "Result or impact",
  "confidence": 0.0-1.0,
  "source": "deep-extraction"
}
```

4. Report summary:
   - Patterns found by domain
   - Patterns found by type
   - New vs duplicate count

## Session Log Location

```bash
ls ~/Documents/Obsidian/meta/session-logs/${date:-$(date +%Y-%m-%d)}*.md
```

## After Extraction

Report:
- New patterns added
- Duplicates skipped
- Breakdown by domain/type

## Integration

This skill is called automatically by:
- SessionEnd hook (auto mode)
- Weekly review process (deep mode)
- Manual invocation for catch-up
