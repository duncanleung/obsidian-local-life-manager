---
name: journal-daily-review
description: Unified daily journal skill. Morning — sets up today, processes yesterday. Evening — fills activities, freezes. Any time — pulls activities for target date. Triggers on "daily review", "end of day", "journal review", "good morning", "morning", "start my day", "what's on for today".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(gh:*), Bash(git:*), Bash(date:*), Read, Write, Edit, Glob, Grep, mcp__claude_ai_Slack__slack_search_public_and_private, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_read_user_profile, mcp__mcp-atlassian__lookupJiraAccountId, mcp__mcp-atlassian__searchJiraIssuesUsingJql, mcp__mcp-atlassian__getJiraIssue, mcp__mcp-atlassian__getAccessibleAtlassianResources
---

Run the Daily Review Workflow.

$ARGUMENTS

## Important: Don't ask about empty sections

Do NOT ask the user to fill in empty sections (📋 What Did I Do?, ⭐ Highlight, 📚 What Did I Study?, etc.). The user fills those in manually. Just proceed directly through the steps.

## Steps

### 1. Determine dates and mode

- Run `date +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'` to get today's filename stem (e.g., `2026-02-14-saturday`)
- Run `date -v-1d +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'` to get yesterday's stem
- Run `date +%H` to get current hour
- Always run `date +%Y-%m-%d` to know today's date

**If `$ARGUMENTS` contains a file path** (e.g., `02 Calendar/2026-02-12-thursday.md`): use that file directly, extract date from filename. Set `explicit_target = true`.

**If `$ARGUMENTS` contains a date** (e.g., `2026-02-12`): compute day name with `date -j -f %Y-%m-%d "2026-02-12" +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'`, use `02 Calendar/YYYY-MM-DD-dayname.md`. Set `explicit_target = true`.

**If no arguments:**
- If current hour < 5 (midnight–4:59AM): set target to **yesterday** (after-midnight guard — a review at 2AM means reviewing the day that just ended). Set `explicit_target = false`.
- Otherwise: set target to **today**. Set `explicit_target = false`.

Compute flags:
- `today_is_target`: whether target date = today's date
- `explicit_target`: whether user passed explicit date/path

---

### 2. Process yesterday's journal (CONDITIONAL)

**Run this step ONLY when ALL of:** `explicit_target = false` AND `today_is_target = true` AND yesterday's journal exists AND yesterday's journal is unfrozen (contains `` ```dataview `` blocks OR frontmatter `status` is not `done`)

This auto-completes yesterday so you don't have to run a separate command.

a. Read yesterday's journal

b. **Pull activities for yesterday** (only sections that are empty or missing — skip already-populated sections):
   - GitHub Activity: read `.claude/skills/journal-shared/references/github-activity.md`, follow all steps using yesterday's date
   - Slack Conversations: read `.claude/skills/journal-shared/references/slack-activity.md`, follow all steps using yesterday's date
   - JIRA Tickets: read `.claude/skills/journal-shared/references/jira-activity.md`, follow all steps using yesterday's date
   - WEB Releases: read `.claude/skills/journal-shared/references/jira-releases.md`, follow all steps using yesterday's date

c. **Extract Notes for carryover**: Read `## 📝 Notes` section from yesterday's journal
   - Capture all content between `## 📝 Notes` and the next `## ` heading (or end of file)
   - If has content (not just whitespace), save for Step 4

d. **Freeze yesterday's journal**: Execute the full freeze algorithm per `references/dataview-freeze.md` — find all `dataview` code blocks, parse DQL, glob files, read frontmatter, evaluate WHERE, sort, generate static tables, replace blocks. If no dataview blocks: already frozen, skip. After freezing, append Day Summary:
   ```markdown
   ## 📊 Day Summary
   - Tasks completed today: [count from completed field = yesterday's date]
   - Tasks still open: [count `- [ ]` lines under ## ✅ Open Tasks heading]
   - Inbox items (unprocessed): [count from frozen 📥 Inbox table, or 0]
   - Journal frozen at: HH:MM
   ```

---

### 3. Target journal setup

- Check if target journal exists at `02 Calendar/YYYY-MM-DD-dayname.md`
- If not: create from template (see `references/template.md`)
- Track whether journal was newly created: `journal_created = true/false`

---

### 4. Morning setup (CONDITIONAL)

**Run this step ONLY when ALL of:** `explicit_target = false` AND `today_is_target = true`

a. **Build Open Tasks todo list**
   - Read today's existing `## ✅ Open Tasks` section — extract any `- [ ]` lines as "today's existing items" (preserve their exact text and order)
   - Find the most recent previous journal: look back from yesterday up to 7 days in `02 Calendar/YYYY-MM-DD-dayname.md` (compute each date's filename stem with `date -v-Nd +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'`)
   - Extract the `## ✅ Open Tasks` section from the previous journal. Handle formats:
     - **Todo list**: extract `- [ ] [[name]]...` lines — keep only unchecked items, drop `- [x]` lines
     - **Frozen table**: extract `[[name]]` wikilinks from table rows
     - **Dataview block** (live query): skip — cannot extract items from a code block
   - Scan `03 TaskNotes/*.md` — read each file's YAML frontmatter, collect all tasks where `status != "done" AND status != "cancelled"`
   - Reconcile all three sources (today's existing items, previous day's list, TaskNotes):
     1. **Start with today's existing items** — keep their exact order and formatting unchanged
     2. **Carry over from previous day** — for each item in the previous day's list that is NOT already in today's list:
        - **Plain-text items** (no `[[wikilink]]`): carry forward as-is if unchecked — these are ad-hoc reminders not tracked in TaskNotes
        - **Wikilinked items**: add only if still open in TaskNotes (status != done/cancelled)
     3. **Add new TaskNotes** — open tasks not in today's list AND not in the previous day's list → append at bottom
   - Build each new todo item as: `- [ ] [[filename]] Title` (add ` 📅 YYYY-MM-DD` suffix if the task has a due date set)
   - Plain-text items are kept exactly as they appeared (no reformatting)
   - The title for wikilinked items comes from the `# H1 heading` in the task file (not the filename)

b. **Populate `## ✅ Open Tasks`**
   - If section has a `dataview` code block (migration): replace the entire code block with the merged todo list
   - Otherwise: replace the section content with the merged todo list (today's existing items are already included and ordered first from Step 4a)

c. **Carry over Notes from yesterday**: If yesterday's `## 📝 Notes` had content (from Step 2c, or extract here if Step 2 didn't run):
   - Write yesterday's notes into today's `## 📝 Notes` section
   - If today already has content, append with blank line separator
   - Do NOT add any "carried over" labels — just copy content as-is

d. **Check Dataview plugin** (`.obsidian/plugins/dataview/`)
   - If missing: warn "⚠️ Dataview plugin not found. Install from Obsidian Community Plugins for live Inbox and Meetings queries."
   - Don't block execution

e. **Inject missing Dataview sections**: If today's journal exists but is missing Inbox/Meetings dataview sections (search for `` ```dataview ``):
   - Insert between `## ✅ Open Tasks` and `## 📋 What Did I Do?`:
     - `## 📥 Inbox (Unsummarized)` (dataview query)
     - `## 🤝 Meetings` (dataview query)
   - Copy exact query blocks from `08 System/Templates/Daily Template.md`
   - NEVER overwrite or remove existing content — only add missing sections

f. **Update DASHBOARD**: Write `![[YYYY-MM-DD-dayname]]` (today's filename stem) to `_DASHBOARD.md` in the vault root, replacing entire file contents

---

### 5. Work log check (skip if has data)

- If `## 🔨 What Did I Work On?` already has table data rows: skip (work-log entries added during day via `/journal-work-log` — see `.claude/skills/journal-shared/references/work-log-format.md`)
- A section with only the table header + separator (no data rows) counts as empty
- If empty: leave empty — user fills this in manually

---

### 6. Activity pulls for target date (ALWAYS — never skip)

a. **GitHub Activity**: Read `.claude/skills/journal-shared/references/github-activity.md` for the complete procedure. Follow ALL steps using target date from Step 1.

b. **Slack Conversations**: Read `.claude/skills/journal-shared/references/slack-activity.md` for the complete procedure. Follow ALL steps using target date from Step 1.

c. **JIRA Tickets**: Read `.claude/skills/journal-shared/references/jira-activity.md` for the complete procedure. Follow ALL steps using target date from Step 1.

d. **WEB Releases**: Read `.claude/skills/journal-shared/references/jira-releases.md` for the complete procedure. Follow ALL steps using target date from Step 1.

---

### 7. Memory capture (silent)

- Review the conversation for anything memory-worthy
- If anything qualifies, create a memory file in `.claude/obsidian-memories/`
- Do this silently unless there's something significant to confirm

---

### 8. Freeze target journal (CONDITIONAL)

**Run ONLY when:** target date is NOT today (`today_is_target = false`)

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
  - Tasks completed today: [count from completed field = target date]
  - Tasks still open: [count `- [ ]` lines under ## ✅ Open Tasks heading]
  - Inbox items (unprocessed): [count from frozen 📥 Inbox table, or 0]
  - Journal frozen at: HH:MM
  ```

If target IS today: skip and warn "⚠️ Skipping freeze — this is today's active journal. Dataview blocks stay live."

---

### 9. Morning status report (CONDITIONAL)

**Run ONLY when:** morning setup ran in Step 4

a. **Check learning plan** (`.claude/learning-sessions/learning-plan.json`)
   - Find topics where `last_covered` + interval < today
   - If any due: note them. Show next item in queue.

b. **Scan tasks** — reuse Open Tasks data from Step 4

c. **Scan meetings** (`04 Meetings/*.md`)
   - Count meetings where `created` = today's date
   - Count meetings where `created` = yesterday's date

d. **Scan in-progress projects** (`06 Projects/*/README.md`)
   - Include projects where `status: active`

e. **Quick status report:**
   - Yesterday: [Complete (frozen) | Processed + frozen | No journal]
   - Today's highlight: [highlight or "not set"]
   - Open tasks: [count from todo list]
   - Open JIRA tickets: [count from Step 6c]
   - Unreleased WEB versions: [count] ([total issues]) — from Step 6d
   - Meetings today: [count]
   - Inbox (unprocessed): [count items with `status: Clipped` in `01 Inbox/`]
   - In-progress projects: [count]
   - Reviews due: [list or "none"]
   - Next in learning queue: [topic]
   - Journal: `02 Calendar/YYYY-MM-DD-dayname.md`

Keep it brief — quick orientation, not a deep dive.

## Dataview Rules

- **No trailing slashes in FROM paths**: `FROM "03 TaskNotes"` works, `FROM "03 TaskNotes/"` silently returns empty.
- **YAML null for unset fields**: Use `due:` (null), never `due: ""` (empty string). Dataview treats `""` as truthy, breaking `!due` checks.
- **Open Tasks uses todo list format** — not Dataview. Only Inbox and Meetings sections use Dataview queries.

Use bulleted lists in the journal.
