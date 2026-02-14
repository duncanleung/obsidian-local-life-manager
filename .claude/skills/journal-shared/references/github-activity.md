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

Every entry MUST include a clickable web link:

```markdown
## ![[github-logo.png|18]] GitHub Activity

### Commits
- **{org/repo}**
  - [`{short-sha}`](https://github.com/{org}/{repo}/commit/{full-sha}) {commit message title}

### Pull Requests
- 🟣 Merged — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) {title}
- 🟢 Opened — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) {title}

### Reviews & Comments
- 💬 Commented on [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) — {title}
- 👀 Review requested — [**{org/repo}#{number}**](https://github.com/{org}/{repo}/pull/{number}) — {title}
```

## Formatting Rules

- Group commits by repo
- Omit any subsection (Commits, Pull Requests, Reviews & Comments) that has zero items
- If NO GitHub activity at all: write `_No GitHub activity._` under the heading

## Placement Rules

1. If `## ![[github-logo.png|18]] GitHub Activity` already exists: **replace it entirely** (from heading to next `##` heading or end of file)
2. If it doesn't exist: insert after `## 🔨 What Did I Work On?`, before `## 📚 What Did I Study?`
3. If neither anchor section exists: append before `## 📝 Notes` or at the end of file
