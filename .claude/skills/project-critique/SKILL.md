---
name: project-critique
description: "Challenge idea assumptions with skeptical VC-style evaluation. Use when user requests critique, validation, or 'is this a good idea' assessment."
model: claude-opus-4-5-20251101
allowed-tools: Read, Write, Edit, Glob, Grep
---

# /project-critique

Skeptical, VC-style evaluation of a project idea.

## Usage

```bash
/project-critique                        # Critique current project
/project-critique --project coordinatr   # Specific project
/project-critique --focus market         # Market validation focus
/project-critique --focus technical      # Technical feasibility focus
/project-critique --focus business       # Business model focus
```

## Output Location

```
06 Projects/[project]/critiques/YYYY-MM-DD.md
```

## Prerequisites

**REQUIRED**: `project-brief.md` must exist

```bash
ls "06 Projects/{project}/project-brief.md"
# If missing: "Run /project-brief first"
```

## Execution Flow

### 1. Load Project Context

```bash
Read: "06 Projects/{project}/project-brief.md"
Read: "06 Projects/{project}/competitive-analysis.md" (if exists)
Read: "06 Projects/{project}/README.md"
Glob: resources/research/*.md
```

### 2. Invoke idea-critic Agent

Analysis areas:
1. **Problem Validity** - Is this a real problem? Evidence?
2. **Market Opportunity** - Size? Timing? Competition?
3. **Solution Fit** - Does solution address root cause?
4. **Differentiation** - Why will this win?
5. **Business Model** - Revenue? Unit economics?
6. **Execution Risk** - Team? Timeline? Resources?
7. **Technical Feasibility** - Can this be built?

### 3. Generate Critique Document

```markdown
---
created: YYYY-MM-DD
verdict: strong | promising | needs_work | reconsider
confidence: high | medium | low
---

# Critique: [Project Name]

## Executive Summary
[Honest 1-2 paragraph assessment]

## Critical Concerns

### High Priority
1. **[Concern]**: [Why concerning]
   - Evidence needed: [What would resolve]

### Medium Priority
1. **[Concern]**: [Why matters]
   - Suggestion: [How to address]

### Strengths
1. **[Strength]**: [Why valuable]

## Analysis by Area
[Problem Validity, Market, Solution Fit, etc.]

## Hard Questions to Answer
1. [Tough question]
2. [Another challenge]

## Recommendations

### If You Proceed
- [De-risk action]
- [Research needed]

### If You Pivot
- [Alternative direction]
```

## Focus Modes

### `--focus market`
- Market size/dynamics
- Competition landscape
- Customer validation
- Timing and trends

### `--focus technical`
- Technical feasibility
- Architecture complexity
- Skill requirements

### `--focus business`
- Revenue model
- Pricing strategy
- Unit economics
- Go-to-market

## When to Use

- After completing project brief
- Before significant time investment
- When seeking honest feedback
- Before presenting to others

**Not for**: Ideas still forming (use /project-brief), when needing encouragement

## Workflow

```
/project-brief → /project-critique → /research-deep → /project-spec
```
