# Daily Journal Template

## Expected Structure

The daily journal template should include sections with Dataview query blocks that will be frozen during the daily review process.

### Common Dataview Sections

The following sections are expected to contain `dataview` code blocks:

**Overdue Tasks**
```dataview
TABLE status AS "Status", due AS "Due", source_type AS "Source"
FROM "03 TaskNotes/"
WHERE status != "done" AND status != "cancelled" AND due < date(today)
SORT due ASC
```

**Due Today**
```dataview
TABLE status AS "Status", due AS "Due", source_type AS "Source"
FROM "03 TaskNotes/"
WHERE status != "done" AND status != "cancelled" AND due = date(today)
```

**Open Tasks**
```dataview
TABLE status AS "Status", due AS "Due", source_type AS "Source"
FROM "03 TaskNotes/"
WHERE status != "done" AND status != "cancelled" AND (due > date(today) OR !due)
SORT choice(due, due, date("9999-12-31")) ASC
```

**Clipped (Unsummarized)**
```dataview
TABLE class AS "Class", url AS "URL"
FROM "07 Knowledge Base/Capture/"
WHERE status = "Clipped"
SORT file.mtime DESC
```

### Day Summary Section

The freeze process automatically appends a Day Summary section:

```markdown
## Day Summary
- Tasks completed today: [count]
- Tasks still open: [count]
- Knowledge clips saved: [count]
- Journal frozen at: HH:MM
```

This section is auto-populated during the freeze step and should not be manually added to the template.

## Notes

- The exact template structure may vary based on user preferences
- The freeze process works with any journal that contains `dataview` code blocks
- If no dataview blocks are found, the journal is considered already frozen