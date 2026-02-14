---
name: journal-good-morning
description: Morning routine check-in. Use at start of day to review yesterday, set up today's journal, check learning reviews, and generate task dashboard. Triggers on "good morning", "morning", "start my day", "what's on for today".
model: claude-haiku-4-5-20251001
allowed-tools: Bash(gh:*), Bash(date:*), Read, Write, Edit, Glob, Grep, mcp__claude_ai_Slack__slack_search_public_and_private, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_read_user_profile, mcp__mcp-atlassian__lookupJiraAccountId, mcp__mcp-atlassian__searchJiraIssuesUsingJql, mcp__mcp-atlassian__getJiraIssue, mcp__mcp-atlassian__getAccessibleAtlassianResources
---

Good morning! Run the morning check-in.

## Steps

1. **Get current date first**
   - Run `date +%Y-%m-%d` to confirm today's date
   - Calculate yesterday's date: `date -v-1d +%Y-%m-%d`
   - DO NOT assume the date - always verify

2. **Check yesterday's journal** (`02 Calendar/YYYY-MM-DD.md`)
   - If "🔨 What Did I Work On?" empty: offer to backfill from GitHub
     - "Yesterday's GitHub activity wasn't captured. Want me to pull that?"
     - If yes: Read `.claude/skills/journal-shared/references/github-activity.md` and follow the complete procedure using **yesterday's date** as the target date
   - If "📋 What Did I Do?" empty: just note it
   - If "![[slack-logo.png|18]] Slack Conversations" empty or missing: offer to backfill from Slack
     - "Yesterday's Slack conversations weren't captured. Want me to pull those?"
     - If yes: Read `.claude/skills/journal-shared/references/slack-activity.md` and follow the complete procedure using **yesterday's date** as the target date
   - If "![[jira-logo.png|18]] JIRA Tickets" empty or missing: offer to backfill from JIRA
     - "Yesterday's JIRA tickets weren't captured. Want me to pull those?"
     - If yes: Read `.claude/skills/journal-shared/references/jira-activity.md` and follow the complete procedure using **yesterday's date** as the target date
   - If "![[jira-logo.png|18]] WEB Releases" empty or missing: offer to backfill WEB releases
     - "Yesterday's WEB releases weren't captured. Want me to pull those?"
     - If yes: Read `.claude/skills/journal-shared/references/jira-releases.md` and follow the complete procedure using **yesterday's date** as the target date
   - **Extract Notes for carryover**: Read the `## 📝 Notes` section from yesterday's journal
     - Capture all content between `## 📝 Notes` and the next `## ` heading (or end of file)
     - If the section has content (not just whitespace), save it for step 5

3. **Build Open Tasks todo list**
   - Find the most recent previous journal: look back from yesterday up to 7 days in `02 Calendar/YYYY-MM-DD.md`
   - Extract the `## ✅ Open Tasks` section from the previous journal. Handle both formats during transition:
     - **Todo list** (new format): extract `- [ ] [[name]]...` lines — keep only unchecked items, drop `- [x]` lines
     - **Frozen table** (old format): extract `[[name]]` wikilinks from table rows (e.g., `| [[task-name]] | open | | |`)
     - **Dataview block** (live query): skip — cannot extract items from a code block
   - Scan `03 TaskNotes/*.md` — read each file's YAML frontmatter, collect all tasks where `status != "done" AND status != "cancelled"`
   - Reconcile previous day's list with current TaskNotes:
     - Items from previous day's list that are still open in TaskNotes → **keep in same order**
     - New open tasks in TaskNotes not in previous day's list → **append at bottom**
     - Tasks now done/cancelled in TaskNotes → **remove from list**
   - Build each todo item as: `- [ ] [[filename]] Title` (add ` 📅 YYYY-MM-DD` suffix if the task has a due date set)
   - The title comes from the `# H1 heading` in the task file (not the filename)

4. **Check Dataview plugin** (`.obsidian/plugins/dataview/`)
   - Check if directory exists
   - If missing: warn "⚠️ Dataview plugin not found. Install from Obsidian Community Plugins for live Inbox and Meetings queries."
   - Don't block execution — journal still useful without live queries

5. **Setup today's journal**
   - Check if today's entry exists at `02 Calendar/YYYY-MM-DD.md`
   - **If not**: create from template at `08 System/Templates/Daily Template.md`
     - Note: Template uses Templater syntax - resolve `<% tp.date... %>` to actual dates
     - Set `created` and `modified` to today's date
   - **If exists**: check if Dataview sections are present for Inbox and Meetings (search for ` ```dataview `)
     - If Dataview sections for Inbox/Meetings **missing**: inject them while preserving ALL existing content
       - Insert between `## ✅ Open Tasks` and `## 📋 What Did I Do?`:
         - `## 📥 Inbox (Unsummarized)` (dataview query)
         - `## 🤝 Meetings` (dataview query)
       - Copy exact query blocks from `08 System/Templates/Daily Template.md`
       - NEVER overwrite or remove existing content — only add missing sections
   - **Populate `## ✅ Open Tasks`**: Write the todo list built in step 3 into this section
     - If section has a `dataview` code block (migration): replace the entire code block with the todo list
     - If section already has `- [ ]` todo items: preserve existing items and their order, only append new tasks not already listed
     - If section is empty: write the full todo list
   - **Carry over Notes from yesterday**: If yesterday's `## 📝 Notes` had content (extracted in step 2):
     - Write yesterday's notes into today's `## 📝 Notes` section
     - If today's Notes section already has content, append yesterday's notes below existing content with a blank line separator
     - Do NOT add any "carried over" labels or attribution — just copy the content as-is
   - Show today's highlight or ask: "What's your main focus today?"
   - **Update DASHBOARD**: Write `![[YYYY-MM-DD]]` (today's date) to `_DASHBOARD.md` in the vault root, replacing the entire file contents. This keeps the dashboard always pointing to the current day's journal.

6. **Check learning plan** (`.claude/learning-sessions/learning-plan.json`)
   - Find topics where `last_covered` + interval < today
   - If any due: "You have [topic] due for review"
   - Show next item in queue

7. **Scan tasks** (`03 TaskNotes/*.md`)
   - Already scanned in step 3 — reuse that data
   - Categorize:
     - **Open**: `due` is null/missing OR `due` >= today
   - For date comparison on macOS, use `date -v` for arithmetic

8. **Scan meetings** (`04 Meetings/*.md`)
   - Read each file's YAML frontmatter
   - Count meetings where `created` = today's date
   - Count meetings where `created` = yesterday's date

9. **Scan in-progress ideas** (`06 Projects/*/README.md`)
   - Read each README's YAML frontmatter
   - Include projects where `status: active`

10. **Quick status report**
    - Count unprocessed inbox items by scanning `01 Inbox/`:
      - Match: `status: Clipped`
    - Yesterday: Complete/Incomplete
    - Today's highlight: [highlight or "not set"]
    - Open tasks: [count from todo list built in step 3]
    - Open JIRA tickets: [count by calling jira-activity procedure quietly]
    - Unreleased WEB versions: [count versions] ([total issues] issues) — query via jira-releases procedure
    - Meetings today: [count]
    - Inbox (unprocessed): [count]
    - In-progress ideas: [count]
    - Reviews due: [list or "none"]
    - Next in learning queue: [topic]
    - Journal: `02 Calendar/YYYY-MM-DD.md` (Open Tasks as todo list, Inbox/Meetings as live Dataview)

Keep it brief - quick morning orientation, not a deep dive.

## Dataview Rules

- **No trailing slashes in FROM paths**: `FROM "03 TaskNotes"` works, `FROM "03 TaskNotes/"` silently returns empty.
- **YAML null for unset fields**: Use `due:` (null), never `due: ""` (empty string). Dataview treats `""` as truthy, breaking `!due` checks.
- **Open Tasks uses todo list format** — not Dataview. Only Inbox and Meetings sections use Dataview queries.
