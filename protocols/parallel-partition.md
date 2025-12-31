# Parallel Partition Protocol

Pattern for splitting work across multiple agents to parallelize processing.

## Overview

When processing large corpora (1000+ files), partition the work and delegate to parallel agents.

```
                    ORCHESTRATOR
   Partitions work -> Launches agents -> Aggregates results
                          |
         +----------------+----------------+
         |                |                |
      Agent 1          Agent 2          Agent 3
     Partition A      Partition B      Partition C
         |                |                |
      Results          Results          Results
         |                |                |
         +--------> Merged Results <------+
```

## Partition Strategies

### 1. Time-Based (Recommended for conversations)

Partition by year/month for natural grouping:

```python
def partition_by_time(files: List[Path]) -> Dict[str, List[Path]]:
    partitions = {}
    for f in files:
        # Extract YYYY/MM from path
        parts = f.relative_to(root).parts
        key = f"{parts[0]}/{parts[1]}"  # e.g., "2024/01"
        partitions.setdefault(key, []).append(f)
    return partitions
```

### 2. Count-Based (Even distribution)

Partition into N equal chunks:

```python
def partition_by_count(files: List[Path], n: int) -> List[List[Path]]:
    chunk_size = len(files) // n
    return [files[i:i+chunk_size] for i in range(0, len(files), chunk_size)]
```

### 3. Topic-Based (Domain isolation)

Partition by classification result:

```python
def partition_by_topic(classifications: List[dict]) -> Dict[str, List[dict]]:
    partitions = {}
    for c in classifications:
        topic = c["primary_topic"]
        partitions.setdefault(topic, []).append(c)
    return partitions
```

## Agent Launch Pattern

Use the Task tool with `run_in_background=true`:

```python
# Orchestrator launches parallel agents
for partition_key, files in partitions.items():
    Task(
        description=f"Process {partition_key}",
        prompt=f"Process these files: {files}",
        subagent_type="extraction-worker",
        run_in_background=True
    )
```

## Output Isolation

Each agent writes to partition-specific output to avoid conflicts:

```
analysis_results/
├── partition-2024-01.jsonl
├── partition-2024-02.jsonl
└── partition-2024-03.jsonl
```

## Progress Tracking

Use a task queue to track each partition:

```python
# Create parent task
parent_id = queue.create("parallel-extraction", {
    "partitions": len(partitions),
    "total_files": len(all_files)
})

# Create subtasks
for key, files in partitions.items():
    queue.create(
        "extraction-partition",
        {"key": key, "files": len(files)},
        parent_id=parent_id
    )
```

Query subtask status:
```python
subtasks = queue.get_by_parent(parent_id)
completed = len([t for t in subtasks if t.status == "completed"])
```

## Result Aggregation

After all partitions complete:

```python
def aggregate_results(partitions: Dict[str, Path]) -> dict:
    all_results = []
    for key, output_path in partitions.items():
        with open(output_path) as f:
            for line in f:
                all_results.append(json.loads(line))

    # Merge to final output
    with open("analysis_results/final.jsonl", 'w') as f:
        for r in all_results:
            f.write(json.dumps(r) + '\n')

    return {
        "total": len(all_results),
        "partitions": len(partitions)
    }
```

## Collision Prevention

Rules to prevent conflicts:

1. **Output isolation**: Each agent writes to unique partition file
2. **Read-only sources**: Agents never modify input files
3. **Append-only logs**: Use JSONL for concurrent-safe logging
4. **Task locking**: Task queue prevents double-processing

## Error Handling

If a partition fails:

1. Mark partition task as failed in queue
2. Other partitions continue independently
3. Retry failed partition with exponential backoff
4. After max retries, mark for manual review

```python
def handle_partition_failure(partition_key: str, error: str):
    task = queue.get_by_params({"key": partition_key})
    queue.fail(task.task_id, error=error, retry=True)

    if task.attempts >= task.max_attempts:
        log_for_review(partition_key, error)
```

## Optimal Partition Size

| Total Files | Recommended Partitions | Files per Partition |
|-------------|------------------------|---------------------|
| 100-500 | 1-2 | 100-250 |
| 500-1000 | 2-4 | 200-300 |
| 1000-2000 | 4-8 | 200-300 |
| 2000+ | 8-12 | 200-300 |

## Example: Full Extraction Pipeline

```python
# 1. Get all files
files = list(ROOT.glob("**/*.md"))
print(f"Total files: {len(files)}")

# 2. Partition by year/month
partitions = partition_by_time(files)
print(f"Partitions: {len(partitions)}")

# 3. Create parent task
parent_id = queue.create("bulk-extraction", {
    "total": len(files),
    "partitions": len(partitions)
})

# 4. Launch agents in parallel
for key, partition_files in partitions.items():
    task_id = queue.create(
        "extraction-partition",
        {"key": key, "count": len(partition_files)},
        parent_id=parent_id
    )

    # Launch agent
    Task(
        description=f"Extract {key}",
        prompt=f"Extract insights from {len(partition_files)} files in {key}",
        subagent_type="insight-extractor",
        run_in_background=True
    )

# 5. Monitor progress
while True:
    subtasks = queue.get_by_parent(parent_id)
    completed = [t for t in subtasks if t.status == "completed"]
    failed = [t for t in subtasks if t.status == "failed"]

    print(f"Progress: {len(completed)}/{len(subtasks)} (failed: {len(failed)})")

    if len(completed) + len(failed) == len(subtasks):
        break

    time.sleep(10)

# 6. Aggregate results
results = aggregate_results({
    t.params["key"]: f"analysis_results/partition-{t.params['key']}.jsonl"
    for t in completed
})

# 7. Mark parent complete
queue.complete(parent_id, result=results)
```

## Related

- `/tasks` skill - Monitor partition progress
- `insight-extractor` agent - Common worker for extraction
- `orchestrator` agent - Manages partition coordination
