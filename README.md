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
| `resume-later` | 0.1.0 | Session-handoff workflow for multi-milestone projects | `/plugin install resume-later@ronkit` |

### hello

Smoke-test plugin. After installing, trigger it with `/hello` or by asking to "test the ronkit marketplace" — it should reply with a fixed confirmation string.

### resume-later

Keeps a project's living planning doc current across sessions, so clearing context between milestones never loses state.

Trigger it either way — both run the same workflow:
- **Say it**: mention you're wrapping up, pausing, or about to clear context. Claude may also raise it proactively when it notices a milestone just finished or the session's gotten long.
- **Run it**: `/resume-later`

It finds your project's planning doc (asking if ambiguous), folds in anything undocumented (discoveries, decisions, milestone status), appends a concise History Log entry with a "what's next" note, flags any uncommitted git work, and asks for confirmation before it's safe to `/clear`.

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
