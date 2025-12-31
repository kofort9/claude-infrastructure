---
name: info-hub
description: Capability registry for agent ecosystem. Query what agent handles a specific task. Enables loose coupling between agents.
arguments: <capability query>
---

# Info Hub Skill

A capability registry that answers "What agent can do X?" - enabling agents to discover capabilities without hardcoding knowledge of each other.

## Usage

```
/info-hub git branch management
/info-hub PR descriptions
/info-hub code security review
/info-hub session checkpoints
```

## Why This Exists

**Problem**: Agents hardcoding knowledge of other agents creates tight coupling. When agent responsibilities change, every agent that knows about it needs updating.

**Solution**: Agents query `/info-hub` for capabilities. Only this registry needs updating when responsibilities shift.

```
Before (tight coupling):
  code-reviewer knows → tech-writer handles docs
  gitops-devex knows → code-reviewer handles reviews
  tech-writer knows → gitops-devex handles PRs

After (loose coupling via info-hub):
  any agent → /info-hub "who handles X?" → routing answer
```

## Capability Registry

### Git & GitHub Operations

| Capability | Agent | Notes |
|------------|-------|-------|
| Branch management | `gitops-devex` | Create, switch, health checks |
| Worktree isolation | `gitops-devex` | Parallel feature development |
| Pre-commit validation | `gitops-devex` | Hard gate, no bypass |
| Pre-push validation | `gitops-devex` | Hard gate, no bypass |
| Force push | **FORBIDDEN** | Never allowed via agents |
| PR creation | `gitops-devex` | Spawns tech-writer for description |
| PR review (automated) | `gitops-devex` | Review loop with code-reviewer |
| Commit creation | `gitops-devex` | After gates pass |

### Code Quality

| Capability | Agent/Skill | Notes |
|------------|-------------|-------|
| Code review | `code-reviewer` | Standalone or review-loop mode |
| Security review | `code-reviewer` | Part of standard review |
| PR review comments | `code-reviewer` | Classification tags for automation |
| Review classification | `/review-classify` | Parse tags → action plan with next steps |
| Review decisions log | `/review-classify` | Captures decisions for pattern learning |
| Architecture analysis | `repo-topology` | Codebase structure, dependencies |

### Documentation

| Capability | Agent | Notes |
|------------|-------|-------|
| README/docs | `tech-writer` | Owns /repo-documentation |
| PR descriptions | `tech-writer` | From PR_SCOPE.md + commits |
| API documentation | `tech-writer` | Interface docs |
| Migration guides | `tech-writer` | Breaking change docs |

### System Operations

| Capability | Agent | Notes |
|------------|-------|-------|
| Quick system tasks | `system-ops` | Run existing scripts, checks |
| Complex system projects | `system-admin` | Build scripts, audits |
| Environment setup | `system-admin` | Configuration management |

### Orchestration & Meta

| Capability | Agent | Notes |
|------------|-------|-------|
| Task routing | `orchestrator` | Multi-agent coordination |
| Capability queries | `orchestrator` | Uses this skill |
| Workflow design | `orchestrator` | Sequential/parallel patterns |
| Session checkpoints | `orchestrator` | Uses /checkpoint skill |
| Agent audit | `orchestrator` | Uses /workflow-map skill |

## Response Format

When queried, respond with:

```markdown
**Capability**: [what was asked]
**Agent**: [agent name]
**How to invoke**: [Task tool with agent type, or skill]
**Notes**: [any caveats or related capabilities]
```

## Example Queries & Responses

**Query**: `/info-hub create isolated workspace for feature`

**Response**:
```
Capability: Isolated workspace for feature development
Agent: gitops-devex
How to invoke: /worktree-create <feature-name>
Notes: Creates worktree at ~/Repos/{repo}-worktrees/{feature}/, runs health check, creates PR_SCOPE.md template
```

---

**Query**: `/info-hub who writes PR descriptions`

**Response**:
```
Capability: PR description generation
Agent: tech-writer
How to invoke: Spawned by gitops-devex during PR creation, or directly via Task tool
Notes: Reads PR_SCOPE.md from worktree + analyzes commits. Structured format with Summary/Changes/Why/Testing/Risks.
```

---

**Query**: `/info-hub security vulnerabilities in code`

**Response**:
```
Capability: Security vulnerability detection
Agent: code-reviewer
How to invoke: Task tool with code-reviewer agent
Notes: Checks OWASP top 10, injection attacks, auth flaws, data exposure. Part of standard review. For agent-specific security (permissions, prompt injection), see future agent-sentinel.
```

## Updating This Registry

When agent responsibilities change:
1. Update this skill's capability tables
2. No changes needed in individual agents
3. Agents querying /info-hub automatically get correct routing

## Integration

Any agent can use this skill:
```
Skill(skill: "info-hub", args: "capability query here")
```

Orchestrator uses this as primary routing mechanism. Other agents use it when uncertain about delegation.
