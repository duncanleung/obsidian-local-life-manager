---
name: research-clip
description: Quick-capture articles, videos, and resources to 01 Inbox/. Auto-summarizes YouTube videos with transcript. Triggers on "clip this", "save for later", "bookmark", "capture".
model: claude-sonnet-4-5-20250929
allowed-tools: Read, Write, Glob, Grep, WebFetch, Bash(date:*), Bash(grep:*), Bash(python3:*), Bash(cd:*)
argument-hint: "title" [--url URL] [--tags tag1,tag2] [--why "reason"]
---

Quick-capture a resource to Knowledge Base. Articles get a quick summary via page fetch. YouTube videos get full transcript-based summarization automatically.

$ARGUMENTS

## Steps

1. **Get current date**
   - Run `date +%Y-%m-%d` to confirm today's date
   - DO NOT assume the date - always verify

2. **Parse arguments**
   - Extract title (required) — if missing, ask the user
   - Extract `--url <url>` (optional)
   - Extract `--tags <tag1,tag2>` (optional) — comma-separated
   - Extract `--why <reason>` (optional) — reason for saving

3. **Auto-detect type from URL**
   - If URL contains `youtube.com` or `youtu.be` → **Video path** (Step 6A)
   - Everything else (or no URL) → **Article path** (Step 6B)

4. **Duplicate detection**
   - Only if `--url` was provided
   - Search for existing notes with the same URL:
     ```bash
     grep -rl "media: URL_HERE" "01 Inbox/"
     ```
   - If found: warn user with existing note path and **stop** — do NOT create a duplicate

5. **Generate filename**
   - For videos: use the video title from metadata (not the user-provided title) if available
   - For articles: use the user-provided title
   - Convert to kebab-case: lowercase, spaces to hyphens, remove special characters (`:/?"|<>+`)
   - Limit to ~80 characters
   - **Prepend today's date**: `YYYY-MM-DD-{kebab-title}.md` (use date from step 1)
   - Check if file already exists at the output path — if so, warn and ask

6A. **YouTube Video — full summarization**

   Extract video ID from URL, then fetch transcript + metadata:
   ```bash
   cd .claude/skills/research-youtube-catchup && python3 scripts/youtube_helper.py full <video_id>
   ```

   Summarize the transcript following the guidelines in `research-youtube-catchup/references/note-template.md`:
   - Bullet points with **bold key concepts**
   - Adjust detail based on length (< 5 min: 3-5 bullets, 5-15 min: 5-8, 15-30 min: 8-12, 30+ min: sections with 12+)
   - For tutorials: use step-by-step format
   - Focus on actionable takeaways, not timestamps

   Assign tags:
   - Check `research-youtube-catchup/references/channels.json` for channel's default tags
   - If channel not in config, use `--tags` if provided, otherwise infer from content
   - Tags must come from canonical list in `08 System/Tag Index.md`

   Write to `01 Inbox/Videos/Unsorted/{YYYY-MM-DD}-{filename}.md`:

   ```markdown
   ---
   class: Video
   media: https://www.youtube.com/watch?v=VIDEO_ID
   publishDate: YYYY-MM-DD
   status: Summarized
   duration: Xm
   reviewFrequency:
   lastReviewedDate:
   review:
   aliases:
   tags: ["tag1", "tag2"]
   cssclasses:
   archived:
   ---
   Related:

   ## Why Saved

   {why_text if --why provided, otherwise omit this section}

   ## Summary

   - **Key point** - Details
   - **Key point** - Details

   ## Discoveries

   - [[Product Name]] - brief context
   - (or "None")

   ## Why Watch?

   <Brief recommendation on whether/why to watch>
   ```

   If transcript unavailable:
   ```markdown
   ## Summary

   NO TRANSCRIPT AVAILABLE - REWATCH VIDEO TO CREATE PROPER SUMMARY

   - Main topic: {title}
   ```

6B. **Article / Other URL — quick capture**

   If URL provided, use `WebFetch` to retrieve the page content and extract:
   - **Key insights**: 4-8 bullets of actual conclusions, data points, takeaways
   - **Author** (if detectable)

   Summarization guidelines:
   - Capture substance, not topic labels
   - **Good**: "Multi-agent systems outperform single agents when context exceeds one prompt — Anthropic's research system beat single-agent Opus 4 by 90.2%"
   - **Bad**: "Discusses multi-agent architectures and when to use them"

   If fetch fails: fall back to placeholder `*Not yet summarized.*`

   Write to `01 Inbox/Articles/Unsorted/{YYYY-MM-DD}-{filename}.md`:

   ```markdown
   ---
   class: Article
   media: {url or ""}
   publishDate:
   status: Clipped
   author: {detected author or ""}
   reviewFrequency:
   lastReviewedDate:
   review:
   aliases:
   tags: [{tags_array}]
   cssclasses:
   archived:
   ---
   Related:

   ## Why Saved

   {why_text if --why provided, otherwise omit this section}

   ## Summary

   {AI-extracted key insights as bullet points}
   ```

7. **Confirm creation**

   For videos:
   ```
   Summarized: 01 Inbox/Videos/Unsorted/{YYYY-MM-DD}-{filename}.md
   Duration: {duration}
   Tags: {tags}
   ```

   For articles:
   ```
   Clipped: 01 Inbox/Articles/Unsorted/{YYYY-MM-DD}-{filename}.md
   URL: {url or "none"}
   Tags: {tags or "none"}
   ```

## Path Handling

**CRITICAL - Never escape spaces with backslashes:**
- Use paths exactly as shown: `01 Inbox/...` (with literal spaces)
- The Write tool handles spaces correctly - backslash escaping creates literal `\` characters in directory names
- When using Bash commands, wrap paths in double quotes: `"01 Inbox/..."`

## Examples

```
/research-clip "Terraform Best Practices"
/research-clip "React 19 Overview" --url https://youtube.com/watch?v=abc
/research-clip "RSC Deep Dive" --url https://example.com/rsc --tags react,nextjs --why "Need for Portal migration"
/research-clip "Obsidian Dataview Tutorial" --url https://youtu.be/xyz --tags obsidian,dataview --why "For task dashboard"
```

## Notes

- YouTube videos are auto-summarized from transcript — no need to run `/research-video-summarize` separately
- Articles get quick page-fetch insights with `status: Clipped`
- Videos get full transcript summary with `status: Summarized`
- Bare command with just a title (no URL) creates a minimal Article clip
- Tags should come from the canonical list in `08 System/Tag Index.md`
- If page/transcript can't be fetched, a placeholder is used — the note is still created
