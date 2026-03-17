# Remove an Entry from the Library

## Context
The user wants to remove a skill, agent, prompt, MCP, or plugin from the library catalog and optionally remove the local installation/configuration.

## Input
The user provides an item name or description.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before modifying:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Find the Entry
- Read `library.yaml`
- Search across all sections for the matching entry
- Determine the type
- If no match, tell the user the item was not found in the catalog

### 3. Confirm with User
Show the entry details and ask:
- "Remove **<name>** from the library catalog?"
- If installed or configured locally, also ask if the local copy or config should be removed

### 4. Remove from `library.yaml`
- Remove the entry from the appropriate section:
  - `library.skills`
  - `library.agents`
  - `library.prompts`
  - `library.mcps`
  - `library.plugins`
- If other entries depend on this one via `requires`, warn the user before proceeding

### 5. Delete Local Copy or Config (if requested)

**For skills, agents, and prompts**
- Check the default and global directories from `default_dirs`
- Remove the matching directory or file

**For MCPs**
- Resolve the scope from the user's request or `default_scopes.mcps`
- Remove it with:
  ```bash
  claude mcp remove --scope <scope> <name>
  ```

**For plugins**
- Resolve the scope from the user's request or `default_scopes.plugins`
- Remove or disable the plugin using Claude-native config:
  - Update the resolved settings file so `enabledPlugins["<plugin-or-name>@<marketplace>"]` is removed or set to `false`
  - Preserve unrelated settings keys
- If the user wants the install removed too:
  ```bash
  claude plugin uninstall --scope <scope> <plugin-or-name>@<marketplace>
  ```
- Only remove the marketplace if no remaining catalog entries reference it
- Never delete `~/.claude/plugins/cache` or edit `installed_plugins.json` directly

### 6. Commit and Push
```bash
cd <LIBRARY_SKILL_DIR>
git add library.yaml
git commit -m "library: removed <type> <name>"
git push
```

### 7. Confirm
Tell the user:
- The entry has been removed from the catalog
- Whether the local copy or config was also removed
- Whether any dependents still need updating
