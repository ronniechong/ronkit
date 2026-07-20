---
name: scaffold-project
description: Scaffold the foundational files for a new project using the two-repo pattern — a PUBLIC code-only project repo plus a PRIVATE working-docs repo (full-context CLAUDE.md, STATE.md, JOURNAL.md, milestones/, ritual prompts, ops/), bridged by a gitignored CLAUDE.local.md. Use whenever the user wants to start, kickstart, bootstrap, or scaffold a new project or repo, set up project structure, "create the repos", or add this working-docs structure to an existing project. Always begins by asking a short set of setup questions (paths, project purpose, stack) before generating anything. For elaborating, planning, and populating the scaffolded files with real content, hand off to the companion skill /documents:plan-project.
---

# scaffold

Generate the foundational two-repo structure for a project. This skill creates
the skeleton; `/documents:plan-project` fills it with real content.

## The pattern

- **Public project repo** — code-only history, safe for the world. Sanitized
  `CLAUDE.md` with repo-level instructions.
- **Private working-docs repo/directory** — full context, living documents
  (STATE, JOURNAL, milestones), ritual prompts, `ops/` for host specifics.
- **Bridge** — `CLAUDE.local.md` in the public repo, gitignored, pointing to
  the private docs.

**Core rule: sensitivity flows one direction.** Private may reference public.
Public must NEVER reference the private repo — not its name, path, or
existence. `CLAUDE.local.md` is the only file that knows both, and it is
gitignored.

## Step 1 — Interview (always, before generating anything)

Ask these questions in one short batch. Do not guess paths. If the user
answered some already in the conversation, only ask the gaps.

1. **Project name** (slug, lowercase kebab-case)
2. **What is the project about?** (one or two sentences — becomes the purpose
   line; anything sensitive stays out of the public files)
3. **Public repo directory** (absolute path)
4. **Working-docs directory** (absolute path; suggest a sibling layout such as
   `<parent>/work-docs/<name>` if the user has no preference)
5. **Stack/language** (shapes the skeleton dirs and .gitignore)
6. **Public from day one?** (default yes — recommend gitleaks + hygiene in the
   first commit either way)

Optional follow-ups only if relevant: learning goal (enables the teaching-mode
rule), whether milestones are already known.

## Step 2 — Privacy screen before writing

The public repo must contain no PII and no sensitive information. Before
generating, screen the user's answers: if the purpose or any input mentions
personal details, employers, health/financial context, host names, home
infrastructure, or anything the user wouldn't publish, keep it OUT of the
public files — rewrite the public purpose line neutrally and put the full
version in the private CLAUDE.md instead. When in doubt, ask: "this detail —
public or private?"

## Step 3 — Generate both trees

Read `references/templates.md` and generate every file, filling placeholders
from the interview. Public tree: CLAUDE.md (sanitized), CLAUDE.local.md
(bridge, gitignored), .gitignore, README stub, skeleton dirs. Private tree:
CLAUDE.md (full context + two-repo section + scrub gate), STATE.md, JOURNAL.md
(Session 1 entry), milestones/ (template + any known milestones as stubs),
prompts/spec-review.md (three rituals), ops/ starters.

In Claude Code, write directly to the two paths given. If either directory is
not accessible, generate the trees in a staging directory and tell the user
exactly where to move them.

## Step 4 — Verify (leak scan)

Before declaring done, scan the PUBLIC tree for: the working-docs path, the
private repo name, any host names or personal details mentioned during the
interview. Only `CLAUDE.local.md` may reference the private repo — and confirm
it is listed in `.gitignore`. Report the scan result to the user.

## Step 5 — Hand off

Tell the user the init order (git init both, hygiene files in the public
repo's first commit) and that `/documents:plan-project` is the companion skill
for turning the skeleton into an implementable plan — offer to start it.
