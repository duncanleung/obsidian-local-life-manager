---
name: project-init-space
description: "Initialize project planning structure in 06 Projects/ and optionally docs structure"
model: claude-sonnet-4-20250514
allowed-tools: Read, Write, Edit, Glob, Bash
argument-hint: <project-name> [--type next|python|empty]
---

# /project-init-space

Initialize a new project's planning structure in `06 Projects/`. Only creates `README.md` and `project-brief.md` at the project root — initiative folders are created on-demand by `/project-issue`. Code repos live externally (e.g., `/Users/duncanleung/Develop/[project]/`) and are referenced via the Projects Index `code:` field in CLAUDE.md.

## Usage

```bash
/init-space coordinatr              # Initialize project planning space
/init-space new-project --type next # Initialize with code repo scaffold
/init-space myapp --type python     # Python project
```

## Supported Project Types

| Type | Command | Creates |
|------|---------|---------|
| `next` | `create-next-app` | Next.js with App Router, TypeScript, Tailwind |
| `python` | `uv init` or manual | Python with pyproject.toml |
| `empty` | manual | Just planning structure |

## Standard Structure

Every project in `06 Projects/` starts minimal — no empty scaffolding dirs:

```
06 Projects/[project]/
├── README.md              # Project index with Initiatives table
└── project-brief.md       # Vision, problem, audience
```

Initiative folders are created just-in-time by other skills:
- `/project-issue` creates `YYYY-MM-DD-name/TASK.md` (or BUG/SPIKE)
- `/project-plan` creates `YYYY-MM-DD-name/PLAN.md`
- `/project-adr` creates `YYYY-MM-DD-name/ADR-###.md`
- `/project-spec` creates `YYYY-MM-DD-name/spec-*.md`

## Execution Flow

### 1. Validate Project

```bash
# Check if project directory already exists
ls "06 Projects/[project]/"
```

### 2. Create Planning Structure

```bash
mkdir -p "06 Projects/[project]/"
```

Create these files:

**README.md:**
```markdown
# [Project Name]

[Brief description]

## Status

| Area | Status |
|------|--------|
| Project | Active |
| Code Repo | `/Users/duncanleung/Develop/[project]/` |
| Remote | `[to be added]` |
| Branch | `main` |

## Initiatives

| Initiative | Date | Status |
|------------|------|--------|
| (none yet) | - | - |

## Recent Activity

- YYYY-MM-DD: Project initialized
```

**project-brief.md:**
```markdown
---
status: draft
created: YYYY-MM-DD
---

# Project Brief: [Project Name]

## Vision

[What is this project and why does it matter?]

## Problem

[What problem does this solve?]

## Solution

[How does this project solve the problem?]

## Audience

[Who is this for?]

## Scope

### In Scope
- [Feature 1]

### Out of Scope
- [Explicitly excluded]
```

### 3. Initialize Code Repo (if --type specified)

For `--type next|python`, scaffold the code repo at `/Users/duncanleung/Develop/[project]/`.

**CLAUDE.md** (in code repo):
```markdown
# [Project Name]

Project-specific instructions for AI assistants.

## Overview

[Brief description from project-brief.md if exists]

## Development

```bash
pnpm dev        # Start dev server
pnpm test       # Run tests
pnpm build      # Production build
```

## Key Files

| File | Purpose |
|------|---------|
| `src/app/` | Next.js app routes |

## Conventions

- [Project-specific conventions]

## Related Documentation

- Planning: `06 Projects/[project]/` in obsidian vault
```

### 4. Initialize Git (if not exists, code repo only)

```bash
git init
```

### 5. Stage Files (DO NOT COMMIT)

```bash
git add .
git status
```

**CRITICAL**: Do NOT commit. Show user the staged files and remind them:

```
Files staged and ready. When you're ready to commit, run:
/project-commit [project]
```

## Safety Rules

1. **NEVER commit automatically** - only stage files
2. **NEVER overwrite existing space** - error if directory exists
3. **No empty scaffolding dirs** - initiative dirs created on-demand by other skills

## Post-Init Checklist

Display to user after init:

```markdown
## Space Initialized: [project]

### Created
- [ ] README.md with Initiatives table
- [ ] project-brief.md (draft)
- [ ] Code repo scaffolded (if --type specified)
- [ ] CLAUDE.md in code repo (if --type specified)
- [ ] Git initialized (if --type specified)
- [ ] Files staged (not committed)

### Next Steps
1. Fill in project-brief.md
2. Add to Projects Index in CLAUDE.md
3. Create first initiative: `/project-issue --project [project]`
```

## Error Handling

| Error | Resolution |
|-------|------------|
| Project already exists | Error: "Project already exists at 06 Projects/[project]/" |
| Scaffold fails | Clean up partial directory, show error |

## Integration with Projects Index

After successful init, remind user to add to CLAUDE.md Projects Index if not present:

```yaml
- name: [project]
  planning: 06 Projects/[project]/
  code: /Users/duncanleung/Develop/[project]/
  remote: [to be added]
  branch: main
  status: active
```
