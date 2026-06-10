# GitHub Repository Security Audit: K-Oxon/gh-security-audit-skill

## Summary

This sample shows the v0.1 remote-api output format for
`K-Oxon/gh-security-audit-skill`. It is an example, not a current audit result.

Collection status: complete with limitations

Key points:

- Basic repository metadata was available.
- Default `GITHUB_TOKEN` workflow permissions were read-only.
- Branch rulesets, active default-branch rules, branch summary, and legacy
  protection were reported as separate evidence surfaces.
- CodeQL default setup was not configured in the sample observation, but this is
  not proof that all code scanning is absent.
- Dependency Graph enablement was probed via the SBOM status endpoint;
  automatic dependency submission, Dependabot config, and SBOM content coverage
  are reported separately.
- Secret scanning and push protection were enabled in repository metadata.
- Deployment, OIDC, self-hosted runner, immutable release, and attestation
  checks were inventoried or marked manual where v0.1 cannot prove coverage.

## Scope and Assumptions

- Mode: remote-api inventory
- Input: `OWNER=K-Oxon`, `REPO=gh-security-audit-skill`
- API version: `2026-03-10`
- Source: read-only `gh api` commands
- Out of scope for v0.1: workflow file diagnostics, cloud provider trust-policy
  inspection, zizmor, Scorecard, local clone inspection, and automatic
  remediation

## Collection Status

Complete with limitations.

The sample finding set uses deterministic review labels. It is an inventory of
observed repository state, not a compliance verdict.

## Observed Security Settings

Secret scanning alert details and secret values are intentionally omitted.

| ID | Status | Severity | Observed state |
| --- | --- | --- | --- |
| `repo_metadata` | `PASS` | `info` | Repository metadata was retrieved and the repository was not archived. Merge and forking settings were recorded as inventory. |
| `repository_security_and_analysis` | `WARN` | `info` | Security and analysis fields were present; two returned statuses were explicitly `disabled`. |
| `dependency_graph` | `MANUAL` | `medium` | The SBOM status probe returned 404, which cannot distinguish disabled, unsupported, or no-manifest states. |
| `automatic_dependency_submission` | `MANUAL` | `medium` | Automatic dependency submission was not set in the sample observation. |
| `code_security_configuration` | `MANUAL` | `info` | No code security configuration was attached in the sample observation. |
| `security_policy_file` | `WARN` | `low` | All three SECURITY.md contents probes (root, `.github/`, `docs/`) returned 404. |
| `actions_repository_permissions` | `WARN` | `medium` | Actions were enabled, all actions were allowed, and SHA pinning was not required. |
| `actions_selected_actions` | `SKIP` | `info` | Selected Actions details were not applicable because `allowed_actions` was not `selected`. |
| `actions_workflow_token_permissions` | `PASS` | `medium` | Default workflow token permission was `read`; workflow files were not inspected. |
| `actions_access_private_repository` | `SKIP` | `info` | Private repository component access was not applicable for the public sample repository. |
| `actions_fork_pr_approval_policy` | `PASS` | `medium` | The fork PR contributor approval policy was `first_time_contributors`; inventory only. |
| `actions_fork_pr_private_repos` | `SKIP` | `info` | The public sample repository returned 422; this status is empirical, not documented. |
| `codeql_default_setup` | `WARN` | `medium` | CodeQL default setup state was `not-configured`. |
| `dependabot_version_updates_config` | `MANUAL` | `low` | No `.github/dependabot.yml` file was found in the sample observation. |
| `dependabot_security_updates` | `PASS` | `medium` | Dependabot security updates were enabled in the sample observation. |
| `vulnerability_alerts` | `PASS` | `medium` | Vulnerability alerts returned the enabled HTTP status in the sample observation. |
| `private_vulnerability_reporting` | `WARN` | `medium` | Private vulnerability reporting was not enabled in the sample observation. |
| `secret_scanning` | `PASS` | `medium` | Secret scanning was enabled in repository metadata. |
| `secret_scanning_push_protection` | `PASS` | `medium` | Secret scanning push protection was enabled in repository metadata. |
| `secret_scanning_validity_checks` | `WARN` | `low` | The validity checks status was explicitly `disabled` in repository metadata. |
| `secret_scanning_non_provider_patterns` | `WARN` | `low` | The non-provider patterns status was explicitly `disabled` in repository metadata. |
| `codeowners_errors` | `MANUAL` | `low` | CODEOWNERS syntax errors were not returned; missing CODEOWNERS is not zero errors. |

## Branch Protection and Rules Inventory

| ID | Status | Severity | Observed state |
| --- | --- | --- | --- |
| `branch_rulesets_inventory` | `PASS` | `info` | Zero branch-targeting rulesets were returned. This is inventory only. |
| `tag_rulesets_inventory` | `PASS` | `info` | Zero tag-targeting rulesets were returned. This is inventory only. |
| `push_rulesets_inventory` | `PASS` | `info` | Zero push rulesets were returned. This is inventory only. |
| `default_branch_active_rules` | `WARN` | `medium` | No active rules were returned for `main` in the sample observation. |
| `default_branch_summary` | `PASS` | `info` | The branch summary reported `protected=false`. |
| `default_branch_legacy_protection` | `MANUAL` | `medium` | Detailed legacy protection was not returned; no policy verdict is made from this alone. |

## Alert Availability and Counts

| ID | Status | Severity | Observed state |
| --- | --- | --- | --- |
| `dependabot_open_alerts` | `PASS` | `medium` | Fully paginated open Dependabot alert count was zero. |
| `secret_scanning_open_alerts` | `PASS` | `high` | Fully paginated open secret scanning alert count was zero. |
| `code_scanning_open_alerts` | `SKIP` | `medium` | Code scanning alerts were unavailable in the sample observation. |

## Actions, Deployment, and Runtime Inventory

| ID | Status | Severity | Observed state |
| --- | --- | --- | --- |
| `deployment_environments` | `MANUAL` | `info` | No deployment environments were returned. |
| `actions_secrets_inventory` | `PASS` | `info` | Repository Actions secrets metadata was retrievable; zero secrets were listed. |
| `actions_variables_inventory` | `PASS` | `info` | Value-redacted Actions variables metadata was retrievable; zero variables were listed. |
| `actions_organization_secrets_inventory` | `SKIP` | `info` | Organization Actions secrets were not applicable in the sample observation. |
| `environment_secrets_inventory` | `SKIP` | `info` | No environment secrets were collected because no environments were returned. |
| `oidc_subject_claim` | `MANUAL` | `medium` | GitHub-side OIDC subject customization was available, but cloud trust was not inspected. |
| `self_hosted_runners` | `PASS` | `medium` | No repository self-hosted runners were returned. |
| `immutable_releases` | `MANUAL` | `medium` | Immutable releases were reported disabled; release process was not inspected. |
| `releases_inventory` | `PASS` | `info` | No releases were returned in the sample observation. |
| `artifact_attestations` | `MANUAL` | `medium` | No subject digest was supplied, so attestation lookup was not performed. |
| `sbom_inventory` | `MANUAL` | `medium` | Dependency Graph SBOM export was not collected by default v0.1 scope. |
| `dependency_review` | `MANUAL` | `medium` | No base/head comparison or workflow inspection was supplied. |
| `workflow_file_security` | `SKIP` | `medium` | Workflow file diagnostics were outside v0.1 scope. |
| `repository_access_surface` | `MANUAL` | `medium` | Collaborators, teams, deploy keys, and webhooks were not collected by default. |

## Manual or Unavailable Checks

- Confirm effective default-branch protection using active rules, legacy
  protection, and the repository's intended policy.
- Confirm whether organization security configurations intentionally manage this
  repository.
- Confirm whether CodeQL advanced setup or third-party SARIF upload exists if
  CodeQL default setup is not configured.
- Confirm whether Dependency Graph, automatic dependency submission, Dependabot
  version updates, and dependency review match the repository's supply-chain
  policy.
- Confirm whether deployment workflows actually use protected environments.
- Confirm cloud provider trust policy if OIDC is used.
- Confirm artifact attestation coverage for released artifacts by supplying
  subject digests.
- Run Dependency Graph SBOM export explicitly if an SBOM inventory is required.
- Run workflow file diagnostics, zizmor, or Scorecard separately when workflow
  content review is needed.

## Data Access and Limitations

- This is a sample output contract, not a current audit result.
- `PASS` does not mean safe; it means the observed state did not match a
  deterministic review flag in this inventory model.
- Open alert counts depend on endpoint availability, feature state, token
  permissions, and pagination.
- No workflow files, local repository contents, cloud provider trust policies, or
  remediation actions were inspected.
- No secret values are collected or reported.

## Optional Review Notes

1. Review default-branch protections before relying on repository ruleset count.
2. Review whether CodeQL default setup, advanced setup, or another scanning path
   is intended.
3. Review Actions restrictions and SHA pinning policy.
4. Review private vulnerability reporting and immutable releases if this is a
   public project with external users or release artifacts.
