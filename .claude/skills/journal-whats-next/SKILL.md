---
name: journal-whats-next
description: "Personal triage — overdue tasks, due-today items, unfrozen journals, inbox backlog, meeting follow-ups, learning reviews. Triggers on \"what's next\", \"what should I do\", \"prioritize\", \"what now\", \"triage\"."
model: claude-haiku-4-5-20251001
allowed-tools: Read, Glob, Bash(date:*)
---

Quick personal triage — what needs your attention right now.

This is READ-ONLY. Never modify any files.

## Steps

### 1. Get today's date

```bash
date +%Y-%m-%d   # $TODAY (for date comparisons)
date +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'   # $TODAY_STEM (for filename, e.g., 2026-02-14-saturday)
date -v-1d +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'  # yesterday stem
date -v-2d +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'  # 2 days ago stem
date -v-3d +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'  # 3 days ago stem
```

### 2. Scan overdue tasks (`03 TaskNotes/*.md`)

- Glob all files in `03 TaskNotes/*.md`
- Read each file's YAML frontmatter
- Collect tasks where: `status` = `open` AND `due` is set (not YAML null) AND `due` < `$TODAY`
- Sort by `due` ASC (oldest overdue first), then by `priority` (high > medium > low > unset)
- Record: count, list of (filename, title from H1, due date, days overdue, priority)

### 3. Scan tasks due today (`03 TaskNotes/*.md`)

- From the same scan: collect tasks where `status` = `open` AND `due` = `$TODAY`
- Sort by `priority` (high first)
- Record: count, list of (filename, title, priority)

### 4. Check incomplete journals (`02 Calendar/`)

- Check the past 3 days (yesterday, 2 days ago, 3 days ago — NOT today, since today is in progress)
- For each date, check if `02 Calendar/YYYY-MM-DD-dayname.md` exists (use the filename stem computed in step 1)
- If exists, check if the file contains ` ```dataview ` — if yes, the journal is unfrozen and needs `/journal-daily-review`
- Also check frontmatter: `status: done` means frozen (skip), anything else means unfrozen
- Record: list of unfrozen past journal dates

### 5. Count unprocessed inbox (`01 Inbox/**/*.md`)

- Glob `01 Inbox/**/*.md` (recursive — includes subdirectories like Articles/, Videos/)
- Read each file's YAML frontmatter
- Count files where `status` = `Clipped`
- Record: count only (no individual listing for speed)

### 6. Check meetings needing follow-up (`04 Meetings/*.md`)

- Glob `04 Meetings/*.md`
- Read each file's YAML frontmatter
- For meetings where `status` = `captured`: read the body and count `- [ ]` lines (unchecked action items)
- Only include meetings that have at least one unchecked action item
- Record: count of meetings, total unchecked items, list of (filename, title, created date, unchecked count)

### 7. Check learning reviews due

- Try to read `.claude/learning-sessions/learning-plan.json`
- If file doesn't exist: skip, note "No learning plan configured"
- If exists: parse JSON, find topics where `last_covered` + review interval < `$TODAY`
- Record: list of overdue topics with days overdue
- Also note: next topic in queue (if any)

### 8. Check today's highlight (`02 Calendar/$TODAY_STEM.md`)

- Read today's journal if it exists
- Find the `## Highlight` section (may have emoji prefix like `## ⭐ Highlight`)
- If it has content after the heading: record the highlight text
- If empty or journal doesn't exist: record "not set"

## Output Format

```
# Personal Triage

## Needs Attention

🚨 **Overdue Tasks** (N)
- [[task-name]] — due YYYY-MM-DD (N days overdue) [HIGH]
- [[task-name]] — due YYYY-MM-DD (N days overdue)

📅 **Due Today** (N)
- [[task-name]] [HIGH]
- [[task-name]]

📓 **Unfrozen Journals** (N)
- YYYY-MM-DD — run `/journal-daily-review YYYY-MM-DD`

🤝 **Meetings: Open Action Items** (N meetings, M items)
- [[meeting-name]] — N unchecked items (YYYY-MM-DD)

📥 **Inbox Backlog**: N items unprocessed

📚 **Learning Reviews Due**: [topic] (N days overdue)

## Today's Focus
⭐ Highlight: [highlight text or "not set"]

## Recommended Next Action
[Single actionable sentence from prioritization framework]
```

- **Skip categories with zero items** — don't show empty sections
- If everything is clear: "All clear! Nothing overdue, inbox clean, journals frozen."

## Prioritization Framework

The "Recommended Next Action" picks the highest-priority item:

```
1. Overdue tasks with priority: high (past deadline, high stakes)
2. Tasks due today with priority: high
3. Other overdue tasks (any priority, sorted by days overdue)
4. Other tasks due today
5. Unfrozen past journals (data integrity — freeze before data becomes stale)
6. Meetings with unchecked action items (commitments made to others)
7. Overdue learning reviews (spaced repetition window closing)
8. Today's highlight (if set — user's declared focus)
9. Inbox backlog (if > 5 items, suggest /ops-process-inbox)
```

## Important

- This is READ-ONLY — never modify any files
- Keep it FAST — Haiku model, scan quickly, don't deep-dive into file contents beyond frontmatter + action items
- Task `due` dates may be YAML null (bare `due:` with no value) — treat as "no due date", skip from overdue/due-today
- If `03 TaskNotes/` has no files, skip task sections silently
- If `04 Meetings/` has no files, skip meeting section silently
- If learning plan file is missing, skip with note
- Do NOT scan `06 Projects/` — project concerns belong in `/project-status-all` and `/project-next-step`
- Do NOT recommend project work — this is personal triage only
- Do NOT commit to git
