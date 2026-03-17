# Search the Library

## Context
Find entries in the catalog by keyword when the user does not remember the exact name.

## Input
The user provides a keyword or description.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before reading:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Read the Catalog
- Read `library.yaml`
- Parse all entries from:
  - `library.skills`
  - `library.agents`
  - `library.prompts`
  - `library.mcps`
  - `library.plugins`

### 3. Search
- Match the keyword case-insensitively against:
  - `name`
  - `description`
- Collect matches across all types

### 4. Display Results
If matches are found, format as:

```
## Search Results for "<keyword>"

| Type | Name | Description | Reference |
|------|------|-------------|-----------|
| skill | matching-skill | description... | /path/to/SKILL.md |
| mcp | github | description... | github.mcp.json |
| plugin | review-toolkit | description... | pr-review-toolkit@team-marketplace |
```

For the `Reference` column:
- skills, agents, prompts, MCPs → show `source`
- plugins → show `<plugin-or-name>@<marketplace>`

If no matches are found:
```
No results found for "<keyword>".

Tip: Try broader keywords or run `/library list` to see the full catalog.
```

### 5. Suggest Next Step
If matches were found, suggest: `Run /library use <name> to install or configure one of these.`
