# Meeting Notes Schema

## Frontmatter

```yaml
---
created: "YYYY-MM-DD"          # Date of the meeting (always quoted)
meeting_type:                   # standup | 1-on-1 | planning | review | brainstorm | other | null
participants: []                # YAML array: ["Alice", "Bob", "Charlie"]
status: captured                # captured | summarized
tags: [meeting]                 # Always includes "meeting" tag
---
```

### Field Details

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `created` | string | Yes | `"YYYY-MM-DD"` |
| `meeting_type` | string/null | No | `standup`, `1-on-1`, `planning`, `review`, `brainstorm`, `other` |
| `participants` | array | No | `["Name 1", "Name 2"]` |
| `status` | string | Yes | `captured`, `summarized` |
| `tags` | array | Yes | Always `[meeting]` |

### YAML Null Convention

Use YAML null (bare key with no value) for unset optional fields:

```yaml
meeting_type:        # CORRECT — null
meeting_type: ""     # WRONG — empty string breaks Dataview
```

## File Location

`04 Meetings/{filename}.md`

## Filename Convention

`YYYY-MM-DD-{kebab-title}.md`

- Date prefix for chronological sorting
- Kebab-case: lowercase, hyphens for spaces, no special characters
- Examples:
  - `2026-02-13-team-standup.md`
  - `2026-02-13-1-on-1-with-sarah.md`
  - `2026-02-13-sprint-planning.md`

## Note Structure

```markdown
# {Meeting Title}

## Participants
- {Person 1}
- {Person 2}

## Agenda
- {topic 1}
- {topic 2}

## Key Decisions
- {decision 1}
- {decision 2}

## Action Items
- [ ] {action item} → [[task-filename|TASK]]

## Notes
{free-form notes}
```

## Action Item → Task Flow

When action items are captured in Detailed mode:

1. Each action item creates a task note in `03 TaskNotes/`
2. Task frontmatter includes:
   - `source: "meeting: {meeting title}"`
   - `source_type: meeting`
3. Task body links back to the meeting note:
   - `From meeting: [[04 Meetings/{filename}|{meeting title}]]`
4. Meeting note links to the task:
   - `- [ ] {action item} → [[task-filename|TASK]]`

## Dataview Query (Daily Journal)

```dataview
TABLE meeting_type AS "Type", participants AS "Attendees"
FROM "04 Meetings"
WHERE created >= date(today) - dur(7 days)
SORT created DESC
```

Shows meetings from the past 7 days. Compatible with the EOD freeze algorithm (`dur()` resolves to a concrete date at freeze time).
