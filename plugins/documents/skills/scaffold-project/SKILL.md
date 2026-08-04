---
name: scaffold-project
description: Scaffold the foundations for a new project's PRIVATE working-docs directory (full-context CLAUDE.md, STATE.md, JOURNAL.md, milestones/, ritual prompts, ops/) — planning infrastructure only, no code and no code repo. Use whenever the user wants to start, kickstart, bootstrap, or scaffold a new project, set up project structure, or add this working-docs pattern to an existing project. Always begins by asking a short set of setup questions (project name, purpose, working-docs location) before generating anything — it deliberately does NOT ask about tech stack or create the code repo; those are planning-stage decisions. For elaborating, planning, deciding stack, creating the code repo, and populating these files with real content, hand off to the companion skill /documents:plan-project.
---

# scaffold

Generate the foundations for a project's private working-docs directory. This
is a setup stage only — no code, and no code repo, gets created here.
`/documents:plan-project` is where stack gets decided, the code repo gets
created (as an early milestone), and these files get filled with real content.

## The pattern (target end-state, not what this skill builds)

- **Public project repo** — code-only history, safe for the world. Sanitized
  `CLAUDE.md` with repo-level instructions. Created later, by
  `/documents:plan-project`, once stack and scope are actually decided.
- **Private working-docs repo/directory** — full context, living documents
  (STATE, JOURNAL, milestones), ritual prompts, `ops/` for host specifics.
  **This is what this skill creates.**
- **Bridge** — `CLAUDE.local.md` in the public repo, gitignored, pointing to
  the private docs. Doesn't exist until the public repo does — created
  alongside it during planning, not here.

**Core rule: sensitivity flows one direction.** Private may reference public.
Public must NEVER reference the private repo — not its name, path, or
existence. `CLAUDE.local.md` is the only file that knows both, and it is
gitignored. This skill has no public tree to enforce that on yet, but the
private files it writes (STATE.md's "very next action", JOURNAL's Session 1)
should say plainly that the code repo doesn't exist yet, so a later session
doesn't assume otherwise.

## Step 1 — Interview (always, before generating anything)

Ask these questions in one short batch, as a **plain chat message** — not
via a structured multi-choice tool. These are free text (names, a one-line
description, an absolute path) with no fixed set of options, which a
forced-choice tool can't represent and will reject outright. Do not guess
paths. If the user answered some already in the conversation, only ask the
gaps.

1. **Project name** (slug, lowercase kebab-case)
2. **What is the project about?** (one or two sentences — becomes the purpose
   line; anything sensitive stays out of files destined for the public repo
   later)
3. **Working-docs directory** (absolute path; suggest a sibling layout such as
   `<parent>/work-docs/<name>` if the user has no preference)

Do NOT ask about stack/language, the public code repo's path, or public-from-
day-one — all three are decisions that belong to `/documents:plan-project`
(stack shapes the code repo's skeleton, which doesn't exist yet). If the user
volunteers them anyway, note them in JOURNAL.md's Uncertain/Next section as
input for the planning session rather than acting on them now.

Optional follow-ups only if relevant: learning goal (enables the teaching-mode
rule), whether milestones are already known.

## Step 2 — Privacy screen before writing

Even though nothing public is created yet, the private working-docs files
will eventually seed public ones. Before generating, screen the user's
answers: if the purpose or any input mentions personal details, employers,
health/financial context, host names, home infrastructure, or anything the
user wouldn't eventually publish, note in the private CLAUDE.md that this
detail must NOT carry over when the public CLAUDE.md gets generated later.
When in doubt, ask: "this detail — public or private?"

## Step 3 — Generate the private tree only

Read `references/templates.md` and generate the private tree: CLAUDE.md (full
context + two-repo section, noting the code repo doesn't exist yet + scrub
gate), STATE.md (very next action = run `/documents:plan-project`), JOURNAL.md
(Session 1 entry), milestones/ (template only — the first real milestone gets
drafted during planning, not here), prompts/spec-review.md (four rituals),
ops/ starters.

Do not generate a public tree, `CLAUDE.local.md`, skeleton code dirs, or
`.gitignore` for a code repo — none of that exists until planning decides the
stack and creates the repo.

In Claude Code, write directly to the working-docs path given. If it is not
accessible, generate the tree in a staging directory and tell the user
exactly where to move it.

## Step 4 — Verify

Confirm the generated tree contains no code, no stack-specific files, and no
reference to a public repo path (there isn't one yet). Report this to the
user.

## Step 5 — Hand off

Tell the user this was setup only — no code repo exists yet — and that
`/documents:plan-project` is the companion skill that elaborates the idea,
decides the stack, creates the public code repo (as one of its earliest
milestones), and populates these files with real content. Offer to start it.
