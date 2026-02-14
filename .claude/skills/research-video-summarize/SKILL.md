---
name: research-video-summarize
description: Summarize a single YouTube video and create a note. Use when user shares a video URL or asks to summarize a specific video. Triggers on "summarize this video", "video summary", YouTube URLs.
allowed-tools: Bash, Read, Write, Edit, Glob
---

Summarize a single YouTube video and create a note.

> **Note:** `/research-clip` now auto-summarizes YouTube videos when it detects a YouTube URL. Use this skill only if you need the `folder` parameter to place the note in a specific channel subfolder.

## Usage

```
/video-summarize <youtube-url> [folder]
```

- `youtube-url`: Full YouTube URL or video ID
- `folder` (optional): Subfolder under `Videos/` (e.g., "Atlassian", "Ali Abdaal"). If not specified, uses channel name.

## Workflow

1. **Extract video ID** from URL (supports youtube.com/watch?v=, youtu.be/, and bare IDs)

2. **Fetch video metadata** using the youtube-catchup helper script:
   ```bash
   python3 .claude/skills/research-youtube-catchup/scripts/youtube_helper.py full <video_id>
   ```

3. **Review the transcript** and create a summary with:
   - Bullet point summary of key concepts
   - "Why Watch?" section with recommendation

4. **Determine folder:**
   - Use provided folder if specified
   - Otherwise use channel name from metadata
   - Create folder if it doesn't exist

5. **Create note** at `01 Inbox/Videos/<folder>/<title>.md`

6. **Assign tags:**
   - Check `research-youtube-catchup/references/channels.json` for channel's default tags
   - If channel not in config, infer tags from the canonical list in `08 System/Tag Index.md`

## Tagging

**Use tags from the canonical list in `08 System/Tag Index.md`.**

Common channel → tag mappings are in `.claude/skills/research-youtube-catchup/references/channels.json`. If the channel exists there, use its `tags` array. Otherwise, infer appropriate tags based on content.

Format tags as: `tags: ["tag1", "tag2"]`

## Note Format

See `../research-youtube-catchup/references/note-template.md` for the full video note template, summary guidelines, duration formatting, title sanitization rules, and discovery note format.

## Examples

```
/video-summarize https://www.youtube.com/watch?v=abc123
/video-summarize abc123 "Tech Talks"
/video-summarize https://youtu.be/xyz789 Atlassian
```

## Path Handling

**CRITICAL - Never escape spaces with backslashes:**
- Use paths exactly as shown: `01 Inbox/...` (with literal spaces)
- The Write tool handles spaces correctly - backslash escaping creates literal `\` characters in directory names
- When using Bash commands, wrap paths in double quotes: `"01 Inbox/..."`

## Notes

- Reuses `youtube_helper.py` from research-youtube-catchup skill (no duplicate code)
- If transcript unavailable, note this in the summary and summarize from description/title only
- Check if note already exists before creating (search by video ID in media field)
