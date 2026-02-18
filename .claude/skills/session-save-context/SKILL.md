---
name: session-save-context
description: "Save AI session context handoff to the obsidian vault's 06 Projects/{project}/ initiative folder for seamless continuation of work. Triggers on 'save context', 'handoff', 'session context'."
model: claude-haiku-4-5-20251001
allowed-tools: Read, Write, Glob, Grep, Bash(date:*, mkdir:*, wc:*, grep:*)
argument-hint: [topic]
---

# /session-save-context

Save a comprehensive context handoff file to the obsidian vault inside the relevant initiative folder at `06 Projects/{project}/YYYY-MM-DD-name/context.md`, optimized for a new AI agent to continue work on a specific topic. The project folder is derived from the current working directory name. Files are always saved to the obsidian vault regardless of which repo this skill is invoked from.

## Usage

```bash
/session-save-context "authentication refactor"
/session-save-context "learning system improvements"
/session-save-context "weekly review process"
```

## Current Environment

- Working Directory: !`pwd`
- Git Branch: !`git branch --show-current 2>/dev/null || echo "not a git repo"`
- Git Status: !`git status --porcelain 2>/dev/null | head -10`
- Recent Commits: !`git log --oneline -5 2>/dev/null || echo "no git history"`
- Project Type: !`ls -la 2>/dev/null | grep -E "(package\.json|pyproject\.toml|Cargo\.toml|go\.mod)" | head -3`

## Topic

**$ARGUMENTS**

## Instructions

You will create a topic-focused context file that captures everything a new AI agent needs to understand and continue work on: **$ARGUMENTS**

### Step 1: Discover Related Context

Search the codebase to find ALL files and context related to "$ARGUMENTS":

1. **Search for related files** using Glob and Grep:
   - Source files mentioning the topic
   - Documentation files related to the topic
   - Test files covering the topic
   - Configuration files affecting the topic
   - Skill files related to the topic
   - Ideas and project planning files

2. **Read key files** that are central to understanding the topic

3. **Check for existing context**:
   - Previous context files in initiative folders: `06 Projects/{project}/*/context.md`
   - Documentation in `.claude/` or `SHARED/`

### Step 2: Gather User Context

Ask the user these focused questions about "$ARGUMENTS":

1. **"What is the current state of work on this topic?"** (completed, in-progress, research phase, blocked)

2. **"What key decisions have been made and what was the reasoning?"**

3. **"What approaches were tried that didn't work? What should be avoided?"**

4. **"What should the next AI session focus on first?"**

### Step 3: Generate Context File

Create the handoff file in the relevant initiative folder. If there's an active initiative matching the topic, save there. Otherwise create a new initiative-scoped context:

```bash
OBSIDIAN_VAULT="/Users/duncanleung/Develop/obsidian-local-life-manager"
TIMESTAMP=$(date "+%Y-%m-%d")
TOPIC_SLUG=$(echo "$ARGUMENTS" | tr ' ' '-' | tr -cd '[:alnum:]-' | tr '[:upper:]' '[:lower:]' | head -c 50)
PROJECT_NAME=$(basename "$(pwd)")

# Try to find matching initiative folder
INITIATIVE_DIR=$(ls -d "${OBSIDIAN_VAULT}/06 Projects/${PROJECT_NAME}/"*-${TOPIC_SLUG}* 2>/dev/null | head -1)

# If no match, create new initiative folder
if [ -z "$INITIATIVE_DIR" ]; then
  INITIATIVE_DIR="${OBSIDIAN_VAULT}/06 Projects/${PROJECT_NAME}/${TIMESTAMP}-${TOPIC_SLUG}"
fi

CONTEXT_FILE="${INITIATIVE_DIR}/context.md"
mkdir -p "$INITIATIVE_DIR"
```

Use this structure for the context file:

```markdown
---
tags:
  - ai-context
  - handoff
created: [YYYY-MM-DD]
topic: $ARGUMENTS
status: [completed/in-progress/research/blocked]
---

# Context: [Topic Title]

**Created:** [timestamp]
**Topic:** $ARGUMENTS
**Purpose:** Comprehensive context for AI agent handoff

---

## QUICK START

**Current State:** [status - completed/in-progress/research/blocked]

**Immediate Priority:** [what the next session should do first]

**Key Constraint:** [most important thing to not break/forget]

---

## TOPIC OVERVIEW

**What This Is:** [clear explanation of the topic scope]

**Why It Matters:** [business/technical importance]

**Current Progress:** [what's done vs what remains]

---

## KEY DECISIONS

| Decision | Reasoning | Date |
|----------|-----------|------|
| [decision 1] | [why] | [when] |
| [decision 2] | [why] | [when] |

---

## TECHNICAL CONTEXT

### Related Files

| File | Purpose | Notes |
|------|---------|-------|
| [file path] | [what it does] | [important details] |

### Architecture/Design

[Relevant architecture notes, diagrams descriptions, or system design]

### Dependencies

[Key dependencies, integrations, or external systems involved]

---

## RESEARCH & FINDINGS

### What Works

- [Verified approach 1]
- [Verified approach 2]

### What Doesn't Work

- [Failed approach 1] - Reason: [why it failed]
- [Failed approach 2] - Reason: [why it failed]

### Gotchas & Warnings

- [Critical gotcha 1]
- [Edge case to watch for]

---

## OPEN QUESTIONS

- [ ] [Unresolved question 1]
- [ ] [Unresolved question 2]

---

## NEXT STEPS

1. **[Primary task]** - [brief description]
2. **[Secondary task]** - [brief description]
3. **[Validation needed]** - [what to test/verify]

---

## REFERENCE LINKS

- [Link to relevant docs, PRs, issues, etc.]

---

**Context Quality:** [High/Medium/Low]
**Ready for Handoff:** [Yes/No - any blockers noted]
```

### Step 4: Validate and Report

After writing the file, confirm:

```bash
echo "Context saved: $CONTEXT_FILE"
echo "  Topic: $ARGUMENTS"
echo "  Lines: $(wc -l < "$CONTEXT_FILE")"
echo ""
echo "Quick Start Preview:"
grep -A6 "QUICK START" "$CONTEXT_FILE" | tail -5
```

Report to the user:
- Full file path (absolute path in the obsidian vault)
- Quick Start summary
- Reminder: "Use `/session-refresh` in your next session, then read the context file at `$CONTEXT_FILE` to continue."

## Key Behaviors

- **Topic-Driven**: All context gathering focuses on the specified topic
- **Auto-Discovery**: Proactively search for related files before asking questions
- **Minimal Questions**: Only 4 essential questions, infer the rest from code
- **Structured Output**: Tables and clear sections for easy AI parsing
- **Actionable**: Clear next steps and priorities for continuation
- **Vault-Centralized**: Always saves to the obsidian vault inside initiative folders at `/Users/duncanleung/Develop/obsidian-local-life-manager/06 Projects/{project}/YYYY-MM-DD-name/context.md`, with the project name derived from the current working directory (creates dir if needed)

## When to Use

- Before ending a productive session on a specific topic
- When switching between projects or topics
- Before a long break to preserve working context
- When handing off work to a future session

**Not for:** General session reflection (use `/session-debrief`), refreshing context (use `/session-refresh`).
