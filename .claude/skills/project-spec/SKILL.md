---
name: project-spec
description: "Manage protocol/standard specifications that define what a system must do. Use to create, import, or update the contract that TASKs implement against."
model: claude-opus-4-5-20251101
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch
---

# /project-spec

Manage protocol-level specifications - the contract defining *what* a system must do.

## What is a Spec?

A **spec** (specification) defines requirements at the protocol/standard level:
- **External specs**: Standards you implement (LEAF spec, OAuth, OpenAPI)
- **Self-authored specs**: Your own protocol defining what your system does

Specs are NOT feature breakdowns or epics. They are the **source of truth** for requirements.

## Usage

```bash
/project-spec                           # Show current project's spec status
/project-spec --import <url>            # Import external spec (GitHub, raw URL)
/project-spec --init                    # Create new protocol spec for project
/project-spec --sync                    # Sync imported spec with upstream
/project-spec --section <name>          # Show specific section of spec
```

## File Structure

All documentation lives in `06 Projects/`:

```
06 Projects/[project]/
├── project-brief.md            # Strategy
├── docs/
│   ├── specs/                  # The protocol spec (source of truth)
│   │   ├── README.md           # Spec overview and compliance status
│   │   ├── api-specification.md # API contract
│   │   ├── data-models.md      # Data structures
│   │   ├── required-features.md # Feature requirements
│   │   └── ...
│   └── adrs/                   # Architecture decisions
└── issues/
    └── 001-auth/
        └── TASK.md             # implements: docs/specs/required-features.md#authentication
```

**Why specs live in 06 Projects/**: All project documentation is centralized in the vault. Code repos (at their actual paths) are READ-ONLY from skills. The `code:` field in the Projects Index points to the actual repo path for reference.

## Execution Flow

### 1. Determine Context

```bash
Read: "06 Projects/[project]/project-brief.md"    # Strategy context
Glob: "06 Projects/[project]/docs/specs/*.md"     # Existing specs
```

Questions:
- Does this project implement an external spec?
- Or does it need its own protocol spec?

### 2a. Import External Spec

For projects implementing a standard (like leaf-nextjs-convex → LEAF spec):

```bash
/project-spec --import https://github.com/leafspec/spec
```

**Process:**
1. Clone/fetch spec files
2. Copy to `06 Projects/[project]/docs/specs/`
3. Create `docs/specs/README.md` with:
   - Source URL and version
   - Last synced date
   - Compliance checklist
4. Suggest initial TASKs based on spec sections

**Sync upstream changes:**
```bash
/project-spec --sync
```

### 2b. Create Protocol Spec

For projects that need their own spec (like coordinatr):

```bash
/project-spec --init
```

**Conversational creation:**
1. What does this system do? (elevator pitch)
2. Who are the actors/users?
3. What are the core operations?
4. What are the API boundaries?
5. What are the data models?

**Output structure:**
```markdown
# [Project] Specification

## Overview
[What this system does and why]

## Actors
[Who/what interacts with the system]

## Core Operations
[The fundamental things the system must do]

## API Specification
[Endpoints, inputs, outputs, errors]

## Data Models
[Entity definitions, relationships, constraints]

## Required Features
[Feature requirements organized by domain]

## Test Criteria
[How to verify compliance]
```

### 3. Spec Status Dashboard

```bash
/project-spec  # No arguments
```

Shows:
- Spec source (external URL or self-authored)
- Last updated/synced
- Sections and their implementation status
- Linked TASKs per section

## Spec vs Old "Feature Specs"

| Old Model (Wrong) | New Model (Correct) |
|-------------------|---------------------|
| SPEC-001, SPEC-002... | Single protocol spec |
| Feature breakdown | Requirements contract |
| Internal planning docs | Source of truth |
| Created per feature | Created once, evolved |
| TASKs link to SPEC-### | TASKs implement spec sections |

## Integration with /project-issue

When creating a TASK, link to the spec section it implements:

```yaml
---
implements: docs/specs/required-features.md#authentication
---
```

The `/project-issue` command will prompt:
> "Which spec section does this implement? (or 'none' for standalone)"

## Compliance Tracking

Status is tracked **inline** within spec documents at the requirement level:

```markdown
### §1 Authentication

**Requirements:**
- ✅ User registration with email/password
- ✅ User login with JWT token
- ⏳ Password reset flow
- ⏳ Email verification

**API Endpoints:**
- ✅ `POST /api/auth/register`
- ✅ `POST /api/auth/login`
- ⏳ `POST /api/auth/reset-password`
```

**Status markers:**
- ✅ Implemented and working
- 🚧 In progress
- ⏳ Not started

This allows granular visibility into what's done without referencing private TASKs.

The `/project-complete` command updates these markers when work is finished.

## Self-Authored Spec Guidelines

When creating your own protocol spec:

1. **Be specific** - Vague specs lead to vague implementations
2. **Define boundaries** - What's in scope vs out of scope
3. **Include test criteria** - How do you verify compliance?
4. **Version it** - Specs evolve; track changes
5. **Keep it stable** - Changes should be deliberate

## Spec Versioning (differs from ADRs)

**ADRs are immutable** - changes create a new superseding document.

**Specs are edited in place** - they're living contracts that evolve:

1. **Frontmatter version** - Use semantic versioning (`version: 1.0.0`)
2. **Git history** - Preserves full evolution
3. **Git tags** - Mark release points (`git tag spec-v1.0.0`)
4. **CHANGELOG** - Note significant spec changes

**Version bumps:**
- **Patch** (1.0.1): Typos, clarifications, no behavior change
- **Minor** (1.1.0): New optional features, backwards compatible
- **Major** (2.0.0): Breaking changes, removed requirements

This keeps specs simple while git provides the audit trail.

## Workflow

```
/project-spec --init or --import    # Define what to build
        ↓
/project-issue                      # Create work items that implement spec sections
        ↓
/project-plan                       # Break down implementation
        ↓
/project-implement                  # Build against the spec
        ↓
/project-validate-spec ──────┐
/project-validate-quality ───┤     # Quality gate (parallel)
/project-validate-security-audit ┘
        ↓
/project-commit                     # Save progress
        ↓
/project-complete                   # Verify spec compliance
```

## Related Commands

- `/project-issue` - Create TASKs that implement spec sections
- `/project-plan` - Break down implementation of a TASK
- `/project-validate-spec` - Check implementation against spec
- `/project-status` - See spec compliance overview
