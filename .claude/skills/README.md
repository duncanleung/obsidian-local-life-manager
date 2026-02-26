# Skills Catalog

Discovery index for Claude Code skills. For detailed usage, examples, args, and workflow guidance, see the [root README Skills Reference](../../README.md#skills-reference). For individual skill internals, read the `SKILL.md` file in each skill directory.

## All Skills

| Skill | Description |
|-------|-------------|
| `/journal-quick-task` | Create a new task note in 03 TaskNotes/ |
| `/journal-work-log` | Quick work log entry to today's journal |
| `/journal-task-done` | Mark a task as complete |
| `/journal-whats-next` | Personal triage — overdue tasks, due-today, unfrozen journals, inbox backlog, meeting follow-ups |
| `/journal-github-activity` | Pull GitHub activity (commits, PRs, reviews) into today's journal |
| `/journal-slack-activity` | Pull Slack work conversations into today's journal |
| `/journal-meeting-notes` | Capture meeting notes with attendees, decisions, and auto-created tasks |
| `/journal-daily-review` | Unified daily journal — morning setup, activity pulls, freeze, status report |
| `/journal-weekly-review` | Weekly review and planning session |
| `/session-refresh` | Silently refresh AI context |
| `/session-save-context` | Save session context handoff for continuation |
| `/session-debrief` | End-of-session reflection and memory extraction |
| `/session-improve-processes` | Suggest improvements to agents, skills, workflows |
| `/project-brief` | Create or update project brief through interactive discovery |
| `/project-critique` | Challenge idea assumptions with skeptical VC-style evaluation |
| `/project-init-space` | Initialize project planning structure in 06 Projects/ and optionally docs structure |
| `/project-features` | Decompose project brief into features with user stories, AC, dependency graph, and MVP scope |
| `/project-import-spec` | Import external requirements (JIRA, Confluence, GitHub, URL) into the project |
| `/project-spec` | (Deprecated) Use /project-features or /project-import-spec |
| `/project-adr` | Create Architecture Decision Records |
| `/project-ui-design` | Create HTML UI mockups in 06 Projects/[project]/docs/ui-designs/ |
| `/project-issue` | Create work items (TASK, BUG, SPIKE) with AI-assisted type detection |
| `/project-plan` | Create PLAN.md with phase-based breakdown for issues |
| `/project-implement` | Execute plan phases, writing code in project code repo (from Projects Index) |
| `/project-advise` | Interactive guidance — you implement with step-by-step advice |
| `/project-teach` | Deep pedagogical guidance with Socratic teaching |
| `/project-worklog` | Add timestamped work log entries |
| `/project-commit` | Create git commits with conventional message format |
| `/project-complete` | Validate, document, review, commit, and merge to develop |
| `/project-status-all` | Project status dashboard — quick glance table + detailed analysis |
| `/project-next-step` | Assess project readiness and recommend the next skill to run |
| `/project-validate-space` | Validate project space structure and consistency |
| `/project-validate-sanity-check` | Step back, reflect, validate direction |
| `/project-validate-troubleshoot` | Systematic debugging with 5-step loop |
| `/project-spec-validate` | Validate spec completeness and implementation compliance |
| `/project-validate-quality` | Comprehensive code quality assessment (multi-agent) |
| `/project-validate-security-audit` | Security audit using 5 parallel agents |
| `/learn-start-session` | Start a new learning session on a topic |
| `/learn-log-session` | Log entries to the current learning session |
| `/learn-end-session` | End the current learning session |
| `/learn-review-session` | Retrieval practice session to test retention |
| `/learn-study-notes` | Create comprehensive study notes on a topic |
| `/learn-flashcards` | Generate spaced repetition flashcards |
| `/research-deep` | Deep research on a topic with persistent documentation |
| `/research-rss-catchup` | Fetch and summarize latest articles from RSS feeds |
| `/research-youtube-catchup` | Fetch and summarize latest videos from YouTube channels |
| `/research-video-summarize` | Summarize a single YouTube video (prefer `/research-clip` which auto-summarizes) |
| `/research-synthesize` | Synthesize across multiple sources into structured documents |
| `/research-clip` | Capture articles/videos to 01 Inbox/. Articles: quick summary, `Clipped`. YouTube: full transcript summary, `Summarized` |
| `/research-clip-read` | Process a clip — extract key insights, write summary, mark Read. Accepts `--content` for blocked sources |
| `/ops-process-inbox` | Process and organize inbox notes |
| `/ops-docs` | Documentation health check, README validation, and maintenance |
| `/ops-git-sync` | Sync all git repos |

## Shared References

Non-skill directories containing shared reference files:

| Directory | Purpose |
|-----------|---------|
| `project-shared/references/` | Shared procedures for project skills (project-discovery) |
| `learn-shared/references/` | Shared schemas for learning skills (session-schema, entry-types, proficiency, flashcard-syntax) |
| `journal-shared/references/` | Shared procedures for journal skills (github-activity, slack-activity, journal-format) |

## Skill File Structure

Each skill has its own directory:
```
skills/
├── skill-name/
│   ├── SKILL.md           # Skill definition (required)
│   └── references/        # Supporting docs (optional)
│       └── schema.md
```

### SKILL.md Frontmatter

```yaml
---
name: skill-name
description: "One-line description"
model: claude-opus-4-5-20251101   # or claude-sonnet-4-20250514, claude-haiku-4-5-20251001
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
argument-hint: [optional arguments]
---
```
