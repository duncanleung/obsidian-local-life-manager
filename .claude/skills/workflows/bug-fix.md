---
name: workflows:bug-fix
description: "Workflow: Bug Fix"
---

# Workflow: Bug Fix

Systematically debug and fix an issue.

## Quick Reference

```bash
/project-validate-troubleshoot → /project-issue → /project-plan → /project-implement → /project-validate-quality → /project-commit
```

## Step-by-Step

### Step 1: Systematic Debugging
```bash
/project-validate-troubleshoot
```
5-step loop: Reproduce → Hypothesize → Test → Identify → Plan Fix

### Step 2: Create Bug Work Item
```bash
/project-issue "Fix [bug description]"
```
AI auto-detects BUG type.

### Step 3: Plan the Fix
```bash
/project-plan
```
Phases: Reproduce test → Fix → Verify → Regression check

### Step 4: Implement the Fix
```bash
/project-implement
```
Or use `/project-advise` to write it yourself with guidance.

### Step 5: Quality Check
```bash
/project-validate-quality
```

### Step 6: Commit the Fix
```bash
/project-commit
```

## Variations

### Quick Fix (Known Issue)
```bash
/project-issue "Fix [bug]" → /project-implement → /project-commit
```

### Critical Bug (Full Process)
```bash
/project-validate-troubleshoot → /project-issue → /project-plan → /project-implement → /project-validate-quality → /project-validate-security-audit → /project-complete
```

## When to Use /project-complete Instead of /project-commit

**Use /project-commit**: Simple fix, low risk, not merging yet
**Use /project-complete**: Complex fix, security-related, ready to merge

## Testing Best Practices

Every bug fix should include:
1. Test that reproduces the bug (fails before fix)
2. Test passes after fix
3. Regression tests for similar edge cases

## Post-Fix Checklist

- [ ] Added test that would catch this in future?
- [ ] Checked for similar bugs elsewhere?
- [ ] Documented why bug occurred (in commit)?
- [ ] Updated relevant documentation?
