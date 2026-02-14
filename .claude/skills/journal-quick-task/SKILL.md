---
name: journal-quick-task
description: Create a new task note in 03 TaskNotes/. Use to quickly capture a task with optional due date. Triggers on "new task", "add task", "quick task", "create task", "todo".
model: claude-haiku-4-5-20251001
allowed-tools: Read, Write, Glob, Bash(date:*)
argument-hint: "task title" [--due DATE] [--source URL] [--source-type TYPE] [--priority LEVEL]
---

Create a new task note in `03 TaskNotes/`.

$ARGUMENTS

## Steps

1. **Get current date**
   - Run `date +%Y-%m-%d` to confirm today's date
   - DO NOT assume the date - always verify

2. **Parse arguments**
   - Extract task title (required)
   - Extract due date if provided with `--due`
   - Extract source URL/description if provided with `--source`
   - Extract source type if provided with `--source-type`
   - Extract priority level if provided with `--priority`
   - Due date formats accepted:
     - Exact: `2026-02-15`, `feb 15`, `February 15`
     - Relative: `today`, `tomorrow`, `next monday`, `in 3 days`, `next week`
   - Convert all dates to `YYYY-MM-DD` format
   - For relative dates on macOS, use `date -v` syntax:
     - tomorrow: `date -v+1d +%Y-%m-%d`
     - next monday: `date -v+monday +%Y-%m-%d`
     - in 3 days: `date -v+3d +%Y-%m-%d`
     - next week: `date -v+7d +%Y-%m-%d`

   **Source type auto-detection** (if `--source` provided but no `--source-type`):
   - `slack://` or `*.slack.com` → `source_type: "slack"`
   - `*.atlassian.net` or `*.jira.com` → `source_type: "jira"`
   - `github.com` → `source_type: "github"`
   - `mail.google.com` or `outlook.` → `source_type: "email"`
   - Unrecognized URL → `source_type: "other"`
   - Non-URL text → `source_type: "other"`

   **Priority values**: `high`, `medium`, `low` (case insensitive)
   **Source type values**: `slack`, `email`, `jira`, `github`, `meeting`, `other`

3. **Generate filename**
   - Convert title to kebab-case: lowercase, spaces to hyphens, remove special characters
   - Example: "Buy groceries" -> `buy-groceries.md`
   - Example: "Call Dr. Smith about results" -> `call-dr-smith-about-results.md`
   - Check if file already exists in `03 TaskNotes/` - if so, warn and ask

4. **Create task note**

   Write to: `03 TaskNotes/{filename}.md`

   ```markdown
   ---
   status: open
   created: "YYYY-MM-DD"
   due: "YYYY-MM-DD"
   completed:
   priority: "high"
   source: "https://github.com/airvet/repo/pull/123"
   source_type: "github"
   tags: [task]
   ---

   # Task Title

   ## Notes

   ```

   - If no due date provided, set `due:` (YAML null — no value, no quotes)
   - If no priority provided, set `priority:` (YAML null)
   - If no source provided, set `source:` and `source_type:` (YAML null)
   - If source provided but no explicit source_type, use auto-detection
   - Title in heading uses original case
   - Always set `completed:` (YAML null — populated later by /journal-task-done)
   - IMPORTANT: Use YAML null (bare key with no value) for unset fields, NOT empty strings `""`. Empty strings break Dataview queries.

5. **Add to today's journal**
   - Find today's journal: `02 Calendar/YYYY-MM-DD.md`
   - If journal exists and has `## ✅ Open Tasks`:
     - Find the section (between `## ✅ Open Tasks` and the next `##` heading)
     - Append a new line: `- [ ] [[filename]] Title` (add ` 📅 YYYY-MM-DD` suffix if due date is set)
   - If journal doesn't exist: skip (will be populated by `/journal-good-morning`)

6. **Confirm creation**
   ```
   Created: 03 TaskNotes/review-pr-from-sarah.md
   Due: 2026-02-13 (Thursday)
   Priority: high
   Source: github (https://github.com/airvet/repo/pull/123)
   ```
   - If no due date: `Due: not set`
   - If no priority: omit priority line
   - If no source: omit source line
   - If source provided: show `source_type (source_url_or_description)`

## Examples

```
/journal-quick-task "Buy groceries" --due tomorrow
/journal-quick-task "Schedule grooming for Loosa" --due 2026-02-14
/journal-quick-task "Read chapter 5 of DDIA"
/journal-quick-task "Call with Hope" --due today
/journal-quick-task "Review PR from Sarah" --source https://github.com/airvet/repo/pull/123
/journal-quick-task "Reply to Dr. Kim about scheduling" --source-type email --priority high
/journal-quick-task "Update JIRA ticket" --source https://airvet.atlassian.net/browse/AP-1234
/journal-quick-task "Urgent bug fix" --source https://airvet.atlassian.net/browse/AP-1234 --priority high --due today
```

## Notes

- One task per file, always in `03 TaskNotes/`
- Keep the note minimal - users can add detail in Obsidian later
- Tags array uses `[task]` for Obsidian tag queries
- See `references/task-schema.md` for the full frontmatter specification
