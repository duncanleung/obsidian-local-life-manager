# Journal Creation Format

Canonical format for creating a new daily journal (`02 Calendar/YYYY-MM-DD-dayname.md`, e.g., `2026-02-14-saturday.md`). Referenced by all journal skills that may need to create a journal as a side effect.

## Filename Convention

- **Format**: `YYYY-MM-DD-dayname.md` (lowercase full day name, e.g., `monday`, `tuesday`)
- **Wikilinks**: `![[2026-02-14-saturday]]` (filename without `.md`)

### Date-to-Filename Commands (macOS)

| Use case | Command |
|----------|---------|
| Today | `date +%Y-%m-%d-%A \| tr '[:upper:]' '[:lower:]'` |
| Yesterday | `date -v-1d +%Y-%m-%d-%A \| tr '[:upper:]' '[:lower:]'` |
| N days ago | `date -v-Nd +%Y-%m-%d-%A \| tr '[:upper:]' '[:lower:]'` |
| From date string | `date -j -f %Y-%m-%d "2026-02-12" +%Y-%m-%d-%A \| tr '[:upper:]' '[:lower:]'` |

## Source of Truth

**`08 System/Templates/Daily Template.md`** is the single source of truth for the daily journal section structure, ordering, and default content (including Dataview queries).

When creating a new journal, read that template and resolve Templater syntax as described below.

## Templater Resolution

Replace Templater expressions with actual values:

| Template Expression | Resolves To | Example |
|---|---|---|
| `<% tp.date.now("YYYY-MM-DD") %>` | Target date in ISO format | `2026-02-14` |
| `<% tp.date.now("MMMM DD, YYYY - dddd") %>` | Full date with day of week | `February 14, 2026 - Saturday` |

## Frontmatter Requirements

```yaml
---
created: YYYY-MM-DD
modified: YYYY-MM-DD
tags: [daily-note]
status: in-progress
---
```

Both `tags` and `status` are required.

## Heading Format

- H1: `# February 14, 2026 - Saturday` (full date with day of week)
- H2 sections use emoji prefixes or Obsidian image embeds as defined in the Daily Template
- All sections are empty by default — populated manually or via skills
