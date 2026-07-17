# ronkit

Ronnie's public Claude Code plugin marketplace — skills, slash commands, agents, and hooks, installable in two commands.

## Installation

```bash
/plugin marketplace add ronniechong/ronkit
/plugin install <plugin-name>@ronkit
```

## Plugins

| Plugin | Version | Description | Install |
|---|---|---|---|
| `hello` | 0.1.0 | Skeleton plugin used to validate the marketplace install flow | `/plugin install hello@ronkit` |

### hello

Smoke-test plugin. After installing, trigger it with `/hello` or by asking to "test the ronkit marketplace" — it should reply with a fixed confirmation string.

## Updating

```bash
/plugin marketplace update ronkit
```
then reinstall the plugin you want to refresh.

## Local development

From the repo root:

```bash
claude
/plugin marketplace add .
/plugin install <plugin-name>@ronkit
# after changes:
/plugin uninstall <plugin-name>@ronkit
/plugin install <plugin-name>@ronkit
```
