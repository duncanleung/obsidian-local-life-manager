# Project Discovery

Standard procedure for resolving which `06 Projects/{project}/` directory a skill should read from or write to. Referenced by all project skills that interact with `06 Projects/`.

## When to Use

Any time a skill needs to resolve `[project]` in a path like `06 Projects/[project]/...`.

## Resolution Order

1. **Explicit argument** — If the user provides `--project <name>` or a positional project argument, use that value exactly
2. **Git repo detection** — Otherwise, derive from the current working directory:
   ```bash
   PROJECT=$(basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
   ```
   This works from any subdirectory of a git repo. Falls back to `basename "$(pwd)"` for non-git directories.

## Vault Path

All `06 Projects/` paths are relative to the obsidian vault:

```bash
OBSIDIAN_VAULT="/Users/duncanleung/Develop/obsidian-local-life-manager"
PROJECT_DIR="${OBSIDIAN_VAULT}/06 Projects/${PROJECT}"
```

## Auto-Create Directory

Before writing any files, ensure the project directory exists:

```bash
mkdir -p "${OBSIDIAN_VAULT}/06 Projects/${PROJECT}/"
```

**Exception:** `/project-init-space` handles its own directory creation and should NOT auto-create — it validates that the directory does not already exist.

## Examples

| Invoked From | `git rev-parse --show-toplevel` | `PROJECT` | Target |
|---|---|---|---|
| `/Users/duncanleung/Develop/care-portal/src/` | `/Users/duncanleung/Develop/care-portal` | `care-portal` | `06 Projects/care-portal/` |
| `/Users/duncanleung/Develop/go-airvet/` | `/Users/duncanleung/Develop/go-airvet` | `go-airvet` | `06 Projects/go-airvet/` |
| `/Users/duncanleung/Develop/obsidian-local-life-manager/` | `/Users/duncanleung/Develop/obsidian-local-life-manager` | `obsidian-local-life-manager` | `06 Projects/obsidian-local-life-manager/` |
| Non-git directory `/tmp/scratch/` | (fails) | `scratch` | `06 Projects/scratch/` |
