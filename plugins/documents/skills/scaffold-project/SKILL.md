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

## The scrub gate

Copied verbatim into every scaffolded project's private CLAUDE.md (see
`references/templates.md` §1) — this is the canonical version, edit here,
not per-project.

Anything WRITTEN into the public repo — code, comments, commit messages,
docs, config — passes this before it's committed: no host names or paths,
no network topology, no exposure/tunnel/monitoring specifics, no PII, no
reference to the private repo's existence, name, or file structure.

This is not just about secrets. Two rules that are easy to violate by
accident, without ever touching a credential:
1. **No conversational name-attribution.** Never write "{{OwnerName}}
   decided/asked/confirmed/chose..." in a public-repo comment or commit
   message. The owner's name in project metadata (git author,
   `pyproject.toml`/`package.json` author field) is fine and expected —
   that's already structurally public. Narrating it through source comments
   as a decision-log narrator is not. Attribute decisions to "the project"
   or state them as settled fact instead.
2. **No naming the private repo's files.** Never write "see JOURNAL", "see
   STATE.md", "work-docs milestone doc", `ops/runbook.md`, or any other
   pointer to the private repo's file/directory names from the public repo.
   Same category as "no reference to the private repo's existence" above,
   just easy to miss because it doesn't look like a secret.

**Mechanical check — run before every commit to the public repo, not just
when something "looks sensitive":**

```
cd <public-repo> && git add -A && git grep -niE \
  "<hostwords>|<owner-first-name>|<owner-username>|ssh |tailscale|cloudflare tunnel|/Users/|/home/|password|api[_-]?key[[:space:]]*[:=]|token[[:space:]]*[:=]|@gmail|healthchecks\.io|journal|state\.md|work-docs|ops/runbook"
gitleaks protect --staged --redact -v   # or: gitleaks detect --source . -v
```

Fill `<hostwords>` with anything host-specific this project ever names
(e.g. `homeserver`, real IP ranges) and `<owner-first-name>`/
`<owner-username>` once known — do this the first time the public repo is
created, don't leave the placeholders in. `git add -A` first (or
`--untracked`) — plain `git grep` silently skips untracked files, so a
brand-new file can pass this check by never being looked at.

Grep hits are not automatically bad (an env var *name* like
`SOME_API_KEY` is fine) — read each one and confirm it's a name/pattern,
not a value or a real host detail, before treating the check as passed.

**This pattern list is not exhaustive by construction.** Don't treat "the
grep passed" as proof nothing's wrong — it only catches what's in the
list. Periodically re-audit (read actual file contents, not just grep) for
categories the fixed pattern list doesn't cover, especially after a long
run of sessions where comment style may have drifted.

## Comment scope

Also copied verbatim into every scaffolded project's private CLAUDE.md
(see `references/templates.md`) — canonical version, edit here.

Code comments explain non-obvious **why** (a hidden constraint, a subtle
invariant, a workaround for a specific bug) — not a running decision log.
Default to no comment. Never write comments that narrate the session:
no timestamps, no "changed from X to Y because...", no per-change
justification trail, no restating what the diff already shows. That
belongs in the commit message or the private JOURNAL, not in source.
This applies project-wide, not to any one file or module — a single
verbose file is evidence the rule has drifted, not an exception to scope
it to.

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
