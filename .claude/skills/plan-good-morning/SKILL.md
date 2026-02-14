---
name: plan-good-morning
description: Morning routine check-in. Use at start of day to review yesterday, set up today's journal, check learning reviews, and generate task dashboard. Triggers on "good morning", "morning", "start my day", "what's on for today".
model: claude-haiku-4-5-20251001
allowed-tools: Bash(gh:*), Bash(date:*), Read, Write, Edit, Glob, Grep, mcp__claude_ai_Slack__slack_search_public_and_private, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_read_user_profile
---

Good morning! Run the morning check-in.

## Steps

1. **Get current date first**
   - Run `date +%Y-%m-%d` to confirm today's date
   - Calculate yesterday's date: `date -v-1d +%Y-%m-%d`
   - DO NOT assume the date - always verify

2. **Check yesterday's journal** (`02 Calendar/YYYY-MM-DD.md`)
   - If "🔨 What Did I Work On?" empty: offer to backfill from GitHub
   - If "📋 What Did I Do?" empty: just note it
   - If "![[slack-logo.png|18]] Slack Conversations" empty or missing: offer to backfill from Slack
     - "Yesterday's Slack conversations weren't captured. Want me to pull those?"
     - If yes: use same search/filter strategy as `/plan-daily-review` Step 5 with yesterday's date

3. **Check Dataview plugin** (`.obsidian/plugins/dataview/`)
   - Check if directory exists
   - If missing: warn "⚠️ Dataview plugin not found. Install from Obsidian Community Plugins for live task queries."
   - Don't block execution — journal still useful without live queries

4. **Setup today's journal**
   - Check if today's entry exists at `02 Calendar/YYYY-MM-DD.md`
   - **If not**: create from template at `08 System/Templates/Daily Template.md`
     - Note: Template uses Templater syntax - resolve `<% tp.date... %>` to actual dates
     - Set `created` and `modified` to today's date
   - **If exists**: check if Dataview sections are present (search for ` ```dataview `)
     - If Dataview sections **missing**: inject them while preserving ALL existing content
       - Insert these 3 sections between `## ⭐ Highlight` and `## 📋 What Did I Do?`:
         - `## ✅ Open Tasks` (dataview query)
         - `## 📎 Clipped (Unsummarized)` (dataview query)
         - `## 🤝 Meetings` (dataview query)
       - Copy exact query blocks from `08 System/Templates/Daily Template.md`
       - NEVER overwrite or remove existing content — only add missing sections
     - If Dataview sections **present**: do nothing, journal is already up to date
   - Show today's highlight or ask: "What's your main focus today?"

5. **Check learning plan** (`.claude/learning-sessions/learning-plan.json`)
   - Find topics where `last_covered` + interval < today
   - If any due: "You have [topic] due for review"
   - Show next item in queue

6. **Scan tasks** (`03 TaskNotes/*.md`)
   - Read each file's YAML frontmatter
   - Skip tasks where status is `complete` or `cancelled`
   - Categorize:
     - **Open**: `due` is null/missing OR `due` >= today
   - For date comparison on macOS, use `date -v` for arithmetic

7. **Scan meetings** (`04 Meetings/*.md`)
   - Read each file's YAML frontmatter
   - Count meetings where `created` = today's date
   - Count meetings where `created` = yesterday's date

8. **Scan in-progress ideas** (`06 Projects/*/README.md`)
   - Read each README's YAML frontmatter
   - Include projects where `status: active`

9. **Quick status report**
   - Count unread clips by scanning `01 Inbox/`:
     - Match: `status: Clipped`
   - Yesterday: Complete/Incomplete
   - Today's highlight: [highlight or "not set"]
   - Open tasks: [count]
   - Meetings today: [count]
   - Unread clips: [count]
   - In-progress ideas: [count]
   - Reviews due: [list or "none"]
   - Next in learning queue: [topic]
   - Journal: `02 Calendar/YYYY-MM-DD.md` (with live Dataview queries)

Keep it brief - quick morning orientation, not a deep dive.

## Dataview Rules

- **No trailing slashes in FROM paths**: `FROM "03 TaskNotes"` works, `FROM "03 TaskNotes/"` silently returns empty.
- **YAML null for unset fields**: Use `due:` (null), never `due: ""` (empty string). Dataview treats `""` as truthy, breaking `!due` checks.
