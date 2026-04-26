# company-claude-marketplace

Claude Code marketplace tools — skills for scaffolding, converting, and verifying Claude plugin marketplaces.

## Add this marketplace

```shell
# From GitHub
/plugin marketplace add budzikt/company-claude-marketplace

# Local testing
/plugin marketplace add ./path/to/this/repo
```

## Available plugins

| Plugin | Description |
|--------|-------------|
| `claude-marketplace-tools` | Skills for creating, converting, and validating Claude Code plugin marketplaces |

## Install a plugin

```shell
/plugin install claude-marketplace-tools@company-claude-marketplace
```

## Validate before publishing

```shell
claude plugin validate .
```

## Structure reference

```
company-claude-marketplace/
├── .claude-plugin/
│   └── marketplace.json        ← catalog: lists plugins + source paths
└── plugins/
    └── claude-marketplace-tools/
        ├── .claude-plugin/
        │   └── plugin.json     ← plugin identity only
        ├── skills/             ← model-invoked (Claude uses automatically)
        │   ├── claude-marketplace-init/
        │   ├── claude-marketplace-convert-mcp-to-marketplace/
        │   ├── claude-marketplace-convert-skill-to-marketplace/
        │   └── claude-marketplace-verify-structure/
        └── README.md
```

## Key rules

- `source` paths in `marketplace.json` resolve from the marketplace root, not from inside `.claude-plugin/`.
- Relative `source` paths only work when the marketplace is added via git (not via direct URL to `marketplace.json`).
- Plugin caches at `~/.claude/plugins/cache` — use `/reload-plugins` after local edits.
- To share with a team, commit `.claude/settings.json` with `extraKnownMarketplaces`.
