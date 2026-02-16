# Shared Resources

Cross-project templates.

## Structure

```
SHARED/
├── TEMPLATES/
│   ├── PM/         # Issue templates (TASK, BUG, SPIKE, PLAN, WORKLOG, SPEC)
│   ├── DOCS/       # Documentation templates (API, architecture, etc.)
│   └── IDEA-MINIMAL/  # New project scaffolding
```

## TEMPLATES/

### PM/ - Project Management

Issue and planning templates:

| Template | Purpose |
|----------|---------|
| `TASK-TEMPLATE.md` | Feature implementation work |
| `BUG-TEMPLATE.md` | Bug fixes |
| `SPIKE-TEMPLATE.md` | Research/exploration |
| `PLAN-TEMPLATE.md` | Phase breakdown for issues |
| `WORKLOG-TEMPLATE.md` | Progress tracking |
| `SPEC-TEMPLATE.md` | Protocol specification |

### DOCS/ - Documentation

Templates for project documentation in `06 Projects/[project]/docs/`:

- API overview
- Architecture overview
- Data model
- Deployment
- Security
- Testing overview
- UI guide

### IDEA-MINIMAL/ - New Project

Minimal scaffolding for new project ideas:

```bash
cp -r SHARED/TEMPLATES/IDEA-MINIMAL "06 Projects/my-new-project"
```

Creates:
- `README.md` - Project status overview
- `PROJECT-BRIEF.md` - Vision, problem, solution
