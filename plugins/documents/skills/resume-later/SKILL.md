---
name: resume-later
description: Use before ending a work session on a multi-milestone project — when the user says they're about to clear context, pause, wrap up, stop for now, or continue "tomorrow/next week/later" (or explicitly runs /documents:resume-later), OR proactively when a milestone just completed or the session has grown long and a natural stopping point is near. Brings the project's living planning doc up to date and gets explicit user confirmation before it's safe to /clear. Not for routine mid-task status updates, and not a substitute for committing code.
---

# resume-later

Multi-milestone projects lose continuity when context is cleared mid-work. This skill makes sure nothing discovered or decided in the session is lost, and that resuming later doesn't require re-deriving anything from a wiped transcript.

Run through this checklist every time, in order. Do not tell the user it is safe to clear until all steps are complete.

## 1. Identify the living project doc

Look for the project's planning/tracking doc. Common patterns, in rough order of likelihood:
- A dedicated planning-doc directory alongside the project's other repos (e.g. `work-docs/<project>.md`)
- `PROJECT.md`, `PLAN.md`, or `STATUS.md` at the repo root
- A `docs/` directory with a status or roadmap file

If more than one candidate exists, or none is obviously the right one, **ask the user which doc to update** — do not guess, and do not create a new one without asking.

## 2. Review the session for undocumented state

Scan back through the session for anything not yet reflected in the doc:
- New discoveries (API quirks, constraints, things that turned out to be wrong)
- Decisions made (design choices, tradeoffs, things ruled out and why)
- Milestone status changes (started, blocked, completed)
- Any TBD/placeholder items that got resolved

## 3. Update the doc

- Update the status line / current milestone marker.
- Append one concise History Log entry: timestamp + what shipped/changed + what's next. Keep it short — a status update, not a narrative walkthrough of debugging steps or dead ends. The "what's next" clause is what makes the entry resumable; write it assuming the reader has zero memory of this session.
- Add Decision Log entries for any new design decisions, if the doc has that section.
- Do not touch sections unrelated to this session's work.

## 4. Check for uncommitted work

Run `git status` in the relevant repo(s). If there's meaningful uncommitted work that the doc update references or depends on, flag it to the user — clearing context doesn't touch the filesystem, but resuming later is harder if the working tree doesn't match what the doc says happened. Don't auto-commit; just surface it.

## 5. Confirm with the user

Summarize exactly what was changed (doc path, sections touched, one-line gist of the History Log entry). Then explicitly ask: is this accurate, and is it safe to clear context now? Wait for confirmation before treating the session as wrapped up.
