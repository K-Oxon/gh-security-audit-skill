# Command Recipe

This recipe performs a remote read-only audit for one GitHub repository. It uses
`gh api` as the primary data source and writes raw output only to a temporary
directory.

## Prerequisites

- `gh` is installed and authenticated.
- The token used by `gh` has read access to the target repository.
- Some optional evidence requires repository Administration, Actions,
  security-events, Attestations, or organization read permissions.
- `jq` is available for optional inspection and URL-encoding branch names.
- The repository is provided as `OWNER/REPO`.

Use a stable pinned GitHub REST API version explicitly. The examples below use
`2026-03-10`, which is the current GitHub REST API version when this recipe was
updated. Verify `references/github_api_sources.md` before changing it.

```sh
OWNER=K-Oxon
REPO=gh-security-audit-skill
API_VERSION=2026-03-10
OUT_DIR="$(mktemp -d "${TMPDIR:-/tmp}/gh-security-audit.XXXXXX")"
```

Raw output belongs in `OUT_DIR` or another temporary directory. Do not place raw
API output in tracked repository state.

## Core Read-Only Collection

The following endpoints are read-only. Query-string endpoints are quoted so zsh
does not treat `?` as a glob.

```sh
gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}" \
  > "${OUT_DIR}/repo.json"

DEFAULT_BRANCH="$(jq -r '.default_branch // empty' "${OUT_DIR}/repo.json")"
BRANCH_ENCODED="$(jq -rn --arg branch "${DEFAULT_BRANCH}" '$branch|@uri')"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/actions/permissions" \
  > "${OUT_DIR}/actions_permissions.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/actions/permissions/selected-actions" \
  > "${OUT_DIR}/actions_selected_actions.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/actions/permissions/workflow" \
  > "${OUT_DIR}/actions_workflow_permissions.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/actions/permissions/access" \
  > "${OUT_DIR}/actions_access.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/actions/permissions/fork-pr-contributor-approval" \
  > "${OUT_DIR}/actions_fork_pr_approval.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/actions/permissions/fork-pr-workflows-private-repos" \
  > "${OUT_DIR}/actions_fork_pr_private_repos.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/rulesets?includes_parents=true&targets=branch&per_page=100" \
  > "${OUT_DIR}/branch_rulesets_pages.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/rulesets?includes_parents=true&targets=tag&per_page=100" \
  > "${OUT_DIR}/tag_rulesets_pages.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/rulesets?includes_parents=true&targets=push&per_page=100" \
  > "${OUT_DIR}/push_rulesets_pages.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/rules/branches/${BRANCH_ENCODED}?per_page=100" \
  > "${OUT_DIR}/default_branch_active_rules_pages.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" \
  "repos/${OWNER}/${REPO}/branches/${BRANCH_ENCODED}" \
  > "${OUT_DIR}/default_branch.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/branches/${BRANCH_ENCODED}/protection" \
  > "${OUT_DIR}/default_branch_protection.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/code-security-configuration" \
  > "${OUT_DIR}/code_security_configuration.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" \
  "repos/${OWNER}/${REPO}/community/profile" \
  > "${OUT_DIR}/community_profile.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/contents/SECURITY.md?ref=${BRANCH_ENCODED}" \
  > "${OUT_DIR}/security_md_root.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/contents/.github/SECURITY.md?ref=${BRANCH_ENCODED}" \
  > "${OUT_DIR}/security_md_github_dir.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/contents/docs/SECURITY.md?ref=${BRANCH_ENCODED}" \
  > "${OUT_DIR}/security_md_docs_dir.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/code-scanning/default-setup" \
  > "${OUT_DIR}/codeql_default_setup.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/contents/.github/dependabot.yml?ref=${BRANCH_ENCODED}" \
  > "${OUT_DIR}/dependabot_yml.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/automated-security-fixes" \
  > "${OUT_DIR}/automated_security_fixes.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/vulnerability-alerts" \
  > "${OUT_DIR}/vulnerability_alerts.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/private-vulnerability-reporting" \
  > "${OUT_DIR}/private_vulnerability_reporting.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/codeowners/errors?ref=${BRANCH_ENCODED}" \
  > "${OUT_DIR}/codeowners_errors.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/immutable-releases" \
  > "${OUT_DIR}/immutable_releases.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/releases?per_page=100" \
  > "${OUT_DIR}/releases_pages.json"
```

## Paginated Alert and Runtime Inventory

These list endpoints can exceed one page. Keep the `--paginate --slurp` output
as pages in raw evidence, then flatten or sum pages during interpretation.

```sh
if ! gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/dependabot/alerts?state=open&per_page=100" \
  > "${OUT_DIR}/dependabot_alerts_open_pages.json"; then
  gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
    "repos/${OWNER}/${REPO}/dependabot/alerts?state=open&per_page=100" \
    > "${OUT_DIR}/dependabot_alerts_open.http" || true
fi

if ! gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/secret-scanning/alerts?state=open&per_page=100" \
  > "${OUT_DIR}/secret_scanning_alerts_open_pages.json"; then
  gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
    "repos/${OWNER}/${REPO}/secret-scanning/alerts?state=open&per_page=100" \
    > "${OUT_DIR}/secret_scanning_alerts_open.http" || true
fi

if ! gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/code-scanning/alerts?state=open&per_page=100" \
  > "${OUT_DIR}/code_scanning_alerts_open_pages.json"; then
  gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
    "repos/${OWNER}/${REPO}/code-scanning/alerts?state=open&per_page=100" \
    > "${OUT_DIR}/code_scanning_alerts_open.http" || true
fi

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/environments?per_page=100" \
  > "${OUT_DIR}/environments_pages.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/actions/secrets?per_page=100" \
  > "${OUT_DIR}/actions_secrets_pages.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/actions/organization-secrets?per_page=100" \
  > "${OUT_DIR}/actions_organization_secrets_pages.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/actions/variables?per_page=100" |
  jq '[.[] | {total_count, variables: [.variables[]? | {name, created_at, updated_at}]}]' \
  > "${OUT_DIR}/actions_variables_redacted_pages.json"

# The variables endpoint returns variable values. The jq filter above removes
# values before anything reaches disk. Never persist or output variable values.

jq -r '.[].environments[]?.name | @uri' "${OUT_DIR}/environments_pages.json" |
  while IFS= read -r ENVIRONMENT_ENCODED; do
    gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
      "repos/${OWNER}/${REPO}/environments/${ENVIRONMENT_ENCODED}/secrets?per_page=100" \
      > "${OUT_DIR}/environment_secrets.${ENVIRONMENT_ENCODED}.pages.json" || true
  done

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/actions/oidc/customization/sub" \
  > "${OUT_DIR}/oidc_subject_claim.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "repos/${OWNER}/${REPO}/dependency-graph/sbom" \
  > "${OUT_DIR}/dependency_graph_sbom_probe.http" || true

if ! gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/actions/runners?per_page=100" \
  > "${OUT_DIR}/self_hosted_runners_pages.json"; then
  gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
    "repos/${OWNER}/${REPO}/actions/runners?per_page=100" \
    > "${OUT_DIR}/self_hosted_runners.http" || true
fi
```

Artifact attestations are not generally enumerable without a subject digest. If
the user provides a release artifact digest, collect only that digest:

```sh
SUBJECT_DIGEST="sha256:..."
gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --paginate --slurp \
  "repos/${OWNER}/${REPO}/attestations/${SUBJECT_DIGEST}?per_page=100" \
  > "${OUT_DIR}/artifact_attestations_pages.json"
```

SBOM content inventory is intentionally not part of the default v0.1 recipe
scope. The `dependency-graph/sbom` probe above is HTTP-status evidence for the
`dependency_graph` finding only: use the status line, do not interpret the SBOM
body. Record SBOM coverage as `MANUAL` unless the user explicitly asks for SBOM
inventory.

## Optional Organization Context

Run these only when `OWNER` is an organization and the token has the required
organization read permissions. Missing access is a limitation, not a failure.

```sh
gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "orgs/${OWNER}/actions/permissions" \
  > "${OUT_DIR}/org_actions_permissions.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "orgs/${OWNER}/actions/permissions/workflow" \
  > "${OUT_DIR}/org_actions_workflow_permissions.http" || true

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include \
  "orgs/${OWNER}/actions/oidc/customization/sub" \
  > "${OUT_DIR}/org_oidc_subject_claim.http" || true
```

## Response Handling

Some endpoints may return non-2xx responses when a feature is disabled,
unavailable on the plan, unsupported for the repository, or inaccessible to the
token. Do not convert those collection failures into `FAIL`; record them in the
top-level or finding-level `limitations` field.

For endpoints where the HTTP status is evidence, use `--include` and inspect
the status line:

- `GET /repos/{owner}/{repo}/vulnerability-alerts` returns `204` when
  vulnerability alerts are enabled and can return `404` when disabled or
  unavailable.
- `GET /repos/{owner}/{repo}/private-vulnerability-reporting` returns the
  observed enablement state when available and may return an access,
  availability, or not-enabled response otherwise.
- `GET /repos/{owner}/{repo}/immutable-releases` can return `200` with JSON
  such as `enabled=true` or `enabled=false`; non-2xx responses are availability
  or access limitations.
- `GET /repos/{owner}/{repo}/branches/{branch}/protection` returns `200` when
  legacy branch protection exists for that branch and can return `404` when it
  does not exist or the token cannot view it. When `repo.json` reports
  `permissions.admin=true`, a `404` is evidence that legacy protection is
  absent, not an access gap.
- `GET /repos/{owner}/{repo}/contents/SECURITY.md` and the `.github/` and
  `docs/` path variants return `200` when a security policy file exists at that
  path on the requested ref and `404` when it does not. These probes are the
  security policy evidence; `GET /community/profile` provides community health
  context only and its response contains no security policy field.
- `GET /repos/{owner}/{repo}/code-security-configuration` can return `200`
  when a security configuration manages the repository, `204` when none is
  attached, or an access/availability error.
- `GET /repos/{owner}/{repo}/actions/permissions/fork-pr-contributor-approval`
  returns `200` with an `approval_policy` value when available.
- `GET /repos/{owner}/{repo}/actions/permissions/fork-pr-workflows-private-repos`
  applies to private repository settings. A public repository was observed to
  return `422`; that status is empirical, not documented, so classify any
  non-2xx as `SKIP` or limitation.
- `GET /repos/{owner}/{repo}/dependency-graph/sbom` returns `200` when
  Dependency Graph can produce an SBOM, which is evidence the feature is
  enabled. `404` is documented only as not found and cannot distinguish
  disabled, unsupported, or no-manifest states; record it as a limitation.

## Optional Inspection Helpers

Only run a `jq` helper after the corresponding endpoint produced JSON.

```sh
jq '{full_name, visibility, private, archived, default_branch, security_and_analysis,
  admin_permission: .permissions.admin,
  allow_forking, web_commit_signoff_required, delete_branch_on_merge}' \
  "${OUT_DIR}/repo.json"

jq '{enabled, allowed_actions, selected_actions_url, sha_pinning_required}' \
  "${OUT_DIR}/actions_permissions.json"

jq '{default_workflow_permissions, can_approve_pull_request_reviews}' \
  "${OUT_DIR}/actions_workflow_permissions.json"

jq '[.[][]] | length' "${OUT_DIR}/branch_rulesets_pages.json"
jq '[.[][]] | length' "${OUT_DIR}/tag_rulesets_pages.json"
jq '[.[][]] | length' "${OUT_DIR}/push_rulesets_pages.json"
jq '[.[][]] | length' "${OUT_DIR}/default_branch_active_rules_pages.json"
jq '{name, protected, protection}' "${OUT_DIR}/default_branch.json"
jq '{health_percentage, files}' "${OUT_DIR}/community_profile.json"
jq '[.[][]] | length' "${OUT_DIR}/secret_scanning_alerts_open_pages.json"
jq '[.[].environments[]?] | length' "${OUT_DIR}/environments_pages.json"
jq '[.[].secrets[]?] | length' "${OUT_DIR}/actions_secrets_pages.json"
jq '[.[].secrets[]?] | length' "${OUT_DIR}/actions_organization_secrets_pages.json"
jq '[.[].variables[]?] | length' "${OUT_DIR}/actions_variables_redacted_pages.json"
jq '[.[].runners[]?] | length' "${OUT_DIR}/self_hosted_runners_pages.json"
jq '[.[][]] | length' "${OUT_DIR}/releases_pages.json"
```

If an endpoint returned an error, classify that response using
`finding_model.md` instead.

## Sample Repository

Use this repository to verify the recipe:

```sh
OWNER=K-Oxon
REPO=gh-security-audit-skill
API_VERSION=2026-03-10
```

Expected sample behavior may change as the repository is hardened. Re-run the
recipe before using sample observations in a report.
