# ronkit

Ronnie's Claude Code plugin marketplace — skills, slash commands, agents, and hooks, installable in two commands.

Plugins here are organized by **category**, not one-plugin-per-skill: each plugin is a scope (`documents`, `test`, ...) that groups related skills. This keeps slash invocation (`/plugin-name:skill-name` — plugin skills are always namespaced, even with zero name collisions) meaningful instead of repeating the same word twice.

## Installation

```bash
/plugin marketplace add ronniechong/ronkit
/plugin install <plugin-name>@ronkit
```

## Plugins

| Plugin | Version | Description | Install |
|---|---|---|---|
| `test` | 0.1.1 | Skills for testing and validating the ronkit marketplace itself | `/plugin install test@ronkit` |
| `documents` | 0.3.0 | Skills for managing project documentation | `/plugin install documents@ronkit` |

### test

- **`hello`** — smoke-test skill. After installing, trigger it with `/test:hello` or by asking to "test the ronkit marketplace" — it should reply with a fixed confirmation string.

### documents

- **`resume-later`** — keeps a project's living planning doc current across sessions, so clearing context between milestones never loses state. Trigger it either way:
  - **Say it**: mention you're wrapping up, pausing, or about to clear context. Claude may also raise it proactively when it notices a milestone just finished or the session's gotten long.
  - **Run it**: `/documents:resume-later` (type `/resume` and autocomplete will get you there)

  It finds your project's planning doc (asking if ambiguous), folds in anything undocumented (discoveries, decisions, milestone status), appends a concise History Log entry with a "what's next" note, flags any uncommitted git work, and asks for confirmation before it's safe to `/clear`.

- **`scaffold-project`** — bootstraps the foundational files for a new project using a two-repo pattern: a public code-only repo plus a private working-docs repo (full-context `CLAUDE.md`, `STATE.md`, `JOURNAL.md`, milestones, ritual prompts), bridged by a gitignored `CLAUDE.local.md`. Trigger it: `/documents:scaffold-project`, or ask to "start/kickstart/bootstrap a new project". Always opens with a short setup interview (paths, purpose, stack) before generating anything, and screens answers for PII/sensitive details before writing the public files.

- **`plan-project`** — turns a scaffolded project into an implementable plan through structured conversation: elaborates the idea, interrogates data sources and constraints, builds a risk register, defines milestones with decision gates, and populates the working-docs files as decisions land. Trigger it: `/documents:plan-project`, or ask to plan, elaborate, or pressure-test a project — typically run right after `scaffold-project`.

- **`review-ai`** — reviews a technical document (system design, architecture, data schema, API spec — not a PRD) for AI usage, and if found, checks it against AI governance categories (data flow, vendor risk, security/guardrails, incident response, evaluation, plus escalation items like disclosure and regulatory mapping), producing a prioritized report for an engineering lead. Trigger it: `/documents:review-ai`, or ask to review a doc for AI governance or check for governance gaps. Read-only — never edits the source doc, gives a short summary in chat, and offers to write the full report to a file next to the doc reviewed.

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
