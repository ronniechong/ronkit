# ronkit — Claude Code Plugin Marketplace

## What this project is

**ronkit** is Ronnie's public Claude Code plugin marketplace: a GitHub monorepo distributing skills, slash commands, agents, and hooks. The marketplace is an *index*, not a host — `.claude-plugin/marketplace.json` at the repo root registers plugins that live under `plugins/`.

Install flow for end users:

```bash
/plugin marketplace add <github-user>/ronkit
/plugin install <plugin-name>@ronkit
```

## Repository layout

```
ronkit/
├── .claude-plugin/
│   └── marketplace.json        # registry — REQUIRED at repo root, name: "ronkit"
├── plugins/
│   └── <plugin-name>/
│       ├── .claude-plugin/plugin.json
│       ├── skills/<skill-name>/SKILL.md
│       ├── commands/<command>.md      # optional
│       ├── agents/<agent>.md          # optional
│       ├── hooks/hooks.json           # optional
│       └── scripts/                   # optional
├── .github/workflows/validate.yml
└── README.md
```

## Plugin organization: category, not one-per-skill

Plugin skills are **always** namespaced as `/plugin-name:skill-name` on invocation — even with zero name collisions, there is no bare `/skill-name` and no alias mechanism for plugin-sourced skills (confirmed against docs, not configurable). Because of this, plugin names are **categories/scopes** that group related skills, not a 1:1 wrapper around a single skill — a plugin named the same as its only skill produces an awkward `/skill:skill` invocation.

Current categories:
- `test` — skills for testing/validating the marketplace itself (e.g. `hello`)
- `documents` — skills for project documentation workflows (e.g. `resume-later`)

When adding a new skill, put it in the category it fits (adding a `skills/<name>/` subdirectory to the existing plugin) rather than creating a new single-skill plugin. Only create a new category plugin when the skill doesn't fit any existing one.

## Hard rules

- Marketplace name is `ronkit`. Never change it — it's part of every user's install string.
- Plugin names: lowercase kebab-case, one per **category** (see above) — not one per skill. Semantic versioning in both plugin.json and the marketplace entry — **update both together** on every release.
- `metadata.pluginRoot` is `./plugins`; plugin `source` values are relative paths under it.
- Plugins are copied to a cache on install. **Never reference files outside a plugin's own directory** (no `../shared`). Duplicate or symlink instead.
- All JSON must validate. Run `claude plugin validate . --strict` (and per-plugin) before any commit that touches marketplace.json or a plugin.json — CI (`.github/workflows/validate.yml`) re-runs this and the version-consistency check on every PR/push to `main`, but don't rely on CI to catch it first.
- Skills load from a plugin's `skills/` directory. Each skill's SKILL.md frontmatter `description` is the trigger mechanism — write it as a routing rule: what the skill does, when to use it, concrete trigger phrases. Vague descriptions are bugs.
- README plugin table must stay in sync with marketplace.json entries.

## Version bumping — MANDATORY on every change

Any change to a plugin's contents (skills, commands, agents, hooks, scripts, or its plugin.json) MUST bump that plugin's version in the **same commit**, in **both** places:

1. `plugins/<name>/.claude-plugin/plugin.json` → `version`
2. `.claude-plugin/marketplace.json` → the matching plugin entry's `version`

Semver rules:
- **patch** (0.1.0 → 0.1.1): fixes, wording tweaks, doc updates inside the plugin
- **minor** (0.1.0 → 0.2.0): new skill/command/agent/hook, new capability, non-breaking behavior change
- **major** (0.2.0 → 1.0.0): breaking change — renamed/removed component, changed command interface, changed skill trigger contract

Before finishing any task that touched a plugin, verify: `git diff` shows both version fields changed and they match. A plugin change without a version bump is an incomplete task. Changes that touch only the repo root (README, CI, this file) do not require plugin bumps; a marketplace-level change may bump `metadata.version` instead. CI enforces the plugin.json/marketplace.json match on every PR and fails the build on mismatch — treat that as a backstop, not the first line of defense.

## README requirements

The repo README.md must always contain, and be kept current with every change:

1. **What ronkit is** — one-paragraph description.
2. **Installation** — exact commands:
   ```bash
   /plugin marketplace add <github-user>/ronkit
   /plugin install <plugin-name>@ronkit
   ```
3. **Plugin catalog table** — one row per plugin: name, current version, description, install command. Versions here must match marketplace.json.
4. **Per-plugin usage** — a section for each plugin: what it does, how to trigger its skills (example prompts), any slash commands with example invocations, configuration if any.
5. **Updating** — how users update (`/plugin marketplace update ronkit`, reinstall the plugin).
6. **Local development** — the dev/test loop from this file.

When adding or changing a plugin, updating the README catalog row and usage section is part of the same task — not a follow-up.

## Manifests

### marketplace.json shape

```json
{
  "name": "ronkit",
  "owner": { "name": "Ronnie" },
  "metadata": {
    "description": "Skills and plugins for agentic workflows",
    "version": "1.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "example-plugin",
      "source": "./plugins/example-plugin",
      "description": "One-line description",
      "version": "0.1.0",
      "keywords": ["..."]
    }
  ]
}
```

### plugin.json shape

```json
{
  "$schema": "https://json.schemastore.org/claude-code-plugin-manifest.json",
  "name": "example-plugin",
  "version": "0.1.0",
  "description": "One-line description",
  "author": { "name": "Ronnie" },
  "repository": "https://github.com/<github-user>/ronkit",
  "license": "MIT"
}
```

Tiny skill-only plugins may use `"strict": false` in the marketplace entry and skip plugin.json — but default to having a plugin.json for consistency.

## Dev & test loop

From the repo root:

```bash
claude
/plugin marketplace add .
/plugin install <plugin-name>@ronkit
# after changes:
/plugin uninstall <plugin-name>@ronkit
/plugin install <plugin-name>@ronkit
```

A plugin is not "done" until it installs cleanly from a fresh clone and its skill actually triggers on a representative prompt.

## Conventions

- This repo only carries a plain README — no planning docs, milestone history, or feature specs live here; they're tracked privately outside the repo.
- Conventional commits from day one (`feat:`, `fix:`, `docs:`, `chore:`). Versioning is manual (see "Version bumping" above) — Release Please was considered and dropped at M4 (private, single-contributor repo; not worth the added moving parts at this scale).
- Keep plugins organized by category (see "Plugin organization" above); add skills to the fitting category plugin rather than creating a new plugin per skill.
- Document every plugin in its own README section: what it does, install command, example usage per skill.
- When uncertain about schema details, check the official docs: https://code.claude.com/docs/en/plugin-marketplaces — do not guess fields.
