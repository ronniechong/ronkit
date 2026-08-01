---
name: plan-project
description: Turn a scaffolded project into an implementable plan through structured conversation — elaborate the idea, interrogate data sources and constraints, build a risk register, define milestones with pre-committed decision gates, and populate the working-docs files (CLAUDE.md decisions table, STATE.md, JOURNAL.md, milestone specs) as decisions land. Use when the user wants to plan, elaborate, validate, or pressure-test a project; asks "what are the risks/unknowns/oversights", "let's continue planning", "help me think this through", or wants milestone docs drafted or the working docs populated; or right after /documents:scaffold-project has run. Works on any project using the two-repo working-docs structure (a CLAUDE.local.md pointing at a work-docs directory is the tell), and can adopt projects that don't have it yet by suggesting /documents:scaffold-project first.
---

# plan

A structured planning conversation that ends with populated working docs and a
reviewable first milestone — not just talk. The scaffold gives empty vessels;
this skill fills them, decision by decision, **writing as it goes**.

## Ground rules for the conversation

- **Write during the session, not at the end.** When a decision lands, record
  it in the relevant file (CLAUDE.md decisions table, milestone doc, JOURNAL)
  in the same exchange. If the session ended abruptly, nothing should be lost.
- **Rationale is the point.** Every decision is recorded as "chose X over Y
  because Z, revisit if W" — never a bare choice.
- **Disagree with reasons; agree briefly.** Raise at least one risk, gap, or
  alternative per proposal (critique gate). No performative nitpicking; no
  uncritical agreement.
- **Pre-commit thresholds.** When a decision depends on unknown data, don't
  debate it — define a measurable gate with a threshold committed BEFORE the
  data exists, and a spike/metric to resolve it. Near-threshold results get an
  explicit decision, never a fudge.
- **Scrub gate.** Anything destined for the public repo is host-neutral, free
  of PII and sensitive detail. Full versions live in the private docs.

## The planning arc (adapt order to the user; skip what's already done)

### 1. Elaborate
Interview until the idea has edges: what it does, for whom, what "done" means,
what the user actually wants to learn or showcase (the real product may be the
skill acquired — record that motivation privately). Capture explicit
priorities (e.g. security, API politeness, cost) as ranked constraints.

### 2. Interrogate reality
Identify every external dependency — APIs, data sources, platforms — and for
each, list what is UNKNOWN: rate limits, update cadence, field availability,
auth quirks, licensing/attribution. Anything that could reshape the design
becomes a **spike**: a small time-boxed investigation with named questions,
each tied to the decision it unlocks. Draft the spike as milestone 01 with a
findings-report structure (measured numbers → decision matrix → planning
resolution) so results mechanically update the plan.

If there are 3+ external dependencies to research, spawn one Explore/
general-purpose agent per dependency in parallel (each digging into that
dependency's rate limits, auth, licensing) rather than researching them one
at a time — fold the returned findings into the UNKNOWN list yourself. For
1-2 dependencies, a single direct lookup is faster than the spawn overhead.

### 3. Risk register
Sweep for risks in layers: data/dependency unknowns, platform/hosting,
security exposure, legal/licensing, scope traps, and the human factor
(motivation, competing projects). Rank by damage. Every risk gets a
mitigation, an owner milestone, or an explicit "accepted" with reason.
Rerun a lighter sweep ("any oversights?") after major decisions land.

For the first full sweep, the layers are independent lenses on the same
project — consider running 2-4 of them (e.g. security exposure, legal/
licensing, platform/hosting) as parallel agents against the elaborated idea
and interrogation findings, then merge results into one ranked register
yourself. Skip this for the lighter re-sweeps after decisions land; those
are quick enough to do directly.

### 4. Milestones
Slice into milestones that are each a **standalone deliverable** — something
that works and is worth having on its own, not an internal checkpoint that
only makes sense once a later milestone exists. If a candidate slice can't be
described as "at the end of this, we have X and could stop here," it's not a
milestone boundary yet — merge it into its neighbour or resequence.

Each milestone spec has: goal, verifiable acceptance criteria (including
soak/unattended requirements where relevant), decision-gate tables, a
**security review item** whenever the milestone adds/changes external
surface, secrets, auth, or user input (state it explicitly even when N/A, with
a one-line reason — never a silent omission), explicit out-of-scope list, an
empty "Risks raised at critique gate" section, and a **Sign-off** checklist
that must be completed before Status can move to DONE (acceptance criteria
verified against real behaviour, decision gates resolved with real data,
security review complete or explicitly N/A, no open uncertainties). A
milestone is not closed by checking off tasks — it closes when it's been
verified and validated against its own acceptance criteria, AND the user has
reviewed a plain-language **milestone summary** (what shipped, decisions
made, deviations, where to look) and actually said so. Claude checks its own
boxes; only the user checks the last one.

Each milestone spec is reviewed in a DEDICATED spec-review session (see the
project's prompts/spec-review.md) before implementation — offer to run it.
Closing a milestone gets its own dedicated pass too (the sign-off ritual in
the same file): write the summary, present it, and wait for the user's
explicit sign-off before touching Status — don't fold this into a routine
session-end and don't mark DONE on Claude's own say-so.

**While slicing, don't guess past ambiguity.** If a milestone boundary,
dependency, scope call, or "is this security-relevant" judgment is genuinely
unclear, stop and ask the user before locking it in — flag it as a question,
not a footnote.

### 5. Populate and close
By the end of planning: CLAUDE.md carries the settled-decisions table,
security invariants, and verified external facts; STATE.md points at the true
next action; JOURNAL.md has the session's entry (decisions + rationale +
surprises + uncertainties that are related to the project. Don't record any 
unrelated notes or observations); milestone 01 or 02 is drafted and awaiting review.
Keep the JOURNAL.md entry short and to the point as this will grow.
Read the project's session-end ritual and follow it.

## Behaviours that make this skill worth invoking

- Convert vague preferences into recorded, revisitable decisions.
- Name the trade-off the user is implicitly making and check they mean it.
- When the user defers a decision, record BOTH options and the criteria that
  will decide, at the milestone where it must be resolved.
- When findings arrive (spike reports, measurements), resolve gates against
  pre-committed thresholds, amend milestone scope, and update the files —
  don't just discuss.
- If the project has a stated learning goal, name the concept each piece of
  work teaches as it comes up (teaching mode).
- Periodically ask: "is this in the doc, or just in the chat?" — anything
  worth keeping gets written.
