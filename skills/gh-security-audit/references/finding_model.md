# Finding Model

v0.1 reports what could be verified from GitHub APIs and what could not be
verified. A missing or inaccessible API response is not a security failure by
itself.

## Status Values

- `PASS`: Retrieved evidence satisfies the expected state in the policy context.
- `WARN`: Retrieved evidence is outside the recommended state or leaves
  material risk, but is not classified as a clear failure.
- `FAIL`: Retrieved evidence does not satisfy the expected state in the policy
  context. Permission gaps and collection failures are not `FAIL`.
- `MANUAL`: Human judgment or non-API evidence is required.
- `SKIP`: The check is out of scope, unsupported, or a prerequisite is missing.
  `SKIP` does not mean safe.

## Severity Values

- `critical`: Immediately exploitable exposure or major defense failure.
- `high`: Important defense disabled or broad impact.
- `medium`: Recommended defense missing or operational risk.
- `low`: Improvement or limited impact.
- `info`: Contextual audit fact.

Keep severity separate from status. A `WARN` can still deserve prompt action,
and a `FAIL` can be low severity depending on the repository.

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
- `expected`
- `evidence`
- `recommendation`
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
    "id": "v0.1-runtime-default",
    "api_version": "2022-11-28",
    "generated_at": "YYYY-MM-DDTHH:MM:SSZ",
    "sources": []
  },
  "findings": [],
  "limitations": []
}
```

Markdown and JSON outputs must be generated from the same finding set. Do not
change statuses, counts, or recommendations between the two formats.

## Minimal Finding Set

| ID | Source | Expected direction |
| --- | --- | --- |
| `repo_metadata` | `GET /repos/{owner}/{repo}` | Repository is not archived and default branch is known. |
| `actions_repository_permissions` | `GET /actions/permissions` | Actions are enabled. Unrestricted allowed actions or disabled SHA pinning are usually `WARN`. |
| `actions_workflow_token_permissions` | `GET /actions/permissions/workflow` | `default_workflow_permissions=read` is `PASS`; `write` is `WARN` or `FAIL` depending on policy context. |
| `repository_rulesets` | `GET /rulesets` | A ruleset protects the default branch. Empty rulesets are `WARN`. Legacy branch protection is `MANUAL` in v0.1. |
| `codeql_default_setup` | `GET /code-scanning/default-setup` | `state=configured` is `PASS`; `not-configured` is `WARN`. |
| `dependabot_security_updates` | `GET /automated-security-fixes` and `GET /repos` | Enabled is `PASS`; disabled is `WARN`. |
| `vulnerability_alerts` | `GET /vulnerability-alerts` and `GET /repos` | `204` or enabled status is `PASS`; disabled is `WARN`; inaccessible is a limitation. |
| `secret_scanning` | `GET /repos` `security_and_analysis` | Enabled is `PASS`; disabled is `WARN`; absent data is a limitation. |
| `secret_scanning_push_protection` | `GET /repos` `security_and_analysis` | Enabled is `PASS`; disabled is `WARN`; absent data is a limitation. |
| `dependabot_open_alerts` | `GET /dependabot/alerts?state=open` | Open count `0` is `PASS`; `1+` is `WARN`; unavailable is a limitation. |
| `secret_scanning_open_alerts` | `GET /secret-scanning/alerts?state=open` | Open count `0` is `PASS`; `1+` is `FAIL`; unavailable is a limitation. |
| `code_scanning_open_alerts` | `GET /code-scanning/alerts?state=open` | Open count `0` is `PASS`; `1+` is `WARN`; no analysis or unavailable is a limitation or `SKIP`. |

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
- Do not include secret scanning secret values, raw secret fragments, file paths,
  line numbers, or location details in v0.1 output.
- Do not describe `PASS` as "safe" or "secure"; it only means the expected
  evidence was observed.
