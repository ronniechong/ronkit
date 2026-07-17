# ronkit

Ronnie's Claude Code plugin marketplace — skills, slash commands, agents, and hooks, installable in two commands. Currently private; installation requires access to this repo.

Plugins here are organized by **category**, not one-plugin-per-skill: each plugin is a scope (`documents`, `test`, ...) that groups related skills. This keeps slash invocation (`/plugin-name:skill-name` — plugin skills are always namespaced, even with zero name collisions) meaningful instead of repeating the same word twice.

## Installation

```bash
/plugin marketplace add ronniechong/ronkit
/plugin install <plugin-name>@ronkit
```

## Plugins

| Plugin | Version | Description | Install |
|---|---|---|---|
| `test` | 0.1.0 | Skills for testing and validating the ronkit marketplace itself | `/plugin install test@ronkit` |
| `documents` | 0.1.0 | Skills for managing project documentation | `/plugin install documents@ronkit` |

### test

- **`hello`** — smoke-test skill. After installing, trigger it with `/test:hello` or by asking to "test the ronkit marketplace" — it should reply with a fixed confirmation string.

### documents

- **`resume-later`** — keeps a project's living planning doc current across sessions, so clearing context between milestones never loses state. Trigger it either way:
  - **Say it**: mention you're wrapping up, pausing, or about to clear context. Claude may also raise it proactively when it notices a milestone just finished or the session's gotten long.
  - **Run it**: `/documents:resume-later` (type `/resume` and autocomplete will get you there)

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
