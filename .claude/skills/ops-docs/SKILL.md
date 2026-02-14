---
name: ops-docs
description: "Documentation health check, README validation, and maintenance across all projects. Use for periodic maintenance, finding gaps, keeping docs accurate, or checking README consistency."
model: claude-haiku-4-5-20251001
allowed-tools: Read, Edit, Glob, Grep
---

# /ops-docs

Documentation health check, validation, and maintenance.

## Usage

```bash
/ops-docs --health                   # Overall health report
/ops-docs --validate                 # Check broken links, missing files
/ops-docs --stale                    # Find documents >30 days old
/ops-docs --sync                     # Sync CLAUDE.md with project state
/ops-docs --readmes                  # Review and update all READMEs
/ops-docs --project coordinatr       # Focus on specific project
```

## Flags

| Flag | Purpose |
|------|---------|
| `--health` | Comprehensive health report with scores |
| `--validate` | Check links, references, required files |
| `--stale` | Find outdated documentation |
| `--sync` | Update CLAUDE.md with current status |
| `--readmes` | Review and update all README.md files |
| `--project` | Focus on specific project |

## Health Report

```
## Documentation Health Report

### Overall Score: 72/100

### By Project
| Project | Brief | Critique | Specs | Issues | Score |
|---------|-------|----------|-------|--------|-------|
| Coordinatr | ✓ | ✓ | 2 | 3 | 85 |
| YourBench | ✓ | ✗ | 1 | 0 | 60 |

### Recommendations
1. Run /critique on [project]
2. Create project-brief.md for [project]
```

## Validation Report

```
## Validation Report

### Broken Links
- 06 Projects/coordinatr/README.md:15 -> ../specs/SPEC-001.md (not found)

### Missing Required Files
- 06 Projects/irl-social/project-brief.md

### Orphaned Files
- 06 Projects/SHARED/DOCS/old-research.md (not referenced)

### Structure Issues
- 06 Projects/lorecraft/ missing issues/ directory
```

## Health Scoring

| Component | Weight |
|-----------|--------|
| README.md | 20% |
| project-brief.md | 25% |
| critique.md | 15% |
| specs/ | 20% |
| issues/ | 10% |
| Freshness | 10% |

## Execution Flow

### --health
1. Scan all `06 Projects/*/` directories
2. Check for required files
3. Count specs, issues, research docs
4. Calculate health score per project

### --validate
1. Parse markdown files for links
2. Verify link targets exist
3. Check for required structure
4. Identify orphaned files

### --stale
1. Get file modification dates via git
2. Flag files older than threshold
3. Categorize by severity

### --sync
1. Read current CLAUDE.md
2. Scan all 06 Projects/*/README.md for status
3. Compare and identify differences
4. Propose updates

### --readmes
1. Discover all READMEs: `Glob: 06 Projects/**/README.md`
2. Review each README for:
   - **Status accuracy**: Matches CLAUDE.md?
   - **Date accuracy**: Is "Last Updated" current?
   - **Link validity**: Are all links working?
   - **Content completeness**: Required sections present?
   - **Consistency**: Formatting matches standards?
3. Update READMEs:
   - Fix outdated status indicators
   - Update dates to current date
   - Fix broken links
   - Add missing sections
   - Standardize formatting
4. Report results:
   - READMEs reviewed (count)
   - Updates made (list of files)
   - Issues requiring manual attention

**Scope for --readmes:**
- `06 Projects/[project]/README.md` — All project overviews
- `06 Projects/[project]/apps/*/README.md` — Multi-app project READMEs
- Root-level documentation files
- Excludes external code repos (they have their own maintenance)

**Notes:**
- Read then write: reads each file before making changes
- Preserves content: only updates metadata and known sections
- Non-destructive: doesn't remove custom content
- Idempotent: safe to run multiple times

## When to Use

- Weekly documentation review
- Before presenting projects
- After adding/removing projects
- Finding forgotten projects
- After major project updates
- Before project reviews/demos
- After status changes in CLAUDE.md
