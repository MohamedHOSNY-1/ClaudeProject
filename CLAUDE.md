# CLAUDE.md

This file provides guidance for AI assistants (Claude and others) working in this repository.

## Repository Overview

This is a GitHub Actions workflow template repository that integrates Anthropic's Claude Code AI into GitHub issue and pull request workflows. It contains no application source code — only CI/CD configuration.

## Repository Structure

```
ClaudeProject/
├── .github/
│   └── workflows/
│       ├── claude.yml               # On-demand Claude assistant (triggered by @claude mentions)
│       └── claude-code-review.yml   # Automatic code review on all PRs
└── README.md
```

## Workflows

### 1. `claude.yml` — On-Demand Claude Assistant

**Purpose**: Allows contributors to invoke Claude directly from GitHub by mentioning `@claude` in comments or issues.

**Triggers**:
- Issue comments containing `@claude`
- Pull request review comments containing `@claude`
- Pull request reviews containing `@claude`
- Issues opened or assigned where the title or body contains `@claude`

**Permissions** (read-only, principle of least privilege):
- `contents: read`
- `pull-requests: read`
- `issues: read`
- `id-token: write`
- `actions: read` (to read CI results on PRs)

**Key configuration**:
- Uses `anthropics/claude-code-action@v1`
- Authenticates via the `CLAUDE_CODE_OAUTH_TOKEN` repository secret
- Performs shallow checkout (`fetch-depth: 1`)

**Optional/commented-out features**:
- Custom `prompt:` override (instead of following the comment instructions)
- `claude_args:` to restrict allowed tools (e.g., `--allowed-tools Bash(gh pr:*)`)

### 2. `claude-code-review.yml` — Automatic Code Review

**Purpose**: Automatically runs a structured code review on every pull request using the Claude Code Review plugin.

**Triggers**: All PRs on `opened`, `synchronize`, `ready_for_review`, `reopened` events.

**Permissions**: Same minimal read-only set as `claude.yml` (without `actions: read`).

**Key configuration**:
- Uses `anthropics/claude-code-action@v1`
- Loads the `code-review` plugin from the `claude-code-plugins` branch of `https://github.com/anthropics/claude-code.git`
- Runs the prompt: `/code-review:code-review <owner>/<repo>/pull/<number>`

**Optional/commented-out features**:
- `paths:` filter — restrict reviews to specific file patterns (e.g., `src/**/*.ts`)
- `if:` author filter — limit reviews to specific contributors or first-time contributors

## Required Secrets

| Secret | Purpose |
|--------|---------|
| `CLAUDE_CODE_OAUTH_TOKEN` | OAuth token used by both workflows to authenticate with the Claude Code Action |

This secret must be set in the repository's **Settings → Secrets and variables → Actions**.

## Development Conventions

### Modifying Workflows

- Edit only files under `.github/workflows/`
- Follow GitHub Actions YAML syntax; validate with a YAML linter before committing
- Preserve the existing minimal-permissions pattern — do not add write scopes without justification
- Leave optional configuration as comments rather than deleting it, so it remains discoverable

### Enabling Optional Features

To enable a commented-out feature, uncomment the relevant YAML block and adjust the value:

- **Custom Claude prompt** (`claude.yml`, line 44): uncomment and replace the example string
- **Tool restrictions** (`claude.yml`, line 49): uncomment `claude_args` and set allowed tools
- **Path filters** (`claude-code-review.yml`, lines 7-11): uncomment `paths:` under `on.pull_request`
- **Author filters** (`claude-code-review.yml`, lines 16-19): uncomment the `if:` block on the job

### Branching

- Default branch: `main`
- Feature work goes on short-lived branches; merge via PRs

### Commit Style

Use concise, imperative commit messages describing what changed:
```
Add path filter to code review workflow
Restrict claude_args to gh pr commands
Update Claude Code Action to v2
```

## Key References

- [Claude Code Action documentation](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
- [Claude Code CLI reference](https://code.claude.com/docs/en/cli-reference)
- [GitHub Actions workflow syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
