---
name: project-issue
description: "Create standalone work items (TASK, BUG, or SPIKE) with AI-assisted type detection. Use when you need to track work that needs doing, exploration needed, or something broken."
model: claude-sonnet-4-20250514
allowed-tools: Read, Write, Edit, Glob, Grep
---

# /project-issue

Create standalone work items through natural conversation with AI-assisted type detection. Each issue creates a dated **initiative folder** at the project root.

## Usage

```bash
/project-issue                                    # Start conversation
/project-issue "Implement authentication"         # AI detects: TASK
/project-issue "Compare GraphQL vs REST"          # AI detects: SPIKE
/project-issue "Broken link in project-brief"     # AI detects: BUG
/project-issue --project coordinatr               # Create for specific project
```

## Issue Types

| Type | Purpose | Detection Keywords |
|------|---------|-------------------|
| **TASK** | Work that needs doing | implement, create, add, build, write, set up |
| **SPIKE** | Time-boxed exploration | compare, vs, should we, evaluate, explore, research, feasibility |
| **BUG** | Something broken/wrong | fix, broken, incorrect, outdated, wrong, error |

## File Structure

Each issue creates an **initiative folder** at the project root (no `issues/` intermediate directory):

```
06 Projects/[project]/
├── README.md                              # project index (stays at root)
├── project-brief.md                       # project vision (stays at root)
├── YYYY-MM-DD-initiative-name/            # ← initiative folder
│   ├── TASK.md       # or SPIKE.md or BUG.md
│   ├── PLAN.md       # Created later by /project-plan
│   └── WORKLOG.md    # Progress tracking
```

**Naming convention:** `YYYY-MM-DD-slug` where slug is a kebab-case short name derived from the issue title (e.g., `2026-02-18-implement-auth`).

## Execution Flow

### 1. Gather Context

If no description: "What needs to be done?"
If no project: "Which project?"

**Question format:** When clarifying, provide numbered options with letter choices:
```
1. What area does this affect?
   A. Authentication / user accounts
   B. UI / frontend components
   C. API / backend services
   D. Infrastructure / deployment
```
Only ask questions when the answer isn't reasonably inferable from the user's description.

### 2. Detect Type

Analyze description keywords, present detection:
> "This sounds like a **TASK**. Create initiative `2026-02-18-implement-auth`? (yes / spike / bug)"

### 3. Generate Initiative Folder Name

Build the folder name from today's date and a slug derived from the title:

```
YYYY-MM-DD-[kebab-case-slug]
```

Rules:
- Date is today's date
- Slug is 2-5 words, kebab-case, derived from the issue title
- Keep it short but descriptive (e.g., `implement-auth`, `api-redesign`, `broken-login-redirect`)
- No sequential numbering — the date provides natural ordering

Check for collisions:
```bash
Glob: "06 Projects"/[project]/YYYY-MM-DD-*
```

If a same-date folder with the same slug exists, append a disambiguator (e.g., `-v2`).

### 3.5. Ask About Feature

Check if project has features:
```bash
Glob: "06 Projects"/[project]/features/F-*.md
```

If features exist, load the feature map and present:
> "Which feature does this belong to?"
> 1. F-001: User Authentication (must-have, MVP)
> 2. F-002: Document Management (must-have, MVP)
> 3. F-003: Sharing (should-have, Phase 2)
> 4. None (standalone task)

Load the selected feature card as context for writing acceptance criteria.
If the feature card has user stories, incorporate relevant Given-When-Then AC into the task's acceptance criteria.

### 4. Ask About Spec Section

Check if project has specs (imported external specs or legacy specs):
```bash
Glob: "06 Projects"/[project]/specs/spec-*.md      # Imported specs
Glob: "06 Projects"/[project]/*/spec-*.md           # Legacy initiative-scoped specs
```

If the task has a feature, the `implements:` field should point to the feature card section:
```yaml
implements: features/F-001-auth.md#account-registration
```

If spec exists and no feature selected:
> "Which spec section does this implement?"
> - spec-required-features.md#authentication
> - spec-required-features.md#documents
> - none (standalone task)

### 5. Create Initiative Folder and Issue File

Create the initiative folder and write the issue file inside it.

**TASK.md:**
```markdown
---
status: open
created: YYYY-MM-DD
feature: F-001                                              # or empty if standalone
implements: features/F-001-auth.md#account-registration     # or spec section, or empty
depends_on: []
---

# TASK: [Title]

## Description

[What needs to be done and why]

## Implements

**Feature:** [F-001: User Authentication](../features/F-001-auth.md)
**Story:** Account Registration

**Acceptance Criteria (from feature):**
- Given a valid email and password, When I submit registration, Then my account is created
- Given an email already in use, When I submit registration, Then I see "Email already registered"

## Acceptance Criteria

- [ ] Criterion 1 (from feature story AC above)
- [ ] Criterion 2 (additional task-specific criteria)

## Non-Goals

- [What this task explicitly will NOT do]
- [Boundaries to prevent scope creep]

## Context

[Background, relevant decisions, technical notes]
```

**SPIKE.md:**
```markdown
---
status: open
created: YYYY-MM-DD
timebox: X hours
---

# SPIKE: [Title]

## Questions

- Question 1?
- Question 2?

## Approaches to Explore

1. Approach A
2. Approach B

## Findings

(filled after exploration)

## Recommendation

(filled after exploration)
```

**BUG.md:**
```markdown
---
status: open
created: YYYY-MM-DD
---

# BUG: [Title]

## What's Broken

[Description of the problem]

## Location

[File paths, URLs, etc.]

## Expected Behavior

[What should happen]

## Steps to Reproduce

1. Step 1
2. Step 2
```

### 6. Mark Spec Section In Progress (if implements spec)

If the issue implements a spec section, update the inline status markers in the spec:

```markdown
# Before
- ⏳ User registration with email/password

# After
- 🚧 User registration with email/password
```

The `/project-complete` command will mark these ✅ when done.

### 7. Update Project README.md

Add the new initiative to the **Initiatives** table in the project README:

```markdown
## Initiatives

| Initiative | Date | Status |
|------------|------|--------|
| [Auth Implementation](2026-02-17-auth/) | 2026-02-17 | in-progress |
| [API Redesign](2026-02-18-api-redesign/) | 2026-02-18 | open (SPIKE) |
```

**Status format in table:**
- TASK/BUG: `open`
- SPIKE: `open (SPIKE)`

If the README doesn't have an Initiatives table yet, create one.

### 8. Next Steps

- For TASK/BUG: Suggest `/project-plan 2026-MM-DD-initiative-name` to create implementation phases
- For SPIKE: Suggest `/project-plan 2026-MM-DD-initiative-name` to create exploration plan

## Writing Standard

Assume the primary reader is a **junior developer**. Descriptions, acceptance criteria, and context should be explicit, unambiguous, and avoid jargon. If a junior dev couldn't implement from the issue alone, the issue isn't clear enough.

## Guardrails

- Do NOT create code files or start implementation
- Do NOT modify files outside `06 Projects/`
- Do NOT skip to planning or implementation — finish the issue first

## Status Values

| Status | Meaning |
|--------|---------|
| `open` | Not started |
| `in_progress` | Being worked on |
| `blocked` | Waiting on something |
| `complete` | Done |

## Feature and Spec Integration

### Task Scoping

**One TASK = One user story (or part of a story if large)**

A TASK should be atomic and shippable:
- Implements one user story from a feature card (or one requirement from a spec)
- Can be pushed to main independently
- Touches 2-3 files maximum (for AI agent effectiveness)

**Don't create:** "TASK: Implement Authentication" (too broad — that's a feature)
**Do create:** "TASK: User registration endpoint" (one story)

### feature: Field

```yaml
# Points to the parent feature
feature: F-001
```

This groups related tasks under a feature for progress tracking.

### implements: Field

```yaml
# Points to the specific story or spec requirement
implements: features/F-001-auth.md#account-registration
```

This creates a direct link between work items and the exact story/requirement they fulfill.

## Workflow

```
/project-features → /project-issue → /project-plan → (work) → quality gate → /project-commit → /project-complete
                       ↓
                 feature: F-001
                 implements: feature story
```
