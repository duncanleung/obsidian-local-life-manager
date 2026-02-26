---
name: project-import-spec
description: "Import external requirements (JIRA epic, Confluence doc, GitHub spec, URL) into the project. Use when implementing a PM's PRD or external standard."
model: claude-sonnet-4-20250514
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch
---

# /project-import-spec

Import external requirements you didn't write -- a PM's PRD from JIRA, a Confluence doc, a protocol spec from GitHub, or any URL.

## Usage

```bash
/project-import-spec JIRA-123                              # Import from JIRA ticket/epic
/project-import-spec https://github.com/leafspec/spec      # Import from GitHub URL
/project-import-spec https://confluence.example.com/page   # Import from URL
/project-import-spec ./local-spec.md                       # Import from local file
/project-import-spec JIRA-123 --project coordinatr         # Explicit project
```

## Output Location

```
06 Projects/[project]/specs/
├── README.md                    # Spec index + compliance dashboard
├── spec-[source-name].md        # Imported spec with compliance checklist
└── spec-[source-name]-raw.md    # Original unmodified source (for diffing)
```

Imported specs live in a `specs/` directory at the project root (not in initiative folders). This is a project-wide reference, like `features/`.

## When to Use

- Work projects where a PM defines the requirements
- Implementing an external standard or protocol
- Following a published specification

**When NOT to use:** Personal projects where you define the requirements yourself. Use `/project-features` directly.

## Execution Flow

### 0. Resolve Project

Resolve `[project]` using the [Project Discovery](../project-shared/references/project-discovery.md) procedure:
- If `--project` is specified, use that value
- Otherwise, auto-detect: `basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"`
- Ensure target directory exists: `mkdir -p` the `06 Projects/[project]/` path in the obsidian vault

### 1. Detect Source Type

| Source | Detection | Fetch Method |
|--------|-----------|-------------|
| JIRA ticket | Matches `[A-Z]+-\d+` | `gh` CLI or JIRA API via WebFetch |
| GitHub URL | Contains `github.com` | `git clone` or raw content fetch |
| Generic URL | Starts with `http` | WebFetch |
| Local file | File path | Read |

### 2. Fetch Content

```bash
# JIRA
WebFetch: "https://jira.example.com/rest/api/2/issue/JIRA-123"

# GitHub
Bash: git clone --depth 1 <repo-url> /tmp/spec-import
Read: /tmp/spec-import/README.md

# Generic URL
WebFetch: <url>

# Local file
Read: <path>
```

### 3. Parse into Structured Spec

Extract from the source:
- **Overview/Summary**: What the system should do
- **Requirements**: Organized by section/domain
- **Acceptance criteria**: If present in source
- **Constraints**: Technical or business limitations

### 4. Create Compliance Checklist

Add inline status markers to each requirement:

```markdown
### Section: Authentication

**Requirements:**
- ⏳ User registration with email/password
- ⏳ User login with JWT token
- ⏳ Password reset flow
- ⏳ Email verification
```

**Status markers:**
- ✅ Implemented and working
- 🚧 In progress
- ⏳ Not started

### 5. Write Output Files

**spec-[name].md** (structured version):
```markdown
---
source: https://jira.example.com/browse/JIRA-123
imported: YYYY-MM-DD
version: 1.0
---

# Imported Spec: [Title]

## Source
- **URL:** [source URL]
- **Imported:** YYYY-MM-DD
- **Version:** 1.0

## Overview
[Parsed summary]

## Requirements

### Section: [Name]
- ⏳ Requirement 1
- ⏳ Requirement 2

### Section: [Name]
- ⏳ Requirement 3
```

**spec-[name]-raw.md** (original unmodified):
- Preserved for diffing when re-importing

**README.md** (spec index):
```markdown
# Imported Specs

| Spec | Source | Imported | Requirements | Done |
|------|--------|----------|-------------|------|
| spec-jira-123.md | JIRA-123 | 2026-02-18 | 12 | 0/12 (0%) |
```

### 6. Suggest Next Steps

```
Imported spec with 12 requirements.

Next steps:
1. Run /project-features to decompose into features and user stories
2. Or run /project-issue directly for individual requirements
```

## Re-importing (Sync)

There is no `--sync` flag. Re-running `/project-import-spec` with the same source:
1. Fetches the latest version
2. Diffs against `spec-[name]-raw.md`
3. Shows what changed
4. Updates the structured spec (preserving existing status markers where requirements haven't changed)
5. Updates the raw backup

## Work Project Pipeline

```
PM writes PRD in JIRA
         |
/project-import-spec JIRA-123    <- pull PM's requirements in
         |
/project-features                <- decompose into features + stories
         |
/project-issue -> /project-plan -> /project-implement
```

## Guardrails

- Do NOT create code files or start implementation
- Do NOT modify files outside `06 Projects/`
- Do NOT create initiative folders -- that is `/project-issue`'s job
- Preserve the raw source file for future diffing

## Writing Standard

Assume the primary reader is a **junior developer**. The parsed spec should be clearer and more structured than the source, not just a copy.
