# GitHub API Sources

Use primary sources when explaining endpoint behavior, required permissions, or
HTTP status meanings. GitHub API behavior can change, so verify uncertain facts
against these docs before making strong claims.

## REST API Version

Use:

```text
X-GitHub-Api-Version: 2026-03-10
```

## Official Documentation

- GitHub REST API overview:
  <https://docs.github.com/en/rest?apiVersion=2026-03-10>
- REST API versioning:
  <https://docs.github.com/en/rest/overview/api-versions>
- Repositories REST API:
  <https://docs.github.com/en/rest/repos/repos?apiVersion=2026-03-10>
- Repository contents REST API:
  <https://docs.github.com/en/rest/repos/contents?apiVersion=2026-03-10>
- Community metrics REST API:
  <https://docs.github.com/en/rest/metrics/community?apiVersion=2026-03-10>
- Branches REST API:
  <https://docs.github.com/en/rest/branches/branches?apiVersion=2026-03-10>
- Protected branches REST API:
  <https://docs.github.com/en/rest/branches/branch-protection?apiVersion=2026-03-10>
- GitHub Actions permissions REST API:
  <https://docs.github.com/en/rest/actions/permissions?apiVersion=2026-03-10>
- GitHub Actions secrets REST API:
  <https://docs.github.com/en/rest/actions/secrets?apiVersion=2026-03-10>
- GitHub Actions OIDC REST API:
  <https://docs.github.com/en/rest/actions/oidc?apiVersion=2026-03-10>
- GitHub Actions self-hosted runners REST API:
  <https://docs.github.com/en/rest/actions/self-hosted-runners?apiVersion=2026-03-10>
- Repository rulesets REST API:
  <https://docs.github.com/en/rest/repos/rules?apiVersion=2026-03-10>
- Code security configurations REST API:
  <https://docs.github.com/en/rest/code-security/configurations?apiVersion=2026-03-10>
- Dependabot alerts REST API:
  <https://docs.github.com/en/rest/dependabot/alerts?apiVersion=2026-03-10>
- Dependency graph REST API:
  <https://docs.github.com/en/rest/dependency-graph?apiVersion=2026-03-10>
- Dependency Graph SBOM REST API:
  <https://docs.github.com/en/rest/dependency-graph/sboms?apiVersion=2026-03-10>
- Automatic dependency submission:
  <https://docs.github.com/en/code-security/how-tos/secure-your-supply-chain/secure-your-dependencies/submit-dependencies-automatically>
- Secret scanning REST API:
  <https://docs.github.com/en/rest/secret-scanning/secret-scanning?apiVersion=2026-03-10>
- Code scanning REST API:
  <https://docs.github.com/en/rest/code-scanning/code-scanning?apiVersion=2026-03-10>
- Deployment environments REST API:
  <https://docs.github.com/en/rest/deployments/environments?apiVersion=2026-03-10>
- Repository attestations REST API:
  <https://docs.github.com/en/rest/repos/attestations?apiVersion=2026-03-10>
- Releases REST API:
  <https://docs.github.com/en/rest/releases/releases?apiVersion=2026-03-10>
- GitHub security configurations:
  <https://docs.github.com/en/code-security/concepts/security-at-scale/security-configurations>
- GitHub security features overview:
  <https://docs.github.com/en/code-security/getting-started/github-security-features>
- GitHub Actions OIDC hardening:
  <https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect>
- Artifact attestations guide:
  <https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds>

## v0.1 Source Policy

- Prefer `gh api` over language-specific scripts.
- Treat GitHub API responses and official docs as primary evidence.
- Treat repository-specific failures as limitations unless a successful response
  provides clear contrary evidence.
- If docs and live API behavior appear to conflict, report the observed behavior
  and link the relevant official source instead of guessing.
