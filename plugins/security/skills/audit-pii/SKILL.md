---
name: security-audit-pii
description: >-
  Audits a source repository for information risk — exposed personal data
  (PII), secrets and sensitive content, private-repo cross-references, and
  leaked conversational/narrative comments — produces a risk-categorized
  report, then gives prioritized, actionable remediation steps. Use when the
  user wants to check a repo (or codebase) before it is made public, before
  pushing, as part of a security/PR review, or asks to scan for secrets,
  PII exposed, sensitive info leaks, API keys, credentials, or accidental
  private/process details in comments or files. Scope is INFORMATION RISK,
  not code implementation or runtime security.
---

# Security Audit — Information Risk (PII, Secrets, Leaks)

Audit the *information* that a codebase exposes, not how the code behaves.
Goal: catch anything that shouldn't be public before it is pushed, published,
or shared, and give the owner a clear report plus concrete steps to fix it.

## Scope & non-goals

This skill is about **risk of exposure**, not implementation quality.

- **In scope:** secrets and credentials, personal/identifying data (PII),
  sensitive operational details, accidental references to private/internal
  systems or repos, conversational or narrative comments that leak context,
  and *inferred* leaks (stuff that on its own looks harmless but, combined,
  reveals identity, hostnames, IPs, names, routes, or process).
- **Explicitly NOT in scope:** code logic bugs, security bugs in code
  (injection, auth flaws), performance, style/style styles, or test coverage.
  If asked whether code is "correct", read that as a signal to clarify scope —
  otherwise stay on information risk.

## Operating principles

1. **No edits during the audit.** Audit first, fix second. Produce the report
   before touching any files.
2. **Report is ordered by severity/risk category**, each finding traceable to
   a file:line and one of the categories below.
3. **Actionable remediation** means "what to change, where, and roughly how" —
   delete, redact, rotate, move, or never commit. Never guess — if a value
   could be a live secret, tell the owner to rotate it, not just hide it.
4. **Anything that looks like a real secret gets flagged HIGH regardless of
   whether it's currently deployed.** Assume worst case for exposure.
5. Spell out findings in plain, lay terms. No niche jargon without a
   one-line explanation the first time you use it.

## Risk categories (use this table in reports)

| # | Category | Severity | Example |
|---|----------|----------|---------|
| 1 | **Hard secrets** | CRITICAL | API keys, password, tokens, private keys, connection strings, `AWS_*`/`AZURE_*`/`GOOGLE_*` values, `BEGIN ... PRIVATE KEY`, DB creds, OAuth secrets |
| 2 | **PII (personal/identifying data)** | HIGH | names, emails, phone numbers, home addresses, dates of birth, government/ID numbers, medical/financial details, IPs linked to a person, biometrics |
| 3 | **Sensitive operational info** | HIGH | hostnames, internal domain names, service endpoints/ports, private network ranges/VPN, internal repo names, deployment topology, runbooks that describe real production incidents, session/milestone codes, internal doc names (e.g. "see JOURNAL", "STATE.md", a private docs dir) |
| 4 | **Conversational / narrative comments** | MEDIUM | "John decided/asked/confirmed...", blow-by-blow "decided X, rejected Y, confirmed live on Z" rationale, dated/meeting-annotated decision journals in source comments |
| 5 | **Inferred / combinability leak** | MEDIUM→HIGH | Alone-harmless strings that, combined with git history, author metadata, or realistic context, reveal identity, people, internal structure, or private repos. Flag the *combination*, not each string. |

## How to run the audit

1. **Clarify the target.** Confirm which directory(ies) / repo to audit and
   whether it is slated to be public. Ask via a short prompt, don't assume.
2. **Audit only (read-only).** Walk the tree. Don't edit files yet.
3. **Grep as a floor, not a ceiling.** Start with targeted searches, then read
   actual file contents — narrative and inferred leaks don't always match a
   keyword.
4. **Scale check first.** If the target is large (rough guide: >500 files or
   >50MB tracked), ask whether to scope the audit to specific
   directories/paths before doing a full read-through — don't silently sample
   a big repo and call it complete, and don't grind through it file-by-file
   without checking in first.
5. **Skip non-text/binary files** (images, PDFs, compiled artifacts,
   `.pyc`/`.so`/etc., large data files) from the read-through — grep noise or
   chokes on these, and they're not where narrative/PII leaks live.

   Starting pattern set (adapt terms to the project):
   ```bash
   # secrets / creds
   grep -rniE "(api[_-]?key|secret|password|passwd|token|priv[ae]te[_-]?key|access[_-]?key|BEGIN (RSA|OPENSSH|EC|PGP) PRIVATE KEY)" --include="*" .
   # common cloud provider prefixes
   grep -rniE "(AKIA[A-Z0-9]{16}|sk-[A-Za-z0-9]{20,}|ghp_[A-Za-z0-9]{36}|xox[baprs]-|AIza[0-9A-Za-z_-]{35}|firebase|mongodb(\+srv)?://)" .
   # emails / PII-ish
   grep -rniE "[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}" --include=*
   # operational references
   grep -rniE "(JOURNAL|STATE\.md|work-docs|internal|intranet|\.internal|10\.|192\.168\.|172\.(1[6-9]|2[0-9]|3[01])\.|localhost|127\.0\.0\.1\b)" --include=*
   # conversational attribution (adapt the first name)
   grep -rniE "(<FirstName>|decided|confirmed|asked|flagged|we chose|tried .* and broke|crash[- ]looped|root cause)" --include=*.py --include=*.ts --include=*.js --include=*.md --include=*.yml --include=*.yaml --include=*.conf --include=*.toml --include=*.json
   ```
   Skip vendored/output dirs (`node_modules`, `.venv`, `__pycache__`,
   `build`, `dist`, lockfiles, generated code) unless they are shipped.

6. **Produce the report.** Group findings by the risk-category table above.
   For each: `file:line`, the offending excerpt (redacted if it's a live
   secret — see redaction rule below), why it's a risk, and the fix. Give
   counts and worst-offender files/dirs. List the ~15–20 most impactful
   examples; summarize the rest grouped by category count and worst-offender
   file/dir (e.g. "+ 12 more Category 3 findings across 4 files (worst:
   `config/`, 7 findings)") rather than a flat "and more."
   - **Redaction rule:** never show a live secret in full. Show a short
     prefix + length only, e.g. `sk-abc1... (redacted, 43 chars)`. This
     applies everywhere the finding appears — chat, and any report file.
   - **If the audit finds nothing:** still state the posture explicitly
     (e.g. "clean — no findings across N files scanned") rather than a bare
     "no issues," so the report reads as complete, not skipped.
   Support **two verbosity levels** — default to concise, expand on request:
   - **Concise (default):** one-line per finding: `file:line — [severity]
     one-line summary of the risk + one-line fix`.
   - **Verbose (opt-in; ask or offer, don't assume):** for every finding show
     the full path, the line number, a short excerpt of the offending line/
     comment, what category it maps to, why it's a risk, the proposed fix, and
     a one-line "impact if left as-is". Ask which level the owner wants, or
     default to concise and offer the verbose version after delivering.
   - **Report destination:** ask whether the owner wants the report only in
     chat, or also written to a file. If written to a file, warn that a
     findings report is itself sensitive (it lists real secrets/PII
     locations) and confirm it should be gitignored or kept outside the repo
     — never let it land as a tracked file by default.
7. **Give remediation — separate phases:**
   - **Phase A (do first, smallest, highest severity):** secrets + PII.
     Rotate/revoke anything sincerely exposed; never leave a real secret even
     if seemingly inactive. Remove PII.
   - **Phase B:** operational info + private cross-references.
   - **Phase C:** narrative-comment cleanup (rewrite to generic, remove
     names/dates/incident narrative) + closing inferred-leak combinations.
   - Re-run the scan after each phase to confirm the old issues are gone
     before moving on.
   - **Report every fix as it is made.** For each change: output the diff (the
     `before` → `after` text, not just a summary), plus the file location
     (path) and the severity/category it addressed, so the owner can verify
     exactly what was altered. Batch the diffs per phase so each phase is
     reviewable on its own.
   - **Ask how the owner wants to approve fixes before applying them.** Offer
     the choice rather than assuming one mode. Options:
     - **Approve per fix (most granular):** pause after each proposed change
       and show the diff + file location + severity; only apply it on the
       owner's go-ahead, then move to the next. Best if the owner wants to
       eyeball and decide on each item individually.
     - **Approve per phase (default):** apply within a phase, then present all
       that phase's diffs once for a single sign-off before committing.
     - **Approve once at the end:** apply everything, then present all diffs
       for a final review.
     Default to "per phase" and let the owner override. Regardless of choice,
     never commit without explicit approval.
8. **Recommend a standing guard (prevention).** Suggest adding a mechanical
   pre-commit check (e.g. a `git grep` door + a secret scanner such as
   gitleaks / trufflehog) matching the patterns above, and note that pattern
   lists are never exhaustive — periodic manual re-audits (this skill) catch
   what the fixed list misses.
9. **Git history caveat (optional, offer — not a must).** Fixing the *current*
   content does not remove text already sitting in past commits (`git log` /
   `git blame` on GitHub still show them). Offer this as an *option* the owner
   can take or decline, never a mandatory step: accept history as-is (only safe
   if nothing was a real secret / PII), or rewrite with `git filter-repo` +
   force-push — destructive, changes every hash, breaks existing clones, ~only
   for genuinely public exposure. If the owner declines, move on; don't keep
   raising it. Lay out the trade-off, let the owner decide; never force-push
   unilaterally.
   - **Also check `.gitignore` coverage against history.** A file being
     gitignored *now* doesn't mean it was always ignored — check whether
     anything currently excluded was tracked in the past (`git log --all
     --full-history -- <path>`) and is still sitting in history despite
     looking "safe" today. Report this as its own line, separate from the
     general history caveat above, since it's a coverage gap rather than a
     pre-existing exposure the owner already knows about.
10. **Do not commit immediately — always prompt first.** No file edits, no
   commits, no stage/Push, and no rewrite of git history without the owner's
   explicit go-ahead. After each remediation phase, pause and present what was
   changed (the diffs + file locations + severities) and ask for review before
   committing anything. Only stage and commit (with the owner's wording or a
   conventional-commit message matching the repo) once the owner approves; a
   commit never happens as a side effect of the audit. If anything real was
   committed, rotate it.

## Inputs the skill can optionally gather

If the user enables them (ask first), these sharpen the report by making
"inferred risk" checks concrete:
- repo path(s) to sweep
- owner's first name (for the conversational-attribution grep)
- hearing of a private docs/storage repo name
- which branches/tags to document (public branch vs. local)

## Output

A markdown report with, in order:

1. **Executive summary** — top line: did we find critical/high risk, one
   sentence on the overall posture.
2. **Findings by risk category**, each finding `file:line`, severity, redacted
   value, why it's a mistake.
3. **Remediation phases** — Phase A (secrets/PII/rotate), Phase B
   (operational/private refs), Phase C (comments + inferred leaks), and how to
   re-verify.
4. **Prevention** — the mechanical check / secret scanner to add, and the
   standing rule about comment scope.
5. **Git history note** — whether history itself is a risk and the options,
   including whether anything now-gitignored was previously tracked.

If the audit finds nothing, still produce all five sections stating the
clean posture explicitly (e.g. "clean — no findings across N files
scanned") rather than a bare "no issues."

## Rules of thumb

- A secret that *might* be real → rotate, don't just delete.
- Any PII → remove it; if nursing it to a real person also combine with other
   fields or git history, flag it HIGH.
- Operational detail that a stranger could use to find/attack the system is
  HIGH; a rest-of-the-story/incident narrative is a higher-sensitivity variant.
- The mechanical check passing is **not** proof of clean — pattern lists are
  a floor.
- If in doubt about a field, flag it in the report rather than silently
  dropping it from the audit.
- Keep scope discipline: stay on information risk, don't balloon into code
  review.
