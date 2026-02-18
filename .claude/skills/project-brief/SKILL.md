---
name: project-brief
description: "Create or update project brief through interactive discovery. Use when starting any new idea, revisiting an idea after time away, or when vision feels unclear."
model: claude-opus-4-5-20251101
allowed-tools: Read, Write, Edit, Glob, Grep
---

# /project-brief

Create or improve project brief through conversational discovery.

## Usage

```bash
/project-brief                           # Interactive discovery for current or new project
/project-brief --project coordinatr      # Brief for specific project
/project-brief --review                  # Analyze existing brief (no edits)
/project-brief --force                   # Start from scratch
```

## Output Location

```
06 Projects/[project]/project-brief.md
```

## Execution Flow

### 1. Determine Mode

- **No brief exists** -> Full Discovery Mode
- **Brief exists, no flags** -> Gap-driven update mode
- **`--review` flag** -> Analysis mode (no edits)
- **`--force` flag** -> Fresh start (after confirmation)

### 2. Invoke brief-strategist Agent

For discovery, conduct 6-phase interactive conversation (one question at a time).

**Skip phases where the answer is clearly inferable from the user's initial description.** Only ask about gaps. If the user provides a rich description, you may skip directly to the unclear areas.

**Question format:** Number all questions and provide letter options where applicable:
```
1. Who is the primary user of this tool?
   A. Developers building side projects
   B. Teams at startups
   C. Enterprise organizations
   D. Open source community
```
This lets the user respond quickly with "1A" instead of writing paragraphs.

### 3. Six-Phase Discovery Topics

1. **Problem Discovery** - What problem? Who experiences it? Current solutions?
2. **Solution Exploration** - How does your solution work? Core value proposition?
3. **Audience Definition** - Who exactly is the target user? Characteristics?
4. **Feature Prioritization** - MVP features? What can wait?
5. **Differentiation** - What makes this different from alternatives?
6. **Success Metrics** - How will you measure success?

## Brief Structure

```markdown
---
status: draft | active | paused | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Project Brief: [Project Name]

## Executive Summary
[One-paragraph overview]

## Problem Statement
[Detailed problem description and impact]

## Solution Approach
[How the solution addresses the problem]

## Target Audience
[Specific user personas and characteristics]

## Success Criteria
[Measurable outcomes and validation metrics]

## Scope and Constraints
[Project boundaries and limitations]

## Non-Goals
[What this project explicitly will NOT do — boundaries to prevent scope creep]

## Project Phases
[High-level implementation roadmap]

## Risk Assessment
[Key risks and mitigation strategies]
```

## Writing Standard

Assume the primary reader is a **junior developer**. Requirements should be explicit, unambiguous, and avoid jargon where possible. If a junior dev couldn't understand the brief and start working from it, the brief isn't clear enough.

## Guardrails

- Do NOT create code files or start implementation
- Do NOT modify files outside `06 Projects/`
- Do NOT skip to spec or plan creation — finish the brief first

## When to Use

- Starting a new idea
- Revisiting an idea after time away
- Before creating specifications
- When vision feels unclear

**Not needed for**: Quick notes (use notes/), technical research (/research), feature decomposition (/project-features)

## Next Steps After Brief

```
/project-brief → /project-features → /project-issue → /project-plan → /project-implement
```
