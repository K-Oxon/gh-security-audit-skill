# gh-security-audit-skill

`gh-security-audit` is a read-only Agent Skill for auditing GitHub repository
security settings with `gh api`.

v0.1 is intentionally small: it gives Codex, Claude Code, and other
Agent-Skills-compatible tools a reproducible remote API recipe for one
`OWNER/REPO`, plus an inventory-first finding model and output contract. It does
not modify repository settings.

## What It Checks

The v0.1 remote-api audit focuses on GitHub repository settings and alert state
available through the GitHub REST API:

- repository metadata and `security_and_analysis`
- GitHub Actions repository permissions
- default `GITHUB_TOKEN` workflow permissions
- repository rulesets
- CodeQL default setup
- Dependabot security updates and vulnerability alerts
- open Dependabot, secret scanning, and code scanning alert counts

Workflow file static analysis, zizmor, Scorecard, local clone inspection, and
automatic remediation are out of scope for v0.1.

## Install

`gh skill` is the primary install path. It is currently a GitHub CLI preview
feature.

Install for Codex at user scope:

```sh
gh skill install K-Oxon/gh-security-audit-skill gh-security-audit --agent codex --scope user
```

Install for Claude Code at user scope:

```sh
gh skill install K-Oxon/gh-security-audit-skill gh-security-audit --agent claude-code --scope user
```

Local checkout validation install:

```sh
gh skill install . gh-security-audit --from-local --agent codex --scope project
gh skill install . gh-security-audit --from-local --agent claude-code --scope project
```

Publishing validation:

```sh
gh skill publish --dry-run
```

## Use

Ask your agent to use the `gh-security-audit` skill for a repository:

```text
Use gh-security-audit for OWNER/REPO and produce Markdown plus JSON findings.
```

The skill will read `skills/gh-security-audit/references/command_recipe.md`,
run read-only `gh api` commands, and produce output using the shared finding
model in `skills/gh-security-audit/references/finding_model.md`.

Raw API output should be written to a temporary directory, not committed to this
repository.

## Constraints

- Requires an authenticated `gh` CLI and repository read access.
- API access failures, permission gaps, feature-disabled responses, and plan
  differences are reported as limitations, not as `FAIL`.
- Alert findings and settings findings are separate.
- Secret scanning output must not include secret values or location details.
- Markdown and JSON should be generated from the same finding set.
- Default v0.1 output reports observed state first. Review flags are assigned
  only from the deterministic table in `finding_model.md`.

## Repository Layout

The canonical skill source is:

```text
skills/gh-security-audit/
├── SKILL.md
├── references/
│   ├── command_recipe.md
│   ├── finding_model.md
│   └── github_api_sources.md
└── examples/
    ├── sample_findings.json
    └── sample_report.md
```

Agent-specific install targets such as `.agents/skills` and `.claude/skills`
are generated destinations, not canonical source.
