# Video Note Template

Shared template for video summarization notes. Used by both `/research-youtube-catchup` and `/research-video-summarize`.

## Note Format

```markdown
---
class: Video
media: https://www.youtube.com/watch?v=VIDEO_ID
publishDate: YYYY-MM-DD
status: Summarized
duration: Xm or Xh Ym
reviewFrequency:
lastReviewedDate:
review:
aliases:
tags: ["tag1", "tag2"]
cssclasses:
archived:
---
Related:

## Summary

- **Key point** - Details
- **Key point** - Details

## Discoveries

- [[Product Name]] - brief context
- (or "None")

## Why Watch?

<Brief recommendation on whether/why to watch, target audience, length note>
```

## Summary Guidelines

- Use bullet points with **bold key concepts**
- Adjust detail level based on video length:
  - < 5 min: 3-5 bullets
  - 5-15 min: 5-8 bullets
  - 15-30 min: 8-12 bullets
  - 30+ min: Use sections/headers, 12+ bullets
- Focus on actionable takeaways, not timestamps
- "Why Watch?" should help user decide if they need to actually watch it

## Tutorial Videos

If the video is a tutorial, use step-by-step format:

```markdown
## Summary

Tutorial: [What you'll build/learn]

### Prerequisites
- Required software/tools
- Prior knowledge needed

### Steps

#### 1. [First major step]
- Specific action
- Code snippet if applicable
- Expected result

#### 2. [Continue for all steps...]

### Final Result
- What you should have
- How to verify it works

### Troubleshooting
- Common issues and solutions
```

**Tutorial goal:** Someone should be able to follow WITHOUT watching the video.

## Duration Formatting

- Under 60 min: `Xm` (e.g., `23m`)
- 60+ min: `Xh Ym` (e.g., `1h 15m`)

## Title Sanitization

Remove or replace these characters for filenames:
- `:` → ` -`
- `/` → `-`
- `|` → `-`
- `?` → (remove)
- `"` → (remove)
- `<` `>` → (remove)

## Tagging

**Use tags from the canonical list in `08 System/Tag Index.md`.**

Each channel in `references/channels.json` has a `tags` array specifying default tags. If channel not in config, infer tags from content. Format: `tags: ["tag1", "tag2"]`

## Discovery Notes

Create notes for: New tools, interesting frameworks, notable projects.
Skip: Well-known things (Python, AWS, React), generic concepts.

```markdown
---
class: Note
tags: ["tag1"]
---
Related: [[Video Title]]

## What is it?
[One sentence description]

## Why look into it?
[Why it seemed interesting]

## Links
- [URL if mentioned]
```

## Output Paths

- Video notes: `01 Inbox/Videos/{channel_folder}/{YYYY-MM-DD}-{title}.md`
- Discovery notes: `01 Inbox/{YYYY-MM-DD}-{name}.md`

**Date prefix**: Always prepend today's date (`YYYY-MM-DD-`) to filenames created in `01 Inbox/`.

## Path Handling

**CRITICAL - Never escape spaces with backslashes:**
- Use paths exactly as shown: `01 Inbox/...` (with literal spaces)
- The Write tool handles spaces correctly - backslash escaping creates literal `\` characters in directory names
- When using Bash commands, wrap paths in double quotes: `"01 Inbox/..."`

## No Transcript Handling

If transcript unavailable:
```markdown
## Summary

NO TRANSCRIPT AVAILABLE - REWATCH VIDEO TO CREATE PROPER SUMMARY

- Main topic: {title}
```
