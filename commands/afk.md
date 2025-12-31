---
name: afk
description: AFK return command - shows time gap, runs stale check, summarizes background tasks
---

# /afk - Return from AFK

Use when returning to a session after being away. Resets context and catches you up.

## Usage

```
/afk                           # Basic return - time gap + stale check
/afk worked on X offline       # Log offline work to session
```

## What It Does

1. **Time Gap Detection**
   - Show time since last interaction
   - Flag if significant gap (>1hr, >8hr, >24hr)

2. **Stale Check**
   - Run /stale to detect modified agents/skills
   - Recommend restart if stale files found

3. **Background Task Summary**
   - Check for completed background tasks
   - Summarize results of overnight/AFK work

4. **Optional: Log Offline Work**
   - If argument provided, add to session log
   - Format: "Offline: [user's description]"

## Output Format

```
═══════════════════════════════════════════════
 Welcome back!
═══════════════════════════════════════════════

⏱️  Time away: 8h 54m (overnight gap)

🔄 Stale Check:
   ✅ No stale files - session is current
   OR
   ⚠️  2 files modified since session start:
      - ~/.claude/agents/pattern-learner.md
      - ~/.claude/skills/timer.md
   → Recommend: Restart session to pick up changes

📋 Background Tasks:
   ✅ insight-validation: 10/10 passed
   ✅ concept-linker: 19 backlinks added
   ⏸️  medium-value-scan: Still pending

📝 Session Log:
   Added return entry to 2025-12-23-late-night.md

Ready to continue!
═══════════════════════════════════════════════
```

## Implementation

### Step 1: Calculate Time Gap

```bash
# Get last interaction time from session log
LAST_ENTRY=$(grep -E "^### [0-9]{2}:[0-9]{2}" ~/Documents/Obsidian/meta/session-logs/$(date +%Y-%m-%d)*.md | tail -1 | cut -d' ' -f2)
CURRENT_TIME=$(date +%H:%M)
# Calculate difference (handle midnight rollover)
```

### Step 2: Run Stale Check

Invoke /stale skill or check directly:
```bash
# Compare mtimes of agent/skill files vs session start
find ~/.claude/agents ~/.claude/skills -name "*.md" -newer /tmp/claude-session-start 2>/dev/null
```

### Step 3: Check Background Tasks

```bash
# List completed tasks from /tmp/claude/tasks/
ls -la /tmp/claude/-Users-you-Documents/tasks/*.output 2>/dev/null | tail -5
```

### Step 4: Add Session Log Entry

```markdown
### HH:MM - Back from AFK (Xh Xm away)
- Time gap: [duration]
- Stale files: [count or "none"]
- Background tasks: [summary]
- Offline work: [if provided]
```

## Gap Thresholds

| Gap | Label | Action |
|-----|-------|--------|
| <1h | short break | Standard return |
| 1-8h | extended break | Check background tasks |
| 8-24h | overnight | Full catchup, recommend /stale |
| >24h | multi-day | Consider new session log |

## Integration Points

- **/stale**: Runs automatically as part of return
- **/log**: Adds return entry to session log
- **/timer**: Could correlate with time tracking
- **pattern-learner**: Feeds work session patterns (gap durations, return times)
- **system-review skill**: Quick health check on return

### Quick Health Check (via system-review skill)

Include a condensed infrastructure health check:

```bash
# Quick health snapshot
echo "═══ Quick Health Check ═══"

# Pattern registry
patterns=$(jq -r '.stats.total_patterns // 0' ~/.claude/patterns/registry.json 2>/dev/null)
if [ "$patterns" -eq 0 ]; then
  echo "Patterns:       ⚠️ (empty)"
else
  echo "Patterns:       ✅ ($patterns)"
fi

# Hooks
hooks_ok=true
for hook in capture-recovery-state.sh stale-warning-hook.sh; do
  [ ! -f "$HOME/.claude/scripts/$hook" ] && hooks_ok=false
done
[ "$hooks_ok" = true ] && echo "Hooks:          ✅" || echo "Hooks:          ⚠️"

# Backlog
if [ -f "$HOME/.claude/queues/backlog.jsonl" ]; then
  backlog=$(wc -l < "$HOME/.claude/queues/backlog.jsonl" | tr -d ' ')
  echo "Backlog:        $backlog items"
fi

# Inbox
inbox=$(ls ~/Documents/Obsidian/inbox/ 2>/dev/null | wc -l | tr -d ' ')
[ "$inbox" -gt 0 ] && echo "Inbox:          ⚠️ $inbox items" || echo "Inbox:          ✅"
```

For detailed health, suggest: "Run `/status` for full system review"

## Pattern Learning Data

Log return events for pattern-learner:
```json
{
  "event": "afk_return",
  "timestamp": "2025-12-23T09:54:00Z",
  "gap_minutes": 534,
  "gap_category": "overnight",
  "stale_files": 0,
  "background_tasks_completed": 3,
  "offline_work_logged": false
}
```

Store at: `~/.claude/patterns/session/returns.jsonl`

## Notes

- Always greet warmly - returning from AFK is a transition moment
- Keep output scannable - user wants quick catchup, not wall of text
- Highlight anything that needs attention (stale files, failed tasks)
- If multi-day gap, suggest reviewing backlog for context
