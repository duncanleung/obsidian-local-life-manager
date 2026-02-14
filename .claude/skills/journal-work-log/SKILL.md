---
name: journal-work-log
description: Quick work log entry to today's journal without full review. Use when user wants to quickly log something they did, add a journal entry, or note an activity. Triggers on "log this", "add to journal", "I just did", "quick note", "work log".
model: claude-haiku-4-5-20251001
argument-hint: [what you did]
allowed-tools: Read, Write, Edit, Glob, Bash(date:*)
---

Add this to today's journal (`02 Calendar/YYYY-MM-DD.md`):

$ARGUMENTS

## Status Verbs

Every work log bullet MUST start with an emoji verb. Pick the best match:

| Emoji | Verb | Use when… |
|-------|------|-----------|
| ✅ | Completed | Finished a task, issue, or milestone |
| 🚧 | Progressed | Made progress but not done yet |
| 🚀 | Shipped | Deployed, released, or merged to production |
| 🐛 | Fixed | Resolved a bug or defect |
| 🔨 | Built | Created something new (feature, component, tool) |
| 🔧 | Updated | Modified, enhanced, or improved existing work |
| 📐 | Designed | Created specs, plans, architecture, or UI designs |
| 📝 | Documented | Wrote docs, READMEs, ADRs, or notes |
| 🔍 | Investigated | Researched, debugged, or explored a topic |
| 🧪 | Tested | Wrote or ran tests, validated behavior |
| ♻️ | Refactored | Restructured code without changing behavior |
| 💡 | Discovered | Found an insight, learned something key |
| 🗣️ | Discussed | Had a meeting, review, or conversation |
| 📋 | Planned | Created tasks, broke down work, triaged |
| ⏸️ | Paused | Put work on hold, context-switched away |
| 🔄 | Reviewed | Code review, PR review, or feedback |

**Format:** `- {emoji} {Verb} {description} — {optional links}`

Example:
```markdown
- ✅ Completed TASK-001 (Enhanced Task System) — [[06 Projects/obsidian-skills/issues/001-enhanced-task-system/TASK.md|TASK-001]] | [[06 Projects/obsidian-skills/issues/001-enhanced-task-system/PLAN.md|PLAN]]
- 🚧 Progressed on auth module — login flow working, still need token refresh
- 🐛 Fixed date parsing bug in daily review skill
- 🔨 Built `/research-clip` skill for quick-capturing articles
```

## Steps

1. **Get current date** — run `date +%Y-%m-%d` to confirm today's date
2. **Read today's journal** at `02 Calendar/YYYY-MM-DD.md`
3. **Resolve related docs** — if the work references tasks, plans, issues, or projects:
   - Search for related files using Glob (e.g., `06 Projects/*/issues/*/PLAN.md`, `06 Projects/*/issues/*/TASK.md`)
   - Use Obsidian wikilinks `[[filename]]` to link to them
   - For files in subdirectories, use the full relative path: `[[06 Projects/project/issues/001-name/PLAN.md|TASK-001 Plan]]`
4. **Choose the right emoji verb** from the Status Verbs table above that best describes the work
5. **Add as bullet** under the appropriate section:
   - "📋 What Did I Do?" for personal activities
   - "🔨 What Did I Work On?" for technical/coding work
   - "📚 What Did I Study?" for learning sessions, courses
   - Every bullet MUST start with an emoji verb from the table
6. If journal doesn't exist: create it with the format shown below, then add the entry

## Linking Guidelines

**Always link to related docs when logging work on tasks, issues, or projects:**

```markdown
# Good — emoji verb + links to source docs
- ✅ Completed TASK-001 (Enhanced Task System) — [[06 Projects/obsidian-skills/issues/001-enhanced-task-system/PLAN.md|PLAN]] | [[06 Projects/obsidian-skills/issues/001-enhanced-task-system/TASK.md|TASK]]

# Bad — no emoji, no links, can't scan or navigate
- Completed TASK-001 (Enhanced Task System)
```

**Link types to include:**
- Task/issue file: `[[06 Projects/{project}/issues/{id}/TASK.md|TASK-{id}]]`
- Plan file: `[[06 Projects/{project}/issues/{id}/PLAN.md|PLAN]]`
- Skill files: `[[.claude/skills/{skill-name}/SKILL.md|/skill-name]]`
- Project README: `[[06 Projects/{project}/README.md|{project}]]`

## Journal Format

If creating a new journal, follow `.claude/skills/journal-shared/references/journal-format.md`.

Keep entries brief but always include links to related docs.

## Important

- Do NOT commit to git. This skill only edits the journal file.
- Do NOT run any git commands.
- Stop after adding the entry — do not perform any follow-up actions.
