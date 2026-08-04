# templates.md — file templates for the public code repo + bridge

Used by the "Repo & stack" milestone (see SKILL.md) — NOT at scaffold time.
By the point these are generated, the interview has already produced:
`{{name}}` project slug, `{{purpose}}` one-liner (scrubbed for public),
`{{public-path}}` public repo dir, `{{docs-path}}` working-docs dir,
`{{stack}}` chosen language/framework, `{{date}}` today, and
`{{wants-agents-md}}` (yes/no, asked in the same milestone interview — see
SKILL.md §0). Adapt bracketed guidance notes and delete them from output;
keep structure and headings.

Table of contents:
1. Public repo tree & .gitignore
2. AGENTS.md (only if `{{wants-agents-md}}` is yes)
3. Public CLAUDE.md
4. CLAUDE.local.md (bridge)

---

## 1. Public repo tree & .gitignore

```
{{public-path}}/
├── CLAUDE.md            CLAUDE.local.md (gitignored)
├── .gitignore           README.md
├── src|service/  web/  deploy/        [adapt to {{stack}}]
└── .github/workflows/
```

.gitignore minimum:
```
.env
.env.*
CLAUDE.local.md
docker-compose.override.yml
data/
*.db
.venv/
__pycache__/
node_modules/
.DS_Store
```

## 2. AGENTS.md (only if `{{wants-agents-md}}` is yes — sanitized, same scrub gate as CLAUDE.md)

Tool-agnostic file per the https://agents.md/ convention: any coding agent
(Claude Code, Cursor, Copilot, Gemini CLI, etc.) reads this, not just Claude.
Everything Claude-specific (behavioral rules, the CLAUDE.local.md bridge)
stays out of this file and lives in CLAUDE.md instead — see §3.

```markdown
# AGENTS.md — {{name}}

> Instructions for AI coding agents working in this repository.

## Project overview
**{{name}}** — {{purpose}}
[2-4 sentences: what it does, design priorities. Host-neutral.]

## Repository layout
[One line per top-level dir.]

## Commands
- **Install:** [e.g. `npm install`]
- **Run dev:** [e.g. `npm run dev`]
- **Test:** [e.g. `npm test`]
- **Lint:** [e.g. `npm run lint`]
- **Typecheck:** [if applicable]
- **Build:** [e.g. `npm run build`]

## Verified facts
[Facts about external APIs/data sources a session must not rediscover.
Only facts that are public or derivable from public sources.]

## Settled technical decisions (do not re-litigate silently — flag first)
| Decision | Choice | Revisit if |
|---|---|---|
[Technical decisions visible in code anyway, incl. {{stack}}. NOT: hosting,
exposure, monitoring routing — those live privately.]

## Security invariants (standing rules — a violation is never a refactor)
[Numbered, host-neutral. Include at minimum: secrets via environment only;
gitleaks pre-commit + CI; host-specific values from env or gitignored
overrides, never hardcoded.]

## Conventions
[Stack, style, testing. Host-neutral.]

## Verification gate
A change is only done when: tests pass, lint passes, typecheck passes (if
configured), and [any project-specific manual check].

## Rules for any AI agent working here
1. Passing the verification gate above means a change is technically sound —
   it does not by itself mean a milestone or feature is complete. Do not mark
   milestone-level work done without the project owner's explicit sign-off
   against a plain-language summary of what shipped.
2. Before implementing any non-trivial task, raise at least one risk, gap, or
   alternative; if genuinely fine as proposed, say so in one sentence.
3. Never silently override a settled decision in this file — flag it and wait
   for a response instead of re-litigating unprompted.
```

## 3. Public CLAUDE.md (sanitized — passes the scrub gate by construction)

If `{{wants-agents-md}}` is yes, CLAUDE.md is a thin wrapper per Claude
Code's documented AGENTS.md interop (`@AGENTS.md` import + Claude-specific
additions below it) — do NOT duplicate AGENTS.md's content here.

```markdown
# CLAUDE.md — {{name}}

@AGENTS.md

## Claude Code

Additional project context may be provided via `CLAUDE.local.md`
(gitignored). If present, read it first and follow its instructions before
doing anything else in this repo.
```

AGENTS.md's "Rules for any AI agent working here" (see §2) already covers the
risk-raising and settled-decision rules — don't repeat them here; this
section is for the one thing genuinely Claude-specific, the `CLAUDE.local.md`
bridge (Claude Code's documented import mechanism, not a generic convention
other agents would recognize).

If `{{wants-agents-md}}` is no, generate the original standalone form instead
(no `@AGENTS.md` import — everything inline):

```markdown
# CLAUDE.md — {{name}}

> Instructions for working in this repository. Read fully before changing code.

## What this project is
**{{name}}** — {{purpose}}
[2-4 sentences: what it does, design priorities. Host-neutral.]

## Repository layout
[One line per top-level dir.]

## Verified facts
[Facts about external APIs/data sources a session must not rediscover.
Only facts that are public or derivable from public sources.]

## Settled technical decisions (do not re-litigate silently — flag first)
| Decision | Choice | Revisit if |
|---|---|---|
[Technical decisions visible in code anyway, incl. {{stack}}. NOT: hosting,
exposure, monitoring routing — those live privately.]

## Security invariants (standing rules — a violation is never a refactor)
[Numbered, host-neutral. Include at minimum: secrets via environment only;
gitleaks pre-commit + CI; host-specific values from env or gitignored
overrides, never hardcoded.]

## Conventions
[Stack, style, testing. Host-neutral.]

## Behavioural rules for Claude in this repo
1. Before implementing any task, raise at least one risk, gap, or
   alternative; if genuinely fine, one sentence why.
2. Never silently undo a settled decision above — flag and wait.
3. Check every change against the security invariants.
4. Additional project context may be provided via `CLAUDE.local.md`
   (gitignored). If present, read it first and follow its instructions.
```

## 4. CLAUDE.local.md (bridge — NEVER commit; must be in .gitignore)

```markdown
# CLAUDE.local.md — NEVER COMMIT (gitignored)

The working docs for this project live in `{{relative-path-to-work-docs}}`
(private repo). That repo is the source of truth for planning and rituals.

Before doing anything in this session:
1. Read `<work-docs>/CLAUDE.md` (full context — extends and overrides this
   repo's public CLAUDE.md where they differ).
2. Read `<work-docs>/STATE.md`, the current milestone file, and the last two
   `JOURNAL.md` entries.
3. Follow the session rituals in `<work-docs>/prompts/`.

Hard rule: anything WRITTEN into this public repo passes the scrub gate in the
private CLAUDE.md — no host names/paths, no topology, no exposure/monitoring
specifics, no PII, no reference to the private repo's existence.
```
