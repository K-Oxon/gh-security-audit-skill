# GitHub API Sources

Use primary sources when explaining endpoint behavior, required permissions, or
HTTP status meanings. GitHub API behavior can change, so verify uncertain facts
against these docs before making strong claims.

## REST API Version

Use:

```text
X-GitHub-Api-Version: 2022-11-28
```

## Official Documentation

- GitHub REST API overview:
  <https://docs.github.com/en/rest?apiVersion=2022-11-28>
- Repositories REST API:
  <https://docs.github.com/en/rest/repos/repos?apiVersion=2022-11-28>
- GitHub Actions permissions REST API:
  <https://docs.github.com/en/rest/actions/permissions?apiVersion=2022-11-28>
- Repository rulesets REST API:
  <https://docs.github.com/en/rest/repos/rules?apiVersion=2022-11-28>
- Dependabot alerts REST API:
  <https://docs.github.com/en/rest/dependabot/alerts?apiVersion=2022-11-28>
- Secret scanning REST API:
  <https://docs.github.com/en/rest/secret-scanning/secret-scanning?apiVersion=2022-11-28>
- Code scanning REST API:
  <https://docs.github.com/en/rest/code-scanning/code-scanning?apiVersion=2022-11-28>
- Agent Skills specification:
  <https://agentskills.io/specification>
- GitHub CLI `gh skill` manual:
  <https://cli.github.com/manual/gh_skill>
- GitHub CLI `gh skill install` manual:
  <https://cli.github.com/manual/gh_skill_install>
- GitHub CLI `gh skill publish` manual:
  <https://cli.github.com/manual/gh_skill_publish>

## v0.1 Source Policy

- Prefer `gh api` over language-specific scripts.
- Treat GitHub API responses and official docs as primary evidence.
- Treat repository-specific failures as limitations unless a successful response
  provides clear contrary evidence.
- If docs and live API behavior appear to conflict, report the observed behavior
  and link the relevant official source instead of guessing.
