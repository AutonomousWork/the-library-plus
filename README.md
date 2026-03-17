# The Library

A meta-skill for private-first distribution of agentics across agents, devices, and teams: skills, agents, prompts, MCPs, and Claude plugins.

![The Library](images/10_meta_skill.svg)

## Who This Is For

If you work across lots of codebases and keep building specialized agent tooling, this was made for you.

If you only work in one or two repos, or you mostly install public tools without reviewing them, you probably do not need this.

The Library solves a specific problem: your best agentics end up scattered across repos, machines, and teammates. They drift out of sync, get copied by hand, and become hard to trust. The Library gives you one private catalog that can rehydrate those capabilities anywhere.

## What It Is

The Library is a single skill whose job is to manage other agent capabilities.

It is a catalog of references:
- local file paths
- GitHub file URLs
- MCP JSON definitions
- Claude plugin marketplace definitions

Nothing is copied or configured until you ask for it.

Think of it as a `package.json` for your private agent stack, except the entries are not npm packages. They are the skills, agents, prompts, MCPs, and plugins your team actually uses.

**This is still a pure agent application.** The Library ships no custom scripts or binaries. The behavior lives in `SKILL.md` and the cookbooks, and the agent uses native Claude surfaces like `/mcp` and `/plugin` when it needs to configure Claude-native integrations.

## Why It Exists

![The Problem: Skill Sprawl](images/26_problem_skill_sprawl.svg)

As you build with AI agents, you accumulate a lot of private leverage:
- skills
- specialized agents
- reusable prompts and commands
- MCP definitions for external systems
- plugin installs and team marketplaces

You need to:
- reuse them across projects without copy-pasting
- distribute them to other machines and sandboxes
- share them with teammates without making everything public
- keep them private because they are competitive advantage
- keep them in sync instead of managing stale copies

![The Problem: Siloed Teams](images/32_problem_team_sharing.svg)

Existing approaches do not quite solve this:
- Global `~/.claude/*` exposes everything to every workflow.
- Standalone `/plugin` and `/mcp` commands configure one Claude surface at a time, but do not give you one private catalog spanning all your agentics.
- A single monorepo rarely matches how teams actually build and own capabilities.

## How It Works

![The Solution: The Library](images/27_solution_library_workflow.svg)

### The Catalog (`library.yaml`)

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
    - name: deploy
      description: Deploy infrastructure changes safely
      source: https://github.com/myorg/infra-tools/blob/main/skills/deploy/SKILL.md
      requires: [agent:reviewer]

  agents:
    - name: reviewer
      description: Reviews changes before deploys
      source: /Users/me/projects/agents/reviewer/AGENT.md

  prompts:
    - name: commit-message
      description: Team commit format
      source: https://github.com/myorg/team-prompts/blob/main/prompts/commit-message.md

  mcps:
    - name: github
      description: Shared GitHub MCP config
      source: https://github.com/myorg/claude-config/blob/main/mcp/github.json

  plugins:
    - name: review-toolkit
      description: Pull request review workflows
      marketplace: team-marketplace
      marketplace_source:
        source: github
        repo: myorg/claude-marketplace
      plugin: pr-review-toolkit
      requires: [mcp:github]
```

The catalog stores pointers, not copies. You install or configure entries on demand with `/library use`, and refresh what is already present with `/library sync`.

### Source-Backed Entries

Skills, agents, prompts, and MCPs use `source`.

| Format | Example |
| --- | --- |
| Local filesystem | `/absolute/path/to/SKILL.md` |
| GitHub browser URL | `https://github.com/org/repo/blob/main/path/to/SKILL.md` |
| GitHub raw URL | `https://raw.githubusercontent.com/org/repo/main/path/to/SKILL.md` |

Rules:
- Skills point at `SKILL.md` and pull the whole parent directory.
- Agents point at the Markdown file to install.
- Prompts point at the Markdown command file to install.
- MCPs point at a single JSON object compatible with `claude mcp add-json`.

### Plugin Entries

Plugins use marketplace metadata instead of `source`.

Each plugin entry stores:
- `marketplace`
- `marketplace_source`
- optional `plugin` when the install id differs from the catalog name

`marketplace_source` should match Claude's supported marketplace source schema from the [plugins docs](https://code.claude.com/docs/en/plugins) and [settings docs](https://code.claude.com/docs/en/settings). Supported kinds include:
- `github`
- `git`
- `url`
- `npm`
- `file`
- `directory`
- `hostPattern`

### Typed Dependencies

Dependencies use typed references to avoid collisions:

```yaml
requires: [skill:base-utils, agent:reviewer, prompt:task-router, mcp:github, plugin:review-toolkit]
```

Dependencies are resolved first, recursively.

### Scopes and Targets

Source-backed items use filesystem targets from `default_dirs`.

Claude-native items use scopes from `default_scopes`:
- `project`
- `local`
- `user`

By default:
- MCPs install at `project`
- Plugins install at `project`
- "global" maps to `user`

The Library uses Claude's supported config surfaces for these:
- MCPs via [`claude mcp`](https://docs.anthropic.com/en/docs/claude-code/mcp)
- Plugins via [`claude plugin`](https://code.claude.com/docs/en/plugins) and Claude settings

The Library never treats plugin caches like `~/.claude/plugins/cache` or `installed_plugins.json` as user-managed source of truth.

## Prerequisites

- **Claude Code** with `/mcp` and `/plugin` support available
- **git** for cloning sources and syncing the catalog
- **gh** (optional) for GitHub access and private repo workflows. Install: [gh docs](https://cli.github.com)
- **GitHub SSH key or `GITHUB_TOKEN`** for private repos
- **just** (optional) for justfile shortcuts. Install: [just docs](https://github.com/casey/just)

## Installation

This is a template repo. Fork it, clone it into `~/.claude/skills/library`, and it becomes a `/library` slash command available in Claude Code.

### 1. Fork This Repo

Fork to your own GitHub account, ideally as a private repo. This fork becomes your personal or team library catalog.

```bash
gh repo fork disler/the-library --private --clone=false
```

### 2. Clone to Global Skills Directory

```bash
mkdir -p ~/.claude/skills/library
git clone <your-fork-url> ~/.claude/skills/library
```

Or:

```bash
gh repo clone <yourname>/the-library ~/.claude/skills/library
```

### 3. Configure

Open `~/.claude/skills/library/SKILL.md` and update the variables:

```markdown
- **LIBRARY_REPO_URL**: `https://github.com/yourname/the-library.git`
```

The default `LIBRARY_YAML_PATH` and `LIBRARY_SKILL_DIR` are correct if you cloned into `~/.claude/skills/library/`.

### 4. Verify

Start a new Claude Code session anywhere and run:

```text
/library list
```

You should see an empty catalog.

## Quick Start

![Full Workflow](images/45_solution_full_workflow.svg)

The usual flow is: build -> catalog -> distribute -> use.

### Add a skill

```text
/library add deploy skill from https://github.com/yourorg/infra-tools/blob/main/skills/deploy/SKILL.md
```

### Add an MCP

```text
/library add github mcp from https://github.com/yourorg/claude-config/blob/main/mcp/github.json
```

Keep MCP definitions shareable. Use env placeholders instead of raw secrets.

### Add a plugin

```text
/library add plugin named review-toolkit from marketplace team-marketplace using github repo yourorg/claude-marketplace install plugin pr-review-toolkit
```

### Use entries elsewhere

```text
/library use deploy
/library use github
/library use review-toolkit globally
```

- Skills, agents, and prompts copy into your local Claude directories.
- MCPs configure Claude using `claude mcp add-json`.
- Plugins configure marketplaces and install via Claude's native plugin flow.

### Push source-backed changes

```text
/library push deploy
```

`push` is for skills, agents, and prompts. MCPs and plugins stay source-of-truth in their JSON or marketplace repos and are not pushed via `/library push` in v1.

### Sync everything

```text
/library sync
```

This refreshes every installed skill, agent, prompt, MCP, and plugin that the current machine already has configured.

## Commands

| Command | What It Does |
| --- | --- |
| `/library install` | First-time setup - fork, clone, configure |
| `/library add <details>` | Register a new skill, agent, prompt, MCP, or plugin |
| `/library use <name>` | Install, configure, or refresh an entry |
| `/library push <name>` | Push local changes back to a source-backed entry |
| `/library remove <name>` | Remove from the catalog and optionally remove local state |
| `/library list` | Show the full catalog with install/config status |
| `/library sync` | Refresh all installed and configured items |
| `/library search <keyword>` | Find entries by name or description |

### Justfile Shortcuts

The included `justfile` lets you run Library commands from a terminal without opening an interactive Claude session.

```bash
just list
just use my-skill
just push my-skill
just add "name: foo, description: bar, source: /path/to/SKILL.md"
just sync
just search "keyword"
```

> Note: the recipes use `--dangerously-skip-permissions` because the agent needs filesystem and git access for clone, copy, and sync operations.

## Architecture

```text
~/.claude/skills/library/     # The Library skill
    SKILL.md                  # Agent instructions
    library.yaml              # Catalog of references
    cookbook/                 # Command-specific execution guides
        install.md
        add.md
        use.md
        push.md
        remove.md
        list.md
        sync.md
        search.md
    justfile                  # Terminal shortcuts
    README.md                 # This file
```

## Design Principles

- **Private-first**: Built for specialized, competitive agentics.
- **Reference-based**: The catalog stores pointers, not payload copies.
- **Claude-native where needed**: MCP and plugin sync use Claude's supported config and CLI surfaces.
- **Pure agent**: The behavior lives in markdown instructions, not custom tooling.
- **Catalog, not manifest**: Entries define what is available, not what must be installed everywhere.

## The Agentic Stack

![The Agentic Stack](images/03_agentic_stack.svg)

| Layer | Purpose |
| --- | --- |
| **Skills** | Raw capabilities |
| **Agents** | Specialization and parallelism |
| **Prompts** | Orchestration and routing |
| **MCPs** | External systems and data planes |
| **Plugins** | Bundled Claude-native extensions |
| **The Library** | Distribution and sync across all of the above |

## Master Agentic Coding

Prepare for the future of software engineering with [Tactical Agentic Coding](https://agenticengineer.com/tactical-agentic-coding?y=tlibms).

Follow the [IndyDevDan YouTube channel](https://www.youtube.com/@indydevdan) to keep sharpening your agentic engineering advantage.
