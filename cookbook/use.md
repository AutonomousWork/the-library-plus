# Use an Entry from the Library

## Context
Pull a skill, agent, or prompt into the local environment, or configure an MCP or plugin from the catalog. If the item is already installed, refresh it.

## Input
The user provides an item name or description.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before reading:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Find the Entry
- Read `library.yaml`
- Search across:
  - `library.skills`
  - `library.agents`
  - `library.prompts`
  - `library.mcps`
  - `library.plugins`
- Match by exact name first, then description keywords
- If multiple matches, show them and ask the user to choose
- If no match, tell the user and suggest `/library search`

### 3. Resolve Dependencies
If the entry has a `requires` field:
- For each typed reference:
  - `skill:name`
  - `agent:name`
  - `prompt:name`
  - `mcp:name`
  - `plugin:name`
- Look it up in `library.yaml`
- If found, recursively run the `use` workflow for that dependency first
- If not found, warn the user that the dependency is missing from the catalog

### 4. Determine the Install Target

**For skills, agents, and prompts**
- Read `default_dirs` from `library.yaml`
- If the user said "global" → use the `global` path
- If the user specified a custom path → use that path
- Otherwise → use the `default` path

**For MCPs and plugins**
- Read `default_scopes` from `library.yaml`
- Resolve scope like this:
  - user said "global" → `user`
  - user said "local" → `local`
  - user said "project" or gave no preference → `project`
- For MCPs use `default_scopes.mcps`
- For plugins use `default_scopes.plugins`

### 5. Fetch or Configure the Item

**If the entry is a skill, agent, or prompt with a local path source**
- Resolve `~`
- For skills: copy the full parent directory to the target
- For agents: copy the referenced file to the target path
- For prompts: copy the referenced file to the target path
- Preserve subdirectory structure for nested agent or prompt files

**If the entry is a skill, agent, or prompt with a GitHub source**
- Parse the GitHub URL into clone URL, branch, and file path
- Clone to a temporary directory
- Copy only the referenced directory or file into the install target
- If HTTPS clone fails for a private repo, retry with SSH

**If the entry is an MCP**
- Fetch the JSON from `source`
- Validate it is a single JSON object compatible with `claude mcp add-json`
- If an MCP with the same name already exists in the resolved scope, remove it first:
  ```bash
  claude mcp remove --scope <scope> <name>
  ```
- Add the refreshed config:
  ```bash
  claude mcp add-json --scope <scope> <name> '<json>'
  ```
- Keep environment placeholders intact; do not inline secrets from the current machine

**If the entry is a plugin**
- Resolve the install id as `<plugin-or-name>@<marketplace>`
- Update the resolved Claude settings file for that scope so it contains:
  - `extraKnownMarketplaces.<marketplace>.source = <marketplace_source>`
  - `enabledPlugins["<plugin-or-name>@<marketplace>"] = true`
- Preserve unrelated settings keys
- If the marketplace source can be represented directly on the CLI:
  - `github` → use `<repo>`
  - `git` or `url` → use `<url>`
  - `file` or `directory` → use `<path>`
- Use `claude plugin marketplace add --scope <scope> <source>` when the marketplace is missing
- Use `claude plugin marketplace update <marketplace>` when it already exists
- If the plugin is already installed in that scope, refresh it:
  ```bash
  claude plugin update --scope <scope> <plugin-or-name>@<marketplace>
  ```
- Otherwise install it:
  ```bash
  claude plugin install --scope <scope> <plugin-or-name>@<marketplace>
  ```
- For settings-only marketplace source kinds such as `npm` or `hostPattern`, write the settings entry first and then update or install once Claude recognizes the marketplace

### 6. Verify Installation

**For skills, agents, and prompts**
- Confirm the target directory or file exists

**For MCPs**
- Confirm `claude mcp get <name>` succeeds

**For plugins**
- Confirm `claude plugin list --json` contains the plugin id in the target scope

### 7. Confirm
Tell the user:
- What was installed or configured
- Where it was installed or which scope it was configured in
- Any dependencies that were installed first
- Whether this was a refresh
