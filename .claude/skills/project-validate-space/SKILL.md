---
name: project-validate-space
description: "Validate project space structure, boilerplate docs, and consistency with 06 Projects/"
model: claude-haiku-4-5-20251001
allowed-tools: Read, Glob, Grep, Bash
argument-hint: <project-name>
---

# /project-validate-space

Validate that a project space has all required structure, boilerplate docs, and stays consistent with its planning docs in 06 Projects/.

## Usage

```bash
/project-validate-space leaf-nextjs-convex    # Validate specific space
/project-validate-space coordinatr            # Another project
/project-validate-space                       # Prompt for project name
```

## Validation Checklist

### Required Files (Every Space)

| File | Purpose | Check |
|------|---------|-------|
| **CLAUDE.md** | AI instructions for codebase | Must exist |
| **README.md** | Entry point for developers | Must exist |
| **package.json** (JS/TS) | Project config | Stack-dependent |

### Initiative Structure

Initiative folders use `YYYY-MM-DD-name/` convention. No intermediate directories (`issues/`, `docs/`, `critiques/`) at project root.

| Check | Description |
|-------|-------------|
| **No `issues/` dir** | Should use initiative folders instead |
| **No `docs/` dir** | Specs/ADRs live inside initiative folders |
| **No `critiques/` dir** | Critiques live inside initiative folders |
| **Initiative naming** | All dated dirs match `YYYY-MM-DD-*` pattern |
| **Work item present** | Each initiative has TASK.md, BUG.md, or SPIKE.md |

### CLAUDE.md Requirements

- Overview section with stack description
- Project structure section
- Commands section (dev, build, deploy)
- Environment variables section (if applicable)
- Link to 06 Projects/ planning docs

### Consistency Checks

| Check | Description |
|-------|-------------|
| **Version sync** | package.json versions match docs (e.g., "Next.js 16" in CLAUDE.md matches `"next": "16.x"`) |
| **Stack accuracy** | Listed technologies actually exist in dependencies |
| **Structure accuracy** | Documented directories actually exist |
| **Ideas link** | Referenced 06 Projects/[project]/ exists and has matching info |

### Cross-Reference with 06 Projects/

| Check | Description |
|-------|-------------|
| **README.md** | Stack listed in 06 Projects/ matches actual code repo |
| **README Initiatives table** | Initiatives table exists and statuses are accurate |
| **project-brief.md** | Technical decisions match actual implementation |

## Execution Flow

### 1. Locate Project

```bash
ls "06 Projects/[project-name]/"
```

Error if not found.

### 2. Check Required Files

```
Read: 06 Projects/[project]/CLAUDE.md
Read: 06 Projects/[project]/README.md
Read: 06 Projects/[project]/package.json (if JS/TS)
```

### 3. Validate CLAUDE.md Sections

Check for required sections:
- Overview / Stack
- Project Structure
- Commands
- Environment Variables (if .env.example exists)

### 4. Check Version Consistency

Extract versions from:
- CLAUDE.md stack description
- 06 Projects/[project]/README.md
- 06 Projects/[project]/project-brief.md
- package.json dependencies

Flag any mismatches.

### 5. Verify Initiative Structure

Check initiative folders follow convention:
```bash
Glob: "06 Projects/[project]/[0-9][0-9][0-9][0-9]-*"
```

For each initiative, verify work item exists:
```bash
Glob: "06 Projects/[project]/YYYY-MM-DD-name/TASK.md"
Glob: "06 Projects/[project]/YYYY-MM-DD-name/BUG.md"
Glob: "06 Projects/[project]/YYYY-MM-DD-name/SPIKE.md"
```

Flag legacy directories that should not exist:
```bash
# These are old convention — should be migrated
ls "06 Projects/[project]/issues/" 2>/dev/null && echo "⚠️ Legacy issues/ dir found"
ls "06 Projects/[project]/docs/" 2>/dev/null && echo "⚠️ Legacy docs/ dir found"
ls "06 Projects/[project]/critiques/" 2>/dev/null && echo "⚠️ Legacy critiques/ dir found"
ls "06 Projects/[project]/context/" 2>/dev/null && echo "⚠️ Legacy context/ dir found"
```

### 6. Cross-Reference 06 Projects/

```
Read: 06 Projects/[project]/README.md
Read: 06 Projects/[project]/project-brief.md
```

Check stack/version consistency.

## Validation Report

```markdown
# Space Validation: [Project Name]

## Status
- Space location: 06 Projects/[project]/
- Ideas location: 06 Projects/[project]/ (exists/missing)

## Required Files
✅ CLAUDE.md - Present
✅ README.md - Present (has Initiatives table)
✅ package.json - Present

## Initiative Structure
✅ No legacy issues/ directory
✅ No legacy docs/ directory
✅ No legacy critiques/ directory
✅ 3 initiative folders found
  - 2026-02-17-auth/ (TASK.md ✅, PLAN.md ✅)
  - 2026-02-18-api-redesign/ (SPIKE.md ✅)
  - 2026-02-18-ui-refresh/ (TASK.md ✅)

## CLAUDE.md Sections
✅ Overview/Stack - Complete
✅ Project Structure - Complete
⚠️  Commands - Missing deploy command
✅ Environment Variables - Complete

## Version Consistency
✅ Next.js: 16.1.3 (package.json) matches "Next.js 16" (docs)
❌ React: 19.0.0 (package.json) but docs say "React 18"

## Ideas Cross-Reference
✅ 06 Projects/leaf-nextjs-convex/ exists
✅ Stack matches between code repo and 06 Projects/
⚠️  project-brief.md says "Next.js 15" - outdated

## Issues Found
1. React version mismatch in documentation
2. project-brief.md has outdated version

## Recommendations
1. Update React version in CLAUDE.md
2. Update project-brief.md to say Next.js 16
3. Fill in overview doc templates with project-specific content
```

## Fixing Issues

If legacy directories found, suggest migration:
```
⚠️ Legacy issues/ directory found. Migrate to initiative folders:
   issues/001-auth/ → 2026-02-17-auth/
   Run /project-issue to create new initiatives.
```

## When to Use

- After initial project scaffolding
- Before starting implementation work
- After upgrading dependencies
- Monthly maintenance checks
- When onboarding to existing project

## Integration

```
/project-validate-space → Fix issues → /project-validate-space again → /implement
```

## Stack-Specific Checks

### Next.js Projects
- Check for `next.config.js` or `next.config.ts`
- Verify `src/app/` structure for App Router
- Check for `public/` directory

### Convex Projects
- Check for `convex/` directory
- Verify `convex/schema.ts` exists
- Check for `convex/_generated/`

### General JS/TS
- Verify `tsconfig.json` if TypeScript
- Check for `.env.example` if env vars documented
- Verify `.gitignore` exists
