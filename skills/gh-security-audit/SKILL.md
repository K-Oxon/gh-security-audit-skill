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
   `API_VERSION=2026-03-10`.
3. Store raw API output only in a temporary directory.
4. Interpret results with `references/finding_model.md`.
5. Use official source links from `references/github_api_sources.md` when
   explaining API behavior.
6. Produce Markdown and JSON from the same finding set.

## Output

Markdown must contain:

1. Summary
2. Scope and assumptions
3. Collection status
4. Observed security settings
5. Branch protection and rules inventory
6. Alert availability and counts
7. Actions, deployment, and runtime inventory
8. Manual or unavailable checks
9. Data access and limitations
10. Optional review notes

JSON must include `schema_version`, `subject`, `policy_context`, `findings`,
and `limitations`. Use `schema_version: "0.1"`.

Default v0.1 output is inventory-first. Report observed values, counts, and API
limitations before making any recommendation. If a status is included, assign it
only from the deterministic rules in `references/finding_model.md`; do not infer
severity from general security intuition.

## v0.1 Boundaries

Do not include workflow file diagnostics, zizmor, Scorecard, local clone
inspection, cloud provider trust-policy inspection, or automatic fixes in v0.1
results. Mark those as `SKIP`, `MANUAL`, or out of scope as appropriate.

Do not treat repository ruleset count as a default-branch protection verdict.
Use active default-branch rules and legacy branch protection evidence.

Do not treat CodeQL default setup as proof that all code scanning is absent.
Advanced setup and third-party SARIF upload require separate evidence.

Never turn API access failures, permission gaps, feature-disabled responses, or
plan differences into `FAIL`. Record them as limitations.

Do not use `FAIL` unless the user explicitly asks to apply a policy profile that
defines failure conditions.

Never output secret scanning secret values or location details.
Never output Actions, Dependabot, environment, or Codespaces secret values.
