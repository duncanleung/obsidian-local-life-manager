# Slack Activity Procedure

Single source of truth for pulling Slack conversations into a daily journal. Referenced by `journal-slack-activity`, `journal-daily-review`, and `journal-good-morning`.

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

### [Descriptive Topic Title]
- **Channel:** #channel-name
- **Participants:** Name1, Name2, Name3
- **Summary:** 1-3 sentences describing the substance and outcome of the discussion.
- **Action Items:**
  - Person: Action description
- **Thread:** [View thread](https://airvet.slack.com/archives/{channel_id}/p{message_ts_no_dot})
```

## Formatting Rules

- Omit **Action Items** line if none exist
- If no meaningful conversations: write `_No significant work conversations today._`
- Thread link format: remove the dot from message_ts (e.g., `1234567890.123456` -> `p1234567890123456`)

## Placement Rules

1. If `## ![[slack-logo.png|18]] Slack Conversations` already exists: **replace it entirely** (from heading to next `##` heading or end of file)
2. If it doesn't exist: insert after `## ![[github-logo.png|18]] GitHub Activity`, before `## 📚 What Did I Study?`
3. If neither anchor section exists: append before `## 📝 Notes` or at the end of file
