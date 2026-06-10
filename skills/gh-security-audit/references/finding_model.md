# Finding Model

v0.1 is inventory-first. It reports what GitHub APIs returned, what could not be
verified, and which observed states should be reviewed. It is not a compliance
verdict or an automated risk rating.

Do not let the model freely decide whether a setting is "good" or "bad".
Statuses in default v0.1 output are deterministic review labels from this file.
Put the actual repository state in `observed`.

## Status Values

- `PASS`: The endpoint returned usable evidence and the deterministic table does
  not mark the observed state for review. `PASS` does not mean safe.
- `WARN`: The endpoint returned usable evidence and the deterministic table marks
  the observed state for human review. `WARN` is not a severity judgment.
- `FAIL`: Reserved for an explicit user-provided policy profile. Do not use
  `FAIL` in default inventory mode.
- `MANUAL`: Human judgment, workflow content, organization context, cloud trust
  policy, or non-API evidence is required.
- `SKIP`: The check is out of scope, unsupported, or a prerequisite is missing.
  `SKIP` does not mean safe.

## Severity Values

- `critical`: Immediately exploitable exposure or major defense failure.
- `high`: Important defense disabled or broad impact.
- `medium`: Recommended defense missing or operational risk.
- `low`: Improvement or limited impact.
- `info`: Contextual audit fact.

Severity is optional in default inventory mode. If included, keep it separate
from status and treat it as a coarse review aid, not as a policy decision.

## Finding Shape

Each finding should include:

- `id`
- `title`
- `status`
- `severity`
- `confidence`
- `source.type`
- `source.name`
- `observed`
- `policy_expected` only when the user supplied or selected a policy profile
- `evidence`
- `review_note`
- `limitations`

Use these source types in v0.1:

- `github_api`
- `manual`
- `out_of_scope`

## Required JSON Shape

```json
{
  "schema_version": "0.1",
  "subject": {
    "owner": "OWNER",
    "repo": "REPO",
    "full_name": "OWNER/REPO",
    "visibility": "public",
    "private": false,
    "default_branch": "main",
    "archived": false
  },
  "policy_context": {
    "id": "v0.1-inventory-default",
    "api_version": "2026-03-10",
    "generated_at": "YYYY-MM-DDTHH:MM:SSZ",
    "mode": "remote-api-inventory",
    "sources": []
  },
  "findings": [],
  "limitations": []
}
```

Markdown and JSON outputs must be generated from the same finding set. Do not
change statuses, counts, observed values, or limitations between the two
formats.

## Deterministic v0.1 Review Labels

Use this table for default v0.1 status assignment. Do not invent stronger
judgments from general security intuition.

| ID | Source | Observed state | Default status |
| --- | --- | --- | --- |
| `repo_metadata` | `GET /repos/{owner}/{repo}` | `archived=false` and `default_branch` present | `PASS` |
| `repo_metadata` | `GET /repos/{owner}/{repo}` | `archived=true` or missing `default_branch` | `WARN` |
| `repository_security_and_analysis` | `GET /repos/{owner}/{repo}` | Record `security_and_analysis` fields returned by GitHub | `PASS` unless a returned status is explicitly `disabled`, then `WARN` |
| `dependency_graph` | `GET /repos/{owner}/{repo}` or security configuration | enabled | `PASS` |
| `dependency_graph` | `GET /repos/{owner}/{repo}` or security configuration | disabled, absent, or unavailable | `WARN` if supported; otherwise limitation |
| `automatic_dependency_submission` | security configuration or repository metadata when present | enabled | `PASS` |
| `automatic_dependency_submission` | security configuration or repository metadata when present | disabled, not set, absent, or unavailable | `MANUAL` unless the repository policy requires it |
| `code_security_configuration` | `GET /code-security-configuration` | `200` with attached configuration details | `PASS` |
| `code_security_configuration` | `GET /code-security-configuration` | `204` no attached configuration | `MANUAL` |
| `security_policy_file` | `GET /contents/SECURITY.md` probes (root, `.github/`, `docs/`) | any probe returns `200` | `PASS` |
| `security_policy_file` | `GET /contents/SECURITY.md` probes (root, `.github/`, `docs/`) | all probes return `404` | `WARN` for public repos; `MANUAL` for private/internal repos |
| `security_policy_file` | `GET /contents/SECURITY.md` probes (root, `.github/`, `docs/`) | any probe returns another non-2xx | limitation |
| `actions_repository_permissions` | `GET /actions/permissions` | Actions enabled with `allowed_actions=selected` or `local_only`, and `sha_pinning_required=true` | `PASS` |
| `actions_repository_permissions` | `GET /actions/permissions` | `enabled=false`, `allowed_actions=all`, or `sha_pinning_required=false` | `WARN` |
| `actions_selected_actions` | `GET /actions/permissions/selected-actions` | selected actions details returned | `PASS`; inventory only |
| `actions_selected_actions` | `GET /actions/permissions/selected-actions` | unavailable because `allowed_actions` is not `selected` | `SKIP` |
| `actions_workflow_token_permissions` | `GET /actions/permissions/workflow` | `default_workflow_permissions=read` and `can_approve_pull_request_reviews=false` | `PASS` |
| `actions_workflow_token_permissions` | `GET /actions/permissions/workflow` | `default_workflow_permissions=write` or PR approval allowed | `WARN` |
| `actions_access_private_repository` | `GET /actions/permissions/access` | private repository access level returned | `PASS`; inventory only |
| `actions_access_private_repository` | `GET /actions/permissions/access` | public repository or endpoint not applicable | `SKIP` |
| `branch_rulesets_inventory` | `GET /rulesets?includes_parents=true&targets=branch` | any count | `PASS`; inventory only, never a protection verdict by count |
| `default_branch_active_rules` | `GET /rules/branches/{branch}` | one or more active rules returned for default branch | `PASS`; describe rule types and sources |
| `default_branch_active_rules` | `GET /rules/branches/{branch}` | zero active rules returned | `WARN`; review legacy branch protection |
| `default_branch_summary` | `GET /branches/{branch}` | branch summary returned | `PASS`; inventory `protected` and protection summary only |
| `default_branch_legacy_protection` | `GET /branches/{branch}/protection` | `200` with protection settings | `PASS`; inventory only |
| `default_branch_legacy_protection` | `GET /branches/{branch}/protection` | `404` and `repo.json` `permissions.admin=true` and zero active default-branch rules | `WARN`; legacy protection is confirmed absent by an admin token and no ruleset rules apply either |
| `default_branch_legacy_protection` | `GET /branches/{branch}/protection` | `404` and `repo.json` `permissions.admin=true` and one or more active default-branch rules | `MANUAL`; legacy protection is confirmed absent, review rule sufficiency via `default_branch_active_rules` |
| `default_branch_legacy_protection` | `GET /branches/{branch}/protection` | `404` without admin-token evidence, or otherwise unavailable | `MANUAL`; absence and lack of access cannot be distinguished |
| `codeql_default_setup` | `GET /code-scanning/default-setup` | `state=configured` | `PASS` |
| `codeql_default_setup` | `GET /code-scanning/default-setup` | `state` other than `configured` | `WARN`, with caveat that advanced setup or third-party SARIF may still exist |
| `dependabot_version_updates_config` | `GET /contents/.github/dependabot.yml` | file found | `PASS`; inventory only |
| `dependabot_version_updates_config` | `GET /contents/.github/dependabot.yml` | file missing or unavailable | `MANUAL`; Dependabot security updates can still work without version updates |
| `dependabot_security_updates` | `GET /automated-security-fixes` and `GET /repos` | enabled | `PASS` |
| `dependabot_security_updates` | `GET /automated-security-fixes` and `GET /repos` | disabled | `WARN` |
| `vulnerability_alerts` | `GET /vulnerability-alerts` and `GET /repos` | `204` or enabled status | `PASS` |
| `vulnerability_alerts` | `GET /vulnerability-alerts` and `GET /repos` | disabled response | `WARN` |
| `private_vulnerability_reporting` | `GET /private-vulnerability-reporting` | enabled response or `enabled=true` | `PASS` |
| `private_vulnerability_reporting` | `GET /private-vulnerability-reporting` | `200` with `enabled=false` on an applicable public repo | `WARN` |
| `private_vulnerability_reporting` | `GET /private-vulnerability-reporting` | non-2xx, unavailable, inaccessible, or inapplicable repository | `SKIP` or limitation; not `WARN` |
| `secret_scanning` | `GET /repos` `security_and_analysis` | enabled | `PASS` |
| `secret_scanning` | `GET /repos` `security_and_analysis` | disabled | `WARN` |
| `secret_scanning_push_protection` | `GET /repos` `security_and_analysis` | enabled | `PASS` |
| `secret_scanning_push_protection` | `GET /repos` `security_and_analysis` | disabled | `WARN` |
| `secret_scanning_validity_checks` | `GET /repos` `security_and_analysis` | status explicitly `enabled` | `PASS` |
| `secret_scanning_validity_checks` | `GET /repos` `security_and_analysis` | status explicitly `disabled` | `WARN` |
| `secret_scanning_validity_checks` | `GET /repos` `security_and_analysis` | field absent from response | `MANUAL` with limitation; do not guess plan or feature support |
| `secret_scanning_non_provider_patterns` | `GET /repos` `security_and_analysis` | status explicitly `enabled` | `PASS` |
| `secret_scanning_non_provider_patterns` | `GET /repos` `security_and_analysis` | status explicitly `disabled` | `WARN` |
| `secret_scanning_non_provider_patterns` | `GET /repos` `security_and_analysis` | field absent from response | `MANUAL` with limitation; do not guess plan or feature support |
| `codeowners_errors` | `GET /codeowners/errors` | zero errors | `PASS` |
| `codeowners_errors` | `GET /codeowners/errors` | one or more errors | `WARN` |
| `codeowners_errors` | `GET /codeowners/errors` | `404`, missing CODEOWNERS, unavailable, or inaccessible | `MANUAL` |
| `immutable_releases` | `GET /immutable-releases` | `enabled=true` | `PASS` |
| `immutable_releases` | `GET /immutable-releases` | `enabled=false` or documented not-enabled response | `WARN` if releases are used; otherwise `MANUAL` |
| `immutable_releases` | `GET /immutable-releases` | unavailable or inaccessible | limitation |
| `deployment_environments` | `GET /environments` | zero environments | `MANUAL` |
| `deployment_environments` | `GET /environments` | one or more environments returned | `PASS`; inventory protection rule types and branch policy only |
| `actions_secrets_inventory` | `GET /actions/secrets` | repository secrets metadata returned | `PASS`; inventory names/counts/updated timestamps only, never values |
| `actions_secrets_inventory` | `GET /actions/secrets` | unavailable or access denied | limitation; not `FAIL` |
| `actions_organization_secrets_inventory` | `GET /actions/organization-secrets` | organization secrets visible to repository returned | `PASS`; inventory names/counts/updated timestamps only, never values |
| `environment_secrets_inventory` | `GET /environments/{environment}/secrets` | environment secrets metadata returned | `PASS`; inventory names/counts/updated timestamps only, never values |
| `oidc_subject_claim` | `GET /actions/oidc/customization/sub` | template returned | `PASS`; inventory only |
| `oidc_subject_claim` | manual cloud/provider trust policy | GitHub-side template alone cannot prove cloud trust constraints | `MANUAL` |
| `self_hosted_runners` | `GET /actions/runners` | zero runners | `PASS` |
| `self_hosted_runners` | `GET /actions/runners` | one or more runners | `WARN`; review runner exposure, labels, persistence, and network trust |
| `releases_inventory` | `GET /releases` | release count and immutable fields returned | `PASS`; inventory only |
| `releases_inventory` | `GET /releases` | unavailable | limitation |
| `dependabot_open_alerts` | `GET /dependabot/alerts?state=open` | open count `0` | `PASS` |
| `dependabot_open_alerts` | `GET /dependabot/alerts?state=open` | open count `1+` | `WARN` |
| `secret_scanning_open_alerts` | `GET /secret-scanning/alerts?state=open` | open count `0` | `PASS` |
| `secret_scanning_open_alerts` | `GET /secret-scanning/alerts?state=open` | open count `1+` | `WARN`; do not expose secret values or locations |
| `code_scanning_open_alerts` | `GET /code-scanning/alerts?state=open` | open count `0` | `PASS` |
| `code_scanning_open_alerts` | `GET /code-scanning/alerts?state=open` | open count `1+` | `WARN` |
| `artifact_attestations` | `GET /attestations/{subject_digest}` | digest supplied and attestations returned | `PASS`; inventory only, cryptographic verification remains separate |
| `artifact_attestations` | v0.1 remote API scope | no subject digest supplied | `MANUAL` |
| `sbom_inventory` | GitHub Dependency Graph SBOM APIs | SBOM export not collected by default v0.1 scope | `MANUAL` |
| `dependency_review` | dependency review API or workflow/action inspection | no base/head comparison or workflow inspection supplied | `MANUAL` |
| `workflow_file_security` | local clone or Contents API inspection | workflow-level permissions, pinned action refs, untrusted input handling, zizmor, or Scorecard not inspected | `SKIP` |
| `repository_access_surface` | collaborators, teams, deploy keys, webhooks, secrets, variables | not collected by default v0.1 because it can expose sensitive operational metadata | `MANUAL` |
| any endpoint | any source | unavailable due to permissions, disabled feature, plan, unsupported repo, or no analysis | top-level or finding-level limitation; not `FAIL` |

## Endpoint Limitations

| Endpoint | Success handling | Limitation handling |
| --- | --- | --- |
| `GET /repos/{owner}/{repo}` | Build `subject`; use `security_and_analysis` when present. | `404` is repo missing or no access. Treat audit as blocked. |
| `GET /code-security-configuration` | Create security configuration finding when attached. | `204`, `403`, or `404` becomes unattached/unavailable context, not a failure. |
| `GET /community/profile` | Record community health context only. | The response at API version 2026-03-10 contains no security policy field; never use it as security policy evidence. |
| `GET /contents/SECURITY.md` (root, `.github/`, `docs/`) | Record which probe paths returned `200` for the requested ref. | `404` means absent at that path; other non-2xx is an access or availability limitation. Do not output file contents unless the user asks. |
| `GET /actions/permissions` | Create Actions repository permissions finding. | `403`/`404` becomes an Actions permissions limitation. |
| `GET /actions/permissions/selected-actions` | Create selected actions inventory when applicable. | Skip if repository policy is not `selected`; limitation if access denied. |
| `GET /actions/permissions/workflow` | Create workflow token permissions finding. | `403`/`404` becomes a workflow permissions limitation. |
| `GET /actions/permissions/access` | Inventory private-repo component sharing access. | Public repos or unavailable endpoint should be `SKIP` or limitation. |
| `GET /rulesets?includes_parents=true&targets=branch` | Inventory branch-targeting repository and parent rulesets. | Never treat count alone as default-branch protection. |
| `GET /rules/branches/{branch}` | Inventory active rules that apply to the default branch, including inherited rules. | Empty result means no active rulesets applied; still check legacy branch protection. |
| `GET /branches/{branch}` | Inventory the branch summary, including `protected` when returned. | Summary is not enough to understand detailed branch protection requirements. |
| `GET /branches/{branch}/protection` | Inventory legacy branch protection. | `404` can mean no legacy protection or no access; when `repo.json` reports `permissions.admin=true`, treat `404` as confirmed absence. |
| `GET /code-scanning/default-setup` | Create CodeQL default setup finding. | `403`/`404` may mean no Advanced Security, unsupported repo, no access, or no analysis; record limitation. |
| `GET /contents/.github/dependabot.yml?ref={branch}` | Record whether Dependabot version-update configuration exists. | Do not output full config unless the user asks; absence is not a security-update verdict. |
| `GET /automated-security-fixes` | Create Dependabot security updates finding. | `403`/`404` becomes limitation. |
| `GET /vulnerability-alerts` | `204` means enabled. | `404` can mean disabled or unavailable; use response message and `GET /repos` evidence when possible. |
| `GET /private-vulnerability-reporting` | Record the returned enablement state. | Non-2xx can mean unavailable, inaccessible, or inapplicable; record as `SKIP` or limitation unless usable response data proves disabled on an applicable repo. |
| `GET /codeowners/errors?ref={branch}` | Count syntax errors only for the requested ref. | Missing CODEOWNERS or inaccessible contents should be limitation or `MANUAL`; do not report missing CODEOWNERS as zero syntax errors. |
| `GET /immutable-releases` | Record `enabled` and `enforced_by_owner` when returned. | Do not infer release process risk without knowing whether releases are used. |
| `GET /dependabot/alerts?state=open` | Use fully paginated open alert count. | `403` disabled/no access is a limitation, not a settings finding. |
| `GET /secret-scanning/alerts?state=open` | Use fully paginated open alert count only. | `403`/`404` is a limitation. Do not output secrets or locations. |
| `GET /code-scanning/alerts?state=open` | Use fully paginated open alert count. | `404` no analysis found is a limitation or `SKIP`, not a settings failure. |
| `GET /environments` | Inventory names, protection rule types, and deployment branch policy. | Do not output secret names or values; missing environments may be normal. |
| `GET /actions/secrets` | Inventory repository secret names and metadata only. | Secret presence/absence is not a policy verdict; never output values. |
| `GET /actions/organization-secrets` | Inventory organization secret names visible to the repository and metadata only. | Availability depends on owner type and permissions; never output values. |
| `GET /environments/{environment}/secrets` | Inventory environment secret names and metadata only. | Environment secret inventory is per environment; never output values. |
| `GET /actions/oidc/customization/sub` | Inventory GitHub-side subject claim template. | Cloud provider trust policy is not visible via GitHub API. |
| `GET /actions/runners` | Inventory self-hosted runner count, status, OS, labels, and ephemeral flag. | Runner security posture requires human review. |
| `GET /releases` | Inventory releases and immutable fields when returned. | Release immutability setting is separate from whether release artifacts are trustworthy. |
| `GET /attestations/{subject_digest}` | Inventory attestations for a provided digest. | No digest means the check is manual; API listing is not a repo-wide attestation coverage proof. |

## Important Separation Rules

- Settings findings describe whether a feature or policy is configured.
- Alert findings describe detected open alerts.
- Do not use an alert endpoint failure as proof that the corresponding setting
  is disabled.
- Open alert count `0` means the endpoint returned zero open alerts for the
  token, feature state, and query used; it is not proof that vulnerabilities or
  secrets do not exist.
- Do not infer "default branch is protected" from a non-empty rulesets response.
  Use active default-branch rules and legacy branch protection evidence.
- Do not infer "default branch is unprotected" from an empty rulesets response
  alone. Also report active default-branch rules and legacy branch protection.
- Do not infer "code scanning is absent" from CodeQL default setup alone.
  Advanced setup, third-party SARIF upload, or other code scanning tools require
  separate evidence.
- Repository default `GITHUB_TOKEN` permissions do not prove workflow-level
  `permissions:` blocks are safe. Workflow file diagnostics are out of scope for
  v0.1.
- OIDC subject customization does not prove cloud provider trust policy is
  constrained. Treat cloud trust as manual.
- Deployment environment inventory does not prove all deployment workflows use
  those environments.
- Artifact attestation lookup by digest does not prove all released artifacts
  have provenance.
- Do not include secret scanning secret values, raw secret fragments, file paths,
  line numbers, or location details in v0.1 output.
- Do not describe `PASS` as "safe" or "secure"; it only means the observed state
  did not match a deterministic review flag in this inventory model.
