# gh-security-audit-skill

[日本語](./README-ja.md)

A read-only Agent Skill that audits the security settings of a single
GitHub repository through `gh api`.

## What it does

Give it `OWNER/REPO`. The skill runs a fixed set of read-only REST API
calls and produces a Markdown report and JSON findings from the same
data: observed settings, alert counts, and what could not be verified.

The output is an inventory of current state, not advice. Statuses
(`PASS` / `WARN` / `MANUAL` / `SKIP`) come from a deterministic table
inside the skill, and `PASS` only means nothing was flagged for review.
Whether the state is acceptable, and what to change, is your call —
read the report against your own requirements and whatever counts as
best practice at the time. `FAIL` is never used unless you supply a
policy profile that defines failure conditions.

## What it checks

- repository metadata, `security_and_analysis`, merge/forking settings
- Actions: repository permissions, workflow token defaults, fork PR
  approval policy, secrets and variables metadata (names only), OIDC
  subject claim, self-hosted runners
- branch protection: branch/tag/push rulesets, active default-branch
  rules, legacy branch protection
- code security configuration, CodeQL default setup
- Dependabot: config file, security updates, vulnerability alerts,
  open alert counts
- secret scanning settings and open alert counts
- SECURITY.md presence, CODEOWNERS errors, private vulnerability
  reporting
- environments, releases, immutable releases, artifact attestations
  (a subject digest is required)

Out of scope in v0.1: workflow file contents (zizmor, Scorecard),
local clones, cloud provider trust policies, SBOM contents, and any
remediation. The skill never changes repository settings.

## Prerequisites

- `gh` installed and authenticated, with read access to the target
  repository. Admin or security scopes unlock a few more checks;
  anything out of reach is recorded as a limitation, not a failure.
- `jq`
- A POSIX shell. On Windows, use WSL or Git Bash.

## Install

`gh skill` is currently a GitHub CLI preview feature.

```sh
gh skill install K-Oxon/gh-security-audit-skill gh-security-audit --agent claude-code --scope user
gh skill install K-Oxon/gh-security-audit-skill gh-security-audit --agent codex --scope user
```

From a local checkout:

```sh
gh skill install . gh-security-audit --from-local --agent claude-code --scope project
```

## Usage

Claude Code:

```text
> Audit the security settings of OWNER/REPO with gh-security-audit
```

Codex:

```sh
codex "Use gh-security-audit to audit OWNER/REPO. Output Markdown and JSON findings."
```

The agent reads `references/command_recipe.md`, runs the read-only
`gh api` commands, writes raw output to a temporary directory, and
assembles the report using `references/finding_model.md`. Sample output
is in `skills/gh-security-audit/examples/`.

## Reading the report

- `PASS`: evidence was retrieved and nothing matched a review flag.
  Not a safety verdict.
- `WARN`: the observed state matched a deterministic review flag.
  Check it against your own policy.
- `MANUAL`: needs human judgment or evidence the API cannot provide.
- `SKIP`: out of scope or not applicable.

API errors, missing permissions, and plan differences are reported as
limitations. Secret values, alert locations, and Actions variable
values never appear in the output.

## Layout

```text
skills/gh-security-audit/
├── SKILL.md
├── references/   # command recipe, finding model, API sources
└── examples/     # sample report and findings
```

License: Apache-2.0
