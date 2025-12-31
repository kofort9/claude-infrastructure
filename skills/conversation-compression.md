---
name: conversation-compression
description: Compress long conversations using RAPTOR-inspired hierarchical summarization
---

# Conversation Compression Skill

Compresses ChatGPT conversations that exceed token limits using recursive abstractive summarization, inspired by the RAPTOR paper (Stanford, ICLR 2024).

## Background

RAPTOR (Recursive Abstractive Processing for Tree-Organized Retrieval) achieves 72% compression while preserving both high-level themes and granular details. We adapt this for structured conversations.

## Compression Strategy

### Level Architecture

```
Level 2: Conversation Summary (theme + key insights)
    |
Level 1: Exchange Clusters (related exchanges summarized)
    |
Level 0: Raw Exchanges (User + Assistant turn pairs)
```

### Compression Modes

| Mode | Output | Use Case |
|------|--------|----------|
| `full` | Entire conversation | Under token limit |
| `summary` | Level 2 only | Quick classification |
| `hierarchical` | Level 1 + Level 2 | Detailed analysis |
| `adaptive` | Auto-select based on length | Default mode |

### Token Thresholds

| Threshold | Action |
|-----------|--------|
| < 3,000 tokens | Return full conversation |
| 3,000 - 8,000 | Hierarchical mode |
| 8,000 - 20,000 | Summary + key exchanges |
| > 20,000 | Summary only with metadata |

## Compression Process

### Step 1: Parse Conversation Structure

```yaml
metadata:
  title: <extracted from filename>
  date: <from file path>
  exchange_count: <number of User turns>

exchanges:
  - id: 1
    user: <user message>
    assistant: <assistant response>
    tokens: <estimated count>
```

### Step 2: Cluster Related Exchanges

Group exchanges by semantic similarity (topic continuity):
- Topic shifts create new clusters
- Follow-up questions stay in same cluster
- Tangents form separate clusters

### Step 3: Summarize Each Cluster (Level 1)

For each cluster, generate:
```yaml
cluster_summary:
  topic: <what this cluster discusses>
  key_points: <main insights/decisions>
  exchange_ids: [1, 2, 3]
```

### Step 4: Generate Conversation Summary (Level 2)

```yaml
conversation_summary:
  main_topic: <primary subject>
  themes: [<theme1>, <theme2>]
  key_insights:
    - <insight 1>
    - <insight 2>
  decisions_made: [<if any>]
  action_items: [<if any>]
  value_signal: <high|medium|low>
```

## Output Format

### Compressed Conversation Structure

```json
{
  "original_file": "2025/03/Salary Negotiation Strategy.md",
  "compression_mode": "hierarchical",
  "original_tokens": 15000,
  "compressed_tokens": 4200,
  "compression_ratio": 0.28,

  "level_2_summary": {
    "main_topic": "Career strategy and salary negotiation",
    "themes": ["negotiation tactics", "Go learning path", "AI architecture"],
    "key_insights": [
      "Market rate for Go developers is X",
      "Negotiation should emphasize unique value"
    ],
    "value_signal": "high"
  },

  "level_1_clusters": [
    {
      "cluster_id": 1,
      "topic": "Current salary situation",
      "summary": "Discussion of current comp and market positioning...",
      "exchange_ids": [1, 2, 3]
    },
    {
      "cluster_id": 2,
      "topic": "Go learning roadmap",
      "summary": "Recommended resources and timeline for Go proficiency...",
      "exchange_ids": [4, 5, 6, 7]
    }
  ],

  "preserved_exchanges": [
    // High-value exchanges kept verbatim
  ]
}
```

## Integration with Classification

The compression output feeds directly into classification:

```
Compressed Conversation
        ↓
  Level 2 Summary → Topic Classification
        ↓
  Level 1 Clusters → Type Detection (coaching vs quick-ref)
        ↓
  Exchange Count + Themes → Value Signal
```

## Summarization Prompt

Based on RAPTOR's approach:

```
System: You are a conversation summarizer that preserves key insights while compressing.

User: Summarize the following conversation exchanges, preserving:
- Key decisions and advice given
- Actionable insights
- Important context for understanding the discussion
- Any unique or personal information (projects, goals, preferences)

Exchanges:
{exchanges}

Output a concise summary (target: 100-150 tokens) that captures the essential content.
```

## Preservation Rules

### Always Preserve
- Project names (StatementStream, Phila, MulaMap)
- Specific numbers (salaries, dates, metrics)
- Technical architecture decisions
- Personal goals and strategies
- Unique insights not easily found elsewhere

### Can Compress
- Pleasantries and acknowledgments
- Repeated explanations
- Generic context-setting
- Standard definitions

## Usage

```
/compress <file_path> [--mode adaptive|summary|hierarchical|full]
```

## References

- RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval (ICLR 2024)
- Compression ratio: 72% (summary is 28% of original)
- Average summary: 131 tokens per cluster
