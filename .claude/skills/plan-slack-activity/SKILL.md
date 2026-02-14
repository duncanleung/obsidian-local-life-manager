---
name: plan-slack-activity
description: Pull Slack conversations into today's journal. Safe to run anytime — does NOT freeze dataviews. Triggers on "slack activity", "pull slack", "slack conversations", "what did I discuss".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(date:*), Read, Write, Edit, Glob, mcp__claude_ai_Slack__slack_search_public_and_private, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_read_user_profile
---

Pull Slack conversations into the daily journal. This is safe to run throughout the day — it only touches the `## ![[slack-logo.png|18]] Slack Conversations` section and does NOT freeze dataview blocks.

$ARGUMENTS

## Steps

1. **Determine target journal**
   - If `$ARGUMENTS` contains a file path (e.g., `02 Calendar/2026-02-12.md`): use that file directly and extract the date from the filename
   - If `$ARGUMENTS` contains a date (e.g., `2026-02-12`): use `02 Calendar/YYYY-MM-DD.md`
   - If no arguments: run `date +%Y-%m-%d` to get today's date, use `02 Calendar/YYYY-MM-DD.md`

2. **Read the journal file**
   - If it doesn't exist: create it with the standard format (see bottom of this file)
   - Note whether a `## ![[slack-logo.png|18]] Slack Conversations` section already exists (it will be replaced)

3. **Search Slack** — run both queries for the target date:

   - `from:<@U0163GG5831> on:{target-date}` (messages sent)
   - `to:<@U0163GG5831> on:{target-date}` (messages received)
   - Use `slack_search_public_and_private`, sort by timestamp ascending, limit 20
   - If cursor returned, paginate up to 3 pages total

4. **Group messages into conversations** by thread:
   - Same channel_id + parent message_ts = same conversation
   - Standalone messages (no thread) = individual conversations

5. **Read full thread context** — for each unique thread, call `slack_read_thread` (concise format)

6. **Resolve user IDs to display names**:
   - Batch unique user IDs from all conversations
   - Call `slack_read_user_profile` for each (skip U0163GG5831 = Duncan Leung)
   - Cache results to avoid duplicate lookups

7. **Filter: ONLY include work conversations**
   - INCLUDE: technical discussions, project planning, decisions, PR reviews, incidents, status updates, meeting follow-ups, debugging, architecture discussions
   - EXCLUDE: social banter, single emoji/thanks messages, bot notifications without discussion, personal DM chatter, GIFs/memes only

8. **Format and insert** the `## ![[slack-logo.png|18]] Slack Conversations` section:

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

9. **Place the section** in the journal:
   - If `## ![[slack-logo.png|18]] Slack Conversations` already exists: **replace it entirely** (from `## ![[slack-logo.png|18]] Slack Conversations` to the next `##` heading or end of file)
   - If it doesn't exist: insert after `## ![[github-logo.png|18]] GitHub Activity`, before `## 📚 What Did I Study?`
   - If neither anchor section exists: append before `## 📝 Notes` or at the end

## Important

- Do NOT freeze dataview blocks — that's only for `/plan-daily-review`
- Do NOT touch any other sections (📋 What Did I Do?, ⭐ Highlight, ![[github-logo.png|18]] GitHub Activity, etc.)
- Do NOT commit to git
- Stop after updating the Slack Conversations section

## Journal Format

If creating a new journal:

```markdown
---
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [daily-note]
---

# {Month} {Day}, {Year} - {DayOfWeek}

## ⭐ Highlight


## 📋 What Did I Do?


## 🔨 What Did I Work On?


## ![[github-logo.png|18]] GitHub Activity


## ![[slack-logo.png|18]] Slack Conversations


## 📚 What Did I Study?


## 📝 Notes

```
