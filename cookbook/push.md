# Push a Library Item Back to Its Source

## Context
The user has improved a locally installed skill, agent, or prompt and wants to push changes back to the source.

## Input
The user provides an item name or description.

## Steps

### 1. Find the Entry
- Read `library.yaml`
- Search across all sections for the matching entry
- If no match, tell the user the item was not found in the catalog

### 2. Check Type Support
- `push` is supported only for source-backed entries:
  - skills
  - agents
  - prompts
- If the entry is an `mcp` or `plugin`, stop and tell the user:
  - MCPs are shareable config templates that should be edited at their source JSON file
  - Plugins are cataloged as marketplace references, not local runtime payloads
  - `/library push` for MCPs and plugins is not supported in v1

### 3. Locate the Local Copy
- Check the default directory for the type (from `default_dirs`)
- Check the global directory
- If found in multiple places, ask which one to push
- If not found locally, tell the user there is nothing to push

### 4. Check for Conflicts

**If source is a local path**
- Compare the local installed copy with the source
- If the source has changed since last pull, warn the user before overwriting it

**If source is a GitHub URL**
- Clone the repo to a temporary directory:
  ```bash
  tmp_dir=$(mktemp -d)
  git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
  ```
- Compare the local copy with the referenced directory or file in the clone
- If the remote has changes not present locally, warn the user and ask them to resolve the conflict first

### 5. Push to Source

**If source is a local path**
- For skills, copy the full local directory back to the source parent directory
- For agents and prompts, copy the local file back to the source file path

**If source is a GitHub URL**
- Use the temporary clone from step 4, or create it now
- Overwrite only the referenced item:
  - skills → replace the referenced directory
  - agents and prompts → replace the referenced file
- Stage only the relevant path
- Commit with:
  ```bash
  git commit -m "library: updated <name> <brief description>"
  ```
- Push the changes
- Clean up the temporary clone

### 6. Confirm
Tell the user:
- What was pushed and where
- The commit message used
- If the source was a local path, confirm the overwrite
