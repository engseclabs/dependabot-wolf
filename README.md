# Dependabot Wolf 🐺

Automated GitHub Action that fixes stuck Dependabot security alerts using GitHub Copilot Workspace.

## What It Does

When Dependabot can't create a PR to fix a security vulnerability (usually due to dependency conflicts), Dependabot Wolf:

1. Finds all open Dependabot alerts without PRs
2. Creates a draft PR for each stuck alert
3. Tags GitHub Copilot Workspace to propose a fix
4. Engineers review and merge (or close) the Copilot-generated fixes

## How It Works

```mermaid
graph LR
    A[Daily Cron] --> B[Scan Dependabot Alerts]
    B --> C{Has PR?}
    C -->|No| D[Create Draft PR]
    C -->|Yes| E[Skip]
    D --> F[@github-copilot workspace]
    F --> G[Engineer Reviews]
```

## Installation

1. Enable Dependabot alerts on your repository (Settings → Security → Dependabot)
2. Copy `.github/workflows/dependabot-wolf.yml` to your repo
3. Ensure Dependabot has appropriate permissions
4. The workflow runs daily or manually via `workflow_dispatch`

## Configuration

The workflow requires the following permissions:
- `contents: write` - To create branches
- `pull-requests: write` - To create PRs
- `security-events: read` - To read Dependabot alerts

These are configured in the workflow file.

## Testing

This repo includes intentionally vulnerable dependencies to test the workflow:
- `lodash@4.17.15` (CVE-2020-8203)
- `minimist@0.0.8` (CVE-2020-7598)
- `axios@0.18.0` (CVE-2019-10742)

## License

MIT
