---
name: claude-marketplace-grouped-plugins
description: >
  How to group Claude Code skills into selectively installable packages using the native
  plugin system. Use when asked about grouping skills, skill bundles, installable skill
  packages, private team marketplaces, or selective skill activation. The grouping
  mechanism is a plugin — plugins bundle skills; marketplaces distribute plugins.
---

# Claude Marketplace — Grouping Skills via Plugins

## Core concept

The native Claude Code marketplace has a three-level hierarchy:

| Unit | What it is | Installable? |
|---|---|---|
| **Skill** | Single `SKILL.md` instruction set | No — part of a plugin |
| **Plugin** | Bundle of skills, commands, agents, hooks, MCP servers | ✅ Yes |
| **Marketplace** | Git repo catalog listing plugins | Registered, not installed |

**Grouping mechanism = Plugin.** There is no separate "group" or "package" concept. If you want users to install a subset of your skills, wrap each subset in a plugin.

---

## Repository structure

One Git repo = one marketplace. Plugins live as subdirectories inside it.

```
company-marketplace/
├── .claude-plugin/
│   └── marketplace.json          ← marketplace catalog
└── plugins/
    ├── plugin-alpha/
    │   ├── .claude-plugin/
    │   │   └── plugin.json       ← plugin manifest
    │   └── skills/
    │       ├── skill-one/
    │       │   └── SKILL.md
    │       └── skill-two/
    │           └── SKILL.md
    └── plugin-beta/
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            ├── skill-three/
            │   └── SKILL.md
            └── skill-four/
                └── SKILL.md
```

---

## File contents

### `.claude-plugin/marketplace.json`

```json
{
  "name": "company-marketplace",
  "owner": {
    "name": "Platform Team",
    "email": "platform@company.com"
  },
  "metadata": {
    "description": "Internal company plugins",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "plugin-alpha",
      "source": "./plugins/plugin-alpha",
      "description": "Alpha domain tools"
    },
    {
      "name": "plugin-beta",
      "source": "./plugins/plugin-beta",
      "description": "Beta domain tools"
    }
  ]
}
```

### `plugins/plugin-alpha/.claude-plugin/plugin.json`

```json
{
  "name": "plugin-alpha",
  "version": "1.0.0",
  "description": "Alpha domain tools",
  "author": {
    "name": "Platform Team"
  }
}
```

---

## Hosting and installation

Any Git host works — GitHub, GitLab, Bitbucket, self-hosted. The `owner/repo` shorthand only works for GitHub; all others require the full URL.

```bash
# Register the marketplace (once per user/project)
/plugin marketplace add https://gitlab.company.com/team/company-marketplace.git

# User installs only the group(s) they need
/plugin install plugin-alpha@company-marketplace
/plugin install plugin-beta@company-marketplace
```

For team-wide auto-registration, add to `.claude/settings.json` in any project repo:

```json
{
  "extraKnownMarketplaces": {
    "company-marketplace": {
      "source": {
        "source": "url",
        "url": "https://gitlab.company.com/team/company-marketplace.git"
      }
    }
  }
}
```

---

## Key rules

- **Bump `version`** in `plugin.json` on every release — unchanged version = no update delivered to users.
- **Plugins can't reference files outside their directory** — they are copied to a cache on install; use symlinks for shared assets.
- **`pluginRoot`** in `marketplace.json` is a convenience shorthand: set it to `"./plugins"` and write `"source": "plugin-alpha"` instead of `"source": "./plugins/plugin-alpha"`.
- **Relative path sources only work with Git-hosted marketplaces** — not with URL-only (`marketplace.json` direct URL) distribution.

---

## Detailed reference

See `references/marketplace-schema.md` for the full `marketplace.json` and `plugin.json` schema including all optional fields (category, tags, homepage, hooks, MCP servers, LSP servers, strict mode, version pinning, release channels).