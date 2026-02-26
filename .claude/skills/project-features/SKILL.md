---
name: project-features
description: "Decompose project brief into features with user stories, acceptance criteria, dependency graph, and MVP scope. Use after creating a project brief to define what to build."
model: claude-opus-4-5-20251101
allowed-tools: Read, Write, Edit, Glob, Grep
---

# /project-features

Decompose a project brief into features with user stories, acceptance criteria, dependencies, and prioritized scope. This is the bridge between "what problem are we solving?" (brief) and "what work items do we create?" (issues).

## Usage

```bash
/project-features                           # Interactive discovery for current project
/project-features --project coordinatr      # Features for specific project
/project-features --add                     # Add new feature(s) to existing map
/project-features --review                  # Analyze existing features (no edits)
```

## Output Location

```
06 Projects/[project]/features/
├── README.md              # Feature map: dependency graph + scope + index
├── F-001-auth.md          # Feature card
├── F-002-documents.md
└── F-003-sharing.md
```

Features are **project-wide reference artifacts** (like `project-brief.md`), not initiative-scoped. A single feature may spawn multiple initiative folders over time.

## Execution Flow

### 0. Resolve Project

Resolve `[project]` using the [Project Discovery](../project-shared/references/project-discovery.md) procedure:
- If `--project` is specified, use that value
- Otherwise, auto-detect: `basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"`
- Ensure target directory exists: `mkdir -p` the `06 Projects/[project]/` path in the obsidian vault

### 1. Determine Mode

- **No `features/` exists** -> Full Discovery Mode (4-phase conversation)
- **`features/` exists, no flags** -> Gap-driven update mode (show current map, ask what to change)
- **`--add` flag** -> Add new feature(s) to existing map
- **`--review` flag** -> Analysis mode (no edits)

### 2. Load Context

```bash
Read: "06 Projects/[project]/project-brief.md"
Glob: "06 Projects/[project]/features/F-*.md"     # Existing features (if any)
Glob: "06 Projects/[project]/specs/*.md"           # Imported specs (if any)
```

If no `project-brief.md` exists, stop and suggest: "Run `/project-brief` first."

### 3. Four-Phase Discovery

Conduct an interactive conversation (one phase at a time). **Skip phases where answers are clearly inferable from the brief.** If the brief is rich, you may skip directly to unclear areas.

**Question format:** Number all questions and provide letter options where applicable:
```
1. I see these feature areas in your brief. Does this capture everything?
   A. User Authentication (registration, login, password reset)
   B. Document Management (CRUD operations)
   C. Team Collaboration (sharing, permissions)
   D. Something else?
```
This lets the user respond quickly with "1A,1B" instead of writing paragraphs.

#### Phase 1: Feature Discovery

Read the project brief and propose feature groupings.

1. Extract capabilities from the brief's Solution, Scope, and Project Phases sections
2. Group related capabilities into coherent features (3-8 features typical)
3. Present proposed features with brief descriptions
4. Ask: "Does this capture everything? Anything to add, merge, or split?"

**Grouping heuristic:** Capabilities belong to the same feature when they:
- Serve the same user goal
- Share a common workflow
- Would be released together to make sense to users
- Have natural technical cohesion

#### Phase 2: Story Writing

For each feature, write user stories with acceptance criteria. Present one feature at a time.

**Story format — Job Stories (preferred for personal tools):**
```
When [situation/context],
I want to [motivation/action],
so I can [expected outcome/benefit].
```

Job Stories are preferred over classic "As a [role]" stories because:
- Personal tools have one user — context matters more than role
- The "When" clause captures the trigger/situation
- Combine strengths: "When [situation], as a [role], I want..."

**Acceptance criteria format — Given-When-Then:**
```
- Given [precondition], When [action], Then [expected result]
```

**Per story:**
1. Write 1-3 sentences describing the story
2. Write 3-5 acceptance criteria in Given-When-Then format
3. If a story has >5 AC, prompt: "This story seems large. Split using SPIDR?"

**Quality checks (apply per story):**
- **INVEST**: Independent, Negotiable, Valuable, Estimable, Small, Testable
- **Vertical slice**: Does this story deliver end-to-end value? (not just a database layer or just a UI)
- **Testable**: Can each AC become an automated test?
- If a story violates INVEST, suggest splitting using **SPIDR** (Spike / Path / Interface / Data / Rules)

#### Phase 3: Dependency Mapping

Present the dependency graph as ASCII art. Ask for corrections.

```
F-001 (Auth) ──> F-002 (Teams)
                      |
                      v
F-003 (Documents) ──> F-004 (Sharing)
```

Rules:
- Arrow means "must be built before"
- Foundational features (no dependencies) go first
- Circular dependencies indicate features that should be merged

#### Phase 4: Scope & Prioritization

1. **MoSCoW priority** per feature:
   - **Must-have**: Non-negotiable for first usable version (~60% of effort)
   - **Should-have**: Important but not critical (~20%)
   - **Could-have**: Desirable if time allows (~20%)
   - **Won't-have**: Explicitly out of scope (for now)

2. **Walking skeleton**: The thinnest end-to-end slice that proves the architecture works. Not user-facing value — validates the technical foundation.
   - Example: "F-001 partial (login only) + F-003 partial (create + view only)"

3. **MVP boundary**: The minimum set of features that delivers usable value.

4. **Implementation order**: Based on dependencies + priority. Must-haves first, respecting the dependency graph.

Present scope as a structured summary. Ask: "Does this MVP feel right? Anything to move up or defer?"

### 4. Write Output Files

#### Feature Card (per feature)

```markdown
---
id: F-001
name: User Authentication
status: planned | in_progress | complete
priority: must-have | should-have | could-have | wont-have
depends_on: []
---

# F-001: User Authentication

## Why This Feature Exists

[One paragraph connecting to user need from project brief]

## User Stories

### Story 1: Account Registration

When I want to start using the platform,
I want to create an account with my email,
so I can access personalized features.

**Acceptance Criteria:**
- Given a valid email and password, When I submit registration, Then my account is created and I'm logged in
- Given an email already in use, When I submit registration, Then I see "Email already registered"
- Given a password under 8 characters, When I submit registration, Then I see a validation error

### Story 2: Login

When I return to the platform,
I want to log in with my credentials,
so I can continue where I left off.

**Acceptance Criteria:**
- Given valid credentials, When I submit login, Then I'm redirected to my dashboard
- Given invalid credentials, When I submit login, Then I see "Invalid email or password"

## Dependencies

- None (foundational feature)

## Non-Goals

- Social login (OAuth) -- separate feature
- Multi-factor authentication -- deferred
```

#### Feature Map README

```markdown
# Feature Map

## Dependency Graph

F-001 (Auth) ──> F-002 (Teams)
                      |
                      v
F-003 (Documents) ──> F-004 (Sharing)

## Scope

### Walking Skeleton
- F-001 partial (login only) + F-003 partial (create + view only)

### MVP (Milestone 1)
- F-001: User Authentication (full)
- F-003: Document Management (full CRUD)

### Phase 2
- F-002: Team Management
- F-004: Document Sharing

### Deferred
- F-005: Document Templates

## Feature Index

| ID | Feature | Priority | Stories | Status | Dependencies |
|----|---------|----------|---------|--------|-------------|
| F-001 | Auth | must-have | 3 | planned | none |
| F-002 | Teams | must-have | 2 | planned | F-001 |
| F-003 | Documents | must-have | 4 | planned | F-001 |
| F-004 | Sharing | should-have | 2 | planned | F-001, F-003 |
| F-005 | Templates | could-have | 1 | planned | F-003 |
```

### 5. Feature ID Assignment

- Sequential within project: `F-001`, `F-002`, `F-003`...
- When adding new features (`--add`), read existing cards to determine next ID
- IDs are stable — never reuse a deleted feature's ID

## Update Mode (features/ already exists)

When `features/` exists and no flag is passed:

1. Load and display current Feature Map README
2. Show feature index with status
3. Ask: "What would you like to do?"
   - A. Add new features
   - B. Update stories on an existing feature
   - C. Re-scope (change priorities, MVP boundary)
   - D. Review only (no changes)

For each option, enter the relevant phase of the discovery flow.

## Writing Standard

Assume the primary reader is a **junior developer**. Feature descriptions, user stories, and acceptance criteria should be explicit, unambiguous, and avoid jargon. If a junior dev couldn't understand the feature and start planning work from it, the feature card isn't clear enough.

## Quality Guardrails

| Guardrail | Rule | Action if Violated |
|-----------|------|-------------------|
| **INVEST** | Each story: Independent, Negotiable, Valuable, Estimable, Small, Testable | Prompt to split |
| **Vertical slicing** | Each story delivers end-to-end value | Suggest reframing |
| **AC count** | 3-5 acceptance criteria per story | >5: prompt to split using SPIDR |
| **Feature size** | 2-8 stories per feature | >8: probably an epic, split into features |
| **Non-Goals** | Every feature states what it does NOT include | Prompt if missing |
| **Dependencies** | No circular dependencies | Suggest merging features |

### SPIDR Splitting (when stories are too large)

| Letter | Technique | Example |
|--------|-----------|---------|
| **S** | Spike | "Time-box 2 days to research options" |
| **P** | Path | Split by user paths (credit card, PayPal, Apple Pay) |
| **I** | Interface | Split by platform (web, mobile, API) |
| **D** | Data | Split by data type (CSV import, JSON import, API sync) |
| **R** | Rules | Split by business rule variations |

Try Paths first (most common), then Data, Rules, Interfaces, Spike (last resort).

## Guardrails

- Do NOT create code files or start implementation
- Do NOT modify files outside `06 Projects/`
- Do NOT create initiative folders -- that is `/project-issue`'s job
- Do NOT skip to issue creation -- finish features first

## When to Use

- After creating a project brief (`/project-brief`)
- When starting implementation planning for a project
- When adding new capabilities to an existing project
- When re-scoping or re-prioritizing a project

**Not needed for**: Quick bug fixes (go straight to `/project-issue`), one-off scripts, research tasks

## When NOT to Use

- Project has <3 capabilities (just create issues directly)
- Implementing an external spec (use `/project-import-spec` first, then `/project-features`)
- Quick prototyping where you'll throw away the code

## Next Steps After Features

```
/project-brief -> /project-features -> /project-issue -> /project-plan -> /project-implement
```

Optional steps (invoke when relevant, not mandatory):
- `/project-critique` -- before significant investment
- `/research-deep` -- when domain is unfamiliar
- `/project-import-spec` -- when implementing external PRD/standard
