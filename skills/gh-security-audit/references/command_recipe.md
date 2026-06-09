# Command Recipe

This recipe performs a v0.1 remote-api audit for one GitHub repository.
It is read-only and uses `gh api` as the primary data source.

## Prerequisites

- `gh` is installed and authenticated.
- The token used by `gh` has read access to the target repository.
- `jq` is available for optional inspection.
- The repository is provided as `OWNER/REPO`.

Use the current GitHub REST API version explicitly:

```sh
OWNER=K-Oxon
REPO=gh-security-audit-skill
API_VERSION=2022-11-28
OUT_DIR="$(mktemp -d "${TMPDIR:-/tmp}/gh-security-audit.XXXXXX")"
```

Raw output belongs in `OUT_DIR` or another temporary directory. Do not place raw
API output in tracked repository state.

## Read-Only Collection

The following endpoints are read-only. Query-string endpoints are quoted so zsh
does not treat `?state=open` as a glob.

```sh
gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}" \
  > "${OUT_DIR}/repo.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/actions/permissions" \
  > "${OUT_DIR}/actions_permissions.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/actions/permissions/workflow" \
  > "${OUT_DIR}/actions_workflow_permissions.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/rulesets" \
  > "${OUT_DIR}/rulesets.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/code-scanning/default-setup" \
  > "${OUT_DIR}/codeql_default_setup.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/automated-security-fixes" \
  > "${OUT_DIR}/automated_security_fixes.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" --include "repos/${OWNER}/${REPO}/vulnerability-alerts" \
  > "${OUT_DIR}/vulnerability_alerts.http"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/dependabot/alerts?state=open" \
  > "${OUT_DIR}/dependabot_alerts_open.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/secret-scanning/alerts?state=open" \
  > "${OUT_DIR}/secret_scanning_alerts_open.json"

gh api -H "X-GitHub-Api-Version: ${API_VERSION}" "repos/${OWNER}/${REPO}/code-scanning/alerts?state=open" \
  > "${OUT_DIR}/code_scanning_alerts_open.json"
```

Some endpoints may return non-2xx responses when a feature is disabled,
unavailable on the plan, unsupported for the repository, or inaccessible to the
token. Do not convert those collection failures into `FAIL`; record them in the
top-level or finding-level `limitations` field.

For endpoints where the HTTP status is the evidence, use `--include` and inspect
the status line. `GET /repos/{owner}/{repo}/vulnerability-alerts` returns `204`
when vulnerability alerts are enabled and can return `404` when disabled or
unavailable.

## Optional Inspection Helpers

These commands help summarize collected responses without changing GitHub state:

```sh
jq '{full_name, visibility, private, archived, default_branch, security_and_analysis}' \
  "${OUT_DIR}/repo.json"

jq '{enabled, allowed_actions, selected_actions_url, sha_pinning_required}' \
  "${OUT_DIR}/actions_permissions.json"

jq '{default_workflow_permissions, can_approve_pull_request_reviews}' \
  "${OUT_DIR}/actions_workflow_permissions.json"

jq 'length' "${OUT_DIR}/rulesets.json"
jq '{state, languages}' "${OUT_DIR}/codeql_default_setup.json"
jq '{enabled}' "${OUT_DIR}/automated_security_fixes.json"
jq 'length' "${OUT_DIR}/secret_scanning_alerts_open.json"
```

Only run a `jq` helper after the corresponding endpoint produced JSON. If an
endpoint returned an error, classify that response using
`finding_model.md` instead.

## Sample Repository

Use this repository to verify the recipe:

```sh
OWNER=K-Oxon
REPO=gh-security-audit-skill
API_VERSION=2022-11-28
```

Expected sample behavior may change as the repository is hardened. At the time
the v0.1 plan was written, the sample repository allowed verification of:

- repository metadata collection
- Actions permission collection
- workflow token permission collection
- empty rulesets producing a deterministic review flag
- CodeQL default setup `not-configured` producing a deterministic review flag
- feature-disabled or unavailable alert endpoints producing limitations

Re-run the recipe before using sample observations in a report.
