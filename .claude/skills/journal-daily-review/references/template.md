# Daily Journal Template

## Expected Structure

The daily journal template should include sections with Dataview query blocks that will be frozen during the daily review process.

### H1 Heading Format

The H1 heading includes the day of the week:

```markdown
# {Month} {Day}, {Year} - {DayOfWeek}
```

Example: `# February 13, 2026 - Friday`

The Obsidian Templater syntax is: `<% tp.date.now("MMMM DD, YYYY - dddd") %>`

### Common Dataview Sections

The following sections are expected to contain `dataview` code blocks:

**🚨 Overdue Tasks**
```dataview
TABLE status AS "Status", due AS "Due", source_type AS "Source"
FROM "03 TaskNotes/"
WHERE status != "done" AND status != "cancelled" AND due < date(today)
SORT due ASC
```

**📅 Due Today**
```dataview
TABLE status AS "Status", due AS "Due", source_type AS "Source"
FROM "03 TaskNotes/"
WHERE status != "done" AND status != "cancelled" AND due = date(today)
```

**✅ Open Tasks** — Uses a plain markdown todo list (not Dataview). Items are `- [ ] [[filename]] Title` checkboxes that the user reorders by priority in Obsidian. This section is NOT frozen during daily review — it stays as-is. The Day Summary counts `- [ ]` lines to get "Tasks still open".

**📥 Inbox (Unsummarized)**
```dataview
TABLE class AS "Class", url AS "URL"
FROM "01 Inbox/"
WHERE status = "Clipped"
SORT file.mtime DESC
```

### GitHub Activity Section

The `## ![[github-logo.png|18]] GitHub Activity` section is auto-populated by the daily review skill (Step 4). It pulls commits, PRs, reviews, and comments from GitHub for the target date. The section is placed after `## 🔨 What Did I Work On?` and before `## 📚 What Did I Study?`.

Uses tables with H3 headers and summary counts (matching JIRA Releases pattern):

```markdown
## ![[github-logo.png|18]] GitHub Activity

### Commits — {N} commits across {M} repos

| Repo | Commit | Message |
|------|--------|---------|
| {org/repo} | [`{short-sha}`](https://github.com/{org}/{repo}/commit/{full-sha}) | {commit message title} |

### Pull Requests — {N} PRs ({X} Merged, {Y} Opened)

| PR | Title | Status |
|----|-------|--------|
| [{org/repo}#{number}](https://github.com/{org}/{repo}/pull/{number}) | {title} | <span style="color: green">Merged</span> |

### Reviews & Comments — {N} items

| PR | Title | Activity |
|----|-------|----------|
| [{org/repo}#{number}](https://github.com/{org}/{repo}/pull/{number}) | {title} | 💬 Commented |
```

Subsections with zero items are omitted. If no GitHub activity at all, shows `_No GitHub activity._`. PR statuses use colored `<span>` tags (green=Merged, blue=Opened, red=Closed).

### Day Summary Section

The freeze process automatically appends a Day Summary section:

```markdown
## 📊 Day Summary
- Tasks completed today: [count]
- Tasks still open: [count]
- Inbox items (unprocessed): [count]
- Journal frozen at: HH:MM
```

This section is auto-populated during the freeze step and should not be manually added to the template.

## Notes

- The exact template structure may vary based on user preferences
- The freeze process works with any journal that contains `dataview` code blocks
- If no dataview blocks are found, the journal is considered already frozen