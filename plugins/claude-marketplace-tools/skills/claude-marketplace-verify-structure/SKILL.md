---
name: claude-marketplace-verify-structure
description: Use this skill when the user wants to verify their marketplace or plugin will work correctly after pushing to a remote git repository, check if their marketplace structure is valid, audit their plugin distribution setup, or asks "will my marketplace work on GitHub", "is my plugin structure correct", "validate my marketplace before publishing", "check my plugin before distributing", or "marketplace readiness check".
version: 1.0.0
---

# Skill: Verify Marketplace Structure

Audit a Claude Code plugin marketplace to confirm it will work correctly when hosted on a remote git repository (GitHub, GitLab, etc.). This goes beyond JSON syntax — it checks the structural contracts that Claude Code enforces at install time.

## How to run this verification

Read the project directory structure. Start from the user's specified root, or scan upward from the current directory to find `.claude-plugin/marketplace.json`.

Run every check below. Report results as a table or checklist with PASS / WARN / FAIL per item, then a summary with specific fixes for anything that isn't PASS.

---

## Check 1 — Marketplace manifest presence and location

**PASS**: `.claude-plugin/marketplace.json` exists at the marketplace root.
**FAIL**: File is missing, or placed at the wrong path (e.g., `marketplace.json` at root without the `.claude-plugin/` directory).

Fix: Create `.claude-plugin/` directory and move `marketplace.json` inside it.

---

## Check 2 — Marketplace JSON validity and required fields

Read `.claude-plugin/marketplace.json` and verify:

- Valid JSON (no syntax errors)
- `name` field present, kebab-case (lowercase letters, digits, hyphens only — no spaces, no underscores, no uppercase)
- `name` is not a reserved name: `claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `knowledge-work-plugins`, `life-sciences`
- `owner` object present with `name` field
- `plugins` array present and non-empty (warn if empty)

**WARN** (non-blocking): Missing `metadata.description`.

---

## Check 3 — Plugin source path resolution

For each plugin entry in `plugins`:

**Relative path sources** (strings starting with `./`):

1. Resolve the path from the **marketplace root** (the directory containing `.claude-plugin/`), NOT from inside `.claude-plugin/`.
   - Example: `"source": "./plugins/my-plugin"` → check `<marketplace-root>/plugins/my-plugin/` exists.
   - Common mistake: accidentally resolving from `.claude-plugin/` would look for `.claude-plugin/plugins/my-plugin/` — that's wrong.
2. Verify the resolved directory exists on disk.
3. Verify no `..` segments in the path — these are blocked and will fail at install time.

**WARN**: Relative paths only work when the marketplace is added via git clone (i.e., `owner/repo` or git URL). If the user plans to distribute via a direct URL to `marketplace.json`, relative paths will not resolve — they must switch to `github`, `url`, or `npm` sources.

**Remote sources** (`github`, `url`, `git-subdir`, `npm`): These resolve at install time from the network. Verify the object has required fields:
- `github`: `repo` field present in `owner/repo` format
- `url`: `url` field present, starts with `https://` or `git@`
- `git-subdir`: `url` and `path` fields both present
- `npm`: `package` field present

---

## Check 4 — Plugin manifest files

For each plugin that has a local path source (relative `./` path):

Verify `<plugin-dir>/.claude-plugin/plugin.json` exists and:
- Valid JSON
- `name` field present and matches the plugin entry name in `marketplace.json`
- `description` field present

**WARN** if `version` is set in both `plugin.json` and the marketplace entry for the same plugin — `plugin.json` silently wins, which can mask version bumps in the marketplace.

---

## Check 5 — Component directory placement

For each local plugin, verify no component directories are **inside** `.claude-plugin/`:

Scan for these paths and flag if found:
- `.claude-plugin/skills/` → FAIL (must be at plugin root: `skills/`)
- `.claude-plugin/commands/` → FAIL
- `.claude-plugin/agents/` → FAIL
- `.claude-plugin/hooks/` → FAIL
- `.claude-plugin/.mcp.json` → FAIL

These directories must be at the **plugin root level**, not inside `.claude-plugin/`.

---

## Check 6 — Skills structure

For each `skills/<name>/` directory found in any local plugin:

- `SKILL.md` file exists inside the skill folder
- `SKILL.md` has YAML frontmatter (starts with `---`)
- Frontmatter contains `description` field — **WARN if missing** (Claude won't know when to auto-invoke)
- `description` is non-empty and describes trigger conditions

**WARN**: If `description` doesn't include trigger phrases or conditions, the skill may never be auto-invoked by Claude.

---

## Check 7 — Commands structure

For each `commands/` directory in local plugins:

- Files are `.md` format
- Each file has frontmatter with a `description` field (warn if missing — shown in `/help`)
- No subdirectories with nested `SKILL.md` inside `commands/` (that's the `skills/` pattern; mixing them is confusing)

---

## Check 8 — Hooks validity

For each local plugin with a `hooks/hooks.json`:

- Valid JSON
- Has top-level `hooks` key
- Hook entries have `matcher` and `hooks` array with `type` and `command` fields
- **CRITICAL**: Check for hardcoded relative paths in hook `command` values (e.g., `./scripts/lint.sh`). These will break after install because plugins are copied to a cache directory.
  - FAIL if relative paths are found without `${CLAUDE_PLUGIN_ROOT}` prefix.
  - Fix: replace `./scripts/lint.sh` with `${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh`

---

## Check 9 — MCP config validity

For each local plugin with a `.mcp.json`:

- Valid JSON
- Each server entry is either:
  - A remote server: has `type` (`http` or `sse`) and `url`
  - A local process server: has `command` and `args`
- **CRITICAL**: Check for relative paths in `command` or `args` values that don't use `${CLAUDE_PLUGIN_ROOT}`. Flag with FAIL if found — these break after install.
  - Fix: `"./server/index.js"` → `"${CLAUDE_PLUGIN_ROOT}/server/index.js"`
- Env var references like `${MY_API_KEY}` are fine — **WARN** to document them in the plugin README so users know to set them.

---

## Check 10 — Version strategy consistency

Review the version setup across `plugin.json` and marketplace entries:

- If `version` is in `plugin.json` only → PASS (standard)
- If `version` is in marketplace entry only → PASS (standard)
- If `version` is in **both** for the same plugin → WARN: `plugin.json` version silently overrides the marketplace version; bump `plugin.json` on every release or remove it from one location
- If `version` is in **neither** → PASS (git SHA auto-versioning); note that users get updates on every commit

---

## Check 11 — Git-distributability

Verify the repo is ready for git distribution:

- `.claude-plugin/marketplace.json` is not in `.gitignore`
- Plugin directories are not in `.gitignore`
- No plugin files reference paths outside the plugin directory (no `../` in `.mcp.json`, `hooks.json`, or skill files)
- If the user plans to use `git-subdir` source type: the plugin directory must be self-contained (no deps on sibling dirs)

---

## Summary report format

After running all checks, present:

```
MARKETPLACE VERIFICATION REPORT
================================
Marketplace: <name>
Path: <path>

CHECKS
------
[ PASS ] Manifest location (.claude-plugin/marketplace.json)
[ PASS ] JSON valid + required fields
[ FAIL ] Plugin source paths — plugins/my-plugin/ not found
[ PASS ] Plugin manifest files
[ WARN ] Skills missing description frontmatter — skills/my-skill/SKILL.md
[ PASS ] Component placement (nothing inside .claude-plugin/)
[ FAIL ] Hook command uses relative path — replace ./scripts/lint.sh with ${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh
[ WARN ] Version in both plugin.json and marketplace entry for "my-plugin"
[ PASS ] Git-distributability

RESULT: NOT READY (2 failures, 2 warnings)

FIXES REQUIRED
--------------
1. plugins/my-plugin/ directory does not exist — create it or fix the source path in marketplace.json
2. hooks/hooks.json line 8: "./scripts/lint.sh" must be "${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh"
```

After the report, offer to fix any FAIL items automatically if the user consents.
