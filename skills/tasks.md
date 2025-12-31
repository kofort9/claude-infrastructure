---
name: tasks
description: Monitor and manage background task queue
arguments:
  - name: action
    description: "status (default) | list | active | cleanup | cancel"
  - name: filter
    description: "Optional filter: task type, status, or task ID"
---

# Tasks Skill

Monitor and manage the background task queue.

## Usage

```bash
/tasks                    # Show queue status
/tasks status             # Same as above
/tasks list               # List all tasks
/tasks list pending       # List pending tasks
/tasks list extraction    # List extraction tasks
/tasks active             # Show currently running tasks
/tasks cancel <id>        # Cancel a task
/tasks cleanup            # Remove old completed tasks
```

## Task Lifecycle

```
┌─────────┐     ┌─────────┐     ┌───────────┐
│ PENDING │ ──► │ RUNNING │ ──► │ COMPLETED │
└─────────┘     └─────────┘     └───────────┘
     │               │                ▲
     │               │                │
     │               ▼                │
     │          ┌────────┐            │
     └────────► │ FAILED │ ──retry───┘
                └────────┘
                     │
                     ▼ (max attempts)
               ┌───────────┐
               │ CANCELLED │
               └───────────┘
```

## Status Dashboard

```
/tasks status

Task Queue Status
═══════════════════════════════════════════════════════════════

📊 Overview
   Total tasks: 45
   Active: 2
   Pending: 12
   Completed: 28
   Failed: 3

📋 By Type
   extraction: 30
   classification: 10
   validation: 5

🏃 Active Tasks
   abc12345  extraction     Running for 2m 30s
   def67890  classification Running for 45s

⏳ Pending (next 5)
   ghi11111  extraction     Scheduled retry at 12:35:00
   jkl22222  extraction     Waiting
   ...
```

## Task Types

| Type | Description |
|------|-------------|
| extraction | Insight extraction from conversation |
| classification | Multi-label classification |
| validation | Insight validation |
| linking | Concept linking |
| import | ChatGPT import processing |

## Auto-Retry

Tasks with transient failures are automatically retried:
- **Max attempts**: 3 (configurable per task)
- **Backoff**: Exponential (2s, 4s, 8s)
- **Retry condition**: `attempts < max_attempts`

## Implementation

Uses `scripts/task_queue.py`:

```python
from task_queue import TaskQueue, TaskStatus

queue = TaskQueue()

# Create task
task_id = queue.create("extraction", {"file": "example.md"})

# Start processing
queue.start(task_id)

# Complete or fail
queue.complete(task_id, result={"insights": 5})
queue.fail(task_id, error="Timeout", retry=True)

# Query
active = queue.get_active()
stats = queue.get_stats()
```

## Files

| File | Purpose |
|------|---------|
| `~/.claude/queues/task-queue.jsonl` | Queue storage |
| `scripts/task_queue.py` | TaskQueue class |

## Related

- `/chatgpt-process` - Uses task queue for extraction
- `/bulk-correct` - Batch processing with task tracking
- `protocols/recovery.md` - Recovery from failures
