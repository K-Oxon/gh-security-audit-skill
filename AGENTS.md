# Repository Instructions

## Tooling

- Run `mise ls` before starting work.
- Except for Python via `uv`, run mise-managed tools through `mise exec --`.
- Use `gh api` as the primary GitHub data source for this repository's audit skill.

## gh-security-audit

- The canonical skill source is `skills/gh-security-audit/SKILL.md`.
- Do not treat `.agents/skills` or `.claude/skills` as canonical source.
- For this repository's own audit, use the read-only endpoints listed in
  `skills/gh-security-audit/references/command_recipe.md`.
- Do not use `gh api -X`, `gh api --method`, GraphQL mutations, repository
  setting changes, alert dismissal, or remediation commands for v0.1 audits.
- Store raw audit output in a temporary directory, not in the tracked source tree.
- API access failures, feature-disabled responses, permission gaps, and plan
  differences are limitations, not `FAIL` findings.
