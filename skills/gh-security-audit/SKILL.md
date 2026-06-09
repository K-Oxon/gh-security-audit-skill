---
name: gh-security-audit
description: Read-only remote GitHub repository security settings audit using gh api for one OWNER/REPO.
license: Apache-2.0
---

# gh-security-audit

Use this skill when the user asks to audit GitHub repository security settings
or Actions repository settings for a specific `OWNER/REPO`.

This skill is read-only. Do not change repository settings, open pull requests,
commit files, dismiss alerts, enable features, or run remediation commands.

## Inputs

Require a repository in `OWNER/REPO` form. If the owner or repo is ambiguous,
ask for clarification before calling GitHub APIs.

## Workflow

1. Read `references/command_recipe.md`.
2. Run the read-only `gh api` recipe with `OWNER`, `REPO`, and
   `API_VERSION=2022-11-28`.
3. Store raw API output only in a temporary directory.
4. Interpret results with `references/finding_model.md`.
5. Use official source links from `references/github_api_sources.md` when
   explaining API behavior.
6. Produce Markdown and JSON from the same finding set.

## Output

Markdown must contain:

1. Summary
2. Scope and assumptions
3. Overall status
4. High-priority findings
5. All findings
6. Manual checks
7. Data access and limitations
8. Recommended next actions

JSON must include `schema_version`, `subject`, `policy_context`, `findings`,
and `limitations`. Use `schema_version: "0.1"`.

## v0.1 Boundaries

Do not include workflow file diagnostics, zizmor, Scorecard, local clone
inspection, or automatic fixes in v0.1 results. Mark those as `SKIP`,
`MANUAL`, or out of scope as appropriate.

Never turn API access failures, permission gaps, feature-disabled responses, or
plan differences into `FAIL`. Record them as limitations.

Never output secret scanning secret values or location details.
