---
name: project-status-all
description: "Project status dashboard — quick glance table of all projects plus detailed analysis. Use for session start context, weekly reviews, or seeing what needs attention."
model: claude-haiku-4-5-20251001
allowed-tools: Read, Glob, Grep, Task
---

# /project-status-all

Status dashboard across all projects. Default mode shows a quick scannable table from README files. Use `--project` or `--detailed` for deeper analysis including specs, issues, dependencies, blockers, and git branches.

## Usage

```bash
/project-status-all                       # Quick glance table of all projects
/project-status-all --project coordinatr  # Deep dive on one project
/project-status-all --detailed            # Comprehensive analysis of all projects
```

## Data Sources

1. Read `CLAUDE.md` for project index
2. For each project, read `06 Projects/[project]/README.md`
3. With `--project` or `--detailed`: also parse specs, issues, git branches

## Example Output (Default)

```
# Project Status Dashboard

## Active Projects

| Project      | Phase           | Last Activity | Next Step              |
|--------------|-----------------|---------------|------------------------|
| CareerBrain  | MVP in progress | 2026-02-10    | Complete auth flow     |
| Coordinatr   | Active planning | 2026-02-08    | Finish SPEC-002        |

## On Hold

| Project    | Reason                 |
|------------|------------------------|
| YourBench  | Job search priority    |
| IRL Social | Waiting on API access  |

## Quick Stats
- Active: 2
- On hold: 2
- Total: 4
```

## Example Output (`--detailed` or `--project`)

```
# Project Status Dashboard (Detailed)

## Active Projects

### Coordinatr
Status: Active planning
Specs: 2 (1 complete, 1 in progress)
Issues: 3 open (1 in_progress, 2 pending)
  - 001-auth-research (TASK, in_progress) ← feature/001-auth
  - 002-data-model (TASK, pending, blocked by 001)
  - 003-api-design (SPIKE, pending)
Branch: feature/001-auth (3 unpushed commits)

### CareerBrain
Status: MVP in progress (60%)
Specs: 1 (complete)
Issues: 1 open (1 in_progress)
  - YB-2-dashboard (TASK, in_progress)
Branch: feature/YB-2 (clean, up to date)

## On Hold

| Project    | Reason                 |
|------------|------------------------|
| YourBench  | Job search priority    |
| IRL Social | Waiting on API access  |

## Needs Attention
- Coordinatr TASK-002 blocked (waiting on TASK-001)
- Coordinatr SPIKE-003 past 3-day timebox
- CareerBrain feature/YB-2 has 3 unpushed commits

## Quick Stats
- 4 projects total (2 active, 2 on hold)
- 3 specs across all projects
- 4 open issues (2 in_progress, 1 blocked, 1 pending)

## Suggested Next Actions
1. Complete Coordinatr TASK-001 to unblock TASK-002
2. Push CareerBrain feature/YB-2
3. Close or extend Coordinatr SPIKE-003
```

## Execution Steps

### 1. Scan Repository Structure

```bash
ls "06 Projects"/
# For each: README.md, specs/, issues/, docs/
```

### 2. Parse Project Status

For each idea folder:
1. Read README.md for status
2. Build quick glance table (default mode stops here)
3. With `--detailed` or `--project`: count specs, analyze issues, check PLAN.md progress

### 3. Parse Dependencies (detailed mode)

Read `depends_on` from issue frontmatter:
```yaml
depends_on: [001, 002]
```

Auto-block detection: If depends on incomplete issues, flag as blocked.

### 4. Check Branch Status (detailed mode)

For in_progress issues:
```bash
# cd to code repo path from Projects Index
git branch -a | grep "feature/###"
git log origin/branch..branch  # Unpushed commits
```

### 5. Identify Attention Items

- Issues with status: blocked
- Issues blocked by dependencies
- Stale issues (no activity 14+ days)
- Incomplete spikes past time box
- Branches with unpushed commits

### 6. Generate Recommendations

- Next logical step for active work
- Items to unblock
- Stale items to review

## Status Taxonomy

### Project Status
- Initial brainstorming
- Active brainstorming
- Active planning
- Concept phase
- Pre-MVP
- MVP in progress (X%)
- Portfolio-first
- Shelved
- Graduated

### Issue Status
- `open` - Not started
- `in_progress` - Currently working
- `blocked` - Waiting
- `complete` - Done

## When to Use

- Quick check on all projects (default mode)
- Session start (get context)
- Before planning (see what's active)
- Weekly review (find stale items)
- After completing work (see what's next)
