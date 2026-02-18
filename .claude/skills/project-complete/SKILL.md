---
name: project-complete
description: "Complete task: validate, document, review, commit, and merge to develop. Use when all implementation is done and ready to finalize."
model: claude-sonnet-4-20250514
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task
---

# /project-complete

Complete a task with full validation, documentation updates, reviews, and automatic merge to develop.

## Usage

```bash
/project-complete careerbrain 2026-02-18-auth        # Complete specific initiative
/project-complete careerbrain                         # Complete current/active initiative
```

## What It Does

1. Validate PLAN completion and spec compliance
2. Update all project documentation
3. Update project README (Initiatives table status)
4. Create final commit with doc changes
5. Run mandatory code review + security audit
6. Merge to develop branch
7. Clean up feature branch

**GitFlow Pattern**: `feature/*` → `develop` (automatic) | `develop` → `main` (requires PR)

## Prerequisites

- All implementation complete (code written)
- Active initiative with completed work
- PLAN.md exists with phases mostly complete

## Initiative Resolution

Find the initiative folder from the argument:

```bash
# If full folder name given:
Read: "06 Projects/[project]/2026-02-18-auth/PLAN.md"

# If partial name given, search:
Glob: "06 Projects/[project]/*-auth*/PLAN.md"

# If no initiative specified, find in-progress ones:
Glob: "06 Projects/[project]/[0-9][0-9][0-9][0-9]-*/TASK.md"
# Then check frontmatter for status: in_progress
```

## Execution Flow

### 1. Check PLAN.md Completion
```
✓ All phases complete (5/5)
  or
✗ Incomplete phases:
  - [ ] 1.3 - Implement session refresh
```

### 2. Spec Compliance Validation

**Acceptance Scenario Coverage:**
- Search tests matching each Given/When/Then scenario
- Create missing tests if needed

**Success Metrics Verification:**
- Check performance, coverage, accessibility

**Out-of-Scope Enforcement:**
- Scan for code that violates out-of-scope items

**Agent Constraints Check:**
- Verify no dependencies added (if constrained)
- Check no files outside scope modified

### 3. Update Spec Status Markers

If the TASK has an `implements:` field, update inline status markers in the spec:

```markdown
# Before (in initiative dir or project-wide spec)
- 🚧 User registration with email/password

# After
- ✅ User registration with email/password
```

This provides public visibility into what's implemented.

### 4. Update All Documentation

Scan and validate docs in the initiative folder and project root:
- Initiative-local docs (ADRs, specs, context)
- Project README.md

### 5. Final WORKLOG Entry

```markdown
## YYYY-MM-DD HH:MM - COMPLETED

Initiative 2026-02-18-auth complete and ready for merge.

Summary:
- [What was implemented]
- [What was deferred]
```

### 6. Update Status

Set issue frontmatter: `status: complete`

### 7. Update Project README

Update `06 Projects/[project]/README.md` Initiatives table:

```markdown
## Initiatives

| Initiative | Date | Status |
|------------|------|--------|
| [Auth Implementation](2026-02-17-auth/) | 2026-02-17 | complete |
| [API Redesign](2026-02-18-api-redesign/) | 2026-02-18 | open (SPIKE) |
```

Also update:
- **Recent Activity**: Add completion entry
- **Updated date**: Set frontmatter `updated:` to today's date

### 8. Create Final Commit

Stage and commit documentation changes.

### 9. Run Final Reviews

If not already done:
1. Launch code-reviewer agent
2. Launch security-auditor agent
3. Block if CRITICAL issues

### 10. Merge to Develop

```bash
git checkout develop
git pull origin develop
git merge --no-ff feature/initiative-slug
git push origin develop
git branch -d feature/initiative-slug
```

### 11. Suggest Next Steps

```
Next actions:
1. Start next initiative: /project-issue --project [project]
2. View status: /project-status-all [project]
3. Merge to main (requires PR): gh pr create --base main --head develop
```

## Documentation Checklist

Before merge, verify:
1. **PLAN.md** - All phases checked off
2. **Linked SPEC** - Updated to reflect what was built
3. **ADRs** - New decisions documented (in initiative dir)
4. **WORKLOG** - Final summary
5. **Project README.md** - Initiatives table updated

## Workflow

```
/project-issue → /project-plan → /project-implement
        ↓
/project-spec-validate ──────┐
/project-validate-quality ───┤  (quality gate, parallel)
/project-validate-security-audit ┘
        ↓
/project-commit → /project-complete
                        ↓
                Update docs + README Initiatives table
                        ↓
                Run reviews
                        ↓
                Merge to develop
```
