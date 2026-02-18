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
3. With `--project` or `--detailed`: also parse initiative folders, specs, git branches

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
Initiatives: 3 (1 in_progress, 2 open)
  - 2026-02-17-auth-research (TASK, in_progress) ← feature/auth-research
  - 2026-02-18-data-model (TASK, open, blocked by auth-research)
  - 2026-02-18-api-design (SPIKE, open)
Branch: feature/auth-research (3 unpushed commits)

### CareerBrain
Status: MVP in progress (60%)
Initiatives: 1 (1 in_progress)
  - 2026-02-18-dashboard (TASK, in_progress)
Branch: feature/dashboard (clean, up to date)

## On Hold

| Project    | Reason                 |
|------------|------------------------|
| YourBench  | Job search priority    |
| IRL Social | Waiting on API access  |

## Feature Progress (if features/ exists)

| Feature | Priority | Stories | Tasks | Done | Status |
|---------|----------|---------|-------|------|--------|
| F-001 Auth | must-have | 3 | 2 | 1/2 | 🚧 50% |
| F-002 Documents | must-have | 4 | 0 | 0/0 | ⏳ not started |

MVP progress: 1/4 features complete (25%)

## Needs Attention
- Coordinatr 2026-02-18-data-model blocked (waiting on auth-research)
- Coordinatr 2026-02-18-api-design SPIKE past 3-day timebox
- CareerBrain feature/dashboard has 3 unpushed commits

## Quick Stats
- 4 projects total (2 active, 2 on hold)
- 4 open initiatives (2 in_progress, 1 blocked, 1 pending)

## Suggested Next Actions
1. Complete Coordinatr auth-research to unblock data-model
2. Push CareerBrain feature/dashboard
3. Close or extend Coordinatr api-design SPIKE
```

## Execution Steps

### 1. Scan Repository Structure

```bash
ls "06 Projects"/
# For each: README.md, YYYY-MM-DD-*/ initiative folders
```

### 2. Parse Project Status

For each idea folder:
1. Read README.md for status
2. Build quick glance table (default mode stops here)
3. With `--detailed` or `--project`: discover initiative folders (`YYYY-MM-DD-*/`), analyze work items, check PLAN.md progress

### 3. Parse Dependencies (detailed mode)

Read `depends_on` from work item frontmatter:
```yaml
depends_on: [2026-02-17-auth]
```

Auto-block detection: If depends on incomplete initiatives, flag as blocked.

### 4. Check Branch Status (detailed mode)

For in_progress initiatives:
```bash
# cd to code repo path from Projects Index
git branch -a | grep "feature/"
git log origin/branch..branch  # Unpushed commits
```

### 5. Identify Attention Items

- Initiatives with status: blocked
- Initiatives blocked by dependencies
- Stale initiatives (no activity 14+ days)
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
