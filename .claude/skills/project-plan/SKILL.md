---
name: project-plan
description: "Create PLAN.md file with phase-based breakdown for issues. Use after creating an issue with /project-issue to break down work into phases."
model: claude-opus-4-5-20251101
allowed-tools: Read, Write, Edit, Glob, Grep
---

# /project-plan

Create PLAN.md with phase-based breakdown for tasks, spikes, and bugs.

## Usage

```bash
/project-plan 2026-02-18-implement-auth              # Create plan for initiative (auto-detect project)
/project-plan 2026-02-18-implement-auth --project coordinatr  # Explicit project
/project-plan 2026-02-18-compare-graphql-rest         # Plan for a spike
/project-plan 2026-02-18-implement-auth --second-opinion  # Peer review from Gemini CLI
```

## Issue Type Detection

| Issue Type | Plan Structure | File Created |
|------------|---------------|--------------|
| Task (TASK.md) | Sequential phases with checkpoints | PLAN.md |
| Bug (BUG.md) | Investigation → Fix phases | PLAN.md |
| Spike (SPIKE.md) | Exploration phases per approach | PLAN-1.md, PLAN-2.md, ... |

## Execution Flow

### 1. Resolve Initiative

Find the initiative folder from the argument:

```bash
# If full folder name given:
Read: "06 Projects/[project]/2026-02-18-implement-auth/TASK.md"

# If partial name given, search:
Glob: "06 Projects/[project]/*-implement-auth*/TASK.md"
Glob: "06 Projects/[project]/*-implement-auth*/BUG.md"
Glob: "06 Projects/[project]/*-implement-auth*/SPIKE.md"

# If no initiative specified, list available:
Glob: "06 Projects/[project]/[0-9][0-9][0-9][0-9]-*"
```

Present available initiatives if multiple match or none specified:
> "Which initiative?"
> 1. 2026-02-17-implement-auth (TASK - open)
> 2. 2026-02-18-api-redesign (SPIKE - open)

### For Task/Bug

2. **Load Context**:
   ```bash
   Read: "06 Projects/[project]/YYYY-MM-DD-name/TASK.md" (or BUG.md)
   Glob: "06 Projects/[project]/YYYY-MM-DD-name/spec-*.md"
   Glob: "06 Projects/[project]/*/ADR-*.md"
   Glob: resources/research/*.md
   ```

   If the issue has a `feature:` field, load the parent feature card:
   ```bash
   Read: "06 Projects/[project]/features/F-###-name.md"
   ```

   Use the feature card for:
   - Understanding the broader context of this task within the feature
   - Feature-level dependencies inform phase ordering
   - User stories provide acceptance-test inspiration

   If the issue has an `implements:` field, locate and load that spec or feature card:
   ```bash
   # Check feature cards first, then specs
   Glob: "06 Projects/[project]/features/F-*.md"
   Glob: "06 Projects/[project]/specs/spec-*.md"
   Glob: "06 Projects/[project]/*/spec-*.md"
   ```

3. **Cross-Project Pattern Search**:
   ```bash
   # Search other projects' code repos for similar implementations
   # (Resolve paths from Projects Index in CLAUDE.md)
   Grep: code repos for relevant patterns
   ```

   Include relevant references in plan:
   ```markdown
   ## Related Implementations

   Found similar patterns in other projects:
   - `/Users/duncanleung/Develop/yourbench/src/auth/clerk.ts` - Clerk auth setup
   - `/Users/duncanleung/Develop/coordinatr/src/lib/session.ts` - Session handling
   ```

4. **Library Documentation** (automatic for integrations):

   **MANDATORY when task involves:**
   - Installing/configuring external libraries or SDKs
   - Framework integrations (auth providers, databases, APIs)
   - Third-party services (Clerk, Stripe, AWS services, etc.)

   **Process:**
   ```bash
   # 1. Resolve library ID
   mcp__context7__resolve-library-id: {libraryName}

   # 2. Fetch current documentation
   mcp__context7__query-docs: {context7CompatibleLibraryID}

   # 3. Search for recent patterns/best practices
   WebSearch: "{library} {framework} integration 2026"
   ```

   **Document findings:**
   ```markdown
   ## Library Documentation Validation

   **{Library Name}** (validated YYYY-MM-DD):
   - Current version: X.Y.Z
   - Key integration patterns: [summary]
   - Recommended approach: [based on current docs]
   ```

5. **Confirm Phase Structure (Two-Phase Confirmation)**:

   Before generating sub-tasks, present high-level phases to user for approval:
   > "Here are the planned phases:
   > 0. Setup — create feature branch, verify dependencies
   > 1. [Phase name] — [one-line description]
   > 2. [Phase name] — [one-line description]
   > 3. [Phase name] — [one-line description]
   >
   > Confirm? (go / adjust)"

   Wait for user confirmation before generating sub-task breakdowns. This prevents wasted planning effort on wrong approaches.

6. **Generate Sub-Tasks**:
   - Break each confirmed phase into actionable sub-tasks
   - Each phase has clear deliverables
   - Include checkpoints between phases
   - Add "Done When" criteria

7. **Write PLAN.md**:
   - Present phases, estimated effort, dependencies
   - Include "Library Documentation Validation" section (if applicable)
   - Include "Second Opinion Analysis" section (if requested)

8. **Commit Suggestion**:
   - Ask: "Commit PLAN.md to ideas repo? (yes/no)"

### For Spike (Exploration)

1. **Load Context**: Read SPIKE.md (questions, success criteria, time box)

2. **Gather Approaches**:
   - Ask: "How many approaches to explore?" → N
   - For each: "Describe approach N?"

3. **Generate**: Create PLAN-N.md for each approach

4. **Commit Suggestion**: Ask to commit all plan files

## Task Plan Example

```markdown
# Implementation Plan: Implement Auth

## Overview
Research authentication patterns for Coordinatr's multi-tenant architecture.

## Relevant Files

### New Files
- `src/auth/clerk.ts` - Clerk authentication setup and configuration
- `src/auth/clerk.test.ts` - Tests for Clerk auth integration
- `src/middleware.ts` - Auth middleware for protected routes
- `src/middleware.test.ts` - Tests for auth middleware

### Modified Files
- `src/app/layout.tsx` - Add ClerkProvider wrapper
- `package.json` - Add @clerk/nextjs dependency

## Phase 0: Setup

### 0.1 - Environment
- [ ] Create feature branch (`feature/implement-auth`)
- [ ] Verify dependencies installed
- [ ] [CHECKPOINT] On correct branch, clean working tree

## Phase 1: Survey Existing Solutions

### 1.1 - Research Auth Libraries
- [ ] Review Better Auth documentation
- [ ] Compare with Auth.js and Lucia
- [ ] Document trade-offs

### 1.2 - Multi-Tenant Patterns
- [ ] Research team-based auth patterns
- [ ] Review how Slack, Linear handle it
- [ ] [CHECKPOINT] Summary document complete

## Phase 2: Architecture Proposal

### 2.1 - Draft Architecture
- [ ] Create architecture diagram
- [ ] Document token strategy
- [ ] Define permission model

### 2.2 - Review
- [ ] Self-critique against requirements
- [ ] [CHECKPOINT] Architecture doc complete

## Done When
- [ ] Auth library recommendation documented
- [ ] Architecture proposal in initiative folder
- [ ] Trade-offs and risks identified
```

## Writing Standard

Assume the primary reader is a **junior developer**. Phase descriptions, sub-tasks, and checkpoints should be explicit enough that someone unfamiliar with the codebase could follow the plan. If a junior dev couldn't execute from the plan alone, it needs more detail.

## Guardrails

- Do NOT create code files or start implementation
- Do NOT modify files outside `06 Projects/`
- Do NOT skip phase confirmation — get user approval on high-level phases before detailing sub-tasks

## Second Opinion Feature

**What**: Optional peer review from Gemini CLI before finalizing plans.

**Usage**: Pass `--second-opinion` flag to trigger Gemini review with Context7 validation.

**Process**:
1. Send plan to Gemini CLI for review
2. Validate each recommendation against Context7 docs
3. Claude makes final decision on each:
   - ✅ **ACCEPT**: Recommendation validated AND improves plan
   - ⚠️ **MODIFY**: Good idea but needs adjustment
   - ❌ **REJECT**: Invalid or not applicable
4. Document all decisions in "Second Opinion Analysis" section

**Requirements**:
- Gemini CLI installed and functional
- `--second-opinion` flag explicitly passed

**Graceful degradation**: If Gemini unavailable, proceeds with Claude-only plan.

## Workflow

```
/project-features → /project-issue → /project-plan {initiative} → (work phases)
                                          ↓                             ↓
                                Load feature card             quality gate (parallel)
                                from feature: field                 ↓
                                                  /project-commit → /project-complete {initiative}
```

**Creates:**
- Task/Bug: `06 Projects/{project}/YYYY-MM-DD-name/PLAN.md`
- Spike: `06 Projects/{project}/YYYY-MM-DD-name/PLAN-1.md`, `PLAN-2.md`, etc.
