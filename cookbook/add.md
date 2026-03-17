# Add a New Entry to the Library

## Context
Register a new skill, agent, prompt, MCP, or Claude plugin in the library catalog.

## Input
The user provides: name, description, and either a `source` file reference or plugin marketplace metadata. They may also provide an explicit type and dependencies.

## Steps

### 1. Sync the Library Repo
Pull the latest changes before modifying:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Determine the Type
Figure out the type from the user's prompt or metadata:
- If the source path contains `SKILL.md` or the user says "skill" → type is `skill`
- If the source path contains `AGENT.md` or the user says "agent" → type is `agent`
- If the user says "prompt" or the source is a standalone Markdown command file → type is `prompt`
- If the source is a JSON file compatible with `claude mcp add-json` or the user says "mcp" → type is `mcp`
- If the user says "plugin" or provides `marketplace` plus `marketplace_source` → type is `plugin`
- If ambiguous, ask the user

### 3. Validate the Source

**For skills, agents, prompts, and MCPs**
- **Local path**: Verify the file exists at the given path
- **GitHub URL**: Verify the URL is well-formed (browser or raw GitHub URL)
- Confirm the source points to a specific file, not a directory
- For MCPs, validate that the file contains one JSON object compatible with `claude mcp add-json`
- For MCPs, prefer environment placeholders such as `${API_KEY}` instead of raw secrets or machine-specific values

**For plugins**
- Require:
  - `marketplace`: the stable marketplace name
  - `marketplace_source`: a Claude marketplace source object
- Validate `marketplace_source` against Claude's supported source kinds:
  - `github`: requires `repo`; optional `ref`, `path`
  - `git`: requires `url`; optional `ref`, `path`
  - `url`: requires `url`; optional `headers`
  - `npm`: requires `package`
  - `file`: requires absolute `path`
  - `directory`: requires absolute `path`
  - `hostPattern`: requires `hostPattern`
- If the plugin install name differs from the catalog `name`, store it in `plugin`

### 4. Parse Dependencies
Detect dependencies and format them as typed references:
- `skill:name`
- `agent:name`
- `prompt:name`
- `mcp:name`
- `plugin:name`

Verify each dependency already exists in `library.yaml` or warn the user.
- If a missing dependency should also be cataloged, add it first
- Detect dependencies from frontmatter, linked slash commands, plugin docs, MCP setup notes, or explicit user input

### 5. Add the Entry to `library.yaml`
Read `library.yaml`, then add the new entry under the correct section.

**For skills, agents, prompts, and MCPs**
```yaml
- name: <name>
  description: <description>
  source: <source>
  requires: [<typed:refs>]  # omit if no dependencies
```

**For plugins**
```yaml
- name: <name>
  description: <description>
  marketplace: <marketplace>
  marketplace_source:
    source: <source kind>
    ...
  plugin: <plugin install name>  # omit if same as name
  requires: [<typed:refs>]       # omit if no dependencies
```

**YAML formatting rules**
- 2-space indentation
- List items use `- `
- Keep entries alphabetically sorted by `name` within each section
- Skills reference the `.../<skill-name>/SKILL.md` file
- Agents reference the `.../<agent-name>.md` file
- Prompts reference the `.../<prompt-name>.md` file in `.claude/commands/`
- MCPs reference the `.json` file that should be passed to `claude mcp add-json`
- Plugins do not use `source`; they use `marketplace` plus `marketplace_source`

### 6. Commit and Push
```bash
cd <LIBRARY_SKILL_DIR>
git add library.yaml
git commit -m "library: added <type> <name>"
git push
```

### 7. Confirm
Tell the user the entry has been added and is now available via `/library use <name>`.
