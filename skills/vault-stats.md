---
name: vault-stats
description: Display quick vault health stats showing file counts, queue status, and last activity
---

# Vault Stats

Display comprehensive vault health statistics.

## Workflow

Execute these bash commands to gather stats:

### 1. File Counts

```bash
echo "| Category     | Count |"
echo "|--------------|-------|"
echo "| Insights     | $(find ~/Documents/Obsidian/insights -name '*.md' 2>/dev/null | wc -l | tr -d ' ') |"
echo "| Concepts     | $(find ~/Documents/Obsidian/concepts -name '*.md' 2>/dev/null | wc -l | tr -d ' ') |"
echo "| Sources      | $(find ~/Documents/Obsidian/sources -name '*.md' 2>/dev/null | wc -l | tr -d ' ') |"
echo "| Patterns     | $(wc -l < ~/.claude/patterns/recovery/corrections.jsonl 2>/dev/null | tr -d ' ') |"
echo "| Session Logs | $(find ~/Documents/Obsidian/meta/session-logs -name '*.md' 2>/dev/null | wc -l | tr -d ' ') |"
```

### 2. Queue Status

```bash
queue=~/Documents/Obsidian/analysis_results/extraction-queue.jsonl
if [ -f "$queue" ]; then
  echo "| Status    | Count |"
  echo "|-----------|-------|"
  echo "| Total     | $(wc -l < "$queue" | tr -d ' ') |"
  echo "| Pending   | $(grep -c '"status":"pending"' "$queue" 2>/dev/null || echo 0) |"
  echo "| Completed | $(grep -c '"status":"completed"' "$queue" 2>/dev/null || echo 0) |"
else
  echo "No extraction queue found."
fi
```

### 3. Last Activity

```bash
echo "Latest insight: $(ls -t ~/Documents/Obsidian/insights/**/*.md 2>/dev/null | head -1 | xargs basename 2>/dev/null || echo 'None')"
echo "Latest session: $(ls -t ~/Documents/Obsidian/meta/session-logs/**/*.md 2>/dev/null | head -1 | xargs basename 2>/dev/null || echo 'None')"
```

### 4. Pattern Health

```bash
echo "| Domain       | Count |"
echo "|--------------|-------|"
python3 ~/.claude/scripts/pattern_query.py --stats 2>/dev/null | grep -E "^\s+\w+:" | head -6
```
