---
name: project-worklog
description: "Add timestamped work log entries to track progress and decisions. Use for documenting work, decisions, gotchas, and handoffs between agents."
model: claude-sonnet-4-20250514
allowed-tools: Read, Write, Edit, Glob
---

# /worklog

Add structured JSON entries to track work, decisions, and learnings.

## Usage

```bash
/worklog yourbench 2026-02-18-auth "Added login button to header"
/worklog yourbench 2026-02-18-auth --decision "Using Clerk for auth"
/worklog yourbench 2026-02-18-auth --gotcha "Token refresh needs cleanup"
/worklog coordinatr 2026-02-18-api-redesign --handoff code-reviewer "Ready for review"
/worklog yourbench 2026-02-18-auth --state              # Show current state
/worklog yourbench 2026-02-18-auth --migrate            # Migrate from WORKLOG.md
```

## Directory Structure

```
06 Projects/yourbench/2026-02-18-auth/
├── TASK.md
├── PLAN.md
└── worklog/
    ├── _state.json              # Current state (quick context load)
    ├── 001-phase-init.json      # Entry files
    └── 002-handoff-review.json
```

## Initiative Resolution

Find the initiative folder from the argument:

```bash
# If full folder name given:
Glob: "06 Projects/[project]/2026-02-18-auth/worklog/"

# If partial name given, search:
Glob: "06 Projects/[project]/*-auth*/worklog/"

# If no initiative specified, list available:
Glob: "06 Projects/[project]/[0-9][0-9][0-9][0-9]-*/TASK.md"
Glob: "06 Projects/[project]/[0-9][0-9][0-9][0-9]-*/BUG.md"
Glob: "06 Projects/[project]/[0-9][0-9][0-9][0-9]-*/SPIKE.md"
```

Present available initiatives if multiple match or none specified.

## Entry Types

| Type | Flag | Use Case |
|------|------|----------|
| Manual | (default) | General progress update |
| Decision | `--decision` | Document architectural choice |
| Gotcha | `--gotcha` | Capture lesson learned |
| Handoff | `--handoff TO` | Agent transition |
| Phase | `--phase NUM` | Phase completion |
| Blocker | `--blocker` | Record impediment |
| Resolution | `--resolve ID` | Resolve blocker |

## Execution Flow

### 0. Resolve Project

If no project argument is provided, resolve it using the [Project Discovery](../project-shared/references/project-discovery.md) procedure:
- Auto-detect: `basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"`
- Ensure target directory exists: `mkdir -p` the `06 Projects/[project]/` path in the obsidian vault

### 1. Parse Arguments

```
/worklog PROJECT INITIATIVE [--type] "message"
```

### 2. Locate Worklog Directory

```bash
"06 Projects"/[project]/YYYY-MM-DD-name/worklog/
mkdir -p [path] if missing
```

### 3. Get Next Sequence Number

```bash
ls worklog/*.json | grep -v _state | wc -l
# Next = count + 1
```

### 4. Create Entry File

Filename: `{sequence:03d}-{type}-{slug}.json`

**Required fields:**
- `$schema`: "worklog-entry-v1"
- `id`: "INITIATIVE-SEQ"
- `sequence`: number
- `timestamp`: ISO 8601
- `type`: entry type
- `author`: { agent: string | null, human: string | null }
- `summary`: description

### 5. Update `_state.json`

After every entry:
- Update `last_entry`
- Update `last_updated`
- Increment `entries_count`
- Add to `key_decisions` if decision
- Update `blockers` if blocker/resolution

## Viewing State

```bash
/worklog yourbench 2026-02-18-auth --state
```

Outputs:
```
Initiative: 2026-02-18-auth - Initialize Next.js project
Status: in_progress (Phase 3)
Progress: 5/5 phases complete
Key Decisions: ...
Blockers: none
```

## Schema Reference

See [references/schema.md](references/schema.md) for full JSON schema specification.

## Best Practices

1. **Be specific**: Include enough context for future AI
2. **Tag consistently**: Use established tag taxonomy
3. **Capture gotchas immediately**: Don't wait until end
4. **Handoff explicitly**: Create handoff entry when switching agents
5. **Update state**: `_state.json` should always reflect current reality
