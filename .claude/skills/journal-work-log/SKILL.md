---
name: journal-work-log
description: Quick work log entry to today's journal without full review. Use when user wants to quickly log something they did, add a journal entry, or note an activity. Triggers on "log this", "add to journal", "I just did", "quick note", "work log".
model: claude-haiku-4-5-20251001
argument-hint: [what you did]
allowed-tools: Read, Write, Edit, Glob, Bash(date:*)
---

Add this to today's journal (`02 Calendar/YYYY-MM-DD.md`):

$ARGUMENTS

## Work Log Table Format

Read `.claude/skills/journal-shared/references/work-log-format.md` for the complete format reference.

The "🔨 What Did I Work On?" section uses a 5-column table:

```markdown
| Status | Project | Work Item | Description | Links |
|--------|---------|-----------|-------------|-------|
| ✅ | obsidian-skills | TASK-001 | Enhanced Task System — all phases complete | [[TASK]] \| [[PLAN]] |
| 🚧 | care-portal | Auth module | Login flow working, still need token refresh | — |
| 🐛 | care-portal | WEB-1234 | Fixed date parsing bug in daily review | [WEB-1234](https://airvet.atlassian.net/browse/WEB-1234) |
| 🔨 | obsidian-skills | `/research-clip` | New skill for quick-capturing articles | [[SKILL]] |
```

### Status Verbs

Pick the best emoji match for the work:

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

## Steps

1. **Get current date** — run `date +%Y-%m-%d` to confirm today's date
2. **Read today's journal** at `02 Calendar/YYYY-MM-DD.md`
3. **Resolve related docs** — if the work references tasks, plans, issues, or projects:
   - Search for related files using Glob (e.g., `06 Projects/*/issues/*/PLAN.md`, `06 Projects/*/issues/*/TASK.md`)
   - Use Obsidian wikilinks `[[filename]]` to link to them
   - For files in subdirectories, use the full relative path: `[[06 Projects/project/issues/001-name/PLAN.md|TASK-001 Plan]]`
4. **Resolve project name** — match work to a project from the Projects Index in CLAUDE.md:
   - Known project reference → use project name (e.g., `obsidian-skills`, `care-portal`)
   - JIRA ticket prefix (WEB-, BE-, AP-) → map to relevant project
   - Skill file reference (.claude/skills/*) → use `obsidian-skills` or relevant project
   - No project context → use `—`
5. **Choose the right emoji verb** from the Status Verbs table
6. **Add as table row** under the appropriate section:
   - "📋 What Did I Do?" for personal activities (keep as bullet list)
   - "🔨 What Did I Work On?" for technical/coding work (table format)
   - "📚 What Did I Study?" for learning sessions, courses (keep as bullet list)
   - **If table already exists** (has `| Status |` header) → append new row after the last `|...|` row
   - **If section is empty** → create table header + separator + first data row
7. If journal doesn't exist: create it with the format from `.claude/skills/journal-shared/references/journal-format.md`, then add the entry

## Linking Guidelines

**Always link to related docs when logging work on tasks, issues, or projects.**

| Work Type | Link Format |
|-----------|------------|
| Task/issue | `[[06 Projects/{project}/issues/{id}/TASK.md\|TASK-{id}]]` |
| Plan file | `[[06 Projects/{project}/issues/{id}/PLAN.md\|PLAN]]` |
| Skill file | `[[.claude/skills/{skill-name}/SKILL.md\|/skill-name]]` |
| Project README | `[[06 Projects/{project}/README.md\|{project}]]` |
| Meeting note | `[[04 Meetings/{filename}\|{title}]]` |
| GitHub PR | `[PR#{num}](https://github.com/{org}/{repo}/pull/{num})` |
| JIRA ticket | `[{KEY}](https://airvet.atlassian.net/browse/{KEY})` |

Use `—` in the Links column when there are no relevant links.

## Journal Format

If creating a new journal, follow `.claude/skills/journal-shared/references/journal-format.md`.

Keep entries brief but always include links to related docs.

## Important

- Do NOT commit to git. This skill only edits the journal file.
- Do NOT run any git commands.
- Stop after adding the entry — do not perform any follow-up actions.
