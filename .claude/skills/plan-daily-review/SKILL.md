---
name: plan-daily-review
description: Complete daily journal review. Use at end of day or next morning to fill in journal sections, review highlights, and plan tomorrow. Triggers on "daily review", "end of day", "journal review", "what did I do today".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(gh:*), Read, Write, Edit, Glob
---

Run the Daily Review Workflow. Keep it conversational - ask one thing at a time.

$ARGUMENTS

## Steps

1. **Determine target journal**
   - If `$ARGUMENTS` contains a file path (e.g., `02 Calendar/2026-02-12.md`): use that file directly and extract the date from the filename
   - If `$ARGUMENTS` contains a date (e.g., `2026-02-12`): use `02 Calendar/YYYY-MM-DD.md`
   - If no arguments: run `date +%Y-%m-%d` to get today's date
   - The **target date** is the date being reviewed (from the filename or argument)
   - Always run `date +%Y-%m-%d` to know what today's date is (needed for "tomorrow" references)

2. **Journal Entry Setup**
   - Check if the target journal entry exists (`02 Calendar/YYYY-MM-DD.md`)
   - Create from template if not (see `references/template.md`)
   - If morning reviewing yesterday: use yesterday's date

3. **What Did I Work On?**
   - Pull GitHub commits for the **target date**: `gh search commits --author=duncanleung --committer-date={target-date}`
   - Summarize into meaningful bullets (not raw commit messages)
   - Every bullet MUST start with an emoji status verb. Pick the best match:
     - ✅ Completed — finished a task, issue, or milestone
     - 🚧 Progressed — made progress but not done yet
     - 🚀 Shipped — deployed, released, or merged to production
     - 🐛 Fixed — resolved a bug or defect
     - 🔨 Built — created something new (feature, component, tool)
     - 🔧 Updated — modified, enhanced, or improved existing work
     - 📐 Designed — created specs, plans, architecture, or UI designs
     - 📝 Documented — wrote docs, READMEs, ADRs, or notes
     - 🔍 Investigated — researched, debugged, or explored a topic
     - 🧪 Tested — wrote or ran tests, validated behavior
     - ♻️ Refactored — restructured code without changing behavior
     - 🔄 Reviewed — code review, PR review, or feedback
   - Format: `- {emoji} {Verb} {description} — {optional links}`
   - Ask: "Any other technical work? (studying, courses, side projects not on GitHub)"

4. **What Did I Do?**
   - Ask: "How about personal stuff? (errands, social, health, appointments, etc.)"

5. **Daily Highlight Check**
   - Review the day's highlight if set
   - Ask: "Did you accomplish your highlight? Want to carry it to tomorrow?"

6. **Quick Inbox Scan** (offer, don't force)
   - "Want me to check your inbox for anything to quickly process?"

7. **Tomorrow's Highlight** (offer, don't force)
   - "Do you know what tomorrow's focus should be?"

8. **Memory Capture Check**
   - Review the conversation for anything memory-worthy:
     - New preferences expressed
     - Corrections to how you understood something
     - Life/job updates
     - Workflow insights
     - Project decisions
   - If anything qualifies, create a memory file in `.claude/obsidian-memories/`
   - Check if `about-me.md` needs updating (job status, current focus, etc.)
   - Do this silently unless there's something significant to confirm

9. **Freeze Journal (EOD Snapshot)**
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
     ## Day Summary
     - Tasks completed today: [count from completed field = today's date]
     - Tasks still open: [count from frozen Open Tasks table]
     - Knowledge clips saved: [count from frozen Clipped table, or 0]
     - Journal frozen at: HH:MM
     ```

Use bulleted lists in the journal.
