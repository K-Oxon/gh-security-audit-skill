# GitHub Repository Security Audit: K-Oxon/gh-security-audit-skill

## Summary

This sample shows the v0.1 remote-api output format for
`K-Oxon/gh-security-audit-skill`. It is an example, not a current audit result.

Overall status: `WARN`

Key points:

- Basic repository metadata was available.
- Default `GITHUB_TOKEN` workflow permissions were read-only.
- Repository rulesets were not configured in the sample observation.
- CodeQL default setup was not configured in the sample observation.
- Secret scanning and push protection were enabled in repository metadata.
- Some alert endpoints were unavailable because features were disabled,
  unavailable, or had no analysis.

## Scope and Assumptions

- Mode: remote-api
- Input: `OWNER=K-Oxon`, `REPO=gh-security-audit-skill`
- API version: `2022-11-28`
- Source: read-only `gh api` commands
- Out of scope for v0.1: workflow file diagnostics, zizmor, Scorecard, local
  clone inspection, and automatic remediation

## Overall Status

`WARN`

The sample finding set contains no `FAIL` findings. Several recommended
repository security settings were either not configured or could not be
verified through the available API responses.

## High-Priority Findings

No high-priority `FAIL` findings are present in this sample. Secret scanning
alert details are intentionally omitted.

## All Findings

| ID | Status | Severity | Summary |
| --- | --- | --- | --- |
| `repo_metadata` | `PASS` | `info` | Repository metadata was retrieved and the repository was not archived. |
| `actions_repository_permissions` | `WARN` | `medium` | Actions were enabled, but repository-level restrictions should be reviewed. |
| `actions_workflow_token_permissions` | `PASS` | `medium` | Default workflow token permission was `read`. |
| `repository_rulesets` | `WARN` | `medium` | No repository rulesets were returned. Legacy branch protection was not checked in v0.1. |
| `codeql_default_setup` | `WARN` | `medium` | CodeQL default setup state was `not-configured`. |
| `dependabot_security_updates` | `WARN` | `medium` | Dependabot security updates were disabled in the sample observation. |
| `vulnerability_alerts` | `WARN` | `medium` | Vulnerability alerts were disabled or unavailable in the sample observation. |
| `secret_scanning` | `PASS` | `medium` | Secret scanning was enabled in repository metadata. |
| `secret_scanning_push_protection` | `PASS` | `medium` | Secret scanning push protection was enabled in repository metadata. |
| `dependabot_open_alerts` | `SKIP` | `medium` | Dependabot alert listing was unavailable. |
| `secret_scanning_open_alerts` | `PASS` | `high` | Open secret scanning alert count was zero in the sample observation. |
| `code_scanning_open_alerts` | `SKIP` | `medium` | Code scanning alerts were unavailable because no analysis was found. |

## Manual Checks

- Confirm whether legacy branch protection is used instead of repository
  rulesets.
- Confirm whether organization policy intentionally manages Actions restrictions.
- Confirm whether CodeQL is intentionally disabled for this repository.
- Confirm whether Dependabot vulnerability alerts are intentionally disabled.

## Data Access and Limitations

- `GET /repos/{owner}/{repo}/dependabot/alerts?state=open` returned a
  feature-disabled or inaccessible response in the sample observation.
- `GET /repos/{owner}/{repo}/code-scanning/alerts?state=open` returned a
  no-analysis response in the sample observation.
- `GET /repos/{owner}/{repo}/vulnerability-alerts` can use HTTP status as
  evidence; a `404` response must be interpreted with endpoint context.
- v0.1 does not inspect workflow files or local repository contents.

## Recommended Next Actions

1. Add or review rulesets for the default branch.
2. Decide whether CodeQL default setup should be enabled.
3. Enable or verify Dependabot vulnerability alerts and security updates.
4. Re-run the remote-api recipe after settings change.
