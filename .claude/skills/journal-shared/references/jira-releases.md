# JIRA WEB Releases Procedure

Single source of truth for pulling WEB board unreleased versions into a daily journal. Referenced by `journal-jira-releases`, `journal-daily-review`, and `journal-good-morning`.

## Configuration

- **JIRA Base URL**: `https://airvet.atlassian.net`
- **Project**: `WEB`
- **Section heading**: `## ![[jira-logo.png|18]] WEB Releases`

## JQL Query

Use `searchJiraIssuesUsingJql`:

```
project = WEB AND fixVersion in unreleasedVersions() ORDER BY fixVersion ASC, priority DESC
```

Request fields: `summary, status, issuetype, fixVersions, priority`

Limit 100 results.

## Data Extraction

For each ticket returned:
- **Key**: Issue key (e.g., `WEB-1234`)
- **Summary**: Issue title/summary field
- **Status**: Current workflow status (e.g., `In Progress`, `To Do`, `Done`)
- **Issue Type**: Type (e.g., `Bug`, `Story`, `Task`)
- **Fix Versions**: Array of version objects — extract `name`, `id`, `releaseDate`, and `description` from each

## Grouping

Group issues by `fixVersions[0].name` (primary fix version). If an issue has multiple fix versions, use the first one.

Build a map of version name -> list of issues.

**Sort releases by soonest first:** Sort version groups by `releaseDate` ascending (earliest date first). Versions with no release date go last.

For each version group, compute:
- **Total count**: number of issues
- **Done count**: issues where status category = "Done" or status name matches "Done", "Closed", "Resolved"
- **In Progress count**: issues where status category = "In Progress" or status name matches "In Progress", "In Review", "In QA"
- **To Do count**: total - done - in progress (everything else)

## Output Format

```markdown
## ![[jira-logo.png|18]] WEB Releases

### [v2.5.0](https://airvet.atlassian.net/projects/WEB/versions/13200) — 8 issues (3 Done, 2 In Progress, 3 To Do)
- **Release Date:** 2026-03-01 (10 business days left)
- **Description:** Sprint 12 release

| Ticket | Title | Status | Type |
|--------|-------|--------|------|
| [WEB-1234](https://airvet.atlassian.net/browse/WEB-1234) | Fix login redirect loop | <span style="color: red">In Progress</span> | Bug |
| [WEB-1200](https://airvet.atlassian.net/browse/WEB-1200) | Add emergency contact fields | <span style="color: red">To Do</span> | Story |
| [WEB-1180](https://airvet.atlassian.net/browse/WEB-1180) | Update partner onboarding | <span style="color: green">Done</span> | Task |

### [v2.4.1](https://airvet.atlassian.net/projects/WEB/versions/13100) — 3 issues (1 Done, 2 In Progress, 0 To Do)
- **Release Date:** _not set_
- **Description:** _none_

| Ticket | Title | Status | Type |
|--------|-------|--------|------|
| [WEB-1100](https://airvet.atlassian.net/browse/WEB-1100) | Hotfix session timeout | <span style="color: red">In Progress</span> | Bug |
```

## Formatting Rules

- **Release header**: `### [{versionName}](https://airvet.atlassian.net/projects/WEB/versions/{versionId}) — {totalCount} issues ({doneCount} Done, {inProgressCount} In Progress, {todoCount} To Do)` — use the `id` from the fixVersion object to build the link
- **Release date**: `- **Release Date:** {releaseDate} ({N} business days left)` — use the `releaseDate` from the fixVersion object. Calculate business days remaining from today's date (exclude weekends — Sat/Sun). If the release date is today, show `(0 business days left)`. If the release date is in the past, show `(overdue)`. If not set, show `_not set_`
- **Description**: `- **Description:** {description}` — use the `description` from the fixVersion object. If empty/missing, show `_none_`
- **Ticket column**: `[KEY](https://airvet.atlassian.net/browse/KEY)` — clickable link to JIRA issue
- **Title column**: Issue summary, truncated if > 50 characters
- **Status column**: Workflow status name wrapped in a colored `<span>` tag:
  - **Red**: To Do, In Progress, Changes Requested, In Review → `<span style="color: red">{status}</span>`
  - **Yellow**: Needs Testing, In QA → `<span style="color: orange">{status}</span>` (use `orange` for readability)
  - **Green**: Done → `<span style="color: green">{status}</span>`
  - Any other status: default to red
- **Type column**: Issue type name as returned by JIRA
- If no unreleased versions have issues: write `_No issues in unreleased WEB versions._`
- Sort: versions by releaseDate ASC (soonest first, no-date last), issues within each version by priority DESC (from JQL)

## Error Handling

- If `searchJiraIssuesUsingJql` fails: write `_Error fetching WEB releases._`
- If no results returned: write `_No issues in unreleased WEB versions._`

## Placement Rules

1. If `## ![[jira-logo.png|18]] WEB Releases` already exists: **replace it entirely** (from heading to next `##` heading or end of file)
2. If it doesn't exist: insert after `## ![[jira-logo.png|18]] JIRA Tickets`, before `## 📚 What Did I Study?`
3. If neither anchor section exists: append before `## 📝 Notes` or at the end of file
