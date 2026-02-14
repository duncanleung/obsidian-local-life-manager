---
name: journal-jira-releases
description: Pull WEB board unreleased versions into today's journal. Shows issues grouped by release with status summary. Safe to run anytime — does NOT freeze dataviews. Triggers on "jira releases", "web releases", "pull releases", "what releases".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(date:*), Read, Write, Edit, Glob, mcp__mcp-atlassian__searchJiraIssuesUsingJql, mcp__mcp-atlassian__getJiraIssue
---

Pull WEB board unreleased versions into the daily journal. This is safe to run throughout the day — it only touches the `## ![[jira-logo.png|18]] WEB Releases` section and does NOT freeze dataview blocks.

$ARGUMENTS

## Steps

1. **Determine target journal**
   - If `$ARGUMENTS` contains a file path (e.g., `02 Calendar/2026-02-12.md`): use that file directly and extract the date from the filename
   - If `$ARGUMENTS` contains a date (e.g., `2026-02-12`): use `02 Calendar/YYYY-MM-DD.md`
   - If no arguments: run `date +%Y-%m-%d` to get today's date, use `02 Calendar/YYYY-MM-DD.md`

2. **Read the journal file**
   - If it doesn't exist: create it with the standard format (see bottom of this file)
   - Note whether a `## ![[jira-logo.png|18]] WEB Releases` section already exists (it will be replaced)

3. **Pull, format, and place WEB Releases**
   - Read `.claude/skills/journal-shared/references/jira-releases.md` for the complete procedure
   - Follow ALL steps (JQL search, data extraction, grouping, formatting, and placement) using the target date from Step 1

## Important

- Do NOT freeze dataview blocks — that's only for `/journal-daily-review`
- Do NOT touch any other sections (📋 What Did I Do?, ⭐ Highlight, ![[github-logo.png|18]] GitHub Activity, ![[slack-logo.png|18]] Slack Conversations, ![[jira-logo.png|18]] JIRA Tickets, etc.)
- Do NOT commit to git
- Stop after updating the WEB Releases section

## Journal Format

If creating a new journal, follow `.claude/skills/journal-shared/references/journal-format.md`.
