---
name: workflows:new-project
description: "Workflow: New Project (Concept → First Commit)"
---

# Workflow: New Project

Take an idea from concept to first working implementation.

## Phase 1: Foundation (30-60 minutes)

### Step 1: Create Project Brief
```bash
/project-brief
```
Output: `06 Projects/[project]/README.md` + `project-brief.md`

### Step 2: Validate Documentation
```bash
/project-validate
```

### Step 3: Get Critical Evaluation
```bash
/project-critique
```
**Decision point**:
- STRONG/PROMISING → Proceed
- NEEDS WORK → Address concerns
- RECONSIDER → Pivot or shelve

## Phase 2: Address Concerns (Variable)

### Step 4: Research Key Questions
```bash
/research-deep "[concern from critique]"
```

## Phase 3: Planning (1-4 hours)

### Step 5: Define First Feature
```bash
/project-spec "MVP feature set"
```

### Step 6: Document Tech Stack Decision
```bash
/project-adr
```

### Step 7 (Optional): Create UI Mockups
```bash
/project-ui-design
```

## Phase 4: Implementation (Variable)

### Step 8: Create Work Item
```bash
/project-issue "Build MVP"
```

### Step 9: Plan Implementation Phases
```bash
/project-plan
```

### Step 10: Choose Implementation Mode

**Option A**: AI writes code (fastest)
```bash
/project-implement
```

**Option B**: You write with guidance
```bash
/project-advise
```

**Option C**: Learn while building
```bash
/project-teach
```

### Step 11: Track Progress
```bash
/project-worklog
```

## Phase 5: Quality & Completion (30-60 minutes)

### Step 12: Code Quality Check
```bash
/project-validate-quality
```

### Step 13: Security Audit
```bash
/project-validate-security-audit
```

### Step 14 (Optional): Verify Spec Implemented
```bash
/project-spec-validate
```

### Step 15: Complete and Merge
```bash
/project-complete
```

## Variations

### Lean Startup (Skip Some Steps)
```bash
/project-brief → /project-spec → /project-issue → /project-implement → /project-commit
```

### Research-Heavy (More Analysis)
```bash
/project-brief → /project-critique → /research-deep → /research-deep → /project-spec → /project-adr → /project-implement
```

### Learning-Focused (You Write Code)
```bash
/project-brief → /project-spec → /project-issue → /project-plan → /project-teach → /project-worklog → /project-validate-quality
```
