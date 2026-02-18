---
name: project-advise
description: "Interactive conversational guidance - user implements with step-by-step advice. Use when you want hands-on implementation with expert guidance while maintaining control."
model: claude-sonnet-4-20250514
allowed-tools: Read, Glob, Grep
---

# /project-advise

Interactive, conversational guidance as you implement a task yourself.

## Usage

```bash
/project-advise 001              # Guide through issue 001
/project-advise yourbench 001    # Explicit project
/project-advise 001 --phase 2.1  # Start at specific phase
```

## Comparison: /project-implement vs /project-advise vs /project-teach

| Aspect | /project-implement | /project-advise | /project-teach |
|--------|------------|---------|--------|
| Who writes code | AI | You | You |
| Speed | Fast | Medium | Slower |
| Depth | Task completion | Task guidance | Conceptual learning |
| Questions | Few | As needed | Frequent, Socratic |

## What to Expect

### Conversational Flow

```
AI: "First step: Initialize Next.js. The directory has some docs,
     so we'll move those temporarily..."

User: "What does the --no-git flag do?"

AI: "Good question! It tells create-next-app to skip running
     'git init' since you already have a repo..."

User: "Done"

AI: "Let me check... Looks good. Next, we need to clean up
     the boilerplate..."
```

### Guidance Style

- **Explains WHAT to do** - Clear instructions per step
- **Answers questions** - Responds to clarifications
- **Checks your work** - Reviews when you say "done"
- **Adapts pace** - Moves forward when ready
- **Practical focus** - Task-focused, not teaching concepts

## Execution Flow

### 1. Load Context

```bash
Read: "06 Projects"/[project]/issues/###-*/TASK.md
Read: "06 Projects"/[project]/issues/###-*/PLAN.md
Read: "06 Projects"/[project]/specs/SPEC-###.md (if linked)
Glob: "06 Projects"/[project]/docs/adrs/ADR-*.md
```

### 2. Conversational Guidance

1. Explain the current step simply
2. Wait for user questions or confirmation
3. Answer questions as they come up
4. Check work when user says "done"
5. Move to next step when ready

### 3. Research as Needed

- Use Context7 for library docs
- WebSearch for best practices
- Reference codebase patterns

### 4. Check Work

When user says "done":
- Read the modified file
- Check for common mistakes
- Verify it matches requirements
- Suggest fixes if needed

### 5. Update WORKLOG

At checkpoints:
```markdown
## YYYY-MM-DD HH:MM - ADVICE: Phase 1 Complete

Guided user through Next.js initialization.
Completed: ...
Next: Phase 2
```

## When to Use

- Know the stack, need guidance on THIS task
- Want control but avoid mistakes
- Prefer hands-on implementation
- Task has complexity worth discussing

**Use /project-implement instead**: Just want it done quickly, boilerplate code
**Use /project-teach instead**: Learning the technology, need conceptual understanding

## Workflow

```
/project-issue → /project-plan → /project-advise → [you implement] → /project-worklog
        ↓
/project-spec-validate ──────┐
/project-validate-quality ───┤  (quality gate, parallel)
/project-validate-security-audit ┘
        ↓
/project-commit → /project-complete
```
