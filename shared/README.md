# Shared Resources

Cross-project standards, templates, and documentation.

## Structure

```
SHARED/
├── DOCS/           # Cross-project standards and guides
├── TEMPLATES/
│   ├── PM/         # Issue templates (TASK, BUG, SPIKE, PLAN, WORKLOG, SPEC)
│   ├── DOCS/       # Documentation templates (API, architecture, etc.)
│   └── IDEA-MINIMAL/  # New project scaffolding
```

## DOCS/

Standards that apply across all projects:

- **branching-strategy.md** - Git workflow and branch naming
- **development-loop.md** - How to work on issues
- **style-guide.md** - Code style standards
- **tech-stack-decisions.md** - Common technology choices
- **pricing-philosophy.md** - Pricing approach for products

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
