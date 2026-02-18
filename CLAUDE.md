# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a life management hub - a unified repo for personal knowledge, project planning, and AI-assisted life management.

## Directory Structure

```
root/
├── .claude/                    # AI configuration
│   ├── skills/                 # Custom skills (project + personal workflows)
│   ├── obsidian-memories/       # Persistent context (create this)
│   ├── learning-sessions/      # Learning progress tracking (create this)
│   ├── agents/                 # Agent definitions
│   └── docs/                   # Skill system documentation
├── 01 Inbox/ … 08 System/     # Obsidian vault directories (gitignored)
├── 06 Projects/[project]/      # Project planning, docs, specs, ADRs
├── 07 Knowledge Base/          # Cross-project research, knowledge dumps, general reference
└── SHARED/                     # Cross-project templates
```

| Directory | Purpose | Git Status |
|-----------|---------|------------|
| `.claude/` | Skills, obsidian-memories, learning sessions | Tracked |
| `01-08 dirs` | Obsidian vault (Inbox, Calendar, Knowledge Base, etc.) | Gitignored |
| `06 Projects/` | Project planning, documentation, specs, ADRs | Tracked |
| `07 Knowledge Base/` | Cross-project research, knowledge dumps, general reference | Gitignored |
| `SHARED/` | Cross-project templates | Tracked |

Code repositories live at their actual paths (e.g., `/Users/duncanleung/Develop/[project]/`) and are referenced via the Projects Index `code:` field. Skills READ from code repos but NEVER WRITE to them — all documentation goes to `06 Projects/`.

## Getting Started

1. **Create your obsidian-memories directory**: `mkdir -p .claude/obsidian-memories`
2. **Create about-me file**: `.claude/obsidian-memories/about-me.md` with your context
3. **Create memories index**: `.claude/obsidian-memories/index.json` (empty array to start)
4. **Configure personal skills**: Update RSS feeds, YouTube channels, etc.

## Claude Data

- **About You**: `.claude/obsidian-memories/about-me.md` - Summary of who you are
- **Memories**: `.claude/obsidian-memories/` - JSON entries of important things learned
- **Learning Sessions**: `.claude/learning-sessions/` - Learning progress tracking
- **Skills**: `.claude/skills/` - Custom skills for project and personal workflows

---

## Project Structure

### Project Structure (`06 Projects/`)

```
06 Projects/[project]/
├── README.md                              # Project index with Initiatives table
├── project-brief.md                       # Vision, problem, audience
├── YYYY-MM-DD-initiative-name/            # Initiative folder (created by /project-issue)
│   ├── TASK.md|BUG.md|SPIKE.md           # Work item
│   ├── PLAN.md                            # Phase breakdown
│   ├── WORKLOG.md                         # Progress tracking
│   ├── context.md                         # AI session handoff
│   ├── critique.md                        # Critique (if applicable)
│   ├── spec-*.md                          # Specs (if applicable)
│   ├── ADR-###-*.md                       # Architecture decisions
│   └── ui-*.html                          # UI mockups
```

**Convention:** No intermediate directories (`issues/`, `docs/`, `critiques/`). All work is scoped under dated initiative folders created on-demand by `/project-issue`. Only `README.md` and `project-brief.md` live at the project root.

### Code Repositories (external)

Code repos live at their actual paths (e.g., `/Users/duncanleung/Develop/[project]/`). The `code:` field in the Projects Index maps project names to repo paths. Skills may READ from code repos to understand the codebase, but all documentation is written to `06 Projects/[project]/`.

### Required Files (Every Project)

| File | Purpose | Requirements |
|------|---------|--------------|
| **README.md** | Status overview, quick links | YAML frontmatter (status, created, updated) |
| **project-brief.md** | Vision, problem, solution | Complete sections: Vision, Problem, Solution, Audience, Scope |

---

## Skill System

Six skill categories, organized by purpose. Full catalog with every skill: `.claude/skills/README.md`. Detailed usage guide with args and examples: `README.md`.

| Prefix | Category | What it covers |
|--------|----------|----------------|
| `journal-*` | Personal/Journal | Daily/weekly journal rhythm, task management, activity pulls (GitHub, Slack), meeting capture, personal triage |
| `session-*` | Session Management | AI context refresh, context handoff, end-of-session debrief, process improvement |
| `project-*` | Project Workflow | Full pipeline: brief -> critique -> spec -> issue -> plan -> implement -> validate -> commit -> complete |
| `learn-*` | Learning | Session lifecycle (start -> log -> end), review sessions, study notes, flashcards |
| `research-*` | Research & Content | Deep research, RSS/YouTube catchup, clip capture, cross-source synthesis |
| `ops-*` | Operations | Git sync, documentation health, inbox processing |

---

## Projects Index

<!-- PROJECTS_INDEX_START -->
```yaml
# Status values: active, on-hold, maintained, archived, experiment
# code: points to actual repo path (skills READ from here, never write)
# planning: points to 06 Projects/ directory (skills WRITE docs here)
projects:
  # Add your projects here
  # - name: my-project
  #   planning: 06 Projects/my-project/
  #   code: /Users/duncanleung/Develop/my-project/
  #   remote: https://github.com/you/my-project
  #   status: active
```
<!-- PROJECTS_INDEX_END -->

---

## Core Rules

1. **All project docs in 06 Projects/**: Briefs, issues, plans, specs, ADRs, architecture → `06 Projects/[project]/`.
2. **Knowledge dumps and research go to 07 Knowledge Base/**: Cross-project research, knowledge dumps, general reference material → `07 Knowledge Base/`. NEVER save to `SHARED/DOCS` (deleted).
3. **Code repos are external**: Referenced via `code:` in Projects Index. Skills READ from code repos but NEVER WRITE to them.
3. **Precedence**: A project's own `CLAUDE.md` (in its code repo) overrides this file.
4. **Never commit unless explicitly asked** or it's part of a prescribed workflow.
5. **Always use absolute file paths** when referencing code in task notes, issues, plans, or any documentation. Use full paths (e.g., `/Users/duncanleung/Develop/airvet/go-airvet/pkg/chat/chat.go:765`) so the next LLM session can jump straight to the files.

---

## Customization

### Personal Skills to Configure

These skills have placeholder configs you should customize:

- **`/research-rss-catchup`**: Add your RSS feeds to `.claude/skills/research-rss-catchup/references/feeds.json`
- **`/research-youtube-catchup`**: Add your channels to `.claude/skills/research-youtube-catchup/references/channels.json`

### About You

Create `.claude/obsidian-memories/about-me.md` with sections like:
- Profile (name, role, expertise)
- Communication preferences
- Current focus/goals
- Key accounts and links

---

## Maintenance

- Update `.claude/obsidian-memories/about-me.md` as preferences change
- Update project READMEs when status changes
- Keep Projects Index current
