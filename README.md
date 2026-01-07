# Dependabot Wolf 🐺

Automated GitHub Action that fixes stuck Dependabot security alerts using GitHub Copilot Workspace.

## What It Does

When Dependabot security alerts remain unfixed, Dependabot Wolf automatically creates PR proposals using GitHub Copilot Workspace.

**Perfect for transitive dependency vulnerabilities:**
- Vulnerability is in a sub-dependency (e.g., `qs` via `body-parser`)
- Dependabot creates PR to update parent dependency without explaining why
- Teams reject/ignore PRs without security context
- **Wolf provides full context + Copilot assistance** to understand and fix

**"Stuck" alerts are those without open PRs:**
- Dependabot PR was closed (unclear purpose, breaking changes, rejected)
- Dependabot couldn't create a PR (dependency conflicts, peer mismatches)
- Alert is ignored (team doesn't understand the transitive relationship)

**Dependabot Wolf:**
1. Finds all open Dependabot alerts without open PRs
2. Creates draft PR with CVE details, affected packages, and dependency tree context
3. Tags `@github-copilot workspace` for AI-assisted analysis
4. Engineers use Copilot to understand transitive relationships and implement fix

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

This repo demonstrates **glob CLI command injection vulnerability (CVE-2025-64756)** via transitive dependency - matching real-world scenario from [playlab PR #2234](https://github.com/playlab-education/playlab/pull/2234).

### The Scenario
```
package.json: rimraf@5.0.0 (devDependency)
  └─> glob@10.4.5 (VULNERABLE - CVE-2025-64756, GHSA-5j98-mcp5-4vw2)
```

We have `glob` locked at 10.4.5 via `overrides` to simulate a dependency conflict.

**The Vulnerability:**
- glob CLI versions 10.2.0 to 10.4.5 have command injection via `-c/--cmd` option
- Allows arbitrary command execution through malicious filenames
- Patched in glob@10.5.0+

**Why this is tricky:**
- `glob` is a **transitive dependency** (not directly installed)
- Dependabot detects vulnerability but solution isn't obvious
- Fix requires updating **parent dependency** (rimraf 5.0.0 → 6.1.2)
- rimraf@6.x depends on glob@^13.0.0 (patched version)
- Understanding transitive relationship is key

**When Wolf activates:**
1. Dependabot detects glob vulnerability (no open PR due to override lock)
2. Wolf creates draft PR that:
   - Updates rimraf from 5.0.0 to 6.1.2 (which uses glob@^13.0.0)
   - Removes glob override from package.json
   - Regenerates package-lock.json with patched glob
3. PR includes CVE details + `@github-copilot workspace` for AI-assisted review
4. Engineer reviews and merges the fix

**Real-world parallel:** This matches the [playlab PR #2234](https://github.com/playlab-education/playlab/pull/2234) where `@sentry/vite-plugin@3.3.1` → `glob@vulnerable` required updating `@sentry/vite-plugin` to `4.6.1`, not glob directly.

**The Fix:**
```diff
  "devDependencies": {
-   "rimraf": "5.0.0"
+   "rimraf": "^6.1.2"
  },
- "overrides": {
-   "glob": "10.4.5"
- }
```
Result: rimraf@6.1.2 → glob@13.0.0 (patched, no vulnerability)

## License

MIT
