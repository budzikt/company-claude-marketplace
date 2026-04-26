---
name: claude-marketplace-convert-skill-to-marketplace
description: Use this skill when the user wants to convert an existing skill, command, set of skills, or standalone `.claude/` configuration into a distributable Claude Code marketplace plugin. Triggers on phrases like "convert my skills to a marketplace", "package my commands for distribution", "make my skills shareable", "wrap my skill in a plugin", "I want to distribute my existing skill", or "how do I publish my skill".
version: 1.0.0
---

# Skill: Convert Skill(s) to Marketplace Plugin

Convert existing standalone skills or `.claude/` configurations into a distributable marketplace plugin with the correct structure.

## Discovery phase — read before touching anything

Before creating any files, inspect the project:

1. **Check `.claude/` for existing standalone config:**
   - `.claude/commands/` — flat `.md` files (user-invoked slash commands)
   - `.claude/skills/` — `<name>/SKILL.md` folders (model-invoked skills)
   - `.claude/settings.json` — check for `hooks` key
   - `.claude/agents/` — agent definitions

2. **Check the project root** for any loose skill or command files the user may be pointing to.

3. **Ask the user** if discovery is ambiguous:
   - Which skills/commands to include?
   - What to name the plugin? (kebab-case, this becomes the namespace prefix)
   - Where to put the plugin directory? (default: `./plugins/<plugin-name>/`)
   - Should a `marketplace.json` be created too, or just the plugin directory?
   - Is there an existing marketplace to add this plugin to?

## Conversion rules

### `.claude/commands/` → `plugins/<name>/commands/`

Flat `.md` files map 1:1. Copy them as-is — the format is identical.

```
.claude/commands/deploy.md  →  plugins/<name>/commands/deploy.md
```

After conversion the command becomes `/plugin-name:deploy` instead of `/deploy`. Tell the user this and note they can remove the original to avoid duplicates.

### `.claude/skills/<skill-name>/SKILL.md` → `plugins/<name>/skills/<skill-name>/SKILL.md`

Copy entire skill directories. Content is identical; only the invocation namespace changes.

### Hooks in `.claude/settings.json` → `plugins/<name>/hooks/hooks.json`

Extract the `hooks` object from settings.json and write it to `hooks/hooks.json` with this wrapper:

```json
{
  "hooks": {
    <paste the hooks object content here>
  }
}
```

The format inside is the same — no transformation needed.

### Agents in `.claude/agents/` → `plugins/<name>/agents/`

Copy agent `.md` files directly.

## Files to create

### `plugins/<name>/.claude-plugin/plugin.json`

```json
{
  "name": "<plugin-name>",
  "description": "<concise description of what these skills/commands do>",
  "author": {
    "name": "<infer from git config or ask>"
  }
}
```

Do NOT copy skills/commands config into `plugin.json` — it is identity metadata only. The directory structure is what Claude Code reads.

### `marketplace.json` (create or update)

If a `.claude-plugin/marketplace.json` already exists, **add** the new plugin entry to the `plugins` array. If none exists, create it at the project root's `.claude-plugin/marketplace.json`:

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

Source path rule: **relative to the marketplace root** (where `.claude-plugin/` lives), not relative to `marketplace.json` itself.

## Resulting structure

```
<marketplace-root>/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── <plugin-name>/
        ├── .claude-plugin/
        │   └── plugin.json
        ├── skills/            ← from .claude/skills/ (model-invoked)
        │   └── <skill-name>/
        │       └── SKILL.md
        ├── commands/          ← from .claude/commands/ (user-invoked)
        │   └── deploy.md
        ├── agents/            ← from .claude/agents/
        └── hooks/             ← extracted from .claude/settings.json
            └── hooks.json
```

## What changes after conversion

| Before | After |
|--------|-------|
| `/deploy` | `/plugin-name:deploy` |
| `/skill-name` | `/plugin-name:skill-name` |
| Only available in this project | Installable from marketplace |
| Hooks in `settings.json` | Hooks in `hooks/hooks.json` |

## Post-conversion steps to communicate

1. **Validate**: `claude plugin validate .`
2. **Test locally**: `/plugin marketplace add ./` then `/plugin install <plugin-name>@<marketplace-name>`
3. **Reload**: `/reload-plugins` after any local edits
4. **Remove originals** from `.claude/` to avoid duplicate invocations (optional — plugin takes precedence when loaded, but duplicates can confuse)
5. **Push to git** and add with: `/plugin marketplace add <github-owner>/<repo>`

## Edge cases to handle

- **Skill `SKILL.md` missing `description` frontmatter**: warn the user — Claude won't know when to auto-invoke the skill without it.
- **Commands referencing `$ARGUMENTS`**: these work identically in plugins, no change needed.
- **Hooks using relative paths** (e.g., `./scripts/lint.sh`): replace with `${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh` — plugins are copied to a cache dir, so relative paths from the project root won't resolve.
- **Multiple skill sets for different purposes**: suggest splitting into separate plugins with distinct names rather than one monolith.
