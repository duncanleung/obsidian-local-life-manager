---
name: journal-task-done
description: Mark a task as complete in 03 TaskNotes/. Use when a task is finished. Triggers on "task done", "complete task", "finished task", "done with".
model: claude-haiku-4-5-20251001
allowed-tools: Read, Edit, Glob, Bash(date:*)
argument-hint: [task name or partial match]
---

Mark a task as complete.

$ARGUMENTS

## Steps

1. **Get current date**
   - Run `date +%Y-%m-%d` to confirm today's date

2. **Find the task**
   - Search `03 TaskNotes/*.md` for a file matching the argument
   - Match strategies (in order):
     1. Exact filename match: `03 TaskNotes/{argument}.md`
     2. Partial match: filename contains the argument
     3. If multiple matches, list them and ask which one
   - If no match found, list all open tasks and ask

3. **Verify task exists and is open**
   - Read the file's frontmatter
   - If status is already `complete`, inform user and stop
   - If status is `cancelled`, warn and ask if they want to reopen-and-complete

4. **Update frontmatter**
   - Set `status: done` (changed from `complete` for Dataview consistency)
   - Add `completed: "YYYY-MM-DD"` field with today's date
   - If `completed` field doesn't exist in old tasks, add it after the `due` field

5. **Check off in today's journal**
   - Find today's journal: `02 Calendar/YYYY-MM-DD-dayname.md` (compute with `date +%Y-%m-%d-%A | tr '[:upper:]' '[:lower:]'`)
   - If journal exists: search for `- [ ] [[task-name]]` in the `## ✅ Open Tasks` section
   - If found: replace `- [ ]` with `- [x]` on that line
   - If not found: skip silently (task may not be in today's list)

6. **Confirm**
   ```
   Completed: [[task-name]]
   Was due: YYYY-MM-DD (or "no due date")
   Completed on: YYYY-MM-DD
   ```

## Examples

```
/journal-task-done buy-groceries
/journal-task-done "call with hope"
/journal-task-done groceries          # partial match
```
