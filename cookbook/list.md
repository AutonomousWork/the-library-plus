# List Available Library Entries

## Context
Show the full library catalog with install status for skills, agents, prompts, MCPs, and plugins.

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

### 3. Check Install Status

**For skills, agents, and prompts**
- Determine the type and corresponding directories from `default_dirs`
- Check for matches in the default and global directories
- Search recursively for name matches when needed
- Mark as `installed (default)`, `installed (global)`, or `not installed`

**For MCPs**
- Run `claude mcp get <name>`
- If it succeeds, mark as `configured` and include the detected scope when Claude reports it
- If it fails, mark as `not configured`

**For plugins**
- Run `claude plugin list --json`
- Match by plugin id: `<plugin-or-name>@<marketplace>`
- Mark as:
  - `installed (<scope>, enabled)` when present and enabled
  - `installed (<scope>, disabled)` when present and disabled
  - `not installed` when absent

### 4. Display Results
Format the output as grouped tables:

```
## Skills
| Name | Description | Source | Status |

## Agents
| Name | Description | Source | Status |

## Prompts
| Name | Description | Source | Status |

## MCPs
| Name | Description | Source | Status |

## Plugins
| Name | Description | Marketplace | Status |
```

If a section is empty, show `No <type> in catalog.`

### 5. Summary
At the bottom, show:
- Total entries in catalog
- Total installed or configured locally
- Total not installed
