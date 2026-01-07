# Dependabot Wolf 🐺

Automated GitHub Action that fixes stuck Dependabot security alerts using GitHub Copilot Workspace.

## What It Does

When Dependabot security alerts remain unfixed, Dependabot Wolf automatically creates PR proposals using GitHub Copilot Workspace.

**"Stuck" alerts are those without open PRs:**
- Dependabot couldn't create a PR (dependency conflicts, peer dependency mismatches)
- Dependabot PR was closed (breaking changes, test failures, rejected by maintainers)

**Dependabot Wolf:**
1. Finds all open Dependabot alerts without open PRs
2. Creates a draft PR for each stuck alert with full context
3. Tags `@github-copilot workspace` to propose a fix
4. Engineers use Copilot to interactively resolve the issue

## How It Works

```mermaid
graph LR
    A[Daily Cron] --> B[Scan Dependabot Alerts]
    B --> C{Has PR?}
    C -->|No| D[Create Draft PR]
    C -->|Yes| E[Skip]
    D --> F[Tag Copilot Workspace]
    F --> G[Engineer Reviews]
```

## Installation

1. **Enable Dependabot alerts** on your repository (Settings → Security → Dependabot)

2. **Create a Personal Access Token (PAT)**:
   - Go to GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens
   - Create a token with the following permissions:
     - Repository access: Select the repository
     - Permissions:
       - `Contents`: Read and write
       - `Pull requests`: Read and write
       - `Security events`: Read only (for Dependabot alerts)
   - Copy the token

3. **Add the PAT as a repository secret**:
   - Go to your repository Settings → Secrets and variables → Actions
   - Create a new secret named `DEPENDABOT_PAT`
   - Paste your PAT as the value

4. **Copy the workflow file** `.github/workflows/dependabot-wolf.yml` to your repo

5. The workflow runs daily (via cron) or manually via `workflow_dispatch`

## Why a PAT is Required

GitHub's default `GITHUB_TOKEN` in workflows cannot access Dependabot alerts for security reasons. A Personal Access Token with `security_events` scope is required to read Dependabot alerts via the API.

## Configuration

The workflow requires the following permissions (configured via the PAT):
- `contents: write` - To create branches
- `pull-requests: write` - To create PRs
- `security-events: read` - To read Dependabot alerts

## Testing

This repo demonstrates stuck alerts with dependencies that have breaking changes:
- `react@16.8.0` - Multiple CVEs, upgrading to 17+ requires code changes
- `webpack@4.28.0` - Multiple CVEs, upgrading to 5+ is a major breaking change
- `webpack-dev-server@3.1.0` - CVEs, requires webpack 4+ peer dependency

**To test:**
1. Dependabot will create PRs for these vulnerabilities
2. Close the PRs (simulating rejection due to breaking changes)
3. Run Dependabot Wolf workflow manually
4. Wolf creates new draft PRs with Copilot Workspace integration
5. Engineers can use Copilot to propose fixes that handle breaking changes

## License

MIT
