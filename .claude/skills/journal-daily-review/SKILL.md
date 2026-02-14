---
name: journal-daily-review
description: Complete daily journal review. Use at end of day or next morning to fill in journal sections, review highlights, and plan tomorrow. Triggers on "daily review", "end of day", "journal review", "what did I do today".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(gh:*), Bash(date:*), Read, Write, Edit, Glob, mcp__claude_ai_Slack__slack_search_public_and_private, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_read_user_profile, mcp__mcp-atlassian__lookupJiraAccountId, mcp__mcp-atlassian__searchJiraIssuesUsingJql, mcp__mcp-atlassian__getJiraIssue, mcp__mcp-atlassian__getAccessibleAtlassianResources
---

Run the Daily Review Workflow. Keep it conversational - ask one thing at a time.

$ARGUMENTS

## Important: Don't ask about empty sections

Do NOT ask the user to fill in empty sections (📋 What Did I Do?, ⭐ Highlight, 📚 What Did I Study?, etc.). The user fills those in manually. Just proceed directly through the steps and freeze the journal.

## Steps

1. **Determine target journal**
   - If `$ARGUMENTS` contains a file path (e.g., `02 Calendar/2026-02-12.md`): use that file directly and extract the date from the filename
   - If `$ARGUMENTS` contains a date (e.g., `2026-02-12`): use `02 Calendar/YYYY-MM-DD.md`
   - If no arguments: run `date +%Y-%m-%d` AND `date +%H` to get today's date and current hour
     - **After-midnight guard**: If current hour < 5 (midnight–4:59AM), default to **yesterday's** date (`date -v-1d +%Y-%m-%d`) — a "daily review" at 2AM almost certainly means reviewing the day that just ended, not the brand-new day
   - The **target date** is the date being reviewed (from the filename or argument)
   - Always run `date +%Y-%m-%d` to know what today's date is (needed for "tomorrow" references)

2. **Journal Entry Setup**
   - Check if the target journal entry exists (`02 Calendar/YYYY-MM-DD.md`)
   - Create from template if not (see `references/template.md`)

3. **🔨 What Did I Work On?** (only if section is empty)
   - If already has table rows: skip (work-log entries are added during the day via `/journal-work-log` in table format — see `.claude/skills/journal-shared/references/work-log-format.md`)
   - A section with only the table header + separator (no data rows) counts as empty
   - If empty: leave empty — user fills this in manually

4. **GitHub Activity** (ALWAYS runs — never skip this step)
   - Read `.claude/skills/journal-shared/references/github-activity.md` for the complete procedure
   - Follow ALL steps (queries, deduplication, URL construction, formatting, and placement) using the target date from Step 1

5. **Slack Conversations** (ALWAYS runs — never skip this step)
   - Read `.claude/skills/journal-shared/references/slack-activity.md` for the complete procedure
   - Follow ALL steps (search, thread grouping, user resolution, filtering, formatting, and placement) using the target date from Step 1

6. **JIRA Tickets** (ALWAYS runs — never skip this step)
   - Read `.claude/skills/journal-shared/references/jira-activity.md` for the complete procedure
   - Follow ALL steps (account ID lookup, JQL search, data extraction, formatting, and placement) using the target date from Step 1

7. **WEB Releases** (ALWAYS runs — never skip this step)
   - Read `.claude/skills/journal-shared/references/jira-releases.md` for the complete procedure
   - Follow ALL steps (JQL search, data extraction, grouping by fixVersion, formatting, and placement) using the target date from Step 1

8. **Memory Capture Check** (silent)
   - Review the conversation for anything memory-worthy
   - If anything qualifies, create a memory file in `.claude/obsidian-memories/`
   - Do this silently unless there's something significant to confirm

9. **Freeze Journal (EOD Snapshot)**
   - **Active journal guard**: Compare the target date to today's date (`date +%Y-%m-%d`). If they are the **same day**, SKIP the entire freeze step and warn: "⚠️ Skipping freeze — this is today's active journal. Dataview blocks stay live. Run again tomorrow or pass yesterday's date explicitly."
   - Find all `dataview` code blocks in the journal
   - For each block:
     a. Parse the DQL query (see `references/dataview-freeze.md`)
     b. Glob files from the FROM folder
     c. Read YAML frontmatter from each file
     d. Evaluate WHERE conditions against frontmatter
     e. Sort results per SORT clause
     f. Generate static markdown table
     g. Replace the code block with the static table
   - If no dataview blocks found: journal already frozen, skip
   - If any block fails to parse: leave it unchanged, warn user
   - After freezing all blocks, append Day Summary:
     ```markdown
     ## 📊 Day Summary
     - Tasks completed today: [count from completed field = today's date]
     - Tasks still open: [count `- [ ]` lines under ## ✅ Open Tasks heading]
     - Inbox items (unprocessed): [count from frozen 📥 Inbox table, or 0]
     - Journal frozen at: HH:MM
     ```

Use bulleted lists in the journal.
