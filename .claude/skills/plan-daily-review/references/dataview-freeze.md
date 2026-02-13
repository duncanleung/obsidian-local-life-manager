# Dataview Freeze Algorithm

## Overview

The freeze process replaces live Dataview query blocks with static markdown tables, creating a permanent historical snapshot of the journal. This allows viewing past journals with the data as it existed on that day, not current data.

## Supported Query Pattern

We support a specific subset of Dataview DQL queries used in daily journal templates:

```
TABLE [columns] FROM "[folder]" WHERE [conditions] SORT [field] [direction]
```

### Example Query
```dataview
TABLE status AS "Status", due AS "Due", source_type AS "Source"
FROM "03 TaskNotes/"
WHERE status != "done" AND status != "cancelled" AND (due > date(today) OR !due)
SORT choice(due, due, date("9999-12-31")) ASC
```

## Parsing Steps

### 1. TABLE Column Parsing

Extract column specifications from the TABLE clause:

**Format**: `field AS "Alias"` or just `field`
**Examples**:
- `status AS "Status"` → {field: "status", alias: "Status"}
- `due AS "Due"` → {field: "due", alias: "Due"}
- `source_type` → {field: "source_type", alias: "source_type"}

**Special handling**:
- First column is always the file link: `[[filename]]`
- Use alias if provided, otherwise use field name as column header

### 2. FROM Clause Extraction

Extract the target folder from the FROM clause:

**Format**: `FROM "folder-name/"`
**Examples**:
- `FROM "03 TaskNotes/"` → "03 TaskNotes/"
- `FROM "07 Knowledge Base/Capture/"` → "07 Knowledge Base/Capture/"

**Implementation**: Glob pattern `{folder}*.md` to find all markdown files in the folder.

### 3. WHERE Condition Tokenization

Parse and evaluate WHERE conditions against YAML frontmatter:

#### Supported Condition Types

| Condition | Example | Evaluation |
|-----------|---------|------------|
| String equality | `status = "done"` | Exact string match |
| String inequality | `status != "done"` | Not equal string comparison |
| Date comparison | `due < date(today)` | Compare date values |
| Date equality | `due = date(today)` | Equal date comparison |
| Null/exists check | `due AND due < date(today)` | Field exists AND condition |
| Negation | `!due` | Field is empty/missing/null |
| Boolean AND | `cond1 AND cond2` | Both conditions true |
| Boolean OR | `cond1 OR cond2` | Either condition true |

#### Date Resolution

**`date(today)`**: Resolves to the current date in YYYY-MM-DD format
- For EOD freeze: `date(today)` = journal date = current date
- Dates in frontmatter can be quoted strings (`"2026-02-12"`) or unquoted

#### Condition Evaluation Order

1. Parentheses are respected: `(due > date(today) OR !due)`
2. AND has higher precedence than OR
3. Negation (`!`) has highest precedence

### 4. SORT Clause Parsing

Extract sorting instructions:

**Format**: `SORT [field] [direction]`
**Examples**:
- `SORT due ASC` → Sort by due date ascending
- `SORT choice(due, due, date("9999-12-31")) ASC` → Sort by due date, nulls last

#### Special Functions

**`choice(condition, value_if_true, value_if_false)`**:
- Used to handle null values in sorting
- `choice(due, due, date("9999-12-31"))` → Use due date if exists, otherwise use far future date
- This effectively sorts nulls last

## Query Evaluation Logic

### 1. File Discovery
```bash
# Glob files from the FROM folder
files=$(glob "{folder}*.md")
```

### 2. Frontmatter Reading

For each file, extract YAML frontmatter:
- Parse YAML between `---` delimiters
- Handle missing fields as null/empty
- Handle malformed YAML gracefully (skip file or treat as empty frontmatter)

### 3. Condition Evaluation

For each file's frontmatter, evaluate the WHERE clause:
- Parse conditions into tokens
- Evaluate each condition against frontmatter values
- Combine using Boolean logic (AND, OR, NOT)
- Include file in results if WHERE clause evaluates to true

### 4. Sorting

Sort results according to SORT clause:
- Extract sort field and direction
- Handle special functions like `choice()`
- Sort files by the specified field value

## Output Format

### Static Markdown Table

Replace the dataview code block with:

```markdown
| File | Status | Due | Source |
|------|--------|-----|--------|
| [[task-name]] | open | 2026-02-13 | github |
| [[other-task]] | open | 2026-02-14 | jira |
```

### Formatting Rules

1. **File Column**: Always first, format as `[[filename]]` (without .md extension, without folder path)
2. **Headers**: Use aliases from TABLE spec or field names if no alias
3. **Empty Cells**: Show empty string for missing/null values
4. **Dates**: Format as YYYY-MM-DD
5. **Empty Results**: Show `*No items*` if no files match WHERE conditions

### Example Transformation

**Before (Dataview block)**:
````markdown
```dataview
TABLE status AS "Status", due AS "Due"
FROM "03 TaskNotes/"
WHERE status != "done"
SORT due ASC
```
````

**After (Static table)**:
```markdown
| File | Status | Due |
|------|--------|-----|
| [[review-pr]] | open | 2026-02-13 |
| [[update-docs]] | open | 2026-02-14 |
```

## Error Handling

### Graceful Degradation

1. **Invalid Query**: Leave dataview block unchanged, warn user
2. **Missing Files**: Empty table with "No items"
3. **Malformed YAML**: Skip file, continue processing others
4. **Missing Fields**: Treat as empty/null, continue evaluation
5. **Parse Errors**: Skip problematic condition, warn user

### Already Frozen Detection

- If no `dataview` code blocks found → journal already frozen
- Skip freeze process gracefully
- Message: "Journal already frozen (no Dataview blocks found)"

## Implementation Notes

### What We Support

- TABLE queries with FROM, WHERE, SORT
- String and date comparisons
- Boolean logic (AND, OR, NOT)
- Column aliases
- The `choice()` function for null handling
- Basic date functions: `date(today)`

### What We DON'T Support

- GROUP BY (not used in our templates)
- TASK queries (we use TABLE)
- Inline DQL
- DataviewJS
- Complex functions beyond `choice()` and `date()`
- Advanced date arithmetic

This subset covers all the queries used in our daily journal templates while keeping the implementation focused and reliable.