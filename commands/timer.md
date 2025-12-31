---
name: timer
description: Timer Skill
arguments:
  - name: action
    description: "start, stop, status, report, or cancel"
---

# Timer Skill

Track time spent on tasks. Separate from `/timestamp` (validation) - this is for duration tracking.

## Usage

```
/timer start [task description]
/timer stop
/timer status
/timer report [today|week|task-name]
/timer cancel
```

## Commands

### start
```
/timer start reviewing PR #42
```
Creates entry in `~/.claude/time-logs.jsonl`:
```json
{"id": "t_abc123", "task": "reviewing PR #42", "started": "2025-12-23T00:45:00Z", "status": "running"}
```

### stop
```
/timer stop
```
Updates entry:
```json
{"id": "t_abc123", "task": "reviewing PR #42", "started": "2025-12-23T00:45:00Z", "stopped": "2025-12-23T01:02:00Z", "duration_min": 17, "status": "completed"}
```

### status
```
/timer status
```
Output:
```
Timer running: "reviewing PR #42"
Started: 00:45
Elapsed: 17 minutes
```

Or if no timer:
```
No active timer.
Last completed: "reviewing PR #42" (17 min) at 01:02
```

### report
```
/timer report today
```
Output:
```
Time Report: 2025-12-23

Tasks:
  reviewing PR #42          17 min
  insight extraction        23 min
  planning session          45 min
                           -------
  Total:                    85 min (1h 25m)
```

### cancel
```
/timer cancel
```
Discards current timer without logging.

## Data Storage

File: `~/.claude/time-logs.jsonl`

Schema:
```json
{
  "id": "t_abc123",
  "task": "task description",
  "started": "ISO8601",
  "stopped": "ISO8601 or null",
  "duration_min": 17,
  "status": "running|completed|cancelled",
  "tags": ["review", "pr"],
  "session": "2025-12-23-late-night"
}
```

## Auto-Tagging

Detect tags from task description:
- "PR" / "pull request" → `pr`
- "review" → `review`
- "insight" / "extract" → `knowledge`
- "plan" / "planning" → `planning`
- "debug" / "fix" → `debugging`

## Integration

### With pattern-learner
Timer data feeds into pattern-learner for:
- Task duration patterns (how long do PR reviews actually take?)
- Estimation accuracy (estimated 10 min, took 45 min)
- Time-of-day patterns (faster in morning?)

### With session logs
Option to auto-log timer completions to session log:
```
### 01:02 - Timer: reviewing PR #42 (17 min)
```

## Implementation

```bash
# Start timer
TIMER_FILE=~/.claude/.active-timer
echo "{\"task\": \"$1\", \"started\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"}" > "$TIMER_FILE"

# Check status
if [ -f "$TIMER_FILE" ]; then
  cat "$TIMER_FILE"
else
  echo "No active timer"
fi

# Stop timer
if [ -f "$TIMER_FILE" ]; then
  # Calculate duration, append to time-logs.jsonl, remove active timer
  STARTED=$(jq -r .started "$TIMER_FILE")
  # ... duration calculation ...
  rm "$TIMER_FILE"
fi
```

## Notes

- Only one timer active at a time (single-tasking encouraged)
- Timers persist across session restarts via file
- Cancel doesn't log (for false starts)
- Tags enable pattern analysis across task types
