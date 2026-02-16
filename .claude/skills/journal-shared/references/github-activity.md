# GitHub Activity Procedure

Single source of truth for pulling GitHub activity into a daily journal. Referenced by `journal-github-activity`, `journal-daily-review`, and `journal-good-morning`.

## Configuration

- **GitHub username**: `duncanleung`
- **Section heading**: `## ![[github-logo.png|18]] GitHub Activity`

## Queries

### Step 1: Local Git Log Scanning (Private Repos)

Scan local repositories for commits not indexed by GitHub Search API:

```bash
# For each repo in Projects Index (from CLAUDE.md code: field) + the vault repo itself:
# (e.g., /Users/duncanleung/Develop/obsidian-local-life-manager, /Users/duncanleung/Develop/[other-project]/)

# Run git log for the target date, extracting: full SHA, commit message
git -C {repo-path} log \
  --format="%H|%s" \
  --since="{target-date}T00:00:00" \
  --until="{target-date}T23:59:59" \
  --author="duncanleung"

# For each repo with matches, also attempt to get remote URL for later link construction:
git -C {repo-path} config --get remote.origin.url
```

**Output:** List of commits with:
- Full SHA (40-char hash)
- Commit message
- Repo path (to map back to org/repo later)
- Remote URL (for `https://github.com/` URL construction)

**Deduplication:** Store all local commits by SHA. After GitHub API queries (Step 2), remove any local commits that already appear in GitHub results (same SHA = already captured by API).

---

### Step 2: GitHub Search API Queries

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

**Commits:**
- Store local commits (Step 1) by full SHA
- After GitHub API queries (Step 2), remove any local commits that match a GitHub API commit SHA
- Remaining local commits = new work not yet indexed by GitHub Search API
- Combine local + GitHub commits into single Commits table

**Pull Requests:**
- PRs may appear in multiple queries. Deduplicate by `{org/repo}#{number}`

## URL Construction

- **GitHub API Commits:** `https://github.com/{org}/{repo}/commit/{full-sha}`
- **Local Commits with Remote:** Extract `{org}/{repo}` from git remote URL `git@github.com:{org}/{repo}.git` → `https://github.com/{org}/{repo}/commit/{full-sha}`
- **Local Commits without Remote (or unpushed):** Use `{repo-name}` from repo path as display name. No hyperlink needed—show as plain text or note as "local-only"
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
| {repo-name} | `{short-sha}` | {commit message title} |

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

- **Combine local + GitHub commits in Commits table**: Both sources appear in the same table. GitHub-API commits use full `org/repo` names with hyperlinks. Local commits use repo name with optional hyperlink (if remote URL available).
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
