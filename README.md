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
| `documents` | 0.8.1 | Skills for managing project documentation | `/plugin install documents@ronkit` |
| `security` | 0.2.0 | Skills for information-risk security review | `/plugin install security@ronkit` |

### test

- **`hello`** — smoke-test skill. After installing, trigger it with `/test:hello` or by asking to "test the ronkit marketplace" — it should reply with a fixed confirmation string.

### documents

- **`resume-later`** — keeps a project's living planning doc current across sessions, so clearing context between milestones never loses state. Trigger it either way:
  - **Say it**: mention you're wrapping up, pausing, or about to clear context. Claude may also raise it proactively when it notices a milestone just finished or the session's gotten long.
  - **Run it**: `/documents:resume-later` (type `/resume` and autocomplete will get you there)

  It finds your project's planning doc (asking if ambiguous), folds in anything undocumented (discoveries, decisions, milestone status), appends a concise History Log entry with a "what's next" note, flags any uncommitted git work, and asks for confirmation before it's safe to `/clear`.

- **`scaffold-project`** — sets up just the foundations: a private working-docs directory (full-context `CLAUDE.md`, `STATE.md`, `JOURNAL.md`, milestones, ritual prompts). This is a setup stage only — no code, and no public code repo, gets created here. Trigger it: `/documents:scaffold-project`, or ask to "start/kickstart/bootstrap a new project". Opens with a short interview (project name, purpose, working-docs location) before generating anything — it deliberately does not ask about tech stack or a public repo path, and screens answers for PII/sensitive details.

- **`plan-project`** — turns a scaffolded working-docs directory into an implementable plan through structured conversation: elaborates the idea, interrogates data sources and constraints, decides the tech stack, builds a risk register, and defines milestones as standalone deliverables with decision gates and a security review where applicable. The public code repo also gets created here, as one of the earliest milestones, once stack and scope are actually settled — not upfront at scaffold time. That milestone also asks whether the repo should get an [AGENTS.md](https://agents.md/) (cross-tool instructions readable by Cursor, Copilot, Gemini CLI, etc., not just Claude Code) — if yes, `CLAUDE.md` becomes a thin `@AGENTS.md` import plus Claude-specific rules, per Claude Code's own documented interop pattern. Each milestone closes only after a dedicated verification pass and the user's explicit sign-off against a plain-language summary — never on Claude's say-so alone. Trigger it: `/documents:plan-project`, or ask to plan, elaborate, or pressure-test a project — typically run right after `scaffold-project`.

- **`review-ai`** — reviews a technical document (system design, architecture, data schema, API spec — not a PRD) for AI usage, and if found, checks it against AI governance categories (data flow, vendor risk, security/guardrails, incident response, evaluation, plus escalation items like disclosure and regulatory mapping), producing a prioritized report for an engineering lead. Trigger it: `/documents:review-ai`, or ask to review a doc for AI governance or check for governance gaps. Read-only — never edits the source doc, gives a short summary in chat, and offers to write the full report to a file next to the doc reviewed.

### security

- **`audit-pii`** — audits a repo for information risk before it goes public: exposed secrets/credentials, PII, sensitive operational details (hostnames, internal repo references), and conversational comments that leak process or names. Produces a severity-ranked report and prioritized, phased remediation (secrets/PII first, then operational refs, then comment cleanup) — never edits or commits without explicit approval. Trigger it: `/security:audit-pii`, or ask to check a repo for secrets, PII, or sensitive leaks before pushing or making it public.

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
