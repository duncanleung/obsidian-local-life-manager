# Slack Activity Procedure

Single source of truth for pulling Slack conversations into a daily journal. Referenced by `journal-slack-activity` and `journal-daily-review`.

## Configuration

- **Slack User ID**: `U0163GG5831` (Duncan Leung)
- **Slack workspace URL**: `airvet.slack.com`
- **Section heading**: `## ![[slack-logo.png|18]] Slack Conversations`

## Search Queries

Run both queries for the target date:

- `from:<@U0163GG5831> on:{target-date}` (messages sent)
- `to:<@U0163GG5831> on:{target-date}` (messages received)
- Use `slack_search_public_and_private`, sort by timestamp ascending, limit 20
- If cursor returned, paginate up to 3 pages total

## Thread Grouping

Group messages into conversations by thread:
- Same `channel_id` + parent `message_ts` = same conversation
- Standalone messages (no thread) = individual conversations

## Channel Grouping

After thread grouping, further group conversations by `channel_id`:
- Resolve channel names via the search results or `slack_read_channel`
- Sort channels by number of conversations DESC (most active channel first)
- Within each channel, keep conversations in chronological order

## Thread Context

For each unique thread, call `slack_read_thread` (concise format).

## User Resolution

- Batch unique user IDs from all conversations
- Call `slack_read_user_profile` for each (skip U0163GG5831 = Duncan Leung)
- Cache results to avoid duplicate lookups

## Filtering Rules

**ONLY include work conversations:**
- INCLUDE: technical discussions, project planning, decisions, PR reviews, incidents, status updates, meeting follow-ups, debugging, architecture discussions
- EXCLUDE: social banter, single emoji/thanks messages, bot notifications without discussion, personal DM chatter, GIFs/memes only

## Output Format

```markdown
## ![[slack-logo.png|18]] Slack Conversations

### #channel-name — 3 conversations

| Topic | Summary | Action Items |
|-------|---------|--------------|
| [Descriptive Topic Title](https://airvet.slack.com/archives/{channel_id}/p{message_ts_no_dot}) | • Key point one<br>• Key point two<br>• Outcome or resolution | • Person: Action description |
| [Another Topic](https://airvet.slack.com/archives/{channel_id}/p{message_ts_no_dot}) | • Summary bullet one<br>• Summary bullet two | — |

### #another-channel — 1 conversation

| Topic | Summary | Action Items |
|-------|---------|--------------|
| [Topic Title](https://airvet.slack.com/archives/{channel_id}/p{message_ts_no_dot}) | • Key point one<br>• Key point two | • Person: Action description |
```

## Formatting Rules

- **Channel header**: `### #{channel-name} — {N} conversation(s)` — sort channels by conversation count DESC
- **Table columns**: `Topic | Summary | Action Items`
  - **Topic**: `[Descriptive Title](thread-link)` — the topic title itself is the clickable thread link
  - **Summary**: 2-4 bullet points using `•` separated by `<br>`. Capture key substance, decisions, and outcomes
  - **Action Items**: Bullet points using `•` separated by `<br>`. Format: `• Person: Action description`. If no actions exist, use `—`
- **Thread link format**: remove the dot from message_ts (e.g., `1234567890.123456` -> `p1234567890123456`)
- If no meaningful conversations: write `_No significant work conversations today._`

## Placement Rules

1. If `## ![[slack-logo.png|18]] Slack Conversations` already exists: **replace it entirely** (from heading to next `##` heading or end of file)
2. If it doesn't exist: insert after `## ![[github-logo.png|18]] GitHub Activity`, before `## 📚 What Did I Study?`
3. If neither anchor section exists: append before `## 📝 Notes` or at the end of file
