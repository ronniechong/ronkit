# templates.md — file templates for the two-repo structure

Placeholders: `{{name}}` project slug, `{{purpose}}` one-liner, `{{public-path}}` public repo dir, `{{docs-path}}` working-docs dir (both from the interview — never assumed), `{{date}}` today. Adapt bracketed guidance notes and
delete them from output; keep structure and headings.

Table of contents:
1. Public repo tree & .gitignore
2. Public CLAUDE.md
3. CLAUDE.local.md (bridge)
4. Private CLAUDE.md
5. STATE.md
6. JOURNAL.md
7. Milestone template (with decision-gate table)
8. prompts/spec-review.md (four rituals)
9. ops/ starters

---

## 1. Public repo tree & .gitignore

```
{{public-path}}/
├── CLAUDE.md            CLAUDE.local.md (gitignored)
├── .gitignore           README.md
├── src|service/  web/  deploy/        [adapt to stack]
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
[Technical decisions visible in code anyway. NOT: hosting, exposure,
monitoring routing — those live privately.]

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

## 4. Private CLAUDE.md (full context)

```markdown
# CLAUDE.md — {{name}} working docs (PRIVATE)

> Read this first, every session. Conflicts with reality → STOP and flag.

## What this project is
[Full version, including the real learning/portfolio motivation.]

## Two-repo structure (settled)
- Public: `{{public-path}}` — code-only history; must NEVER reference this
  repo. Its CLAUDE.md is the sanitized tier.
- Private (this repo): `{{docs-path}}` — full context, rituals,
  `ops/` (host specifics, env templates, exposure/monitoring/backup notes).
- Bridge: gitignored `CLAUDE.local.md` in the public repo points here.
  Sensitivity flows ONE direction: private→public references allowed only.

## Scrub gate (standing rule)
[Copy verbatim from SKILL.md "The scrub gate" section.]

## Full context
[Everything the public CLAUDE.md omits: hosting, exposure decision + options,
monitoring/alert routing, backup strategy, plus the complete decisions table
with rationale ("chose X over Y because Z, revisit if W") and a
"Things we tried and rejected" section.]

## Standing behavioural rules for Claude
1. Critique gate  2. Session-start ritual  3. Session-end ritual (incl. scrub
confirmation for public files touched)  4. Settled decisions — flag, don't
undo  5. Disagreement calibration — reasons over verdicts, no performative
nitpicking  6. Security/citizenship check on every spec and change
[7. Optional: teaching mode — name the concept being practised, if the
project has a stated learning goal.]
```

## 5. STATE.md

```markdown
# STATE.md — overwrite every session, keep under 15 lines

**As of:** {{date}}, project kickstarted

**What is true right now**
- Repos initialised at {{public-path}} (public) and
  {{docs-path}} (private). No production code exists.
- Milestone 01 spec drafted — NOT yet reviewed.

**In flight**
- Nothing.

**Very next action**
- Run prompts/spec-review.md against milestones/01-*.md in a dedicated
  session. No implementation before that review.
```

## 6. JOURNAL.md

```markdown
# JOURNAL.md — append-only. Newest at the bottom. Never edit old entries.

Entry format:
- **Done:** finished (not attempted — finished)
- **Decisions:** what and WHY (rationale is the point)
- **Half-finished:** in flight, with file/branch pointers
- **Surprises:** anything that didn't behave as expected
- **Next:** the literal next command or step
- **Uncertain:** things Claude flagged as unsure (review these!)

---

## Session 1 — {{date}} (kickstart)
[First entry: what was scaffolded, initial decisions + rationale, Next per
STATE.md, any Uncertain items from the kickstart conversation.]
```

## 7. Milestone template — milestones/NN-slug.md

```markdown
# Milestone NN — Title

**Status:** NOT STARTED — awaiting spec-review session
**Depends on:** …   **Blocks:** …

## Goal
[2-3 sentences: what done looks like and why it matters. Must stand alone as
a deliverable — "at the end of this, we have X and could stop here" — not an
internal step that only makes sense once a later milestone lands.]

## Acceptance criteria (definition of done)
- [ ] [Verifiable, includes any soak/unattended-run requirements]

## Decision gates (thresholds pre-committed — resolve with data)
| Gate | Metric | Threshold | Resolution path |
|---|---|---|---|
[Commit thresholds BEFORE seeing data. Near-misses decided explicitly.]

## Security review
[Required whenever this milestone adds/changes external-facing surface,
secrets, auth, or user input — list what was checked (new attack surface,
secret handling, input validation, dependency risk). If none of that applies,
say so explicitly: "Not applicable — no new surface/secrets/auth/input in
this milestone." Never leave this section silently blank.]

## Tasks
- [ ] …    [markers: [ ] todo, [x] done, [~] in progress, [!] blocked]

## Explicitly out of scope (deferred)
- … → M(NN+1)

## Risks raised at critique gate
_(filled by the dedicated spec-review session — run prompts/spec-review.md
against this file before any implementation)_

## Milestone summary (for user review — fill in at sign-off time)
[Plain-language recap of what was actually delivered, written for the user to
skim and check against reality — not a copy of the task list. Include: what
shipped, key decisions made or changed along the way, any deviations from the
original spec and why, and where to look (files/commits/URLs) to verify it
directly.]
- [ ] …

## Sign-off (required before Status: DONE)
- [ ] Acceptance criteria verified against actual behaviour, not assumed
- [ ] Decision gates resolved with real data (near-misses decided explicitly)
- [ ] Security review complete, or explicitly marked N/A with reason
- [ ] No open uncertainties, or each has an owner and a resolution path
- [ ] User has reviewed the Milestone summary above and approved closing
_(filled by the dedicated milestone sign-off session — run
prompts/spec-review.md's sign-off ritual against this file before closing.
The last box is the user's call, not Claude's — Status only moves to DONE
after the user has actually said so.)_
```

## 8. prompts/spec-review.md — the four rituals

```markdown
# Spec Review Prompt (its OWN session — no implementation allowed)

Read CLAUDE.md, STATE.md, and `milestones/NN-<name>.md`.
Your ONLY job is to find problems with this spec. Do not implement. Check:
1. Ambiguities — two reasonable engineers would read it differently.
2. Missing edge cases — inputs, failure modes, states ignored.
3. Sequencing risks — late discovery forces rework.
4. Over-scoping — belongs in a later milestone.
5. Conflicts — contradicts settled decisions in CLAUDE.md.
6. Steelman one alternative — strongest different approach; does it hold?
7. Security/citizenship — new path to any upstream API, wider public
   surface, secrets touched, or a security invariant weakened? AI paths too.
Rank findings by severity. Zero findings across seven categories is
suspicious; look harder. Finish by rewriting the milestone's "Risks raised
at critique gate" section.

---

# Milestone Sign-off Prompt (its OWN session — run before Status: DONE)

Read CLAUDE.md, STATE.md, and `milestones/NN-<name>.md`. Your job is to
verify and validate this milestone is actually finished, not just to check
that tasks are ticked. Check:
1. Acceptance criteria — walk each one against real, observed behaviour
   (run it, read the output, don't take a task checkbox's word for it).
2. Decision gates — each resolved with real data against its pre-committed
   threshold; any near-miss has an explicit decision recorded, not a fudge.
3. Security review — complete and matches what actually shipped, or
   explicitly N/A with a reason that still holds.
4. Deliverable check — does this milestone stand on its own (could ship or
   demo as-is), or did scope quietly leak into "finish next milestone first"?
5. Uncertainties — anything flagged during the milestone is resolved, or
   carries an owner and a resolution path in a later milestone.
6. Regressions — spot-check that this milestone didn't silently break an
   earlier milestone's acceptance criteria.
Write the "Milestone summary" section — plain language, for the user to
review, not a restated task list. Then present that summary to the user and
ask them directly to review and sign off; do not check that final Sign-off
box or move Status to DONE yourself. Fill in the rest of the "Sign-off"
checklist with the result of each check above. Only mark **Status: DONE**
once every box is genuinely checked AND the user has confirmed the summary —
an unresolved item, or a summary the user hasn't actually seen, means the
milestone stays open, however close it looks.

---

# Resume Prompt (start of every working session)

Read CLAUDE.md, STATE.md, the current milestone file, and the last two
JOURNAL.md entries. Summarise where we are in 5 lines and confirm the next
step. Do not touch code until the summary is confirmed correct.

---

# Session-End Prompt (end of every working session)

Overwrite STATE.md (max 15 lines), append a JOURNAL.md entry in the standard
format, update milestone task markers. List anything uncertain under
"Uncertain", and list any public-repo files touched with confirmation each
passed the scrub gate. Show the diffs.
```

## 9. ops/ starters (private repo)

ops/README.md:
```markdown
# ops/ — host-specific material (PRIVATE, never mirrored publicly)

Belongs here: host paths and mounts, exposure/tunnel config, firewall/egress
rules, alert routing (webhook URLs), external monitor setup, backup
destinations + restore steps, .env templates with real variable names,
docker-compose.override.yml template. If it names a host, path, credentialed
URL, or topology — it lives here.
```

ops/env.template: one commented line per required variable, values empty,
with a header noting where to copy it (public repo's gitignored `.env`).
