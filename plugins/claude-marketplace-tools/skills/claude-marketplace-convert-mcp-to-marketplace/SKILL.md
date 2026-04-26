---
name: claude-marketplace-convert-mcp-to-marketplace
description: Use this skill when the user wants to convert a standalone MCP server configuration into a distributable Claude Code marketplace plugin. Triggers on phrases like "convert my MCP to a plugin", "package my MCP server for distribution", "make my MCP shareable", "wrap MCP in a marketplace plugin", "publish my MCP config", "distribute my MCP server", or "I have an MCP and want to share it".
version: 1.0.0
---

# Skill: Convert Standalone MCP to Marketplace Plugin

Convert an existing MCP server configuration into a proper Claude Code plugin that can be distributed through a marketplace.

## Discovery phase — read before touching anything

Locate the existing MCP configuration. Check in this order:

1. **`.claude/settings.json`** — look for `mcpServers` key (global Claude Code MCP config)
2. **`.mcp.json`** at the project root — project-scoped MCP config
3. **User-provided path** — if they point to a specific file

Read the MCP entry carefully. Identify:
- Server name(s) — these become the keys in the plugin's `.mcp.json`
- Server type: `http`, `sse`, or process-based (has `command` + `args`)
- Any environment variable references (`${VAR_NAME}`)
- Any local file paths in `command` or `args`

Ask the user:
- Which MCP server(s) to include? (if multiple exist)
- What to name the plugin? (kebab-case, e.g., `asana-mcp`, `my-db-tools`)
- Plugin description?
- Should a marketplace also be created, or just the plugin directory?
- For local process servers: is the server binary/script inside the project, or system-installed?

## MCP type classification

### Type A — Remote HTTP/SSE server (simplest)

Source config looks like:
```json
"my-server": { "type": "http", "url": "https://api.example.com/mcp" }
"my-server": { "type": "sse", "url": "https://api.example.com/sse" }
```

These copy directly into the plugin's `.mcp.json` unchanged. No path rewriting needed.

### Type B — Remote with auth headers

```json
"my-server": {
  "type": "http",
  "url": "https://api.example.com/mcp",
  "headers": { "Authorization": "Bearer ${MY_API_KEY}" }
}
```

Copy as-is. `${MY_API_KEY}` is an environment variable — the user must set it in their shell. Document this in the plugin README.

### Type C — Local process server

```json
"my-server": {
  "command": "node",
  "args": ["./server/index.js", "--port", "3000"]
}
```

**Critical**: The plugin is copied to a cache directory on install (`~/.claude/plugins/cache/...`). Any relative path in `args` that points to project files will break.

Rewrite paths using `${CLAUDE_PLUGIN_ROOT}`:
```json
"my-server": {
  "command": "node",
  "args": ["${CLAUDE_PLUGIN_ROOT}/server/index.js", "--port", "3000"]
}
```

**Also ask**: Is the server script/binary something that should be bundled with the plugin, or is it a system binary (like `npx`, `uvx`, `bun`)? If it needs bundling, the server files must be inside the plugin directory.

### Type D — npx/uvx/bun-based server

```json
"my-server": {
  "command": "npx",
  "args": ["-y", "@org/mcp-server"]
}
```

These are self-installing — copy as-is. No path rewriting needed.

## Files to create

### `plugins/<plugin-name>/.claude-plugin/plugin.json`

```json
{
  "name": "<plugin-name>",
  "description": "<what this MCP server provides to Claude>",
  "author": {
    "name": "<owner>"
  }
}
```

### `plugins/<plugin-name>/.mcp.json`

Write the converted MCP server config. Apply path rewrites from the type classification above.

Example for a remote SSE server (like Asana):
```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse"
  }
}
```

Example for a local node server:
```json
{
  "my-server": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/server/index.js"]
  }
}
```

### `plugins/<plugin-name>/README.md`

```markdown
# <plugin-name>

<description>

## Requirements

<!-- List env vars, system dependencies, or auth steps -->
- Set `MY_API_KEY` environment variable  ← only if applicable

## Installation

```shell
/plugin install <plugin-name>@<marketplace-name>
```

## MCP server

Provides the `<server-name>` MCP server.
Adds the following tools to Claude: <!-- list if known -->
```

### `marketplace.json` (create or update)

If `.claude-plugin/marketplace.json` exists, add to the `plugins` array. Otherwise create:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "<marketplace-name>",
  "owner": { "name": "<owner>" },
  "plugins": [
    {
      "name": "<plugin-name>",
      "source": "./plugins/<plugin-name>",
      "description": "<description>"
    }
  ]
}
```

## Resulting structure

### Minimal (remote server — Type A/B)
```
<marketplace-root>/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── <plugin-name>/
        ├── .claude-plugin/
        │   └── plugin.json
        └── .mcp.json
```

### With bundled server (Type C)
```
<marketplace-root>/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── <plugin-name>/
        ├── .claude-plugin/
        │   └── plugin.json
        ├── .mcp.json
        └── server/              ← server files go here
            └── index.js
```

## Post-conversion checklist to walk through with the user

1. **Path check**: If the server uses local files, confirm they are inside the plugin directory and paths use `${CLAUDE_PLUGIN_ROOT}`.
2. **Env vars**: List any `${VAR_NAME}` references — the user must set these in their shell before Claude Code starts.
3. **Validate**: `claude plugin validate .`
4. **Test locally**:
   ```shell
   /plugin marketplace add ./
   /plugin install <plugin-name>@<marketplace-name>
   ```
5. **Verify MCP loaded**: Check that the MCP server tools appear after install (Claude should mention them, or user can check Claude's tool list).
6. **Push to git** and distribute: `/plugin marketplace add <github-owner>/<repo>`

## Key difference from standalone MCP

| Standalone (`settings.json`) | Plugin (`.mcp.json`) |
|---|---|
| One project only | Installable from any project |
| Short server name in Claude | Same server name |
| Relative paths work | Must use `${CLAUDE_PLUGIN_ROOT}` for paths |
| No versioning | Version via `plugin.json` or git SHA |
