---
name: claude-marketplace-init
description: Use this skill when the user wants to create a new Claude Code plugin marketplace from scratch, scaffold a marketplace repository, initialize a marketplace structure, set up a new plugin distribution repo, or asks "how do I create a marketplace". Produces the full directory tree, marketplace.json, plugin.json stubs, and an instructions README.
version: 1.0.0
---

# Skill: Init Claude Distributed Marketplace

Scaffold a complete, git-distributable Claude Code plugin marketplace from scratch.

## What to produce

Create the following directory tree relative to the path the user specifies (default: current directory):

```
<marketplace-root>/
├── .claude-plugin/
│   └── marketplace.json          ← marketplace catalog
├── plugins/
│   └── example-plugin/
│       ├── .claude-plugin/
│       │   └── plugin.json       ← plugin identity
│       ├── skills/
│       │   └── example-skill/
│       │       └── SKILL.md
│       └── README.md
└── README.md                     ← marketplace instructions
```

## Step-by-step instructions

### 1. Gather info from the user

Ask (or infer from context) before writing any files:
- **Marketplace name** — kebab-case, no spaces. This becomes the `@marketplace-name` suffix users type when installing plugins. Must not be one of the reserved names: `claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `knowledge-work-plugins`, `life-sciences`.
- **Owner name** and optionally email.
- **Root path** — where to scaffold (default: current working directory).
- **First plugin name** — suggest an example, or create a placeholder `example-plugin` if they have nothing yet.

### 2. Create `.claude-plugin/marketplace.json`

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "<marketplace-name>",
  "description": "<one-line description of what this marketplace provides>",
  "owner": {
    "name": "<owner-name>",
    "email": "<owner-email-optional>"
  },
  "metadata": {
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "example-plugin",
      "source": "./plugins/example-plugin",
      "description": "Example plugin — replace or remove this entry",
      "author": {
        "name": "<owner-name>"
      }
    }
  ]
}
```

**Key rules to apply:**
- `"source"` is a path relative to the **marketplace root** (the directory containing `.claude-plugin/`), NOT relative to `marketplace.json` itself.
- Because `pluginRoot` is set to `"./plugins"`, plugin source paths can be shortened to `"./example-plugin"` instead of `"./plugins/example-plugin"` — mention this in the README.
- Do not add a `version` field in the marketplace entry unless the user asks for explicit versioning; omitting it means every new commit is treated as a new version automatically.

### 3. Create `plugins/example-plugin/.claude-plugin/plugin.json`

```json
{
  "name": "example-plugin",
  "description": "Example plugin — replace with real description",
  "author": {
    "name": "<owner-name>"
  }
}
```

Rules:
- `plugin.json` is **identity only** — no skills, commands, or hooks config goes here.
- Do NOT add a `version` field unless the user wants explicit version pinning. Explain: if both `plugin.json` and `marketplace.json` have `version`, `plugin.json` silently wins, which can mask marketplace version bumps.

### 4. Create `plugins/example-plugin/skills/example-skill/SKILL.md`

```markdown
---
name: example-skill
description: Replace this description with clear trigger phrases. Describe when Claude should use this skill. Example: "Use when the user asks to X, Y, or Z."
version: 1.0.0
---

# Example Skill

Replace this content with your skill's instructions.

Describe what Claude should do when this skill is active.
```

### 5. Create `plugins/example-plugin/README.md`

Brief placeholder:

```markdown
# example-plugin

Describe what this plugin does.

## Skills

- `/marketplace-name:example-skill` — describe what it does

## Installation

```shell
/plugin install example-plugin@<marketplace-name>
```
```

### 6. Create root `README.md`

Include:

```markdown
# <marketplace-name>

<description>

## Add this marketplace

```shell
# From GitHub (recommended after pushing)
/plugin marketplace add <github-owner>/<repo-name>

# Local testing before pushing
/plugin marketplace add ./path/to/this/repo
```

## Available plugins

| Plugin | Description |
|--------|-------------|
| `example-plugin` | Example plugin — replace me |

## Install a plugin

```shell
/plugin install example-plugin@<marketplace-name>
```

## Validate before publishing

```shell
claude plugin validate .
```

## Structure reference

```
<marketplace-root>/
├── .claude-plugin/
│   └── marketplace.json   ← catalog: lists plugins + their source paths
└── plugins/
    └── <plugin-name>/
        ├── .claude-plugin/
        │   └── plugin.json    ← plugin identity only (name, description, author)
        ├── skills/            ← model-invoked (Claude uses automatically)
        │   └── <skill>/
        │       └── SKILL.md
        ├── commands/          ← user-invoked (/plugin-name:command-name)
        ├── agents/
        ├── hooks/
        │   └── hooks.json
        └── .mcp.json          ← MCP server connections
```

## Key rules

- All plugin dirs (`skills/`, `commands/`, `agents/`, `hooks/`, `.mcp.json`) live at the **plugin root**, never inside `.claude-plugin/`.
- `source` paths in `marketplace.json` resolve from the marketplace root (next to `.claude-plugin/`), not from inside `.claude-plugin/`.
- Use `${CLAUDE_PLUGIN_ROOT}` in `.mcp.json` and `hooks/hooks.json` for paths — plugins are copied to a cache dir on install.
- Run `claude plugin validate .` before pushing.
```

### 7. Post-scaffold summary

After creating all files, show the user:

1. The exact tree that was created.
2. The next steps:
   ```
   # Validate
   claude plugin validate .

   # Test locally
   /plugin marketplace add ./this-directory
   /plugin install example-plugin@<marketplace-name>

   # After pushing to GitHub
   /plugin marketplace add <github-owner>/<repo-name>
   ```
3. Remind them:
   - Relative `source` paths only work when the marketplace is added via git (not via direct URL to `marketplace.json`).
   - Plugin caches at `~/.claude/plugins/cache` — use `/reload-plugins` after local edits.
   - To share with a team, commit `.claude/settings.json` with `extraKnownMarketplaces`.
