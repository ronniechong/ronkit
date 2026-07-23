---
name: review-ai
description: Reviews a technical document (system design, architecture, data schema, API spec — not a PRD) for AI usage and, if found, runs due diligence against AI governance categories (data flow, vendor risk, security/guardrails, incident response, evaluation, plus escalation items like disclosure and regulatory mapping), producing a prioritized report for an engineering lead. Use when the user asks to review a tech doc for AI governance, wants to know if a doc needs AI governance, asks "does this need guardrails/oversight", or runs /documents:review-ai. Read-only — never modifies the source doc, and does not draft or propose governance revisions to it — this skill only reviews and reports. Not for PRDs.
---

# review-ai

Reviews one or more technical documents for AI usage and produces a prioritized AI-governance report for an engineering lead. Never edits the source doc(s) — output is a report only. Proposing or drafting actual governance changes to a doc is a different, future skill; this one stops at "here's what's missing and how urgent it is."

## 1. Scope and inputs

- Input is a technical document: system design, architecture doc, data schema, API spec. Not a PRD — if handed a PRD, say so and ask for the tech doc instead.
- Input can be a file path/reference or text pasted directly into the prompt.
- Multiple related docs can be reviewed together, producing **one** consolidated report. Every finding cites which specific doc it came from.
- **Scope discipline:** only read the doc(s) explicitly provided. Never auto-read sibling files in the same folder, even obviously-related ones (system cards, risk registers, PRDs, eval reports). If such sibling files exist, name them in a "Scope note" as unread rather than silently ignoring them or reading them uninvited — some findings may already be addressed in a doc you didn't read, and the report should say so rather than confidently claim a gap.

## 2. Intake

Three inputs shape the review. For all three: **infer from the doc with confidence; ask the user only if it can't be determined.** Never silently assume when uncertain, and never ask when it's already clear.

1. **Commercial / personal / internal-tool** — which of the three, inferred from scale/audience/framing in the doc, confirmed with the user if unclear. Drives the severity multiplier (see §5). Personal projects score gentler; internal tools handling real data for real people don't get a free pass just because they're not customer-facing.
2. **Doc freshness/status** — draft, approved-but-unbuilt, or written after the system already shipped. Inferred from a status header, last-updated date, or tense used ("we will build" vs. "this is deployed").
3. **User base / jurisdiction** — who actually uses this system and roughly where. Inferred from audience/customer language or geography mentioned in the doc.

Run inference before detection where possible, but only surface a question to the user if it's both uncertain *and* actually needed for scoring — e.g. don't ask the multiplier question if AI turns out not to be detected at all, since there's nothing left to score.

## 3. AI detection

Always scan and report a verdict, even though the skill's own invocation implies AI is expected — never skip this step.

- **Detected** (confidence: high/med/low) — cite the specific evidence (component name, data flow, feature description).
- **Not detected** — show what was scanned and explicitly ruled out (don't just assert absence; name the technologies/features considered and why they don't count as AI). Frame this as **contestable**: invite the user to point out something missed, update the doc, or explain in conversation why AI is actually present, rather than treating "not detected" as a final verdict.
- Cast a broad net: personalization, auto-categorization, recommendation, generation, and classification all count as AI usage, not just doc text that explicitly says "AI/ML/LLM." Bias toward flagging ambiguous cases rather than silently passing them.
- If genuinely uncertain (some signal present but unclear), say **Uncertain** rather than forcing Detected or Not detected.

If the verdict is "Not detected," skip straight to a short report (verdict + evidence + contest invitation) — the categories and scoring below don't apply.

## 4. Categories

**Eng-lane** (the engineering lead can act on these directly):
1. AI detection & inventory
2. Data flow into AI systems — PII/sensitive data presence, storage/transmission, retention, training vs. inference use
3. Model/vendor & dependency risk — self-hosted vs. third-party, vendor data-use/training terms, lock-in/deprecation exposure
4. Security & abuse boundaries (guardrails) — prompt injection, adversarial input handling, agentic action blast radius/permissions, sandboxing, cost/rate-abuse protection
5. Incident response & human oversight — escalation path, postmortem/retro requirement, kill switch, human-in-the-loop gates, override mechanics. Rollback capability is a sub-item here, not the primary framing — don't let "is there an undo button" substitute for "is there an actual response process."
6. Evaluation & monitoring — accuracy/quality metrics, success/fail criteria, drift detection, decision logging/observability, model/prompt versioning

**Escalate-lane** (visible from the doc but outside engineering's authority — flag for follow-up, never drop silently):
7. User disclosure & consent
8. Consequential-decision fairness & appeal path — only relevant if the AI's output drives a decision affecting a person's opportunities, rights, or safety
9. IP & licensing — training data provenance, generated-content ownership
10. Regulatory mapping — sector-specific triggers visible in the doc (health data, minors, financial decisions, etc.)

For each category: if the doc addresses it well, list it under **Addressed — no finding** with a one-line reason (don't invent a gap in a well-governed doc just to have something to say). If it's genuinely not relevant given the doc's scope, list it under **Not applicable** with why. Only categories with a real, evidenced gap become **Findings**.

## 5. Risk signals, scoring, and rollout stage

**Hard-override signals** — force "Must address" outright, but *only when confirmed by the doc*, never when merely inferred or plausible. If ambiguous, it becomes an Undetermined finding instead of a redline:
- Safety-critical / physical-world impact
- Minors involved

**Weighted signals** — combine judgment, not arithmetic, into Must / Nice-to-have / Lower priority:
- PII or sensitive data present
- Decision is consequential (affects a person's opportunities, rights, or safety)
- Autonomous/agentic action with real-world side effects
- Reversibility of the action (irreversible raises severity; read-only/reversible lowers it)
- Generative output vs. classification/recommendation (generative carries more risk)
- Data leaves the org's infra to a third-party AI vendor
- Untrusted/adversarial input exposure (public-facing raises severity; internal-only lowers it)

As a rough guide: 2+ signals clearly present, or a commercial project with any signal present → lean **Must address**. Exactly one clear signal, or a personal project with multiple → lean **Nice to have**. A signal that's only weakly/indirectly implicated → **Lower priority**. Apply judgment over the letter of this rule — a single severe signal (e.g. real PII flowing to an unnamed third-party vendor) can outweigh a mechanical count.

**Multiplier — commercial / personal / internal-tool:** scales overall severity without capping any individual finding. An internal HR tool touching employee PII still scores high on data-flow/consequential-decision even though its overall multiplier sits mid-range; the multiplier tunes stakes, it doesn't erase a specific gap.

**Rollout/exposure stage** — extract from the doc's own language (canary, beta, A/B test, feature flag, GA, internal-only, "pre-implementation," etc.). Don't assume a fixed sequence (teams order canary/A/B/GA differently) — just bucket by degree of real-user exposure:
- **None** — internal/pre-launch, no real users or real data
- **Partial** — any % of real users (canary, beta, A/B, staged rollout)
- **Full** — GA, all users
- **Not specified** — the doc is silent on rollout; this is itself a finding, not a default assumption

Capture both the bucket and the doc's verbatim rollout description. Any real exposure (Partial or Full) makes baseline governance items live requirements *now*, regardless of how small or controlled — however, when the bucket is **None**, phrase Must-address findings as gates to clear **before production**, not as if they're blocking something already live.

**Priority values:** Must address · Nice to have · Lower priority · **Undetermined — needs input** (the doc is silent on a category entirely; flag the gap but don't invent a severity — that call belongs to the human).

## 6. Evidence and redaction

Every finding's evidence should be specific enough to act on: which doc section, what kind of data/decision, when in the flow it happens, how it's handled or transmitted. But redact literal sensitive content — real PII values, secrets, internal hostnames, specific contract terms — describing what it is rather than reproducing it verbatim. Structural detail stays; literal sensitive payloads don't.

## 7. Overall risk rating

A single Low / Medium / High figure for the report's headline, derived from severity **combined with** rollout exposure — not severity alone, since the same gap is worse on a live system than on a pre-launch POC:
- **High** — a confirmed hard-override signal present, OR a Must-address finding alongside real exposure (Partial/Full rollout)
- **Medium** — a Must-address finding exists but rollout is None, OR multiple Nice-to-have findings alongside real exposure
- **Low** — only Nice-to-have/Lower-priority findings, or no findings, regardless of exposure

## 8. Report structure

1. **Header** — version (`v<N>`), doc(s) reviewed (with paths), review date. No "reviewed by" line.
2. **Scope note** — unread sibling docs, if any (§1)
3. **AI Detection** — verdict, confidence, evidence (§3)
4. **Intake** — the three answers/inferences from §2, with source (inferred vs. asked)
5. **Rollout / exposure stage** — bucket + verbatim doc description (§5)
6. **Summary** — overall risk rating (§7) as the headline, then a counts table by priority × lane
7. **Findings** — one table per **category** (not one mixed table), each showing Priority | Source doc | Evidence | Signals triggered | Recommendation. Categories with no gap go under "Addressed — no finding" or "Not applicable" instead of an empty table.
8. **Escalation follow-ups** — Escalate-lane findings pulled into their own section so they don't get lost
9. **History log** — one line per version generated for this doc (`v1`, `v2`, ...): date, headline finding counts, scope. Append to this on every re-run rather than silently overwriting.
10. **Disclaimer** — one line: this is a due-diligence checklist to surface gaps for follow-up, not a legal or compliance sign-off.

## 9. Output delivery

- Always give a short conversational summary first: verdict, overall risk rating, count of findings by priority, and the single most important finding — not the full report.
- Then offer to write the full report to a file.
- Filename: `<tech-doc-name>-ai-governance-review-v<N>-<YYYY-MM-DD>.md` — `<tech-doc-name>` is the reviewed doc's own filename (slugified kebab-case, extension dropped; primary/first doc's name for multi-doc reviews). `v<N>` increments per re-run against the same doc. Date only, no time-of-day — `v<N>` already disambiguates same-day re-runs.
- File location: if the doc was provided as a file, write the report to that doc's own folder (primary doc's folder for multi-doc reviews). If the doc was pasted directly into the prompt with no file, ask the user where to save.

## 10. Non-goals

- Never modifies or annotates the source doc.
- Does not propose or draft governance revisions — report and flag only.
- PRDs are out of scope.
- No diffing against a prior version's report, no skipping findings because "the org already has governance for this," no machine-readable/CI-integration output — none of these are handled by this skill.
