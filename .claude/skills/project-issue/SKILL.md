---
name: project-issue
description: "Create standalone work items (TASK, BUG, or SPIKE) with AI-assisted type detection. Use when you need to track work that needs doing, exploration needed, or something broken."
model: claude-sonnet-4-20250514
allowed-tools: Read, Write, Edit, Glob, Grep
---

# /project-issue

Create standalone work items through natural conversation with AI-assisted type detection.

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

```
06 Projects/[project]/issues/
└── 001-implement-auth/
    ├── TASK.md       # or SPIKE.md or BUG.md
    ├── PLAN.md       # Created by /project-plan
    └── WORKLOG.md    # Progress tracking
```

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
> "This sounds like a **TASK**. Create as issue 002? (yes / spike / bug)"

### 3. Determine Next Issue Number

```bash
ls "06 Projects"/[project]/issues/ | grep -E '^[0-9]{3}-' | sort -n | tail -1
```

### 4. Ask About Spec Section

Check if project has a spec:
```bash
Glob: "06 Projects"/[project]/docs/specs/*.md
```

If spec exists:
> "Which spec section does this implement?"
> - docs/specs/required-features.md#authentication
> - docs/specs/required-features.md#documents
> - none (standalone task)

### 5. Create Issue Files

**TASK.md:**
```markdown
---
status: open
created: YYYY-MM-DD
implements: docs/specs/required-features.md#authentication  # or empty if standalone
depends_on: []
---

# TASK-###: [Title]

## Description

[What needs to be done and why]

## Implements

**Spec Section:** [docs/specs/required-features.md#authentication](docs/specs/required-features.md#authentication)

**Requirements from spec:**
- [Requirement 1 from spec]
- [Requirement 2 from spec]

## Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2

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

# SPIKE-###: [Title]

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

# BUG-###: [Title]

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

### 7. Next Steps

- For TASK/BUG: Suggest `/project-plan ###` to create implementation phases
- For SPIKE: Suggest `/project-plan ###` to create exploration plan

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

## Spec Integration

### Task Scoping

**One TASK = One requirement line item**

A TASK should be atomic and shippable:
- Implements exactly one spec requirement
- Can be pushed to main independently
- Updates one `⏳` → `✅` marker when complete

```markdown
# Spec line items (each becomes a TASK)
- ⏳ User registration with email/password    ← TASK-002
- ⏳ User login with JWT token                ← TASK-003
- ⏳ Password reset flow                      ← TASK-004
```

**Don't create:** "TASK: Implement Authentication" (too broad)
**Do create:** "TASK: User registration endpoint" (one requirement)

### implements: Field

```yaml
# Points to the specific requirement
implements: docs/specs/required-features.md#user-registration-with-email-password
```

This creates a direct link between work items and the exact requirement they fulfill.

## Workflow

```
/project-spec → /project-issue → /project-plan → (work) → quality gate → /project-commit → /project-complete
                  ↓
            implements: spec section
```
