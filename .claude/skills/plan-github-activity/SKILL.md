---
name: plan-github-activity
description: Pull GitHub activity (commits, PRs, reviews, comments) into today's journal. Safe to run anytime — does NOT freeze dataviews. Triggers on "github activity", "pull github", "git activity", "what did I push".
model: claude-haiku-4-5-20251001
argument-hint: [YYYY-MM-DD or path to journal file]
allowed-tools: Bash(gh:*), Bash(date:*), Read, Write, Edit, Glob
---

Pull GitHub activity into the daily journal. This is safe to run throughout the day — it only touches the `## ![[github-logo.png|18]] GitHub Activity` section and does NOT freeze dataview blocks.

$ARGUMENTS

## Steps

1. **Determine target journal**
   - If `$ARGUMENTS` contains a file path (e.g., `02 Calendar/2026-02-12.md`): use that file directly and extract the date from the filename
   - If `$ARGUMENTS` contains a date (e.g., `2026-02-12`): use `02 Calendar/YYYY-MM-DD.md`
   - If no arguments: run `date +%Y-%m-%d` to get today's date, use `02 Calendar/YYYY-MM-DD.md`

2. **Read the journal file**
   - If it doesn't exist: create it with the standard format (see bottom of this file)
   - Note whether a `## ![[github-logo.png|18]] GitHub Activity` section already exists (it will be replaced)

3. **Pull GitHub data** — run ALL of these queries for the target date:

   ```bash
   # 1. Commits authored (extract: org/repo, full SHA, commit title)
   gh search commits --author=duncanleung --committer-date={target-date} --limit=50

   # 2. PRs merged (extract: org/repo, PR number, title)
   gh search prs --author=duncanleung --merged-at={target-date} --limit=20

   # 3. PRs created/opened (extract: org/repo, PR number, title)
   gh search prs --author=duncanleung --created={target-date} --limit=20

   # 4. PRs commented on (extract: org/repo, PR number, title)
   gh api search/issues --method GET -f q="commenter:duncanleung type:pr updated:{target-date}" --jq '.items[] | "\(.repository_url | split("/")[-2:] | join("/")) #\(.number) \(.title) \(.state) \(.html_url)"'

   # 5. PRs involved in (reviews requested, approvals, etc.)
   gh api search/issues --method GET -f q="involves:duncanleung type:pr updated:{target-date}" --jq '.items[] | "\(.repository_url | split("/")[-2:] | join("/")) #\(.number) \(.title) \(.state) \(.html_url)"'
   ```

4. **Deduplicate** — PRs may appear in multiple queries. Dedupe by repo/number.

5. **Build URLs** for every entry:
   - **Commits:** `https://github.com/{org}/{repo}/commit/{full-sha}`
   - **Pull Requests:** `https://github.com/{org}/{repo}/pull/{number}`
   - **Reviews & Comments:** `https://github.com/{org}/{repo}/pull/{number}` (links to the PR)

6. **Format and insert** the `## ![[github-logo.png|18]] GitHub Activity` section:

   Every entry MUST include a clickable web link. Use markdown link syntax.

   ```markdown
   ## ![[github-logo.png|18]] GitHub Activity

   ### Commits
   - **{org/repo}**
     - [`{short-sha}`](https://github.com/{org}/{repo}/commit/{full-sha}) {commit message title}
   - **{org/repo}**
     - [`{short-sha}`](https://github.com/{org}/{repo}/commit/{full-sha}) {commit message title}

   ### Pull Requests
   - 🟣 Merged — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) {title}
   - 🟢 Opened — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) {title}

   ### Reviews & Comments
   - 💬 Commented on [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) — {title}
   - 👀 Review requested — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) — {title}
   ```

   - Group commits by repo
   - Omit any subsection (Commits, Pull Requests, Reviews & Comments) that has zero items
   - If NO GitHub activity at all: write `_No GitHub activity._` under the `## ![[github-logo.png|18]] GitHub Activity` heading

7. **Place the section** in the journal:
   - If `## ![[github-logo.png|18]] GitHub Activity` already exists: **replace it entirely** (from `## ![[github-logo.png|18]] GitHub Activity` to the next `##` heading or end of file)
   - If it doesn't exist: insert after `## 🔨 What Did I Work On?`, before `## 📚 What Did I Study?`
   - If neither anchor section exists: append before `## 📝 Notes` or at the end

## Important

- Do NOT freeze dataview blocks — that's only for `/plan-daily-review`
- Do NOT touch any other sections (📋 What Did I Do?, ⭐ Highlight, etc.)
- Do NOT commit to git
- Stop after updating the GitHub Activity section

## Journal Format

If creating a new journal:

```markdown
---
created: YYYY-MM-DD
modified: YYYY-MM-DD
status: in-progress
---

# {Month} {Day}, {Year} - {DayOfWeek}

## ⭐ Highlight


## 📋 What Did I Do?


## 🔨 What Did I Work On?


## ![[github-logo.png|18]] GitHub Activity


## 📚 What Did I Study?


## 📝 Notes

```
