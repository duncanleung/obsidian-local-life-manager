# Task Note Schema

## Location

All task notes live in `03 TaskNotes/` as individual markdown files.

## Filename Convention

- Kebab-case: `schedule-grooming-loosa.md`
- Lowercase, hyphens for spaces, no special characters
- No numbered prefixes (unlike issues in 06 Projects/)

## Frontmatter Schema

```yaml
---
status: open           # open | in_progress | done | cancelled
created: "YYYY-MM-DD"  # date task was created
due:                   # "YYYY-MM-DD" or null (YAML null = bare key, no value)
completed:             # "YYYY-MM-DD" or null (set by /journal-task-done)
priority:              # high | medium | low | null
source:                # URL or description of origin, or null
source_type:           # slack | email | jira | github | meeting | other | null
tags: [task]           # always includes "task" for Obsidian queries
---
```

**IMPORTANT**: Optional fields use YAML null (bare key with no value), NOT empty strings `""`. Empty strings break Dataview queries because Dataview treats `""` as a set value, not as missing/empty.

### Field Reference

| Field | Type | Required | Values | Description |
|-------|------|----------|--------|-------------|
| `status` | enum | yes | `open`, `in_progress`, `done`, `cancelled` | Current task state (note: `done` replaces `complete` for Dataview compatibility) |
| `created` | date | yes | `YYYY-MM-DD` | When the task was created |
| `due` | date | no | `YYYY-MM-DD` or null | When the task is due |
| `completed` | date | no | `YYYY-MM-DD` or null | When the task was completed (added by /journal-task-done) |
| `priority` | enum | no | `high`, `medium`, `low`, or null | Task priority level |
| `source` | string | no | URL or description, or null | Where the task came from (Slack message, JIRA ticket, etc.) |
| `source_type` | enum | no | `slack`, `email`, `jira`, `github`, `meeting`, `other`, or null | Type of source, auto-detected from URL patterns |
| `tags` | array | yes | `[task, ...]` | Always includes `task`; user can add more |

### Source Type Auto-Detection

When using `/journal-quick-task --source <url>`, the `source_type` is automatically detected:

| URL Pattern | Detected Type |
|-------------|---------------|
| `slack://` or `*.slack.com` | `slack` |
| `*.atlassian.net` or `*.jira.com` | `jira` |
| `github.com` | `github` |
| `mail.google.com` or `outlook.` | `email` |
| Explicit `--source-type` flag | Use provided value |
| No URL or unrecognized pattern | `other` (if source provided) or null |

### Status Transitions

```
open -> in_progress -> done
open -> cancelled
open -> done (skip in_progress for quick tasks)
cancelled -> open (reopen)
```

**Note**: Both `done` and `complete` are valid status values for backwards compatibility, but new tasks use `done` for consistency with Dataview queries.

## Relationship to 06 Projects/ Issues

Task notes in `03 TaskNotes/` are **personal tasks** -- errands, calls, appointments, chores.
They are distinct from project issues in `06 Projects/[project]/issues/` which track development work.

The morning dashboard (`/journal-daily-review`) combines both:
- Task notes from `03 TaskNotes/` (personal tasks)
- Active projects from `06 Projects/*/README.md` (project work)

## Obsidian Integration

- Files use `#task` tag for Obsidian Dataview/tag queries
- Wikilinks `[[task-name]]` work in the dashboard
- Checkboxes `- [ ]` are interactive in Obsidian
- Template available at `08 System/Templates/Task Template.md`
