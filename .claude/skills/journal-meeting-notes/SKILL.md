---
name: journal-meeting-notes
description: Capture meeting notes with attendees, decisions, and action items. Auto-creates tasks from action items. Triggers on "meeting notes", "meeting", "capture meeting", "log meeting".
model: claude-haiku-4-5-20251001
argument-hint: [meeting details, date, title, notes — any format]
allowed-tools: Read, Write, Edit, Glob, Bash(date:*)
---

Capture meeting notes in `04 Meetings/`. Two modes based on arguments.

$ARGUMENTS

## Date Detection

**CRITICAL**: The meeting date may NOT be today. Always look for a date in the arguments before defaulting to today.

1. **Scan `$ARGUMENTS` for a date** — look for any of these formats:
   - `YYYY-MM-DD` (e.g., `2026-02-12`)
   - `MM/DD/YYYY` (e.g., `02/12/2026`)
   - `MM-DD-YYYY` (e.g., `02-12-2026`)
   - Natural language: "yesterday", "last Monday", "on Feb 12"
   - Meeting invite screenshots often show dates like `02/12/2026 9:30 AM`

2. **If a date is found**: use that as the **meeting date**
3. **If no date found**: run `date +%Y-%m-%d` and use today

The **meeting date** is used for:
- The filename prefix (`YYYY-MM-DD-{title}.md`)
- The `created` field in frontmatter
- The task note filenames
- The journal entry (work log goes to the meeting date's journal, NOT today's)

## Mode Detection

- **If `$ARGUMENTS` is non-empty**: Quick Capture mode (but if args contain structured notes/details, treat as Detailed mode with pre-filled data — no need to prompt)
- **If `$ARGUMENTS` is empty**: Detailed mode (prompt for info)

---

## Quick Capture Mode (has arguments, just a title)

Fast meeting note creation — no prompts. Use when args are just a short title like "Team standup".

### Steps

1. **Detect meeting date** (see Date Detection above)

2. **Parse arguments** — extract the meeting title from `$ARGUMENTS`
   - Strip leading/trailing whitespace
   - Remove any date strings already extracted

3. **Generate filename**
   - Format: `{meeting-date}-{kebab-title}.md`
   - Kebab-case: lowercase, replace spaces with hyphens, remove special characters
   - Example: `2026-02-12-team-standup.md`

4. **Check for duplicates** — Glob `04 Meetings/{meeting-date}-{kebab-title}*.md`
   - If exists: append `-2`, `-3`, etc.

5. **Create meeting note** at `04 Meetings/{filename}`:

   ```markdown
   ---
   created: "{meeting-date}"
   meeting_type:
   participants: []
   status: captured
   tags: [meeting]
   ---

   # {Meeting Title}

   ## Participants


   ## Agenda


   ## Key Decisions


   ## Action Items


   ## Notes

   ```

6. **Confirm** — `Created meeting note: 04 Meetings/{filename}`

---

## Detailed Mode

Structured post-meeting capture. Use when:
- `$ARGUMENTS` is empty (prompt for everything)
- `$ARGUMENTS` contains rich notes/details (extract and pre-fill — only prompt for missing info)

### Steps

1. **Detect meeting date** (see Date Detection above)

2. **Extract or ask for meeting details**:
   - If `$ARGUMENTS` has rich content: parse out title, attendees, decisions, action items from the provided text. Only prompt for truly missing info.
   - If `$ARGUMENTS` is empty: prompt for each:
     - **Title**: "What was the meeting about?"
     - **Attendees**: "Who attended? (comma-separated names)"
     - **Meeting type**: "What type? (standup, 1-on-1, planning, review, brainstorm, other)" — default to `other` if skipped
     - **Key decisions**: "Any decisions made? (one per line, or skip)"
     - **Action items**: "Any action items? (one per line, or skip)"

3. **Generate filename**: `{meeting-date}-{kebab-title}.md`

4. **Create meeting note** at `04 Meetings/{filename}`:

   ```markdown
   ---
   created: "{meeting-date}"
   meeting_type: {type}
   participants: [{attendees as YAML array}]
   status: captured
   tags: [meeting]
   ---

   # {Meeting Title}

   ## Participants
   - {attendee 1}
   - {attendee 2}

   ## Agenda


   ## Key Decisions
   - {decision 1}
   - {decision 2}

   ## Action Items
   - [ ] {action item 1} → [[task-filename-1|TASK]]
   - [ ] {action item 2} → [[task-filename-2|TASK]]

   ## Notes

   ```

5. **Create task notes** — for each action item:
   - Filename: `03 TaskNotes/{meeting-date}-{kebab-action-item}.md`
   - Use the task schema:

   ```yaml
   ---
   status: open
   created: "{meeting-date}"
   due:
   completed:
   priority:
   source: "meeting: {meeting title}"
   source_type: meeting
   tags: [task]
   ---

   # {Action Item Title}

   ## Notes
   From meeting: [[04 Meetings/{meeting-filename}|{meeting title}]]
   ```

   - Link each task back in the meeting note's Action Items section

6. **Ensure `## 🤝 Meetings` section exists in journal** — compute the meeting date's filename stem with `date -j -f %Y-%m-%d "{meeting-date}" +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'`, then read `02 Calendar/{meeting-date-dayname}.md`:
   - If journal doesn't exist: skip steps 6-7 entirely (don't create a journal for this)
   - If `## 🤝 Meetings` section is **missing**: inject it between `## 📥 Inbox (Unsummarized)` and `## 📋 What Did I Do?`
     - If the journal is **frozen** (no dataview blocks, or `status: done`): insert a static table with the meeting data
       ```markdown
       ## 🤝 Meetings

       | File | Type | Attendees |
       |------|------|-----------|
       | [[{meeting-filename-without-ext}]] | {meeting_type} | {comma-separated participants} |
       ```
     - If the journal is **live** (has dataview blocks): insert the Dataview query
       ```markdown
       ## 🤝 Meetings
       ```dataview
       TABLE meeting_type AS "Type", participants AS "Attendees"
       FROM "04 Meetings"
       WHERE created >= date(today) - dur(7 days)
       SORT created DESC
       ```
       ```
   - If `## 🤝 Meetings` section **already exists** and is a frozen table: append the new meeting row to the existing table
   - If `## 🤝 Meetings` section **already exists** and is a Dataview query: do nothing (query auto-includes new meetings)

7. **Add work log entry** — in the same journal (`02 Calendar/{meeting-date-dayname}.md`):
   - Add a table row under `## 🔨 What Did I Work On?` (see `.claude/skills/journal-shared/references/work-log-format.md`):
   - `| 🗣️ | — | {title} | Discussed {brief summary} | [[04 Meetings/{filename}|meeting]] |`
   - If the table doesn't exist yet (section is empty), create the header first:
     ```
     | Status | Project | Work Item | Description | Links |
     |--------|---------|-----------|-------------|-------|
     ```
   - Then append the data row

8. **Summary** — show what was created:
   - Meeting note path
   - Number of tasks created (with filenames)
   - Work log entry added (yes/no)

## Important

- Use YAML null (bare key, no value) for optional fields — NEVER empty strings `""`
- Participants array: `["Alice", "Bob"]` in frontmatter, bullet list in body
- Kebab-case filenames: lowercase, hyphens, no special characters
- The meeting date drives ALL filenames and journal entries — never assume today
- Do NOT commit to git
- Stop after creating files — no follow-up actions
