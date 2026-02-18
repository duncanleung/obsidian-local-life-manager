---
name: project-next-step
description: "Assess project readiness and recommend the next skill to run. Use when unsure what to do next on a specific project."
model: claude-haiku-4-5-20251001
allowed-tools: Read, Glob, Grep
---

# /project-next-step

Assess project readiness and recommend the next skill to run.

## Usage

```bash
/project-next-step coordinatr       # What's next for this project?
/project-next-step yourbench        # Check another project
```

## Validation Checklist

### Required Files (Minimum Viable Idea)

| File | Required | Purpose |
|------|----------|---------|
| **README.md** | Yes | Status, overview, progress |
| **project-brief.md** | Yes | Vision, problem, audience, solution |

### Recommended Files (Phase-Dependent)

| File/Directory | When Needed | Purpose |
|----------------|-------------|---------|
| **features/README.md** | Before creating issues | Feature map with dependency graph and scope |
| **features/F-*.md** | Before creating issues | Individual feature cards with stories and AC |
| **YYYY-MM-DD-*/critique.md** | Before planning | Risk assessment |
| **specs/spec-*.md** | Imported external specs | External requirements with compliance checklist |
| **YYYY-MM-DD-*/ADR-*.md** | Major tech decisions | Architecture Decision Records |
| **YYYY-MM-DD-*/TASK.md** | In development | Work tracking (initiatives) |

## Phase-Aware Validation

**Concept Phase:**
- README + project-brief.md sufficient
- critique.md optional (recommend before planning)

**Features Phase:**
- Should have `features/` directory with feature cards
- Walking skeleton and MVP scope defined in `features/README.md`
- critique.md optional (recommend before significant investment)

**Planning Phase:**
- Should have features with stories and acceptance criteria
- Should have initiative folders with TASK.md files

**Development/Implementation Phase:**
- Must have initiative folders with PLAN.md files
- Should have ADRs in initiative folders if major decisions made

## Execution Flow

### 1. Locate Project
```bash
ls "06 Projects"/[project-name]/
```

### 2. Check Required Files
- README.md: Has status, last updated date, progress
- project-brief.md: Vision, problem, audience, solution complete

### 3. Check Recommended Files (Phase-Aware)
Based on project phase in README.

### 4. Verify Consistency
- README status matches CLAUDE.md
- Brief aligns with README description
- Specs reference features
- Issues link to specs

### 5. Suggest Next Steps

| Current State | Suggested Next Step |
|---------------|---------------------|
| Just README | Run `/project-brief` |
| Has brief | Run `/project-features` |
| Has features | Run `/project-issue` (for MVP features first) |
| Has issues | Run `/project-plan` |
| Has plans | Run `/project-implement` |

**Optional steps (suggest when relevant, not mandatory):**
- `/project-critique` -- when brief is complete and project is ambitious
- `/research-deep` -- when domain is unfamiliar
- `/project-import-spec` -- when implementing external PRD/standard

## Validation Report

```markdown
# Validation Report: [Project Name]

## Status
- Current phase: [Concept / Planning / Development]
- Documentation completeness: X/Y files

## Required Files
✅ README.md - Complete
✅ project-brief.md - Complete

## Recommended Files
⚠️  No critique found (run /project-critique)
✅ 2026-02-18-auth/spec-required-features.md - Present

## Issues Found
1. README last updated is stale
2. Status mismatch with CLAUDE.md

## Recommendations
1. Update README last updated
2. Run /critique before specs

## Readiness Assessment
- Ready for specs: ⚠️ After fixing issues
- Ready for implementation: ❌ No specs yet
- Overall health: 7/10
```

## Readiness Criteria

### Ready for /project-features
- project-brief.md complete
- Vision and problem clearly stated

### Ready for /project-issue
- At least one feature card complete (in `features/`)
- MVP scope defined in `features/README.md`
- Walking skeleton identified

### Ready for /project-plan
- At least one initiative with TASK.md
- Acceptance criteria clear (from feature card)
- Technical decisions made (ADRs if needed)

## When to Use

- Before starting spec work
- After long pause in project
- Monthly project health checks
- Before presenting to stakeholders
- When unsure what to do next

## Integration

```
/project-next-step → Fix issues → /project-next-step again → /project-features or /project-issue
```
