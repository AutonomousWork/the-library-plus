---
name: library
description: Private skill distribution system. Use when the user wants to install, use, add, push, remove, sync, list, or search for skills, agents, prompts, MCPs, or Claude plugins from their private library catalog. Triggers on /library commands or mentions of library, skill distribution, MCP sync, or plugin management.
argument-hint: [command or prompt] [name or details]
---

# The Library

A meta-skill for private-first distribution of agentics across agents, devices, and teams: skills, agents, prompts, MCPs, and Claude plugins.

## Variables

> Update these after forking and cloning the library repo.

- **LIBRARY_REPO_URL**: `<your forked repo url>`
- **LIBRARY_YAML_PATH**: `~/.claude/skills/library/library.yaml`
- **LIBRARY_SKILL_DIR**: `~/.claude/skills/library/`

## How It Works

The Library is a catalog of references to your agentics. The `library.yaml` file points to where skills, agents, prompts, MCP definitions, and plugin marketplace definitions live. Nothing is fetched or configured until you ask for it.

**The `library.yaml` is a catalog, not a manifest.** Entries define what is *available* - not what gets installed. You pull or configure specific items on demand with `/library use <name>`.

- Skills, agents, prompts, and MCPs are source-backed entries that point to local files or GitHub URLs
- Plugins are marketplace-backed entries that point to a Claude marketplace name plus marketplace source metadata
- `/library sync` refreshes whatever is already installed or configured locally

## Commands

| Command                     | Purpose                                                  |
| --------------------------- | -------------------------------------------------------- |
| `/library install`          | First-time setup: fork, clone, configure                 |
| `/library add <details>`    | Register a new entry in the catalog                      |
| `/library use <name>`       | Install, configure, or refresh an entry from the catalog |
| `/library push <name>`      | Push local changes back to a source-backed entry         |
| `/library remove <name>`    | Remove from catalog and optionally remove local state    |
| `/library list`             | Show full catalog with install/config status             |
| `/library sync`             | Refresh all installed and configured items               |
| `/library search <keyword>` | Find entries by keyword                                  |

## Cookbook

Each command has a detailed step-by-step guide. **Read the relevant cookbook file before executing a command.**

| Command | Cookbook                                 | Use When                                                              |
| ------- | ---------------------------------------- | --------------------------------------------------------------------- |
| install | [cookbook/install.md](cookbook/install.md) | First-time setup on a new device                                      |
| add     | [cookbook/add.md](cookbook/add.md)         | User wants to register a new skill, agent, prompt, MCP, or plugin     |
| use     | [cookbook/use.md](cookbook/use.md)         | User wants to install, configure, or refresh a catalog entry          |
| push    | [cookbook/push.md](cookbook/push.md)       | User improved a local skill, agent, or prompt and wants to update it  |
| remove  | [cookbook/remove.md](cookbook/remove.md)   | User wants to remove an entry from the catalog or local environment   |
| list    | [cookbook/list.md](cookbook/list.md)       | User wants to see what is available and what is already installed     |
| sync    | [cookbook/sync.md](cookbook/sync.md)       | User wants to refresh all installed and configured items at once      |
| search  | [cookbook/search.md](cookbook/search.md)   | User knows roughly what they want but not the exact catalog name      |

**When a user invokes a `/library` command, read the matching cookbook file first, then execute the steps.**

## Source-Backed Entry Format

The `source` field in `library.yaml` supports these formats for skills, agents, prompts, and MCPs:

- `/absolute/path/to/file` - local filesystem
- `https://github.com/org/repo/blob/main/path/to/file` - GitHub browser URL
- `https://raw.githubusercontent.com/org/repo/main/path/to/file` - GitHub raw URL

Both GitHub URL formats are supported. Parse org, repo, branch, and file path from the URL structure. For private repos, use SSH or `GITHUB_TOKEN` automatically.

**Important**
- Skills point to `SKILL.md`; pull the entire parent directory
- Agents point to the referenced Markdown file; copy that file, preserving nested structure when useful
- Prompts point to the referenced Markdown file in `.claude/commands/`
- MCPs point to a single JSON object compatible with `claude mcp add-json`

## Source Parsing Rules

**Local paths** start with `/` or `~`:
- Use the path directly
- Skills copy the parent directory
- Agents and prompts copy the referenced file
- MCPs read the referenced JSON file directly

**GitHub browser URLs** match `https://github.com/<org>/<repo>/blob/<branch>/<path>`:
- Parse: `org`, `repo`, `branch`, `file_path`
- Clone URL: `https://github.com/<org>/<repo>.git`
- File location within repo: `<path>`

**GitHub raw URLs** match `https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>`:
- Parse: `org`, `repo`, `branch`, `file_path`
- Clone URL: `https://github.com/<org>/<repo>.git`
- File location within repo: `<path>`

## Plugin Marketplace Format

Plugins do not use `source`. They use:

- `marketplace` - the stable Claude marketplace name
- `marketplace_source` - the Claude marketplace source object
- `plugin` - optional install name when it differs from `name`

Supported marketplace source kinds should match Claude's settings schema:
- `github`
- `git`
- `url`
- `npm`
- `file`
- `directory`
- `hostPattern`

Prefer storing the same shape Claude expects in `extraKnownMarketplaces`, for example:

```yaml
marketplace: team-marketplace
marketplace_source:
  source: github
  repo: myorg/claude-marketplace
plugin: pr-review-toolkit
```

## GitHub Workflow

When working with GitHub sources, prefer `gh api` for accessing single files (for example reading `SKILL.md` to inspect metadata). For pulling directories or files, clone into a temp directory per the cookbook steps.

**Fetching (use or sync)**
1. Clone the repo with `git clone --depth 1 <clone_url>` into a temporary directory
2. Navigate to the parent directory or referenced file
3. Copy the relevant directory or file to the target local directory
4. Clean up the temporary directory

**Pushing (push)**
1. Clone the repo with `git clone --depth 1 <clone_url>` into a temporary directory
2. Overwrite only the referenced skill, agent, or prompt path in the clone
3. Stage only the relevant path
4. Commit with message: `library: updated <name> <what changed>`
5. Push to remote
6. Clean up the temporary directory

## Typed Dependencies

The `requires` field uses typed references to avoid ambiguity:
- `skill:name` - references a skill in the library catalog
- `agent:name` - references an agent in the library catalog
- `prompt:name` - references a prompt in the library catalog
- `mcp:name` - references an MCP in the library catalog
- `plugin:name` - references a plugin in the library catalog

When resolving dependencies: look up each reference in `library.yaml`, install or configure all dependencies first (recursively), then install the requested item.

## Target Directories and Scopes

Source-backed items use `default_dirs` in `library.yaml`:

```yaml
default_dirs:
  skills:
    - default: .claude/skills/
    - global: ~/.claude/skills/
  agents:
    - default: .claude/agents/
    - global: ~/.claude/agents/
  prompts:
    - default: .claude/commands/
    - global: ~/.claude/commands/
```

Claude-native config items use `default_scopes`:

```yaml
default_scopes:
  mcps:
    - default: project
    - local: local
    - global: user
  plugins:
    - default: project
    - local: local
    - global: user
```

- If the user says "global" or "globally", use the `global` target or the `user` scope
- If the user says "local", use the `local` scope
- If the user specifies a custom filesystem path, that only applies to skills, agents, and prompts
- Otherwise, use the `default` target or `project` scope

## Claude-Native Rules

For MCPs:
- Use Claude's CLI as the source of truth: `claude mcp add-json`, `claude mcp get`, `claude mcp list`, `claude mcp remove`
- Store only shareable templates in the catalog
- Keep environment placeholders intact; do not write raw secrets into `library.yaml`

For plugins:
- Use Claude's settings model and plugin CLI: `enabledPlugins`, `extraKnownMarketplaces`, `claude plugin marketplace ...`, `claude plugin install`, `claude plugin update`, `claude plugin uninstall`
- Never treat `~/.claude/plugins/cache` or `installed_plugins.json` as user-managed source of truth
- `push` is not supported for plugins in v1 because the catalog stores marketplace references, not unpacked runtime payloads

## Library Repo Sync

The library skill itself lives in `<LIBRARY_SKILL_DIR>` as a cloned git repo. When running `add` or `remove` (which modify `library.yaml`), always:
1. `git pull` in the library directory first to get the latest catalog
2. Make the changes
3. `git add library.yaml && git commit && git push`

This keeps the catalog in sync across devices.

## Example Filled Library File

```yaml
default_dirs:
  skills:
    - default: .claude/skills/
    - global: ~/.claude/skills/
  agents:
    - default: .claude/agents/
    - global: ~/.claude/agents/
  prompts:
    - default: .claude/commands/
    - global: ~/.claude/commands/

default_scopes:
  mcps:
    - default: project
    - local: local
    - global: user
  plugins:
    - default: project
    - local: local
    - global: user

library:
  skills:
    - name: firecrawl
      description: Scrape, crawl, and search websites using Firecrawl CLI
      source: /Users/me/projects/tools/skills/firecrawl/SKILL.md

    - name: diagram-kroki
      description: Generate diagrams via Kroki HTTP API supporting 28+ languages
      source: https://github.com/myorg/private-skills/blob/main/skills/diagram-kroki/SKILL.md
      requires: [skill:firecrawl]

  agents:
    - name: code-reviewer
      description: Reviews code for quality, security, and performance
      source: https://github.com/myorg/agent-configs/blob/main/agents/code-reviewer/AGENT.md

  prompts:
    - name: commit-message
      description: Standardized commit message format for all projects
      source: https://github.com/myorg/team-prompts/blob/main/prompts/commit-message.md

  mcps:
    - name: github
      description: Shared GitHub MCP configuration for Claude Code
      source: https://github.com/myorg/claude-config/blob/main/mcp/github.json

  plugins:
    - name: review-toolkit
      description: Pull request review workflows for the whole team
      marketplace: team-marketplace
      marketplace_source:
        source: github
        repo: myorg/claude-marketplace
      plugin: pr-review-toolkit
      requires: [mcp:github]
```
