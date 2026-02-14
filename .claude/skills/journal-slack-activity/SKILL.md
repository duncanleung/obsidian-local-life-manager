---
name: journal-slack-activity
description: Pull Slack conversations into today's journal. Safe to run anytime — does NOT freeze dataviews. Triggers on "slack activity", "pull slack", "slack conversations", "what did I discuss".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(date:*), Read, Write, Edit, Glob, mcp__claude_ai_Slack__slack_search_public_and_private, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_read_user_profile
---

Pull Slack conversations into the daily journal. This is safe to run throughout the day — it only touches the `## ![[slack-logo.png|18]] Slack Conversations` section and does NOT freeze dataview blocks.

$ARGUMENTS

## Steps

1. **Determine target journal**
   - If `$ARGUMENTS` contains a file path (e.g., `02 Calendar/2026-02-12-thursday.md`): use that file directly and extract the date from the filename
   - If `$ARGUMENTS` contains a date (e.g., `2026-02-12`): compute the day name with `date -j -f %Y-%m-%d "2026-02-12" +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'`, use `02 Calendar/YYYY-MM-DD-dayname.md`
   - If no arguments: run `date +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'` to get today's filename stem, use `02 Calendar/YYYY-MM-DD-dayname.md`

2. **Read the journal file**
   - If it doesn't exist: create it with the standard format (see bottom of this file)
   - Note whether a `## ![[slack-logo.png|18]] Slack Conversations` section already exists (it will be replaced)

3. **Pull, format, and place Slack Conversations**
   - Read `.claude/skills/journal-shared/references/slack-activity.md` for the complete procedure
   - Follow ALL steps (search, thread grouping, user resolution, filtering, formatting, and placement) using the target date from Step 1

## Important

- Do NOT freeze dataview blocks — that's only for `/journal-daily-review`
- Do NOT touch any other sections (📋 What Did I Do?, ⭐ Highlight, ![[github-logo.png|18]] GitHub Activity, etc.)
- Do NOT commit to git
- Stop after updating the Slack Conversations section

## Journal Format

If creating a new journal, follow `.claude/skills/journal-shared/references/journal-format.md`.
