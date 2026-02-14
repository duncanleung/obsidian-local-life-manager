---
name: research-clip-read
description: Process a clipped resource — extract key insights, populate summary, and mark as Read. Accepts pasted JSON content for blocked sources (Reddit, paywalled sites). Triggers on "clip read", "mark read", "read clip", "process clip".
allowed-tools: Read, Edit, Write, Glob, Grep, Bash(date:*), WebFetch
argument-hint: [clip name] [--content '{...}'] [--skip-fetch]
---

Process a clipped item: extract key insights from the source, write a summary, and mark as Read.

$ARGUMENTS

## Steps

1. **Get current date**
   - Run `date +%Y-%m-%d` to confirm today's date

2. **Find the clip**
   - Search both directories recursively:
     - `01 Inbox/Videos/**/*.md`
     - `01 Inbox/Articles/**/*.md`
   - Match strategies (in order):
     1. Exact filename match (with or without `.md`)
     2. Partial match: filename contains the argument (case-insensitive)
     3. If multiple matches, list them and ask which one
   - If no clip name provided: find all files with `status: Clipped` and list them for the user to pick

3. **Verify status**
   - Read the file's frontmatter
   - If `status` is already `Read`: inform user "Already marked as Read" and stop
   - If `status` is `Summarized`: inform user "Already summarized — no change needed" and stop
   - If `status` is not `Clipped`: warn and ask if they want to proceed

4. **Get the source content** (pick the first that applies)

   **a) User provided `--content`:**
   - Use the JSON/text passed directly in the arguments
   - This is the primary method for blocked sources (Reddit, paywalled sites, etc.)
   - The content can be JSON (from browser dev tools, extensions, etc.) or plain text

   **b) Try fetching the URL:**
   - If the clip has a `media:` URL and `--skip-fetch` was NOT passed
   - Use WebFetch to retrieve the page content
   - If WebFetch fails (blocked, paywall, etc.): tell the user and ask them to provide content via `--content`

   **c) No content available:**
   - If no `--content` and no fetchable URL (or fetch failed): ask the user to provide the content
   - Suggest: "Paste the article content, or re-run with `--content '{...}'`"
   - Do NOT proceed without content — the whole point is extracting insights

5. **Extract key insights and write summary**

   Analyze the content and update the note's `## Summary` section with:

   - **4-8 substantive bullet points** capturing the key takeaways
   - Focus on actionable insights, conclusions, data points — NOT topic labels
   - Use **bold key concepts** at the start of each bullet
   - If the source has comments/discussion (e.g., Reddit), capture the best community insights too

   **Good summary bullets:**
   - **Claude Code + Obsidian workflow** — Use CLAUDE.md to define vault structure, custom slash commands for journaling, task management, and knowledge capture
   - **Persistent memory via markdown** — Store context in `.claude/` directory files that persist across sessions

   **Bad summary bullets:**
   - Discusses using Claude with Obsidian
   - Various approaches are mentioned

   Also populate:
   - `## Discoveries` — any tools, products, frameworks worth exploring (or "None")
   - `## Why Read?` — one sentence on whether it was worth reading

6. **Update frontmatter**
   - Change `status: Clipped` → `status: Read`
   - If `modified:` field exists, update to today's date
   - If no `modified:` field, add `modified: "YYYY-MM-DD"` after the `status:` line

7. **Update the note body**
   - Replace the placeholder `## Summary` content with the extracted insights
   - Add `## Discoveries` and `## Why Read?` sections if not already present

8. **Confirm**
   ```
   Processed: [[clip-name]]
   Type: {Video or Article}
   Key insights: {count} bullets extracted
   Status: Clipped → Read
   ```

## Path Handling

**CRITICAL - Never escape spaces with backslashes:**
- Use paths exactly as shown: `01 Inbox/...` (with literal spaces)
- The Write tool handles spaces correctly - backslash escaping creates literal `\` characters in directory names
- When using Bash commands, wrap paths in double quotes: `"01 Inbox/..."`

## Examples

```bash
# Fetch and process (works for non-blocked sites)
/research-clip-read terraform-best-practices

# Blocked source — pass content as JSON (e.g., from browser copy)
/research-clip-read claude-obsidian --content '{"title": "Solutions to using Claude Code as your personal assistant", "body": "I set up CLAUDE.md with my vault structure...", "comments": [{"author": "user1", "body": "I do something similar with..."}]}'

# Blocked source — pass content as plain text
/research-clip-read claude-obsidian --content 'The post describes setting up Claude Code with Obsidian for life management. Key points: 1) Define vault structure in CLAUDE.md 2) Create custom slash commands...'

# Partial match
/research-clip-read terraform

# Interactive: list all clipped items
/research-clip-read

# Skip auto-fetch, just ask me for content
/research-clip-read some-article --skip-fetch
```

## Tips for Getting Content from Blocked Sources

When a source is blocked (Reddit, paywalled sites):
1. **Browser copy-paste**: Select all text on the page, copy, pass as `--content`
2. **Reader mode**: Use browser reader mode, then copy the clean text
3. **JSON from dev tools**: Copy the API response from browser Network tab
4. **Browser extension**: Use a "copy as markdown" extension

## Notes

- This skill EXTRACTS INSIGHTS — it's not just a status toggle
- Without content to analyze, the skill will ask for it rather than proceeding empty
- For full video summarization with transcripts, use `/research-clip` with a YouTube URL (auto-summarizes)
- Status progression: `Clipped` → `Read` (this skill) or `Clipped` → `Summarized` (`/research-clip` for videos, `/research-youtube-catchup` for batch)
