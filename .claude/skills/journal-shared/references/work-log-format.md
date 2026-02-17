# Work Log Format Procedure

Single source of truth for adding entries to the `## 🔨 What Did I Work On?` section of daily journals. Referenced by `journal-work-log`, `journal-meeting-notes`, and `journal-daily-review`.

## Output Format

```markdown
## 🔨 What Did I Work On?

| Status | Project | Work Item | Description | Links |
|--------|---------|-----------|-------------|-------|
| ✅ | obsidian-skills | TASK-004 | EOD Journal Freeze — enhanced freeze functionality | [[TASK]] \| [[PLAN]] |
| 🔨 | obsidian-skills | `/journal-github-activity` | New skill to pull GitHub activity into daily journal | [[SKILL]] |
| 🗣️ | — | Sprint Planning | Discussed Q1 priorities with team | [[meeting]] |
| 🚧 | care-portal | Auth module | Login flow working, still need token refresh | — |
```

## Column Definitions

| Column | Content | Examples |
|--------|---------|---------|
| **Status** | Emoji only from Status Verbs table below | ✅, 🚧, 🔨, 🐛, 🚀 |
| **Project** | Project name from Projects Index in CLAUDE.md, or `—` for non-project work | `obsidian-skills`, `care-portal`, `—` |
| **Work Item** | Identifier: task ID, JIRA key, skill name, or brief label | `TASK-004`, `WEB-1234`, `/journal-github-activity`, `Auth module` |
| **Description** | What was done (keep concise) | `Enhanced freeze functionality` |
| **Links** | Obsidian wikilinks or external links, pipe-separated. Use `—` if none | `[[TASK]] \| [[PLAN]]`, `—` |

## Status Verbs

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

## Project Name Resolution

Resolve the project name from the work description:
1. If work references a known project from the Projects Index `projects:` list in CLAUDE.md → use that project name
2. If work references a JIRA ticket prefix (e.g., `WEB-`, `BE-`, `AP-`) → map to the relevant project
3. If work references a skill file path (e.g., `.claude/skills/journal-*`) → use `obsidian-skills` or the relevant project
4. If no project context → use `—`

## Adding a Row

1. Read today's journal (`02 Calendar/YYYY-MM-DD-dayname.md`)
2. Find `## 🔨 What Did I Work On?` section
3. **If table already exists** (has `| Status |` header row) → append new row after the last `|...|` row in the table
4. **If section is empty** (no table) → create table header + separator + first row:
   ```markdown
   | Status | Project | Work Item | Description | Links |
   |--------|---------|-----------|-------------|-------|
   | {emoji} | {project} | {item} | {description} | {links} |
   ```

## Detecting Empty Section

A section counts as "empty" (no real work entries) if:
- The section has no content below the heading, OR
- The section only has the table header + separator (no data rows)

A section has entries if there is at least one `| {emoji} |` data row below the separator.

## Linking Guidelines

**Always link to related docs when logging work on tasks, issues, or projects:**

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

## Placement Rules

1. If `## 🔨 What Did I Work On?` already exists: add row to the table within it
2. If it doesn't exist: insert after `## 📋 What Did I Do?`, before `## ![[github-logo.png|18]] GitHub Activity`
3. If neither anchor section exists: append before `## 📝 Notes` or at the end of file
