# JIRA Activity Procedure

Single source of truth for pulling JIRA tickets into a daily journal. Referenced by `journal-jira-activity` and `journal-daily-review`.

## Configuration

- **JIRA Username**: `duncanleung`
- **JIRA Base URL**: `https://airvet.atlassian.net`
- **Section heading**: `## ![[jira-logo.png|18]] JIRA Tickets`

## JQL Query

Use `searchJiraIssuesUsingJql` with `currentUser()` (resolves to the authenticated Atlassian user automatically):

```
assignee = currentUser() AND status NOT IN ("Done", "Closed", "Resolved") ORDER BY priority DESC, updated DESC
```

Limit 50 results.

### Fallback: explicit account ID

If `currentUser()` fails, resolve the account ID via `lookupJiraAccountId` with `searchString: "duncan"` (note: full username `"duncanleung"` returns 0 results), then use:

```
assignee = "{accountId}" AND status NOT IN ("Done", "Closed", "Resolved") ORDER BY priority DESC, updated DESC
```

## Data Extraction

For each ticket returned:
- **Key**: Issue key (e.g., `WEB-1234`, `AP-456`)
- **Summary**: Issue title/summary field
- **Status**: Current workflow status (e.g., `In Progress`, `To Do`, `In Review`)
- **Priority**: Issue priority (used for sorting)

## Output Format

```markdown
## ![[jira-logo.png|18]] JIRA Tickets

| Ticket | Title | Status |
|--------|-------|--------|
| [WEB-1234](https://airvet.atlassian.net/browse/WEB-1234) | Fix login redirect loop | In Progress |
| [WEB-1200](https://airvet.atlassian.net/browse/WEB-1200) | Add emergency contact fields | To Do |
| [AP-456](https://airvet.atlassian.net/browse/AP-456) | Update partner onboarding flow | In Review |
```

## Formatting Rules

- **Ticket column**: `[KEY](https://airvet.atlassian.net/browse/KEY)` — clickable link to JIRA issue
- **Title column**: Issue summary, truncated if > 50 characters
- **Status column**: Workflow status name as returned by JIRA
- If no assigned tickets: write `_No open JIRA tickets assigned._`
- Sort by priority DESC, then updated DESC (handled by JQL ORDER BY)

## Error Handling

- If `currentUser()` JQL fails: retry with explicit account ID lookup (see Fallback above)
- If `searchJiraIssuesUsingJql` fails: write `_Error fetching JIRA tickets._`
- If no results returned: write `_No open JIRA tickets assigned._`

## Placement Rules

1. If `## ![[jira-logo.png|18]] JIRA Tickets` already exists: **replace it entirely** (from heading to next `##` heading or end of file)
2. If it doesn't exist: insert after `## ![[slack-logo.png|18]] Slack Conversations`, before `## 📚 What Did I Study?`
3. If neither anchor section exists: append before `## 📝 Notes` or at the end of file