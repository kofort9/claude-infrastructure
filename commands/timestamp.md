---
name: timestamp
description: Timestamp Validation Skill
arguments:
  - name: context
    description: "What the timestamp is for (optional)"
---

# Timestamp Validation Skill

Returns validated current timestamp with pattern learning for error correction.

## Usage

```
/timestamp [context]
```

**Arguments:**
- `context` (optional): What the timestamp is for (e.g., "session log entry", "insight creation")

## Behavior

1. **Get current time**: Run `date "+%Y-%m-%d %H:%M"` to get system time
2. **Validate**: Check timestamp is reasonable (not stale, not future)
3. **Midnight detection**: If context suggests time near midnight, verify date rollover
4. **Return**: Formatted timestamp ready for use

## Validation Rules

### Stale Detection
- If you're resuming from compacted context and propose a timestamp >5 minutes from system time, flag as potentially stale
- Common pattern: Agent proposes 23:56 when it's actually 00:05 (midnight rollover)

### Future Detection
- Timestamps should never be in the future
- If proposed time > system time, use system time

### Midnight Rollover
- When time is between 23:45 - 00:15, explicitly verify date
- Log any corrections to pattern file

## Correction Logging

When a correction is made, append to `~/.claude/corrections/timestamps.jsonl`:

```json
{"timestamp": "2025-12-23T00:05:00", "proposed": "23:56", "corrected": "00:05", "context": "session log entry", "pattern": "midnight_rollover"}
```

## Pattern Categories

- `midnight_rollover`: Time crossed midnight but date wasn't updated
- `stale_context`: Timestamp from before context compaction
- `future_time`: Proposed time was ahead of system time
- `timezone_mismatch`: Rare, but possible with travel

## Output Format

Return just the timestamp in the requested format:
- Default: `HH:MM` (for session logs)
- With date: `YYYY-MM-DD HH:MM`
- ISO: `YYYY-MM-DDTHH:MM:SS`

## Example

```
User: /timestamp session log entry
Assistant: 00:25

User: /timestamp --iso
Assistant: 2025-12-23T00:25:00
```

## Integration

This skill should be called by `/log` before writing session log entries to ensure accurate timestamps.

## Implementation Notes

```bash
# Get current timestamp
date "+%H:%M"

# Get with date
date "+%Y-%m-%d %H:%M"

# Get ISO format
date "+%Y-%m-%dT%H:%M:%S"
```

Always verify against system time before using any timestamp in session logs or insights.
