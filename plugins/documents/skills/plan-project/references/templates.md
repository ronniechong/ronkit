# templates.md — file templates for the public code repo + bridge

Used by the "Repo & stack" milestone (see SKILL.md) — NOT at scaffold time.
By the point these are generated, the interview has already produced:
`{{name}}` project slug, `{{purpose}}` one-liner (scrubbed for public),
`{{public-path}}` public repo dir, `{{docs-path}}` working-docs dir,
`{{stack}}` chosen language/framework, `{{date}}` today. Adapt bracketed
guidance notes and delete them from output; keep structure and headings.

Table of contents:
1. Public repo tree & .gitignore
2. Public CLAUDE.md
3. CLAUDE.local.md (bridge)

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

## 2. Public CLAUDE.md (sanitized — passes the scrub gate by construction)

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

## 3. CLAUDE.local.md (bridge — NEVER commit; must be in .gitignore)

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
