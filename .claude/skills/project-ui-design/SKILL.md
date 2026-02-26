---
name: project-ui-design
description: "Create HTML UI mockups stored in 06 Projects/[project]/docs/ui-designs/"
model: claude-opus-4-5-20251101
allowed-tools: Read, Write, Edit, Glob, Grep, Task
---

# /ui-design

Generate HTML UI mockups with optional parallel variant exploration.

## Usage

```bash
/ui-design yourbench "login screen"
/ui-design yourbench "dashboard" --variants 3
/ui-design coordinatr "project list" --tech shadcn
/ui-design yourbench list                    # Show existing designs
```

## Where Designs Live

Designs live inside initiative folders:

```
06 Projects/yourbench/2026-02-18-auth/
├── TASK.md
├── ui-login-screen-v1.html
├── ui-login-screen-v2a.html      # Variant A
├── ui-login-screen-v2b.html      # Variant B (approved)
└── ui-dashboard-v1.html
```

**Why in initiative folders?** Designs are planning artifacts scoped to specific work.

## Execution Flow

### 0. Resolve Project

If no project argument is provided, resolve it using the [Project Discovery](../project-shared/references/project-discovery.md) procedure:
- Auto-detect: `basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"`
- Ensure target directory exists: `mkdir -p` the `06 Projects/[project]/` path in the obsidian vault

### 1. Parse Request
- Project (yourbench)
- Design name (login screen)
- Variant count (--variants 3)
- Technology (--tech shadcn)

### 2. Load Context
```bash
Glob: "06 Projects/[project]/*/ui-*.html"
Read: "06 Projects/[project]/project-brief.md"
```

### 3. Generate Design(s)

**Single design:**
```
→ 06 Projects/yourbench/2026-02-18-auth/ui-login-screen-v1.html
```

**Multiple variants (parallel ui-ux-designer agents):**
```
→ ui-login-screen-v1a.html
→ ui-login-screen-v1b.html
→ ui-login-screen-v1c.html
```

### 4. Present Options

```
Created 3 login screen variants:

1. ui-login-screen-v1a.html - Minimal, centered form
2. ui-login-screen-v1b.html - Split screen with illustration
3. ui-login-screen-v1c.html - Card-based with social logins

View: open "06 Projects/yourbench/2026-02-18-auth/ui-login-screen-v1a.html"

Which direction? (a/b/c/iterate/combine)
```

### 5. Iterate

User requests changes:
- "Move OAuth buttons below the form"
- "Try a darker color scheme"

### 6. Approve

```
User: approve v1b

AI: ✓ Marked ui-login-screen-v1b.html as APPROVED

    Reference in TASK.md:
    "Implement login per ui-login-screen-v1b.html"
```

## Technology Options

| Option | Description |
|--------|-------------|
| `--tech vanilla` | Plain HTML/CSS/JS (default) |
| `--tech shadcn` | Styled for shadcn/ui with implementation hints |
| `--tech chakra` | Styled for Chakra UI |

## HTML Structure

Self-contained with embedded CSS/JS:
- CSS variables from style-guide.md
- Responsive breakpoints
- Interactive behaviors
- Metadata block at end (status, decisions, related specs)

## Listing Designs

```bash
/ui-design yourbench list

UI Designs for yourbench:
├── 2026-02-18-auth/ui-login-screen-v1b.html [APPROVED]
├── 2026-02-18-auth/ui-dashboard-v1.html [DRAFT]
└── 2026-02-19-settings/ui-settings-v1a.html [DRAFT]
```

## Integration with Implementation

```bash
/implement yourbench 2026-02-18-auth 1.3  # "Implement login UI"

AI: Found approved design: ui-login-screen-v1b.html
    Implementing to match design...
```

Reference in TASK.md:
```markdown
## Acceptance Criteria
- [ ] Matches ui-login-screen-v1b.html
- [ ] Responsive at 320px, 768px, 1280px
```

## Best Practices

1. **Start with variants** - Explore before converging
2. **Approve explicitly** - Clear handoff to implementation
3. **Include metadata** - Future you will thank you
4. **Test responsiveness** - Check 320px, 768px, 1280px
5. **Document decisions** - Why this approach?
