# Review Items Full Implementation Plan

## Scope

Implement the full post-review coverage pass for `skills/gh-security-audit`
without adding remediation commands or local workflow/static analysis.

## Desired State

- The recipe collects broader read-only GitHub REST API evidence for repository
  security settings, effective default-branch rules, legacy branch protection,
  CODEOWNERS errors, private vulnerability reporting, deployment environments,
  OIDC subject customization, self-hosted runners, immutable releases, and alert
  counts.
- SBOM export remains `MANUAL` for v0.1 scope reasons, even though GitHub
  provides read-only Dependency Graph SBOM APIs.
- Paginated list endpoints either use `--paginate` or document pagination
  limitations explicitly.
- The finding model stays inventory-first. It reports observed state and review
  labels only from deterministic rules; it does not infer policy failure from
  general security best practice.
- Rulesets are not treated as sufficient branch protection by count alone.
  Active rules for the default branch and legacy branch protection are separate
  evidence surfaces.
- CodeQL default setup is not treated as proof that all code scanning is absent;
  advanced setup and third-party SARIF remain separately described limitations.
- Current GitHub security best-practice areas that v0.1 cannot prove are listed
  as explicit `MANUAL`, `SKIP`, or limitation items.

## Steps

1. Update `command_recipe.md` with the expanded read-only endpoint set and
   pagination guidance.
2. Update `finding_model.md` with deterministic labels for the expanded
   evidence, plus stronger separation rules.
3. Update `github_api_sources.md` with official documentation links used by the
   expanded recipe.
4. Update `SKILL.md`, `README.md`, and sample outputs to match the new contract.
5. Validate JSON, run `gh skill publish --dry-run`, request an independent
   sub-agent review, fix any concrete misses, then commit and push.

## Risks

- Some endpoints require repository, Actions, security-events, or organization
  read permissions. Missing permissions must be limitations, not `FAIL`.
- Some GitHub features are plan-dependent or unavailable for personal
  repositories. Those responses must not become policy findings.
- OIDC, environments, workflow-level permissions, artifact attestations, and
  self-hosted runner exposure may require workflow or organization context to
  interpret correctly; v0.1 should inventory them, not judge them.

## Verification

- `jq empty skills/gh-security-audit/examples/sample_findings.json`
- `gh skill publish --dry-run`
- Independent sub-agent review after implementation
