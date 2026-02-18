---
name: project-implement
description: "Execute plan phases, writing code in the project's code repo (from Projects Index) while tracking in 06 Projects/. Use after creating a plan with /project-plan to implement work."
model: claude-sonnet-4-20250514
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task
---

# /project-implement

Execute implementation phases from PLAN.md, writing code in the project's code repo (resolved from the Projects Index `code:` field in CLAUDE.md) while tracking progress in `06 Projects/`.

## Usage

```bash
/project-implement yourbench 2026-02-18-auth 1.1      # Execute phase 1.1
/project-implement yourbench 2026-02-18-auth --next   # Auto-find next uncompleted phase
/project-implement yourbench --next                    # Auto-detect initiative + next phase
/project-implement coordinatr 2026-02-18-api --full   # Execute all remaining phases
```

## The Bridge Pattern

```
06 Projects/yourbench/2026-02-18-auth/   /Users/duncanleung/Develop/yourbench/
├── TASK.md (requirements)            ├── src/
├── PLAN.md (phases)          ←→      │   └── auth/  ← CODE WRITTEN HERE
└── worklog/                          └── tests/
    ├── _state.json (current state)   (code path from Projects Index)
    └── 001-phase-*.json (entries)
```

## Prerequisites

**REQUIRED:**
- `YYYY-MM-DD-name/PLAN.md` must exist (under `06 Projects/[project]/`)
- Active initiative context (initiative name specified or inferable)
- Code repo exists at the path specified in Projects Index `code:` field

**If PLAN.md missing:** Run `/project-plan` first to create implementation plan

## Initiative Resolution

Find the initiative folder from the argument:

```bash
# If full folder name given:
Read: "06 Projects/[project]/2026-02-18-auth/PLAN.md"

# If partial name given, search:
Glob: "06 Projects/[project]/*-auth*/PLAN.md"

# If no initiative specified, list available with open status:
Glob: "06 Projects/[project]/[0-9][0-9][0-9][0-9]-*/PLAN.md"
```

Present available initiatives if multiple match or none specified.

## Execution Flow

### 1. Parse & Validate
- Locate initiative in `06 Projects/[project]/YYYY-MM-DD-name/`
- Resolve code repo path from Projects Index in CLAUDE.md
- Check git branch (warn if on main/develop)
- Check dependencies (warn if incomplete)

### 2. Load Worklog Context
Read `worklog/_state.json` for:
- Current phase progress
- Key decisions made
- Previous agent context
- Blockers

### 3. Branch & Status Management
On first phase:
- Create feature branch: `feature/initiative-slug` or `bugfix/initiative-slug`
- Update issue status to `in_progress`
- Initialize worklog directory

### 4. Execute Phase
1. Load context:
   - PLAN.md (phases and checkboxes)
   - Spec section from `implements:` field in TASK.md
   - ADRs and research docs from initiative dir
2. Select agent based on phase domain
3. Write code to the project's code repo
4. Run tests and quality gates
5. Create worklog entry
6. **Update PLAN.md checkboxes (Checkpoint Discipline)**
7. Update `_state.json`

### Checkpoint Discipline

**IMPORTANT:** After completing each sub-task, immediately mark the checkbox in PLAN.md:
```markdown
- [ ] 1.1 Create auth module    →    - [x] 1.1 Create auth module
```
- Update after **each sub-task**, not just after completing an entire phase
- This creates a live progress dashboard in PLAN.md
- Never skip this step — it ensures no sub-tasks are missed and enables `--next` mode to find the right resumption point

### 5. Agent Coordination

| Phase Type | Primary Agent |
|------------|---------------|
| Frontend UI | frontend-specialist |
| Backend API | backend-specialist |
| Database | database-specialist |
| Tests (RED) | test-engineer |
| Refactor | code-reviewer |

### 6. Per-Phase Security Checks (Conditional)

Trigger security-auditor after phase if touching:
- Authentication/Authorization
- Secrets/API keys
- Database operations
- File operations
- External APIs
- Deployment config

### 7. Worklog Entries (MANDATORY)

After each phase, create JSON entry:
- Filename: `{sequence:03d}-phase-{slug}.json`
- Required: `type`, `phase`, `author`, `summary`, `work`, `learnings`, `next_steps`, `blockers`

### 8. Final Review (--full mode)

When all phases complete:
1. Launch code-reviewer agent
2. Launch security-auditor agent
3. Process combined results
4. Block if CRITICAL issues

## Modes

### Specific Phase
```bash
/project-implement yourbench 2026-02-18-auth 1.1  # Execute only phase 1.1
```

### Next Phase
```bash
/project-implement yourbench 2026-02-18-auth --next  # Find first uncompleted checkbox
```

### Full Run
```bash
/project-implement yourbench 2026-02-18-auth --full  # All remaining phases + final review
```

## Flags

| Flag | Purpose |
|------|---------|
| `--full` | Execute all remaining phases |
| `--next` | Auto-detect next phase |
| `--skip-branch-check` | Skip branch warnings |
| `--skip-security-checks` | Skip per-phase security |
| `--skip-reviews` | Skip final reviews |
| `--force` | Skip dependency warnings |

## Workflow

```
/project-spec → /project-issue → /project-plan → /project-implement
                    ↓                                    ↓
             implements:                           Load spec section
             spec section                          for requirements
                                                         ↓
                                               worklog/ entries created
                                                         ↓
                                               code written to project repo
                                                         ↓
                                        /project-spec-validate ──────┐
                                        /project-validate-quality ───┤  (quality gate)
                                        /project-validate-security-audit ┘
                                                         ↓
                                        /project-commit → /project-complete
```

## Spec Compliance

When implementing, the code should fulfill the requirements from the spec section referenced in the issue's `implements:` field. The `/project-complete` command will validate that all spec requirements are met.
