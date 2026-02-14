# GitHub Activity Procedure

Single source of truth for pulling GitHub activity into a daily journal. Referenced by `journal-github-activity`, `journal-daily-review`, and `journal-good-morning`.

## Configuration

- **GitHub username**: `duncanleung`
- **Section heading**: `## ![[github-logo.png|18]] GitHub Activity`

## Queries

Run ALL of these `gh` queries for the **target date**:

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

## Deduplication

PRs may appear in multiple queries. Deduplicate by `{org/repo}#{number}`.

## URL Construction

- **Commits:** `https://github.com/{org}/{repo}/commit/{full-sha}`
- **Pull Requests:** `https://github.com/{org}/{repo}/pull/{number}`
- **Reviews & Comments:** `https://github.com/{org}/{repo}/pull/{number}`

## Output Format

Every entry MUST include a clickable web link. Use tables with H3 headers and summary counts (mirroring the JIRA Releases pattern).

```markdown
## ![[github-logo.png|18]] GitHub Activity

### Commits — {N} commits across {M} repos

| Repo | Commit | Message |
|------|--------|---------|
| {org/repo} | [`{short-sha}`](https://github.com/{org}/{repo}/commit/{full-sha}) | {commit message title} |

### Pull Requests — {N} PRs ({X} Merged, {Y} Opened)

| PR | Title | Status |
|----|-------|--------|
| [{org/repo}#{number}](https://github.com/{org}/{repo}/pull/{number}) | {title} | <span style="color: green">Merged</span> |
| [{org/repo}#{number}](https://github.com/{org}/{repo}/pull/{number}) | {title} | <span style="color: blue">Opened</span> |

### Reviews & Comments — {N} items

| PR | Title | Activity |
|----|-------|----------|
| [{org/repo}#{number}](https://github.com/{org}/{repo}/pull/{number}) | {title} | 💬 Commented |
| [{org/repo}#{number}](https://github.com/{org}/{repo}/pull/{number}) | {title} | 👀 Review Requested |
```

## Formatting Rules

- **H3 headers with summary counts**: Each subsection header includes a count summary
  - Commits: `### Commits — {N} commits across {M} repos`
  - Pull Requests: `### Pull Requests — {N} PRs ({X} Merged, {Y} Opened)` (omit zero-count statuses)
  - Reviews & Comments: `### Reviews & Comments — {N} items`
- **Title column**: Truncate to 50 characters if longer
- **Status column (PRs)**: Colored `<span>` tags:
  - **Green**: Merged → `<span style="color: green">Merged</span>`
  - **Blue**: Open → `<span style="color: blue">Opened</span>`
  - **Red**: Closed (not merged) → `<span style="color: red">Closed</span>`
- **Activity column (Reviews)**: Emoji prefix + activity type (e.g., `💬 Commented`, `👀 Review Requested`, `✅ Approved`)
- Omit any subsection (Commits, Pull Requests, Reviews & Comments) that has zero items
- If NO GitHub activity at all: write `_No GitHub activity._` under the heading

## Placement Rules

1. If `## ![[github-logo.png|18]] GitHub Activity` already exists: **replace it entirely** (from heading to next `##` heading or end of file)
2. If it doesn't exist: insert after `## 🔨 What Did I Work On?`, before `## 📚 What Did I Study?`
3. If neither anchor section exists: append before `## 📝 Notes` or at the end of file
