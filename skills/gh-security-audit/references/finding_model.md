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
- `MANUAL`: Human judgment or non-API evidence is required, such as legacy branch
  protection when v0.1 only checked rulesets.
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
    "api_version": "2022-11-28",
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
| `actions_repository_permissions` | `GET /actions/permissions` | Actions enabled with restricted/selected actions and SHA pinning required | `PASS` |
| `actions_repository_permissions` | `GET /actions/permissions` | `allowed_actions=all` or `sha_pinning_required=false` | `WARN` |
| `actions_workflow_token_permissions` | `GET /actions/permissions/workflow` | `default_workflow_permissions=read` | `PASS` |
| `actions_workflow_token_permissions` | `GET /actions/permissions/workflow` | `default_workflow_permissions=write` | `WARN` |
| `repository_rulesets` | `GET /rulesets` | one or more rulesets returned | `PASS`, plus describe targets if available |
| `repository_rulesets` | `GET /rulesets` | empty array | `WARN`, meaning review branch protection separately |
| `repository_rulesets` | v0.1 remote API scope | legacy branch protection not checked | `MANUAL` if the user needs a protection verdict |
| `codeql_default_setup` | `GET /code-scanning/default-setup` | `state=configured` | `PASS` |
| `codeql_default_setup` | `GET /code-scanning/default-setup` | `state` other than `configured` | `WARN` |
| `dependabot_security_updates` | `GET /automated-security-fixes` and `GET /repos` | enabled | `PASS` |
| `dependabot_security_updates` | `GET /automated-security-fixes` and `GET /repos` | disabled | `WARN` |
| `vulnerability_alerts` | `GET /vulnerability-alerts` and `GET /repos` | `204` or enabled status | `PASS` |
| `vulnerability_alerts` | `GET /vulnerability-alerts` and `GET /repos` | disabled response | `WARN` |
| `secret_scanning` | `GET /repos` `security_and_analysis` | enabled | `PASS` |
| `secret_scanning` | `GET /repos` `security_and_analysis` | disabled | `WARN` |
| `secret_scanning_push_protection` | `GET /repos` `security_and_analysis` | enabled | `PASS` |
| `secret_scanning_push_protection` | `GET /repos` `security_and_analysis` | disabled | `WARN` |
| `dependabot_open_alerts` | `GET /dependabot/alerts?state=open` | open count `0` | `PASS` |
| `dependabot_open_alerts` | `GET /dependabot/alerts?state=open` | open count `1+` | `WARN` |
| `secret_scanning_open_alerts` | `GET /secret-scanning/alerts?state=open` | open count `0` | `PASS` |
| `secret_scanning_open_alerts` | `GET /secret-scanning/alerts?state=open` | open count `1+` | `WARN`; do not expose secret values or locations |
| `code_scanning_open_alerts` | `GET /code-scanning/alerts?state=open` | open count `0` | `PASS` |
| `code_scanning_open_alerts` | `GET /code-scanning/alerts?state=open` | open count `1+` | `WARN` |
| any endpoint | any source | unavailable due to permissions, disabled feature, plan, unsupported repo, or no analysis | top-level or finding-level limitation; not `FAIL` |

## Endpoint Limitations

| Endpoint | Success handling | Limitation handling |
| --- | --- | --- |
| `GET /repos/{owner}/{repo}` | Build `subject`; use `security_and_analysis` when present. | `404` is repo missing or no access. Treat audit as blocked. |
| `GET /actions/permissions` | Create Actions repository permissions finding. | `403`/`404` becomes an Actions permissions limitation. |
| `GET /actions/permissions/workflow` | Create workflow token permissions finding. | `403`/`404` becomes a workflow permissions limitation. |
| `GET /rulesets` | Create ruleset finding from returned array. | `403`/`404` becomes a rulesets limitation. |
| `GET /code-scanning/default-setup` | Create CodeQL default setup finding. | `403`/`404` may mean no Advanced Security, unsupported repo, no access, or no analysis; record limitation. |
| `GET /automated-security-fixes` | Create Dependabot security updates finding. | `403`/`404` becomes limitation. |
| `GET /vulnerability-alerts` | `204` means enabled. | `404` can mean disabled or unavailable; use response message and `GET /repos` evidence when possible. |
| `GET /dependabot/alerts?state=open` | Use open alert count. | `403` disabled/no access is a limitation, not a settings finding. |
| `GET /secret-scanning/alerts?state=open` | Use open alert count only. | `403`/`404` is a limitation. Do not output secrets or locations. |
| `GET /code-scanning/alerts?state=open` | Use open alert count. | `404` no analysis found is a limitation or `SKIP`, not a settings failure. |

## Important Separation Rules

- Settings findings describe whether a feature or policy is configured.
- Alert findings describe detected open alerts.
- Do not use an alert endpoint failure as proof that the corresponding setting
  is disabled.
- Do not infer "default branch is unprotected" from an empty rulesets response;
  report "no rulesets returned" and add `MANUAL` for legacy branch protection if
  the user needs a protection verdict.
- Do not include secret scanning secret values, raw secret fragments, file paths,
  line numbers, or location details in v0.1 output.
- Do not describe `PASS` as "safe" or "secure"; it only means the observed state
  did not match a deterministic review flag in this inventory model.
