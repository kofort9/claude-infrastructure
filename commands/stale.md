---
name: stale
description: Check if agents/skills modified since session start, recommend restart if stale
---

# Session Stale Check Skill

Checks if agents, skills, commands, hooks, or protocols have been modified since session start, indicating a restart may be needed to pick up changes. Includes cross-session awareness via `meta-state.json`.

## Usage

```
/stale
```

## Behavior

1. **Record session start**: On first invocation, capture current timestamp
2. **Check local modifications**: Compare file mtimes in monitored directories against session start
3. **Check cross-session changes**: Read `~/.claude/meta-state.json` for changes from other sessions
4. **Report**: List any files modified after session started, with source info

## Directories Monitored

- `~/.claude/agents/` - Agent definitions
- `~/.claude/skills/` - Skill definitions
- `~/.claude/commands/` - Command definitions
- `~/.claude/hooks/` - Hook scripts
- `~/.claude/protocols/` - Protocol definitions
- `~/.claude/scripts/` - Scripts (hooks, utilities)
- `~/.claude/settings.json` - Global settings
- `~/.claude/settings.local.json` - Local settings

## Cross-Session Awareness

The skill reads `~/.claude/meta-state.json` which is updated by SessionEnd hooks in other sessions:

```json
{
  "last_meta_change": "2025-12-23T16:35:00Z",
  "changed_files": [
    {"file": "pattern-learner.md", "type": "agent", "ts": "2025-12-23T16:35:00Z"}
  ]
}
```

## Output Format

### No Changes
```
✓ Session is current. No changes detected.
Started: 2025-12-23 00:15
```

### Changes Detected (This Session)
```
⚠ Session may be stale. Modified since session start:

Agents (2):
  - gitops-devex.md (00:45) [this session]
  - orchestrator.md (00:32) [this session]

Commands (1):
  - stale.md (00:40) [this session]

Recommendation: Restart session to pick up changes.
Started: 2025-12-23 00:15
Current: 2025-12-23 00:50
```

### Changes from Other Session
```
⚠ Cross-session changes detected!

Another session modified:
  - pattern-learner.md (agent) at 16:35
  - vocabulary.md (protocol) at 16:30

This session started: 2025-12-23 14:00
Changes detected after: 2025-12-23 16:30

Recommendation: Restart to pick up changes from other session.
```

## Implementation

```bash
# Get session start time (store in env or temp file on first run)
SESSION_START=${SESSION_START:-$(date +%s)}

# Find files modified after session start
find ~/.claude/agents ~/.claude/skills -type f -name "*.md" -newermt "@$SESSION_START" 2>/dev/null

# Check settings files
for f in ~/.claude/settings.json ~/.claude/settings.local.json; do
  if [ -f "$f" ] && [ $(stat -f %m "$f") -gt $SESSION_START ]; then
    echo "Modified: $f"
  fi
done
```

## Session Start Tracking

The skill needs to know when the current session started. Options:

1. **First invocation**: Store timestamp on first `/stale` call
2. **Environment variable**: Set `CLAUDE_SESSION_START` at session init
3. **Temp file**: Write to `~/.claude/.session-start`

Recommended: Use temp file approach for persistence across tool calls.

```bash
# On first call, create session marker
SESSION_FILE=~/.claude/.session-start
if [ ! -f "$SESSION_FILE" ]; then
  date +%s > "$SESSION_FILE"
fi
SESSION_START=$(cat "$SESSION_FILE")
```

## Integration Points

- **SessionEnd hook**: Could delete `.session-start` file on session end
- **Orchestrator**: Could proactively call `/stale` before delegating to agents
- **/sync-agents**: After syncing, remind user session may need restart

## Proactive Use

Consider calling `/stale` automatically:
- Before spawning agents for important tasks
- After running `/sync-agents`
- When user asks "why isn't X working?"

## Notes

- This skill doesn't force restart, just informs
- Changes to agents/skills require session restart to take effect
- Settings changes may or may not require restart depending on the setting
