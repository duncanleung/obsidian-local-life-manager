# Journal Creation Format

Canonical format for creating a new daily journal (`02 Calendar/YYYY-MM-DD.md`). Referenced by skills that may need to create a journal as a side effect (e.g., `journal-github-activity`, `journal-slack-activity`, `journal-work-log`).

**Note:** `journal-good-morning` uses the richer `08 System/Templates/Daily Template.md` which includes Dataview queries, Open Tasks, and Inbox sections. This is the minimal fallback for when a skill needs to create a journal and the full template is not available.

## Canonical Frontmatter

```yaml
---
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [daily-note]
status: in-progress
---
```

Both `tags` and `status` are required.

## Section Order

```markdown
# {Month} {Day}, {Year} - {DayOfWeek}

## ⭐ Highlight


## 📋 What Did I Do?


## 🔨 What Did I Work On?


## ![[github-logo.png|18]] GitHub Activity


## ![[slack-logo.png|18]] Slack Conversations


## ![[jira-logo.png|18]] JIRA Tickets


## 📚 What Did I Study?


## 📝 Notes

```

## Heading Format

- H1: `# February 13, 2026 - Friday` (full date with day of week)
- H2 sections use emoji prefixes or Obsidian image embeds as shown above
- All sections are empty by default — users fill them in manually or via skills
