# Local Life Manager

- A framework for AI-assisted life and project management using Claude Code.

See:

- [I'm Letting Claude Manage My Life (Sort Of)](https://taylorhuston.me/2026/01/13/Claude-Life-Management.html)
- [Local Life Management (LLM? Get It?)](https://taylorhuston.me/2026/01/21/Local-Life-Management.html)

---
## What This Is

A structured approach to:

- **Project planning** with briefs, specs, and issue tracking
- **Personal knowledge management** with daily journals and learning systems
- **AI-powered workflows** via 50 custom Claude Code skills
- **Multi-project coordination** with shared standards and templates

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed
- [Obsidian](https://obsidian.md/) for personal knowledge management
- Git

## What Can It Do?

### Journal, Daily Review, Weekly Review

- Commands to help you keep track of your daily journal.
- Have it help you with your daily review each evening.
- Have it help you plan your day each morning.
- Have it help you reflect on the previous week and set goals for the upcoming week every Sunday.

### Spec Driven Development

- Contains an updated version of the skills and workflows from my [Claude Code SDD Plugin](https://github.com/TaylorHuston/ai-toolkit).
- Skills to create detailed specs, plans, tasks and then implement those plans in a structured way.
- Also includes a "/teach" mode where the LLM will walk you through building the spec yourself, great for a truly personalized tutorial.

### Personal Tutor

- The LLM can create study sessions for you, tracking what topics you've already worked on.
- Help you study any topic you want, taking in your previous study sessions and existing notes into account.

### Summarize YouTube Channels and RSS Feeds

- Constantly find yourself drowning in all of the different things you feel like you need to watch and read to stay on top of things?
- The LLM can track a list of YouTube channels you want to follow, download the transcripts, and create summaries for each of them in your vault.
- The LLM can track a list of RSS feeds you want to follow, read the latest entries and create summaries for each article in your vaut.
- Then you can read these summaries and determine which ones are actually worth your time to go watch or read.

_Note that a lot of this could still be considered a beta at best, this is definitely a work in progress. Skills and workflows are changing frequently, these instructions might not always be up to date, but you should be able to ask Claude itself what it can do in this project._

---
## Directory Structure

```
local-life-manager/
├── .claude/
│   ├── skills/              # 50 custom Claude Code skills
│   ├── agents/              # 26 specialized agent definitions
│   ├── docs/                # System documentation
│   ├── obsidian-memories/    # Persistent AI context about you (create this)
│   └── learning-sessions/   # Learning progress tracking (create this)
├── 01 Inbox/ … 08 System/   # Obsidian vault directories (gitignored)
├── 06 Projects/             # Project planning (private strategy docs)
│   └── [project]/
│       ├── README.md
│       ├── project-brief.md
│       ├── specs/
│       ├── issues/
│       └── notes/
├── SHARED/
│   ├── TEMPLATES/           # Project and doc templates
│   └── DOCS/                # Cross-project standards
├── CLAUDE.md                # AI instructions (customize this)
└── CHANGELOG.md
```

## Setup

### 1. Clone and Initialize

```bash
git clone https://github.com/TaylorHuston/local-life-manager.git
cd local-life-manager

# Create personal directories
mkdir -p .claude/obsidian-memories .claude/learning-sessions
```

### 2. Vault Directories

The numbered directories at the repo root are your Obsidian vault (gitignored). They're created during setup:

```
├── 01 Inbox/           # Capture location for new notes
├── 02 Calendar/        # Daily notes (YYYY-MM-DD.md), weekly reviews (YYYY-Www.md)
├── 03 TaskNotes/       # Tasks (each task = note with #task tag)
├── 04 Meetings/        # Meeting notes
├── 05 Personal/        # Personal notes, decisions, career, health
├── 06 Projects/        # Active projects
├── 07 Knowledge Base/  # Courses, videos, tech notes
└── 08 System/          # Templates, Classes, assets
```

Open this repo root in Obsidian as your vault.

### 3. Create Your Context

Create `.claude/obsidian-memories/about-me.md`:

```markdown
# About Me

## Profile

- **Name:** Your Name
- **Role:** What you do
- **Focus:** Current priorities

## Communication Preferences

- Style preferences
- What to avoid

## Key Accounts

- GitHub: https://github.com/you
- Other relevant links
```

Initialize the memories index:

```bash
echo '[]' > .claude/obsidian-memories/index.json
```

### 4. Customize CLAUDE.md

Edit `CLAUDE.md` to:

- Update the overview with your context
- Add your projects to the Projects Index
- Adjust paths if your vault has different structure
- Add any personal preferences

### 5. Configure Personal Skills

These skills need your data:

**RSS Feeds** (`.claude/skills/research-rss-catchup/references/feeds.json`):

```json
{
  "feeds": [
    {
      "name": "Blog Name",
      "url": "https://example.com/feed.xml",
      "category": "tech"
    }
  ]
}
```

**YouTube Channels** (`.claude/skills/research-youtube-catchup/references/channels.json`):

```json
{
  "channels": [{ "name": "Channel", "channel_id": "UC...", "priority": "high" }]
}
```

---
## Usage

### Starting a Session

```bash
cd local-life-manager
claude
```

---
## Skills Reference

Args: `<required>`, `[optional]`, `"quoted"` for strings, `\|` for alternatives.

### Overview: How Skills Relate

All skills fit into a daily rhythm. The six groups form parallel tracks wrapped by session bookends:

```
┌─────────────────────────────────────────────────────────────┐
│  SESSION START                                              │
│  /session-refresh                                           │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  MORNING                                                    │
│  /journal-good-morning                                         │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  WORK (parallel tracks — pick what's needed)                │
│                                                             │
│  ┌─ Project ───────── /project-* pipeline ───────────────┐  │
│  │  └─ Quality ────── /project-validate-* at checkpoints         │  │
│  ├─ Learning ──────── /learn-start → work → /learn-end   │  │
│  ├─ Research ──────── /research-* → /research-synthesize  │  │
│  ├─ Triage + tasks ── /journal-whats-next, /journal-quick-task  │  │
│  └─ Maintenance ───── /ops-*                              │  │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  EVENING                                                    │
│  /journal-daily-review                                         │
│  /session-debrief                                           │
│  /session-save-context  (if switching topics)               │
└─────────────────────────────────────────────────────────────┘

Periodic:  /journal-weekly-review (weekly)
           /ops-docs, /ops-git-sync (maintenance)
```

---

### 🛠️ Personal Skills

Daily and weekly rhythm skills. Ordered by time of day, not by dependency. "During day" skills are all independent — use in any order.

```
┌─────────────────────────────────────────────────────────────┐
│  MORNING                                                    │
│  /journal-good-morning  (creates journal + task dashboard)     │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  DURING DAY (any order, use as needed)                      │
│                                                             │
│  /journal-whats-next       (triage — what needs attention?)    │
│                                                             │
│  /journal-quick-task      · /journal-task-done                    │
│  /journal-work-log        · /journal-meeting-notes                │
│                                                             │
│  /journal-github-activity · /journal-slack-activity               │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  EVENING                                                    │
│  /journal-daily-review  (completes journal, captures memories) │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  WEEKLY      /journal-weekly-review  (reviews 7 days)          │
└─────────────────────────────────────────────────────────────┘
```

| Stage    | Skill                   | Purpose                                               | Args                   |
| -------- | ----------------------- | ----------------------------------------------------- | ---------------------- |
| Morning  | `/journal-good-morning`    | Morning routine + task dashboard                      |                        |
| Triage   | `/journal-whats-next`      | Personal triage (overdue, due today, inbox, journals) |                        |
|          |                         |                                                       |                        |
| Tasks    | `/journal-quick-task`      | Create task in 03 TaskNotes/                          | `"title" [--due DATE]` |
|          | `/journal-task-done`       | Mark a task as complete                               | `"task name"`          |
| Activity | `/journal-work-log`        | Quick work log entry                                  | `"entry"`              |
|          | `/journal-meeting-notes`   | Capture meeting notes, auto-create tasks              | `[meeting details]`    |
| Fetch    | `/journal-github-activity` | Pull GitHub activity into journal                     | `[YYYY-MM-DD \| path]` |
|          | `/journal-slack-activity`  | Pull Slack work conversations into journal            | `[YYYY-MM-DD \| path]` |
| Evening  | `/journal-daily-review`    | End of day journal completion                         |                        |
| Weekly   | `/journal-weekly-review`   | Weekly planning                                       |                        |

#### 📕 `/journal-good-morning`

**Morning routine check-in + task dashboard generation**

Checks yesterday's journal completeness, creates today's journal from template if missing, carries over Notes from yesterday, shows learning reviews due, generates task dashboard from open tasks and active projects.

```bash
/journal-good-morning  # Morning check-in
```

**Creates:** `02 Calendar/YYYY-MM-DD.md` (today's journal), `02 Calendar/tasks-YYYY-MM-DD.md` (task dashboard).
**Triggers on:** "good morning", "morning", "start my day"


#### 📕 `/journal-quick-task`

**Create a new task note in 03 TaskNotes/**

Creates a task file with YAML frontmatter (status, created, due, tags). Checks for duplicate filenames.

```bash
/journal-quick-task "Review PR for auth module"
/journal-quick-task "Submit expense report" --due 2026-02-14
```

**Creates:** `03 TaskNotes/{kebab-case-title}.md`
**Triggers on:** "new task", "add task", "create task"

#### 📕 `/journal-work-log`

**Quick work log entry to today's journal without full review**

Adds a bullet point to the appropriate section of today's journal. Creates journal from template if missing.

```bash
/journal-work-log "finished AWS EFS module"
```

**Triggers on:** "log this", "add to journal", "I just did", "work log"

#### 📕 `/journal-task-done`

**Mark a task as complete**

Finds task by name or partial match, sets `status: complete` and adds `completed` date.

```bash
/journal-task-done "expense report"
```

**Triggers on:** "task done", "complete task", "finished task"

#### 📕 `/journal-whats-next`

**Personal triage — what needs attention right now**

Scans overdue tasks, tasks due today, unfrozen past journals, unprocessed inbox items, meetings with open action items, and learning reviews due. Produces a quick triage report with counts and a single prioritized recommendation. Does not scan projects — use `/project-status-all` for that.

```bash
/journal-whats-next  # Quick personal triage
```

**Triggers on:** "what's next", "what should I do", "prioritize", "what now", "triage"

#### 📕 `/journal-daily-review`

**Complete daily journal review**

Fills in journal sections (Work, Personal, Study). Pulls GitHub commits. Reviews highlight. Captures memories. Can set tomorrow's highlight.

```bash
/journal-daily-review  # Evening review
```

**Triggers on:** "daily review", "end of day", "journal review"

#### 📕 `/journal-weekly-review`

**Weekly review and planning session**

Reviews past week: checks all 7 daily journals for completeness, pulls GitHub activity, reviews learning progress and project worklogs. Plans next week with focus areas.

```bash
/journal-weekly-review  # Weekly review and planning
```

**Creates:** `02 Calendar/YYYY-Www.md` (weekly note from template).
**Triggers on:** "weekly review", "plan my week", "Sunday planning"

#### 📕 `/journal-github-activity`

**Pull GitHub activity (commits, PRs, reviews, comments) into today's journal**

Queries GitHub for commits authored, PRs merged/opened, and reviews/comments. Deduplicates across queries, builds clickable links, groups commits by repo. Only touches the `## GitHub Activity` section — safe to run anytime without affecting other journal sections or dataview blocks.

```bash
/journal-github-activity                       # Pull today's activity
/journal-github-activity 2026-02-12            # Pull for specific date
/journal-github-activity 02 Calendar/2026-02-12.md  # Target specific journal file
```

**Triggers on:** "github activity", "pull github", "git activity", "what did I push"

#### 📕 `/journal-slack-activity`

**Pull Slack work conversations into today's journal**

Searches Slack for messages sent and received on the target date. Groups by thread, resolves user names, filters to work-relevant conversations only (excludes social banter, bot noise). Only touches the `## Slack Conversations` section — safe to run anytime.

```bash
/journal-slack-activity                        # Pull today's conversations
/journal-slack-activity 2026-02-12             # Pull for specific date
/journal-slack-activity 02 Calendar/2026-02-12.md  # Target specific journal file
```

**Triggers on:** "slack activity", "pull slack", "slack conversations", "what did I discuss"

#### 📕 `/journal-meeting-notes`

**Capture meeting notes with attendees, decisions, and action items**

Two modes: Quick Capture (just a title) creates a skeleton note; Detailed mode (empty args or rich content) prompts for attendees, decisions, and action items. Auto-creates task notes in `03 TaskNotes/` from action items and links them back. Adds a work log entry to the meeting date's journal.

```bash
/journal-meeting-notes "Team standup"          # Quick capture
/journal-meeting-notes                         # Detailed: prompts for all info
/journal-meeting-notes "Sprint planning 02/12/2026 with Alice, Bob — decided on React, action: set up repo"
```

**Creates:** `04 Meetings/YYYY-MM-DD-{title}.md`, `03 TaskNotes/` (from action items).
**Triggers on:** "meeting notes", "meeting", "capture meeting", "log meeting"

---

### 🛠️ Session Skills

AI session lifecycle management. Bookend pattern: refresh at start, save/debrief at end.

```
┌─────────────────────────────────────────────────────────────┐
│  START                                                      │
│  /session-refresh                                           │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
                  [ ... work ... ]
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  END (independent — natural ordering shown)                 │
│                                                             │
│  /session-save-context        (topic-specific handoff)      │
│  /session-debrief             (session-wide learnings)      │
│  /session-improve-processes   (meta-improvement)            │
└─────────────────────────────────────────────────────────────┘
```

| Stage       | Skill                        | Purpose                          | Args                                                     |
| ----------- | ---------------------------- | -------------------------------- | -------------------------------------------------------- |
| Start       | `/session-refresh`           | Reload AI context                |                                                          |
| Mid-session | `/session-save-context`      | Save session context for handoff | `"topic"`                                                |
| End         | `/session-debrief`           | End-of-session reflection        |                                                          |
|             | `/session-improve-processes` | Suggest workflow improvements    | `[--focus agents \| commands \| templates \| workflows]` |

#### 📕 `/session-refresh`

**Silently refresh AI context by reading project configuration**

Reads CLAUDE.md, about-me.md, recent memories (last 3 days), shared docs, and last 3 git commits. Silent operation — just outputs "Context refreshed."

```bash
/session-refresh  # Silent context reload
```

**Use when:** Starting new conversation, after context loss, before major tasks.

#### 📕 `/session-save-context`

**Save AI session context handoff to Obsidian vault**

Creates a comprehensive context handoff file optimized for a new AI agent to continue work. Auto-discovers related files, asks 4 focused questions, generates structured markdown with YAML frontmatter.

```bash
/session-save-context "authentication refactor"
/session-save-context "learning system improvements"
/session-save-context "weekly review process"
```

**Creates:** `06 Projects/{project}/context/YYYY-MM-DD-{topic-slug}.md`
**Triggers on:** "save context", "handoff", "session context"
**Use when:** Before ending a session on a specific topic, switching between projects, before a long break.

#### 📕 `/session-debrief`

**End-of-session reflection and memory extraction**

Extracts memories, suggests updates to about-me.md and CLAUDE.md. Interactive approval process for each proposed change. Run before ending a long session or when context is getting full.

```bash
/session-debrief  # End-of-session reflection
```

**Creates:** `.claude/obsidian-memories/YYYY-MM-DD-NNN.json` (memory files).
**Triggers on:** "debrief", "extract memories", "session summary"

#### 📕 `/session-improve-processes`

**Reflect on session and suggest improvements to tooling**

Analyzes conversation for friction points, missing commands, workflow gaps. Suggests improvements to agents, skills, templates, workflows. Interactive: implement now, create issue, or skip.

```bash
/session-improve-processes                    # Reflect on session
/session-improve-processes --focus agents     # Focus on agent improvements
/session-improve-processes --focus workflows  # Focus on workflow improvements
```

**Use when:** End of significant work session, after implementing new features, when something felt "off."

---

### 🛠️ Project Skills

Take an idea from concept to shipped code. Skills form a pipeline — later skills require artifacts from earlier ones. Parallel steps are shown side-by-side; alternatives are shown as branches.

```
┌─────────────────────────────────────────────────────────────┐
│  OVERVIEW (use anytime)                                     │
│                                                             │
│  /project-status-all     (quick table + detailed dashboard) │
│  /project-next-step      ("what should I do next?")         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  FOUNDATION (sequential)                                    │
│                                                             │
│  /project-brief                                             │
│       ↓                                                     │
│  /project-critique       ← requires project-brief.md       │
│       ↓                                                     │
│  /project-init-space                                        │
│       └→ /project-validate-space  (verify scaffolding)              │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  PLANNING                                                   │
│                                                             │
│  /project-spec ─────┐                                       │
│  /project-adr ──────┤  (parallel — no dependencies)         │
│  /project-ui-design ┘                                       │
│       ↓                                                     │
│  /project-issue          ← link to spec via implements:     │
│       ↓                                                     │
│  /project-plan           ← requires TASK/BUG/SPIKE.md      │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  BUILD (choose one path)                                    │
│                                                             │
│  ┌─ /project-implement   (AI writes code)    ← PLAN.md     │
│  ├─ /project-advise      (you code, AI guides)              │
│  └─ /project-teach       (you code, AI teaches deeply)      │
│       ↓                                                     │
│  /project-worklog  (auto with implement, manual otherwise)  │
│                                                             │
│  On-demand during build:                                    │
│  · /project-validate-sanity-check    (when direction uncertain)     │
│  · /project-validate-troubleshoot    (when bugs occur)              │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  QUALITY GATE (parallel — before shipping)                  │
│                                                             │
│  /project-validate-spec ──────────┐                                 │
│  /project-validate-quality ───────┤  (run in parallel)              │
│  /project-validate-security-audit ┘                                 │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIP                                                       │
│                                                             │
│  /project-commit                                            │
│       ↓                                                     │
│  /project-complete       ← requires PLAN.md phases done     │
└─────────────────────────────────────────────────────────────┘
```

| Stage      | Skill                              | Purpose                                           | Requires                  | Args                                                         |
| ---------- | ---------------------------------- | ------------------------------------------------- | ------------------------- | ------------------------------------------------------------ |
| Overview   | `/project-status-all`              | Project status dashboard (quick table + detailed) | —                         | `[--project name] [--detailed]`                              |
|            | `/project-next-step`               | Assess readiness, recommend next skill            | —                         | `<project>`                                                  |
| Foundation | `/project-brief`                   | Create project briefs through discovery           | —                         | `[--project name] [--review] [--force]`                      |
|            | `/project-critique`                | VC-style skeptical evaluation                     | `project-brief.md`        | `[--project name] [--focus market \| technical \| business]` |
|            | `/project-init-space`              | Initialize project planning structure in 06 Projects/   | —                         | `<project> [--type next \| python]`                          |
| Planning   | `/project-spec`                    | Write feature specifications                      | —                         | `[--import url] [--init] [--sync]`                           |
|            | `/project-adr`                     | Create Architecture Decision Records              | —                         | `["topic"]`                                                  |
|            | `/project-ui-design`               | Create HTML UI mockups                            | —                         | `<project> "screen" [--variants N]`                          |
|            | `/project-issue`                   | Create work items (TASK/BUG/SPIKE)                | —                         | `["description"] [--project name]`                           |
|            | `/project-plan`                    | Break issues into phases                          | `TASK/BUG/SPIKE.md`       | `<issue#> [--project name] [--second-opinion]`               |
| Build      | `/project-implement`               | Execute implementation plans                      | `PLAN.md`                 | `<project> <issue#> [phase \| --next \| --full]`             |
|            | `/project-advise`                  | Get guidance without AI writing code              | —                         | `<issue#> [--phase N]`                                       |
|            | `/project-teach`                   | Deep learning with Socratic method                | —                         | `<issue#> [--phase N]`                                       |
|            | `/project-worklog`                 | Track work progress and decisions                 | —                         | `<project> <issue#> "entry" [--decision \| --gotcha]`        |
| Quality    | `/project-validate-space`          | Validate project space structure                  | —                         | `<project>`                                                  |
|            | `/project-validate-sanity-check`   | Step back and validate direction                  | —                         | `[--project name]`                                           |
|            | `/project-validate-troubleshoot`   | Systematic debugging                              | —                         | `<project> ["description"] [issue#]`                         |
|            | `/project-validate-spec`           | Check spec completeness                           | —                         | `<spec-id> [--pre \| --post]`                                |
|            | `/project-validate-quality`        | Code quality assessment                           | —                         | `<project> [--focus security \| testing]`                    |
|            | `/project-validate-security-audit` | Security review                                   | —                         | `<project>`                                                  |
| Ship       | `/project-commit`                  | Create quality commits                            | —                         | `<project> ["message"] [--amend]`                            |
|            | `/project-complete`                | Finalize and merge work                           | `PLAN.md` phases complete | `<project> [issue#]`                                         |

#### 📕 `/project-status-all`

**Project status dashboard — quick glance table + detailed analysis**

Default mode shows a quick scannable table from README files. Use `--project` or `--detailed` for deeper analysis including specs, issues, dependencies, blockers, and git branches.

```bash
/project-status-all                       # Quick glance table of all projects
/project-status-all --project coordinatr  # Deep dive on one project
/project-status-all --detailed            # Comprehensive analysis of all projects
```

**Default output (quick glance):**

```
# Project Status Dashboard

## Active Projects

| Project      | Phase           | Last Activity | Next Step              |
|--------------|-----------------|---------------|------------------------|
| CareerBrain  | MVP in progress | 2026-02-10    | Complete auth flow     |
| Coordinatr   | Active planning | 2026-02-08    | Finish SPEC-002        |

## On Hold

| Project    | Reason                 |
|------------|------------------------|
| YourBench  | Job search priority    |
| IRL Social | Waiting on API access  |

## Quick Stats
- Active: 2
- On hold: 2
- Total: 4
```

**Detailed output (`--detailed` or `--project`):**

```
# Project Status Dashboard (Detailed)

## Active Projects

### Coordinatr
Status: Active planning
Specs: 2 (1 complete, 1 in progress)
Issues: 3 open (1 in_progress, 2 pending)
  - 001-auth-research (TASK, in_progress) ← feature/001-auth
  - 002-data-model (TASK, pending, blocked by 001)
  - 003-api-design (SPIKE, pending)
Branch: feature/001-auth (3 unpushed commits)

### CareerBrain
Status: MVP in progress (60%)
Specs: 1 (complete)
Issues: 1 open (1 in_progress)
  - YB-2-dashboard (TASK, in_progress)
Branch: feature/YB-2 (clean, up to date)

## On Hold

| Project    | Reason                 |
|------------|------------------------|
| YourBench  | Job search priority    |
| IRL Social | Waiting on API access  |

## Needs Attention
- Coordinatr TASK-002 blocked (waiting on TASK-001)
- Coordinatr SPIKE-003 past 3-day timebox
- CareerBrain feature/YB-2 has 3 unpushed commits

## Suggested Next Actions
1. Complete Coordinatr TASK-001 to unblock TASK-002
2. Push CareerBrain feature/YB-2
3. Close or extend Coordinatr SPIKE-003
```

**Triggers on:** "project status", "show projects", "what projects"
**Use when:** Quick check on all projects, session start context, weekly reviews, seeing what needs attention.

#### 📕 `/project-next-step`

**Assess project readiness and recommend the next skill to run**

Phase-aware validation: checks what files should exist based on current project phase. Suggests next skill to run based on current state.

```bash
/project-next-step coordinatr        # What's next for this project?
/project-next-step yourbench         # Check another project
```

**`/project-next-step` as bridge to the pipeline:**

```
"What should I do next with this project?"
                ↓
       /project-next-step <project>
                ↓
       Suggests next skill based on current state:
       ├─ Just README?       →  /project-brief
       ├─ Has brief?         →  /project-critique
       ├─ Has critique?      →  /research-deep
       ├─ Has research?      →  /project-spec
       └─ Has specs?         →  /project-issue + /project-plan
```

**Use when:** Unsure what to do next on a project, checking if documentation is complete.

#### 📕 `/project-brief`

**Create or update project brief through interactive discovery**

Interactive 6-phase conversation: Problem Discovery, Solution Exploration, Audience Definition, Feature Prioritization, Differentiation, Success Metrics.

```bash
/project-brief                           # Interactive discovery for current or new project
/project-brief --project coordinatr      # Brief for specific project
/project-brief --review                  # Analyze existing brief (no edits)
/project-brief --force                   # Start from scratch
```

**Creates:** `06 Projects/[project]/project-brief.md`
**Use when:** Starting any new idea, revisiting after time away, or when vision feels unclear.

#### 📕 `/project-critique`

**Challenge idea assumptions with skeptical VC-style evaluation**

Analyzes 7 areas: Problem Validity, Market Opportunity, Solution Fit, Differentiation, Business Model, Execution Risk, Technical Feasibility. Produces a verdict (strong / promising / needs_work / reconsider).

```bash
/project-critique                        # Critique current project
/project-critique --project coordinatr   # Specific project
/project-critique --focus market         # Market validation focus
/project-critique --focus technical      # Technical feasibility focus
/project-critique --focus business       # Business model focus
```

**Creates:** `06 Projects/[project]/critiques/YYYY-MM-DD.md`
**Requires:** `project-brief.md` must exist.
**Use when:** Before significant time investment to validate the idea is worth pursuing.

#### 📕 `/project-init-space`

**Initialize project planning structure in 06 Projects/ with docs, specs, and ADRs**

Creates standard planning structure: README.md, project-brief.md, docs/architecture.md, docs/specs/, docs/adrs/. Code repos live at their actual paths (e.g., `/Users/duncanleung/Develop/[project]/`) and are referenced via the Projects Index `code:` field.

```bash
/project-init-space coordinatr              # Initialize existing idea
/project-init-space new-project --type next # Next.js project
/project-init-space myapp --type python     # Python project
```

**Creates:** `06 Projects/[project]/` directory with standard planning + docs structure.
**Use when:** After creating a brief and before implementation.

#### 📕 `/project-spec`

**Manage protocol/standard specifications that define what a system must do**

Import external specs (LEAF, OAuth, etc.) or create self-authored protocol specs. Specs are the "source of truth" contract — TASKs link to spec sections via the `implements:` field. Tracks compliance with inline status markers: ⏳ pending, 🚧 in progress, ✅ done.

```bash
/project-spec                            # Show current project's spec status
/project-spec --import <url>             # Import external spec (GitHub, raw URL)
/project-spec --init                     # Create new protocol spec for project
/project-spec --sync                     # Sync imported spec with upstream
/project-spec --section <name>           # Show specific section of spec
```

**Creates:** `06 Projects/[project]/docs/specs/` with spec files.
**Use when:** Defining requirements before implementation, importing external standards.

#### 📕 `/project-adr`

**Create Architecture Decision Records through interactive conversation**

Interactive 6-question conversation documenting technology choices, architecture patterns, and third-party selections. ADRs live in `06 Projects/[project]/docs/adrs/`.

```bash
/project-adr                                      # Start conversation
/project-adr "database selection for coordinatr"  # Provide topic
```

**Creates:** `06 Projects/[project]/docs/adrs/ADR-###-title.md`
**Use when:** Making technology/framework selections, architecture patterns, third-party service decisions.

#### 📕 `/project-ui-design`

**Create HTML UI mockups stored in 06 Projects/[project]/docs/ui-designs/**

Generates self-contained HTML prototypes with embedded CSS/JS. Supports parallel variant exploration and approval workflow. Approved designs are referenced by TASKs during implementation.

```bash
/project-ui-design yourbench "login screen"
/project-ui-design yourbench "dashboard" --variants 3
/project-ui-design coordinatr "project list" --tech shadcn
/project-ui-design yourbench list                  # Show existing designs
```

**Creates:** `06 Projects/[project]/docs/ui-designs/[name]-v[N].html`
**Use when:** Designing interfaces before implementation, exploring UI variations.

#### 📕 `/project-issue`

**Create standalone work items (TASK, BUG, or SPIKE) with AI-assisted type detection**

Auto-detects type from description. Links to spec sections via `implements:` field for traceability. Auto-numbers sequentially (001, 002, 003...).

```bash
/project-issue                                    # Start conversation
/project-issue "Implement authentication"         # AI detects: TASK
/project-issue "Compare GraphQL vs REST"          # AI detects: SPIKE
/project-issue "Broken link in project-brief"     # AI detects: BUG
/project-issue --project coordinatr               # Create for specific project
```

**Creates:** `06 Projects/[project]/issues/###-name/TASK.md` (or BUG.md, SPIKE.md)
**Use when:** Need to track work, exploration needed, or something is broken.

#### 📕 `/project-plan`

**Create PLAN.md file with phase-based breakdown for issues**

Breaks work into logical phases with checkpoints. Validates library documentation with Context7 (mandatory for integrations). Optional `--second-opinion` for Gemini CLI peer review.

```bash
/project-plan 001                        # Create plan for issue 001
/project-plan 001 --project coordinatr   # Explicit project
/project-plan SPIKE-003                  # Plan for a spike
/project-plan 003 --second-opinion       # Get peer review from Gemini CLI
```

**Creates:** `06 Projects/[project]/issues/###-name/PLAN.md`
**Requires:** Issue file (TASK.md / BUG.md / SPIKE.md) must exist.
**Use when:** After creating an issue, to break down work into executable phases.

#### 📕 `/project-implement`

**Execute plan phases, writing code in project code repo while tracking in 06 Projects/**

Coordinates specialist agents based on phase type (frontend, backend, database, tests). Creates feature branches, manages issue status, writes worklog entries automatically.

```bash
/project-implement yourbench YB-2 1.1     # Execute phase 1.1 of issue YB-2
/project-implement yourbench YB-2 --next  # Auto-find next uncompleted phase
/project-implement yourbench --next       # Auto-detect issue + next phase
/project-implement coordinatr 003 --full  # Execute all remaining phases
```

**Requires:** PLAN.md must exist.
**Use when:** Ready to write code from a plan.

#### 📕 `/project-advise`

**Interactive conversational guidance — you implement with step-by-step advice**

Explains WHAT to do, answers questions, checks your work when you say "done." Task-focused, medium-paced.

```bash
/project-advise 001              # Guide through issue 001
/project-advise yourbench 001    # Explicit project
/project-advise 001 --phase 2.1  # Start at specific phase
```

**Use when:** Know the stack, need guidance on THIS task, want to maintain control.

#### 📕 `/project-teach`

**Deep pedagogical guidance — learn technology by doing with Socratic teaching**

Line-by-line explanations (1-3 lines at a time) with WHY not just HOW. Frequent comprehension checks, first principles, analogies.

```bash
/project-teach 001              # Learn through issue 001
/project-teach yourbench 001    # Explicit project
/project-teach 001 --phase 2.1  # Start at specific phase
```

**Use when:** Learning new framework, want to understand deeply WHY not just HOW.

**Implementation mode comparison:**

| Aspect    | `/project-implement` | `/project-advise` | `/project-teach`    |
| --------- | -------------------- | ----------------- | ------------------- |
| Who codes | AI                   | You               | You                 |
| Speed     | Fast                 | Medium            | Slower              |
| Depth     | Task completion      | Task guidance     | Conceptual learning |

#### 📕 `/project-worklog`

**Add timestamped work log entries to track progress and decisions**

JSON-based entries with types: manual, decision, gotcha, handoff, phase, blocker. Called automatically by `/project-implement`, or manually during `/project-advise` and `/project-teach`.

```bash
/project-worklog yourbench YB-2 "Added login button"
/project-worklog yourbench YB-2 --decision "Using Clerk for auth"
/project-worklog yourbench YB-2 --gotcha "Token refresh needs cleanup"
/project-worklog coordinatr 003 --handoff code-reviewer "Ready for review"
/project-worklog yourbench YB-2 --state              # Show current state
```

**Creates:** `06 Projects/[project]/issues/###-name/worklog/` entries.
**Use when:** Documenting work, decisions, gotchas, agent handoffs.

#### 📕 `/project-commit`

**Create git commits in project code repo with conventional message format**

Auto-detects changes and commit type (feat/fix/docs/refactor/test/chore). Runs quality checks (tests, lint) unless `--no-verify`. Handles both project code repo (from Projects Index) and meta-repo (06 Projects/) commits.

```bash
/project-commit yourbench                     # Interactive commit
/project-commit yourbench "feat: add auth"    # Direct with message
/project-commit coordinatr --amend            # Amend last commit (safety checks)
/project-commit yourbench "fix: typo" --no-verify  # Skip hooks
```

**Use when:** Saving progress with quality checks and proper commit messages.

#### 📕 `/project-complete`

**Complete task: validate, document, review, commit, and merge to develop**

Full completion pipeline: validates PLAN completion, checks spec compliance, updates all documentation (specs, architecture, CHANGELOG), runs code review + security audit, merges feature branch to develop.

```bash
/project-complete careerbrain 002         # Complete specific issue
/project-complete careerbrain             # Complete current/active issue
```

**Requires:** PLAN.md with phases mostly complete, feature branch.
**Use when:** All implementation is done and ready to finalize.

#### Quality Checkpoints

Quality skills fire at specific moments in the project pipeline — not a separate sequence.

```
PROJECT PHASE                QUALITY CHECKPOINT
─────────────                ──────────────────
/project-init-space     ───→ /project-validate-space

/project-implement      ───→ /project-validate-sanity-check   (anytime, on-demand)
       │                     /project-validate-troubleshoot    (when bugs occur)
       ↓
                        ┌──→ /project-validate-quality        ┐
/project-complete     ←─┤    /project-validate-security-audit  ├ run in parallel
                        └──→ /project-validate-spec           ┘
```

#### 📕 `/project-validate-space`

**Validate project space structure, boilerplate docs, and consistency with 06 Projects/**

Checks 06 Projects/[project]/ for required files, docs structure, version consistency, and cross-references with the code repo.

```bash
/project-validate-space leaf-nextjs-convex   # Validate specific space
/project-validate-space coordinatr           # Another project
```

**Use when:** After scaffolding, before implementation, after dependency upgrades, monthly maintenance.

#### 📕 `/project-validate-sanity-check`

**Step back, reflect on current work, validate direction and alignment**

Uses sequential thinking to reflect. Checks alignment with project brief and critique. Categorizes findings as green/yellow/red.

```bash
/project-validate-sanity-check                        # Reflect on current work
/project-validate-sanity-check --project coordinatr   # Focus on specific project
```

**Use when:** Complexity increasing, feeling uncertain, before major decisions, something feels off.

#### 📕 `/project-validate-troubleshoot`

**Systematic debugging with 5-step loop**

Research → Hypothesize → Implement → Test → Document. One hypothesis at a time. Liberal debug logging. Documents findings in WORKLOG.

```bash
/project-validate-troubleshoot yourbench "tests failing after auth changes"
/project-validate-troubleshoot yourbench 001       # Debug in context of issue
/project-validate-troubleshoot coordinatr          # General debugging session
```

**Use when:** Encountering bugs, test failures, or unexpected behavior.

#### 📕 `/project-validate-spec`

**Validate spec completeness and implementation compliance**

Pre-implementation mode: checks structure, scenarios, metrics. Post-implementation mode: verifies test coverage and spec compliance.

```bash
/project-validate-spec SPEC-001              # Auto-detect mode
/project-validate-spec SPEC-001 --pre        # Pre-implementation validation
/project-validate-spec SPEC-001 --post       # Post-implementation compliance
```

**Use when:** Before approving a spec or before `/project-complete` to verify compliance.

#### 📕 `/project-validate-quality`

**Comprehensive code quality assessment in project code repo**

Multi-agent analysis (code, security, performance, testing). Produces scores (XX/100) per dimension with prioritized actions.

```bash
/project-validate-quality yourbench                  # Full assessment
/project-validate-quality yourbench --focus security # Security-focused
/project-validate-quality coordinatr --focus testing # Test coverage focus
```

**Use when:** Before commits, merges, or releases to ensure consistent quality.

#### 📕 `/project-validate-security-audit`

**Comprehensive security audit of codebase using multiple agents**

5 agents run in parallel (Auth, Input, Crypto, Config, Dependencies). OWASP Top 10 coverage. Produces timestamped audit report.

```bash
/project-validate-security-audit yourbench           # Full security review
/project-validate-security-audit coordinatr          # Audit specific project
```

**Use when:** Before production deployments, after major features, monthly reviews.

---

### 🛠️ Learning Skills

Structured learning with spaced repetition. Session skills follow a strict lifecycle; post-session skills are independent.

```
┌─────────────────────────────────────────────────────────────┐
│  SESSION LIFECYCLE (strict order)                           │
│                                                             │
│  /learn-start-session "topic"                               │
│       ↓                                                     │
│  /learn-log-session  (0+ times during session)              │
│       ↓                                                     │
│  /learn-end-session                                         │
└──────────────────────────┬──────────────────────────────────┘
                           ↓  (days/weeks later)
┌─────────────────────────────────────────────────────────────┐
│  POST-SESSION (any order, independent)                      │
│                                                             │
│  /learn-review-session  · /learn-study-notes                │
│  /learn-flashcards                                          │
└─────────────────────────────────────────────────────────────┘
```

**Shared state:** `.claude/learning-sessions/learning-plan.json` tracks proficiency per topic and schedules reviews (7d / 14d / 30d / 90d based on proficiency level).

| Stage         | Skill                    | Purpose                          | Requires                   | Args                           |
| ------------- | ------------------------ | -------------------------------- | -------------------------- | ------------------------------ |
| Session start | `/learn-start-session`   | Begin learning session           | —                          | `"topic" or /path/to/notes.md` |
| Mid-session   | `/learn-log-session`     | Log mid-session progress         | Active session             |                                |
| Session end   | `/learn-end-session`     | Complete session with summary    | Active session             |                                |
| Post-session  | `/learn-review-session`  | Test retention                   | Previous session on topic  | `"topic"`                      |
|               | `/learn-study-notes`     | Create comprehensive notes       | —                          | `"topic" ["dest path"]`        |
|               | `/learn-flashcards`      | Generate spaced repetition cards | —                          | `<file or topic>`              |


#### 📕 `/learn-start-session`

**Start a new learning session on a topic**

Checks for topics due for review. Conducts retrieval warm-up if topic covered before. Creates session file in `.claude/learning-sessions/`.

```bash
/learn-start-session "AWS EFS"
/learn-start-session /path/to/notes.md
```

**Creates:** `.claude/learning-sessions/YYYY-MM-DD-NNN.json`
**Triggers on:** "teach me", "let's learn", "start session", "study [topic]"

#### 📕 `/learn-log-session`

**Log entries to the current learning session**

Saves progress mid-session. Captures concepts, corrections, misconceptions, elaborations, connections. Requires an active session.

```bash
/learn-log-session
```

**Requires:** Active session from `/learn-start-session`.
**Triggers on:** "log this", "save progress"

#### 📕 `/learn-end-session`

**End the current learning session**

Generates summary. Updates `learning-plan.json` with proficiency adjustments. Suggests next topic from queue.

```bash
/learn-end-session
```

**Requires:** Active session from `/learn-start-session`.
**Triggers on:** "end session", "done learning", "finish studying"

#### 📕 `/learn-review-session`

**Retrieval practice session to test retention on a topic**

Tests what you remember (NOT teaching). Generates questions from past session entries and misconceptions. Logs confidence vs accuracy. Calculates retention score and may adjust proficiency.

```bash
/learn-review-session "AWS EFS"
```

**Requires:** At least one previous teaching session on the topic.
**Triggers on:** "quiz me", "test my knowledge", "what do I remember"

#### 📕 `/learn-study-notes`

**Create comprehensive study notes on a topic**

Well-structured notes with tables, callouts, examples. Links to existing vault notes. Includes flashcard-ready tags.

```bash
/learn-study-notes "AWS EFS" "07 Knowledge Base/AWS/"
```

**Triggers on:** "study notes", "create notes", "document [topic]"

#### 📕 `/learn-flashcards`

**Generate spaced repetition flashcards from notes or topics**

Supports multiple formats for the Obsidian Spaced Repetition plugin: `::` single-sided, `:::` bidirectional, `;;` multi-line, `==text==` cloze deletions. Asks where to add cards (inline, section, or separate file).

```bash
/learn-flashcards /path/to/notes.md
/learn-flashcards "AWS EFS concepts"
```

**Triggers on:** "flashcards", "make cards", "spaced repetition"

---

### 🛠️ Research Skills

Independent entry points for deep research, batch content catchup, and knowledge synthesis. All source skills are independent; `/research-synthesize` aggregates notes from any of them. `/research-clip` provides quick capture for later processing.

```
CAPTURE                                  SOURCES (batch catchup)                  SYNTHESIS
───────                                  ──────────────────────                   ─────────
/research-clip                           /research-deep ─────────────────┐
  ├─ Article URL → Clipped (quick)       /research-rss-catchup ──────────┼────→ /research-synthesize
  ├─ YouTube URL → Summarized (full)     /research-youtube-catchup ──────┘
  └─ /research-clip-read → Read
```

| Stage     | Skill                       | Purpose                                              | Args                                                   |
| --------- | --------------------------- | ---------------------------------------------------- | ------------------------------------------------------ |
| Capture   | `/research-clip`            | Capture article (Clipped) or YouTube video (Summarized) | `"title" [--url URL] [--tags t1,t2] [--why "reason"]`  |
|           | `/research-clip-read`       | Process clip — extract insights, mark Read           | `[clip name] [--content '{...}'] [--skip-fetch]`       |
| Deep dive | `/research-deep`            | Deep research with docs                              | `"topic"`                                              |
| Catchup   | `/research-rss-catchup`     | Summarize RSS feeds (batch)                          |                                                        |
|           | `/research-youtube-catchup` | Summarize YouTube channels (batch)                   |                                                        |
| Synthesis | `/research-synthesize`      | Synthesize across multiple sources                   | `<topic> \| new "topic" \| add <topic> <path> \| list` |

#### 📕 `/research-deep`

**Deep research on a topic, creating persistent documentation**

Creates comprehensive research documents (20-30+ sources distilled to 3-5 pages). Uses research-specialist agent with Context7 and WebSearch.

```bash
/research-deep "authentication patterns for multi-tenant SaaS"
/research-deep "Jira vs Linear competitive analysis"
/research-deep "WebSocket vs SSE trade-offs"
```

**Output:** `06 Projects/[project]/notes/research/`, `SHARED/DOCS/research/`, or `resources/research/`
**Use when:** Technology decisions, competitive analysis, complex topics requiring deep investigation.

#### 📕 `/research-rss-catchup`

**Fetch and summarize latest articles from RSS feeds**

Processes feeds since last run. Creates article notes with bullet summaries. Extracts discovery notes for mentioned tools/products. Deduplicates against existing notes.

```bash
/research-rss-catchup  # Process new articles from feeds
```

**Config:** `.claude/skills/research-rss-catchup/references/feeds.json`
**Output:** `01 Inbox/Articles/{Feed Name}/{Title}.md`
**Triggers on:** "rss catchup", "blog catchup", "check feeds"

#### 📕 `/research-youtube-catchup`

**Fetch and summarize latest videos from priority YouTube channels**

Uses coordinator + Haiku subagents pattern. Processes 4-6 videos in parallel. Downloads transcripts and creates summary notes with discoveries.

```bash
/research-youtube-catchup  # Process new videos from configured channels
```

**Config:** `.claude/skills/research-youtube-catchup/references/channels.json`
**Output:** `01 Inbox/Videos/{channel_folder}/{title}.md`
**Triggers on:** "youtube catchup", "video catchup", "check youtube"

#### 📕 `/research-synthesize`

**Synthesize information across multiple sources into structured documents**

Coordinator + Haiku extractors for parallel processing (6-10 sources per batch). Auto-discovers related sources from citations. Enforces 300-line limit (suggests sub-topics for deeper dives).

```bash
/research-synthesize adhd                      # Continue processing
/research-synthesize list                      # Show all syntheses
/research-synthesize new "ADHD Medications"    # Create new
/research-synthesize add adhd path/to/note.md  # Add sources
/research-synthesize update adhd               # Scan for new sources
```

**Output:** `07 Knowledge Base/Synthesis/[Topic]/`
**Triggers on:** "synthesize", "synthesis", "research [topic]"

#### 📕 `/research-clip`

**Capture articles and videos — auto-summarizes YouTube**

For articles: fetches page, extracts key insights, saves with `status: Clipped`. For YouTube URLs: fetches transcript, creates full summary with `status: Summarized`. Auto-detects type from URL.

```bash
/research-clip "Terraform Best Practices"
/research-clip "React 19 Overview" --url https://youtube.com/watch?v=abc
/research-clip "RSC Deep Dive" --url https://example.com/rsc --tags react,nextjs --why "Need for Portal migration"
```

**Output:** `01 Inbox/Videos/Unsorted/` (YouTube) or `01 Inbox/Articles/Unsorted/` (everything else)
**Triggers on:** "clip this", "save for later", "bookmark", "capture"

#### 📕 `/research-clip-read`

**Process a clipped resource — extract insights, mark Read**

Finds clip by name or partial match. Fetches source content (or accepts `--content` for blocked sites like Reddit). Extracts key insights into the Summary section. Updates `status: Clipped` → `Read`.

```bash
/research-clip-read terraform-best-practices
/research-clip-read terraform                    # partial match
/research-clip-read claude-obsidian --content '{"title": "...", "body": "..."}'
/research-clip-read                              # interactive: list clipped items
```

**Triggers on:** "clip read", "mark read", "read clip", "process clip"

---

### 🛠️ Operations Skills

Independent maintenance tasks. No fixed order — run as needed.

| Stage  | Skill                | Purpose                            | Args                                                             |
| ------ | -------------------- | ---------------------------------- | ---------------------------------------------------------------- |
| Triage | `/ops-process-inbox` | Process inbox notes                |                                                                  |
| Health | `/ops-docs`          | Documentation health check + README validation | `[--health \| --validate \| --stale \| --sync \| --readmes] [--project name]` |
| Sync   | `/ops-git-sync`      | Sync all git repos                 | `[--pull \| --push \| --clone]`                                  |

#### 📕 `/ops-process-inbox`

**Process notes in inbox folder**

Helps organize captured notes in `01 Inbox/`. Suggests destinations based on content. Processes one note at a time.

```bash
/ops-process-inbox  # Process 01 Inbox/
```

**Triggers on:** "process inbox", "organize notes", "inbox zero"

#### 📕 `/ops-docs`

**Documentation health check, README validation, and maintenance across all ideas**

Health scoring by project. Validates links. Finds stale documents (>30 days). Can sync CLAUDE.md with current project state. Includes README-specific validation mode.

```bash
/ops-docs --health                   # Overall health report
/ops-docs --validate                 # Check broken links
/ops-docs --stale                    # Find outdated docs
/ops-docs --sync                     # Sync CLAUDE.md with project state
/ops-docs --readmes                  # Review and update all READMEs
/ops-docs --project coordinatr       # Focus on specific project
```

**Use when:** Periodic maintenance, finding gaps, keeping docs accurate, after major updates, after status changes in CLAUDE.md.

#### 📕 `/ops-git-sync`

**Sync all git repos — meta-repo and project code repos (from Projects Index)**

Checks status of all repos and branches. Safe operations only (no force push, no auto-commit). Reports ahead/behind/dirty/diverged status.

```bash
/ops-git-sync              # Check status
/ops-git-sync --pull       # Pull updates
/ops-git-sync --push       # Push local commits
/ops-git-sync --clone      # Clone missing repos
```

---

## 🛠️ Workflow Templates

Reference workflows combining multiple skills. Located in `.claude/skills/workflows/`. Not executable — use as guides.

| Workflow              | Purpose                                                                  |
| --------------------- | ------------------------------------------------------------------------ |
| `new-project`         | Full lifecycle: concept to first commit (15 steps across 5 phases)       |
| `bug-fix`             | Systematic: troubleshoot → issue → plan → implement → quality → commit   |
| `pre-merge-checklist` | Parallel quality checks (quality + security + docs + spec) then complete |

### New Project Flow

```
/project-brief → /project-critique → /research-deep → /project-spec
→ /project-adr → /project-issue → /project-plan → /project-implement
→ /project-validate-quality → /project-validate-security-audit → /project-complete
```

### Bug Fix Flow

```
/project-validate-troubleshoot → /project-issue → /project-plan → /project-implement
→ /project-validate-quality → /project-commit
```

### Lean Startup (skip some steps)

```
/project-brief → /project-spec → /project-issue → /project-implement → /project-commit
```

### Learning-Focused (you write code)

```
/project-brief → /project-spec → /project-issue → /project-plan
→ /project-teach → /project-worklog → /project-validate-quality
```

### Content Processing Flow

```
/research-clip "article" --url ...  → [status: Clipped]  → /research-clip-read → [status: Read]
/research-clip "video" --url yt...  → [status: Summarized]  (auto from transcript)

/research-youtube-catchup → [batch summaries] → /research-synthesize add → /research-synthesize [topic]
/research-rss-catchup     → [batch summaries] → /research-synthesize add → /research-synthesize [topic]
```

---

## Architecture Patterns

### Coordinator + Subagent Pattern

Some skills use parallel subagents for ~3x throughput:

- **/research-synthesize**: Haiku extractors process 6-10 sources in parallel
- **/research-youtube-catchup**: Haiku summarizers handle 4-6 videos in parallel
- **/project-implement**: Specialist agents for different phase types (frontend, backend, database)
- **/project-validate-quality**: Multiple agents analyze different quality dimensions
- **/project-validate-security-audit**: 5 security agents cover different OWASP areas in parallel

### Agent Specialization

Skills invoke specialized agents based on task type:

- **research-specialist**: Deep research with 20-30+ sources
- **brief-strategist**: Interactive project discovery
- **idea-critic**: Skeptical VC-style evaluation
- **ui-ux-designer**: Interface mockup generation
- **frontend-specialist**, **backend-specialist**, **database-specialist**: Domain implementation
- **code-reviewer**, **security-auditor**, **test-engineer**: Quality assurance

---

## Best Practices

1. **Start with `/project-brief`** for new projects before diving into implementation
2. **Run `/project-critique`** before significant time investment to validate ideas
3. **Use `/project-spec`** to define requirements before creating tasks
4. **Link issues to specs** via `implements:` field for traceability
5. **Document as you go** with `/project-worklog` entries
6. **Run quality checks** (`/project-validate-quality`, `/project-validate-security-audit`) before completion
7. **Complete properly** with `/project-complete` to update all documentation
8. **Review regularly** — `/journal-daily-review`, `/journal-weekly-review` for reflection
9. **Learn actively** — `/learn-start-session` with retrieval warm-ups
10. **Synthesize knowledge** — use `/research-synthesize` for deep understanding of topics

---

## Configuration Files

Skills read from several configuration files:

| File                                                               | Purpose                                             |
| ------------------------------------------------------------------ | --------------------------------------------------- |
| `.claude/skills/[skill]/SKILL.md`                                  | Skill definitions                                   |
| `.claude/skills/[skill]/references/`                               | Skill-specific reference docs and config            |
| `.claude/learning-sessions/learning-plan.json`                     | Learning system state (proficiency, queue, reviews) |
| `.claude/learning-sessions/index.json`                             | Session tracking index                              |
| `.claude/skills/research-youtube-catchup/references/channels.json` | YouTube channels to follow                          |
| `.claude/skills/research-youtube-catchup/references/state.json`    | Processed video tracking                            |
| `.claude/skills/research-rss-catchup/references/feeds.json`        | RSS feeds to follow                                 |
| `.claude/skills/research-rss-catchup/references/state.json`        | Processed article tracking                          |
| `.claude/skills/research-synthesize/references/queues/`            | Synthesis topic queues                              |
| `.claude/obsidian-memories/about-me.md`                            | User profile and context                            |
| `.claude/obsidian-memories/index.json`                             | Memory tracking index                               |

---

## Philosophy

### Planning vs Code

- `06 Projects/` contains all project documentation: briefs, specs, ADRs, issues, architecture
- Code repos live at their actual paths (e.g., `/Users/duncanleung/Develop/[project]/`), referenced via the Projects Index `code:` field
- Skills READ from code repos but NEVER WRITE to them

### Memory is Persistent

Claude learns your preferences over time via `.claude/obsidian-memories/`. The AI will proactively capture:

- Preferences you express
- Corrections you make
- Workflow insights
- Project decisions

### Skills are Workflows

Skills aren't just prompts — they're structured multi-step processes with:

- Clear triggers and entry points
- Reference materials
- Output formats
- Integration with your vault and repos

## Maintenance

- Update `CLAUDE.md` Projects Index when adding/archiving projects
- Review and prune `.claude/obsidian-memories/` periodically
- Keep your vault's daily notes current
- Update skill configs as your feeds/channels change

## Limitations and Future Plans

- It's entirely reliant on my workflows and Obsidian setup, I'd like to generalize it more.
- It's entirely based on Claude Code, I'd like to generalize it to work better with any LLM, including local ones, possible folding in my [Local Ollama Chatbot experiment](https://github.com/TaylorHuston/ollama-chat).
- The YouTube and RSS catchup skills both just store their state in a giant JSON file, which probably won't scale.
- As the size of the vault grows I might look into adding a RAG to help the LLM search through it easier.

## License

MIT
