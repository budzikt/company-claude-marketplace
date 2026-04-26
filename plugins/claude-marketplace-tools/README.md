# claude-marketplace-tools

Skills for creating, converting, and validating Claude Code plugin marketplaces.

## Skills

- `claude-marketplace-init` — Scaffold a new Claude Code marketplace repository from scratch
- `claude-marketplace-convert-mcp-to-marketplace` — Convert a standalone MCP server config into a distributable marketplace plugin
- `claude-marketplace-convert-skill-to-marketplace` — Package existing skills/commands into a distributable marketplace plugin
- `claude-marketplace-verify-structure` — Audit a marketplace or plugin to confirm it will work when pushed to git

## Installation

```shell
/plugin install claude-marketplace-tools@company-claude-marketplace
```

## Usage

These are model-invoked skills — Claude activates them automatically when you ask things like:

- "Init a marketplace repo" → triggers `claude-marketplace-init`
- "Convert my MCP to a plugin" → triggers `claude-marketplace-convert-mcp-to-marketplace`
- "Package my skills for distribution" → triggers `claude-marketplace-convert-skill-to-marketplace`
- "Verify my marketplace structure" / "Is my plugin ready to publish?" → triggers `claude-marketplace-verify-structure`
