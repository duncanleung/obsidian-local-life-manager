---
name: journal-github-activity
description: Pull GitHub activity (commits, PRs, reviews, comments) into today's journal. Safe to run anytime — does NOT freeze dataviews. Triggers on "github activity", "pull github", "git activity", "what did I push".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(gh:*), Bash(date:*), Read, Write, Edit, Glob
---

Pull GitHub activity into the daily journal. This is safe to run throughout the day — it only touches the `## ![[github-logo.png|18]] GitHub Activity` section and does NOT freeze dataview blocks.

$ARGUMENTS

## Steps

1. **Determine target journal**
   - If `$ARGUMENTS` contains a file path (e.g., `02 Calendar/2026-02-12-thursday.md`): use that file directly and extract the date from the filename
   - If `$ARGUMENTS` contains a date (e.g., `2026-02-12`): compute the day name with `date -j -f %Y-%m-%d "2026-02-12" +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'`, use `02 Calendar/YYYY-MM-DD-dayname.md`
   - If no arguments: run `date +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'` to get today's filename stem, use `02 Calendar/YYYY-MM-DD-dayname.md`

2. **Read the journal file**
   - If it doesn't exist: create it with the standard format (see bottom of this file)
   - Note whether a `## ![[github-logo.png|18]] GitHub Activity` section already exists (it will be replaced)

3. **Pull, format, and place GitHub Activity**
   - Read `.claude/skills/journal-shared/references/github-activity.md` for the complete procedure
   - Follow ALL steps (queries, deduplication, URL construction, formatting, and placement) using the target date from Step 1

## Important

- Do NOT freeze dataview blocks — that's only for `/journal-daily-review`
- Do NOT touch any other sections (📋 What Did I Do?, ⭐ Highlight, etc.)
- Do NOT commit to git
- Stop after updating the GitHub Activity section

## Journal Format

If creating a new journal, follow `.claude/skills/journal-shared/references/journal-format.md`.
