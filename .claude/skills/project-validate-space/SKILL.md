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

### Required Directory Structure

| Directory | Purpose | Check |
|-----------|---------|-------|
| **docs/** | Documentation root | Must exist |
| **docs/specs/** | Protocol/feature specs | Must exist |
| **docs/adrs/** | Architecture Decision Records | Must exist |

### Required Overview Docs (in docs/)

| File | Purpose |
|------|---------|
| **architecture-overview.md** | System architecture |
| **api-overview.md** | API documentation |
| **data-model.md** | Data structures |
| **deployment.md** | Deployment guide |
| **security.md** | Security considerations |
| **testing-overview.md** | Testing strategy |
| **ui-guide.md** | UI patterns and components |

Templates available at `SHARED/TEMPLATES/DOCS/`

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
| **project-brief.md** | Technical decisions match actual implementation |
| **Issues** | Current phase/status is accurate |

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

### 5. Verify Directory Structure

Check required directories exist:
```bash
ls -la "06 Projects/[project]/docs/"
ls -la "06 Projects/[project]/docs/specs/"
ls -la "06 Projects/[project]/docs/adrs/"
```

Check overview docs present:
```bash
ls "06 Projects/[project]/docs/"*.md
# Should have: architecture-overview.md, api-overview.md, data-model.md,
#              deployment.md, security.md, testing-overview.md, ui-guide.md
```

Compare documented structure in CLAUDE.md against actual:
```bash
ls -la "06 Projects/[project]/"
ls -la "06 Projects/[project]/src/" (if documented)
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
✅ README.md - Present
✅ package.json - Present

## Required Directories
✅ docs/ - Present
✅ docs/specs/ - Present
✅ docs/adrs/ - Present

## Overview Docs (in docs/)
✅ architecture-overview.md - Present
✅ api-overview.md - Present
✅ data-model.md - Present
✅ deployment.md - Present
✅ security.md - Present
✅ testing-overview.md - Present
✅ ui-guide.md - Present

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

## Fixing Missing Structure

If docs/ structure is missing, create it:
```bash
mkdir -p "06 Projects/[project]/docs/specs"
mkdir -p "06 Projects/[project]/docs/adrs"
cp SHARED/TEMPLATES/DOCS/*.md "06 Projects/[project]/docs/"
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
