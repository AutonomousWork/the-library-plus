# Sync All Installed Items

## Context
Refresh every locally installed or configured skill, agent, prompt, MCP, and plugin from the catalog. This is the fast "make everything current" command.

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

### 3. Find All Installed Items

**For skills, agents, and prompts**
- Determine the directories from `default_dirs`
- Check default and global install locations
- Search recursively for matching names
- Collect installed entries with their detected location

**For MCPs**
- Treat an MCP as installed if `claude mcp get <name>` succeeds
- Prefer the scope reported by Claude; otherwise fall back to `default_scopes.mcps`

**For plugins**
- Run `claude plugin list --json`
- Treat a plugin as installed if `<plugin-or-name>@<marketplace>` appears in the list
- Record the detected scope from Claude's JSON output

If nothing is installed or configured, tell the user and exit.

### 4. Resolve Dependencies
For each installed entry that has a `requires` field:
- Check whether each dependency is already installed or configured
- If not, install it first using the matching `use` workflow
- Support all typed dependency forms:
  - `skill:name`
  - `agent:name`
  - `prompt:name`
  - `mcp:name`
  - `plugin:name`

### 5. Refresh Each Installed Item

**For skills, agents, and prompts**
- Re-fetch from local path or GitHub source exactly like `/library use`
- Overwrite the installed target with the latest source contents

**For MCPs**
- Fetch the latest JSON object from the MCP `source`
- Validate it is still compatible with `claude mcp add-json`
- Refresh by replacing the configured entry:
  ```bash
  claude mcp remove --scope <scope> <name>
  claude mcp add-json --scope <scope> <name> '<json>'
  ```
- Do not expand env placeholders or inject raw secrets during sync

**For plugins**
- Ensure the resolved Claude settings file still contains:
  - `extraKnownMarketplaces.<marketplace>.source = <marketplace_source>`
  - `enabledPlugins["<plugin-or-name>@<marketplace>"] = true`
- Preserve unrelated settings keys
- Refresh the marketplace first:
  - If the source can be expressed as a marketplace CLI argument (`github`, `git`, `url`, `file`, `directory`), use `claude plugin marketplace add` when missing and `claude plugin marketplace update <marketplace>` when present
  - For settings-only source kinds such as `npm` or `hostPattern`, write the settings entry first, then run `claude plugin marketplace update <marketplace>` if Claude already knows the marketplace
- Refresh the plugin:
  ```bash
  claude plugin update --scope <scope> <plugin-or-name>@<marketplace>
  ```
- If the plugin is enabled in settings but missing from the installed list, install it:
  ```bash
  claude plugin install --scope <scope> <plugin-or-name>@<marketplace>
  ```
- Never treat `~/.claude/plugins/cache` or `installed_plugins.json` as the source of truth

### 6. Report Results
Display a summary table:

```
## Sync Complete

| Type | Name | Status |
|------|------|--------|
| skill | deploy | refreshed |
| mcp | github | refreshed (project) |
| plugin | review-toolkit | updated (project) |
| prompt | commit-message | failed: <reason> |

Synced: X items
Failed: Y items
```

If any items failed, list the reason so the user can fix them individually.
