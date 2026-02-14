---
name: plan-daily-review
description: Complete daily journal review. Use at end of day or next morning to fill in journal sections, review highlights, and plan tomorrow. Triggers on "daily review", "end of day", "journal review", "what did I do today".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(gh:*), Bash(date:*), Read, Write, Edit, Glob, mcp__claude_ai_Slack__slack_search_public_and_private, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_read_user_profile
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
   - If already has entries: skip (work-log entries are added during the day via `/plan-work-log`)
   - If empty: leave empty — user fills this in manually

4. **GitHub Activity** (ALWAYS runs — never skip this step)
   - This is a dedicated section for all GitHub activity on the target date
   - **Always populate this section**, regardless of whether other sections have content
   - Run ALL of these `gh` queries for the **target date**:

   ```bash
   # 1. Commits authored (extract: org/repo, full SHA, commit title)
   gh search commits --author=duncanleung --committer-date={target-date} --limit=50

   # 2. PRs merged (extract: org/repo, PR number, title)
   gh search prs --author=duncanleung --merged-at={target-date} --limit=20

   # 3. PRs created/opened (extract: org/repo, PR number, title)
   gh search prs --author=duncanleung --created={target-date} --limit=20

   # 4. PRs commented on (extract: org/repo, PR number, title)
   gh api search/issues --method GET -f q="commenter:duncanleung type:pr updated:{target-date}" --jq '.items[] | "\(.repository_url | split("/")[-2:] | join("/")) #\(.number) \(.title) \(.state) \(.html_url)"'

   # 5. PRs involved in (reviews requested, approvals, etc.)
   gh api search/issues --method GET -f q="involves:duncanleung type:pr updated:{target-date}" --jq '.items[] | "\(.repository_url | split("/")[-2:] | join("/")) #\(.number) \(.title) \(.state) \(.html_url)"'
   ```

   - **Deduplicate** PRs that appear in multiple queries (by repo/number)
   - **Build URLs** for every entry:
     - **Commits:** `https://github.com/{org}/{repo}/commit/{full-sha}`
     - **Pull Requests:** `https://github.com/{org}/{repo}/pull/{number}`
     - **Reviews & Comments:** `https://github.com/{org}/{repo}/pull/{number}`
   - **Place the section** in the journal:
     - If `## ![[github-logo.png|18]] GitHub Activity` already exists: **replace it entirely** (from `## ![[github-logo.png|18]] GitHub Activity` to the next `##` heading or end of file)
     - If it doesn't exist: insert after `## 🔨 What Did I Work On?`, before `## 📚 What Did I Study?`
     - If neither anchor section exists: append before `## 📝 Notes` or at the end of file
   - Every entry MUST include a clickable web link. Format the section as:

   ```markdown
   ## ![[github-logo.png|18]] GitHub Activity

   ### Commits
   - **{org/repo}**
     - [`{short-sha}`](https://github.com/{org}/{repo}/commit/{full-sha}) {commit message title}
   - **{org/repo}**
     - [`{short-sha}`](https://github.com/{org}/{repo}/commit/{full-sha}) {commit message title}

   ### Pull Requests
   - 🟣 Merged — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) {title}
   - 🟢 Opened — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) {title}

   ### Reviews & Comments
   - 💬 Commented on [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) — {title}
   - 👀 Review requested — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) — {title}
   ```

   - Group commits by repo
   - Omit any subsection (Commits, Pull Requests, Reviews & Comments) that has zero items
   - If NO GitHub activity at all: write `_No GitHub activity._` under the `## ![[github-logo.png|18]] GitHub Activity` heading

5. **Slack Conversations** (ALWAYS runs — never skip this step)
   - Search for target date's Slack messages (run both queries):
     - `from:<@U0163GG5831> on:{target-date}` (messages sent)
     - `to:<@U0163GG5831> on:{target-date}` (messages received)
     - Use `slack_search_public_and_private`, sort by timestamp ascending, limit 20
     - If cursor returned, paginate up to 3 pages total
   - Group messages into conversations by thread:
     - Same channel_id + parent message_ts = same conversation
     - Standalone messages (no thread) = individual conversations
   - For each unique thread, read full context with `slack_read_thread` (concise format)
   - Resolve user IDs to display names:
     - Batch unique user IDs from all conversations
     - Call `slack_read_user_profile` for each (skip U0163GG5831 = Duncan Leung)
     - Cache results to avoid duplicate lookups
   - **Filter: ONLY include work conversations**
     - INCLUDE: technical discussions, project planning, decisions, PR reviews, incidents, status updates, meeting follow-ups, debugging, architecture discussions
     - EXCLUDE: social banter, single emoji/thanks messages, bot notifications without discussion, personal DM chatter, GIFs/memes only
   - **Place the section** in the journal:
     - If `## ![[slack-logo.png|18]] Slack Conversations` exists: replace it entirely (from heading to next `##` or EOF)
     - If doesn't exist: insert after `## ![[github-logo.png|18]] GitHub Activity`, before `## 📚 What Did I Study?`
     - If neither anchor exists: append before `## 📝 Notes` or at EOF
   - Write each meaningful conversation:
     ```markdown
     ## ![[slack-logo.png|18]] Slack Conversations

     ### [Descriptive Topic Title]
     - **Channel:** #channel-name
     - **Participants:** Name1, Name2, Name3
     - **Summary:** 1-3 sentences describing the substance and outcome of the discussion.
     - **Action Items:**
       - Person: Action description
     - **Thread:** [View thread](https://airvet.slack.com/archives/{channel_id}/p{message_ts_no_dot})
     ```
   - Omit **Action Items** line if none exist
   - If no meaningful conversations: write `_No significant work conversations today._`
   - Thread link format: remove the dot from message_ts (e.g., `1234567890.123456` → `p1234567890123456`)

6. **Memory Capture Check** (silent)
   - Review the conversation for anything memory-worthy
   - If anything qualifies, create a memory file in `.claude/obsidian-memories/`
   - Do this silently unless there's something significant to confirm

7. **Freeze Journal (EOD Snapshot)**
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
