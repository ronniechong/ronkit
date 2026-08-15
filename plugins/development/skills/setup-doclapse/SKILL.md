---
name: development-setup-doclapse
description: >-
  Generates a GitHub Actions workflow file that wires up doc-lapse-action
  (github.com/ronniechong/doc-lapse-action) — a documentation-drift-detection
  Action — for a chosen repo. Interviews the user for the target repo, LLM
  provider/model (informed by doc-lapse-action's own compatibility table),
  and config options, then writes .github/workflows/doc-lapse.yml. Checks
  first whether the repo already has it configured and pauses if so. Use
  when the user wants to set up, add, wire up, or configure doc-lapse-action
  (or "the doc drift/lapse action") for a repo.
---

# Setup DocLapse

Generate a working `.github/workflows/doc-lapse.yml` for a repo adopting
[doc-lapse-action](https://github.com/ronniechong/doc-lapse-action). This
skill only writes that one file — it never runs git commands and never
touches a live secret value.

## Step 1 — Target repo

Ask for the target repo as an **absolute path**. Don't assume the current
working directory — this skill can be invoked from any session. Confirm the
path exists and contains a `.git` directory before proceeding; if either
check fails, say so and stop.

## Step 2 — Existing-setup check (before any interview questions)

Scan `<repo>/.github/workflows/*.yml` and `*.yaml` for any **active**
(non-commented) `uses:` line referencing `ronniechong/doc-lapse-action`, at
any ref/tag. Do not rely on the filename `doc-lapse.yml` alone — the repo
may have adopted it under a different name.

If a match is found: **stop and warn** before asking anything else. Report
the matching file name(s) and the version already pinned there, then ask the
user to choose one of:

- **Abort** — leave the existing setup alone, end here.
- **Overwrite** — replace that existing file with a freshly generated one.
- **Write fresh** — create a new `doc-lapse.yml` alongside the existing
  file. If chosen, warn explicitly: two workflows both triggering on
  `pull_request` and referencing the same action means duplicate runs and
  duplicate PR comments. Make sure the user understands this before
  proceeding, don't just note it in passing.

Do not fall through to Step 3 silently — this is a hard pause, not a
soft heads-up folded into the write step.

## Step 3 — Compatibility table

Fetch `COMPATIBILITY.md` live via WebFetch from:

```
https://raw.githubusercontent.com/ronniechong/doc-lapse-action/main/COMPATIBILITY.md
```

Render its provider/model tables so the user can pick an informed
`llm-provider` + `model`. If the fetch fails (network issue, file moved),
fall back to this summary and say the live table couldn't be reached:

- **Verified:** OpenAI, Anthropic (native structured-output mode)
- **Should work, unverified:** any OpenAI-compatible endpoint — Ollama, LM
  Studio, OpenRouter, self-hosted vLLM (`llm-provider: ollama`)
- **Not supported:** anything without an official or community AI SDK
  provider package

Also mention: reasoning models (`gpt-5` family, `o`-series) don't expose
temperature control, so they have inherent run-to-run verdict variance —
worth knowing when picking a model, not just which provider is "supported."

## Step 4 — Interview (two-tier)

**Essentials — always ask:**

- `llm-provider` (`openai`, `anthropic`, `ollama`, or another
  OpenAI-compatible endpoint via `ollama`)
- `model` — informed by Step 3's table; leaving it blank is valid (the
  Action falls back to a built-in per-provider default)
- `docs-path` — newline-separated glob patterns, default `**/*.md`
- `api-key-secret` — the **name** of the repo secret holding the LLM key.
  Never ask for or accept the actual key value.

**Advanced — offer as one yes/no gate** ("Want to configure advanced
options, or use the defaults?"). If yes, walk through:

- `fail-on-drift` (default `false`)
- `custom-instructions` (default empty)
- `docs-repo` / `docs-ref` / `docs-token-secret` — only relevant if docs
  live in a different repo than the one being checked
- `max-checks` (default `20`)
- `pii-strict` (default `false`)

If declined, omit all of these keys from the generated `with:` block
entirely — let `action.yml`'s own defaults apply rather than writing
redundant default lines.

## Step 5 — Resolve the version to pin

Always pin the **latest published release tag**, not the moving `@v1` major
tag. Resolve it via WebFetch against the public releases page:

```
https://github.com/ronniechong/doc-lapse-action/releases/latest
```

This URL redirects to the tag-specific release page — read the resolved tag
from that redirect. This deliberately avoids the GitHub REST API: don't
assume the user has `gh` CLI, a GitHub PAT, or the GitHub MCP server
configured. If the fetch fails, fall back to `@v1` and tell the user the
exact tag couldn't be resolved, so they can check manually.

Mention in the output: pinning an exact tag means updates are exact and
reproducible but won't auto-follow non-breaking releases the way the
moving `@v1` tag would — the user can switch to `@v1` themselves if they'd
rather have that tradeoff.

## Step 6 — Generate and write the file

Build the workflow matching doc-lapse-action's documented usage shape:

```yaml
on:
  pull_request:

permissions:
  pull-requests: write   # required — the Action posts/updates a PR comment

concurrency:
  group: doc-lapse-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  check-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ronniechong/doc-lapse-action@<resolved-version>
        with:
          docs-path: |
            <docs-path>
          llm-provider: <llm-provider>
          api-key-secret: <api-key-secret>
          model: <model>          # omit if left blank
          # ...any advanced inputs the user configured
        env:
          <api-key-secret>: ${{ secrets.<api-key-secret> }}
          # plus <docs-token-secret> mapping, if configured
```

Write to `<repo>/.github/workflows/doc-lapse.yml`. Step 2 already covers the
case where doc-lapse-action is configured elsewhere; this step separately
guards the narrower case where a `doc-lapse.yml` exists but does **not**
reference the action (e.g. a stray/renamed file) — ask before overwriting,
don't silently clobber it.

## Step 7 — Output: next steps, no git, no secrets

Never run `git add`/`commit`/`push`. Never ask for or handle a live secret
value. After writing the file, tell the user plainly:

1. Add the named secret(s) via repo settings (`Settings → Secrets and
   variables → Actions`) or `gh secret set <name>`.
2. **Fork-PR limitation:** `pull_request`-triggered workflows get a
   read-only token on PRs from forks — comment-posting silently won't
   happen on external-contributor PRs unless the user takes on
   `pull_request_target`'s own tradeoffs themselves (not something this
   skill does for them).
3. **Never also trigger on `issue_comment`** (or any event that fires when
   the Action posts its own sticky comment) — that risks an unbounded
   self-triggering loop against the user's LLM budget.

## Non-goals

This skill does not: validate the generated YAML against `action.yml`'s
schema, create the repo secret, open a pull request, or modify any other
file in the target repo.
